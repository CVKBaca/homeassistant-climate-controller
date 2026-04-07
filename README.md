# Climate AC Controller — Home Assistant Blueprint

Automatically controls a split AC unit based on **indoor temperature**, **time of day**, **season**, and **presence/away mode** — without any hardcoded thresholds.

## Features

- **Dynamic temperature targets** per time period — Morning, Day, Evening, Night
- **Hysteresis control** — configurable on/off offsets prevent short-cycling
- **Seasonal guard** — AC only activates automatically within configured months (default: May–September)
- **Away & Holiday modes** — separate cooling targets when nobody is home or the house is empty for days
- **Door/window sensors** — AC won't start if a door is open
- **Schedule-aware** — follows an `input_select` helper that tracks the current period (integrates seamlessly with the [Hydronic Heating Schedule](https://github.com/CVKBaca/homeassistant-hydronic-heating) blueprint)
- **Boot-safe** — waits 2 minutes after HA restart before evaluating state
- **Auto setpoint update** — if AC is already running when the period changes (e.g. Day → Evening), it updates the setpoint without restarting the unit

## Prerequisites

Before installing this blueprint you need:

1. A **climate entity** for your AC unit (any integration: SmartIR, Midea, etc.)
2. A **temperature sensor** measuring indoor temperature
3. An **`input_select`** helper with exactly these options (case-sensitive):
   `Morning`, `Day`, `Evening`, `Night`, `Away`, `Holiday`
4. Four **`input_number`** helpers for cooling targets (Morning / Day / Evening / Night)
5. Optionally two more `input_number` helpers for Away and Holiday targets

> See [docs/prerequisites.md](docs/prerequisites.md) for step-by-step setup.

## Installation

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FCVKBaca%2Fhomeassistant-climate-controller%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Fclimate_ac_controller.yaml)

Or manually: **Settings → Automations → Blueprints → Import Blueprint** and paste the URL of the raw YAML file.

## Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `climate_entity` | ✅ | — | Your AC climate entity |
| `temperature_sensor` | ✅ | — | Indoor temperature sensor |
| `heating_mode_entity` | ✅ | — | `input_select` with Morning/Day/Evening/Night/Away/Holiday |
| `morning_temperature` | ✅ | — | `input_number` — cooling target for Morning |
| `day_temperature` | ✅ | — | `input_number` — cooling target for Day |
| `evening_temperature` | ✅ | — | `input_number` — cooling target for Evening |
| `night_temperature` | ✅ | — | `input_number` — cooling target for Night |
| `on_offset` | — | 0.4 °C | Turn on when indoor temp > target + offset |
| `off_offset` | — | 0.3 °C | Turn off when indoor temp < target − offset |
| `off_delay` | — | 120 min | Minutes below threshold before turning off |
| `door_sensors` | — | *(none)* | Binary sensors — AC won't start if any is open |
| `away_temperature` | — | *(off)* | `input_number` — cooling target in Away mode |
| `holiday_temperature` | — | *(off)* | `input_number` — cooling target in Holiday mode |
| `season_start_month` | — | 5 (May) | First month of cooling season |
| `season_end_month` | — | 9 (September) | Last month of cooling season |

## How It Works

The blueprint reads the current value of the `heating_mode_entity` to determine which temperature target to use:

```
Morning  → morning_temperature
Day      → day_temperature
Evening  → evening_temperature
Night    → night_temperature
Away     → away_temperature  (or AC off if not configured)
Holiday  → holiday_temperature  (or AC off if not configured)
```

The `input_select` is expected to be managed externally — either by the [Hydronic Heating Schedule](https://github.com/CVKBaca/homeassistant-hydronic-heating) blueprint, or by your own automation.

### Turn-on logic

AC turns **ON** when all of these are true:
- Current month is within season
- `indoor_temp > target + on_offset`
- AC is not already in `cool` mode
- No configured door/window sensors are open

### Turn-off logic

AC turns **OFF** when:
- `indoor_temp < target − off_offset` for `off_delay` minutes, OR
- Mode changes to Away/Holiday with no temperature configured, OR
- Current month is outside the season

### Setpoint update

If the AC is already running and the mode changes (e.g. Day → Evening), the blueprint updates the setpoint **without** restarting the unit.

## Integration with Hydronic Heating Schedule

This blueprint is designed to work alongside the [Hydronic Heating Schedule](https://github.com/CVKBaca/homeassistant-hydronic-heating) blueprint family, which manages the `input_select` automatically based on time of day and presence.

When used together:
- The heating schedule transitions Morning → Day → Evening → Night based on time
- The presence controller switches to Away (everyone left) or Holiday (away 18h+)
- This blueprint reacts to every mode change and adjusts the AC accordingly

> See [docs/schedule-integration.md](docs/schedule-integration.md) for details.

## Related

- [homeassistant-hydronic-heating](https://github.com/CVKBaca/homeassistant-hydronic-heating) — heating schedule & presence controller blueprints that manage the `input_select` this blueprint relies on
