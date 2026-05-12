# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workspace Purpose

This workspace is a **Sales & Marketing Agent Org** — a team of Claude Code subagents that simulate a full B2B revenue organization. The user interacts with the `cro` agent as the single entry point; everything else cascades from there.

---

## Agent Org Chart

```
CRO  ← entry point — talk to this agent directly
├── VP of Sales
│   ├── Sales Ops
│   ├── Sales Enablement
│   └── Junior Sales Manager
│       ├── SDR
│       │   ├── prospect-researcher  (specialist)
│       │   └── outreach-writer      (specialist)
│       ├── Account Executive
│       │   ├── meeting-prepper      (specialist)
│       │   └── pipeline-reporter    (specialist)
│       └── Account Manager
└── VP of Marketing
    ├── Demand Gen
    └── Content Marketing
```

---

## Agents — Roles & Trigger Phrases

### Leadership
| Agent | Role | Invoke when… |
|-------|------|-------------|
| `cro` | Chief Revenue Officer — top orchestrator | Any revenue goal, strategy, or cross-functional decision |
| `vp-sales` | VP of Sales — owns sales team and quota | Quota setting, forecast review, deal escalations, pipeline coverage |
| `vp-marketing` | VP of Marketing — owns pipeline from marketing | Campaign strategy, MQL quality, attribution, positioning |

### Sales Management
| Agent | Role | Invoke when… |
|-------|------|-------------|
| `junior-sales-manager` | Mid-level orchestrator — manages SDR, AE, AM | Working a lead through the cycle, team status updates, escalations |
| `sales-ops` | CRM hygiene, forecasting, metrics | Pipeline audits, weekly forecasts, win rate / deal velocity reports |
| `sales-enablement` | Battle cards, talk tracks, playbooks | Competitive intel tools, objection guides, rep training material |

### Sales Execution
| Agent | Role | Invoke when… |
|-------|------|-------------|
| `sdr` | Top-of-funnel — prospecting and outreach | New lead assigned, buying signal check, lead list building |
| `account-executive` | Deal management — first meeting to close | Meeting booked, deal stalling, proposal needed, pipeline review |
| `account-manager` | Post-close — renewals, upsell, churn | Deal closed won, renewal approaching, account gone quiet |

### Specialists (invoked by SDR and AE)
| Agent | Role |
|-------|------|
| `prospect-researcher` | Company firmographics, decision makers, buying signals |
| `outreach-writer` | Cold email + LinkedIn sequences (5–7 touches) |
| `meeting-prepper` | Pre-call briefs, BANT/MEDDIC scoring, discovery questions |
| `pipeline-reporter` | Pipeline status, stall flags, weighted revenue forecast |

### Marketing
| Agent | Role | Invoke when… |
|-------|------|-------------|
| `demand-gen` | Campaigns, MQL scoring, attribution | New campaign brief, MQL definition, channel mix recommendation |
| `content-marketing` | Collateral, nurture sequences, thought leadership | One-pagers, case studies, blog outlines, nurture emails |

---

## Context Directory Structure

All agent inputs and outputs live in `Context/`. This directory is the **shared state** of the org — agents communicate asynchronously by writing files here and reading what prior agents produced.

```
Context/
├── prospects/        — prospect-researcher output (company profiles)
├── outreach/         — outreach-writer output (email/LinkedIn sequences, proposals)
├── pipeline/         — deal records (input for pipeline-reporter and AE)
├── reports/          — all generated reports (pipeline, forecast, CRM audit, CRO weekly)
├── meeting-preps/    — meeting-prepper output (pre-call briefs)
├── accounts/         — account-manager records (post-close customer files)
├── playbooks/        — sales-ops and sales-enablement output (process docs, battle cards, talk tracks)
├── campaigns/        — demand-gen output (campaign briefs, attribution reports, content calendar)
└── content/          — content-marketing output (one-pagers, case studies, nurture sequences)
```

---

## How to Use

**Start here:** Address the `cro` agent with a high-level revenue goal:
> *"We need to hit €2m ARR this year — make it happen"*
> *"Give me the full revenue picture this week"*
> *"We're entering a new market — how do we go to market?"*

The CRO cascades down to VP Sales and VP Marketing, who delegate further to their teams.

