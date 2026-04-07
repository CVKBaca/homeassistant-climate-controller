# Installation

## Step 1 — Set up prerequisites

Follow [prerequisites.md](prerequisites.md) to create all required helpers before continuing.

---

## Step 2 — Import the blueprint

### Via button

[![Open your Home Assistant instance and show the blueprint import dialog.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FCVKBaca%2Fhomeassistant-climate-controller%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Fclimate_ac_controller.yaml)

### Manually

1. Go to **Settings → Automations & Scenes → Blueprints**
2. Click **Import Blueprint** (bottom right)
3. Paste this URL:
   ```
   https://github.com/CVKBaca/homeassistant-climate-controller/blob/master/blueprints/automation/climate_ac_controller.yaml
   ```
4. Click **Preview** → **Import**

---

## Step 3 — Create an automation from the blueprint

1. Go to **Settings → Automations & Scenes → Automations**
2. Click **Create Automation**
3. Select **Climate AC Controller** from the blueprint list
4. Fill in the parameters:

### Required

| Field | Value |
|-------|-------|
| AC Climate Entity | your `climate.*` entity |
| Indoor Temperature Sensor | your indoor `sensor.*` entity |
| Heating/presence mode entity | your `input_select.*` (see prerequisites) |
| Morning target temperature | `input_number.cooling_temperature_morning` |
| Day target temperature | `input_number.cooling_temperature_day` |
| Evening target temperature | `input_number.cooling_temperature_evening` |
| Night target temperature | `input_number.cooling_temperature_night` |

### Optional but recommended

| Field | Value |
|-------|-------|
| Door sensors | `binary_sensor.*` for balcony/terrace doors |
| Away mode target temperature | `input_number.cooling_temperature_away` |
| Holiday mode target temperature | `input_number.cooling_temperature_holiday` |

### Tuning (defaults are a good starting point)

| Field | Default | Notes |
|-------|---------|-------|
| Turn-on offset | 0.4 °C | Increase if AC starts too eagerly |
| Turn-off offset | 0.3 °C | Increase for more hysteresis |
| Turn-off delay | 120 min | Prevents AC cycling on/off |
| Season start month | 5 (May) | |
| Season end month | 9 (September) | |

5. Save the automation.

---

## Step 4 — Set your temperature targets

Go to your HA dashboard (or **Settings → Helpers**) and set values for your `cooling_temperature_*` input_number helpers to your desired comfort temperatures.

Typical starting values:

| Period | Target |
|--------|--------|
| Morning | 25 °C |
| Day | 24 °C |
| Evening | 24 °C |
| Night | 25 °C |
| Away | 27 °C |
| Holiday | 29 °C |

---

## Step 5 — Verify

Check that the automation appears as **on** in **Settings → Automations**. On the next hot day the AC should start automatically when indoor temperature exceeds your Day target + 0.4 °C.
