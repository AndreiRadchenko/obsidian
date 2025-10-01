---
title: Install Claude Desktop on Windows
tags: [Claude, Windows, Installation]
created: 2025-09-29T00:00:00Z
updated: 2025-09-29T00:00:00Z
---

## Overview
Claude Desktop is the official desktop application for Anthropic’s Claude AI. This guide explains how to download, install, and configure the app on a Windows machine, as well as the limitations of the free tier.

## Prerequisites
- A Windows 10 or later PC (64‑bit)
- An active internet connection
- An Anthropic account (free or paid) – you’ll need your API key if using the free tier.

## Step‑by‑step Installation
1. **Download the Installer**
   - Visit the official [Claude Desktop download page](https://claude.ai/downloads/desktop) (or search for “Claude Desktop Windows download”).
   - Click the **Windows** link to get the `.exe` installer.

2. **Run the Installer**
   - Double‑click the downloaded `Claude-Desktop-<version>.exe` file.
   - If Windows blocks the file, click **Run anyway**.
   - Follow the wizard:
     * Accept the license agreement.
     * Choose an installation folder (default is fine).
     * Decide whether to create a desktop shortcut.
   - Click **Install** and wait for the process to finish.

3. **Launch Claude Desktop**
   - After installation, open the app via the Start menu or desktop shortcut.
   - The first launch will prompt you to sign in with your Anthropic account or paste an API key.

4. **Configure Preferences** (optional)
   - Open the settings panel by clicking the gear icon.
   - Adjust language, theme (light/dark), and auto‑update preferences.

## Free Tier Restrictions
| Feature | Free Tier | Paid Tier |
|---------|-----------|----------|
| Daily token limit | 200 k tokens per day | Unlimited (or higher tier limits) |
| Max prompt length | 4 k tokens | 32 k tokens |
| Priority access to new models | No | Yes |
| API key usage | Requires free Anthropic account | Included in subscription |

> **Tip**: If you hit the token limit, consider upgrading to a paid plan or waiting until the next day. The free tier is great for casual use and testing.

## Troubleshooting
- **App won’t start**: Make sure you have the latest Windows updates installed. Try running the installer again as Administrator.
- **Login errors**: Verify that your API key is correct. You can generate a new key in the Anthropic dashboard.
- **Performance issues**: Close other heavy applications; Claude Desktop is lightweight but can be affected by overall system load.

## Resources
- [Claude Desktop GitHub repository](https://github.com/anthropic/claude-desktop) – for source and issue tracking.
- [Anthropic Help Center](https://help.anthropic.com) – for detailed FAQ and support.

---
