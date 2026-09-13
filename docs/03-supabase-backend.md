# 3. Supabase Backend — `voice-agent-api`

## Why a new endpoint

The existing web chat backend, `asistente-chat`, is built for the browser chat UI: it streams its response and authenticates with the logged-in user's JWT. Neither fits a voice-agent webhook tool, which needs a simple, synchronous request/response JSON call and has no browser session to authenticate with. Rather than bend `asistente-chat` to a use case it wasn't designed for, a new, minimal Edge Function was created: `supabase/functions/voice-agent-api/index.ts`.

- **Public URL**: `https://cgbkxjkhqscafdjuxitf.supabase.co/functions/v1/voice-agent-api`
- **Method**: `POST` only (`405` for anything else, `OPTIONS` handled for CORS)
- **Body contract**: `{ "accion": "<name>", "params": { ... } }`
- **Success response**: `{ "ok": true, "data": <result> }`
- **Unknown action**: `400 { "error": "Acción no reconocida: <accion>" }`

## Authentication

A single static header, `x-agent-secret`, is checked against the `AGENT_SHARED_SECRET` environment variable on the Supabase side. No match → `401 { "error": "No autorizado" }`. This same secret value is stored as an **ElevenLabs workspace secret** (referenced by the tool's `Authorization`-equivalent header, never typed in plaintext into a tool's URL or body).

The function uses the Supabase **service role key** internally (not the anon/user key), since there's no end-user session to scope the query to — the shared secret is what stands in for "this request is legitimately from our voice agent."

## Actions

| `accion` | `params` | What it does |
|---|---|---|
| `resumen_periodo` | `{ mes: "YYYY-MM" }` | Sums `precio_final`, `comision_inmo`, and `pago_prop` from the `historico` table for the given month. Financial summary: total billed, agency commission, owner payout. |
| `contratos_por_vencer` | `{ dias?: number, default 90 }` | Properties whose lease end date (`fecha_inicio_contrato + duracion_meses`) falls within the next N days. |
| `actualizaciones_proximas` | `{ meses?: number, default 3 }` | Properties whose next rent-index update (by frequency and index type — IPC/ICL) falls within the next N months. |
| `detalle_propiedad` | `{ id?: uuid, buscar?: string }` | Looks up a property by id, or free-text search across address/tenant/property name. |
| `listar_propiedades` | `{ buscar?: string, limit?: number, default 50 }` | Lists properties, optionally filtered by free text. |
| `enviar_reporte` | `{ destinatario: string, asunto: string, cuerpo: string }` | Sends an email via Resend. See [04-email-integration.md](04-email-integration.md) for the full story, including a bug that was found and fixed. |

All the query logic (except `enviar_reporte`) is ported directly from the equivalent tools already implemented in `asistente-chat/index.ts` — same table, same filtering logic — just stripped of the AI SDK's `tool()` wrapper and streaming, and re-exposed as a plain function call inside the `accion` switch.

## `enviar_reporte` — validation and error handling

This action went through a hardening pass after being audited:

1. **Per-field validation**: missing `destinatario` → `400 { "error": "Missing required field: destinatario" }`; missing `cuerpo` → the equivalent for that field.
2. **Email format validation** (regex) before calling Resend → `400 { "error": "Invalid email format" }`.
3. **Error detail is never leaked to the client.** Resend failures are logged server-side only (`console.error`); the client receives a generic `500 { "error": "Failed to send email" }`. This is deliberate — see [10-security-and-secrets.md](10-security-and-secrets.md) for the reasoning.
4. **Action logging**: every call logs `voice-agent-api action: <accion>` (never the secret) for observability.

## Known limitation worth noting

Because Resend error detail isn't surfaced to the client by design, debugging a delivery failure (see [04-email-integration.md](04-email-integration.md) for exactly this happening) requires access to the Supabase Edge Function logs. In this project that meant going through the person who has direct Supabase/Lovable access rather than the assistant configuring the ElevenLabs side — a reasonable tradeoff for demo purposes, but worth calling out as an operational consideration for production.
