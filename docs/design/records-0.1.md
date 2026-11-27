# Design spec: gz-cartridge 0.1 records

Status: **draft for the Operator's review, 2026-11-27. Not approved.**
Nothing in `spec/` or `schema/`, and no validator code, is written until the
Operator approves this design (see `AGENTS.md` and the design map).

This is step 2 of the AFK factory build plan: the cartridge, run request and
outcome records, plus a reference validator. It turns the settled inputs in
[map.md](map.md) into one concrete proposal. Every choice that belongs to an
open HITL ticket is marked **[PROPOSAL]** and listed again under
"Questions for the Operator". Research behind the proposals is in
[research-t4-t5-t6.md](research-t4-t5-t6.md).

## 1. Scope

In 0.1:

| Record | Written by | Read by |
| --- | --- | --- |
| `cartridge` | The cartridge-writing skill and its tools; attested at Gate 5 | gzfactory (register, bind), gzyoke (run) |
| `run_request` | gzfactory, once per run of one workflow step | gzyoke or another layer-1 executor |
| `outcome` | gzyoke or another layer-1 executor, once per run | gzfactory (accept, retry, escalate) |

Plus a minimal `attestation` record (section 6), a reference validator for
all four, and the checks that span them.

Out of 0.1, with the reason:

- **Mission** (T7). Its contents wait for gzcontractor's design. gzfactory can
  run a work order without one.
- **Acceptance record.** The model only proposes done; code or a human
  accepts. Acceptance is gzfactory's record (Gate 5 at the work order), so
  0.1 only makes sure the outcome never claims it.
- **Memory.** gzyoke Stage 3 has no memory, and durable memory is admitted
  only after acceptance outside the agent. 0.1 keeps a declared read scope
  and an always-empty list of proposed memories, so the fields exist when the
  rules arrive.

## 2. Shared rules for every document

1. **Envelope.** **[SETTLED, T2, Operator 2026-11-27]** Every document starts with two fields:
   `spec_version` (the gz-cartridge version it follows, semver, for example
   `"0.1.0"`) and `kind` (`"cartridge"`, `"run_request"` or `"outcome"`).
   This follows OpenAPI's in-document version and Agent Spec's type tag.
   Before 1.0, a minor bump may break documents; from 1.0, only a major does.
2. **Strict.** Every object rejects unknown fields
   (`additionalProperties: false`). An extension, when needed, goes under one
   reserved `x` object whose keys are namespaced (`"x": {"gzfactory": {...}}`).
3. **Plain JSON.** UTF-8, no comments, no executable code. Times are RFC 3339
   strings in UTC. Sizes are bytes. Durations are seconds.
4. **Errors that teach.** Each validation error has `path` (JSON Pointer),
   `expected`, `found` and `fix`.
5. **Names.** Cartridge and skill names match `^[a-z][a-z0-9-]{1,62}$`.
   Versions are semver.

## 3. The cartridge record

One cartridge is one atomic unit of work, run as one workflow step (Q3). It
cites skills and does not restate them (Q4). It is sealed: no field may be
changed after attestation without making a new version (Q6).

| Part (of 13) | Field | Rule |
| --- | --- | --- |
| Identity | `name`, `version`, `author` | Content digest is computed, never stored inside (section 6) |
| Purpose | `purpose` | One sentence: the job and what counts as done |
| Rationale | `rationale` | Why this unit exists. Not sent to the model as policy |
| Typed inputs and outputs | `inputs`, `outputs` | JSON Schema objects (2020-12), in the 0.1 subset (section 7) |
| Instructions | `instructions` | Task text for this unit only; general norms come from skills |
| Guidance references | `skills` | List of `{name, version}`; each cited skill must exist and carry its evaluation case |
| Tools and effects | `tools` | List of `{name, effect}`; `effect` is `read`, `propose`, `write` or `external` |
| Capabilities required | `capabilities`, `models` | `capabilities`: for example `tool_calls`, `long_context`. `models` (optional): the models the cartridge was shaped and tested for. The run request names the model (section 4) |
| Limits and stop rules | `limits` | Every limit required and finite (section 3.1) |
| Human gates | `approval_required_for` | Effect categories that need a human approval before dispatch; `write` and `external` at least |
| Evidence | `evidence` | What the run must record, and the checks that test the output |
| Knowledge and memory | `knowledge` | `read_scopes` (list, may be empty); 0.1 proposes no memories |
| Fixtures | `fixtures` | At least one `{name, input, expect}`, so the cartridge can run against a fake model |
| Failure handling | `failure_handling` | For `stopped`, `blocked` and `failed`: `retry`, `escalate` or `stop`, plus `max_retries` |