**Invoke agents directly** for specific tasks:
> *"Research Anthropic as a prospect"* → `prospect-researcher`
> *"Write outreach for Anthropic"* → `outreach-writer`
> *"Prep me for my meeting with Salesforce on Thursday"* → `meeting-prepper`
> *"Generate this week's pipeline report"* → `pipeline-reporter`
> *"Build a battle card against HubSpot"* → `sales-enablement`
> *"Write a one-pager for our enterprise use case"* → `content-marketing`

---

## File Naming Conventions

| Type | Pattern |
|------|---------|
| Prospect profile | `Context/prospects/[company-name].md` |
| Outreach sequence | `Context/outreach/[company-name]-sequence.md` |
| Proposal | `Context/outreach/[company-name]-proposal.md` |
| Deal record | `Context/pipeline/[company-name].md` |
| Meeting brief | `Context/meeting-preps/[company-name]-[YYYY-MM-DD].md` |
| Account record | `Context/accounts/[company-name].md` |
| Pipeline report | `Context/reports/pipeline-report-[YYYY-MM-DD].md` |
| Forecast | `Context/reports/forecast-[YYYY-MM-DD].md` |
| CRO weekly | `Context/reports/cro-weekly-[YYYY-MM-DD].md` |
| Campaign brief | `Context/campaigns/brief-[campaign-name].md` |
| Battle card | `Context/playbooks/battle-card-[competitor].md` |
| Objection guide | `Context/playbooks/objection-guide-[topic].md` |
| Talk track | `Context/playbooks/talk-track-[scenario].md` |
| Content asset | `Context/content/[type]-[topic].md` |

---

## Agent Definition Format

All agents are defined in `.claude/agents/[name].md`. Each file has two parts:

**Frontmatter** — consumed by the Claude Code harness:
```yaml
---
name: agent-name          # must match the subagent_type used to invoke it
description: One sentence explaining when to pick this agent over others
tools:
  - Read
  - Write
  - WebSearch
  - WebFetch              # only for agents that fetch external pages
---
```

**System prompt** — the body below the frontmatter. Defines the agent's role, workflows, output format, and rules.

### Tool Access by Tier

| Tier | Agents | Tools |
|------|--------|-------|
| Leadership / Management | `cro`, `vp-sales`, `vp-marketing`, `junior-sales-manager`, `sales-ops`, `sales-enablement` | Read, Write, WebSearch |
| Execution (outbound) | `sdr`, `account-executive`, `account-manager`, `demand-gen`, `content-marketing` | Read, Write, WebSearch, WebFetch |
| Specialists | `prospect-researcher`, `outreach-writer`, `meeting-prepper`, `pipeline-reporter` | WebSearch, WebFetch, Read, Write (varies) |

Add `WebFetch` only to agents that need to fetch live web pages (external research). Management agents coordinate via files in `Context/` and don't need it.

---

## Inter-Agent Communication Pattern

Agents do not call each other directly at runtime. The pattern is:

1. **Caller** invokes a sub-agent via the `Agent` tool with `subagent_type` set to the agent's `name`.
2. **Sub-agent** does its work and **writes output to `Context/`**.
3. **Sub-agent** returns a short summary to its caller.
4. **Caller** reads the summary, optionally reads the file, and decides the next step.

This means every agent must produce a file artifact *and* a summary — files are the persistent record, summaries are the coordination signal.

### Handoff Protocol

When an agent completes its role in a lead's lifecycle (e.g. SDR → AE), the handing-off agent:
- Confirms the relevant `Context/` files exist and are complete
- Returns a handoff summary to the orchestrator (JSM or CRO) that includes file paths and a one-line status per file
- The receiving agent reads those files at the start of its next task — it does not re-derive context from scratch

---

## Adding a New Agent

1. Create `.claude/agents/[name].md` with the frontmatter and system prompt.
2. Give it the minimum tool set it needs (prefer fewer tools).
3. Define a clear `description` — the harness uses this to decide when to invoke it.
4. Assign it a slot in the org chart above and update the relevant manager agent's "Direct Reports" section so it knows to delegate to the new agent.
5. Add its output directory to `Context/` and its file pattern to the naming convention table.
