# CLAUDE.md — Delta Force AI SOC Platform

## Project Identity
- **Project:** Delta Force AI SOC Platform
- **Company:** Delta Force Security LLC
- **Owner:** Roman Mares — Cybersecurity Engineer & AI SOC Builder
- **Email:** Roman.mares2012@gmail.com
- **Live Dashboard:** https://romanm24.github.io/deltaforce-soc-ai
- **Repos:** `deltaforce-soc-ai` (main platform), `soc-command-center` (agent system + automation)

---

## What This Project Is
This is a **working, production AI-powered Security Operations Center** — not a tutorial, not a demo.
The pipeline ingests real-time global threat intelligence, classifies threats using Claude AI,
maps them to MITRE ATT&CK, and fires automated alerts via Discord webhooks through n8n workflows.

---

## Tech Stack
| Layer | Tools |
|---|---|
| AI Analysis | Claude AI (Anthropic) — `claude-sonnet-4-6` |
| Automation | n8n workflows |
| Threat Map | D3.js v7 + TopoJSON |
| Alerting | Discord webhooks |
| Agents | VS Code + Claude Code |
| Languages | Python 3.10+, JavaScript (ES6+) |
| Env Management | `python-dotenv` |

---

## Coding Standards

### General
- All code must be **modular and reusable** — no monolithic scripts
- Every Python script must include **logging** by default (`import logging`)
- Use **environment variables** for all secrets (API keys, webhook URLs, tokens) — never hardcode
- Include **error handling** on all external API calls and webhook dispatches
- Add **docstrings** to all Python functions

### Python
- Python 3.10+ syntax
- Use `python-dotenv` for loading `.env` files
- Follow PEP8 formatting
- Prefer `requests` for HTTP calls; `aiohttp` for async pipelines
- Log at `INFO` level for normal flow, `ERROR` for failures, `DEBUG` for raw payloads

### JavaScript
- ES6+ syntax
- Use `async/await` — no raw `.then()` chains
- D3.js v7 for all visualization work
- Keep DOM manipulation separate from data logic

---

## Environment Variables
All secrets are loaded from `.env` — never hardcoded. Reference `.env.example` for the full list.

| Variable | Purpose |
|---|---|
| `ANTHROPIC_API_KEY` | Claude AI API access |
| `DISCORD_WEBHOOK_URL` | Discord alert channel |
| `N8N_WEBHOOK_SECRET` | Validates inbound n8n triggers |
| `THREAT_FEED_API_KEY` | Third-party threat feed auth |
| `LOG_LEVEL` | Logging verbosity (`INFO` / `DEBUG`) |

---

## Claude AI Usage in the Pipeline

### Model
- Default model: `claude-sonnet-4-6`
- Use `claude-opus-4-7` for deep threat correlation tasks where accuracy outweighs speed
- Enable **prompt caching** on the system prompt — the MITRE context block is static and eligible for caching

### Prompt Design
- System prompt must always include the MITRE ATT&CK context and required output format
- Keep prompts **deterministic** — set `temperature` to `0.2` or lower for classification tasks
- Always parse Claude's response for structured fields before passing downstream
- Use Claude for: threat summarization, MITRE classification, recommended action generation

### Prompt Caching
```python
messages = [
    {
        "role": "user",
        "content": [
            {
                "type": "text",
                "text": MITRE_SYSTEM_CONTEXT,
                "cache_control": {"type": "ephemeral"}  # cache the static MITRE block
            },
            {
                "type": "text",
                "text": threat_payload
            }
        ]
    }
]
```

---

## MITRE ATT&CK Standards
- All threat classifications **must** map to a MITRE ATT&CK Tactic and Technique
- Use official MITRE IDs (e.g., `T1566` for Phishing, `TA0001` for Initial Access)
- Reference: https://attack.mitre.org/
- Format for classification output:
  ```
  Tactic: <TA####> — <Tactic Name>
  Technique: <T####> — <Technique Name>
  Confidence: HIGH / MEDIUM / LOW
  ```

