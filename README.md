# BreachRoom

> An AI "War Room" for data breach incident response — built on [Band](https://www.band.ai/)
>
> Built for the [Band of Agents Hackathon](https://lablab.ai/ai-hackathons/band-of-agents-hackathon) — Track 3: Regulated & High-Stakes Workflows

---

## The Problem

When a B2B SaaS company (50–500 employees) discovers a customer data breach, the clock starts immediately — under GDPR Art. 33, regulators must be notified within **72 hours**. But most companies this size don't have a dedicated security team, in-house legal counsel, or a PR function on call at 2am on a Saturday.

Large enterprises convene a "war room" of specialists. Smaller companies panic, scramble, and often miss critical steps — or the deadline itself.

## What BreachRoom Does

BreachRoom is a multi-agent incident response system where **5 agents collaborate inside a shared Band room** to produce a first-response package — technical analysis, regulatory notification draft, customer communication, and a full audit trail — in minutes instead of days.

When the human incident commander joins (even hours later), the analysis, drafts, and decision log are already waiting for them.

## How Band Powers This (not a thin wrapper)

- All 5 agents live in **one Band room** and communicate via `@mention`.
- `IncidentTriage` classifies the incident, calculates the regulatory deadline, and **dynamically recruits** the agents it needs using `thenvoi_lookup_peers` + `thenvoi_add_participant` — not every incident pulls in every agent.
- Each agent shares structured findings in the room, building on context from agents that ran before it (task handoff).
- `IncidentRecorder` continuously builds a timestamped audit log of every decision made by every agent — this log is itself a product deliverable, used to demonstrate traceability to regulators after the fact.

## Agents

| Agent | Framework | Role |
|---|---|---|
| `IncidentTriage` | Claude Agent SDK | Classifies the incident, calculates the regulatory deadline, dynamically recruits specialists |
| `ForensicsAnalyst` | CrewAI | Analyzes scope/cause of exposure, proposes containment steps |
| `ComplianceCounsel` | LangGraph | Determines notification obligations (regulatory + contractual/SLA), drafts regulator filing |
| `CustomerComms` | Anthropic SDK | Drafts customer notification email + FAQ |
| `IncidentRecorder` | Pydantic AI | Builds the timeline, audit log, and final Executive Incident Report |

## Architecture Diagram

View the full agent architecture and Band collaboration flow on Figma (viewer access):
[BreachRoom Architecture — Figma](https://www.figma.com/board/d5CicDGLjXLk5LMxMVFnBk/BreachRoom?node-id=0-1&t=bBVWzWBiHv9Wnigs-1)

## Demo Scenario

A 120-person B2B SaaS company. 2am Saturday. A misconfigured S3 bucket exposed ~30,000 customer records (emails, hashed passwords, partial payment data) for 6 hours. The on-call security engineer — alone — posts the alert into the BreachRoom.

Within minutes:
1. `IncidentTriage` classifies the incident, calculates the 72-hour GDPR notification deadline, and recruits `ForensicsAnalyst` + `ComplianceCounsel`.
2. `ForensicsAnalyst` identifies the root cause and proposes immediate containment.
3. `ComplianceCounsel` determines notification obligations and recruits `CustomerComms`.
4. `CustomerComms` drafts the customer notification email and FAQ.
5. `IncidentRecorder` publishes an Executive Incident Report — timeline, decisions, rationale, time remaining before the deadline.
6. The human incident commander joins, already briefed.

See [`scenario/incident_report.md`](scenario/incident_report.md) for the exact input used in the demo.

## Tech Stack

- [Band](https://www.band.ai/) — agent coordination layer (rooms, `@mention`, dynamic recruitment)
- Claude Agent SDK, LangGraph, CrewAI, Anthropic SDK, Pydantic AI — one framework per agent, demonstrating cross-framework collaboration
- Python 3.11+

## Getting Started

```bash
# 1. Install dependencies
uv sync

# 2. Configure agents
cp agent_config.yaml.example agent_config.yaml
# Fill in the agent_id / api_key for each of the 5 agents
# (register agents at https://app.band.ai/agents)

# 3. Add LLM provider keys
cp .env.example .env

# 4. Run each agent (5 separate terminals)
python agents/incident_triage/main.py
python agents/forensics_analyst/main.py
python agents/compliance_counsel/main.py
python agents/customer_comms/main.py
python agents/incident_recorder/main.py

# 5. Create a "BreachRoom" room at app.band.ai and add all 5 agents
# 6. Paste the contents of scenario/incident_report.md into the room
```

## Why This Approach

Most compliance tooling reviews documents *before* anything happens. BreachRoom starts *the moment an incident is detected* — the regulatory clock is live, and the system decides in real time which experts the situation needs.

The same `IncidentTriage` → dynamic recruitment → `IncidentRecorder` pattern generalizes beyond data breaches: ransomware response (with sanctions-screening for ransom decisions), PCI-DSS payment data exposure, SLA-driven service outages, and third-party/vendor breach exposure all fit the same architecture with different specialist agents.

## License

MIT
