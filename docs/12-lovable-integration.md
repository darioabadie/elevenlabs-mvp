# 12. Lovable / App Integration

## What it does

Embeds the same Renti IA voice agent — the one used from the ElevenLabs dashboard for the golden demo path — directly inside the agency's own web app (**Property Flow**, built on Lovable), behind login, on a new **"Hablar con RentIA"** button on the `/app/asistente` route. This gives the agency a real, always-available in-product entry point to the agent, independent of the ElevenLabs dashboard.

## Why this was worth doing for the demo

The take-home brief mentions embedding the agent into an existing application. Property Flow already existed as the agency's real app (Supabase-backed, same backend used by `voice-agent-api`), so wiring the finished agent into it turns the interview demo from "an agent configured in a dashboard" into "an agent living inside a real product a salesperson would actually open."

## How it was done: ElevenLabs' native "Add to your app" feature

ElevenLabs Agents has a built-in **"Add to your app"** menu (via the "..." next to the agent's name in the dashboard) with per-framework buttons — Lovable, Claude Code, Blackbox, Codex CLI, Cursor, among others. Selecting **Lovable** auto-generates a tailored deployment prompt (written to match this specific agent's ID, tools, and auth mode) and copies it to the clipboard, ready to paste directly into Lovable's own chat.

Two integration approaches were available:

1. **Method 1 (used)**: paste ElevenLabs' auto-generated deployment prompt directly into Lovable's chat and let Lovable's own AI implement the integration end-to-end.
2. **Method 2 (not used)**: hand-write the integration — a manual `ConversationProvider`/`useConversation` setup, a Supabase Edge Function to mint a signed URL or conversation token, and a new UI component — for finer control over exactly how it's wired in.

Method 1 was chosen and executed by the project owner directly in Lovable's chat. It worked end-to-end on the first real attempt (with one runtime bug surfaced and fixed along the way — see below).

## Why a server-minted token is required (not just the Agent ID)

The agent is configured as **Private** (auth-enabled) in ElevenLabs, which is the correct setting for an agent embedded in a real product — it must not be startable by anyone who finds the Agent ID in the client-side bundle. For a Private agent, the client cannot start a session with just the Agent ID; the server must first mint a short-lived credential:

- **WebSocket flow**: `GET https://api.elevenlabs.io/v1/convai/conversation/get-signed-url?agent_id=...`
- **WebRTC flow (recommended, used here)**: `GET https://api.elevenlabs.io/v1/convai/conversation/token?agent_id=...`

Both calls require the `xi-api-key` header (the ElevenLabs API key) and must happen server-side, never in the browser. The client then calls `conversation.startSession({ conversationToken: token, connectionType: "webrtc" })` via the `@elevenlabs/react` SDK's `useConversation` hook.

## What Lovable actually built

Reading back through the real Lovable chat session (not just the original plan) confirms the following was implemented:

- **API key storage**: Lovable used its own native **Connectors** feature to store the ElevenLabs API key securely on Lovable's side. This is a different (but security-equivalent) mechanism from the original plan, which assumed a manually-created Supabase environment variable — the same pattern already used for Resend and HubSpot secrets in this project. Lovable's Connectors card achieves the same goal (the key is never exposed to the browser) without a manual env-var step.
- **New server function**: `elevenlabs-conversation-token` — validates that the caller is a logged-in Property Flow user, then calls ElevenLabs' token endpoint server-side and returns a single-use conversation token to the client. This mirrors the existing `x-agent-secret`-gated pattern used by `voice-agent-api`, but authenticates via the app's own login session instead of a static header.
- **New UI**: a **"Hablar con RentIA"** button on `/app/asistente`, gated behind login, using `@elevenlabs/react`'s `useConversation` hook. Includes a real-time speaking/listening indicator and a hang-up control.

## Bug found and fixed during the integration

Immediately after the first deploy, opening `/app/asistente` threw:

```
Unexpected Application Error!
useRegisterCallbacks must be used within a ConversationProvider
```

**Root cause**: the version of `@elevenlabs/react` in use requires any component calling `useConversation` (which internally uses `useRegisterCallbacks`) to be wrapped in a `ConversationProvider` React context provider — a requirement not obvious from a first reading of the SDK's hook API, and easy to miss when a single button component is added directly to a page without an explicit provider tree.

**Fix**: Lovable's own AI diagnosed the error, wrapped the voice-agent button component in `ConversationProvider`, verified the type-check passed, and republished. Confirmed working immediately after by the project owner in a live conversation from `/app/asistente`.

**Takeaway for anyone repeating this integration**: when using `@elevenlabs/react`'s `useConversation` hook, always wrap the consuming component tree in `ConversationProvider` from the start — this saves the one extra fix-and-republish cycle hit here.

## Verified outcome

Confirmed working end-to-end by the project owner: talking to the agent from inside Property Flow, at `/app/asistente`, behind login, with no dependency on having the ElevenLabs dashboard open. The golden demo path (see [11-demo-script.md](11-demo-script.md)) can now be run either from the ElevenLabs dashboard's test-call UI or from inside the app itself — the same agent, same tools, same backend, two entry points.

## What stayed unchanged

Consistent with the standing constraint followed throughout this project: the agent's LLM (Qwen3.5-397B-A17B) and its ElevenLabs dashboard configuration language (English) were not touched as part of this integration. This was purely a client-embedding change — no agent configuration, tools, or prompts were modified.
