# Helpers for top-up mode

Two helpers support the [weekly top-up charge](../README.md#weekly-top-up-charge). The third one, the interval in days, is the number helper `input_number.ezhi_top_up_days` in [`ezhi-adjustable-values.md`](ezhi-adjustable-values.md).

| Helper | Type | Name | Entity ID |
|---|---|---|---|
| Last full charge | Date and time | EZHI last full charge | `input_datetime.ezhi_last_full_charge` |
| Top-up | Toggle | EZHI top-up | `input_boolean.ezhi_top_up` |

## What they do

- **`input_datetime.ezhi_last_full_charge`** is set by [`top-up-record-full-charge.yaml`](../automations/top-up-record-full-charge.yaml) whenever the SoC shows 100 %. A new date helper starts in 1970, so the first top-up starts at the next 09:00.
- **`input_boolean.ezhi_top_up`** is on while a top-up is in progress. [`top-up-start.yaml`](../automations/top-up-start.yaml) turns it on, or you turn it on by hand: [`top-up-toggled-on-set-no-output.yaml`](../automations/top-up-toggled-on-set-no-output.yaml) then sets `No output`. [`top-up-finish.yaml`](../automations/top-up-finish.yaml) and [`top-up-cancel.yaml`](../automations/top-up-cancel.yaml) turn it off.

## Starting and ending a top-up by hand

- Turn the toggle on to start a top-up. The mode becomes `No output`.
- To end it early, select another mode. The cancel automation turns the toggle off.
- If you select another mode and then `No output` again, turn the toggle on again. Otherwise nothing switches back to `Zero export` when the battery is full.

## Create them in the UI

Settings > Devices & services > Helpers > Create helper > Date and/or time (choose "Date and time"), and > Toggle.

## Or in `configuration.yaml`

```yaml
input_datetime:
  ezhi_last_full_charge:
    name: EZHI last full charge
    has_date: true
    has_time: true

input_boolean:
  ezhi_top_up:
    name: EZHI top-up
    icon: mdi:battery-charging-100
```
