---
title: Webhooks
description: Plan event-driven integrations around signed, retryable BranchRunner notifications.
group: Integrate
order: 120
---
## Availability
The public webhook contract is being finalized. Event names and payloads are not yet a compatibility promise.

## Delivery model
Published webhooks will use signed requests, unique event identifiers, bounded retries, and documented ordering behavior. Receivers should be idempotent and return success only after durable acceptance.

Expected event families include task changes, run transitions, attention requests, validation results, and delivery outcomes.
