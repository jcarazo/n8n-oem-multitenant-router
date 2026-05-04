# n8n OEM Multi-Tenant Event Router

A production-pattern workflow architecture demonstrating how an OEM platform 
can embed n8n to serve multiple tenants from a single workflow instance — 
with per-tenant configuration, PII compliance, and modular service execution.

Built as a portfolio demonstration of n8n architecture patterns relevant to 
embedded and OEM deployment contexts.

---

## The Problem

When a SaaS company embeds n8n into their product, they typically serve many 
tenants from a single instance. Each tenant has different downstream systems, 
compliance requirements, and business logic. Naive approaches — one workflow 
per tenant, or a single workflow with hardcoded branching — don't scale.

This architecture solves that with a config-driven execution model: one 
workflow, any number of tenants, zero structural changes to add a new one.

---

## Architecture

Webhook → Tenant Config Lookup → PII Gate → [Service Gates] → Results

Each service gate follows the same pattern:
1. Call `oem-activation-gate` sub-workflow with the service name
2. If active: call the service sub-workflow
3. If inactive: skip and pass data forward
4. Merge and continue to next service

Adding a new service means adding a new sub-workflow and one gate block 
in the main workflow. Tenant configs live in one place.

---

## v2 — Optimized Architecture (branch: v2-loop-orchestrator)

After implementing the service chain pattern, a clear scaling problem emerges: 
each new service adds 5 nodes to the main workflow. At scale this becomes 
unmaintainable.

v2 refactors to a loop-based orchestration pattern:
- A single array defines all active services
- A Loop Over Items node iterates over them
- A central orchestrator sub-workflow handles gate + dispatch per service
- Main workflow structure never changes when adding new services

The deliberate decision to show both versions reflects how real platform 
architecture evolves — working first, then optimized for scale.

---

## Workflows

| File | Purpose |
|---|---|
| `oem-event-router.json` | Main workflow — entry point for all tenant events |
| `oem-activation-gate.json` | Reusable sub-workflow — config-driven activation check |
| `oem-action-email.json` | Email service sub-workflow |
| `oem-action-crm.json` | CRM service sub-workflow |

---

## Demo Tenants

Three tenants are pre-configured to demonstrate different profiles:

**Tenant A — E-commerce SMB**
Email + sheet logging enabled. Simple linear execution, 
continue-on-error mode. Standard data handling.

**Tenant B — B2B SaaS**
Email + CRM + Slack enabled. Event-to-record-type mapping 
for Salesforce. Block-on-error mode for data integrity.

**Tenant C — Healthcare Provider**
Email + audit enabled. PII masking fires before any service 
executes. EU data residency flag. Hard stop on any failure.

---

## Tenant Configuration

Each tenant is defined by a config object in the Tenant Config Lookup node.
Adding Tenant D requires adding one config entry — no workflow changes.

```json
{
  "tenant_id": "tenant_x",
  "error_mode": "continue | block",
  "data_handling": "standard | pii_masked",
  "email": { "enabled": true, "from_name": "...", "templates": {} },
  "sheet_log": { "enabled": false },
  "slack": { "enabled": false },
  "crm": { "enabled": true, "record_type": "Lead" },
  "audit": { "enabled": false }
}
```

In production this config would live in a database or Airtable, 
queried dynamically by tenant_id on each execution.

---

## Service Stubs

Email and CRM write nodes are stubs — Code nodes simulating the 
actual provider call. Each is documented with the native n8n node 
that replaces it in production:

- **Email** → Gmail, SendGrid, Send Email (SMTP), Mailchimp
- **CRM** → Salesforce, HubSpot, Pipedrive, Zoho CRM

Credentials are configured at platform level in n8n's credential 
store — never exposed to or configurable by tenants.

---

## How to Import

1. Open your n8n instance
2. Go to Workflows → Import
3. Import in this order:
   - `oem-activation-gate.json`
   - `oem-action-email.json`
   - `oem-action-crm.json`
   - `oem-event-router.json`
4. In the main workflow, update Execute Workflow nodes to reference 
   the correct imported workflow IDs

---

## Testing

A manual trigger and Mock Payload node are included in the main workflow.
Swap the payload to test different tenants and event types:

**Tenant A — Cart Abandoned**
```json
{ "body": { "tenant_id": "tenant_a", "event_type": "cart_abandoned",
"payload": { "customer_email": "anna@shop.com", "cart_value": 89.99 }}}
```

**Tenant B — Trial Signup**
```json
{ "body": { "tenant_id": "tenant_b", "event_type": "trial_signup",
"payload": { "customer_email": "john@acme.com", "company": "Acme Corp" }}}
```

**Tenant C — Appointment Reminder (triggers PII masking)**
```json
{ "body": { "tenant_id": "tenant_c", "event_type": "appointment_reminder",
"payload": { "customer_email": "patient@clinic.com", 
"patient_name": "Jane Smith", "dob": "1985-03-22" }}}
```

---

## Extensibility

Services not yet implemented — following the same gate + sub-workflow pattern:
- `oem-action-slack.json`
- `oem-action-sheet-log.json`  
- `oem-action-audit.json`

---

## Author

Javier Carazo — [linkedin.com/in/javiercarazo](https://linkedin.com/in/javiercarazo)