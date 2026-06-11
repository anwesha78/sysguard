# SysGuard — Automated System Health Monitor & Incident Response Engine

> **"Don't wait for systems to fail. Watch them breathe."**

SysGuard is a Python-based Site Reliability Engineering (SRE) automation tool that continuously monitors real-time system health, intelligently detects anomalies, declares incidents, and auto-generates professional Root Cause Analysis (RCA) reports — without any human intervention.

Built to mirror real-world SRE practices used by engineering teams at scale.

---

## Why Does SysGuard Exist?

In production environments, systems fail silently. A memory leak at 2 AM, a CPU spike during peak traffic, a disk filling up over days — these are the incidents that take down services and cost businesses millions.

Traditional monitoring requires:
- A human sitting and watching dashboards
- Manual log digging after something breaks
- Writing incident reports from scratch under pressure

**SysGuard eliminates all three.**

It watches your system 24/7, thinks for itself, and hands you a complete incident report the moment something goes wrong — formatted, timestamped, and ready to act on.

---

## What SysGuard Does

### Real-Time Metric Collection
Every 5 seconds, SysGuard collects:
- **CPU Usage** — percentage of processor being consumed
- **RAM Usage** — percentage of memory in use
- **Disk Usage** — percentage of storage consumed
- **Network I/O** — total MB sent and received since boot

### Intelligent Anomaly Detection
SysGuard does NOT fire alerts on a single bad reading.

It uses **consecutive-breach logic** — a metric must exceed its threshold for **3 or more consecutive monitoring cycles** before an incident is declared.

Why does this matter? Because:
- A CPU spike of 1 second is normal — a browser tab loading
- A CPU spike for 15+ seconds is a problem — a runaway process

This logic is exactly how production monitoring systems like Prometheus Alertmanager work. Most student projects use naive single-threshold alerts. SysGuard does not.

### Incident Declaration
Once 3 consecutive breaches are confirmed:
- A `[CRITICAL]` incident is declared in real time
- Every subsequent breach is logged with its exact value and timestamp
- The incident stays active until metrics normalise

### Auto-Generated RCA Reports
The moment an incident resolves (metrics drop below threshold OR the monitor is stopped):

SysGuard **automatically writes a complete Root Cause Analysis report** containing:

1. **Incident Summary** — what happened, which metrics breached, for how long
2. **Incident Timeline** — exact timestamps from first detection to resolution
3. **Metrics at Time of Incident** — peak values vs thresholds
4. **Root Cause Analysis** — probable cause based on which metric was involved
5. **Impact Assessment** — duration, cycles breached, systems affected
6. **Resolution Steps** — step-by-step commands to investigate and fix
7. **Preventive Measures** — what to do so this never happens again
8. **Lessons Learned** — engineering takeaways from the incident

Each report is saved as a uniquely named `.txt` file with incident ID and timestamp.

### Structured Date-Wise Logging
Every monitoring cycle is written to a date-stamped log file in `/logs`:
```
logs/sysguard_2026-06-11.log
```
Logs include level tags: `[INFO]`, `[WARN]`, `[CRITICAL]`, `[REPORT]` — making them grep-friendly and audit-ready.

---

## Real Incident Detected During Development

SysGuard wasn't just tested on simulated data. During its first run, it detected a **live RAM incident** on the development machine:

```
[2026-06-11 13:27:06] [WARN] ⚠ WARNING — RAM at 81.4% (threshold: 80%) [breach #1]
[2026-06-11 13:27:12] [WARN] ⚠ WARNING — RAM at 81.4% (threshold: 80%) [breach #2]
[2026-06-11 13:27:18] [WARN] ⚠ WARNING — RAM at 81.8% (threshold: 80%) [breach #3]
[2026-06-11 13:27:18] [CRITICAL] 🚨 INCIDENT DECLARED — RAM breach sustained for 3 consecutive readings
...
[2026-06-11 13:28:32] [REPORT] RCA Report generated → reports/RCA_INCIDENT_2026-06-11_13-28-32.txt
```

**Incident Summary:**
- Metric breached: RAM
- Peak value: 82.6% (threshold: 80%)
- Duration: 65 seconds across 13 monitoring cycles
- RCA auto-generated with full timeline, cause, and resolution steps

This was not a simulation. This was a real system behaving as production systems do.

---

## Project Structure

```
sysguard/
├── monitor.py                          ← Core monitoring engine
├── logs/
│   └── sysguard_2026-06-11.log        ← Date-wise structured logs
├── reports/
│   └── RCA_INCIDENT_2026-06-11_13-28-32.txt  ← Auto-generated RCA
└── README.md
```

