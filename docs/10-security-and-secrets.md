# 10. Security and Secrets — Decisions Made During Setup

This project involved handling several real API credentials while being configured by an AI assistant (Claude, via browser automation) working alongside the project owner. A few deliberate practices were followed throughout, worth documenting explicitly since they shaped how the integrations were built.

## Secrets are never typed by the assistant into a secret-store field

When wiring the HubSpot service-key token into ElevenLabs (as the `HUBSPOT_API_KEY` workspace secret used by `consultar_performance_vendedores`), the browser-automation tooling's own safety classifier blocked the specific action of writing a value into ElevenLabs' "Add secret" field (category: `[Secret-Store Writes]`). This block was respected, not worked around: the assistant cancelled the in-progress dialog and asked the project owner to complete that one step themselves, with exact instructions for the secret's name and value. The owner did so, and the assistant verified afterward (via the "Secret created successfully" confirmation and the secret now being selected in the tool's header config) that it worked — without ever having seen or handled the raw token value being entered.

The same pattern applies to every credential in this project: secrets live in ElevenLabs workspace secrets or Supabase environment variables, referenced by name from tool configs, never inlined as plaintext.

## Least-privilege API scopes, with temporary elevation only when needed

The HubSpot service key used by `consultar_performance_vendedores` is scoped to **read-only** access at rest: `crm.objects.contacts.read`, `crm.objects.deals.read`, `crm.objects.owners.read`. This is all it needs for its actual job (querying deals from ElevenLabs at runtime).

During the one-off dummy-data seeding script (creating the custom `vendedor` property, 20 contacts, and 26 deals — see [06-hubspot-crm.md](06-hubspot-crm.md)), the script needed write scopes it didn't have (`crm.objects.contacts.write`, `crm.objects.deals.write`, `crm.schemas.deals.write`), which surfaced as an HTTP 403 `MISSING_SCOPES` error on the first run. Rather than leaving those write scopes on permanently "just in case," they were added right before re-running the script and **removed again immediately afterward** — the credential's standing privileges outside that one script execution stayed read-only.

## Local files containing plaintext secrets are deleted after use

The seeding script (`seed_hubspot.py`) contained the HubSpot service-key token in plaintext (necessary to make authenticated API calls from a local script). Once the script had run successfully and the data was verified in HubSpot, the file was deleted (`rm -f`) rather than left on disk.

## Error responses don't leak upstream provider detail

`voice-agent-api`'s `enviar_reporte` action logs the real Resend error server-side (for debugging) but returns only a generic `500 { "error": "Failed to send email" }` to the caller. This is a deliberate choice, made before it was known that a real bug (the sandbox-domain restriction, [04-email-integration.md](04-email-integration.md)) would need debugging through those very logs — the tradeoff (less self-service debuggability from the ElevenLabs side, but no leaking of a third-party provider's internal error messages to an LLM-driven caller) was kept even once it made that particular bug slightly more annoying to diagnose.

## Discarding an account rather than reusing one with pre-existing access

Configuration work initially had programmatic access to an existing HubSpot account through prior project tooling. Rather than keep using it, a **brand-new, dedicated HubSpot portal was created specifically for this demo**, and the old account's access was explicitly not used for any of the work described in this repo. This avoids ever touching real or unrelated account data while building a public/interview-facing demo.

## Model and language configuration were never touched

Not a credentials issue, but the same "don't drift beyond what's asked" principle: the agent's LLM (kept at ElevenLabs' default) and its dashboard configuration language (English) were an explicit standing constraint from early in the project, respected through every later change — see [08-elevenlabs-agent-config.md](08-elevenlabs-agent-config.md).

## Summary of the underlying principle

Across all of the above, the same pattern repeats: **prefer the smallest privilege and the smallest blast radius that gets the job done, elevate only for the specific action that needs it, and hand off anything that touches raw credentials to the human whenever automation would otherwise have to see or type them.**
