# MQTT Connection State Monitor for Home Assistant

<p align="center">
  <img src="https://img.shields.io/badge/version-0.1.0--beta-orange"/>
</p>

**Home Assistant automation to monitor MQTT Connection State binary sensors and notify when devices remain offline longer than the configured duration.**

---

## ⚠️ Public Beta

This is a public beta release. Features may change, inputs may be added or modified. Please report bugs and suggestions via [Issues](https://github.com/dabo53ck/mqtt-connection-state-monitor/issues) or [Pull Requests](https://github.com/dabo53ck/mqtt-connection-state-monitor/pulls).

---

## Requirements

- Home Assistant 2024.6.0 or newer
- [MQTT Connection State integration](https://github.com/studioIngrid/mqtt_connection_state)
- One Input Text helper with maximum length of 255 for notification tracking

---

## Setup Instructions

### Create Input Text Helper (Required)

The blueprint requires one Input Text helper with a maximum length of **255 characters** to track notified devices.

**Via Home Assistant UI:**

1. Go to **Settings** → **Devices & Services** → **Helpers** (bottom right)
2. Click **"Create Helper"**
3. Select **"Text"**
4. Fill in:
   - **Name:** `MQTT Notification Tracker` (or your choice)
   - **Max length:** `255` ← **Important!**
5. Click **"Create"**
6. Copy the entity ID (e.g., `input_text.mqtt_notification_tracker`)

**Note:** If the max length is not set to 255, the blueprint may fail when tracking more than a few devices.

---

## Installation

1. Create the Input Text helper as described above
2. Click the button below to import the blueprint
3. Configure your desired options:
   - Select the Input Text helper you created
   - Choose notification device (for Companion App notifications)
   - Set offline duration threshold (default: 180 minutes)
   - Optionally configure exclusion list and actions
4. Click **"Import"** and **"Create Automation"**

[![Open your Home Assistant instance and import a blueprint.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/dabo53ck/mqtt-connection-state-monitor/main/mqtt-connection-state-monitor.yaml)

---

## Features

- **Delayed offline detection** – Notifications only trigger after device stays offline longer than configured duration
- **Duplicate notification protection** – Each device notified only once per offline event
- **Optional offline/online notifications** – Enable/disable Companion App push notifications separately
- **Optional offline/online actions** – Trigger scripts, webhooks, Telegram, Discord, etc.
- **Automatic recovery handling** – Tracks when devices come back online and removes from tracking list
- **Variables available** – Pass device info to custom actions (see below)
- **Device exclusion list** – Exclude specific devices from monitoring
- **iOS notification grouping** – Consolidate alerts on iOS devices using Group ID
- **Custom notification icons** – Different icons for offline (🔴) vs online (🟢) status

---

## Companion App Notifications

Notifications are grouped on iOS using Group ID `mqtt_connection_state`.

**Notification Messages:**
- Offline: `🔴 {friendly_name} offline - Device has been offline for {duration} minutes.`
- Online: `🟢 {friendly_name} online - Device is back online.`

---

## Custom Actions

Use **Offline Actions** and **Online Actions** for integrations beyond Companion App:

### Supported Platforms
- Telegram
- Discord
- Email
- Persistent Notifications
- Scripts
- Webhooks
- Other automations

### Available Variables

**Offline Actions:**
| Variable | Description | Example |
|----------|-------------|---------|
| `device_name` | Entity ID without domain/prefix | `3_gang_schalter` |
| `friendly_name` | Human-readable name | `3 Gang Schalter Connection` |
| `entity_id` | Full entity ID | `binary_sensor.3_gang_schalter_connection_state` |
| `offline_seconds` | Seconds since state change | `10800` |
| `threshold_seconds` | Configured duration in seconds | `10800` |
| `offline_time` | Formatted timestamp (YYYY-MM-DD HH:MM:SS) | `2026-07-20 14:23:15` |
| `offline_timestamp` | ISO timestamp with timezone | `2026-07-20T14:23:15+02:00` |

**Online Actions:**
| Variable | Description | Example |
|----------|-------------|---------|
| `device_name` | Entity ID without domain/prefix | `3_gang_schalter` |
| `friendly_name` | Human-readable name | `3 Gang Schalter Connection` |
| `entity_id` | Full entity ID | `binary_sensor.3_gang_schalter_connection_state` |
| `online_time` | Formatted timestamp (YYYY-MM-DD HH:MM:SS) | `2026-07-20 17:45:30` |
| `online_timestamp` | ISO timestamp with timezone | `2026-07-20T17:45:30+02:00` |

### Example: Telegram Message

**Offline Action:**

    - service: notify.telegram_bot
      data:
        message: "⚠️ Device {{ friendly_name }} is offline ({{ offline_seconds }}s)"

**Online Action:**

    - service: notify.telegram_bot
      data:
        message: "✅ Device {{ friendly_name }} is back online!"

---

## Known Limitations

The blueprint uses an Input Text helper to track devices that have already been reported as offline. Because Input Text helpers have a maximum length of **255 characters**, approximately **10-15 devices** can be tracked simultaneously, depending on the length of device names.

For environments with a large number of devices going offline simultaneously, consider splitting into multiple blueprints with separate helpers.

---

## Future Enhancements

The following features are being considered for future releases. Contributions and feedback are welcome!

- **Customizable notification messages** – Define custom templates for Companion App notifications
- **Minimum offline time before recovery** – Ignore brief connection drops to prevent false alerts
- **Multiple device notifications** – Send to multiple targets simultaneously
- **Persistent event logging** – Track history beyond temporary helper storage

Interested in contributing? Check the [Roadmap](#future-enhancements), open an [Issue](https://github.com/dabo53ck/mqtt-connection-state-monitor/issues), or submit a [Pull Request](https://github.com/dabo53ck/mqtt-connection-state-monitor/pulls).

---

## Changelog

### v0.1.0-beta – Initial public beta release

- Delayed offline detection
- Duplicate notification protection
- iOS notification grouping
- Device exclusion list
- Custom notification icons
- `max_exceeded: silent` to prevent log warnings
- Template error fixes for `last_changed` handling

---

## License

MIT License - See LICENSE file for details
