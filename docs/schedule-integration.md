# Schedule Integration

The Climate AC Controller blueprint does **not** manage the time schedule itself. It relies on an external `input_select` to know the current period (Morning / Day / Evening / Night / Away / Holiday).

This page explains how to wire up that schedule.

---

## Option A — Hydronic Heating Schedule blueprint (recommended)

If you already use the [Hydronic Heating Schedule](https://github.com/CVKBaca/homeassistant-hydronic-heating) blueprint for heating, it already manages an `input_select` with exactly the right options. Simply point the AC Controller at the same entity.

```
input_select.heating_mode  ←  managed by Hydronic Heating Schedule
                           ←  managed by Hydronic Presence Controller
                           ←  read by Climate AC Controller
```

**Benefits:**
- Zero duplication — one schedule drives both heating and cooling
- Presence-based Away/Holiday transitions are shared
- Time-of-day transitions happen at the same times for both systems
- If you change the Evening start time in the heating schedule, the AC follows automatically

### How the two systems stay in sync

| Event | Heating Schedule sets mode to | AC Controller reacts |
|-------|-------------------------------|----------------------|
| 06:00 | `Morning` | Switches to morning cooling target |
| 07:00 | `Day` | Switches to day cooling target |
| Sunset (± offset) | `Evening` | Switches to evening cooling target |
| 22:00 | `Night` | Switches to night cooling target |
| Everyone leaves | `Away` (Presence Controller) | Switches to away target (or turns off) |
| Away 18h+ | `Holiday` (Presence Controller) | Switches to holiday target (or turns off) |
| Someone arrives home | Mode restored by time | AC resumes normal schedule |

---

## Option B — Standalone input_select with your own automation

If you don't use the Hydronic Heating blueprints, create your own automation to manage the `input_select`.

### 1. Create the input_select

**Settings → Helpers → Create Helper → Dropdown**

Options (exact spelling required):
```
Morning
Day
Evening
Night
Away
Holiday
```

### 2. Create a schedule automation

Example automation that transitions between time periods:

```yaml
alias: Climate Mode Schedule
triggers:
  - trigger: time
    at: "06:00:00"
    id: morning
  - trigger: time
    at: "07:00:00"
    id: day
  - trigger: sun
    event: sunset
    id: evening
  - trigger: time
    at: "22:00:00"
    id: night
actions:
  - choose:
    - conditions:
        - condition: trigger
          id: morning
      sequence:
        - action: input_select.select_option
          target:
            entity_id: input_select.climate_mode
          data:
            option: Morning
    - conditions:
        - condition: trigger
          id: day
      sequence:
        - action: input_select.select_option
          target:
            entity_id: input_select.climate_mode
          data:
            option: Day
    - conditions:
        - condition: trigger
          id: evening
      sequence:
        - action: input_select.select_option
          target:
            entity_id: input_select.climate_mode
          data:
            option: Evening
    - conditions:
        - condition: trigger
          id: night
      sequence:
        - action: input_select.select_option
          target:
            entity_id: input_select.climate_mode
          data:
            option: Night
```

### 3. Create a presence automation

Example automation that handles Away/Holiday:

```yaml
alias: Climate Mode Presence
triggers:
  - trigger: state
    entity_id: group.all_persons
    to: not_home
    id: left
  - trigger: state
    entity_id: group.all_persons
    to: not_home
    for:
      hours: 18
    id: vacation
  - trigger: state
    entity_id: group.all_persons
    to: home
    id: home
actions:
  - choose:
    - conditions:
        - condition: trigger
          id: left
      sequence:
        - action: input_select.select_option
          target:
            entity_id: input_select.climate_mode
          data:
            option: Away
    - conditions:
        - condition: trigger
          id: vacation
      sequence:
        - action: input_select.select_option
          target:
            entity_id: input_select.climate_mode
          data:
            option: Holiday
    - conditions:
        - condition: trigger
          id: home
      sequence:
        # Restore time-based mode when someone returns
        - action: input_select.select_option
          target:
            entity_id: input_select.climate_mode
          data:
            option: >
              {% set h = now().hour %}
              {% if h >= 6 and h < 7 %}Morning
              {% elif h >= 7 and is_state('sun.sun', 'above_horizon') %}Day
              {% elif is_state('sun.sun', 'below_horizon') and h < 22 %}Evening
              {% else %}Night{% endif %}
```

---

## Away vs Holiday — what's the difference?

| Mode | Triggered by | Typical use | Recommended AC target |
|------|-------------|-------------|----------------------|
| `Away` | Everyone left home | At work, short trip | 27 °C — less aggressive cooling, saves energy |
| `Holiday` | Away for 18+ hours | Vacation | 29 °C — only prevents extreme overheating |

If you leave `away_temperature` or `holiday_temperature` unconfigured in the blueprint, the AC stays **off** in that mode.
