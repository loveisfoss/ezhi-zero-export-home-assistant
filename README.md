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

clamped to 30 to 800 W. It only acts when grid power is outside the 0 to 8 W margin, and waits for a 5 second cooldown after each write.

The mode is selected with a dropdown helper:

| Mode | Behavior |
|---|---|
| `Zero export` | The control loop keeps grid import at about 4 W |
| `Stable output (auto)` | The inverter is held at a fixed limit (40 W by default). Set by the automations on low battery or connectivity problems, and switched back to Zero export when the battery has recovered or the sensors have been stable again |
| `Stable output (permanent)` | Same fixed limit, but chosen by you: no automation changes the mode until you do |

## Requirements

- EZHI set to **Local mode**
- Home Assistant with the [EZHI integration](https://github.com/kamilkosek/EZHI) (or any other way to set the EZHI on-grid power limit as a `number` entity)
- A grid meter exposing signed power in W (positive = import)
- A battery state of charge sensor in %
- The two helpers below
- Optional: the Home Assistant companion app for phone notifications

## Helpers

| Helper | Purpose |
|---|---|
| [`input_select.ezhi_mode`](helpers/ezhi-mode.md) | Mode selector: `Zero export` / `Stable output (auto)` / `Stable output (permanent)` |
| [`timer.ezhi_cooldown`](helpers/ezhi-cooldown.md) | 5 second pause between inverter writes |

## Automations

| File | What it does |
|---|---|
| [`zero-export.yaml`](automations/zero-export.yaml) | The control loop: adjusts the power limit to keep grid import at about 4 W |
| [`stable-output-on-selection.yaml`](automations/stable-output-on-selection.yaml) | Sets the fixed limit (40 W) when either Stable output option is selected |
| [`stable-output-timed-check.yaml`](automations/stable-output-timed-check.yaml) | Re-applies the fixed limit every minute while in either Stable output mode |
| [`low-battery-mode.yaml`](automations/low-battery-mode.yaml) | Switches from Zero export to Stable output (auto) when SoC drops below 17 % |
| [`battery-recovered.yaml`](automations/battery-recovered.yaml) | Switches from Stable output (auto) back to Zero export when SoC rises above 17 %. Does nothing in Stable output (permanent) |
| [`connectivity-problem.yaml`](automations/connectivity-problem.yaml) | Notifies you if a sensor goes unavailable, and switches from Zero export to Stable output (auto) |
| [`connectivity-recovery.yaml`](automations/connectivity-recovery.yaml) | Switches from Stable output (auto) back to Zero export once the SoC sensor and grid meter have been valid for 3 minutes and SoC is above 17 %, and notifies you. Does nothing in Stable output (permanent) |

## Setup

1. Create the two [helpers](#helpers).
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
- **Between 15 % and 17 %.** After the inverter has stopped, the mode stays `Stable output (auto)` while the battery charges. The inverter starts again at 17 %, and the battery recovered automation (above 17 %) then returns to `Zero export`.
- **The margin is small at high output.** 2 % is only a few minutes at 800 W, and the SoC sensor on my setup updates about every 5 minutes. Under heavy load the inverter can reach 15 % before the automation reacts. The app settings are the real safety net, and the automations are a softer first step.
- **If you change the app values,** adjust the thresholds in [`low-battery-mode.yaml`](automations/low-battery-mode.yaml) and [`battery-recovered.yaml`](automations/battery-recovered.yaml): keep the low battery threshold above the stop level, and the recovered threshold at or above the level where the inverter starts again.

## Things to know

- **Fixed values.** The 4 W target, the 0 to 8 W margin, the 30 to 800 W range, the 40 W stable output and the 17 % battery thresholds are my values, set where I want them for now. Each is explained in the comments at the top of its file.
- **Setpoint based.** The control loop works from the limit it last wrote, not from the inverter's measured output, because the inverter output sensor on my setup only updates every few minutes. If your sensor updates quickly, using the real output can be more accurate.
- **Battery thresholds.** Keep a gap between the low and recovered values if the SoC tends to hover around them, otherwise the mode can flip back and forth.
- **Your choice is respected.** Select `Stable output (permanent)` and the low battery, battery recovered, connectivity problem and connectivity recovery automations leave the mode alone (the connectivity problem automation still notifies you).
- **No delay on connectivity problems.** Even a one-second unavailable moment of the grid meter or the SoC sensor switches to `Stable output (auto)` and sends a notification. This is intentional.
- **Automatic recovery.** Once both sensors have been valid for 3 minutes and the battery is above 17 %, `connectivity-recovery.yaml` switches back to `Zero export` and notifies you. Use `Stable output (permanent)` if you want to decide yourself.

## Contributing

Issues and pull requests are welcome.