Example (non-normative):

```json
{
  "spec_version": "0.1.0",
  "kind": "cartridge",
  "name": "diagnose-failing-test",
  "version": "0.1.0",
  "author": "cartridge-writing skill, attested by the Operator",
  "purpose": "Name the most likely cause of one failing test; done when a cause and its evidence are reported.",
  "rationale": "Diagnosis-only work is safe to run unattended and saves the Operator triage time.",
  "instructions": "Read the failing test and the code it covers. Report the most likely cause with file and line evidence. Change nothing.",
  "inputs": {
    "type": "object",
    "properties": {"test_id": {"type": "string"}, "log": {"type": "string"}},
    "required": ["test_id", "log"],
    "additionalProperties": false
  },
  "outputs": {
    "type": "object",
    "properties": {
      "cause": {"type": "string"},
      "evidence": {"type": "array", "items": {"type": "string"}, "minItems": 1}
    },
    "required": ["cause", "evidence"],
    "additionalProperties": false
  },
  "skills": [{"name": "diagnose-before-fix", "version": "1.0.0"}],
  "tools": [
    {"name": "read_file", "effect": "read"},
    {"name": "search_code", "effect": "read"}
  ],
  "capabilities": ["tool_calls"],
  "limits": {
    "max_model_requests": 4,
    "max_tool_calls": 4,
    "deadline_seconds": 30,
    "operation_timeout_seconds": 5,
    "max_observation_bytes": 8192,
    "max_final_bytes": 16384,
    "context_budget_bytes": 32768,
    "max_repeated_calls": 2
  },
  "approval_required_for": ["write", "external"],
  "evidence": {
    "record": ["input", "skills", "model", "tool_calls", "output"],
    "checks": [{"name": "evidence-cites-files", "kind": "code"}]
  },
  "knowledge": {"read_scopes": []},
  "fixtures": [
    {
      "name": "import-error",
      "input": {"test_id": "tests/test_x.py::test_y", "log": "ImportError: no module named z"},
      "expect": {"status": "done_proposed", "output_has": ["cause", "evidence"]}
    }
  ],
  "failure_handling": {
    "stopped": "retry",
    "blocked": "escalate",
    "failed": "stop",
    "max_retries": 1
  }
}
```

