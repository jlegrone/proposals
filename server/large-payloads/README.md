# Large Payload Service

Author: Jacob LeGrone

_This proposal is a follow up to the talk "Temporal at Datadog" from Replay 2022: https://youtu.be/LxgkAoTSI8Q?t=1686_

## Problem Statement

Temporal imposes limits on history size in two dimensions: event count, and total storage used to represent those events measured in bytes.

Those limits are 50 thousand events, and 50MB in total event payloads.
Preventing overflowing event count is generally straightforward, as Temporal supports both resetting workflow history via its ContinueAsNew API, and handling larger numbers of tasks by creating trees of child workflows.
Reasoning about total storage used on the other hand is more difficult, especially if a use case involves request/response types of dynamic or unbounded size.

This is an issue that’s occasionally raised in slack and support forums, and the standard advice given is to keep large payloads in an external database or blob storage system, and then include pointers to these records in the actual event payloads persisted by Temporal server.
The problems with this approach are that
Workflow or activity code needs to have special treatment for large payloads, so a “sometimes large” payload might never be treated this way until it causes an issue in production.
There is no documented standard or open source implementation for this pattern
Very complex to deal with making branching decisions in workflow code based on large payloads
External datastores aren’t aware of Temporal cluster lifecycle, replication, or failover

## Requirements

### Do not load payloads into memory on Server

In order to support payloads of arbitrary size while keeping memory limits on individual Temporal server hosts small, we should avoid any protocols that require loading all payload bytes into memory on the server.

### Caching

Large payloads will need to be fetched during each workflow replay

### Support GDPR takedown requests

A large payload service may hold customer data, and therefore needs to support deleting payloads on request for specific customers. A Temporal server may be multi-tenant across and within namespaces, so deletion should be more fine-grained than per-namespace.

### Deduplicate Client Uploads

In certain cases clients may produce a large number of duplicate payloads, eg. a long-running workflow that frequently calls ContinueAsNew, or a heartbeating activity that persists large heartbeat details. In these cases, the ability to skip uploads would improve average latency on the client side and reduce storage costs on the server side.

### Integrity Checks

Payload bytes may be transmitted over wide area networks, so both both client, server, and caching nodes should be able to verify payload integrity at any time.

### Data Expiration

Clean up old payloads from storage without breaking running workflows.

### Developer Experience

The existing solution of manually uploading payloads to blob storage introduces a lot of friction for developers. Ideally large payloads should be handled more or less transparently.

## Proposed Solution


