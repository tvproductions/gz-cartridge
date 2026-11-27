# Research notes: tickets T4, T5 and T6

Status: AFK research report, 2026-11-27. These are findings, not decisions.
The proposals they feed are in [records-0.1.md](records-0.1.md), and the
Operator settles them there.

Sources were checked on 2026-11-27 through web search over the current
published pages. Where a page could not be read in full, the note says so.

## T4: MCP and A2A task states

### MCP tasks

- Statuses: `working`, `input_required`, `completed`, `failed`, `cancelled`.
- Terminal: `completed`, `failed`, `cancelled`. A terminal task never changes
  state again.
- `input_required` is a pause: the server needs something from the client and
  waits. It can go back to `working`.
- `completed` carries the result. `failed` carries an error. There is no
  separate state for "stopped by a limit".
- Standing: tasks entered the 2025-11-25 specification as experimental. Later
  pages describe them as a separate extension (SEP-2663, `ext-tasks`). The
  state names are the same in both. Treat MCP tasks as a moving target.

Sources: <https://modelcontextprotocol.io/specification/2025-11-25/basic/utilities/tasks>,
<https://modelcontextprotocol.io/seps/2663-tasks-extension>,
<https://modelcontextprotocol.io/extensions/tasks>.

### A2A tasks

- States: `submitted`, `working`, `input-required`, `auth-required`,
  `completed`, `failed`, `canceled`, `rejected` (plus `unknown` in some SDKs).
- Terminal: `completed`, `failed`, `canceled`, `rejected`.
- Interrupted (not terminal): `input-required`, `auth-required`.
- `rejected` means the agent refused the task before or instead of doing it,
  for example on a policy violation.
- Spelling differs from MCP: `input-required` and `canceled`.

Sources: SDK documentation that tracks A2A v1.0 section 5.5, for example
<https://docs.rs/crate/adk-server/1.0.0/source/src/a2a/v1/state_machine.rs> and
<https://docs.cloud.google.com/gemini/data-agents/reference/rest/Shared.Types/TaskState>.
The A2A specification page itself was not read in full; check it before 1.0.

### Mapping to gzyoke edition 3 outcomes

gzyoke edition 3 (approved 2026-11-27) ends a run with one of `done_proposed`,
`stopped` (with a stop reason), `blocked`, `cancelled` or `failed`.

| gzyoke outcome | MCP | A2A | Note |
| --- | --- | --- | --- |
| `done_proposed` | `completed` | `completed` | Ours is weaker on purpose: the model only proposes done; acceptance happens outside the run |
| `stopped` + reason | `failed` | `failed` | Neither protocol names a limit stop. Ours keeps it apart from errors |
| `blocked` | `input_required` | `input-required` / `auth-required` | Ours is terminal for the run; the caller starts a new run after a human decides |
| `cancelled` | `cancelled` | `canceled` | Same meaning |
| `failed` | `failed` | `failed` | Same meaning |
| (none) | (none) | `rejected` | Refusal before the run starts, for example an unattested cartridge |

Findings:

1. Both protocols describe a *live* task that a client polls. Our outcome is
   the *final* record of one bounded run. So the outcome needs only terminal
   values. Live states (`submitted`, `working`) belong to whatever runs the
   step (Prefect, gzfactory), not to this record.
2. Neither protocol separates "the agent says it is done" from "someone
   accepted it". The map's rule ("the model only proposes done") needs that
   separation, so adopting MCP's `completed` as is would lose it.
3. A2A's `rejected` covers a case gzyoke now folds into configuration errors:
   a run that never started. A cartridge loader that refuses an unattested
   cartridge needs a way to say so.

## T5: Oracle Open Agent Specification

What it is: a declarative, framework-neutral language for agents and flows,
serialized as JSON or YAML, with no executable code. Versions are published
per release (25.4.1, 26.1.0, 26.1.2).

Shape, as far as the published pages show:

- Every object is a `Component` with `id`, `name`, optional `description`
  and a type tag (`component_type` in the Python SDK).
- Components carry the range of spec versions they need
  (`min_agentspec_version`, `max_agentspec_version`).
- Components with inputs and outputs (`ComponentWithIO`) list each input and
  output as a JSON Schema value with a required `title` and `type`.
- Agents, flows, nodes, tools and LLM configs are kinds of component; other
  components are referenced by id.

What to reuse:

- **Typed inputs and outputs as JSON Schema values.** This matches our
  "typed inputs and outputs" part directly, and keeps us free of Pydantic.
- **A type tag plus a spec version on every document.** Good precedent for T2.
- **No executable code in a document.** Matches the sealed-cartridge idea.

What not to copy:

- Agent Spec describes *how an agent is built* (flows, nodes, LLM configs).
  A cartridge describes *one bounded unit of work* and its limits, effects,
  gates and evidence. Agent Spec has no limits, effect categories, human
  gates, attestation or failure handling. Those stay our own.
- Flows and nested components. A cartridge is atomic (Q3 settled), so no
  nesting.

Sources: <https://oracle.github.io/agent-spec/26.1.2/agentspec/language_spec_26_1_2.html>,
<https://oracle.github.io/agent-spec/26.1.2/api/components.html>,
<https://arxiv.org/html/2510.04173v2>. Exact field names were taken from the
SDK pages; read the language spec in full before mapping field by field.

## T6: OpenAPI, JSON Schema 2020-12 and OCI digests

- **OpenAPI** states its version in the document itself (`openapi: "3.1.0"`),
  with semver for the specification. Tools read that field first.
- **JSON Schema 2020-12** gives `$schema` and `$id`, `additionalProperties:
  false` and `unevaluatedProperties: false` for strictness, and `$defs` for
  shared parts. `format` is an annotation by default, so a validator must opt
  in to checking it, or the spec must say which formats are checked.
- **OCI digests** use the form `algorithm:encoded`, for example
  `sha256:` plus 64 lowercase hex characters. The digest is over the exact
  bytes stored. OCI says JSON content *should* be canonical, but it does not
  name a canonical form. RFC 8785 (JSON Canonicalization Scheme) is the
  standard choice when one is needed.

Sources: <https://github.com/opencontainers/image-spec/blob/v1.0/manifest.md>,
OCI `descriptor.md` and `considerations.md` (mirrors read, not upstream).
JSON Schema and OpenAPI facts are well established; re-read the current
pages before the 0.1 spec is frozen.
