# In-Chat Runtime (ICR)

A protocolized in-chat execution and audit runtime for structured multi-step reasoning.

## Status

**ICR v0.1 is a specification release.**

This repository currently defines:

- `SYS1` — boot contract, canonical state header, and sync governance
- `RTR` — router, protocol registry, sequence tracking, chain control, and watchdog
- `PROTO-EX` — a 4-step Tree-of-Thought execution protocol
- `PROTO-AU-EX` — a 3-step audit protocol for validation and patch-loop control
- `INDEX` — component registry, line references, and patch history

This is a design and protocol repository, not yet a full software implementation.

## Why it exists

Most chat-based workflows are informal, lossy, and difficult to audit. ICR defines a stricter runtime model:

- canonical message headers
- explicit state transitions
- versioned protocol registration
- deterministic sequence handling
- separate execution and audit roles
- word-count sync governance

The goal is to make multi-message reasoning more inspectable, replayable, and governable.

## Core runtime shape

```text
USER
  │
  ▼
SYS1 (boot / canonical state contract)
  │
  ▼
RTR (dispatch / sequence / watchdog / chain control)
  │
  ├─► PROTO-EX    (execution, 4-part)
  │        │
  │        ▼
  └────► PROTO-AU-EX (audit, 3-part)
             │
             ├─► SYS1 authoritative word count
             └─► RTR verdict signal (GO / CORRECT)
```

## Canonical state header

Every runtime message begins with:

```text
[STATE: <state> | PROTO: <active_protocol> | SEQ: <part/total> | SYNC: <status>]
```

Every runtime message ends with a word-count footer:

```text
[MSG_WORDS: <n> | TOTAL: <cumulative> | SYNC: <status>]
```

## Components

### SYS1
Defines runtime identity, canonical state-header format, word-count tracking, sync governance, and boot state.

### RTR
Defines protocol admission, dispatch behavior, transition rules, manual advancement, chain handling, unknown-protocol blocking, and watchdog reporting.

### PROTO-EX
Defines a four-stage execution protocol:

1. Plan
2. Build
3. Evaluate
4. Refine

### PROTO-AU-EX
Defines a three-stage audit protocol:

1. Receive
2. Analyze
3. Verdict

### INDEX
Tracks component registration, linguistic routing, line references, and patch history.

## Repository layout

```text
.
├─ CHANGELOG.md
├─ LICENSE
├─ ROADMAP.md
├─ docs/
│  ├─ architecture/
│  │  ├─ index-library.md
│  │  ├─ proto-au-ex.md
│  │  ├─ proto-ex.md
│  │  ├─ router.md
│  │  └─ sys1-boot.md
│  ├─ concepts/
│  │  └─ runtime-model.md
│  └─ examples/
│     └─ sample-flow.md
```

## Release scope

Initial public scope:

- publish the protocol contracts in readable Markdown
- establish versioned naming and component boundaries
- document current runtime semantics and known limitations
- provide a foundation for later implementation work

## Non-goals for v0.1

- full executable runtime
- automated transport or message passing
- persistent storage implementation
- UI layer
- provider adapters
- formal test harness

## Near-term direction

- complete repo normalization and naming cleanup
- add implementation-oriented schemas
- define protocol extension rules
- add runnable examples and validation fixtures
- publish the first software-backed prototype

## License

Apache-2.0.
