# T-02 — Action Alphabet Recorder

Status: execution complete. Audit analyze artifact captured with 7/7 checks passed. Final verdict line is not present in the uploaded audit log artifact.

## Goal

Define and implement the IDEOS action alphabet recorder with:

- deterministic semantics
- replayability
- visually grounded targeting
- RPA-grade action structure
- serializability into the T-01 raw capture layer

## Selected solution

Selected branch: **Branch C — Typed action schema + hierarchical session/run model**.

This task closed the provenance gap between the action layer and the raw capture layer by making action records reference raw capture rather than duplicating it.

## Closed action enum

```text
C | P | CO | W | SC | SR | CUTOUT
```

Unknown action type is rejected and never promoted into the committed action log.

## Cutout registry

Session-level registry at:

```text
session_{id}/cutouts/registry.json
```

Shape:

```json
{
  "cutouts": [
    {
      "id": "SEND_BTN",
      "label": "Send",
      "region": {"x": 0, "y": 0, "w": 0, "h": 0},
      "capture_ts": "<ISO-8601>",
      "last_resolved": null
    }
  ]
}
```

Rules:

- cutouts are session-scoped and shared across runs
- `last_resolved` is updated after successful replay resolution
- cutout updates are append-only; prior entries are retained

## Per-type schemas

### C — Click

```json
{
  "seq": 1,
  "ts": "<ISO>",
  "action": "C",
  "run_id": "<id>",
  "target_id": "<cutout_id>",
  "coordinates": {"x": 0, "y": 0},
  "button": "left"
}
```

### P — Paste

```json
{
  "seq": 1,
  "ts": "<ISO>",
  "action": "P",
  "run_id": "<id>",
  "target_id": "<cutout_id>",
  "content_ref": 12
}
```

`content_ref` points to a QAi raw capture record in `raw/MSG_{n}.json`. Paste content is referenced, never duplicated.

### CO — Copy

```json
{
  "seq": 1,
  "ts": "<ISO>",
  "action": "CO",
  "run_id": "<id>",
  "target_id": "<cutout_id>",
  "result_ref": 19
}
```

`result_ref` points to an AAi raw capture record written before the copy action is committed.

### W — Wait

```json
{
  "seq": 1,
  "ts": "<ISO>",
  "action": "W",
  "run_id": "<id>",
  "condition_type": "element_exists",
  "condition_value": "<cutout_id>"
}
```

Allowed condition types:

- `element_exists`
- `timeout`

### SC — Scroll

```json
{
  "seq": 1,
  "ts": "<ISO>",
  "action": "SC",
  "run_id": "<id>",
  "target_id": "<cutout_id>",
  "direction": "down",
  "magnitude": 1
}
```

### SR — Screen record

```json
{
  "seq": 1,
  "ts": "<ISO>",
  "action": "SR",
  "run_id": "<id>",
  "event": "checkpoint",
  "label": "after-submit",
  "file_ref": "recordings/after-submit_123.mp4"
}
```

### CUTOUT — Target definition

```json
{
  "seq": 1,
  "ts": "<ISO>",
  "action": "CUTOUT",
  "run_id": "<id>",
  "id": "SEND_BTN",
  "label": "Send",
  "region": {"x": 0, "y": 0, "w": 0, "h": 0},
  "capture_ts": "<ISO-8601>"
}
```

CUTOUT writes both:

- the action record in the run log
- the mirrored session-level registry entry

## Session directory model

T-02 extends T-01 like this:

```text
session_{id}/
  raw/
  meta/
    session.json
    recovery.json
  integrity/
    checksums.json
  runs/
    run_{n}/
      actions/
        ACTION_{seq}.json
      meta/
        run.json
        rejected.json
  cutouts/
    registry.json
  recordings/
    <label>_{ts}.mp4
```

## Run lifecycle

1. initialize `runs/run_{n}/`
2. write `meta/run.json` with `state: OPEN`
3. validate each action against its type schema
4. reject invalid actions into `meta/rejected.json`
5. commit valid actions sequentially
6. close run with `state: CLOSED` and `action_count: n`

## Write sequence per action

1. validate against type schema
2. if invalid, append to `rejected.json` and skip promotion
3. write `actions/tmp_ACTION_{seq}.json`
4. atomically rename to `actions/ACTION_{seq}.json`
5. if action type is `CUTOUT`, sync to `cutouts/registry.json`

## Provenance chain

```text
P action  → content_ref → raw/MSG_{QAi_n}.json
CO action → result_ref  → raw/MSG_{AAi_n}.json
SR action → file_ref    → recordings/<file>
CUTOUT    → registry.json entry
```

This preserves the key IDEOS invariant:

**ground truth = raw capture**

The action layer references the raw layer. It does not replace it, copy it, or rewrite it.

## Replay sequence

1. open session
2. run T-01 integrity scan
3. load `cutouts/registry.json`
4. iterate runs in order
5. load actions in sequence order
6. resolve `target_id` to current cutout region
7. if resolution fails, log `REPLAY_FAIL_{seq}` and continue
8. execute action against the resolved target
9. for `W(element_exists)`, poll until satisfied or timeout fallback fires

## Cross-branch carryover decisions

Rejected from Branch A:

- single flat `actions.json` array per run
- reason: weaker crash isolation and worse alignment with T-01 atomic rename pattern

Rejected from Branch B:

- `.jsonl` append-only action log
- reason: introduces flock + partial-write recovery complexity and breaks consistency with the one-file-per-action atomic commit model

## Why this branch won

Branch C was the only option that simultaneously delivered:

- strict typed semantics
- replay-stable cutout targeting
- first-class wait conditions
- direct provenance links back to QAi/AAi raw capture
- full structural alignment with the T-01 session bundle

## Audit evidence captured in uploaded logs

The uploaded audit artifact for T-02 shows:

- structural intake clean
- audit-counted and execution-reported unit total both at 2,562 words
- analyze summary with **7/7 checks passed** and **0/7 flagged**

The final explicit MSG 3 verdict line is not present in the uploaded audit file, so this document records T-02 conservatively as execution-complete with captured positive analyze evidence rather than claiming a stored final verdict that is not present in the artifact.
