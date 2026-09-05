# MQTT Connection State Monitor for Home Assistant

<p align="center">
  <img src="https://img.shields.io/badge/version-0.2.2-blue"/>
</p>

**Home Assistant automation to monitor MQTT Connection State binary sensors and notify when devices remain offline longer than the configured duration.**

---

## Stable Release

v0.2.2 is a stable release. Please report bugs and suggestions via [Issues](https://github.com/dabo53ck/mqtt-connection-state-monitor/issues) or [Pull Requests](https://github.com/dabo53ck/mqtt-connection-state-monitor/pulls).

---

## Requirements

- Home Assistant **2026.3.0** or newer (the offline-duration inputs use the hours/minutes duration picker)
- [MQTT Connection State integration](https://github.com/studioIngrid/mqtt_connection_state) (installed separately via HACS)
- One Input Text helper with maximum length of **255** for notification tracking

> **⚠️ Breaking change in v0.2.1** — **Offline Duration** is now an hours/minutes
> picker instead of a raw minute count, and the minimum supported Home Assistant
> version is now **2026.3.0**. After updating the blueprint, open your automation
> and re-enter the offline duration (the old default of `180` minutes is `3 h 0 min`).
> Existing automations keep running but show the threshold as empty until re-saved.
> See the [Changelog](CHANGELOG.md) for full details.

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
   - Choose **notification devices** (multiple Companion App devices supported)
   - Set **Offline Duration** (default: 3 h 0 min)
   - Optionally adjust **Check Interval** – how often the periodic scan runs (5 / 10 / 15 / 30 min, default 15)
   - Optionally set **Battery Device Offline Duration** – a separate, longer threshold auto-applied to battery-powered Zigbee devices (0 = disabled)
   - Optionally configure iOS/Android notification settings
   - Optionally configure exclusion list and custom actions
4. Click **"Import"** and **"Create Automation"**

[![Open your Home Assistant instance and import a blueprint.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/dabo53ck/mqtt-connection-state-monitor/main/mqtt-connection-state-monitor.yaml)

---

## Features

- **Delayed offline detection** – Notifications only trigger after device stays offline longer than configured duration
- **Battery-device offline threshold** – Optional separate, longer threshold auto-applied to battery-powered Zigbee devices (auto-detected via a battery entity on the same device); disabled by default
- **Duplicate notification protection** – Each device notified only once per offline event
- **Multiple device notifications** – Send to multiple phones/tablets simultaneously
- **Platform-aware notifications** – Separate iOS and Android options
- **Notification timestamp** – Optional time-of-day (with seconds) on each alert; iOS subtitle / Android subject; 12H or 24H (24H default)
- **Customizable notification text** – Override the offline/online notification title and message with your own templates (defaults match the built-in wording)
- **iOS interruption level control** – Configure offline/online alert urgency (active/critical/time-sensitive/passive)
- **iOS notification grouping** – Consolidate alerts on iOS using Group ID `mqtt_connection_state`
- **Android high priority delivery** – Ensure offline alerts bypass normal delivery delays
- **Android sticky notifications** – Keep offline alerts visible until manually dismissed
- **Android notification channels** – Route alerts to a dedicated channel with custom sound/importance
- **Optional offline/online notifications** – Enable/disable Companion App push notifications separately
- **Optional offline/online actions** – Trigger scripts, webhooks, Telegram, Discord, etc.
- **Configurable check interval** – Run the periodic offline scan every 5, 10, 15, or 30 minutes (default 15)
- **Automatic recovery handling** – Tracks when devices come back online and removes from tracking list
- **Self-healing tracking list** – The periodic scan clears devices that recovered while their online event was missed (e.g. an HA restart), so future alerts keep working; recovery notification/actions optional
- **Variables available** – Pass device info to custom actions (see below)
- **Device exclusion list** – Exclude specific entities from monitoring
- **Concurrent event support** – Parallel `mqtt_connection_state_changed` events processed correctly (`mode: queued`)

---

## Offline Duration & Battery Devices

**Offline Duration** sets how long any monitored device must stay offline before
it is reported. It is an hours/minutes picker; detection runs on the periodic
polling cycle set by **Check Interval** (5 / 10 / 15 / 30 minutes, default 15),
so a value shorter than one interval is treated as one interval and actual
detection may lag the threshold by up to one interval.

On every periodic run the blueprint also **self-heals the tracking helper**: a
device still listed as offline but currently reporting online (usually because
its recovery event was missed during a Home Assistant restart) is removed from
the helper, so future offline alerts for it work again. **Notify on Self-Healed
Recovery** (on by default) decides whether that also sends the normal online
notification and runs *Online Actions*, or heals the list silently.

**Battery Device Offline Duration** is an optional, usually longer threshold for
battery-powered Zigbee devices, which check in far less often than mains-powered
ones. When set to a non-zero value, the blueprint scans for devices that expose a
battery entity (a battery-percentage `sensor` or a Low/OK `binary_sensor`) and
records their device IDs and device-registry identifiers. A **monitored**
connectivity sensor is then treated as battery-powered when its own device ID or
one of its registry identifiers is in that set. Matched devices use this
threshold instead of the normal one. Leave it at `0 h 0 min` to disable — then
every device uses **Offline Duration**.

Detection is automatic; there is no manual device list. The `mqtt_connection_state`
integration registers its connectivity sensor on a separate device entry from the
Zigbee2MQTT device that owns the battery entity, so the match is made on the
shared Zigbee identifier (IEEE address) as well as the `device_id`. Device
**names** are never used for matching — they are not unique in Home Assistant.
Only entities that are actually monitored by this automation (a
`*_connection_state` connectivity sensor, not excluded) are ever checked, so
unrelated battery devices — phones, a UPS, vacuums, an inverter — are never
affected.

---

## Companion App Notifications

Notifications are sent via the **Home Assistant Companion App** (iOS and Android).

**iOS Options:**
- Interruption level (offline/online configurable)
- Notification grouping via Group ID `mqtt_connection_state`
- Timestamp shown as the notification subtitle

**Android Options:**
- High priority delivery (bypasses normal delays)
- Sticky notifications (manual dismissal required)
- Notification channels (custom sound, vibration, importance)
- Timestamp shown as the notification subject line

**Timestamp:**
The current time (with seconds) is added to every notification — as the `subtitle`
on iOS and the `subject` field on Android (the title and message are unchanged).
Enable/disable it with **Include Timestamp in Notifications** (on by default) and
pick **24-hour** (default, e.g. `14:23:15`) or **12-hour** (e.g. `02:23:15 PM`)
via **Timestamp Format**. The value is the time the notification is sent.

**Notification Messages:**

Default wording:
- Offline — title `🔴 {{ friendly_name }} offline`, message `{{ friendly_name }} has been offline for {duration} minutes.`
- Online — title `🟢 {{ friendly_name }} online`, message `{{ friendly_name }} is back online.`

Override any of these with **Offline/Online Notification Title** and **Offline/Online Notification Message** in the *Notifications* section. The fields accept templates and can use `friendly_name`, `device_name`, `entity_id`, `notification_time`, and (offline only) `offline_seconds`. Leave a field at its default to keep the built-in wording. The timestamp `subtitle`/`subject` is added independently and is unaffected.

For other targets (Alexa, Telegram, SMTP, etc.), use the **Custom Actions** section.

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
| `threshold_seconds` | Effective threshold for **this** device (battery threshold if it was matched, otherwise the normal one) | `10800` |
| `battery_powered` | `true` if the device was detected as battery-powered | `false` |
| `offline_time` | Formatted timestamp (YYYY-MM-DD HH:MM:SS) | `2026-07-20 14:23:15` |
| `offline_timestamp` | ISO timestamp with timezone | `2026-07-20T14:23:15+02:00` |
| `notification_time` | Send time, format per **Timestamp Format** setting | `14:23:15` (or `02:23:15 PM`) |

**Online Actions:**
| Variable | Description | Example |
|----------|-------------|---------|
| `device_name` | Entity ID without domain/prefix | `3_gang_schalter` |
| `friendly_name` | Human-readable name | `3 Gang Schalter Connection` |
| `entity_id` | Full entity ID | `binary_sensor.3_gang_schalter_connection_state` |
| `online_time` | Formatted timestamp (YYYY-MM-DD HH:MM:SS) | `2026-07-20 17:45:30` |
| `online_timestamp` | ISO timestamp with timezone | `2026-07-20T17:45:30+02:00` |
| `notification_time` | Send time, format per **Timestamp Format** setting | `17:45:30` (or `05:45:30 PM`) |

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

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for the full version history.

---

## License

MIT License — see [LICENSE](LICENSE) for details.
