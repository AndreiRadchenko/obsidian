---
date_created: 2026-07-03
date_modified: 2026-07-03
document_type: note
project: Home Assistant
tags: home-assistant note
---
[[Projects/Home Assistant/Home|Home]] | [[Projects/Home Assistant/Meetings/All Meetings|Meetings]] | [[Projects/Home Assistant/Notes/All Notes|Notes]] | [[Projects/Home Assistant/References|References]]
# Reolink Doorbell in Scrypted
**Overview**
Description:: 
Get Reolink D310W dorbell running in aple home through home assistant
## Note

The **[ODROID-M1S](https://www.google.com/search?ibp=oshop&prds=pvt:hg,pvo:29,mid:576462563330319977,imageDocid:2986699268488415004,gpcid:10369503464993826690,headlineOfferDocid:17616326852284908508,catalogid:2001291659532868240,productDocid:14200411560375144569,rds:PC_3304843417990777905%7CPROD_PC_3304843417990777905&q=product&sa=X&ved=2ahUKEwj6mJiwk7KVAxVOFBAIHX11DxYQxa4PegkIAggACJ8CEAI) running HAOS** is fully capable of running the official Scrypted Add-on natively. Because the ODROID-M1S is an `arm64` architecture board, the official repository automatically provides the correct container image. [[1](https://www.home-assistant.io/installation/odroid/), [2](https://github.com/koush/scrypted/wiki/Installation:-Home-Assistant-OS)]

Since you are running HAOS, you can proceed directly with the installation via the standard Add-on Store. [[1](https://www.youtube.com/watch?v=OqLvuhk8HgU&t=29), [2](https://github.com/koush/scrypted/wiki/Installation:-Home-Assistant-OS)]

How to Install on Your ODROID-M1S

1. In Home Assistant, navigate to **Settings** > **Add-ons** > **Add-on Store**.
2. Click the three vertical dots in the top-right corner and select **Repositories**.
3. Paste the official link: `https://github.com/koush/scrypted` and click **Add**.
4. Close the repository box and refresh or search for **Scrypted**.
5. Click **Install** (this can take a few minutes on the ODROID as it downloads the image).
6. Turn on **Watchdog** and **Show in sidebar**, then click **Start**. [[1](https://community.home-assistant.io/t/add-on-scrypted-homekit-secure-video/398487?page=5), [2](https://docs.scrypted.app/home-assistant.html), [3](https://support.atmotube.com/en/articles/10450153-integrating-with-home-assistant-via-mqtt-and-setting-up-automations), [4](https://lazyadmin.nl/smart-home/enable-ssh-home-assistant/), [5](https://github.com/koush/scrypted/wiki/Installation:-Home-Assistant-OS)]

⚠️ Crucial Resource Warning for the ODROID-M1S

The ODROID-M1S features a quad-core processor and either 4GB or 8GB of RAM. It can comfortably handle a single Reolink D340W stream parsed over to Apple Homekit, but you **must configure it efficiently** to avoid spiking your CPU: [[1](https://www.home-assistant.io/blog/2023/04/18/home-assistant-os-release-10/), [2](https://community.home-assistant.io/t/add-on-scrypted-homekit-secure-video/398487), [3](https://github.com/koush/scrypted/discussions/2000), [4](https://ameridroid.com/blogs/ameriblogs/best-home-assistant-devices)]

- **Avoid Transcoding:** Do not use the "FFmpeg" or transcoding plugins inside Scrypted to convert video. The D340W natively records in H.264/H.265. Apple Home prefers H.264. Ensure you pull the H.264 stream profile directly from the doorbell so the ODROID only has to pass the packets through without rendering them (Direct Copy/No-Op). [, [2](https://community.home-assistant.io/t/add-on-scrypted-homekit-secure-video/398487), [3](https://github.com/koush/scrypted/discussions/2000)]
- **Do Not Activate Scrypted NVR:** Scrypted offers a paid NVR object-detection recording service. Do **not** use the NVR features or video analysis plugins on this hardware, as continuous image analysis will overload the ODROID's processor. Use Scrypted _strictly_ as a bridge to HomeKit. [[1](https://www.reddit.com/r/Scrypted/comments/1qiq6u7/performance_difference_between_scrypted/), [2](https://docs.scrypted.app/home-assistant.html), [3](https://github.com/koush/scrypted/discussions/2000)]

Configuring Your Doorbell Inside Scrypted

Once inside your new Scrypted panel:

1. Install the **Reolink Plugin**, the **ONVIF Plugin**, and the **HomeKit Plugin** from the Scrypted management console.
2. Add your D340W camera via the ONVIF plugin using its local IP address and the custom password you restored earlier.
3. Go to the camera's **HomeKit** settings panel, switch it to **Standalone Accessory Mode**, click **Reset Pairing**, and reload the plugin.
4. A native **Apple Home QR Code** will generate directly on your Scrypted screen. Scan it with your iPhone's Apple Home app to completely link your doorbell with native two-way audio! [[1](https://www.youtube.com/watch?v=KTc9KiDqrFg&t=1)]

Let me know if you run into any **error logs during the installation** process or if you need help finding the right stream profiles once Scrypted boots up!

## References


---
[[Projects/Home Assistant/Home|Home]] | [[Projects/Home Assistant/Meetings/All Meetings|Meetings]] | [[Projects/Home Assistant/Notes/All Notes|Notes]] | [[Projects/Home Assistant/References|References]]
