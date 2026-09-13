# 7. BCRA Integration — Tenant Credit Check

## What it does

Given a CUIT/CUIL (an 11-digit Argentine tax ID), looks up that person's or company's registered debt situation with Argentina's Central Bank (**Banco Central de la República Argentina**, BCRA) — outstanding debt, days past due, and a risk classification from 1 (best) to 5 (worst). This is used when the agency wants to evaluate a prospective tenant before signing a lease.

## Why this integration matters for the demo

This is the single most distinctive integration in the project: it's a **live call to a real Argentine government API**, not a mock. It demonstrates that the agent can reach outside its own backend to a domain-specific, country-specific public data source — something no generic "CRM + calendar + email" demo would show, and a natural fit for an agency that specifically wants to know: *is this person a credit risk?*

## Configuration

- **Method**: `GET`
- **URL pattern**: `https://api.bcra.gob.ar/CentralDeDeudores/v1.0/Deudas/{cuit}`
- **Auth**: none — the BCRA API is public.
- **Parameter**: `cuit` (string, 11 digits, no dashes).
- **Description given to the LLM**: "Consults Argentina's Central Bank Central de Deudores for a person's/company's current credit situation by CUIT/CUIL: debt amount, days past due, risk classification 1–5. Use this when asked to evaluate a prospective tenant."

## Behavior enforced in the system prompt

Before calling this tool, the agent is instructed to ask for the CUIT/CUIL explicitly (11 digits) rather than guessing or inferring it, and to explain the result in plain terms afterward — risk classification 1 is the best (lowest risk), 5 is the worst.

## Setup required

None beyond wiring the webhook tool — no API key, no account, no rate-limit concerns encountered during testing. This is the lowest-friction integration in the whole project.
