# 6. HubSpot CRM Integration

## What it does

Lets the agent answer "how are salespeople performing this period?" by querying deal data in HubSpot — amount, stage, close date, and which salesperson owns each deal.

## Two parallel integration paths

The agent has **two independent HubSpot integrations** attached:

1. **Native HubSpot connector** (OAuth, `hubspot-renti`): standard CRUD tools for contacts and companies (`hubspot_get_contact`, `hubspot_create_contact`, `hubspot_search_contacts`, `hubspot_get_company`, etc.). Connected via ElevenLabs' integrations marketplace. Not used in the golden demo path, but available for future contact/company-management use cases.
2. **Custom webhook tool** (`consultar_performance_vendedores`): a direct `GET` call to the HubSpot deals API, used specifically for sales-performance reporting. This exists as a separate custom tool because the native connector's tools don't support filtering/reading by an arbitrary **custom** deal property, which the performance use case needed (see below).

## Switching to a dedicated demo account

Configuration started on an existing HubSpot account (already connected to project tooling from earlier work), but the decision was made to **discard that account and create a brand-new, dedicated HubSpot portal for this demo** — a cleaner story for an interview demo than reusing an unrelated account, and it avoids any risk of touching real data.

- **New portal ID**: `52023619` (workspace name: "Renti")
- **Pipeline**: HubSpot's default "Pipeline de compradores" (buyer-qualification template) — its stages already fit a rental/real-estate flow well enough to use as-is, no custom pipeline needed.

## Simulating multiple salespeople

The new portal only has **one real HubSpot user** (its owner), which isn't enough to demo a "compare salespeople" query. Two options were considered:

- **Invite real HubSpot users** for each simulated salesperson — rejected, since this would send real email invitations to people who don't exist for this demo.
- **A custom `vendedor` (salesperson) property on the Deal object** (chosen) — an enumeration/select property with 5 fictional salesperson names, assigned round-robin across deals. No emails sent, no real user accounts needed, and it's exactly the kind of "custom property" pattern a real agency might actually use if salespeople aren't 1:1 with HubSpot seats.

Salespeople: Lucía Fernández, Martín Coria, Valentina Suárez, Nicolás Peralta, Camila Ortiz.

## Seeding dummy data

A one-off Python script (using `urllib`, no extra dependencies) called the HubSpot REST API v3 directly to seed:

- The custom `vendedor` deal property (enumeration, 5 options).
- **20 contacts** (name, email, phone, `lifecyclestage: lead`).
- **26 deals** (name, amount, stage — a weighted mix of won/lost/open, close date, and `vendedor` assigned round-robin).
- Deal↔contact associations for every deal.

The resulting dataset was verified with a read-only query grouped by `vendedor`, confirming each of the 5 salespeople shows varied, plausible numbers — not a flat or empty distribution.

The script and the resulting HubSpot state are the only record of this data — the script itself was deleted after running, per the credential-hygiene practice described in [10-security-and-secrets.md](10-security-and-secrets.md).

## The `consultar_performance_vendedores` tool

- **Method**: `GET`
- **URL**: `https://api.hubapi.com/crm/v3/objects/deals?properties=dealname,amount,dealstage,vendedor,closedate&limit=100`
- **Auth**: `Authorization: Bearer <token>` header, referencing an ElevenLabs workspace secret (`HUBSPOT_API_KEY`) rather than a hardcoded value.
- **Description given to the LLM**: "Fetches HubSpot deals with amount, deal stage, salesperson (vendedor) and close date, to report sales performance per salesperson. Use this when the user asks how salespeople are performing this period."

### API credential used

A HubSpot **service key** (the modern, scoped alternative to legacy Private Apps) was created specifically for this tool:

- Name: "Renti IA - Agente de voz"
- **Permanent scopes** (least privilege, read-only): `crm.objects.contacts.read`, `crm.objects.deals.read`, `crm.objects.owners.read`.
- **Temporarily elevated** during the seeding script only: `crm.objects.contacts.write`, `crm.objects.deals.write`, `crm.schemas.deals.write` — added right before running the script, removed again immediately after. See [10-security-and-secrets.md](10-security-and-secrets.md) for why.

### Verification

Tested with ElevenLabs' built-in "Test Tool" — returned `Success`, with real deals from the new portal correctly showing the `vendedor` field (e.g. deal "Depto 2 amb. Villa Urquiza - deal 11" → `vendedor: "Lucía Fernández"`). Confirmed attached and active on the agent's tool list.

## Known documentation drift, now corrected

An earlier draft of the tool spec referenced `hubspot_owner_id` (i.e., filtering by the real HubSpot owner) instead of the custom `vendedor` property. That was the original plan before the "one real owner" limitation was discovered — the final implementation uses `vendedor`, and all docs here reflect that.
