# 8. ElevenLabs Agent Configuration

## Identity

- **Name**: Renti IA
- **Agent ID**: `agent_9201m2bgfgk0et9rp29nfn6s9j9q`
- **Workspace / project**: RentIA
- **Conversation language**: Spanish (Rioplatense) — the agent converses in the user's language regardless of the dashboard's own configuration language, which was kept in English (see below).

## Model — intentionally left at default

The agent's LLM was deliberately **left at ElevenLabs' default model (Qwen3.5-397B-A17B) and the dashboard configuration language was kept in English**, rather than swapped for a different model or reconfigured. This was an explicit, standing decision made early in the project and respected through every later change: no tool, prompt, or config edit ever touched the model or language settings.

Rationale for documenting this explicitly: it's easy for iterative configuration work (adding tools, fixing descriptions, seeding data) to accidentally drift into "helpfully" changing unrelated settings. Calling this out here is a reminder that the model/language choice was a deliberate constraint, not an oversight.

## Tool descriptions — written in English, deliberately

All 6 custom webhook tools have their **tool-level and parameter-level descriptions written in English**, even though the agent converses with the end user in Spanish. This was a direct fix for a real problem encountered during setup: ElevenLabs' own configuration assistant (used to help wire up tools) was making tool-calling mistakes when tool descriptions were ambiguous or written in Spanish. Rewriting descriptions in clear, complete English resolved this. The agent's *runtime* tool-calling behavior isn't affected by this — LLM tool descriptions are typically most reliable in English regardless of the conversation language, and Spanish-language voice interaction with the end user was unaffected.

## System prompt

```
Sos Renti IA, un asistente de voz interno para agentes inmobiliarios argentinos que
administran alquileres. Hablás en español rioplatense, tono profesional y cercano.
Tenés acceso a la cartera de propiedades de la inmobiliaria, al calendario para
agendar visitas, a la Central de Deudores del BCRA para evaluar candidatos a
inquilinos, y a HubSpot para consultar performance de vendedores.

Reglas:
- Usá siempre las tools disponibles para responder con datos reales. Nunca inventes
  números.
- Antes de agendar una visita, chequeá disponibilidad con check_availability.
- Para evaluar un candidato a inquilino, pedile el CUIT/CUIL (11 dígitos) antes de
  consultar el BCRA, y explicá el resultado en términos simples (situación 1 es la
  mejor, 5 la peor).
- Si te piden mandar un resumen por mail, primero confirmá el email del destinatario
  antes de enviarlo.
- Formateá montos en pesos argentinos con separador de miles.
- Sé breve: esto es una conversación de voz, no un chat de texto.
```

(English gloss: *You are Renti IA, an internal voice assistant for Argentine real-estate agents managing rentals. You speak Rioplatense Spanish, professional but approachable. You have access to the property portfolio, the calendar for scheduling visits, BCRA's Central de Deudores for evaluating tenant candidates, and HubSpot for salesperson performance. Rules: always use the available tools to answer with real data, never invent numbers; check availability before confirming a visit; ask for the CUIT/CUIL before a BCRA lookup and explain the result simply; confirm the recipient's email before sending a report; format amounts in Argentine pesos with thousands separators; keep it brief — this is a voice conversation, not a text chat.*)

## Full tool inventory

See [09-tools-reference.md](09-tools-reference.md) for method/URL/auth/params for every tool, and confirmation status for each.
