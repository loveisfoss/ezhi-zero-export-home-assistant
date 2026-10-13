# Helper: `timer.ezhi_cooldown`

Short timer used by the zero export automation as a pause between two writes to the inverter.

| Setting | Value |
|---|---|
| Type | Timer |
| Name | EZHI cooldown |
| Entity ID | `timer.ezhi_cooldown` |
| Duration | `00:00:03` (3 seconds) |
| Restore on restart | Off (not needed for a 3 second timer) |

The duration is set only here: [`zero-export.yaml`](../automations/zero-export.yaml) starts the timer without its own duration, so change the cooldown in this helper's settings.

## Create it in the UI

Settings > Devices & services > Helpers > Create helper > Timer.

## Or in `configuration.yaml`

```yaml
timer:
  ezhi_cooldown:
    name: EZHI cooldown
    duration: "00:00:03"
    restore: false
```
