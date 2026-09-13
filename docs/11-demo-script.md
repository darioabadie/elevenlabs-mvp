# 11. Golden Demo Path

The exact conversational flow used to showcase the agent end-to-end, touching every integration.

## Flow

1. **"Hola Renti IA, ¿cómo viene el mes?"** ("How's this month going?")
   → `consultar_resumen_periodo` — pulls a real financial summary (total billed, commission, owner payout) for the current month.

2. **"¿Qué contratos vencen en los próximos 60 días?"** ("Which leases expire in the next 60 days?")
   → `consultar_contratos_por_vencer` — lists real properties with upcoming lease expirations.

3. **"Tengo un candidato para alquilar, CUIT 20-XXXXXXXX-X, ¿cómo está su situación crediticia?"** ("I have a rental candidate, CUIT ..., what's their credit situation?")
   → `consultar_deudor_bcra` — **the standout moment of the demo**: a live lookup against Argentina's Central Bank, not a mocked response.

4. **"Agendale una visita para el jueves a las 15 con [email]"** ("Schedule a viewing for Thursday at 3pm with [email]")
   → Google Calendar `check_availability`, then `create_event` — demonstrates the agent checking before committing, not just blindly booking.

5. **"Mandale un resumen de la propiedad por mail a [email]"** ("Send a summary of the property by email to [email]")
   → `enviar_resumen_email` — a real email, delivered from `renti.com.ar`, confirmed working end-to-end (see [04-email-integration.md](04-email-integration.md)).

6. **"¿Cómo vienen los vendedores este mes?"** ("How are the salespeople doing this month?")
   → `consultar_performance_vendedores` — queries HubSpot deals grouped by the custom `vendedor` property, returning varied real numbers across 5 simulated salespeople (see [06-hubspot-crm.md](06-hubspot-crm.md)).

## What this sequence demonstrates

- **Varied tool-use patterns**: a custom backend webhook, a public unauthenticated government API, a native OAuth integration, and a CRM query — all in one conversation.
- **Real data, not mocked**: every number spoken by the agent traces back to an actual query against a real system (Supabase, BCRA, Google Calendar, HubSpot).
- **A country-specific integration (BCRA)** that's unlikely to appear in another candidate's demo, and that maps directly onto a real, valuable use case for an Argentine real-estate agency (tenant credit risk).
- **Guardrails in action**: the agent checks calendar availability before confirming a booking, and confirms the recipient's email before sending — both enforced by the system prompt, not left to chance.

## Status

All 6 steps are demoable end-to-end as of this writing. See [09-tools-reference.md](09-tools-reference.md) for the per-tool verification status.
