# OperationScore Dashboard

Framework-free, offline-ready web dashboard for OperationScore.

## Quick Start

```bash
cd dashboard
python3 -m http.server 5173
```

Open [http://localhost:5173](http://localhost:5173) in your browser.

## Configuring API_BASE

By default the dashboard connects to `http://127.0.0.1:8000`.

To change this, edit the `<script>` block in both **index.html** and **device.html**:

```html
<script>
  window.API_BASE = "http://YOUR_SERVER:PORT";
</script>
```

The WebSocket URL is derived automatically from `API_BASE` (replacing `http` with `ws`).

## File Structure

| File | Purpose |
|---|---|
| `index.html` | Main page — "En Problemli Cihazlar" device table |
| `device.html` | Device detail page (`?hostname=PC-A`) |
| `app.js` | Shared JS: API client, WebSocket, render helpers |
| `style.css` | Shared styles (CSS variables, components) |

## API Endpoints Used

| Endpoint | Method | Purpose |
|---|---|---|
| `/snapshot/latest` | GET | Fetch latest snapshot |
| `/snapshot/rebuild` | POST | Trigger snapshot rebuild |
| `/ws` | WebSocket | Realtime `snapshot_updated` events |
