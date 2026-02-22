# Demo QA Script

## Prerequisites
```bash
# Start backend
cd operationscore
uvicorn server.main:app --host 0.0.0.0 --port 8000
```

## Demo Steps

### 1. Open Dashboard
- Navigate to `http://127.0.0.1:8000/dashboard/index.html`
- **Expect**: Header shows "OperationScore", "Son veri: —", "Taramayı Başlat" button
- **Expect**: Tab bar: Cihazlar (active) / Analiz / Metodoloji
- **Expect**: If no devices: "Henüz cihaz yok…" message

### 2. Start Scan
- Click "🔍 Taramayı Başlat"
- **Expect**: Button changes to "⏳ Taranıyor…" (disabled)
- **Expect**: Progress pill appears in header with spinner + "Scan: 0/N (0%)"
- **Expect**: Scan status card below header: "Tarama başlatıldı..."
- Wait for devices to report
- **Expect**: Progress updates "Scan: 1/N → 2/N → N/N (100%)"
- **Expect**: On Cihazlar tab: green/gray dots in Scan column
- **Expect**: On completion: "✅ Tamamlandı ✓" → 3s → "Son tarama: HH:MM"
- **Expect**: Button returns to "🔍 Taramayı Başlat"

### 3. Verify Cihazlar Tab
- **Expect**: Devices split into "Sunucular (SERVER)" and "Clientlar (CLIENT)"
- **Expect**: Sorted by score ascending (worst first), nulls at bottom
- **Expect**: KPI cards: device count, critical count, avg score, server/client ratio
- Try search box — filters by hostname
- Try CSV export — downloads CSV

### 4. Click a Device → Detail
- Click any device row
- **Expect**: URL changes to `#device?hostname=...`
- **Expect**: Header: hostname, device type badge, last_seen, IP, score + risk
- **Expect**: Score history chart (y-axis 0–100) or "Henüz tarama geçmişi yok"
- **Expect**: "Top Nedenler" panel (max 3 items or "Öneri yok.")
- **Expect**: "Yapılacaklar" panel (max 3 items or "Öneri yok.")
- Click "← Cihazlara Dön" → returns to list

### 5. Open Analiz Tab
- Click "Analiz" tab
- **Expect**: Fleet avg score chart (line)
- **Expect**: Server vs Client avg chart (two lines)
- **Expect**: Critical count chart (bar) — only if data has critical_count
- **Expect**: If no data: "Henüz fleet geçmiş verisi yok…"

### 6. Open Metodoloji Tab
- Click "Metodoloji" tab
- **Expect**: Title "OperationScore Metodoloji"
- **Expect**: "Skor = 100 − Σ cezalar" formula
- **Expect**: Risk levels table (EXCELLENT → CRITICAL)
- **Expect**: 8 criteria cards (K1–K8)

## API Endpoints Used

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/tasks/broadcast` | Start scan, body: `{"command":"run_scan"}` |
| GET | `/scan_sessions/{run_id}` | Poll scan progress |
| GET | `/api/devices` | Device list with scores |
| GET | `/api/fleet/history?limit=200` | Fleet analytics charts |
| GET | `/api/fleet/history?limit=1` | Header "Son veri" timestamp |
| GET | `/api/devices/{hostname}/history?limit=100` | Device score history chart |

## Expected Response Shapes

```json
// POST /tasks/broadcast
{ "run_id": "uuid", "devices_targeted": 3, "command": "run_scan", "timestamp": "..." }

// GET /scan_sessions/{run_id}
{ "run_id": "uuid", "created_at": "...", "devices": { "host1": "completed", "host2": "pending" } }

// GET /api/devices
{ "device_count": 3, "devices": [{ "hostname": "...", "device_type": "SERVER", "latest_score": 72.5, "risk_level": "MEDIUM", "last_seen_at": "...", "last_seen_ip": "..." }] }

// GET /api/fleet/history?limit=N
{ "points": [{ "timestamp": "...", "fleet_avg": 75.2, "server_avg": 80, "client_avg": 70, "critical_count": 1 }] }

// GET /api/devices/{hostname}/history?limit=100
{ "hostname": "...", "points": [{ "timestamp": "...", "score": 72.5 }] }
```

## Edge Cases to Test
- No devices registered → "Kayıtlı cihaz yok" on scan
- Device with null score → shows "—" and sorted to bottom
- Empty fleet history → "Henüz fleet geçmiş verisi yok"
- Empty device history → "Henüz tarama geçmişi yok"
- Direct URL `#device?hostname=nonexistent` → "bulunamadı" with back button
- Scan timeout (>120s) → "Zaman aşımı" message
