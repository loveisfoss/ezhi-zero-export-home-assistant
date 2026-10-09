# Helper: `timer.ezhi_cooldown`

Short timer used by the zero export automation as a pause between two writes to the inverter.

| Setting | Value |
|---|---|
| Type | Timer |
| Name | EZHI cooldown |
| Entity ID | `timer.ezhi_cooldown` |
| Duration | `00:00:05` (5 seconds) |
| Restore on restart | Off (not needed for a 5 second timer) |

If you change the duration here, also change the `duration: '00:00:05'` value in
[`zero-export.yaml`](../automations/zero-export.yaml), because the automation starts the timer with its own duration.

## Create it in the UI

Settings > Devices & services > Helpers > Create helper > Timer.

## Or in `configuration.yaml`

```yaml
timer:
  ezhi_cooldown:
    name: EZHI cooldown
    duration: "00:00:05"
    restore: false
```
