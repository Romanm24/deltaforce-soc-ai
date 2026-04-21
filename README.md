# Delta Force AI SOC Platform

> AI-powered Security Operations Center with automated threat detection, MITRE ATT&CK mapping, and real-time Discord alerting.

---

## What This Does

Delta Force SOC automatically ingests security logs, analyzes them using Claude AI, maps threats to the MITRE ATT&CK framework, and delivers structured incident reports to your team in real time.

**This is not a demo. This is a working pipeline.**

---

## Live Pipeline

Log Ingestion → Claude AI Analysis → MITRE Mapping → Escalation Logic → Discord Alert

---

## Sample Alert Output

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DELTA FORCE SOC — INCIDENT REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ID:           DF-1745256832941
TIMESTAMP:    2026-04-21T16:45:51Z
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SEVERITY:     MEDIUM | CONFIDENCE: High
ATTACK:       Brute Force / Credential Attack
KILL CHAIN:   Reconnaissance/Weaponization
ESCALATE:     Yes
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SOURCE IP:    192.168.0.239
PROTOCOL:     SSH2
ASSET:        SSH service on port 22
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MITRE TACTIC:     Initial Access
MITRE TECHNIQUE:  T1110.001 - Brute Force
FALSE POSITIVE:   Low
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DETECTION ENGINE:
Pipeline: Schedule Trigger > File Ingest > Claude AI > Parser > Discord

---

## Tech Stack

| Layer | Tool |
|-------|------|
| AI Analysis | Claude AI (Anthropic) |
| Automation | n8n workflows |
| Alerting | Discord webhooks |
| Agent System | VS Code + Claude Code |
| Version Control | GitHub |
| Languages | JavaScript, Python |

---

## Key Capabilities

- AI-powered threat analysis with severity scoring
- MITRE ATT&CK tactic and technique mapping
- Kill chain phase identification
- Confidence scoring and false positive assessment
- Automated escalation based on attempt count
- Real-time Discord alerts with full incident reports
- Unique workflow ID per incident for tracking

---

## Deployment

Currently deployed in a sandboxed cyber lab environment. Pilot campus deployment in progress.

---

## Project Status

- [x] Core AI pipeline
- [x] Discord alerting
- [x] MITRE ATT&CK mapping
- [x] Escalation logic
- [x] GitHub documentation
- [ ] Campus pilot deployment
- [ ] Dashboard UI
- [ ] Multi-source log ingestion

---

## Built By

Roman Mares — Cybersecurity student, SOC engineer, AI automation builder.

*Currently seeking pilot deployments at educational institutions and small businesses.*

---

**License:** Proprietary — Delta Force AI SOC Platform
