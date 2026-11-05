# Adjustable values (Number helpers)

Seven Number helpers let you change the tuning of the automations from the Home Assistant dashboard, without editing any YAML.

| Entity | Name | My value | Range (step) | Unit | Used by |
|---|---|---|---|---|---|
| `input_number.ezhi_zero_export_target` | EZHI zero export target | 4 | 0 to 50 (1) | W | [`zero-export.yaml`](../automations/zero-export.yaml) |
| `input_number.ezhi_zero_export_band_min` | EZHI zero export band min | 0 | -50 to 50 (1) | W | `zero-export.yaml` |
| `input_number.ezhi_zero_export_band_max` | EZHI zero export band max | 8 | 0 to 100 (1) | W | `zero-export.yaml` |
| `input_number.ezhi_max_power_limit` | EZHI max power limit | 800 | 30 to 1200 (10) | W | `zero-export.yaml`, [`max-power-limit-clamp.yaml`](../automations/max-power-limit-clamp.yaml) |
| `input_number.ezhi_stable_power` | EZHI stable output power | 40 | 30 to 800 (5) | W | [`stable-output-on-selection.yaml`](../automations/stable-output-on-selection.yaml), [`stable-output-timed-check.yaml`](../automations/stable-output-timed-check.yaml) |
| `input_number.ezhi_low_battery_soc` | EZHI low battery level | 22 | 5 to 50 (1) | % | [`low-battery-mode.yaml`](../automations/low-battery-mode.yaml) |
| `input_number.ezhi_recovery_soc` | EZHI recovery level | 27 | 5 to 100 (1) | % | [`battery-recovered.yaml`](../automations/battery-recovered.yaml), [`connectivity-recovery.yaml`](../automations/connectivity-recovery.yaml), [`connectivity-problem.yaml`](../automations/connectivity-problem.yaml) (message) |

The values are mine. The ranges are suggestions: adjust them to your hardware.

## Set the values after creating the helpers

The Home Assistant UI has no starting-value field when you create a Number helper, and a new helper starts at its **minimum** value. Set every value right after you create the helpers, before you switch the automations on. For example, an unset zero export target would be 0 W and an unset stable output power would be 30 W.

Ways to set a value:

- Settings > Devices & services > Helpers, click the helper and type or drag the value in the dialog that opens.
- Put the helpers on a dashboard (an Entities card works) and set them there. This is also where you change them later.
- Developer Tools > Actions: `input_number.set_value` with the entity and the value.

The `initial:` option of the YAML version is not used on purpose: it would reset the value to that number at every Home Assistant restart.

## Rules of thumb

- Keep the target inside the margin: band min <= target <= band max.
- The maximum power limit cannot be higher than the maximum of your power limit entity (check its `max` attribute in Developer Tools > States). Make sure the inverter, the battery's discharge rating and the circuit protection are rated for the value you set, and check your local rules. The automations only set the limit, they cannot make the hardware deliver it.
- The zero export loop works from the limit it last wrote, not from the measured output. If the inverter cannot deliver the full maximum, a sudden load drop can cause more export than with a lower maximum.
- Lowering the maximum power limit lowers the current power limit too, once the value has stayed unchanged for 10 seconds ([`max-power-limit-clamp.yaml`](../automations/max-power-limit-clamp.yaml)).
- Keep the recovery level above the low battery level, and the low battery level above the SoC where the inverter stops (see the README).
- A changed stable output power is applied within a minute by the timed check. The zero export values apply on the next grid meter reading.
- The 30 W minimum power limit is fixed in `zero-export.yaml`.

## Create them in the UI

Settings > Devices & services > Helpers > Create helper > Number. Use the name, range, step and unit from the table, and the display mode "Input field".

## Or in `configuration.yaml`

```yaml
input_number:
  ezhi_zero_export_target:
    name: EZHI zero export target
    min: 0
    max: 50
    step: 1
    unit_of_measurement: W
    mode: box
  ezhi_zero_export_band_min:
    name: EZHI zero export band min
    min: -50
    max: 50
    step: 1
    unit_of_measurement: W
    mode: box
  ezhi_zero_export_band_max:
    name: EZHI zero export band max
    min: 0
    max: 100
    step: 1
    unit_of_measurement: W
    mode: box
  ezhi_max_power_limit:
    name: EZHI max power limit
    min: 30
    max: 1200
    step: 10
    unit_of_measurement: W
    mode: box
  ezhi_stable_power:
    name: EZHI stable output power
    min: 30
    max: 800
    step: 5
    unit_of_measurement: W
    mode: box
  ezhi_low_battery_soc:
    name: EZHI low battery level
    min: 5
    max: 50
    step: 1
    unit_of_measurement: "%"
    mode: box
  ezhi_recovery_soc:
    name: EZHI recovery level
    min: 5
    max: 100
    step: 1
    unit_of_measurement: "%"
    mode: box
```

The YAML version does not set the starting values either, so set them in the dashboard after the first start.
