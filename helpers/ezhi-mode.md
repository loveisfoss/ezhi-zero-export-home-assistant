# Helper: `input_select.ezhi_mode`

Dropdown that selects the operating mode. All automations in this repo depend on it.

| Setting | Value |
|---|---|
| Type | Dropdown (`input_select`) |
| Name | EZHI mode |
| Entity ID | `input_select.ezhi_mode` |
| Options | `Zero export`, `Stable output (auto)`, `Stable output (permanent)` |

Option names must match exactly (case, spaces and brackets), because the automations compare against them.

## Modes

| Mode | Behavior |
|---|---|
| `Zero export` | The control loop keeps grid import at about 4 W |
| `Stable output (auto)` | Fixed output (40 W). Set by the automations on low battery or connectivity problems. `battery-recovered.yaml` may switch back to `Zero export` |
| `Stable output (permanent)` | Fixed output (40 W). Select it yourself: no automation changes the mode until you do |

## Create it in the UI

Settings > Devices & services > Helpers > Create helper > Dropdown.

If you are renaming an existing `Stable output` option, first set the dropdown to `Zero export`, so the current state is not left on a name that no longer exists.

## Or in `configuration.yaml`

```yaml
input_select:
  ezhi_mode:
    name: EZHI mode
    options:
      - Zero export
      - Stable output (auto)
      - Stable output (permanent)
    icon: mdi:solar-power
```