Rules beyond the schema (checked by the validator's domain):

- Tool names are unique. Every `effect` in `approval_required_for` is a real
  category. `write` and `external` are always in it.
- Each fixture's `input` is valid against `inputs`.
- `retry` never repeats a completed or uncertain `write` or `external`
  effect; a cartridge with such tools may not choose `retry` for `stopped`
  or `failed`. Retry happens in gzfactory between runs, never inside one.
- `escalate` means the outcome is `blocked` and a human decides. Nothing
  invents consent.
- `evidence.checks[].kind` is `code` or `llm_judge`. An LLM judge must be a
  different model from the one that ran; the run request records both.

### 3.1 Limits

Every field is required in a cartridge, except `token_budget`. Each value is
a positive integer (or a positive finite number for times); zero, negative,
non-finite and missing values are rejected. A run request may set any of them
again, but only lower. Counting follows gzyoke edition 3.

| Field | Counts | Trips stop reason |
| --- | --- | --- |
| `max_model_requests` | Every model invocation, including failed ones | `request_limit` |
| `max_tool_calls` | Every attempted tool dispatch, checked before the call; denied or invalid proposals do not count | `tool_limit` |
| `deadline_seconds` | Wall time from run start | `deadline` |
| `operation_timeout_seconds` | One awaited model or tool operation | `operation_timeout` |
| `max_observation_bytes` | UTF-8 bytes of one tool observation; longer results are cut, not stopped | (none: truncation is recorded) |
| `max_final_bytes` | UTF-8 bytes of the final answer | `final_size` |
| `context_budget_bytes` | UTF-8 bytes of one assembled model request | `context_limit` |
| `max_repeated_calls` | Consecutive repeats of an already answered call fingerprint | `repeated_call` |
| `token_budget` (optional) | Model tokens, only when the model reports usage; a request for it against a model that cannot report is rejected | `token_budget` |

## 4. The run request record

gzfactory writes one run request per run. It pins one attested cartridge
version and supplies everything the run needs that is not part of the
cartridge: the input, the model, tighter limits and the effects approved for
this run. It replaces gzyoke edition 3's "task request" **[PROPOSAL]**: gzyoke
keeps its own word inside its code and maps it at its input port.

| Field | Rule |
| --- | --- |
| `run_id` | Unique string, set by gzfactory |
| `work_order_id` | gzfactory's work order this run belongs to |
| `cartridge` | `{name, version, digest}`; the digest must match the cartridge's computed digest |
| `attestation_ref` | Pointer to the Gate 5 attestation of that digest (format is T3) |
| `input` | Must be valid against the cartridge's `inputs` |
| `model` | `{name}`; one named model, no fallback (gzyoke edition 3). If the cartridge has `models`, it must be one of them; otherwise any model with the cartridge's `capabilities` |
| `judge_model` | `{name}`, required when any evidence check is `llm_judge`; must differ from `model` |
| `limits` | Optional; each value may only tighten the cartridge's limit |
| `effects_approved` | Effect categories approved for this run. Default: the cartridge's declared effects that are `read` or `propose` |
| `approval_ref` | Required when `effects_approved` holds any effect in the cartridge's `approval_required_for`: a pointer to the human approval of that effect for this workflow |
| `attended` | `true` when a human watches the run, `false` for AFK |
| `requested_at` | RFC 3339 time |

Rules beyond the schema:

- `effects_approved` may only name effects the cartridge's tools declare.
- Every effect in `effects_approved` that the cartridge lists in
  `approval_required_for` needs `approval_ref`, attended or not. The executor
  checks it before the first dispatch, not when the effect is first used.
- Unattended means diagnosis-only (gzkit ADR-0.0.62): when `attended` is
  `false`, the default above is all a run gets unless `approval_ref` points to
  the Operator's approval of that workflow to change things. Because `write`
  and `external` are always in `approval_required_for`, the rule above
  already enforces this.
- An executor refuses a request whose cartridge digest, attestation or input
  does not check out, and reports `rejected` (section 5).

## 5. The outcome record

One outcome per run, written by the executor. It is the run's last word. It
never says the work is done; it says the model proposed done and the run's
own checks passed.

**[SETTLED, T1, Operator 2026-11-27]** `status` is one of six terminal values:
gzyoke edition 3's five, plus `rejected` from A2A.

| `status` | Meaning | MCP / A2A nearest |
| --- | --- | --- |
| `done_proposed` | Final answer passed the runtime checks; the caller accepts or rejects | `completed` |
| `stopped` | A limit tripped; `stop_reason` says which | `failed` |
| `blocked` | A human decision is needed (approval, escalation) | `input_required` |
| `cancelled` | The caller cancelled the run | `cancelled` / `canceled` |
| `failed` | The run ended without an acceptable answer; `failure_kind` says why | `failed` |
| `rejected` | The run never started: bad digest, missing attestation, invalid input | A2A `rejected` |

`failure_kind` (required exactly when `status` is `failed`) **[PROPOSAL]**:
`operational` (model, tool, journal or policy error), `output_invalid` (the
final answer does not match the cartridge's `outputs`) or `check_failed` (a
code evidence check failed). The cartridge's `failure_handling.failed` policy
applies to all three. An answer can only end `done_proposed` when it matches
`outputs` and every `code` check passed; an `llm_judge` check is run by the
caller after the outcome, because judging is part of acceptance.

`rejection_reason` (required exactly when `status` is `rejected`):
`digest_mismatch`, `not_attested`, `invalid_input`, `invalid_request`,
`effect_not_declared` or `approval_missing`. Rejection is never retried
automatically; it always goes back to whoever wrote the run request.

`stop_reason` (required exactly when `status` is `stopped`), from gzyoke
edition 3: `request_limit`, `tool_limit`, `deadline`, `operation_timeout`,
`context_limit`, `final_size`, `repeated_call`, `token_budget`.

| Field | Rule |
| --- | --- |
| `run_id`, `cartridge` | Copy of the run request's; must match |
| `status`, `stop_reason`, `reason` | `reason` is short text for a human |
| `output` | Present exactly when `status` is `done_proposed`; valid against the cartridge's `outputs` |
| `model` | `{name, version}` as the model reported it, or `version: null` |
| `usage` | `model_requests`, `tool_calls`, `elapsed_seconds`, optional `tokens`, and `provenance`: `reported`, `estimated` or `unavailable` |
| `checks` | One `{name, passed, detail}` per evidence check that ran |
| `evidence` | `{journal_ref, last_event_seq}`: where the run's journal is, and the last committed event |
| `proposed_memories` | Always an empty list in 0.1 |
| `started_at`, `ended_at` | RFC 3339 times |

A `rejected` outcome reports no execution it did not do: `started_at`,
`model` and `evidence` are `null`, `usage` counts are `0` with provenance
`unavailable`, `checks` is empty, and `ended_at` is when the request was
refused. The validator rejects a `rejected` outcome that carries any other
values in these fields.

Example (non-normative):

```json
{
  "spec_version": "0.1.0",
  "kind": "outcome",
  "run_id": "wo-0007-r1",
  "cartridge": {"name": "diagnose-failing-test", "version": "0.1.0", "digest": "sha256:9f2c...e1"},
  "status": "stopped",
  "stop_reason": "tool_limit",
  "reason": "Fifth tool call refused: limit is 4.",
  "model": {"name": "qwen3-coder-30b", "version": null},
  "usage": {"model_requests": 3, "tool_calls": 4, "elapsed_seconds": 11.2, "provenance": "reported"},
  "checks": [],
  "evidence": {"journal_ref": "journal/wo-0007-r1.jsonl", "last_event_seq": 17},
  "proposed_memories": [],
  "started_at": "2026-11-27T13:00:00Z",
  "ended_at": "2026-11-27T13:00:11Z"
}
```

## 6. Digest and attestation

**[SETTLED, T3 digest, Operator 2026-11-27]** A cartridge's identity for Gate 5 is
`sha256:` plus the lowercase hex SHA-256 of the cartridge document in RFC 8785
canonical form (JSON Canonicalization Scheme). The digest covers the whole
document; the document never contains its own digest. Canonical form means
whitespace and key order cannot change identity, and any change to content
does. The form `algorithm:encoded` follows OCI, so another algorithm can be
added later.

**[SETTLED, T3 attestation, Operator 2026-11-27]** The Gate 5 attestation is
a minimal record of `kind` `"attestation"`:

| Field | Rule |
| --- | --- |
| `cartridge` | `{name, version, digest}` of the attested cartridge |
| `gate` | Always `"5"` in 0.1 |
| `attested_by` | A role, for example `"operator"`, never a personal name |
| `attested_at` | RFC 3339 time |
| `note` | Optional short text |

No signatures in 0.1. Where attestations are stored is the reader's choice;
a run request's `attestation_ref` points to one, and the executor checks that
its digest matches the pinned cartridge before the run starts.

## 7. JSON Schema subset for cartridge inputs and outputs

Cartridge authors write `inputs` and `outputs` as JSON Schema. To keep the
validator tiny and its errors good, **[SETTLED, Operator 2026-11-27]** 0.1
allows only this subset:
`type`, `properties`, `required`, `additionalProperties` (must be `false` on
every object), `items`, `enum`, `const`, `minLength`, `maxLength`, `minimum`,
`maximum`, `minItems`, `maxItems`, `pattern`, `description`, `title`.
No `$ref`, no `oneOf`/`anyOf`/`allOf`, no remote schemas. A cartridge that
needs more waits for a later version. This also suits small local models,
whose structured output is most reliable on simple shapes.

## 8. The reference validator

Strict hexagonal per gz-starter ADR-0006, Python standard library only at
runtime **[PROPOSAL]**.

| Part | Contents |
| --- | --- |
| Driving adapters | CLI: `gz-cartridge check FILE...` and `gz-cartridge digest FILE`. Exit 0 when valid, 1 with errors |
| Input ports | `CheckDocument` (one document in, a report out); `CheckRun` (cartridge plus run request, and optionally outcome, in; a report out); `ComputeDigest` |
| Handlers | Load documents and bundled schemas through output ports, then call the domain. The only callers of output ports |
| Domain (pure) | Frozen records, the error model, the 0.1 JSON Schema subset evaluator, the rules in sections 3 to 5, RFC 8785 canonical form and digest |
| Output ports | `DocumentSource` (bytes by location), `SchemaCatalog` (the bundled 0.1 schemas) |
| Driven adapters | File system source; package-resource catalog. Tests use hand-written fakes, never mocks |
| Configurator | One function that wires adapters to handlers; the only module that imports adapters |

A CI import-rule test fails the build on any boundary breach, with an empty,
shrink-only baseline from the first commit. A dev-only test runs the bundled
schemas and examples through the full `jsonschema` library as a cross-check;
it is never a runtime dependency.

The validator is the "heavy tool" behind the cartridge-writing skill: the
skill says what a good cartridge is and why; the validator and digest tools
do the checking.

## 9. Build plan after approval

Each step is one small PR:

0. ADRs for the lasting choices this design makes: the record envelope and
   versioning (T2), the outcome statuses (T1), the digest (T3) and the
   validator's architecture. Each ADR edition needs the Operator's own
   approval, separately from approving this design; per the gz-starter ADR
   policy, approving a design does not approve an ADR. No step below that
   depends on an ADR starts before that ADR edition is approved.
1. `spec/` prose for the three records, `schema/*.json` and the examples,
   from this design as approved.
2. The validator skeleton: six parts, configurator, import-rule test, CI.
3. Document checks and teaching errors, with tests over valid and invalid
   examples.
4. Cross-record checks (`CheckRun`) and the digest.

gzyoke's Stage 3 records then track these records, as edition 3 asks.

## 10. Questions for the Operator

Asked one at a time, in this order. Answers are recorded here and in the map.

1. **T1.** Outcome statuses: gzyoke's five plus `rejected` (section 5)?
   **Answered yes by the Operator, 2026-11-27.**
2. **T2.** Envelope: `spec_version` (semver) and `kind` in every document,
   with breaking minor bumps allowed before 1.0 (section 2)?
   **Answered yes by the Operator, 2026-11-27.**
3. **T3.** Digest: SHA-256 over RFC 8785 canonical JSON, written
   `sha256:<hex>` (section 6)? And what must the attestation record hold?
   **Digest answered yes by the Operator, 2026-11-27. Minimal attestation
   record (digest, role, time, optional note, no signatures) answered yes the
   same day.**
4. **Model.** **Settled by the Operator, 2026-11-27:** the run request names
   the model. The cartridge lists `capabilities` and may also list `models`
   it was shaped and tested for; when it does, the run request must pick one
   of them.
5. **Schema subset.** Limit cartridge `inputs` and `outputs` to the subset in
   section 7 for 0.1? **Answered yes by the Operator, 2026-11-27.**
6. **Validator.** Own small evaluator, standard library only, with
   `jsonschema` as a dev-only cross-check (section 8)?
7. **Term.** Keep "run request" in gz-cartridge while gzyoke says "task
   request" internally?
