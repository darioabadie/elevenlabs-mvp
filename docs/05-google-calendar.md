# 5. Google Calendar Integration

## What it does

Lets the agent schedule property viewings directly on a real calendar, and — critically — check availability before doing so, rather than blindly creating overlapping events.

## Why a native integration, not a custom webhook

Google Calendar is available as a **native integration** in ElevenLabs' integrations marketplace, using standard OAuth against the demo agency's own Google account. There's no reason to hand-roll a webhook against the Calendar REST API (auth refresh tokens, scopes, etc.) when ElevenLabs already manages that OAuth lifecycle natively — so this integration was connected directly from the marketplace rather than built as a custom tool.

## Tools exposed to the agent

| Tool | Purpose |
|---|---|
| `check_availability` | Checks whether the connected calendar is free at a requested date/time before booking. |
| `create_event` | Creates the calendar event for the viewing, with the interested party's email added as an invitee. |

## Behavior enforced in the system prompt

The agent is explicitly instructed (see [08-elevenlabs-agent-config.md](08-elevenlabs-agent-config.md)) to **always call `check_availability` before confirming a visit** — this prevents the agent from verbally "confirming" a viewing time that's actually already booked, which would be a poor (and embarrassing, in a live demo) failure mode for a scheduling assistant.

## Setup requirement

Authorizing this integration requires interactively signing in with the demo agency's Google account through ElevenLabs' OAuth flow — this is a one-time manual step done directly in the ElevenLabs dashboard, not something scriptable via API.
