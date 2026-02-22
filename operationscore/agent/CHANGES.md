# OperationScore Agent — Changes Report

All changes are scoped to `agent/` only. No server, docs, or README files were modified.

---

## agent/collector.py

### 1. K5 Blacklist Updated
**Location:** `get_unnecessary_services()`

```diff
- unnecessary = ["apache2", "bluetooth", "cups", "avahi-daemon"]
- result = subprocess.run(f"systemctl is-active --quiet {service}", shell=True, ...)
+ blacklist = ["telnet", "vsftpd", "proftpd", "transmission-daemon"]
+ result = subprocess.run(["systemctl", "is-active", "--quiet", service], capture_output=True, timeout=3)
```

- Blacklist updated to hackathon-specified services
- Switched from `shell=True` string interpolation to list form (safer, no shell injection)
- Per-service try/except: handles `FileNotFoundError` (no systemctl), `TimeoutExpired`, generic errors
- On any failure: service treated as "not active", script continues

---

### 2. `sudo` Password Prompt Removed
**Location:** `check_firewall_enabled()`

```diff
- output = self.run_command("sudo ufw status", shell=True)
+ for fw in ("ufw", "firewalld"):
+     result = subprocess.run(["systemctl", "is-active", "--quiet", fw], timeout=3, capture_output=True)
+     if result.returncode == 0:
+         return True
+ return False
```

- Eliminated the only `sudo` call in the codebase
- No root required: `systemctl is-active --quiet` only checks service state
- Falls back gracefully if systemctl is unavailable

---

### 3. `requests` Import Made Lazy
**Location:** `send_metrics()` and `get_task()`

```diff
- import requests  # top of file
+ # Inside send_metrics() and get_task():
+ import requests  # lazy — only needed for network calls
```

- Module now imports cleanly on systems without `requests` installed (e.g. during --dry-run)
- Each method imports locally only when actually executing a network call

---

### 4. UTC-Aware Timestamps
**Location:** `__init__()` and `poll_tasks()` loop

```diff
- self.timestamp = datetime.now().isoformat()
+ self.timestamp = datetime.now(timezone.utc).isoformat()
```

- All timestamps are now timezone-aware UTC (e.g. `2026-02-21T10:14:12+00:00`)
- Previously naive (local time, no TZ info) which could cause issues in multi-timezone deployments

---

### 5. Safer Task URL Construction
**Location:** `__init__()`

```diff
- self.task_api_url = self.api_url.replace("/report", "/tasks")
+ parsed = urlparse(api_url)
+ self.task_api_url = parsed._replace(path="/tasks").geturl()
```

- Old method would corrupt URLs containing `/report` elsewhere in the path
- `urlparse` replaces only the URL path component, safely

---

### 6. `run_command()` — Improved Exception Handling + Timeout Parameter
**Location:** `run_command()`

```diff
- except Exception as e:
-     logger.warning("Command error: %s", e)
+ except FileNotFoundError:
+     logger.warning("Command not found: %s", command)
+ except subprocess.TimeoutExpired:
+     logger.warning("Command timed out after %ss: %s", timeout, command)
+ except Exception as exc:
+     logger.warning("Unexpected error running %r: %s", command, exc)
```

- Now distinguishes: binary missing vs timeout vs unexpected error
- Added `timeout: int = 5` parameter — callers can override per-command

---

### 7. Standard Library Imports Sorted Alphabetically
Top-level imports reorganized; removed unused entries.

---

## agent/ops_collect.py (New File)

Single-shot entrypoint for Liderahenk Betik / cron jobs.

### CLI Arguments (all 9)

| Argument | Default |
|---|---|
| `--api-url` | `http://127.0.0.1:8000/report` |
| `--token` | `""` |
| `--dry-run` | `False` |
| `--write-status` | `/var/lib/operationscore/status.txt` |
| `--no-status` | `False` |
| `--notify` | `False` |
| `--critical-threshold` | `60` |
| `--timeout` | `5` |
| `--print-pretty` | `False` |

### Execution Flow

```
collect_metrics()
    ↓
--dry-run? → print OPERATIONSCORE_JSON → exit 0
    ↓
POST to api_url (with Content-Type + X-OPS-TOKEN if set)
    ↓
Error/non-200? → print OPERATIONSCORE_RESULT (error) → OPERATIONSCORE_JSON → exit 1
    ↓
Parse score, issues → compute risk_level → build top_reasons + actions
    ↓
Write status.txt (unless --no-status, permission errors warned to stderr)
    ↓
--notify AND score < threshold? → print OPERATIONSCORE_ALERT → try notify-send
    ↓
print OPERATIONSCORE_RESULT → OPERATIONSCORE_JSON → exit 0
```

### Risk Level Mapping

| Score | Level |
|---|---|
| 90–100 | `EXCELLENT` |
| 75–89 | `LOW` |
| 60–74 | `MEDIUM` |
| 40–59 | `HIGH` |
| 0–39 | `CRITICAL` |

### Top Reasons + Actions Algorithm

- Sort issues by `penalty` descending
- Take top 3
- `top_reasons`: `f"{rule_id} {message}"`
- `actions`: `issue["recommendation"]`, truncated to 120 chars + `"..."` if longer

### Status File Format (4 lines, always overwritten)

```
OperationScore: 25.0/100 (CRITICAL)
Top: <reason1>; <reason2>; <reason3>
Todo: <action1> | <action2> | <action3>
Last: 2026-02-21T10:14:06.447702+00:00
```

### Deployment Hardening

```python
_SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
if _SCRIPT_DIR not in sys.path:
    sys.path.insert(0, _SCRIPT_DIR)
from collector import MetricsCollector
```

Works correctly when deployed to `/opt/operationscore/` or any flat directory,
regardless of the current working directory.

### Non-Interactive Guarantees

- No `input()`, no `getpass()`, no `time.sleep()` in the execution path
- All subprocess calls use `capture_output=True` and `timeout=`
- `notify-send` attempted only if `shutil.which("notify-send")` finds the binary
- `requests` imported lazily (only on actual POST path)
- Permission errors on status.txt: warning to stderr, never crashes

---

## What Was NOT Changed

- `server/` — untouched
- `README.md` — untouched
- `docs/` — untouched
- No new dependencies added (stdlib + requests only)