---

## Tech Stack

| Technology | Purpose |
|---|---|
| **Python 3** | Core automation language |
| **psutil** | Industry-standard library for real-time system metrics |
| **os module** | File system and directory management |
| **datetime** | Precise timestamping for incident timelines |
| **File I/O** | Structured log writing and RCA report generation |
| **Exception handling** | Graceful shutdown with final RCA on Ctrl+C |
| **CLI / Terminal** | Runs entirely on command line — no dependencies, no UI |

---

## How to Run

**1. Install dependency:**
```bash
pip install psutil
```

**2. Clone the repo:**
```bash
git clone https://github.com/anwesha78/SysGuard-Automated-System-Health-Monitor.git
cd SysGuard-Automated-System-Health-Monitor
```

**3. Create required directories:**
```bash
mkdir -p logs reports
```

**4. Run the monitor:**
```bash
python3 monitor.py
```

**5. To trigger an incident manually (stress test):**
```bash
# Open a new terminal and run:
python3 -c "x = [i**2 for i in range(10**8)]"
```
This will spike CPU — SysGuard will detect, declare the incident, and generate an RCA.

**6. Stop with Ctrl+C** — RCA auto-generates if an incident is active.

---

## Configurable Thresholds

Open `monitor.py` and adjust at the top:

```python
CPU_THRESHOLD = 85       # Alert if CPU exceeds 85%
RAM_THRESHOLD = 80       # Alert if RAM exceeds 80%
DISK_THRESHOLD = 70      # Alert if Disk exceeds 70%
CHECK_INTERVAL = 5       # Seconds between each check
INCIDENT_COUNT = 3       # Consecutive breaches to declare incident
```

---

## Sample RCA Report Output

```
╔══════════════════════════════════════════════════════════╗
         INCIDENT ROOT CAUSE ANALYSIS REPORT
╚══════════════════════════════════════════════════════════╝

Project         : SysGuard — Automated System Health Monitor
Generated By    : Anwesha Sahoo
Incident ID     : INC-2026-06-11_13-28-32
Severity        : HIGH

1. INCIDENT SUMMARY
   RAM breached 80% threshold for 13 consecutive cycles (65 seconds)

2. INCIDENT TIMELINE
   13:27:18 — First anomaly detected
   13:27:18 — Alert triggered
   13:28:30 — Final anomaly reading
   13:28:32 — RCA auto-generated

3. ROOT CAUSE
   High RAM — possible memory leak or excessive application load

4. RESOLUTION STEPS
   1. ps aux --sort=-%mem
   2. Investigate top memory consumers
   3. Terminate or optimise offending process
   ...
```

---

## Advantages Over Basic Monitoring Scripts

| Feature | Basic Script | SysGuard |
|---|---|---|
| Monitors multiple metrics | Sometimes | ✅ CPU, RAM, Disk, Network |
| False positive reduction | ❌ Single threshold | ✅ Consecutive-breach logic |
| Incident declaration | ❌ | ✅ Automatic with severity |
| Auto RCA generation | ❌ | ✅ 8-section professional report |
| Structured logging | ❌ | ✅ Date-wise, level-tagged |
| Graceful shutdown handling | ❌ | ✅ Final RCA on exit |
| Zero external dependencies | ✅ | ✅ Only psutil needed |

---

## How This Maps to Real SRE Work

| SRE Responsibility | SysGuard Implementation |
|---|---|
| Write Python automation scripts | Core monitoring engine in Python |
| Monitor system health (infra + app) | CPU, RAM, Disk, Network tracked in real time |
| Incident management & response | Automatic incident declaration + logging |
| Root cause analysis | Auto-generated 8-section RCA report |
| Document troubleshooting steps | Resolution steps embedded in every RCA |
| Proactive issue detection | Anomaly detection before system failure |
| Reduce alert fatigue | Consecutive-breach logic eliminates false positives |

---

## Future Enhancements

- [ ] Slack / PagerDuty webhook integration for real-time alerts
- [ ] Email notification on incident declaration
- [ ] Prometheus metrics exporter endpoint
- [ ] Web dashboard using Flask + Chart.js
- [ ] Systemd service for running as background daemon
- [ ] Docker containerisation for cross-platform deployment
- [ ] Multi-host monitoring via SSH

---

## Author

**Anwesha Sahoo**
B.Tech — Computer Science (AI & Data Science), JUIT
[GitHub](https://github.com/anwesha78) • [LinkedIn](https://www.linkedin.com/in/anwesha-sahoo-bb94ab411/)

---

> *SysGuard was built to understand what SRE teams do every day — automate the boring, catch the critical, document everything.*
