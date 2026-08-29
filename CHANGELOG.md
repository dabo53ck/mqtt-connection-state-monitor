# Changelog

All notable changes to this project are documented here.

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
