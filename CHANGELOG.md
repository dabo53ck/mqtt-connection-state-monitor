# Changelog

All notable changes to this project are documented here.

## [v0.3.1] - 2026-09-08

### Fixed
- **Mass outage: `infra_recovered` no longer fires on the bridge entity alone**
  (#14). It previously ended the episode on the first `active` check while most
  devices were still offline — sending a bogus *recovered* alert and, because
  remediation is gated on `infra == 0`, permanently disarming the configured
  coordinator-restart action.

### Changed
- **Mass outage recovery is harder to fool.** The *infrastructure recovered*
  step now also requires visible device recovery (≥ 50 % of mains devices back,
  or the offline count down to 80 % of its peak). A coordinator that hangs while
  its MQTT-connection sensor stays `on` no longer ends the episode prematurely.
- **Remediation runs before recovery is declared.** When Mass Outage Actions are
  configured, the action now fires at the first eligible check instead of being
  pre-empted by a recovery check on the same run.
- **Premature recovery is reversible.** If the outage re-expands (back to the
  trigger count and ≥ 80 % of peak) or a bridge entity trips again after a
  declared recovery, the episode reopens — up to twice — re-arming the
  remediation action and reminders. Bounded by *Maximum Outage Duration*.
- Episode marker gains two trailing fields (`peak`, `reopen`); older markers are
  read with both defaulting to `0`, so in-flight episodes upgrade cleanly.

### Added
- CI (`.github/workflows/validate.yml`): yamllint + Home Assistant
  `check_config` against a full-input test automation, on every push and PR.

## [v0.3.0] - 2026-09-07

### Added
- **Mass Outage Detection** — new *Mass Outage Detection* section (disabled by
  default). When many monitored devices drop in the same check — a Zigbee
  coordinator, MQTT bridge or hub failure rather than genuine per-device
  outages — the blueprint sends **one** aggregated alert instead of a per-device
  flood, suppresses all per-device offline/online handling (Companion App and
  Custom Actions) until recovery, and optionally runs a one-time remediation
  action (e.g. pressing the coordinator's restart button) (#11).
  - Trigger on an absolute **Trigger Count**, a **Trigger Fraction (%)** of all
    monitored devices (OR-combined), or any listed **Bridge / Coordinator
    Entity** being `off`/`unavailable` (immediate, no debounce). Count/fraction
    triggers are confirmed over two consecutive checks.
  - **Reminder Interval** repeats the alert while the infrastructure is still
    down; reminders stop once it recovers.
  - **Mass Outage Actions** + **Action Delay** run once, only if the outage is
    still ongoing after the delay.
  - Two-phase recovery: an *infrastructure recovered* alert fires when all bridge
    entities are back `on` or ≥ 90 % of mains-powered devices have returned
    (battery devices are ignored for this decision); then separate *not
    recovered* follow-ups list any mains devices still offline after twice the
    Check Interval and any battery devices still offline after **Battery Recovery
    Grace** (default 1 h).
  - **Maximum Outage Duration** (default 24 h) force-closes the episode; any
    devices still offline then fall back to normal per-device tracking.
  - Episode state is held in a single fixed-size marker at the front of the
    tracking helper, so a coordinator-wide outage no longer fills the 255-char
    helper regardless of device count.
  - New context variables for Mass Outage Actions and the aggregated
    notification templates: `offline_count`, `monitored_count`,
    `offline_fraction`, `affected_devices`, `outage_started`, `outage_duration`,
    `trigger_reason`, `bridge_entity`, `is_reminder`, `recovered_count`,
    `still_offline_count`, `still_offline_devices`, `followup_kind`.
- **Warn When Tracking Helper Is Full** — new toggle in *Monitoring Options*
  (`default: true`). When the tracking helper would exceed the 255-character
  Input Text limit, it is now truncated (instead of the write failing silently)
  and a persistent notification is raised; it clears itself once the helper fits
  again (#12).
- **Warn When Threshold Is Clamped** — new toggle in *Monitoring Options*
  (`default: true`). Raises a persistent notification when **Offline Duration**
  or **Battery Device Offline Duration** is shorter than the **Check Interval**
  and is therefore silently treated as one interval; dismisses itself when the
  values are no longer clamped (#13).

### Changed
- The connectivity scan now runs once per periodic check and feeds the
  due-devices, self-heal and mass-outage logic from a single pass.
- While a mass outage episode is active, the event-triggered "back online" branch
  is a no-op, so a recovery event storm no longer produces a per-device push
  flood.

## [v0.2.2] - 2026-09-05

### Added
- **Check Interval** — new input in *Monitoring Options* to run the periodic
  offline check every 5 / 10 / 15 / 30 minutes (`default: 15`, so existing
  automations keep the current cadence after a re-import). Shorter intervals
  detect outages sooner; longer intervals cut template-evaluation load on large
  or constrained instances (#10).
- **Self-healing of the tracking helper** — the periodic check now reconciles the
  notified-devices helper against live state. Any device still listed as offline
  but currently reporting online (typically because its `online` recovery event
  was missed during a Home Assistant restart) is removed from the helper, so it
  gets "back online" handling now and future offline alerts for it work again.
  Previously such a device was silently skipped for every future offline alert
  until the helper was edited by hand (#9).
- **Notify on Self-Healed Recovery** — new toggle in *Monitoring Options*
  (`default: true`) controlling whether a self-healed recovery also sends the
  normal online notification and runs *Online Actions*, or heals the tracking
  list silently (#9).

### Changed
- The minimum effective offline threshold now follows **Check Interval** (one
  polling cycle) instead of a hard-coded 15 minutes. With the default interval
  the previous 900-second floor is unchanged (#8, #10).
- Blueprint input descriptions and `README.md` no longer hard-code a
  "15-minute polling cycle".

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
