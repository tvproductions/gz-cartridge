# Design spec: gz-cartridge 0.1 records

Status: **draft for the Operator's review, 2026-11-27. Not approved.**
Nothing in `spec/` or `schema/`, and no validator code, is written until the
Operator approves this design (see `AGENTS.md` and the design map).

This is step 2 of the AFK factory build plan: the shared records plus a
reference validator. It turns the settled inputs in [map.md](map.md) into one
concrete design. Choices the Operator settled are marked **[SETTLED]**; the
rest are marked **[PROPOSAL]** and listed under "Questions for the Operator".
Research behind them is in [research-t4-t5-t6.md](research-t4-t5-t6.md).

## 1. The model in one paragraph

A **form** is the reusable structure of a unit of work: its purpose, typed
inputs and outputs, tools, limits, gates, evidence and failure handling. The
Operator attests each form revision at Gate 5, as design-time work. A
**cartridge** is one filled form: it names the TASKs it carries out and the
REQ or FIX each TASK serves, and it supplies the input, the model and the
effects approved for this run. One cartridge goes in; one **outcome** (the
receipt) comes out. A rerun is a fresh cartridge that cites its priors by
digest, like commit hashes. Runs stay AFK: code checks each cartridge
against its attested form, and the Operator steps in only to accept the work
order or to approve write and external effects.

