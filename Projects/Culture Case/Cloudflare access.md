---
date_created: 2026-02-07
date_modified: 2026-02-07
document_type: note
project: Culture Case
tags:
  - Cloudflare
  - Culture_case
  - domain/software-dev
  - domain/smart-home
---
[[Projects/Culture Case/Home|Home]] | [[Projects/Culture Case/Meetings/All Meetings|Meetings]] | [[Projects/Culture Case/Notes/All Notes|Notes]] | [[Projects/Culture Case/References|References]]
# Cloudflare access samba server
**Overview**
Description:: How to get access to entire local network through cloudflare

## Note

## Access Entire Local Network via CIDR Routing
[[Cloudflare access]]
If you want to access your whole home network by IP address:

- Navigate to your Tunnel Settings:
    
    - Go to: [[https://one.dash.cloudflare.com/6a817637126870fc68bc7a443ca469c4/networks/connectors/cloudflare-tunnels/cfd_tunnel/{tunnelId}/edit](https://one.dash.cloudflare.com/6a817637126870fc68bc7a443ca469c4/networks/connectors/cloudflare-tunnels/cfd_tunnel/%7BtunnelId%7D/edit)](https://one.dash.cloudflare.com/6a817637126870fc68bc7a443ca469c4/networks/routes)
    - Replace `{tunnelId}` with your tunnel ID
- Add CIDR Routes:
    
    - Go to the Routes tab
    - Add a route for your entire network (e.g., `192.168.1.0/24` or `10.0.0.0/8`)
    - This allows WARP clients to access any device on your network
- Configure WARP Client:
    
    - Ensure your WARP client has Split Tunnels enabled
    - Add your home network CIDR range to the Include list
    - This ensures traffic to your local network goes through Cloudflare

## What is CIDR?
CIDR stands for Classless Inter-Domain Routing.

CIDR is a method for allocating IP addresses and routing in the internet. Instead of using the old Class A, B, C system, CIDR uses a slash notation to define network ranges.

### The Format

```
IP address /prefix_length
```

Example: `192.168.1.0/24`

- 192.168.1.0 - The IP address of your network
- /24 - The prefix length (number of bits that define the network)

### How It Works

The prefix length tells you how many bits represent the network portion of the IP address:

- /24 = 24 bits → 8 bits for host (254 usable IPs)
- /16 = 16 bits → 16 bits for host (65,534 usable IPs)
- /32 = 32 bits** → 1 bit for host (only 1 IP - a single host)
- /8 = 8 bits → 24 bits for host (16,777,214 usable IPs)**

### Visual Example: /24

Network: 192.168.1.0
          └─ 24 bits are the network part

Available IPs:
192.168.1.0   (network address - reserved)
192.168.1.1   (your router)
192.168.1.2   (your Samba server)
...
192.168.1.254  (last usable)
192.168.1.255  (broadcast - reserved)



*To change split tunnel mode in the iOS WARP client, you need to configure it through the Cloudflare One dashboard. The iOS app itself doesn't have a split tunnel toggle - it's managed centrally.*

## Steps to Configure Split Tunnels:

### 1. Navigate to Device Profiles

Go to: [https://one.dash.cloudflare.com/6a817637126870fc68bc7a443ca469c4/team-resources/devices/profiles](https://one.dash.cloudflare.com/6a817637126870fc68bc7a443ca469c4/team-resources/devices/profiles)

### 2. Select or Create a Device Profile

- Click on Device profiles
- Select the profile your iOS device is using, or create a new one

![[Screenshot 2026-02-28 at 18.24.05.png]]

### 3. Configure Split Tunnels

- Scroll down to the Split Tunnels section
- Click Manage to see current settings
- Choose your mode:
    - Exclude IPs and domains (Default) - All traffic goes through WARP except what you specify
    - Include IPs and domains - Only traffic to specified IPs/domains goes through WARP

### 4. Add Your Network Range

For your home network access:

In Exclude mode (recommended):

- Select Exclude IPs and domains
- Click Add route
- Enter:
    - Address: `192.168.1.0/24` (or your actual network range)
    - Description: "Home network"
- Save

In Include mode:

- Select Include IPs and domains
- Click Add route
- Enter:
    - Address: `192.168.1.0/24`
    - Description: "Home network"
- Save

### 5. Sync to iOS Device

- Install Cloudflare WARP client [https://developers.cloudflare.com/cloudflare-one/team-and-resources/devices/warp/download-warp/#macos](https://developers.cloudflare.com/cloudflare-one/team-and-resources/devices/warp/download-warp/#macos)
- In Preferences -> Account login with the name you obtain during setup device profile

![[Image 28.02.2026 at 17.28.png]]

- in Finder in find field type: `smb://192.168.1.2` - IP of your samba server, select URL. 
- Press `>>` select action -> services -> Open URL
- in the opened windows enter samba credentials
## References


---
[[Projects/Culture Case/Home|Home]] | [[Projects/Culture Case/Meetings/All Meetings|Meetings]] | [[Projects/Culture Case/Notes/All Notes|Notes]] | [[Projects/Culture Case/References|References]]
