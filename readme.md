# Satellite Tracking Visualizer

A browser-based dashboard for inspecting satellite antenna tracking logs. Drop in a
telemetry log file and the app turns it into interactive charts — pointing accuracy,
signal strength, motor angles, sky position and tracking path — with a scrubbable
timeline, time-zone aware clocks, and one-click PDF reports.

Everything runs locally in the browser. No data is uploaded to a server.

---

## Features

### Load logs
- Drag and drop, or pick files from disk
- Accepts `.csv` logs and Excel workbooks (`.xlsx`, `.xls`, `.xlsm`)
- Handles both epoch-microsecond timestamps and wall-clock `MM:SS.s` / `HH:MM:SS.s` logs
- Reads the newer logs that include an `nbd` (dBm) column, and derives signal level from raw
  RSSI for older logs that don't
- Load several logs at once and switch between them from **Manage Logs**

### Views
| View | What it shows |
| --- | --- |
| **Dashboard** | All key charts on one page, plus a sky plot and a timeline scrubber |
| **AZ / EL** | Actual vs. target azimuth and elevation over flight time |
| **PAE** | Pointing accuracy error on the X and Y joints |
| **RSSI** | Received signal strength, in dBm |
| **X / Y Motor** | Motor joint angles over time |
| **Tracking Path** | Antenna pointing vs. satellite look angle vs. conical scan center |

### Chart controls
- **Zoom and pan** every time-series chart — scroll to zoom, drag to pan, or use the zoom
  buttons and scrollbars
- **Combine series** across views (e.g. overlay RSSI and PAE on one chart)
- **Time zone picker** — display timestamps in UTC, your local zone, or any of ~40 common
  zones, with a searchable list
- **Timeline scrubber** — step through the log sample by sample; every chart, the sky plot
  and the readout panel follow the same cursor
- **Click any data point** to jump the timeline to that sample

### Export
- Export the current view as a **PDF report**
- Choose a sample range to produce one page per sample (with a warning for very large ranges)
- Charts are embedded as vectors where possible, so the output stays sharp and searchable
- Report types: dashboard snapshots, AZ/EL, PAE, RSSI

---

## Getting started

```bash
# install dependencies
npm install

# start the dev server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) and drop a log file onto the upload panel.

### Other scripts

```bash
npm run build   # production build
npm start       # serve the production build
npm run lint    # lint the project
```

---

## Log file format

Logs are comma-separated with **no header row**. Columns, in order:

| # | Column | Unit | Description |
| --- | --- | --- | --- |
| 0 | `timestamp` | µs since epoch | Sample time (or a wall-clock string in some exports) |
| 1 | `cur_joint_Y` | deg | Current Y-axis motor angle |
| 2 | `cur_joint_X` | deg | Current X-axis motor angle |
| 3 | `cur_az` | deg | Current azimuth |
| 4 | `cur_el` | deg | Current elevation |
| 5 | `target_az` | deg | Target azimuth |
| 6 | `target_el` | deg | Target elevation |
| 7 | `pae_joint_Y` | deg | Pointing accuracy error, Y |
| 8 | `pae_joint_X` | deg | Pointing accuracy error, X |
| 9 | `cs_center_az` | deg | Conical scan center azimuth |
| 10 | `cs_center_el` | deg | Conical scan center elevation |
| 11 | `cs_target_az` | deg | Conical scan target azimuth |
| 12 | `cs_target_el` | deg | Conical scan target elevation |
| 13 | `rssi` | raw | Received signal strength indicator |
| 14 | `nbd` | dBm | Signal level — newer logs only |
| 15 | `tx_enable` | bool | Transmit enabled |

Older logs omit the `nbd` column; in that case `tx_enable` (if present) sits at index 14 and
the signal level is derived from the raw RSSI.

Quoted fields containing commas (e.g. `"1,234"`) are handled correctly.

---

## Tech stack

- **Next.js 15** (App Router) with **React 19** and **TypeScript**
- **Tailwind CSS** for styling
- **Recharts** for time-series charts; the sky plot is hand-drawn SVG
- **SheetJS (`xlsx`)** for reading Excel logs
- **jsPDF** + **svg2pdf.js** for vector PDF export, with **html2canvas** as a raster fallback

---

## Project structure

```
app/                 App shell and the single dashboard page
components/          Charts, sidebar, upload panel, zoom/scroll controls
lib/                 Log parsing, chart layout maths, zoom/pan hooks, PDF export
```

---

## Notes

- All parsing and rendering happens client-side; nothing leaves the browser.
- Very large logs are parsed in one pass and kept in memory — extremely long logs may be
  slow to load on low-memory devices.
- PDF export of a large sample range creates one page per sample; the app warns before
  starting a run over 250 pages.
