---
date_created: 2026-02-23
date_modified: 2026-02-23
document_type: note
project: Culture Case
tags:
  - domain/smart-home
  - "#Culture_case"
  - HA_Media_Player
---
[[Projects/Culture Case/Home|Home]] | [[Projects/Culture Case/Meetings/All Meetings|Meetings]] | [[Projects/Culture Case/Notes/All Notes|Notes]] | [[Projects/Culture Case/References|References]]
# Android ADB Integration
**Status**: Completed  
**Last Updated**: 2026-02-22  
**Related**: Home Assistant, Android Tablet Volume Control[1]

## Note

## Overview
Configured Android Debug Bridge (ADB) integration in Home Assistant to control video playback loudness on Android tablet. Required one-time MacBook ADB setup for stable port 5555 connection.

## Prerequisites
- Android tablet/phone with Developer Options enabled[2]
- Home Assistant core integration (no deprecated add-on)[3]
- MacBook for initial ADB port setup

## Step-by-Step Setup

1. **Enable Developer Options on Android**:
   ```
   Settings → About tablet → Tap Build number 7x
   → Developer options → Enable Wireless debugging
   ```

2. **Install ADB on MacBook**:
   ```
   brew install android-platform-tools
   adb version  # Verify installation
   ```

3. **Enable Port 5555 (USB One-Time)**:
   ```
   # Connect tablet via USB cable
   adb devices  # Approve RSA key popup on tablet
   adb tcpip 5555  # "restarting in TCP mode port: 5555"
   # Disconnect USB
   ```

4. **Configure HA Android ADB Integration**:
   ```
   Settings → Devices & Services → + Add Integration
   → Android Debug Bridge → Enter: 192.168.1.103:5555
   ```

5. **Test Volume Control**:
   ```
   Developer Tools → Services → remote.send_command
   Target: remote.your_tablet_remote
   Data: {"command": "VOLUME_UP"}
   ```

## Key Commands

| Purpose | Service Call |
|---------|--------------|
| Volume Up | `remote.send_command` `{"command": "VOLUME_UP"}` [3] |
| Volume Down | `remote.send_command` `{"command": "VOLUME_DOWN"}` [3] |
| Mute | `remote.send_command` `{"command": "MUTE"}` [3] |
| Media Volume | `media_player.volume_set` `{"volume_level": 0.5}` |

## Troubleshooting Notes

**Connection Failed (Python ADB)**: Dynamic ports (39555, 43201) don't work reliably. Port 5555 via `adb tcpip 5555` required.[4]

**Network OK**: Ping 192.168.1.103 confirmed connectivity. [user input]

**Wireless Debugging**: Keep enabled; use Wakelock app to prevent sleep.[5]

## Result
✅ Volume control working for video playback on Android tablet from HA dashboard/automations.

**Next Steps**: Create HA dashboard buttons + Music Assistant integration for multi-room audio.

Sources
[2] Configure on-device developer options | Android Studio https://developer.android.com/studio/debug/dev-options
[3] How to control volume on android tablet : r/homeassistant - Reddit https://www.reddit.com/r/homeassistant/comments/1huevfq/how_to_control_volume_on_android_tablet/
[4] Not able to integrate androidtv (with built-in Python ADB) https://community.home-assistant.io/t/not-able-to-integrate-androidtv-with-built-in-python-adb/340209
[5] Android Debug Bridge https://www.home-assistant.io/integrations/androidtv/


## References


---
[[Projects/Culture Case/Home|Home]] | [[Projects/Culture Case/Meetings/All Meetings|Meetings]] | [[Projects/Culture Case/Notes/All Notes|Notes]] | [[Projects/Culture Case/References|References]]
