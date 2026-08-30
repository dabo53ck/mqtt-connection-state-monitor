# Changelog

All notable changes to this project are documented here.

## [v0.2.1] - 2026-08-30

### ⚠️ Breaking Changes
- **Offline Duration** input changed from a raw minutes `number` field to an
  hours/minutes `duration` picker, and its key was renamed
  `offline_duration_minutes` → `offline_duration` (#8). Existing automations keep
  running but show the threshold as empty until reopened and re-saved — re-enter
  the value (old default `180` → `3 h 0 min`).
- **Minimum Home Assistant version is now 2026.3.0** (was 2024.8.0). The duration
  inputs use `enable_second: false`, available from 2026.3. Older instances
  cannot import the blueprint.

### Added
- **Battery Device Offline Duration** — optional separate, longer offline
  threshold for battery-powered Zigbee devices (#7). Devices that expose a
  battery entity (percentage `sensor` or Low/OK `binary_sensor`) are
  auto-detected; a monitored connectivity sensor is matched to them by
  `device_id` and device-registry identifier (Zigbee IEEE address) — never by
  device name. Only entities actually monitored by this automation are checked.
  Defaults to `0` (disabled); all devices then use **Offline Duration** as
  before.
- New Custom Actions context variables for offline actions: `battery_powered`
  (bool), plus `threshold_seconds` now reports the effective per-device value.
- Customizable notification text — new **Offline/Online Notification Title** and
  **Offline/Online Notification Message** inputs in the *Notifications* section.
  Each defaults to the current built-in wording and accepts templates
  (`friendly_name`, `device_name`, `entity_id`, `notification_time`, and
  `offline_seconds` for offline), so notifications can be localised or restyled
  without forking the blueprint (#6)

### Changed
- Offline thresholds below 15 minutes are now treated as 15 minutes (previously
  enforced by the `number` selector's `min: 15`).
- `notified_helper` input description now links to the input_text config flow
  and the README setup instructions.
- README: link the `LICENSE` file from the License section instead of only
  naming it (#5)

## [v0.2.0] - 2026-08-29

First stable release.

### Added
- Optional timestamp (with seconds) on Companion App notifications
  - iOS: shown as notification `subtitle`
  - Android: shown as notification `subject` (title and message unchanged)
  - Enable/disable toggle (**Include Timestamp in Notifications**, enabled by default)
  - 12-hour / 24-hour format selector (**Timestamp Format**, 24-hour default)
- `notification_time` variable exposed to Custom Actions (offline & online)

### Changed
- Promoted from public beta to stable

## [v0.1.2-beta] - 2026-08-15

### Fixed
- Missing underscore in offline notify call (#3)
- Inconsistent `device_name` extraction breaking deduplication (#3)
- Broken `friendly_name` fallback: `replace('', ' ')` → `replace('_', ' ')` (#3)

### Changed
- Pre-filter to offline-only entities before loop processing (#4)
- Batch helper read/write: single `input_text.set_value` per run (#4)
- Report actual offline duration instead of static threshold (#4)
- `state_attr()` instead of `attributes.get()` for robustness (#4)
- `max_exceeded: warning` + explicit `max: 10` (#4)
- `offline_duration_minutes` minimum raised to 15 (#4)

### Note
⚠️ If upgrading from v0.1.1-beta: Some users may need to clear their `input_text` helper value once after updating if devices don't recover correctly. This affects setups where devices were stored with the buggy trailing underscore format.

## [v0.1.1-beta] - 2026-08-07

⚠️ **Breaking Changes** — Read carefully before updating.

### Breaking Changes
- **Notification selector:** Changed from `entity` (notify domain) to `device` (mobile_app filter)
  - Existing notification target selections need reconfiguration
  - Only Companion App devices supported directly; use Custom Actions for Alexa/Telegram/etc.
- **Minimum version:** Now requires Home Assistant 2024.8.0 or newer

### Added
- Multiple device notification support (#2)
- Platform-aware notification options (iOS / Android)
- iOS interruption level control (offline/online)
- iOS notification grouping
- Android high priority delivery
- Android sticky notifications
- Android notification channels

### Fixed
- Concurrent event race condition (#1)

### Tested
- ✅ iOS & Android notifications
- ✅ Multi-device parallel events

## v0.1.0-beta – Initial public beta release

- Delayed offline detection
- Duplicate notification protection
- iOS notification grouping
- Device exclusion list
- Custom notification icons
- `max_exceeded: silent` to prevent log warnings
- Template error fixes for `last_changed` handling
