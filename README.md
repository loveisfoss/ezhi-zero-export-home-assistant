# EZHI zero export for Home Assistant

Home Assistant automations that keep an **APsystems EZHI** hybrid microinverter, running in **Local mode**, at (almost) zero grid export, using a grid meter such as a **Shelly Pro EM-50**. They also protect the battery by dropping to a fixed output when the state of charge is low, and warn you if a sensor goes offline.

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

clamped to 30 to 800 W (up to 1200 W while high power mode is on). It only acts when grid power is outside the 0 to 8 W margin, and waits for a 3 second cooldown after each write.

The mode is selected with a dropdown helper:

| Mode | Behavior |
|---|---|
| `Zero export` | The control loop keeps grid import at about 4 W |
| `Stable output (auto)` | The inverter is held at a fixed limit (40 W by default). Set by the automations on low battery or connectivity problems, and switched back to Zero export when the battery has recovered or the sensors have been stable again |
| `Stable output (permanent)` | Same fixed limit, but chosen by you: no automation changes the mode until you do |
| `No output` | The power limit is set to 0 W and re-applied every minute. Set when local mode is switched off, or chosen by you: no automation switches out of it |

## Requirements

- EZHI set to **Local mode**
- Home Assistant with the [EZHI integration](https://github.com/kamilkosek/EZHI) (or any other way to set the EZHI on-grid power limit as a `number` entity)
- A grid meter exposing signed power in W (positive = import)
- A battery state of charge sensor in %
- The four helpers below
- Optional: the Home Assistant companion app for phone notifications

## Helpers

| Helper | Purpose |
|---|---|
| [`input_select.ezhi_output_mode`](helpers/ezhi-output-mode.md) | Output mode selector: `Zero export` / `Stable output (auto)` / `Stable output (permanent)` / `No output` |
| [`timer.ezhi_cooldown`](helpers/ezhi-cooldown.md) | Short pause between inverter writes (3 seconds) |
| [`input_boolean.ezhi_local_mode`](helpers/ezhi-local-mode.md) | Master switch: the automations run only while it is on. Turning it off sets No output and hands the EZHI back to the cloud |
| [`input_boolean.ezhi_high_power_mode`](helpers/ezhi-high-power-mode.md) | Raises the maximum power limit of the zero export loop from 800 W to 1200 W |

## Automations

| File | What it does |
|---|---|
| [`zero-export.yaml`](automations/zero-export.yaml) | The control loop: adjusts the power limit to keep grid import at about 4 W |
| [`high-power-mode-off-clamp.yaml`](automations/high-power-mode-off-clamp.yaml) | Lowers the power limit to 800 W as soon as high power mode is switched off |
| [`stable-output-on-selection.yaml`](automations/stable-output-on-selection.yaml) | Sets the fixed limit (40 W) when either Stable output option is selected |
| [`stable-output-timed-check.yaml`](automations/stable-output-timed-check.yaml) | Re-applies the fixed limit every minute while in either Stable output mode |
| [`no-output-on-selection.yaml`](automations/no-output-on-selection.yaml) | Sets the power limit to 0 W when No output is selected |
| [`no-output-timed-check.yaml`](automations/no-output-timed-check.yaml) | Re-applies the 0 W limit every minute while in No output |
| [`local-mode-off-no-output.yaml`](automations/local-mode-off-no-output.yaml) | When local mode is switched off, sets the mode to No output and the limit to 0 W |
| [`low-battery-mode.yaml`](automations/low-battery-mode.yaml) | Switches from Zero export to Stable output (auto) when SoC drops below 17 %, and checks again when local mode is switched on |
| [`battery-recovered.yaml`](automations/battery-recovered.yaml) | Switches from Stable output (auto) back to Zero export when SoC reaches 22 %. Does nothing in the other modes |
| [`connectivity-problem.yaml`](automations/connectivity-problem.yaml) | Notifies you if a sensor goes unavailable, and switches from Zero export to Stable output (auto) |
| [`connectivity-recovery.yaml`](automations/connectivity-recovery.yaml) | Switches from Stable output (auto) back to Zero export once the SoC sensor and grid meter have been valid for 3 minutes and SoC is at 22 % or more, and notifies you. Does nothing in the other modes |

## Setup

1. Create the four [helpers](#helpers).
2. For each automation file, replace every placeholder written in capital letters (see the table below). Placeholders are in the form `sensor.YOUR_...`, `number.YOUR_...` and `notify.mobile_app_YOUR_PHONE`.
3. Create the automations in Home Assistant (Settings > Automations & scenes > Create automation > three dots > Edit in YAML) and paste each file.
4. Start in `Stable output (permanent)`, then switch to `Zero export` once you have checked that the values look right. (In `Stable output (auto)` the recovery automations would switch to Zero export on their own.)

### Placeholders

| Placeholder | Replace with |
|---|---|
| `sensor.YOUR_GRID_POWER_SENSOR` | Grid meter power sensor in W (positive = import, negative = export) |
| `number.YOUR_EZHI_POWER_LIMIT` | Number entity that sets the EZHI on-grid power limit |
| `sensor.YOUR_EZHI_BATTERY_SOC_SENSOR` | Battery state of charge sensor in % |
| `notify.mobile_app_YOUR_PHONE` | Notify service of your phone |

## Battery limits in the AP EasyPower app

My settings in the AP EasyPower app:

| Setting | Value | Effect |
|---|---|---|
| Stop discharge | 15 % | The inverter stops on-grid output when the SoC reaches 15 % |
| Discharge protection | 17 % | The inverter starts on-grid output again when the SoC reaches 17 % |

How they fit together with the automations:

- **The automations act first and line up with the inverter.** The low battery automation triggers below 17 %, which is 2 points above the level where the inverter stops. Home Assistant moves the system to the fixed 40 W output before the inverter's own limit is reached. At 40 W the remaining 2 % of a 2.71 kWh battery (roughly 55 Wh) lasts well over an hour.
- **It protects the zero export loop.** The loop works from the limit it last wrote. If the inverter stopped on-grid output at 15 % while the loop was still running, grid import would rise and the loop would keep raising the limit towards 800 W without the real output following. Switching to Stable output writes the fixed limit again, which resets that.
- **Between 15 % and 17 %.** After the inverter has stopped, the mode stays `Stable output (auto)` while the battery charges. The inverter starts again at 17 % and runs at the fixed output (40 W). The battery recovered automation then returns to `Zero export` at 22 %.
- **The margin is small at high output.** 2 % is only a few minutes at 800 W, and the SoC sensor on my setup updates about every 3 minutes. Under heavy load the inverter can reach 15 % before the automation reacts. The app settings are the real safety net, and the automations are a softer first step.
- **If you change the app values,** adjust the thresholds in [`low-battery-mode.yaml`](automations/low-battery-mode.yaml) and [`battery-recovered.yaml`](automations/battery-recovered.yaml): keep the low battery threshold above the stop level, and the recovered threshold at or above the level where the inverter starts again.

## Things to know

- **Fixed values.** The 4 W target, the 0 to 8 W margin, the 30 to 800 W range (1200 W in high power mode), the 40 W stable output and the 17 % low battery and 22 % recovery thresholds are my values, set where I want them for now. Each is explained in the comments at the top of its file.
- **Setpoint based.** The control loop works from the limit it last wrote, not from the inverter's measured output, because the inverter output sensor on my setup only updates every few minutes. If your sensor updates quickly, using the real output can be more accurate.
- **Battery thresholds.** Keep a gap between the low and recovered values if the SoC tends to hover around them, otherwise the mode can flip back and forth.
- **High power mode.** With `input_boolean.ezhi_high_power_mode` on, the zero export loop may raise the limit up to 1200 W instead of 800 W. Check that the number entity's `max` allows it and that your hardware is rated for it. If the inverter cannot deliver the full limit, the loop can overshoot, because it works from the limit it last wrote and not from the measured output. See [`ezhi-high-power-mode.md`](helpers/ezhi-high-power-mode.md).
- **Local mode switch.** Switching `input_boolean.ezhi_local_mode` off sets the mode to `No output` and the limit to 0 W once. After that no automation runs: no inverter writes, no mode changes and no notifications. In cloud mode, writes to the power limit succeed but do not change the real output. After you switch it on again, the mode stays `No output` until you select one. See [`ezhi-local-mode.md`](helpers/ezhi-local-mode.md).
- **Your choice is respected.** Select `Stable output (permanent)` or `No output` and the low battery, battery recovered, connectivity problem and connectivity recovery automations leave the mode alone (the connectivity problem automation still notifies you).
- **No delay on connectivity problems.** Even a one-second unavailable moment of the grid meter or the SoC sensor switches to `Stable output (auto)` and sends a notification. This is intentional.
- **Automatic recovery.** Once both sensors have been valid for 3 minutes and the battery is at 22 % or more, `connectivity-recovery.yaml` switches back to `Zero export` and notifies you. Use `Stable output (permanent)` if you want to decide yourself.

## Contributing

Issues and pull requests are welcome.
