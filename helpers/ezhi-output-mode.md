# Helper: `input_select.ezhi_output_mode`

Dropdown that selects the output mode. All automations in this repo depend on it.

| Setting | Value |
|---|---|
| Type | Dropdown (`input_select`) |
| Name | EZHI output mode |
| Entity ID | `input_select.ezhi_output_mode` |
| Options | `Zero export`, `Stable output (auto)`, `Stable output (permanent)`, `No output` |

Option names must match exactly (case, spaces and brackets), because the automations compare against them.

## Modes

| Mode | Behavior |
|---|---|
| `Zero export` | The control loop keeps grid import at about 4 W |
| `Stable output (auto)` | Fixed output (40 W). Set by the automations on low battery or connectivity problems. The recovery automations may switch back to `Zero export` |
| `Stable output (permanent)` | Fixed output (40 W). Select it yourself: no automation changes the mode until you do |
| `No output` | The power limit is set to 0 W and re-applied every minute. Set automatically when local mode is switched off, or select it yourself: no automation switches out of it |

`No output` needs a power limit entity that accepts 0. Check the `min` attribute of the entity in Developer Tools > States.

## Create it in the UI

Settings > Devices & services > Helpers > Create helper > Dropdown.

If you are renaming an existing helper or option, first set the dropdown to `Zero export`, so the current state is not left on a name that no longer exists. After renaming the entity ID, check that every automation uses the new one.

## Or in `configuration.yaml`

```yaml
input_select:
  ezhi_output_mode:
    name: EZHI output mode
    options:
      - Zero export
      - Stable output (auto)
      - Stable output (permanent)
      - No output
    icon: mdi:solar-power
```
