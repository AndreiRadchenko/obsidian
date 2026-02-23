---
date_created: 2026-01-18
date_modified: 2026-01-18
document_type: note
project: Culture Case
tags:
  - culture-case note
  - Music_Assistant
  - HA_Media_Player
  - "#Culture_case"
---
[[Projects/Culture Case/Home|Home]] | [[Projects/Culture Case/Meetings/All Meetings|Meetings]] | [[Projects/Culture Case/Notes/All Notes|Notes]] | [[Projects/Culture Case/References|References]]
# Music Assistant
**Overview**
Description:: 

## Note
## To achieve 20 independent media players from 20 USB audio cards in Home Assistant (HA) for use with Music Assistant (MA), 
you successfully implemented a multi-instance Squeezelite architecture. This setup leverages the **Slimproto** protocol to allow separate music programs and individual control over each zone.

### Core Configuration Logic
The key to your success was bypassing direct hardware (ALSA) locks and instead using the virtualized **PulseAudio Sink Names** managed by Home Assistant OS. This ensures that each player is mapped to a specific USB card without causing hardware conflicts.[1][2]

| Component | Setting Used | Purpose |
| :--- | :--- | :--- |
| **Add-on** | Squeezelite (Multi-instance) | Runs 20 separate player processes |
| **Build** | `pulse` | Uses the HAOS sound server for hardware access |
| **Output** | `alsa_output.usb-KTMicro_...analog-stereo` | Locks a player to a specific physical USB card |
| **MAC Address** | `00:00:00:00:00:XX` (Unique) | Allows MA to identify 20 distinct devices |

![[squeezelite.jpeg.jpeg| center | 800]]
### Implementation Steps
1.  **Identify Cards**: Use the Squeezelite add-on logs to find the unique PulseAudio sink names for all 20 KTMicro cards (e.g., those ending in `.2`, `.3`, etc.).[3]
2.  **Configure Players**: Install 20 instances of the Squeezelite add-on, assigning each a unique output sink name and a unique MAC address.
3.  **Enable Music Assistant**: In the MA dashboard, navigate to **Settings > Player Providers** and enable the **Slimproto** provider.[4][5]
4.  **Manage Zones**: The 20 players will automatically appear in Music Assistant, allowing you to assign different tracks, group players, or adjust volumes independently.[6][7]

## Another way 
### [HA Music (or just voice notifications) with multiroom sound setup and with multiple USB sound cards](https://community.home-assistant.io/t/ha-music-or-just-voice-notifications-with-multiroom-sound-setup-and-with-multiple-usb-sound-cards/920404)
### [snapcast github](https://github.com/trianglesis/pub_commands/blob/dc71ebaf43c46c9ae19f253a12275ef8c40e1951/raspberry/setup/sound/snapcast/multiroom_milty_card_usb_sound.md)


### Essential Documentation Links
*   **Music Assistant Player Support**: [official-documentation](https://www.music-assistant.io/player-support/) — Details on how Slimproto interacts with the MA engine.[8]
*   **Installation Guide**: [music-assistant-ha](https://www.music-assistant.io/installation/) — Instructions for setting up the server and provider ecosystem.[9]
*   **Community Discussion**: [Sound Card Multi-Zone Setup](https://github.com/orgs/music-assistant/discussions/3492) — A specific thread for mapping identical USB cards to separate players.[3]
*   **Music Assistant Blog**: [Version 2.0 Feature Guide](https://www.home-assistant.io/blog/2024/05/09/music-assistant-2/) — Explains how MA handles multiple player types and synchronized streaming.[10]

Sources
[1] Music Assistant 2.0: Your Music, Your Players - Page 50 - Blog https://community.home-assistant.io/t/music-assistant-2-0-your-music-your-players/727626?page=50
[2] HA Music (or just voice notifications) with multiroom sound ... https://community.home-assistant.io/t/ha-music-or-just-voice-notificati
[8] Player Support - Music Assistant https://www.music-assistant.io/player-support/
[9] Installation https://www.music-assistant.io/installation/
[10] Music Assistant 2.0: Your Music, Your Players https://www.home-assistant.io/blog/2024/05/09/music-assistant-2/


## References


---
[[Projects/Culture Case/Home|Home]] | [[Projects/Culture Case/Meetings/All Meetings|Meetings]] | [[Projects/Culture Case/Notes/All Notes|Notes]] | [[Projects/Culture Case/References|References]]
