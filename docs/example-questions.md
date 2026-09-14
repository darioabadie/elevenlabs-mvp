# Example questions for the demo

Ready-to-use questions for demoing the agent (or for practicing beforehand). Grouped by the capability they trigger, following the golden demo path in [11-demo-script.md](11-demo-script.md), plus a new set of questions the agent can now answer straight from the Knowledge Base.

## 1. Monthly financial summary (`consultar_resumen_periodo`)

- "Hi Renti IA, how's this month going?"
- "How did September close out?"
- "Give me a summary of August: total billed, commissions, and owner payouts."

## 2. Leases expiring soon (`consultar_contratos_por_vencer`)

- "Which leases expire in the next 60 days?"
- "Do I have any lease expiring this month?"
- "Show me the leases expiring in the next 90 days."

## 3. Upcoming rent adjustments (`consultar_actualizaciones_proximas`)

- "Which properties will have a rent adjustment in the next 3 months?"
- "What are the upcoming ICL adjustments?"

## 4. Evaluating a prospective tenant — BCRA (`consultar_deudor_bcra`)

- "I have a rental candidate, CUIT 20-12345678-9, how's their credit standing?"
- "Can you check with the BCRA if this CUIT has any outstanding debt? 27-98765432-1"
- "What's the situation of a tenant with CUIT 20-11223344-5?"

## 5. Scheduling — Google Calendar (`check_availability` + `create_event`)

- "Schedule a viewing for Thursday at 3pm with juan@example.com"
- "Do I have availability Friday morning to show a property?"
- "Set up a visit for tomorrow at 11am with the tenant, her email is maria@example.com"

## 6. Sending reports by email (`enviar_resumen_email`)

- "Send a summary of the property by email to maria@example.com"
- "Email me this month's summary at my own address"
- "Send the owner a summary of their payout by email"

## 7. Salesperson performance — HubSpot (`consultar_performance_vendedores`)

- "How are the salespeople doing this month?"
- "How is Martín Coria doing this month?"
- "Who's the salesperson with the most closed deals?"
- "Email me a performance summary for Lucía Fernández"

## 8. Questions about how Renti works — Knowledge Base

These no longer depend on any tool: the agent answers directly from what's loaded in the Knowledge Base (glossary, rent calculation, receipts, roles and permissions, ICL/IPC guides).

**Glossary and general concepts**
- "What is the ICL?"
- "What's the difference between IPC and ICL?"
- "What is a cycle?"
- "What is a property's historical record?"
- "What does it mean for a lease to have a fixed-percentage adjustment?"

**Rent calculation**
- "How is the agency's commission calculated?"
- "How is the final price the tenant pays put together?"
- "What happens if there's no published IPC value for a given month?"
- "If the rent is 300,000 pesos and there's a 5% increase from IPC, what does the adjusted price come out to?"
- "Do the deposit installments count toward the commission base?"

**Receipts**
- "What does the owner's receipt include?"
- "What's the difference between the tenant's receipt and the owner's receipt?"
- "What's the internal receipt used for?"
- "Why can the commission shown on the receipt change even though that month's historical record was already calculated?"

**Roles and permissions**
- "What can an operator do that a viewer can't?"
- "Who can delete a historical record or a cycle?"
- "Who can create a service account?"
- "What is the superadmin role?"

**Index guides (live content from renti.com.ar)**
- "What's this month's ICL value?"
- "What was July's inflation according to the IPC?"
- "Where can I calculate a lease adjustment by IPC or ICL?"
