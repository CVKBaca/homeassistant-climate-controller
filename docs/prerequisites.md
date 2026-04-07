# Prerequisites

Before creating an automation from the Climate AC Controller blueprint, you need to set up the following helpers in Home Assistant.

---

## 1. Climate Mode Selector (`input_select`)

The blueprint requires an `input_select` helper that tracks the current period of the day and presence state. It **must** have exactly these options (case-sensitive):

| Option | When active |
|--------|-------------|
| `Morning` | Early morning (e.g. 6:00–7:00) |
| `Day` | Daytime until sunset |
| `Evening` | After sunset until late evening |
| `Night` | Late evening / overnight |
| `Away` | Everyone has left home |
| `Holiday` | House empty for an extended period |

### Option A — Create manually

Go to **Settings → Devices & Services → Helpers → Create Helper → Dropdown** and create an `input_select` with the six options listed above.

Suggested entity ID: `input_select.climate_mode`

You will then need a separate automation to transition between Morning/Day/Evening/Night based on time, and to set Away/Holiday based on presence. See [schedule-integration.md](schedule-integration.md) for a ready-made solution.

### Option B — Use Hydronic Heating Schedule (recommended)

If you already use the [Hydronic Heating Schedule](https://github.com/CVKBaca/homeassistant-hydronic-heating) blueprint, it manages this `input_select` automatically — you just point the AC Controller at the same entity.

---

## 2. Cooling Temperature Helpers (`input_number`)

You need one `input_number` for each active time period. Create them via **Settings → Devices & Services → Helpers → Create Helper → Number**.

| Helper | Suggested entity ID | Suggested range | Example value |
|--------|---------------------|-----------------|---------------|
| Morning cooling target | `input_number.cooling_temperature_morning` | 18–28 °C | 25 °C |
| Day cooling target | `input_number.cooling_temperature_day` | 18–28 °C | 24 °C |
| Evening cooling target | `input_number.cooling_temperature_evening` | 18–28 °C | 24 °C |
| Night cooling target | `input_number.cooling_temperature_night` | 18–28 °C | 25 °C |

### Optional — Away and Holiday targets

If you want the AC to maintain a minimum temperature while away (instead of staying fully off):

| Helper | Suggested entity ID | Suggested range | Example value |
|--------|---------------------|-----------------|---------------|
| Away cooling target | `input_number.cooling_temperature_away` | 19–32 °C | 27 °C |
| Holiday cooling target | `input_number.cooling_temperature_holiday` | 19–32 °C | 29 °C |

> **Tip:** Holiday temperature is typically higher than Away — when you are on vacation for several days, you only want to prevent extreme overheating (e.g. for plants or electronics), not actively cool the house.

If these helpers are not configured in the blueprint, the AC will simply stay off in Away/Holiday mode.

---

## 3. Indoor Temperature Sensor

Any `sensor` entity with `device_class: temperature`. Examples:
- Aqara temperature sensor
- ESPHome multisensor
- Zigbee thermometer

Use a sensor that reflects the general indoor temperature of the house, not a single room.

---

## 4. AC Climate Entity

Any `climate` entity in Home Assistant that supports `cool` mode. Compatible with:
- Midea (native integration or SmartIR)
- Daikin, Fujitsu, Mitsubishi, LG, Samsung, etc.
- Any integration that exposes a `climate` entity with `cool` hvac_mode

---

## Next step

Once all helpers are ready, go to [installation.md](installation.md).
