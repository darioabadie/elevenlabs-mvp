# Renti IA — Voice Agent for Argentine Property Management (ElevenLabs Take-Home)

**Renti IA** is a Spanish-speaking voice agent built on **ElevenLabs Agents** for internal use by a property management agency in Argentina. It's a demo built for the ElevenLabs interview take-home challenge: an agent that a real estate agency's staff (not the tenant) can talk to, to get financial summaries, check expiring leases, evaluate a prospective tenant's credit history with a live government API, schedule visits, email reports, and check salesperson performance in the CRM.

This repository holds the full documentation of how the agent was configured, which integrations it connects to, the tradeoffs made along the way, and how to run the demo.

## Quick links

| Doc | What's in it |
|---|---|
| [docs/01-overview.md](docs/01-overview.md) | What the agent does, who it's for, and why |
| [docs/02-architecture.md](docs/02-architecture.md) | System diagram: ElevenLabs ↔ Supabase ↔ external APIs |
| [docs/03-supabase-backend.md](docs/03-supabase-backend.md) | The `voice-agent-api` Edge Function that backs most tools |
| [docs/04-email-integration.md](docs/04-email-integration.md) | Resend email integration, the sandbox-domain bug, and the fix |
| [docs/05-google-calendar.md](docs/05-google-calendar.md) | Native Google Calendar integration for scheduling visits |
| [docs/06-hubspot-crm.md](docs/06-hubspot-crm.md) | HubSpot CRM integration and the dummy sales-performance dataset |
| [docs/07-bcra-integration.md](docs/07-bcra-integration.md) | Live tenant credit-check via Argentina's Central Bank (BCRA) public API |
| [docs/08-elevenlabs-agent-config.md](docs/08-elevenlabs-agent-config.md) | Agent settings: model, language, system prompt |
| [docs/09-tools-reference.md](docs/09-tools-reference.md) | Full reference table of every tool: method, URL, auth, params |
| [docs/10-security-and-secrets.md](docs/10-security-and-secrets.md) | Secret handling, least-privilege scopes, and safety decisions made during setup |
| [docs/11-demo-script.md](docs/11-demo-script.md) | Golden demo path — the exact flow used to showcase the agent |
| [docs/12-lovable-integration.md](docs/12-lovable-integration.md) | Embedding the agent into the Property Flow app (Lovable), auth model, and a troubleshooting note |
| [CHANGELOG.md](CHANGELOG.md) | Chronological log of decisions and fixes made during setup |

## Why this stack

- **ElevenLabs Agents** as the voice orchestration layer (STT, LLM tool-calling, TTS) — no custom voice pipeline needed.
- **Supabase Edge Functions** as a thin backend that already existed for the web app (`property-flow`), reused here for a voice-specific endpoint.
- **A public government API (BCRA)** to bring a real, non-mocked Argentina-specific data source into the demo — something most other candidates are unlikely to showcase.
- **HubSpot** as the CRM for sales-performance reporting, seeded with a realistic dummy dataset.
- **Google Calendar** as a native OAuth integration for scheduling, with an availability check enforced before booking.
- **Lovable** to embed the finished agent directly into the agency's existing app (Property Flow), so the demo isn't confined to the ElevenLabs dashboard — see [docs/12-lovable-integration.md](docs/12-lovable-integration.md).

See [docs/01-overview.md](docs/01-overview.md) for the full picture.
