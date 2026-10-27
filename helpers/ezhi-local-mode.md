# Helper: `input_boolean.ezhi_local_mode`

Master switch for the whole setup. Every automation in this repo runs only while it is **on**, except [`local-mode-off-no-output.yaml`](../automations/local-mode-off-no-output.yaml), which runs when it is switched **off**.

| Setting | Value |
|---|---|
| Type | Toggle (`input_boolean`) |
| Name | EZHI local mode |
| Entity ID | `input_boolean.ezhi_local_mode` |

## What it does

- **On:** the automations control the EZHI through Local mode as described in the README.
- **Off:** `local-mode-off-no-output.yaml` sets the output mode to `No output` and the power limit to 0 W once, at the moment you switch it off. After that Home Assistant makes no adjustments and sends no notifications. Use it when you put the EZHI back on the APsystems cloud. In cloud mode, writes to the power limit succeed but do not change the real output, so it makes no difference whether you switch it off before or after moving the inverter to the cloud.

## When you switch it on again

- The mode is still `No output`, so the limit stays at 0 W (the timed check re-applies it within a minute if something changed it). Select the mode you want. Nothing else happens until you do.
- If you selected another mode while local mode was off, it takes effect now:
  - The zero export loop resumes with the next grid meter reading.
  - In either Stable output mode, the fixed limit is re-applied within a minute by the timed check.
  - `low-battery-mode.yaml` runs right away and switches to `Stable output (auto)` if the battery is below its threshold.
  - `connectivity-recovery.yaml` starts its 3 minute check, if the mode is `Stable output (auto)`.

While the switch is off, selecting a mode in the dropdown changes nothing on the inverter.

## Create it in the UI

Settings > Devices & services > Helpers > Create helper > Toggle.

## Or in `configuration.yaml`

```yaml
input_boolean:
  ezhi_local_mode:
    name: EZHI local mode
    icon: mdi:lan-connect
```
