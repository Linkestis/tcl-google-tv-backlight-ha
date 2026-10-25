# TCL Google TV backlight control via ADB and Home Assistant

Set the panel backlight level of a tested TCL Google TV from Home Assistant through an ADB service command. The examples also show a small automatic controller with a temporary manual override.

## Tested on

**TCL 65Q7C-UK / Google TV / Android 14.** The backlight command and values were tested on firmware V667 and V669. Other TCL models or firmware versions may expose different commands or ranges.

| Firmware | Build date | Result |
| --- | --- | --- |
| V667 | 15 September 2026 | Tested |
| V669 | 29 September 2026 | Tested |

## Requirements

- Home Assistant with the Android TV integration connected to the TV.
- ADB debugging enabled and authorized on the TV.
- A Home Assistant entity for the TV, represented below as `media_player.tcl_tv`.
- Optional ambient-light sensor (`lux`) if the light-based controller logic is used.

For the ADB connection and input-discovery setup, see [tcl-google-tv-adb-hdmi-ha](https://github.com/Linkestis/tcl-google-tv-adb-hdmi-ha).

## Command

The tested command is sent through Home Assistant's `androidtv.adb_command` action:

```text
service call tcl_tv_display 100 i32 1 i32 LEVEL i32 0
```

Replace `LEVEL` with an integer from 0 to 100. In the tested setup, the command applies the requested panel backlight value without saving a permanent display setting. This repository documents how the command is used; it does not document TCL internal implementation details.

Level `0` makes the picture extremely dark on the tested TV, which is why the music mode in the examples uses `1` instead.

There is no separate read command in the production script. The script keeps a cached last-requested value in an `input_number`; that cache is not independent confirmation from the TV.

## Home Assistant script

The core script accepts two fields:

- `level`: clamped to 0–100.
- `force`: sends the command even when the requested value differs from the cached value by less than five points.

Without `force`, the example sends only when there is no cached value or the difference is at least five. See [examples/script_tcl_set_backlight.yaml](examples/script_tcl_set_backlight.yaml).

## Ready-made modes

The production setup includes two small wrappers:

- Music: level `1`, forced.
- Normal evening: level `45`, forced.

They are examples rather than universal recommendations. See [examples/scripts_backlight_modes.yaml](examples/scripts_backlight_modes.yaml).

## Smart controller and manual override

The full manual-override cycle is: the two-hour hold script turns on the manual override helper and starts the timer; the timer-finished automation turns the helper off and triggers the controller only when the TV is on; the source/TV-off automation clears the helper and timer when the source changes or the TV turns off; and the Home Assistant-start reconciliation automation restores an active hold or clears an expired one. The resume script also clears the helper, cancels the timer, waits briefly, and triggers the controller once.

The simplified examples in [examples/automation_smart_backlight_controller.yaml](examples/automation_smart_backlight_controller.yaml) and [examples/automations_manual_override.yaml](examples/automations_manual_override.yaml) show the pattern without private entity names or the full production decision table. The hold/resume scripts are in [examples/scripts_backlight_modes.yaml](examples/scripts_backlight_modes.yaml), and the helper and timer definitions are in [examples/helpers.yaml](examples/helpers.yaml).

## Notes and limitations

- Backlight is the panel-lighting level. It should not be described as the same thing as picture brightness, gamma, or HDR tone mapping.
- The tested range and command may differ on another TCL model or firmware.
- The cached `input_number` is not a hardware readback.
- `force` is useful after wake or source changes when the TV may ignore the first ADB call.
- The Home Assistant Android TV documentation warns about another active ADB connection and explains the Python ADB and ADB server options. Multiple ADB clients can cause connection failures or timeouts; see the [official Android TV integration documentation](https://www.home-assistant.io/integrations/androidtv/).
- No official TCL documentation was found describing this external backlight command. Treat it as tested device-specific behaviour, not a general TCL API.
- If the command stops working after a firmware update, verify it again because it is device-specific.

## Related

- [tcl-google-tv-adb-hdmi-ha](https://github.com/Linkestis/tcl-google-tv-adb-hdmi-ha) — TCL Google TV HDMI input selection.
- [IR-HDMI-Switch-HA](https://github.com/Linkestis/IR-HDMI-Switch-HA) — IR HDMI switch input selection from Home Assistant.

## Support

[Support this work on Revolut](https://revolut.me/mariannud)

Contributed by MA-Linkestis (2026).
