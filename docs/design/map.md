# gz-cartridge design map

Status: open fog, 2026-11-26. Resolve these before drafting the 0.1 spec.
Each ticket is HITL (only the owner can settle it, in a live session) or AFK
(an agent can research it and report back). An agent never answers a HITL
ticket on the owner's behalf.

Process: resolve tickets in direct sessions in this repository, using
Superpowers brainstorming. Record each lasting choice as an ADR in
`docs/adr/`, then write one 0.1 design spec. Start `spec/` only after the
owner approves that design spec.

## Settled inputs

- Purpose, boundaries and design rules: see `README.md` and ADR 0001.
- Cartridge parts (13): purpose, rationale, typed inputs and outputs, tools
  and effects, limits and stop rules, human gates, evidence, failure handling
  (retry, escalate to the owner, or stop), plus the parts listed in the
  original cartridge brief. Confirm the full list in the first session.
- Authoring: a thin cartridge-writing skill with heavy tools; a human attests
  each version (Gate 5) before it can run.
- A cartridge is self-contained and can live in gzkit or gzfactory projects.
- Layers: gzcontractor (3) sequences cartridges, gzfactory (2) runs them,
  gzyoke or Pydantic AI (1) executes one.

## Tickets

| ID | Question | Mode | Blocks |
| --- | --- | --- | --- |
| Q3 | One cartridge per workflow step, or can a cartridge span steps or call others? | HITL | Schema shape |
| Q4 | Does a cartridge cite skills, or is it a kind of skill? | HITL | Relation to Agent Skills |
| Q6 | What does the name "cartridge" imply (swappable, sealed, versioned)? | HITL | Vocabulary, docs |
| T1 | Outcome states: adopt MCP task states as is, or extend with gzyoke's `blocked` and `limit_exceeded`? | HITL, after T4 | Outcome record |
| T2 | Versioning: semver for the spec, and how each document declares its version | HITL | Every document |
| T3 | Attestation: what exact bytes a Gate 5 digest covers, and its format | HITL | Gate 5 records |
| T4 | Survey MCP task states and A2A task states in depth; propose a mapping | AFK | T1 |
| T5 | Read Oracle Open Agent Spec closely; list what to map to or reuse | AFK | Schema shape |
| T6 | Check OpenAPI, JSON Schema 2020-12 and OCI digest practice (current docs) | AFK | T2, T3 |
| T7 | Mission and run-request contents | HITL, after gzcontractor design | Shared records |

## Sources

- Prior-art survey (2026-11-26), project doc "gz-cartridge prior art".
- Book-impact discussion, project doc "AI Agents in Action vs gzfactory and gzyoke".
