---
date_created: 2026-06-13
date_modified: 2026-06-13
document_type: note
project: Suppoint-bot
tags: suppoint-bot note
---
[[Projects/Suppoint-bot/Home|Home]] | [[Projects/Suppoint-bot/Meetings/All Meetings|Meetings]] | [[Projects/Suppoint-bot/Notes/All Notes|Notes]] | [[Projects/Suppoint-bot/References|References]]
# Odroid
**Overview**
Description:: 

## Note

OneLiner to increase cpu load:
```
systemd-run --unit=odroid-stress --property=CPUAccounting=1 --property=CPUQuota=30% taskset -c 0 stress-ng --cpu 1 --cpu-method matrixprod --cpu-load 30 --timeout 600m --metrics-brief --temp-path /tmp
```
To stop process use:
```
sudo systemctl stop odroid-stress
```
## References


---
[[Projects/Suppoint-bot/Home|Home]] | [[Projects/Suppoint-bot/Meetings/All Meetings|Meetings]] | [[Projects/Suppoint-bot/Notes/All Notes|Notes]] | [[Projects/Suppoint-bot/References|References]]
