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

All agent inputs and outputs live in `Context/`:

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
