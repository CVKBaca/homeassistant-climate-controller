# Configuration Reference

Full parameter reference for the Climate AC Controller blueprint.

---

## Required parameters

### `climate_entity`
Your AC climate entity. Must support `cool` hvac_mode.

```yaml
climate_entity: climate.living_room_ac
```

---

### `temperature_sensor`
Indoor temperature sensor. Should reflect the general temperature of the house, not a single room.

```yaml
temperature_sensor: sensor.indoor_temperature
```

---

### `heating_mode_entity`
An `input_select` with exactly these options: `Morning`, `Day`, `Evening`, `Night`, `Away`, `Holiday`.

Managed externally — either by the [Hydronic Heating Schedule](https://github.com/CVKBaca/homeassistant-hydronic-heating) blueprints or your own automation. See [schedule-integration.md](schedule-integration.md).

```yaml
heating_mode_entity: input_select.climate_mode
```

---

### `morning_temperature` / `day_temperature` / `evening_temperature` / `night_temperature`
`input_number` entities holding the cooling target for each period. The AC turns on when indoor temperature exceeds `target + on_offset` and turns off when it drops below `target - off_offset`.

```yaml
morning_temperature: input_number.cooling_temperature_morning
day_temperature: input_number.cooling_temperature_day
evening_temperature: input_number.cooling_temperature_evening
night_temperature: input_number.cooling_temperature_night
```

---

## Optional parameters

### `on_offset` (default: 0.4 °C)
How many degrees above the target the indoor temperature must reach before the AC turns on.

- Higher value → AC starts less eagerly, saves more energy
- Lower value → tighter temperature control

```yaml
on_offset: 0.4
```

---

### `off_offset` (default: 0.3 °C)
How many degrees below the target the indoor temperature must drop before the AC turns off.

Combined with `on_offset`, this creates a hysteresis band. With defaults:
- AC turns ON at `target + 0.4°C`
- AC turns OFF at `target - 0.3°C`
- Total hysteresis band: 0.7 °C

```yaml
off_offset: 0.3
```

---

### `off_delay` (default: 120 minutes)
How long the indoor temperature must stay below `target - off_offset` before the AC turns off. This prevents the AC from cycling on and off too frequently.

Set lower (e.g. 30 min) for tighter control; keep high (120 min) to protect AC compressor from short-cycling.

```yaml
off_delay: 120
```

---

### `door_sensors`
List of binary sensors (doors, windows). If any sensor is `on` (open), the AC will not turn on. Useful for balcony or terrace doors — no point cooling with the door open.

```yaml
door_sensors:
  - binary_sensor.balcony_door
  - binary_sensor.terrace_door
```

---

### `away_temperature`
Cooling target when `Away` mode is active. If not set, AC stays off in Away mode.

Typical value: 27 °C — the house cools less aggressively while nobody is home.

```yaml
away_temperature: input_number.cooling_temperature_away
```

---

### `holiday_temperature`
Cooling target when `Holiday` mode is active. If not set, AC stays off in Holiday mode.

Typical value: 29 °C — only prevents extreme overheating during a multi-day absence.

```yaml
holiday_temperature: input_number.cooling_temperature_holiday
```

---

### `season_start_month` (default: 5)
Month number from which the AC is allowed to turn on automatically. Default: May (5).

```yaml
season_start_month: 5
```

---

### `season_end_month` (default: 9)
Month number after which the AC will not turn on automatically. Default: September (9).

```yaml
season_end_month: 9
```

---

## Full example

```yaml
alias: Climate - AC Controller
use_blueprint:
  path: CVKBaca/homeassistant-climate-controller/climate_ac_controller.yaml
  input:
    climate_entity: climate.living_room_ac
    temperature_sensor: sensor.indoor_temperature
    heating_mode_entity: input_select.climate_mode
    morning_temperature: input_number.cooling_temperature_morning
    day_temperature: input_number.cooling_temperature_day
    evening_temperature: input_number.cooling_temperature_evening
    night_temperature: input_number.cooling_temperature_night
    door_sensors:
      - binary_sensor.balcony_door
      - binary_sensor.terrace_door
    on_offset: 0.4
    off_offset: 0.3
    off_delay: 120
    away_temperature: input_number.cooling_temperature_away
    holiday_temperature: input_number.cooling_temperature_holiday
    season_start_month: 5
    season_end_month: 9
```
