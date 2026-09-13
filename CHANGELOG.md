# Changelog / Decision Log

Chronological record of the decisions, fixes, and tradeoffs made while building Renti IA. This is meant as a companion to the topic-based docs in [`docs/`](docs/) — those explain *what* was built and *why* it's structured that way; this file captures *the order things happened in* and the specific problems hit along the way.

## Supabase backend

- Created `voice-agent-api`, a new Edge Function separate from the existing `asistente-chat`, exposing a simple `POST {accion, params}` contract authenticated with a static `x-agent-secret` header, ported from `asistente-chat`'s existing query logic for 5 read actions plus a new `enviar_reporte` email action.
- Confirmed the auth check works correctly (401 without the header).

## Email (Resend)

- Audited `enviar_reporte`: confirmed it was a real Resend integration (not a stub), and hardened it — per-field validation, email format validation, generic (non-leaking) error responses, action logging.
- First live test from ElevenLabs failed with a generic `Failed to send email`. Diagnosed as Resend's sandbox-domain (`onboarding@resend.dev`) restriction: it only delivers to the Resend account's own registered address.
- Decision: verify a real domain instead of continuing with the sandbox domain. Two verified-domain options existed (`deployr.ai`, already verified, vs. `renti.com.ar`, not yet verified); chose to verify `renti.com.ar` for a more coherent demo (emails appear to come from the fictional agency itself).
- Domain verified, `from` address updated, re-tested — confirmed working, email received successfully.

## HubSpot

- Decision: stop using the existing HubSpot account (already reachable via prior project tooling) and create a dedicated new HubSpot portal (`52023619`, "Renti") for this demo, to avoid touching unrelated account data.
- Since existing HubSpot tooling remained wired to the old account, configured the new account's data programmatically via direct HTTP calls (Python `urllib`) using a new least-privilege service key.
- Needed to simulate multiple salespeople for a meaningful "performance" query, but the new portal only had one real user. Considered inviting real HubSpot users (rejected — would send real invite emails to nonexistent people) vs. a custom `vendedor` deal property (chosen).
- Seeded the account: created the `vendedor` custom property (5 values), 20 dummy contacts, 26 dummy deals (weighted stage mix, round-robin salesperson assignment), and deal↔contact associations.
- Hit an HTTP 403 `MISSING_SCOPES` on the first seeding run — temporarily added the 3 needed write scopes, re-ran successfully, then removed the write scopes again immediately, returning the service key to read-only.
- Verified the seeded data via a read-only query grouped by `vendedor` — confirmed varied, realistic numbers across all 5 simulated salespeople.
- Configured `consultar_performance_vendedores` as a custom webhook tool pointing at the HubSpot deals API with the `vendedor` property included.
- Hit a safety block when trying to enter the HubSpot token as an ElevenLabs secret value directly (browser-automation classifier blocked `[Secret-Store Writes]`) — handed that one step to the project owner to complete manually, with exact instructions; verified success afterward without ever handling the raw token.
- The URL field in the tool's form was observed to reset after the secret sub-dialog closed — caught via screenshot before saving and re-entered.
- Tested the finished tool with ElevenLabs' "Test Tool" — confirmed `Success`, with real seeded deals returned including populated `vendedor` values. Confirmed the tool is attached and active on the agent (visible in the agent's tools list alongside the other 5 custom tools and the native HubSpot connector).
- Deleted the local seeding script (containing the plaintext token) after use.
- First real usage (a live voice conversation asking for a salesperson's performance, emailed as a report) surfaced a data-quality bug: `dealstage` was showing HubSpot's raw internal stage ID instead of a readable name, and the LLM's own "(Closed)" guess wasn't reliable. Fetched the real pipeline stage mapping from HubSpot's API and added it directly to the tool's description, also instructing the agent to only count the "won" stage as actual sales revenue rather than summing every deal regardless of stage.

## ElevenLabs agent configuration

- Standing constraint set early and respected throughout: never change the agent's LLM (left at the default, Qwen3.5-397B-A17B) or its dashboard configuration language (kept in English).
- Fixed the 5 Supabase-backed custom tools' request bodies to the correct `{accion, params}` shape, verified with real HTTP 200 responses (not just the UI's save confirmation).
- Rewrote all custom-tool descriptions in English (tool-level and per-parameter) after observing that ambiguous/Spanish-language descriptions were causing ElevenLabs' own configuration assistant to make tool-calling mistakes.

## Lovable / app integration

- Decision: embed the finished agent directly into the agency's existing app (Property Flow, built on Lovable) rather than leaving the demo confined to the ElevenLabs dashboard.
- Used ElevenLabs' native "Add to your app → Lovable" feature: generates a tailored deployment prompt for this specific agent (ID, tools, auth mode) and copies it to the clipboard for pasting into Lovable's chat.
- Two approaches were considered: Method 1 (paste the auto-generated prompt into Lovable's chat, let Lovable's AI implement it) vs. Method 2 (hand-write the `ConversationProvider`/`useConversation` wiring and a token-minting server function manually). Method 1 was used.
- Because the agent is Private (auth-enabled), embedding required a server-minted credential — the client cannot start a session with just the Agent ID. Used the WebRTC flow (`GET /v1/convai/conversation/token`) rather than the WebSocket signed-URL flow, called server-side only.
- Lovable implemented the integration end-to-end: stored the ElevenLabs API key via its native Connectors feature (rather than a manually-created Supabase env var, as originally planned — a different but security-equivalent mechanism), added a new `elevenlabs-conversation-token` server function gated on the app's own login session, and added a "Hablar con RentIA" button with a live speaking/listening indicator to `/app/asistente`.
- Hit a runtime bug immediately after first deploy: `useRegisterCallbacks must be used within a ConversationProvider`, thrown by `@elevenlabs/react`'s `useConversation` hook when its consuming component isn't wrapped in a `ConversationProvider`. Lovable diagnosed and fixed it by adding the wrapper, then republished.
- Verified working end-to-end in a live conversation from inside the app, independent of the ElevenLabs dashboard. Agent LLM and dashboard configuration language were not touched as part of this work, consistent with the project's standing constraint.

## Documentation

- This repository's `docs/` structure and `CHANGELOG.md` were written to capture the above for the demo, in English, to be legible to reviewers unfamiliar with the project's day-to-day Spanish-language working notes.
- Added `docs/12-lovable-integration.md` documenting the app-embedding work, and updated `README.md`'s Quick Links and `docs/11-demo-script.md` to point to it.
