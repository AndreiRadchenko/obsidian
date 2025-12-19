---
date_created: 2025-12-06
date_modified: 2025-12-06
document_type: note
tags:
  - note
  - "#domain/smart-home"
  - "#voice-assist"
  - "#gpt-api"
  - ai
  - "#elevenlabs"
---
[[Notes Dashboard|Notes Dashboard]] / **[[Projects/Home Assistant/GPT Voice Assist|GPT Voice Assist]]**
# GPT Voice Assist
**Overview**
Description:: Create HA voice assit

## Note



---
[[Notes Dashboard|Notes Dashboard]] / **[[Projects/Home Assistant/GPT Voice Assist|GPT Voice Assist]]**#06.12.2025
## Run whisper on remote machine:
```bash
mkdir whisperdata

docker run -itd -p 10300:10300 -v ~/whisperdata/:/data rhasspy/wyoming-whisper --model tiny-int8 --language uk
```

## Run pipe on remote machine:
```bash
mkdir piperdata

docker run -itd -p 10200:10200 -v ~/piperdata/:/data rhasspy/wyoming-piper --voice uk_UA-ukrainian_tts-medium
```