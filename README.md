# TRV dial to Versatile Thermostat

A Home Assistant blueprint that lets people change the temperature on the **radiator valve itself** and have that change carried over to the [Versatile Thermostat](https://github.com/jmcollin78/versatile_thermostat) (VTherm) that controls it.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2F%3Cm0g3ns%3E%2Fha-trv-dial-to-vtherm%2Fblob%2Fmain%2Ftrv_dial_to_vtherm.yaml)

## Why this exists

If a VTherm uses `over_climate` with the self-regulation mode **Direct control of valve** (the recommended setup for the Sonoff TRVZB), VTherm drives the valve opening itself through a `number` entity. The setpoint it sends to the TRV's climate entity is mostly there for display. Turning the dial on the TRV changes a value that nothing listens to, so the heating carries on as before.

This blueprint listens for setpoint changes on the TRV and, when one doesn't match the VTherm's target, sets that value as the VTherm's new target.

## Requirements

- Home Assistant 2024.10 or newer
- A Versatile Thermostat of type `over_climate` that controls the TRV
- A TRV whose setpoint changes show up in Home Assistant (tested with Sonoff TRVZB via Zigbee2MQTT)

## Installation

1. Click the import button above, or go to **Settings → Automations & Scenes → Blueprints → Import Blueprint** and paste:

   ```
   https://github.com/m0g3ns/ha-trv-dial-to-vtherm/blob/main/trv_dial_to_vtherm.yaml
   ```

2. Go to **Settings → Automations & Scenes → Create automation**, choose this blueprint, and fill in:
   - **TRV climate entity**: the physical TRV (e.g. `climate.living_room_thermostat`)
   - **Versatile Thermostat entity**: the VTherm controlling it (e.g. `climate.living_room_versatile_thermostat`)

3. Repeat for every TRV. Name each automation after its room, so traces are easy to read.

If one VTherm controls several TRVs, create one automation per TRV, all pointing at the same VTherm. Turning any dial then updates the VTherm, which passes the new value on to the other TRVs.

## How it works

1. The automation triggers when the `temperature` attribute of the TRV's climate entity changes.
2. It compares the new value with the VTherm's current target.
3. If they differ, it calls `climate.set_temperature` on the VTherm with the new value.

When VTherm or a scheduler changes the target, VTherm updates its own target first and then writes it to the TRV. By the time the TRV reports the new value the two already match, so the automation does nothing. Only a real dial change creates a mismatch, which avoids feedback loops.

## Tips

- **Set the VTherm's target temperature step to 0.5.** TRVs like the TRVZB work in 0.5 °C steps. If the VTherm holds a value such as 20.3, the TRV's value (20.5) never matches it, and the VTherm's target would jump on every update.
- **Schedulers still win at the next slot.** If you use the [Scheduler](https://github.com/nielsfaber/scheduler-component) integration or VTherm presets with automations, a dial change lasts until the next scheduled change. For a longer manual override, pause the schedule or use a timer in addition to this blueprint.
- **Don't lock the VTherm against users.** VTherm's lock feature can reject `climate.set_temperature` calls. With the lock on, dial changes won't get through.
- **The TRV's child lock must be off.** Otherwise the dial can't be turned.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| VTherm doesn't change after turning the dial | In **Developer Tools → States**, check whether the TRV's `temperature` attribute updates. If not, the problem is on the Zigbee side (e.g. slow reporting in Zigbee2MQTT), not in this blueprint. |
| TRV updates but VTherm doesn't | Open the automation's **Traces** and see whether the condition blocked it. A value mismatch from step size is the most common cause. |
| Target keeps jumping back | A scheduler or another automation is overwriting it. See the tips above. |
| Action fails | The VTherm may be locked, or `climate.set_temperature` is not allowed in its current state (e.g. unavailable). |

## Contributing

Issues and pull requests are welcome. If you tested this with TRVs other than the Sonoff TRVZB, or with ZHA instead of Zigbee2MQTT, please say so in an issue so the requirements section can be updated.

## License

[MIT](LICENSE)