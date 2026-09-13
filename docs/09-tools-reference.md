# 9. Tools Reference

Full technical reference for every tool attached to the Renti IA agent. All 6 custom webhook tools are confirmed configured, saved, and (where noted) tested end-to-end.

## Custom webhook tools

| Tool | Method | Endpoint | Auth | Params | Status |
|---|---|---|---|---|---|
| `consultar_resumen_periodo` | POST | `voice-agent-api` (`accion: resumen_periodo`) | `x-agent-secret` header (secret) | `mes` (YYYY-MM) | ✅ Configured, HTTP 200 verified |
| `consultar_contratos_por_vencer` | POST | `voice-agent-api` (`accion: contratos_por_vencer`) | `x-agent-secret` header (secret) | `dias` (default 90) | ✅ Configured, HTTP 200 verified |
| `consultar_actualizaciones_proximas` | POST | `voice-agent-api` (`accion: actualizaciones_proximas`) | `x-agent-secret` header (secret) | `meses` (default 3) | ✅ Configured, HTTP 200 verified |
| `consultar_deudor_bcra` | GET | `https://api.bcra.gob.ar/CentralDeDeudores/v1.0/Deudas/{cuit}` | none (public API) | `cuit` (11 digits) | ✅ Configured |
| `enviar_resumen_email` | POST | `voice-agent-api` (`accion: enviar_reporte`) | `x-agent-secret` header (secret) | `destinatario`, `asunto`, `cuerpo` | ✅ Tested end-to-end, email delivered from `renti.com.ar` |
| `consultar_performance_vendedores` | GET | `https://api.hubapi.com/crm/v3/objects/deals?properties=dealname,amount,dealstage,vendedor,closedate&limit=100` | `Authorization: Bearer` header (secret, `HUBSPOT_API_KEY`) | none | ✅ Tested with "Test Tool" — returns real seeded deals with `vendedor` populated |

`voice-agent-api` base URL: `https://cgbkxjkhqscafdjuxitf.supabase.co/functions/v1/voice-agent-api`

## Native integrations

| Integration | Tools exposed | Auth | Status |
|---|---|---|---|
| Google Calendar | `check_availability`, `create_event` | OAuth (agency Google account) | ✅ Connected |
| HubSpot (native connector, `hubspot-renti`) | `hubspot_get_contact`, `hubspot_create_contact`, `hubspot_update_contact`, `hubspot_search_contacts`, `hubspot_get_company`, `hubspot_create_company`, `hubspot_search_companies`, `hubspot_get_contact_notes`, `hubspot_get_contact_tasks`, `hubspot_get_contact_companies`, `hubspot_get_contact_deals`, `hubspot_find_contact_by_email`, `hubspot_find_company_by_domain` | OAuth (portal `52023619`) | ✅ Connected; not used in the golden demo path, available for future use |

## System tools (ElevenLabs built-ins)

Only **2 active**, deliberately kept minimal:

- `End conversation`
- `Detect language`

(`Skip turn`, `Update state`, `Transfer to agent`, `Transfer to number`, `Play keypad touch tone`, `Voicemail detection` were left off — none are relevant to this agent's use case.)

## Why descriptions were rewritten in English

See [08-elevenlabs-agent-config.md](08-elevenlabs-agent-config.md#tool-descriptions--written-in-english-deliberately) — ambiguous/Spanish-language tool descriptions were causing tool-selection mistakes during configuration; English descriptions fixed this without affecting the agent's Spanish conversational behavior.
