# Helper: `input_boolean.ezhi_high_power_mode`

Switch that raises the maximum power limit of the zero export loop from 800 W to 1200 W.

| Setting | Value |
|---|---|
| Type | Toggle (`input_boolean`) |
| Name | EZHI high power mode |
| Entity ID | `input_boolean.ezhi_high_power_mode` |

## What it does

- **Off:** [`zero-export.yaml`](../automations/zero-export.yaml) never sets a limit above 800 W.
- **On:** it may set a limit up to 1200 W.
- **Switching it off:** [`high-power-mode-off-clamp.yaml`](../automations/high-power-mode-off-clamp.yaml) lowers the limit to 800 W immediately if it is above that.
- Stable output is not affected, because the fixed limit (40 W) is below both maximums.

## Before you use it

- Check that your power limit number entity accepts values up to 1200 (Developer Tools > States, attribute `max`).
- Make sure the inverter, the battery's discharge rating and the circuit protection are rated for it, and check your local rules. The automation only sets the limit, it cannot make the hardware deliver it.
- The loop works from the limit it last wrote, not from the measured output. If the inverter cannot deliver the full limit, a sudden load drop can cause more export than at 800 W.

## Create it in the UI

Settings > Devices & services > Helpers > Create helper > Toggle.

## Or in `configuration.yaml`

```yaml
input_boolean:
  ezhi_high_power_mode:
    name: EZHI high power mode
    icon: mdi:flash
```
