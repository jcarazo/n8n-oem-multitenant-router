# v3 — AI Agent Dispatch via MCP

## What This Version Does

The event router receives a webhook and looks up the tenant configuration. 
Instead of a loop or a fixed orchestrator, an AI agent reads the tenant config, 
reasons about which services to invoke and in what order, and dispatches them 
as MCP tools — including enforcing PII masking before any downstream service 
receives sensitive data.

## Workflows

- **oem-event-router-v3.json** — Main entry point. Handles webhook intake and 
  tenant config lookup, then hands off to the AI agent for service orchestration.
- **oem-mcp-server.json** — Exposes available services as MCP tools that the 
  AI agent can discover and invoke dynamically.
- **oem-action-pii-masker.json** — PII masking service, invoked by the agent 
  before any downstream service that handles personally identifiable data.

## Architecture Decision

The orchestration logic is no longer encoded in a workflow — it is reasoned 
over at runtime by the AI agent. The agent reads the tenant config, determines 
which MCP tools to call, enforces ordering constraints (PII masking before 
downstream), and dispatches accordingly. The service catalog is the MCP server; 
adding a new service means registering a new tool, not modifying any workflow.

## What This Version Taught

When the routing logic itself needs to be dynamic — varying by tenant context, 
data sensitivity, and service dependencies — a static orchestrator cannot keep 
up without constant maintenance. An AI agent operating over a tool registry 
shifts the burden from workflow authoring to tool registration, which scales 
cleanly. The tradeoff is observability: agent reasoning is less predictable to 
debug than a deterministic loop, which is why authentication, retry logic, and 
monitoring are the natural next layer to build.

## Why This Matters Beyond n8n

The v1 → v2 → v3 progression mirrors a broader platform engineering pattern: 
hardcoded logic → data-driven abstraction → AI-driven reasoning. Each step 
increases flexibility and reduces maintenance overhead, at the cost of 
transparency and debuggability. Knowing when each tradeoff is worth making 
is the core of scalable platform design.