### Common Mappings Reference
| Threat Type | Tactic | Technique |
|---|---|---|
| Phishing | TA0001 — Initial Access | T1566 — Phishing |
| C2 Beacon | TA0011 — Command & Control | T1071 — App Layer Protocol |
| Credential Dump | TA0006 — Credential Access | T1003 — OS Credential Dumping |
| Lateral Movement | TA0008 — Lateral Movement | T1021 — Remote Services |
| Data Exfil | TA0010 — Exfiltration | T1041 — Exfil Over C2 Channel |

---

## Discord Webhook Payload Format
All alerts sent to Discord must follow this structure:
```json
{
  "embeds": [{
    "title": "[ALERT] <Threat Name>",
    "color": 16711680,
    "fields": [
      { "name": "Threat Level", "value": "CRITICAL / HIGH / MEDIUM / LOW" },
      { "name": "MITRE Tactic", "value": "TA#### — Tactic Name" },
      { "name": "MITRE Technique", "value": "T#### — Technique Name" },
      { "name": "Source IP", "value": "x.x.x.x" },
      { "name": "Timestamp", "value": "ISO 8601 UTC" },
      { "name": "Recommended Action", "value": "..." }
    ]
  }]
}
```

### Embed Colors by Severity
| Severity | Decimal Color |
|---|---|
| CRITICAL | `16711680` (red) |
| HIGH | `16744272` (orange) |
| MEDIUM | `16776960` (yellow) |
| LOW | `65280` (green) |

---

## n8n Workflow Design Guidelines
- Each workflow should do **one thing** — keep workflows single-purpose
- Always include an **error branch** on every node
- Use **Set nodes** to normalize data before passing between nodes
- Webhook triggers should validate incoming payloads before processing
- Name nodes clearly: `[ACTION] — [DESCRIPTION]` (e.g., `HTTP — Fetch Threat Feed`)

### Standard Workflow Pattern
```
[Trigger] → [Validate Payload] → [Set — Normalize] → [Claude — Classify] → [Set — Format Alert] → [Discord — Send Alert]
                                                                                                  ↓ (on error)
                                                                                        [Discord — Send Error]
```

---

## Threat Feed Sources
Ingest from public and commercial feeds — normalize all to a common schema before classification:

| Feed | Type | Module |
|---|---|---|
| AbuseIPDB | IP reputation | `pipeline/ingest.py` |
| AlienVault OTX | IOCs + pulses | `pipeline/ingest.py` |
| Shodan | Exposed services | `pipeline/ingest.py` |
| Custom feeds | Via n8n HTTP node | `workflows/` |

Normalized schema passed to Claude:
```python
{
    "source_ip": str,
    "destination_ip": str,
    "threat_type": str,
    "raw_indicator": str,
    "feed_source": str,
    "ingested_at": str  # ISO 8601 UTC
}
```

---

## File Structure (Reference)
```
deltaforce-soc-ai/
├── CLAUDE.md              ← You are here
├── .env                   ← Secrets (never commit)
├── .env.example           ← Template for env vars
├── requirements.txt
├── dashboard/             ← D3.js threat map frontend
│   ├── index.html
│   ├── map.js
│   └── styles.css
├── pipeline/              ← Core SOC pipeline
│   ├── ingest.py          ← Threat feed ingestion
│   ├── classify.py        ← Claude AI classification
│   ├── mitre.py           ← MITRE ATT&CK mapping
│   └── alert.py           ← Discord webhook dispatch
├── agents/                ← Automation agents
└── workflows/             ← n8n workflow exports (JSON)
```

---

## What Claude Code Should NOT Do
- Never hardcode API keys, tokens, or webhook URLs
- Never skip error handling on external calls
- Never create a single-file monolith — keep concerns separated
- Never use deprecated D3.js v4/v5 syntax
- Never commit `.env` files
- Never set `temperature` above `0.3` for threat classification prompts
- Never use `print()` for logging — always use the `logging` module

---

## Current Build Priorities
1. AI-powered SOC pipeline — Claude AI + n8n automation
2. Campus cyber lab pilot deployment
3. Real-time threat intelligence dashboard
4. Automated MITRE ATT&CK classification
