# 4. Email Integration — Resend

## What it does

The `enviar_resumen_email` tool lets the agent send a report or summary by email — e.g. "send the owner a summary of this property." It's implemented as the `enviar_reporte` action inside [`voice-agent-api`](03-supabase-backend.md), which internally calls the [Resend](https://resend.com) API. The Resend API key is a Supabase environment variable and is **never** exposed to ElevenLabs — the agent only ever talks to `voice-agent-api`, which does the actual sending server-side.

## Two options considered

Before implementing, two approaches were weighed:

- **Option A — Resend via the existing backend** (chosen): reuse the same integration pattern already used by the `enviar-contacto` function in the web app. Fastest to implement, keeps the API key server-side only.
- **Option B — Gmail via Zapier/Make**: a webhook-triggered Zap calling the Gmail "Send Email" action. Would give a "real Gmail" sending experience without writing backend code, at the cost of an extra third-party dependency.

Resend was chosen for speed and to keep the credential surface smaller.

## Bug found: sandbox domain restriction

The first end-to-end test of `enviar_resumen_email` (via ElevenLabs' "Test Tool") failed with a generic `{"error":"Failed to send email"}`. The request from ElevenLabs was confirmed well-formed (checked via the tool's "Raw Request" view: correct `accion`, `params`, and `x-agent-secret` header) — so the failure was happening inside the call to Resend, and the actual Resend error was hidden from the client by design (see [03-supabase-backend.md](03-supabase-backend.md)).

**Root cause**: the `from` address was `onboarding@resend.dev`, Resend's sandbox/test domain. Resend restricts sandbox-domain senders to only deliver to the email address the Resend account itself is registered with — any other recipient is silently rejected.

## Fix: verifying a real sending domain

Two verified-domain options were available:
1. `deployr.ai` — already verified in the connected Resend account.
2. `renti.com.ar` — the fictional agency's own domain, not yet verified.

**Decision**: verify `renti.com.ar` instead of reusing `deployr.ai`, specifically because it makes the demo more coherent — emails arriving from a `renti.com.ar` address look like they're actually coming from the (fictional) agency, rather than from an unrelated engineering domain.

Steps taken:
1. Added `renti.com.ar` as a domain in the Resend dashboard.
2. Published the required DNS records (SPF/DKIM/etc.) and waited for verification.
3. Updated the `from` address used by both `enviar_reporte` (voice agent) and `enviar-contacto` (web app) to send from a `renti.com.ar` address instead of `onboarding@resend.dev`.
4. Re-tested end-to-end from a live ElevenLabs conversation — confirmed the email was received successfully by an arbitrary recipient (no longer restricted to the Resend account's own address).

## Current status

✅ Fully working end-to-end, verified with a real test send. No known open issues.

## Takeaway for anyone extending this

If you fork this project and email sending starts failing with a generic error after a domain/DNS change, check the Resend dashboard's domain verification status first — that class of failure produces exactly this symptom (generic client error, real cause only visible in provider-side logs).
