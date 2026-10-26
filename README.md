# Net Max

A professional Windows desktop network monitoring application built with Python and PyQt6.
Tracks real-time internet usage for the whole system and per application, stores history in SQLite, and sends quota alerts.

---

## Features

- **Dashboard** – Live download/upload speeds, active connections, today/month/30-day data totals, adapter info
- **Applications** – Per-process traffic table with search, sort, and PID info
- **Statistics** – Live last-60 s graph, rolling 24 h chart, 30-day bar chart
- **Settings** – Windows autostart, minimize-to-tray, monthly quota alerts, CSV export, data reset

---

## Requirements

- Windows 10 or 11 (64-bit)
- Python 3.12 or newer
- Standard user account (admin improves per-app accuracy)

---

## Installation

### 1. Clone or extract the project

```
git clone https://github.com/ahmadibrahim4geo/NetMax.git
cd NetMax
```

### 2. Create a virtual environment (recommended)

```
python -m venv .venv
.venv\Scripts\activate
```

### 3. Install dependencies

```
pip install -r requirements.txt
```

> **Note:** `pywin32` requires the post-install step only if it prints an error:
> `python .venv\Scripts\pywin32_postinstall.py -install`

### 4. Run the application

```
python main.py
```

---

## Project Structure

```
netscope/
│
├── main.py                     Application entry point
│
├── database/
│   ├── __init__.py
│   └── db_manager.py           Thread-safe SQLite manager, schema, all queries
│
├── models/
│   ├── __init__.py
│   └── data_models.py          Pure dataclasses (no UI/DB logic)
│
├── monitoring/
│   ├── __init__.py
│   └── network_monitor.py      psutil-based polling loop; emits samples + app records
│
├── services/
│   ├── __init__.py
│   ├── app_service.py          Bridges monitor → DB → alerts → Qt signals
│   ├── export_service.py       CSV export helpers
│   └── system_service.py       Windows registry autostart helpers
│
├── ui/
│   ├── __init__.py
│   ├── styles.py               Global dark stylesheet + colour constants
│   ├── widgets.py              Reusable widgets (StatCard, SidebarButton, etc.)
│   ├── main_window.py          Main window with sidebar, stack, tray icon
│   ├── dashboard_page.py       KPI cards + live speed graph
│   ├── apps_page.py            Per-application traffic table
│   ├── stats_page.py           Historical charts (1 h, 24 h, 30 days)
│   └── settings_page.py        User preferences UI
│
├── assets/                     Icons and images (optional)
├── exports/                    CSV exports are saved here
├── settings/                   Reserved for future per-user config files
│
├── requirements.txt
├── netscope.spec               PyInstaller build spec
└── README.md
```

---

## Privacy and generated files

Net Max stores local network activity history and executable paths in a SQLite database at `%APPDATA%\\Net Max\\netscope.db` (created automatically on first launch). The application does **not** need a prepopulated database in the source repository. Generated database files (including SQLite WAL/SHM), usage reports, caches, logs and local settings must not be committed or attached to public issues. Check screenshots and exported reports before sharing them. Existing git history can retain data even after removing files from the current branch.

**Measurement limitation:** `psutil.Process.io_counters().read_bytes/write_bytes` measure process **disk I/O**, not network bytes. The current per-application traffic values should **not** be interpreted as accurate per-process network usage, including when running as Administrator. Total adapter/network counters use `psutil.net_io_counters()`. An accurate per-process network meter would need a separate supported collection mechanism and Windows-native validation.

## Database Schema

The SQLite database (`netscope.db`) is created automatically on first run.

| Table                  | Purpose                                              |
|------------------------|------------------------------------------------------|
| `bandwidth_snapshots`  | Raw per-second total NIC byte counts                 |
| `app_traffic`          | Per-process byte deltas, recorded every poll cycle   |
| `daily_totals`         | Aggregated bytes per calendar day                    |
| `monthly_totals`       | Aggregated bytes per calendar month                  |
| `app_daily_totals`     | Per-app aggregated bytes per day                     |
| `settings`             | Key-value user preferences                           |

Raw snapshots older than 7 days are pruned automatically to keep the DB small.

---

## Building a Standalone Executable

### Prerequisites

```
pip install pyinstaller
```

### Build

```
pyinstaller netscope.spec --clean
```

The output is in `dist\Net Max\Net Max.exe`.

### Single-file build (optional, slower startup)

```
pyinstaller main.py --onefile --noconsole --name "Net Max" --clean
```

---

## Per-Application Traffic Accuracy

Net Max currently reads process disk-I/O counters as a proxy. **This does not measure per-application network traffic accurately.** See the measurement limitation above.

- **Standard user:** some process connection/metadata information may be unavailable. Aggregate adapter counters still report traffic, subject to counter reset, virtual interfaces and polling limitations.
- **Administrator:** extra visibility does **not** turn process disk-I/O values into network-usage measurements.

Do not elevate solely to improve the currently unsupported per-app network attribution.

---

## Troubleshooting

| Symptom                    | Solution                                                   |
|----------------------------|------------------------------------------------------------|
| App shows 0 B/s always     | Check that psutil can access network counters. Try admin.  |
| No tray icon               | Ensure system tray is enabled in Windows taskbar settings. |
| pywin32 import error       | Run `python Scripts\pywin32_postinstall.py -install`       |
| High CPU usage             | Increase the UI refresh interval in Settings.              |
| DB grows large             | Use *Reset All Statistics* in Settings to clear history.   |

---

## License

MIT — free to use, modify, and distribute.
