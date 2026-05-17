# AutoTech Industrial Automation Suite

> A production-ready, full-stack industrial automation dashboard combining **OIS (Operations Intelligence System)** and **NIS (Network Intelligence System)** into a unified engineering platform.

Built by **Alexander.S D Great** — with care.

---

> ## ⚡ Live Demo Notice
>
> **What you see here is a read-only interactive demo.**
>
> The live GitHub Pages version runs entirely in the browser with simulated data — no backend, no database, no real PLC or network connection. All interactions (work orders, alarms, parts, topology edits) are in-memory and reset on refresh.
>
> **The full production source code — including the Node.js backend, SQLite database, Modbus TCP integration, SNMP polling engine, JWT authentication, and Docker deployment — is not published here.**
>
> 📬 **If you'd like access to the complete codebase, want to discuss a deployment, or are interested in collaboration — reach out to me directly:**
>
> - **LinkedIn:** [Alexander S. D Great](https://www.linkedin.com/in/alexander-s-d-great)
> - **Email:** alexanderdgreat08@gmail.com
>
> _I'm happy to walk you through the system, discuss architecture decisions, or explore how it can be adapted for your use case._

---

## Table of Contents

1. [⚡ Live Demo Notice](#-live-demo-notice)
2. [Overview](#overview)
3. [Screenshots](#screenshots)
4. [Tech Stack](#tech-stack)
5. [Prerequisites](#prerequisites)
6. [Quick Start](#quick-start)
7. [Environment Configuration](#environment-configuration)
8. [Default Credentials](#default-credentials)
9. [Connecting to Real Hardware](#connecting-to-real-hardware)
   - [PLC via Modbus TCP](#plc-via-modbus-tcp)
   - [Network Devices via SNMP](#network-devices-via-snmp)
10. [Module Reference](#module-reference)
11. [Docker Deployment](#docker-deployment)
12. [API Reference](#api-reference)
13. [Security Architecture](#security-architecture)
14. [Project Structure](#project-structure)
15. [Keyboard Shortcuts](#keyboard-shortcuts)
16. [Troubleshooting](#troubleshooting)

---

## Overview

AutoTech is a browser-based industrial automation dashboard designed for plant floor engineers and operators. It provides:

| Module | Description |
|--------|-------------|
| **OIS Dashboard** | Live PLC metrics, OEE tracking, production counters |
| **NIS Topology Editor** | Drag-and-drop network mapping with VLAN/OSPF overlay |
| **Work Orders** | Full CRUD with priority levels and CSV export |
| **Preventive Maintenance** | Scheduled task tracking with overdue detection |
| **Spare Parts Inventory** | Stock level management with low-stock alerts |
| **AI Diagnostics** | Fault code lookup powered by AI engine |
| **SNMP Device Management** | Live polling of network devices via SNMP v2c |
| **Alarm History** | Acknowledging and filtering system alarms |
| **Firmware Management** | .bin/.hex upload and flash simulation |
| **Security Audit Log** | Full event log of every authenticated action |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Node.js 18+, Express 4 |
| Database | SQLite 3 (persistent file) |
| Auth | JWT (HTTP-only cookie) + CSRF double-submit |
| Real-time | WebSocket (`ws`) for live PLC data |
| PLC Comms | Modbus TCP (`modbus-serial`) |
| Network | SNMP v2c (`net-snmp`) |
| Frontend | Vanilla HTML/CSS/JS — no framework |
| Typography | Plus Jakarta Sans, DM Mono (Google Fonts) |
| Containerisation | Docker + docker-compose |

---

## Prerequisites

- **Node.js** ≥ 18.x
- **npm** ≥ 9.x
- _(Optional)_ Docker & docker-compose for containerised deployment

---

## Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/your-username/autotech.git
cd autotech

# 2. Install dependencies
npm install

# 3. Copy and configure environment
cp .env.example .env
#    → Edit .env (see Environment Configuration below)

# 4. Start the server
npm start
```

Open your browser at **http://localhost:3000**

> **Tip:** Run `npm run sim` in a second terminal to start the Modbus simulator
> so the dashboard shows live-looking PLC data without real hardware.

---

## Environment Configuration

Copy `.env.example` to `.env` and set the following values:

```env
# Server
PORT=3000

# Database — persistent SQLite file path
DB_PATH=./server/autotech.db

# JWT — CHANGE THIS before any production deployment
JWT_SECRET=your-very-long-random-secret-here

# PLC Connection (Modbus TCP)
PLC_HOST=192.168.1.10
PLC_PORT=502

# SNMP (optional — for live network polling)
SNMP_POLL_INTERVAL_MS=30000
```

> ⚠️ **NEVER commit `.env` to Git.** The `.gitignore` already excludes it.
> Generate a strong JWT secret with: `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`

---

## Default Credentials

> **Demo note:** The live GitHub Pages demo has no login screen — it opens directly on the dashboard as an Engineer. The credentials below apply to the full backend deployment only.

| Username | Password | Role |
|----------|----------|------|
| `aj` | `engineer456` | **Engineer** (full access) |
| `ramon` | `password123` | **Operator** (limited access) |

### Role Permissions

| Feature | Operator | Engineer |
|---------|----------|----------|
| Dashboard / OEE | ✅ | ✅ |
| Work Orders (view, update status) | ✅ | ✅ |
| Parts (view, consume) | ✅ | ✅ |
| PM (view, mark done) | ✅ | ✅ |
| NIS Topology Editor | ✅ | ✅ |
| Add / Delete Parts, PM, WO | ❌ | ✅ |
| Robot Teach Points | ❌ | ✅ |
| Firmware Upload | ❌ | ✅ |
| **Audit Log** | ❌ | ✅ |
| SNMP Device Management | ❌ | ✅ |

---

## Connecting to Real Hardware

### PLC via Modbus TCP

The system communicates with PLCs using the **Modbus TCP** protocol over your plant network.

**Step 1 — Configure the PLC IP in `.env`:**
```env
PLC_HOST=192.168.1.10   # Your PLC's IP address
PLC_PORT=502            # Default Modbus port
```

**Step 2 — Verify register mapping in `server/plc_comm.js`:**

```js
// Holding registers read from the PLC
// FC3 — Read Holding Registers
const REGISTERS = {
  lineSpeed:   0,   // HR0: Line speed (RPM × 10)
  temperature: 1,   // HR1: Temperature (°C × 10)
  pressure:    2,   // HR2: Air pressure (PSI × 10)
  oeeValue:    3,   // HR3: OEE % × 10
  partCount:   4,   // HR4: Part counter (units)
};
```

Edit these register addresses to match your PLC program's memory map.

**Step 3 — Check network connectivity:**
```bash
# From the server machine, test Modbus reachability
node -e "
const Modbus = require('modbus-serial');
const client = new Modbus();
client.connectTCP('192.168.1.10', { port: 502 })
  .then(() => { console.log('PLC connected!'); client.close(); })
  .catch(e => console.error('Failed:', e.message));
"
```

**Step 4 — Start the server.** Live data streams via WebSocket to the dashboard.

> If no PLC is available, run `npm run sim` for a Modbus simulator on `localhost:5020`.

---

### Network Devices via SNMP

AutoTech polls network devices (switches, routers, firewalls) using **SNMP v2c** to get real-time CPU, memory, and interface data.

**Requirements:**
- Install the optional SNMP dependency: `npm install net-snmp`
- SNMP must be **enabled** on each device with a community string (default: `public`)

**Adding a device via the UI:**

1. Log in as **Engineer**
2. Switch to **NIS** module → **SNMP Devices**
3. Fill in:
   - **Node ID** — must match the node's `id` in your topology (e.g. `core-01`)
   - **IP Address** — device's management IP
   - **Community** — SNMP read community string (e.g. `public`, `readonly`)
4. Click **Add Device** — polling starts immediately

**Adding a device via API:**
```bash
curl -X POST http://localhost:3000/api/snmp/targets \
  -H "Content-Type: application/json" \
  -H "X-CSRF-Token: YOUR_CSRF_TOKEN" \
  -b "autotech_session=YOUR_SESSION_COOKIE" \
  -d '{"nodeId":"core-01","ip":"192.168.1.1","community":"public"}'
```

**OIDs polled by default:**

| Metric | OID |
|--------|-----|
| CPU Load (1-min avg) | `1.3.6.1.4.1.9.2.1.57` (Cisco) or `1.3.6.1.2.1.25.3.3.1.2` |
| Memory Used % | `1.3.6.1.4.1.9.2.1.8` |
| Interface Status | `1.3.6.1.2.1.2.2.1.8` |
| System Uptime | `1.3.6.1.2.1.1.3.0` |

Edit `server/snmp_poller.js` to customise OIDs for your vendor.

---

## Module Reference

### NIS Topology Editor

The detached topology editor (`/topology_editor.html`) is a full Visio-style network mapping tool.

| Action | How |
|--------|-----|
| Add device | Drag from **Device Library** sidebar OR press **A** then click canvas |
| Draw link | Press **L**, click source node, click target node |
| Delete | Press **Del** after selecting, or right-click → Delete |
| Multi-select | Hold **Ctrl** and click nodes |
| Move group | Select multiple → drag any selected node |
| Auto-save | Layout saves automatically every **30 seconds** |
| Manual save | Click **Save** button or press nothing — it just works |
| Shortcut help | Press **?** |

**Link types** — right-click a link to change style:
- **Solid line** = Fiber / uplink
- **Dashed line** = Copper / access
- **Dotted line** = Wireless

### Audit Log

Navigate to **OIS → Audit Log** (Engineer only). Click **Refresh** to load the latest 200 events. Use the search bar to filter by user, action, or IP address.

### CSV Export

Every major table has a **CSV** button in the top-right:
- Work Orders → `work_orders.csv`
- Parts Inventory → `parts_inventory.csv`
- PM Tasks → `pm_tasks.csv`

---

## Docker Deployment

```bash
# Build and start
docker-compose up --build -d

# View logs
docker-compose logs -f

# Stop
docker-compose down
```

The `docker-compose.yml` maps port **3000** and mounts `./server/autotech.db`
as a persistent volume so data survives container restarts.

> ⚠️ Set `JWT_SECRET` as a Docker secret or environment variable — never
> hard-code it in the image.

---

## API Reference

All endpoints require a valid session cookie. State-changing endpoints (`POST`, `PUT`, `PATCH`, `DELETE`) also require the `X-CSRF-Token` header.

### Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/login` | Login — returns JWT cookie + CSRF token |
| `POST` | `/api/logout` | Invalidates session |
| `GET` | `/api/session` | Returns current user info + fresh CSRF token |

### Work Orders
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/work_orders` | List all WOs |
| `POST` | `/api/work_orders` | Create WO (`description`, `assigned_to`, `priority`) |
| `PATCH` | `/api/work_orders/:id/status` | Update status |
| `PUT` | `/api/work_orders/:id` | Edit WO |
| `DELETE` | `/api/work_orders/:id` | Delete WO (engineer only) |

### Parts Inventory
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/parts` | List parts |
| `POST` | `/api/parts` | Add part |
| `POST` | `/api/parts/:id/consume` | Decrement quantity by 1 |
| `PATCH` | `/api/parts/:id` | Restock (set quantity) |
| `DELETE` | `/api/parts/:id` | Delete part (engineer only) |

### Preventive Maintenance
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/pm` | List PM tasks |
| `POST` | `/api/pm` | Add task |
| `PATCH` | `/api/pm/:id/complete` | Mark complete |
| `PUT` | `/api/pm/:id` | Edit task |
| `DELETE` | `/api/pm/:id` | Delete task |

### NIS Topology
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/nis/snapshot` | Full topology snapshot |
| `PATCH` | `/api/nis/topology/positions` | Bulk update node positions |
| `POST` | `/api/nis/topology/node` | Add node |
| `PATCH` | `/api/nis/topology/node/:id` | Edit node |
| `DELETE` | `/api/nis/topology/node/:id` | Delete node |
| `POST` | `/api/nis/topology/link` | Add link |
| `PATCH` | `/api/nis/topology/link` | Edit link status |
| `DELETE` | `/api/nis/topology/link` | Delete link |

### SNMP Management
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/snmp/targets` | List polled devices |
| `POST` | `/api/snmp/targets` | Add device |
| `DELETE` | `/api/snmp/targets/:nodeId` | Remove device |
| `PATCH` | `/api/snmp/config` | Update poll interval |

### Export
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/export/work_orders.csv` | Download WO CSV |
| `GET` | `/api/export/parts.csv` | Download Parts CSV |
| `GET` | `/api/export/pm.csv` | Download PM CSV |

### Security
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/audit_logs` | Last 200 audit events (engineer only) |
| `GET` | `/api/health` | Server + DB health check |

---

## Security Architecture

| Layer | Implementation |
|-------|---------------|
| **Password Hashing** | PBKDF2 — 120,000 iterations, SHA-256, 32-byte key |
| **Session Auth** | JWT in HTTP-only, SameSite=Strict cookie |
| **CSRF Protection** | Double-submit cookie + `X-CSRF-Token` header on all mutations |
| **SQL Injection** | 100% parameterized queries — no string concatenation |
| **XSS** | All user data escaped via `escapeHtml()` before DOM insertion |
| **HTTP Headers** | `X-Content-Type-Options`, `X-Frame-Options`, `X-XSS-Protection`, `Content-Security-Policy`, `HSTS` |
| **Rate Limiting** | Login: 8 attempts / 15 min · API: 240 req / min |
| **Input Sanitization** | `cleanString()` applied to all string params before DB ops |
| **RBAC** | Role checked server-side on every protected route |
| **Audit Logging** | Every authenticated action written to `audit_logs` table |
| **Atomic File Writes** | Inventory JSON written to temp file then renamed — no corruption |

---

## Project Structure

```
autotech/
├── server/
│   ├── server.js              # Main Express app, all API routes
│   ├── database.js            # SQLite init, migrations, seed data
│   ├── plc_comm.js            # Modbus TCP client
│   ├── modbus_simulator.js    # Local PLC simulator (npm run sim)
│   ├── snmp_poller.js         # SNMP polling engine
│   ├── diagnostics.js         # AI fault code lookup engine
│   ├── network_intelligence.js # NIS data processing
│   ├── nis_inventory.json     # Persistent NIS topology store
│   └── autotech.db            # SQLite database (git-ignored)
│
├── js/
│   ├── api.js                 # Frontend fetch wrapper + auth helpers
│   ├── app.js                 # Main dashboard logic
│   ├── topology_editor.js     # NIS canvas editor
│   ├── topology_enhancements.js # Auto-save + shortcut overlay
│   └── utils.js               # Search, CSV export, audit log viewer
│
├── css/
│   ├── styles.css             # Core design system (sand-gold theme)
│   └── enhancements.css       # Pulse animation, search bars, overlays
│
├── automation_tech_dashboard.html  # Main dashboard SPA
├── topology_editor.html            # Detached topology editor window
├── .env                            # Environment config (git-ignored)
├── .env.example                    # Template — safe to commit
├── .gitignore                      # Protects .env, DB, uploads
├── Dockerfile
├── docker-compose.yml
└── package.json
```

---

## Keyboard Shortcuts

### Dashboard (Global)

| Action | Shortcut |
|--------|----------|
| Acknowledge alarm | Click **Ack** button |
| Add shift log entry | Click **+ Add Entry** |
| Export PDF report | Click **Export Engineering Report** |

### Topology Editor

| Action | Key |
|--------|-----|
| Select / Move mode | `V` |
| Add Device mode | `A` |
| Draw Link mode | `L` |
| Toggle Grid Snap | `G` |
| Zoom In | `+` |
| Zoom Out | `-` |
| Fit to View | `0` |
| Select All | `Ctrl+A` |
| Undo | `Ctrl+Z` |
| Delete Selected | `Del` |
| Clear Selection | `Esc` |
| Shortcut Help | `?` |

---

## Troubleshooting

### "Invalid credentials" on login

The DB may have stale plaintext passwords. The server auto-migrates on startup — just restart:
```bash
npm start
```
If the problem persists, delete `server/autotech.db` to trigger a fresh seed.

### Topology editor is blank (no nodes)

1. Hard-refresh the browser: `Ctrl+Shift+R`
2. Confirm you are logged in on the **main dashboard** before opening the topology editor — it inherits the session cookie.
3. Check the browser console for `Load failed` errors. The server must be running on port 3000.

### PLC not connecting

```bash
# Test TCP reachability first
curl telnet://192.168.1.10:502
# Or with netcat
nc -zv 192.168.1.10 502
```
Verify `PLC_HOST` and `PLC_PORT` in `.env`. Check plant firewall rules.

### SNMP data not updating

1. Ensure `net-snmp` is installed: `npm install net-snmp`
2. Test manually:
   ```bash
   snmpget -v2c -c public 192.168.1.1 1.3.6.1.2.1.1.3.0
   ```
3. Confirm the device has SNMP v2c enabled and the community string matches.

### Session expires too quickly

Adjust `JWT_TTL_SECONDS` in `.env` (default: 3600 = 1 hour). A warning banner appears 5 minutes before expiry.

### Database locked errors (SQLite)

SQLite does not support multiple writer processes. Ensure only **one** instance of the server is running. If using Docker, do not mount the DB file in a shared network drive.

---

## License

Copyright © 2026 **Alexander.S D Great®**. All rights reserved.

Built with care — AS

---

## Contact

The full source code for this project is not publicly available. If you're interested in the complete system, a custom deployment, or have any questions:

- **LinkedIn:** [Alexander S. D Great](https://www.linkedin.com/in/alexander-s-d-great)
- **Email:** alexanderdgreat08@gmail.com

---

*For issues or questions about the demo — open a GitHub issue.*
