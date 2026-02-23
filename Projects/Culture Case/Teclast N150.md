---
date_created: 2026-01-17
date_modified: 2026-01-17
document_type: note
project: Culture Case
tags:
  - N150
  - BIOS
---
[[Projects/Culture Case/Home|Home]] | [[Projects/Culture Case/Meetings/All Meetings|Meetings]] | [[Projects/Culture Case/Notes/All Notes|Notes]] | [[Projects/Culture Case/References|References]]
# Teclast N150
**Overview**
Description:: 

## Note

# To enable auto power-on (also known as "Restore on AC/Power Loss") 
on the Teclast Mini PC N150, you need to access the BIOS and change the power management settings.

2. **Navigate to Power Settings:**
    - Use the arrow keys to navigate to the **Chipset** tab.
    - Select **South Cluster Configuration** (or similar, like **PCH-IO Configuration**).
3. **Configure AC Power Loss:**
    - Locate the setting named **Restore AC Power Loss** or **State After G3**.
    - Change this setting from "Power Off" to **Power On** or **Always On**.
4. **Save and Exit:**

> [!blank-container|right-medium] Relationship of Data, Information and Intelligence
> ![[IMG_5720.jpeg|right-medium | 464]]

# To connect to the Home Assistant Samba share from your Mac,
you should use the Connect to Server feature in Finder. Since your logs show NetBIOS name registration issues, connecting via your Mini PC’s static IP address is the most reliable method.
Steps to Connect on macOS
	1.	Open Finder: Ensure you are in the Finder app.
	2.	Open Connect to Server: From the top menu bar, select Go > Connect to Server… (or press  Command + K ).
	3.	Enter the Address: In the “Server Address” field, type your Mini PC’s IP address with the  smb://  prefix:
	•	Example:  smb://192.168.0.228 
	4.	Authenticate: Click Connect. When prompted for credentials, select Registered User and enter the username and password you defined in the Samba addon’s configuration (not your HA web login).
	5.	Select Volume: Choose the folder you want to mount (e.g.,  config ,  share , or  media ).
Fixes for “Connection Denied” on Mac
If your Mac fails to connect even with the correct IP, check these common HAOS/Samba settings:

# Cloudflare Tunnel 
works very well with HAOS on a bare‑metal N150 because everything runs on one box and you don’t need any router port‑forwarding. The easiest way is to use the **Cloudflared add-on** in Home Assistant.[1][2]

## 1. Prerequisites on Cloudflare
1. Create a free Cloudflare account and add your domain (or subdomain) to Cloudflare DNS.[3][4]
2. In the Cloudflare dashboard, open **Zero Trust → Networks → Tunnels → Create Tunnel** and choose **Cloudflared** as the connector type.[2][1]
3. Give it a name (e.g. `ha-n150`) and save. On the **Install connector** step, copy the **token** string from the example command; you will paste this into Home Assistant later.[1][2]

## 2. Install Cloudflared Add-on on HAOS
1. In Home Assistant UI on your N150, go to **Settings → Add-ons → Add-on Store**.  
2. Click the three dots (⋮) → **Repositories**, add this URL:  
   `https://github.com/brenner-tobias/ha-addons` or another Cloudflare Tunnel repo (e.g. `https://github.com/fzerorubigd/homeassistant-cloudflare-tunnel`).[5][6]
3. After it appears, open **Cloudflared / Cloudflare Tunnel** add-on → **Install**.  
4. Enable **Start on boot**, **Watchdog**, and **Auto update**.[2]

## 3. Configure the Tunnel in the Add-on
1. Open the add-on **Configuration** tab.  
2. Paste the **tunnel token** you copied from Cloudflare into the `Cloudflare Tunnel Token` field (if your add-on uses a simple YAML, there will be a `token:` option).[1][2]
3. Save, then go back to **Info** tab and click **Start**. Status should turn to “running/connected”.

## 4. Point Tunnel to Home Assistant
Back in the Cloudflare Zero Trust dashboard, on your tunnel page:

1. Under **Public Hostnames** (or “Add route”), create a record:  
   - **Hostname**: e.g. `ha.yourdomain.com`  
   - **Service Type**: `HTTP`  
   - **URL/Service**: `http://homeassistant:8123` or `http://<N150_LAN_IP>:8123` (HA default port).[7][2][1]
2. Save. The tunnel status should become **Healthy**, and the hostname will be created in Cloudflare DNS automatically.

## 5. Update `configuration.yaml` for Reverse Proxy
In HAOS (File Editor, Samba, or SSH), edit `configuration.yaml` to trust the Cloudflared container network:

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.30.33.0/24
```

This CIDR is the default Docker network used by HA add-ons.[8][2]
Restart Home Assistant from **Settings → System → Restart**.

## References
[Install HAOS](https://www.home-assistant.io/installation/generic-x86-64/)

---
[[Projects/Culture Case/Home|Home]] | [[Projects/Culture Case/Meetings/All Meetings|Meetings]] | [[Projects/Culture Case/Notes/All Notes|Notes]] | [[Projects/Culture Case/References|References]]