(Operator, 2026-11-27: "one form in and a receipt out"; "I attest to
revisions of the form ... that is design-time work"; a cartridge "has a TASK
or TASKs - they are connected to either a REQ or a FIX".)

## 2. Scope

In 0.1:

| Record | `kind` | Written by | Read by |
| --- | --- | --- | --- |
| Form | `form` | The cartridge-writing skill and its tools | gzfactory (register), the validator |
| Attestation | `attestation` | The Operator, at Gate 5, per form revision | gzfactory, the executor |
| Cartridge | `cartridge` | gzfactory, once per run, from an attested form | gzyoke or another layer-1 executor |
| Outcome (receipt) | `outcome` | The executor, once per cartridge | gzfactory (accept, rerun, escalate) |

Out of 0.1, with the reason:

- **Mission** (T7). Its contents wait for gzcontractor's design.
- **Acceptance record.** The model only proposes done; code or a human
  accepts. Acceptance belongs to gzfactory at the work order, so 0.1 only
  makes sure the outcome never claims it.
- **Memory.** gzyoke Stage 3 has no memory, and durable memory is admitted
  only after acceptance outside the agent. 0.1 keeps a declared read scope
  and an always-empty list of proposed memories.

## 3. Shared rules for every document

1. **Envelope.** **[SETTLED, T2]** Every document starts with `spec_version`
   (semver, for example `"0.1.0"`) and `kind` (one of the four above). Before
   1.0, a minor bump may break documents; from 1.0, only a major does.
2. **Strict.** Every object rejects unknown fields
   (`additionalProperties: false`). An extension goes under one reserved `x`
   object with namespaced keys (`"x": {"gzfactory": {...}}`).
3. **Plain JSON.** UTF-8, no comments, no executable code. Times are RFC 3339
   strings in UTC. Sizes are bytes. Durations are seconds.
4. **Errors that teach.** Each validation error has `path` (JSON Pointer),
   `expected`, `found` and `fix`.
5. **Names and ids.** Form and skill names match `^[a-z][a-z0-9-]{1,62}$`.
   Versions are semver. TASK, REQ and FIX ids use gzkit's formats (section 5).
6. **Digest.** **[SETTLED, T3]** A document's identity is `sha256:` plus the
   lowercase hex SHA-256 of the document in RFC 8785 canonical form. The
   digest covers the whole document, which never contains its own digest.
   Forms and cartridges are both identified this way.

## 4. The form record

A form is sealed: changing any field makes a new revision, which needs a new
attestation (Q6). It cites skills and does not restate them (Q4). It holds
the 13 parts:

| Part (of 13) | Field | Rule |
| --- | --- | --- |
| Identity | `name`, `version`, `author` | Digest is computed (section 3) |
| Purpose | `purpose` | One sentence: the job and what counts as done |
| Rationale | `rationale` | Why this unit exists. Not sent to the model as policy |
| Typed inputs and outputs | `inputs`, `outputs` | JSON Schema objects in the 0.1 subset (section 8) |
| Instructions | `instructions` | Task text for this unit only; general norms come from skills |
| Guidance references | `skills` | List of `{name, version}`; each cited skill must carry its evaluation case |
| Tools and effects | `tools` | List of `{name, effect}`; `effect` is `read`, `propose`, `write` or `external` |
| Capabilities required | `capabilities`, `models` | `capabilities`: for example `tool_calls`. `models` (optional): the models the form was shaped and tested for **[SETTLED]** |
| Limits and stop rules | `limits` | Ceilings, all required and finite (section 4.1) |
| Human gates | `approval_required_for` | Effects that need a human approval before dispatch; always includes `write` and `external` |
| Evidence | `evidence` | What the run must record, and the checks that test the output |
| Knowledge and memory | `knowledge` | `read_scopes` (may be empty); 0.1 proposes no memories |
| Fixtures | `fixtures` | At least one `{name, input, expect}`, so the form can run against a fake model |
| Failure handling | `failure_handling` | For `stopped`, `blocked` and `failed`: `rerun`, `escalate` or `stop`, plus `max_reruns` |

Example (non-normative):

```json
{
  "spec_version": "0.1.0",
  "kind": "form",
  "name": "diagnose-failing-test",
  "version": "0.1.0",
  "author": "cartridge-writing skill",
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
  "models": ["qwen3-coder-30b"],
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
  "failure_handling": {"stopped": "rerun", "blocked": "escalate", "failed": "stop", "max_reruns": 1}
}
```

Rules beyond the schema (checked by the validator's domain):

- Tool names are unique. `write` and `external` are always in
  `approval_required_for`.
- Each fixture's `input` is valid against `inputs`.
- `rerun` means gzfactory writes a fresh cartridge that cites this one in
  `priors`. It never happens inside a run, and a form with `write` or
  `external` tools may not choose `rerun` for `stopped` or `failed`, so an
  uncertain effect is never repeated.
- `escalate` means the outcome is `blocked` and a human decides. Nothing
  invents consent.
- `evidence.checks[].kind` is `code` or `llm_judge`. An LLM judge must be a
  different model from the one that ran.

### 4.1 Limits

Every field is required in a form, except `token_budget`. Each value is a
positive integer (or a positive finite number for times); zero, negative,
non-finite and missing values are rejected. A cartridge may set any of them
again, but only lower. Counting follows gzyoke edition 3.

| Field | Counts | Trips stop reason |
| --- | --- | --- |
| `max_model_requests` | Every model invocation, including failed ones | `request_limit` |
| `max_tool_calls` | Every attempted tool dispatch, checked before the call; denied or invalid proposals do not count | `tool_limit` |
| `deadline_seconds` | Wall time from run start | `deadline` |
| `operation_timeout_seconds` | One awaited model or tool operation | `operation_timeout` |
| `max_observation_bytes` | UTF-8 bytes of one tool observation; longer results are cut and the cut is recorded | (none) |
| `max_final_bytes` | UTF-8 bytes of the final answer | `final_size` |
| `context_budget_bytes` | UTF-8 bytes of one assembled model request | `context_limit` |
| `max_repeated_calls` | Consecutive repeats of an already answered call fingerprint | `repeated_call` |
| `token_budget` (optional) | Model tokens, only when the model reports usage; asking for it with a model that cannot report is rejected | `token_budget` |

## 5. The cartridge record (a filled form)

gzfactory fills one cartridge per run, from one attested form revision. It
replaces the earlier "run request" **[SETTLED]**; gzyoke maps it to its own
"task request" at its input port.

| Field | Rule |
| --- | --- |
| `form` | `{name, version, digest}` of the form it fills |
| `attestation_ref` | Pointer to the Gate 5 attestation of that form digest |
| `tasks` | At least one `{task, serves}`: `task` is a gzkit TASK id, `serves` is the REQ or FIX it is connected to **[SETTLED]** |
| `work_order_id` | gzfactory's work order this cartridge belongs to |
| `priors` | Digests of earlier cartridges this one reruns or follows; empty for a first run |
| `input` | Valid against the form's `inputs` |
| `model` | `{name}`; one named model, no fallback. If the form lists `models`, one of them; otherwise any model with the form's `capabilities` **[SETTLED]** |
| `judge_model` | `{name}`, required when any check is `llm_judge`; must differ from `model` |
| `limits` | Optional; each value may only tighten the form's |
| `effects_approved` | Default: the form's declared effects that are `read` or `propose` |
| `approval_ref` | Required when `effects_approved` holds any effect in the form's `approval_required_for` |
| `attended` | `true` when a human watches the run, `false` for AFK |
| `filled_at` | RFC 3339 time |

Id formats **[PROPOSAL]**, taken from gzkit: TASK is
`TASK-<semver>-<obpi>-<req>-<seq>`, REQ is `REQ-<semver>-<obpi>-<criterion>`,
FIX is `FIX-<n>`. How a gz-starter project without gzkit mints these ids is
open (T10).

Example (non-normative):

```json
{
  "spec_version": "0.1.0",
  "kind": "cartridge",
  "form": {"name": "diagnose-failing-test", "version": "0.1.0", "digest": "sha256:9f2c...e1"},
  "attestation_ref": "attestations/diagnose-failing-test-0.1.0.json",
  "tasks": [{"task": "TASK-0.1.0-01-01-01", "serves": "FIX-12"}],
  "work_order_id": "wo-0007",
  "priors": [],
  "input": {"test_id": "tests/test_x.py::test_y", "log": "ImportError: no module named z"},
  "model": {"name": "qwen3-coder-30b"},
  "effects_approved": ["read"],
  "attended": false,
  "filled_at": "2026-11-27T13:00:00Z"
}
```

Rules beyond the schema:

- The form digest matches the form, and the attestation names that digest.
- `effects_approved` may only name effects the form's tools declare.
- Every approved effect in the form's `approval_required_for` needs
  `approval_ref`, attended or not, checked before the first dispatch.
- Unattended means diagnosis-only (gzkit ADR-0.0.62): without an
  `approval_ref` to the Operator's approval of that workflow, an AFK
  cartridge gets only the default effects.
- Each prior is the digest of an earlier cartridge in the same work order.
- An executor refuses a cartridge that does not check out and reports
  `rejected` (section 6).

## 6. The outcome record (the receipt)

One outcome per cartridge, written by the executor. It never says the work
is done; it says the model proposed done and the run's own checks passed.

**[SETTLED, T1]** `status` is one of six terminal values:

| `status` | Meaning | MCP / A2A nearest |
| --- | --- | --- |
| `done_proposed` | Final answer passed the runtime checks; the caller accepts or rejects | `completed` |
| `stopped` | A limit tripped; `stop_reason` says which | `failed` |
| `blocked` | A human decision is needed (approval, escalation) | `input_required` |
| `cancelled` | The caller cancelled the run | `cancelled` / `canceled` |
| `failed` | The run ended without an acceptable answer; `failure_kind` says why | `failed` |
| `rejected` | The run never started; `rejection_reason` says why | A2A `rejected` |

- `stop_reason` (exactly when `stopped`): `request_limit`, `tool_limit`,
  `deadline`, `operation_timeout`, `context_limit`, `final_size`,
  `repeated_call`, `token_budget`.
- `failure_kind` (exactly when `failed`) **[PROPOSAL]**: `operational`,
  `output_invalid` or `check_failed`. The form's `failure_handling.failed`
  applies to all three. A run ends `done_proposed` only when its answer
  matches `outputs` and every `code` check passed; `llm_judge` checks run
  after the outcome, as part of acceptance.
- `rejection_reason` (exactly when `rejected`): `form_digest_mismatch`,
  `form_not_attested`, `invalid_input`, `invalid_cartridge`,
  `effect_not_declared` or `approval_missing`. Never rerun automatically.

| Field | Rule |
| --- | --- |
| `cartridge` | Digest of the cartridge this is the receipt for |
| `form` | `{name, version, digest}`, copied from the cartridge; must match |
| `tasks` | Copied from the cartridge; must match |
| `status` and its reason field, `reason` | `reason` is short text for a human |
| `output` | Present exactly when `done_proposed`; valid against the form's `outputs` |
| `model` | `{name, version}` as the model reported it, or `version: null` |
| `usage` | `model_requests`, `tool_calls`, `elapsed_seconds`, optional `tokens`, and `provenance`: `reported`, `estimated` or `unavailable` |
| `checks` | One `{name, passed, detail}` per check that ran |
| `evidence` | `{journal_ref, last_event_seq}` |
| `proposed_memories` | Always empty in 0.1 |
| `started_at`, `ended_at` | RFC 3339 times |

A `rejected` outcome reports no execution it did not do: `started_at`,
`model` and `evidence` are `null`, `usage` counts are `0` with provenance
`unavailable`, `checks` is empty, and `ended_at` is when it was refused.

Example (non-normative):

```json
{
  "spec_version": "0.1.0",
  "kind": "outcome",
  "cartridge": "sha256:4b1a...07",
  "form": {"name": "diagnose-failing-test", "version": "0.1.0", "digest": "sha256:9f2c...e1"},
  "tasks": [{"task": "TASK-0.1.0-01-01-01", "serves": "FIX-12"}],
  "status": "stopped",
  "stop_reason": "tool_limit",
  "reason": "Fifth tool call refused: limit is 4.",
  "model": {"name": "qwen3-coder-30b", "version": null},
  "usage": {"model_requests": 3, "tool_calls": 4, "elapsed_seconds": 11.2, "provenance": "reported"},
  "checks": [],
  "evidence": {"journal_ref": "journal/wo-0007-1.jsonl", "last_event_seq": 17},
  "proposed_memories": [],
  "started_at": "2026-11-27T13:00:00Z",
  "ended_at": "2026-11-27T13:00:11Z"
}
```

## 7. The attestation record (Gate 5)

**[SETTLED, T3]** The Operator attests each form revision once, at design
time, before any cartridge filled from it can run. Filled cartridges are not
attested; code checks each one against its attested form.

| Field | Rule |
| --- | --- |
| `form` | `{name, version, digest}` of the attested form revision |
| `gate` | Always `"5"` in 0.1 |
| `attested_by` | A role, for example `"operator"`, never a personal name |
| `attested_at` | RFC 3339 time |
| `note` | Optional short text |

No signatures in 0.1. Where attestations are stored is the reader's choice.

## 8. JSON Schema subset for form inputs and outputs

**[SETTLED]** 0.1 allows only: `type`, `properties`, `required`,
`additionalProperties` (must be `false` on every object), `items`, `enum`,
`const`, `minLength`, `maxLength`, `minimum`, `maximum`, `minItems`,
`maxItems`, `pattern`, `description`, `title`. No `$ref`, no
`oneOf`/`anyOf`/`allOf`, no remote schemas. This keeps the validator small,
and small local models are most reliable on simple shapes.

## 9. The reference validator

Strict hexagonal per gz-starter ADR-0006, Python standard library only in the
core (domain, ports, handlers) **[PROVISIONAL]**. Pydantic stays out of the
core for now, following the Operator's most recent direction, but the
cross-repo ruling is open (T8): gzkit's
`docs/governance/hexagonal-architecture.md` puts frozen Pydantic models in
its kernel, which its domain imports. Until T8 is settled, Pydantic may
appear only in an optional binding package that gzfactory can install.

| Part | Contents |
| --- | --- |
| Driving adapters | CLI: `gz-cartridge check FILE...` and `gz-cartridge digest FILE`. Exit 0 when valid, 1 with errors |
| Input ports | `CheckDocument` (one document, a report); `CheckCartridge` (cartridge plus its form and attestation, and optionally its outcome, a report); `ComputeDigest` |
| Handlers | Load documents and bundled schemas through output ports, then call the domain. The only callers of output ports |
| Domain (pure) | Frozen records, the error model, the 0.1 schema-subset evaluator, the rules in sections 4 to 7, RFC 8785 canonical form and digest |
| Output ports | `DocumentSource` (bytes by location), `SchemaCatalog` (the bundled 0.1 schemas) |
| Driven adapters | File system source; package-resource catalog. Tests use hand-written fakes, never mocks |
| Configurator | One function that wires adapters to handlers; the only module that imports adapters |

A CI import-rule test fails the build on any boundary breach, with an empty,
shrink-only baseline from the first commit. A dev-only test runs the bundled
schemas and examples through the `jsonschema` library as a cross-check.

The validator is the heavy tool behind the cartridge-writing skill: the
skill says what a good form is and why; the validator and digest tools do
the checking.

## 10. Build plan after approval

Each step is one small PR:

0. ADRs for the lasting choices: the envelope and versioning (T2), the
   outcome statuses (T1), the digest and attestation (T3), the form and
   filled-cartridge model, and the validator's architecture (after T8).
   Each ADR edition needs the Operator's own approval; per the gz-starter ADR
   policy, approving a design does not approve an ADR. No step below starts
   before the ADRs it depends on are approved.
1. `spec/` prose for the four records, `schema/*.json` and the examples.
2. The validator skeleton: six parts, configurator, import-rule test, CI.
3. Document checks and teaching errors, with tests over valid and invalid
   examples.
4. Cross-record checks (`CheckCartridge`) and the digest.

gzyoke's Stage 3 records then track these records, as edition 3 asks.

## 11. Decisions and open questions

Settled by the Operator on 2026-11-27:

1. **T1.** Outcome statuses: gzyoke's five plus `rejected`.
2. **T2.** Envelope: `spec_version` (semver) and `kind`; breaking minor
   bumps allowed before 1.0.
3. **T3.** Digest: SHA-256 over RFC 8785 canonical JSON, `sha256:<hex>`.
   Minimal attestation record, no signatures.
4. **Model.** The cartridge names the model; the form lists capabilities and
   may list the models it was shaped for.
5. **Schema subset** as in section 8.
6. **Validator** (provisional): Pydantic out of the core; see T8.
7. **Terms.** Align with gzkit: a cartridge has a TASK or TASKs, each
   connected to a REQ or a FIX.
8. **Filled form.** A form is reusable only as a structure; each cartridge
   is a filled form. One form in, one receipt out. A rerun is a fresh
   cartridge citing its priors by digest.
9. **Gate 5** attests form revisions, at design time; filled cartridges are
   checked by code.

Still open:

- **T8.** Pydantic in the hex core, across repos.
- **T10.** Id minting for projects without gzkit.
- `failure_kind` values (section 6).
- Whether `outcome` should be renamed `receipt`, the Operator's word.
