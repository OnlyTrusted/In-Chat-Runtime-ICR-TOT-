# Runtime Model

ICR is a protocolized chat runtime built around explicit state, sequence, and audit semantics.

## Design intent

The system treats a chat as a controlled execution surface rather than an informal conversation log. Each message is expected to carry runtime metadata, participate in a defined protocol, and remain legible to a paired audit process.

## Core principles

- **Canonical state envelope** — every message carries the same state-header format.
- **Protocol registration** — only registered protocol IDs may be dispatched.
- **Manual advancement** — protocol progression is explicit, not implicit.
- **Execution/audit separation** — execution and validation are distinct roles.
- **Boundary-safe sync** — resets occur only at clean execution boundaries.
- **Index-backed governance** — components, line references, and patches are tracked.

## Runtime actors

### SYS1
Global boot and sync contract.

### RTR
Execution-side router responsible for dispatch, sequencing, chain state, and watchdog footer generation.

### PROTO-EX
A four-part Tree-of-Thought execution protocol.

### PROTO-AU-EX
A three-part audit protocol that validates execution units and controls patch loops.

## Message model

Each message includes:

1. a canonical state header
2. protocol-specific body content
3. a word-count footer
4. router and watchdog metadata where applicable

## Control model

Protocol progression is intentionally constrained:

- unknown protocols are blocked
- advancement is manual
- audit determines whether a completed unit closes or re-enters a correction loop
- sync execution is deferred until a clean boundary is available
