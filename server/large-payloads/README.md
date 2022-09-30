# Large Payload Service

Author: Jacob LeGrone


## Problem Statement

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

## Proposed Solution


