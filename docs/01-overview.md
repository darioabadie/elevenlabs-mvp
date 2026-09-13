# 1. Overview

## What is Renti IA?

Renti IA is a voice agent for **internal use by a property management agency** ("inmobiliaria") in Argentina that manages rental properties on behalf of owners. It is **not** tenant-facing — it's meant for the agency's own staff (an agent or administrator) to talk to instead of digging through spreadsheets, HubSpot, and a calendar app separately.

The agent speaks Rioplatense Spanish (Argentina/Uruguay dialect), professional but approachable tone, and is deliberately terse — voice conversations should not read like a text chat.

## Problem it solves

A property manager's day involves repeatedly answering the same kinds of questions and doing the same kinds of tasks:

- "How did this month close out financially?"
- "Which leases are about to expire and need renewal outreach?"
- "Is this rental candidate a credit risk?"
- "Can we schedule a viewing, and is the agent even free?"
- "Send the owner a summary by email."
- "How is each salesperson performing this month?"

Each of these normally means switching between a database/spreadsheet, a CRM, a calendar app, and email. Renti IA collapses that into a single voice conversation, backed by real (not mocked) data sources.

## Who it's for

- **Primary user**: the agency's internal staff — the person managing the property portfolio.
- **Not the tenant**: the agent never talks to renters directly; it's a productivity tool for the agency side.

## What makes this demo distinctive

1. **A real, live Argentina-specific government data source** (BCRA's Central de Deudores) — this is the "wow" moment of the demo: a live credit check on a real CUIT/CUIL, not a mocked API response.
2. **A mix of integration patterns**: custom webhook tools against a bespoke backend (Supabase), a public unauthenticated government API (BCRA), a native OAuth integration (Google Calendar), and a third-party CRM (HubSpot) with both native connector tools and a custom webhook tool.
3. **Realistic supporting data**: rather than a toy dataset, the CRM was seeded with a plausible set of contacts and deals across 5 simulated salespeople, so the "sales performance" query returns varied, believable numbers instead of a flat/empty result.
4. **Deliberate security hygiene** during setup: least-privilege API scopes, temporary elevation only when needed (with immediate revocation), secrets never handled in plaintext by the assistant configuring the agent, and local credential files deleted after use. See [10-security-and-secrets.md](10-security-and-secrets.md).

## High-level capabilities (tools)

| Capability | Backed by |
|---|---|
| Monthly financial summary | Supabase (`voice-agent-api`) |
| Expiring leases | Supabase (`voice-agent-api`) |
| Upcoming rent index updates | Supabase (`voice-agent-api`) |
| Tenant credit check by CUIT/CUIL | BCRA public API |
| Schedule a property viewing | Google Calendar (native integration) |
| Email a report/summary | Supabase (`voice-agent-api`) → Resend |
| Salesperson performance | HubSpot CRM |

Full technical reference for each tool: [09-tools-reference.md](09-tools-reference.md).

For the exact conversational flow used to demonstrate all of this end-to-end, see [11-demo-script.md](11-demo-script.md).
