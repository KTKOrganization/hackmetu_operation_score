# QA Checklist — OperationScore Dashboard

**Date:** 2026-02-21  
**Reviewer:** antigravity (QA pass)

---

## Bugs Found & Fixed

| # | Bug | Fix | File |
|---|-----|-----|------|
| 1 | `safeFetch` mutated caller's `options` object by adding `signal` | Clone options before assigning signal | app.js:116 |
| 2 | `postRebuild` used raw `fetch` — no timeout protection | Changed to `safeFetch` | app.js:410 |
| 3 | `setDemoMode(false)` didn't reset `_wsReconnectFails` — WS stuck permanently after demo toggle | Added `_wsReconnectFails = 0` | app.js:261 |
| 4 | `_updateWsIndicator` only updated dot class, never synced `wsLabel` text | Added wsLabel text update | app.js:527 |

---

## Static Review

- [x] `escapeHtml` used on all user-facing data (hostname, reasons, actions, messages)
- [x] `csvField` correctly handles commas, quotes, newlines
- [x] `formatRelativeTime` handles null, invalid dates, seconds/min/hours/days in Turkish
- [x] `mapRiskToBadge` maps CRITICAL/HIGH→Kritik, MEDIUM→Dikkat, LOW/EXCELLENT/others→İyi
- [x] `_deriveRiskLevel` maps score thresholds: <50=CRITICAL, <60=HIGH, <75=MEDIUM, <90=LOW, ≥90=EXCELLENT
- [x] `normalizeSnapshot` validates all device fields with safe defaults
- [x] No duplicate event listeners — `_attachIndexEvents` called once per render (innerHTML replaces all)
- [x] No interval leaks — `_cleanupPing`, health/age intervals use `clearInterval` before re-creating
- [x] No duplicate WS connections — guarded by `readyState` check in `connectWebSocket`
- [x] WS max reconnect failures (5) prevents infinite retry loops
- [x] `safeFetch` clones options (fixed)
- [x] `postRebuild` uses `safeFetch` (fixed)
- [x] Scan poll timer properly cleared before starting new poll

## Runtime QA — Backend OFF

- [x] Index page: shows clean "API'ye bağlanılamadı" error with retry button
- [x] Retry button works (reattempts fetch, no crash)
- [x] No JavaScript errors in console (only expected network errors)
- [x] Demo Mode ON: renders full dashboard with 3 demo devices
- [x] Demo Mode banner: "Demo Modu açık: Veriler örnek veridir."
- [x] KPI cards show correct values (3 / 1 / 65 / az önce)
- [x] Connection card: Mod=Demo, API=Kapalı (Demo), WS=Kapalı (Demo)
- [x] Search input filters devices case-insensitively
- [x] Sort toggle on Skor column works (asc/desc)
- [x] CSV İndir button triggers download
- [x] Scan button shows info message in demo mode
- [x] Row click navigates to device.html?hostname=...

## Runtime QA — Device Page

- [x] Device detail renders correct hostname, score, risk badge
- [x] Issues table shows K1/K2/K6 for PC-A with penalties
- [x] "Tümünü İşaretle" checks all action checkboxes
- [x] "Temizle" unchecks all checkboxes
- [x] Rule filter chips work: Tümü / Sadece Fail / Sadece Kritik
- [x] Copy button shows "✓ Kopyalandı" feedback
- [x] status.txt preview renders correctly
- [x] "← Geri" link navigates back to index

## WebSocket QA

- [x] No duplicate connections (readyState guard)
- [x] Reconnect backoff: 1s→2s→3s→4s→5s cap
- [x] Max 5 consecutive failures stops retrying
- [x] `_wsReconnectFails` resets on successful connect
- [x] `_wsReconnectFails` resets when exiting demo mode (fixed)
- [x] Non-JSON messages silently ignored (try/catch in onmessage)
- [x] Keepalive ping (5s interval) cleaned up on disconnect

## UI/UX Text & Consistency

- [x] Seviye labels consistent: Kritik / Dikkat / İyi
- [x] Button labels consistent: Yenile / Tekrar Dene / Taramayı Başlat / CSV İndir
- [x] WS label syncs with actual connection state (fixed)
- [x] No "undefined" or "null" text in UI anywhere
- [x] All Turkish characters display correctly

## Console Health

- [x] Zero JavaScript errors (TypeError, ReferenceError, etc.)
- [x] Network errors (ERR_CONNECTION_REFUSED) are expected when backend is off — not JS errors
