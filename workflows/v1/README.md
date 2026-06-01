# v1 — Linear Service Chain

## What This Version Does

The event router receives a webhook, looks up the tenant configuration, and 
routes execution through a fixed sequence of gate-and-execute blocks — one per 
service. Each service (email, CRM) has a dedicated activation gate that checks 
whether that tenant has enabled it before executing the corresponding action.

## Workflows

- **oem-event-router.json** — Main entry point. Handles webhook intake, tenant 
  config lookup, and sequential routing through each service gate.
- **oem-activation-gate.json** — Reusable gate logic that checks tenant config 
  before allowing a service to execute.
- **oem-action-email.json** — Email service action, triggered when enabled for 
  the tenant.
- **oem-action-crm.json** — CRM service action, triggered when enabled for the 
  tenant.

## Architecture Decision

Each service is hardwired as a node in the main workflow. The structure is 
intentionally flat and explicit — every execution path is visible at a glance, 
making it easy to understand and debug.

## What Breaks at Scale

Adding a new service means adding new nodes directly to the main workflow. At 
three or four services this is manageable. At ten it becomes fragile — the 
workflow grows linearly with every new integration, coupling the routing logic 
tightly to the service catalog.

## What This Version Taught

Simplicity is a feature, not a shortcut. v1 is the right starting point because 
it makes the problem fully legible before introducing abstraction. The ceiling 
it hits — tight coupling between routing and execution — is exactly what v2 
is designed to break.
