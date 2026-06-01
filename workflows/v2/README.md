# v2 — Loop-Based Orchestration

## What This Version Does

The event router receives a webhook, looks up the tenant configuration, and 
builds a dynamic list of enabled services. A loop iterates over that list and 
delegates each service execution to a central orchestrator sub-workflow. Adding 
a new service requires no structural changes to the main router.

## Workflows

- **oem-event-router-v2.json** — Main entry point. Handles webhook intake, 
  tenant config lookup, and constructs the service execution list. Delegates 
  to the orchestrator via loop.
- **oem-service-orchestrator.json** — Central sub-workflow that receives a 
  service identifier and routes execution to the correct action. Acts as the 
  single dispatch point for all service calls.

## Architecture Decision

Services are decoupled from the router by treating them as data — an array of 
identifiers — rather than hardwired nodes. The loop iterates over whatever the 
tenant config returns, making the main workflow structurally stable regardless 
of how many services are added or removed.

## What Breaks at Scale

The orchestrator sub-workflow still contains explicit routing logic mapping 
service identifiers to execution paths. As the service catalog grows, the 
orchestrator accumulates conditional branches. The routing intelligence is 
more contained than v1, but it still lives in a static workflow that requires 
manual updates when the catalog changes.

## What This Version Taught

Decoupling routing from execution is the right move — but the orchestration 
logic itself still needs a home. In v2 that home is a sub-workflow, which is 
better than the main router but still a fixed artifact. v3 asks: what if the 
orchestration logic didn't need to be written down at all?
