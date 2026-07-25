# MQTT Connection State Monitor for Home Assistant

<p align="center">
  <img src="https://img.shields.io/badge/version-0.1.0--beta-orange"/>
</p>

**Home Assistant automation to monitor MQTT Connection State binary sensors and notify when devices remain offline longer than the configured duration.**

## ⚠️ Public Beta

This is a public beta release. Features may change, inputs may be added or modified. Please report bugs and suggestions in the [GitHub repository](https://github.com/dabo53ck/mqtt-connection-state-monitor).

## Requirements

- Home Assistant 2024.6.0 or newer
- [MQTT Connection State integration](https://github.com/studioIngrid/mqtt_connection_state)
- One Input Text helper with maximum length of 255 for notification tracking

## Features

- Delayed offline detection
- Duplicate notification protection
- Optional offline/online notifications
- Optional offline/online actions
- Automatic recovery handling
- Variables available for custom actions
- Device exclusion list
- iOS notification grouping
- Custom notification icons

## Companion App Notifications

Notifications are grouped on iOS using Group ID `mqtt_connection_state`.

## Custom Actions

Use Offline Actions and Online Actions for:
- Telegram
- Discord
- Email
- Persistent Notifications
- Scripts
- Webhooks
- Other automations

Variable details and examples are documented in the corresponding action fields.

## Installation

1. Click the button below to import the blueprint
2. Configure your desired options
3. Create the automation

[![Open your Home Assistant instance and import a blueprint.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/dabo53ck/mqtt-connection-state-monitor/main/mqtt-connection-state-monitor.yaml)

## Known Limitations

The blueprint uses an Input Text helper to track devices that have already been reported as offline. Because Input Text helpers have a maximum length, approximately 10-15 devices can be tracked simultaneously, depending on the length of device names.

## Changelog

### v0.1.0-beta – Initial public beta release

- Delayed offline detection
- Duplicate notification protection
- iOS notification grouping
- Device exclusion list
- Custom notification icons
- `max_exceeded: silent` to prevent log warnings
- Template error fixes for `last_changed` handling

## License

MIT License - See LICENSE file for details
