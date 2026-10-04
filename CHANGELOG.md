# Changelog — lidl-extender

## 2026-10-04 — Working tree

### Speed-based check interval

The main check interval is now adjusted according to the configured internet speed.

* Added `INTERNET_SPEED_MBPS` to `.env` and `.env.example`

  * Placeholder: `500`
  * Empty/unset values fall back to `500 Mbps`
* Added `SPEED_BASE_MBPS`, `SPEED_FACTOR_MIN`, and `SPEED_FACTOR_MAX`
* Added `speedFactor()` and `scaledInterval()`
* `getSmartInterval()` now scales its data-volume intervals by connection speed
* Scaling factor: `500 / speed`, clamped to `0.5–3.0`

  * Slower connection → longer wait
  * Faster connection → shorter wait

Base intervals at 500 Mbps:

| Data volume | Check interval |
| ----------- | -------------: |
| ≥10 GB      |      15–30 min |
| ≥5 GB       |      10–15 min |
| ≥3 GB       |      5–7.5 min |
| ≥2 GB       |      2.5–4 min |
| ≥1.2 GB     |    1.5–2.5 min |
| ≥1 GB       |      1–1.5 min |
| <1 GB       |          1 min |

The interval controls how long the script waits before the next site reload/check. It determines how quickly the 80% refill threshold is detected; it is not a page-load timeout.

### Keep-alive jitter

Keep-alive jitter was changed from symmetric to additive jitter.

* Before: ±10%
* Now: +0–50%
* The calculated interval can only be extended, never shortened
* Example: the 30-second minimum now produces 30–45 seconds

The underlying adaptive keep-alive logic is unchanged:

* 2 min per 25 GB
* Maximum base interval: 30 min
* Linear reduction toward 80% usage
* Minimum: 30 s
* Recursive `setTimeout`

### Bugfixes (post-release)

* **Version sync:** `package.json` still carried the stale version `1.1.1` while `script.js` and this changelog declare `1.2.4`. Synced `package.json` to `1.2.4` so the auto-update version comparison is consistent.
* **Log level env var:** `.env.example` used `INFOLEVEL`, but the script only read `INFO_LEVEL` — the setting from the example was silently ignored. `.env.example` now uses `INFO_LEVEL`; the script also accepts `INFOLEVEL` for backward compatibility with existing `.env` files.

### Unchanged

* Page-load timeouts and delays remain at their original values.
* Version remains `1.2.4`.

---

## v1.2.4 — 80% refill refactor

Original repository state: `ff1d451` (v1.0.0)

### Refill trigger

Refill activation changed from a fixed remaining-volume threshold to 80% total usage.

* Added `REFILL_USAGE_THRESHOLD = 0.8`
* Refill is attempted when usage reaches 80%, refill is available, and the retry cooldown has expired.
* Usage is calculated from the live DOM.
* Failed refill attempts now record `refillFailedAt`, making the existing 10-minute retry cooldown functional.
* Updated refill log message to reflect the 80% threshold.

### Keep-alive

The previous fixed 2-minute reload interval was replaced with an adaptive loop.

* Base interval: 2 min per 25 GB
* Maximum: 30 min
* Linear reduction as usage approaches 80%
* Minimum: 30 s
* Recursive `setTimeout`
* Randomized interval using +0–50% jitter

### 80% progress reporting

Added `buildRefillProgressLine(usage)` to show progress toward the refill threshold.

Example:

```text id="x8r4pn"
⏳ WAITING FOR 80%
used 18.2/26.0 GB (70%)
[██████████████████░░]
88% of 80% used · to 80%: 2.6 GB
```

At the threshold:

```text id="m5q2vd"
🔄 RECHARGING NOW
used 20.8/26.0 GB (80%)
[████████████████████]
100% of 80% used
```

The progress block is included in Telegram, Discord, success messages, and output logs.

### Robustness and bug fixes

* Extracted `readConsumptionUsage(page)` into a reusable helper.
* Added support for both legacy and current Lidl DOM identifiers, with positional label matching as fallback.
* Refill data is re-read through the same helper after a refill.
* Fixed post-refill reading of `refill.available`.
* Prevented missing refill data from producing `NaN` totals.
* Broadened the data-usage wait selector.
* Fixed `.env` path handling with `fileURLToPath`.
* Changed login navigation from `networkidle` to `domcontentloaded`.
* `HEADLESS=false` is now respected.
* Existing NaN handling, circuit breaker, and memory checks remain unchanged.

### Watchdog / heartbeat

Fixed false "Deadlock" detections during intentional long waits.

* Added a 30-second heartbeat timer.
* Added `isRestarting` to prevent overlapping browser restarts.
* `restartBrowser()` updates the heartbeat immediately.

This prevents unnecessary browser restarts and repeated watchdog notifications while the script is legitimately waiting.

### Messages

* Added `📡 Lidl-Extender` header to status and success messages.
* Added the 80% progress block to status and success messages.
* Added `formatDuration()` for human-readable wait times.
* Removed raw seconds/milliseconds from user-facing messages.
* Example: `Nächste Prüfung in 83 Min.` or `1 Std. 25 Min.`
* Watchdog messages use the same duration formatting.
* Other messages remain unchanged.

### package-lock.json

| Field   | Original | v1.2.4                    |
| ------- | -------- | ------------------------- |
| Version | `1.0.0`  | `1.2.4`                   |
| License | `ISC`    | `GNU Public License v3.0` |

---

## Working-copy files

Never tracked by Git:

* `start.bat` — launcher
* `_check_env.ps1` — environment check helper

### Cleanup

Removed temporary files:

* `script.js.bak`
* `__git_diff.txt`
* `conv.txt`
* `.editcheck.cjs`
* `run-err.log`
* `run-out.log`

Ignored local/runtime files:

* `.env`
* `cookies.json`
* `session_meta.json`
* `lidl-extender-data/`
* logs
