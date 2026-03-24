# T-01 — Storage Contract

Status: execution complete, audit passed, router returned to standby.

## Goal

Define the IDEOS raw capture layer with a storage contract that is:

- lossless
- immutable
- strictly ordered
- crash-safe
- never rewritten

## Selected solution

Selected branch: **Branch C — Structured session bundle**.

This task converged on a directory-as-unit model where each session contains raw capture, metadata, and integrity artifacts in one self-contained bundle.

## Session directory structure

```text
sessions/
  session_{timestamp}_{uuid}/
    raw/
      MSG_{n}.json
    meta/
      session.json
      recovery.json
    integrity/
      checksums.json
```

## Raw message contract

Each captured message is stored as `raw/MSG_{n}.json` using this schema:

```json
{
  "seq": 1,
  "ts": "<ISO-8601 timestamp>",
  "origin": "QAi",
  "type": "capture",
  "topic": null,
  "content": "<raw string — unmodified>"
}
```

Notes:

- `seq` is globally monotonic within the session.
- `topic` is present from day one and remains `null` until the later indexing phase.
- raw content is never rewritten.

## Write sequence

Per capture event:

1. Serialize payload to JSON.
2. Write to `raw/tmp_MSG_{n}.json`.
3. Atomic rename to `raw/MSG_{n}.json`.
4. Compute SHA-256 of the committed file.
5. Append `{seq, file, sha256, ts}` to `integrity/checksums.json`.

## Session lifecycle

### Init

- generate `session_id = {timestamp}_{uuid4}`
- create the directory tree
- write `meta/session.json`
- create empty `integrity/checksums.json` as `[]`

### Close

Update `meta/session.json` with:

- `state: CLOSED`
- `record_count: n`

## Crash recovery

On session open:

1. read `integrity/checksums.json`
2. recompute hash for each raw file
3. compare computed hash against recorded hash

Outcomes:

- hash match → `VERIFIED`
- hash mismatch → `INTEGRITY_FAIL_{n}`
- missing file → `MISSING_{n}`

If any flags exist:

- write them to `meta/recovery.json`
- set session state to `DEGRADED`
- preserve all verified records as valid

If no flags exist:

- set session state to `CLEAN`

## Replay rules

- run the integrity scan first
- load only verified records from `raw/` in sequence order
- surface degraded or missing records explicitly
- never silently skip failures

## Immutability guarantee

Raw files are write-once. Post-write modification is detected by SHA-256 recorded in `integrity/checksums.json`. No later IDEOS layer rewrites `raw/`.

## Cross-branch carryover

Adopted from Branch A:

- orphan detection via `tmp_*` scan during recovery

Rejected from Branch B:

- `.jsonl` manifest with byte offsets
- reason: duplicates integrity responsibility with weaker guarantees than hash verification

## Why this branch won

Branch C was the only option that simultaneously delivered:

- verifiable immutability
- portable self-contained auditability
- a clean provenance surface for later QAi/AAi lineage
- explicit degraded-state handling without data deletion

## Audit outcome captured in logs

The uploaded audit artifact shows:

- structural intake clean
- all 7 checks passed
- final verdict `GO`
- authoritative unit word count: 2,012

That makes T-01 the first task-grade artifact appropriate for public repository inclusion.
