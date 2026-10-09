# EZHI zero export for Home Assistant

Home Assistant automations that keep an APsystems EZHI hybrid microinverter, running in Local mode, at (almost) zero grid export, using a grid meter such as a Shelly Pro EM-50. They also protect the battery by dropping to a fixed output when the state of charge is low, and warn you if a sensor goes offline.

> Use at your own risk. Test carefully and check the rules for balcony/plug-in PV in your country. This is a hobby project, not electrical or legal advice.

## My setup

| Part | Model |
|---|---|
| Inverter | APsystems EZHI |
| Grid meter | Shelly Pro EM-50 (LAN connection) |
| Battery | EPEVER LFP2.71KWH51.2V-P65H1 ([manual](https://solarv.de/Datenblatt/2025/09/epever/LFP2.71KWH51.2V-P65H1-Manual-EN-V1.0.pdf)) |
| Panels | 2x Luxor 460W Mono N-Type TOPCon Glass-Glass Bifacial (LX-460M/182R-96+GG) |
| Home Assistant integration | [kamilkosek/EZHI](https://github.com/kamilkosek/EZHI) |

## How it works

The grid meter reports power in watts: **positive = importing, negative = exporting**.

In **Zero export** mode, every time the meter value changes the automation calculates:

```
new limit = current limit + grid power - 4 W
```

clamped to 30 to 800 W. It only acts when grid power is outside the 0 to 8 W margin, and waits for a 5 second cooldown after each write.

In **Stable output** mode the inverter is held at a fixed limit (40 W by default).

The mode is selected with a dropdown helper. Automations switch to Stable output on low battery or connectivity problems, and back to Zero export when the battery has recovered.

## Requirements

- Home Assistant with the [EZHI integration](https://github.com/kamilkosek/EZHI) (or any other way to set the EZHI on-grid power limit as a `number` entity)
- EZHI set to Local mode
- A grid meter exposing signed power in W (positive = import)
- A battery state of charge sensor in %
- The two helpers below
- Optional: the Home Assistant companion app for phone notifications

## Helpers

| Helper | Purpose |
|---|---|
| [`input_select.ezhi_mode`](helpers/ezhi-mode.md) | Mode selector: `Zero export` / `Stable output` |
| [`timer.ezhi_cooldown`](helpers/ezhi-cooldown.md) | 5 second pause between inverter writes |

## Automations

| File | What it does |
|---|---|
| [`zero-export.yaml`](automations/zero-export.yaml) | The control loop: adjusts the power limit to keep grid import at about 4 W |
| [`stable-output-on-selection.yaml`](automations/stable-output-on-selection.yaml) | Sets the fixed limit (40 W) when you select Stable output |
| [`stable-output-timed-check.yaml`](automations/stable-output-timed-check.yaml) | Re-applies the fixed limit every minute while in Stable output |
| [`low-battery-mode.yaml`](automations/low-battery-mode.yaml) | Switches to Stable output when SoC drops below 17 % |
| [`battery-recovered.yaml`](automations/battery-recovered.yaml) | Switches back to Zero export when SoC rises above 17 % |
| [`connectivity-problem.yaml`](automations/connectivity-problem.yaml) | Switches to Stable output and notifies you if a sensor goes unavailable |

## Setup

1. Create the two [helpers](#helpers).
2. For each automation file, replace every placeholder written in capital letters (see the table below). Placeholders are in the form `sensor.YOUR_...`, `number.YOUR_...` and `notify.mobile_app_YOUR_PHONE`.
3. Create the automations in Home Assistant (Settings > Automations & scenes > Create automation > three dots > Edit in YAML) and paste each file.
4. Start in Stable output mode, then switch to Zero export once you have checked that the values look right.

### Placeholders

| Placeholder | Replace with |
|---|---|
| `sensor.YOUR_GRID_POWER_SENSOR` | Grid meter power sensor in W (positive = import, negative = export) |
| `number.YOUR_EZHI_POWER_LIMIT` | Number entity that sets the EZHI on-grid power limit |
| `sensor.YOUR_EZHI_BATTERY_SOC_SENSOR` | Battery state of charge sensor in % |
| `notify.mobile_app_YOUR_PHONE` | Notify service of your phone |

## Things to know

- **Fixed values.** The 4 W target, the 0 to 8 W margin, the 30 to 800 W range, the 40 W stable output and the 17 % battery thresholds are my values. Each is explained in the comments at the top of its file.
- **Setpoint based.** The control loop works from the limit it last wrote, not from the inverter's measured output, because the inverter output sensor on my setup only updates every few minutes. If your sensor updates quickly, using the real output can be more accurate.
- **Battery thresholds.** The recovery automation fires on any upward crossing of its threshold, even if you selected Stable output manually. Keep a gap between the low and recovered values if the SoC tends to hover around them.
- **No automatic return after connectivity problems.** You are notified and switch back to Zero export yourself.

## Contributing

Issues and pull requests are welcome, especially for other inverters and meters.
