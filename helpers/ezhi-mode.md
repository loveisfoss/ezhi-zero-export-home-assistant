# Helper: `input_select.ezhi_mode`

Dropdown that selects the operating mode. All automations in this repo depend on it.

| Setting | Value |
|---|---|
| Type | Dropdown (`input_select`) |
| Name | EZHI mode |
| Entity ID | `input_select.ezhi_mode` |
| Options | `Zero export`, `Stable output` |

Option names must match exactly (case and spaces), because the automations compare against them.

## Create it in the UI

Settings > Devices & services > Helpers > Create helper > Dropdown.

## Or in `configuration.yaml`

```yaml
input_select:
  ezhi_mode:
    name: EZHI mode
    options:
      - Zero export
      - Stable output
    icon: mdi:solar-power
```
