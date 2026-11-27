# gz-cartridge design map

Status: open fog, 2026-11-26; Q3, Q4 and Q6 settled by the owner 2026-11-27.
T4 to T6 researched 2026-11-27 ([research-t4-t5-t6.md](research-t4-t5-t6.md)).
The 0.1 records design spec, with proposals for T1 to T3, is
[records-0.1.md](records-0.1.md) and awaits the owner's review.
Each ticket is HITL (only the owner can settle it, in a live session) or AFK
(an agent can research it and report back). An agent never answers a HITL
ticket on the owner's behalf.

Process: resolve tickets in direct sessions in this repository, using
Superpowers brainstorming. Record each lasting choice as an ADR in
`docs/adr/`, then write one 0.1 design spec. Start `spec/` only after the
owner approves that design spec.

## Settled inputs

- Purpose, boundaries and design rules: see `README.md` and ADR-0001.
- Cartridge parts (13): purpose, rationale, typed inputs and outputs, tools
  and effects, limits and stop rules, human gates, evidence, failure handling
  (retry, escalate to the owner, or stop), plus the parts listed in the
  original cartridge brief
  (`docs/design/cartridge-brief.md`, draft 0.7, historical input). Confirm the full list in the first session.
- Authoring: a thin cartridge-writing skill with heavy tools; a human attests
  each version (Gate 5) before it can run. An agent can likely run the skill
  unattended up to Gate 5. The skill needs at least one evaluation case.
  (Owner, 2026-11-27.)
- A cartridge is self-contained and can live in gzkit or gzfactory projects.
- Layers: gzcontractor (3) sequences cartridges, gzfactory (2) runs them,
  gzyoke or Pydantic AI (1) executes one.
- Granularity (was Q3): a cartridge is likely atomic, the baseline work unit,
  and likely matches gzkit's Task. One cartridge is one workflow step; it does
  not span steps or call other cartridges. (Owner, 2026-11-27.)
- gzkit terms (Operator, 2026-11-27): a cartridge has a TASK or TASKs, and
  each TASK is connected to either a REQ or a FIX. gz-cartridge aligns with
  gzkit's terms. In gzkit, a REQ is a catalog requirement
  (`REQ-<semver>-<obpi>-<criterion>`), a FIX is a repair requirement (its own
  `FIX-<n>` series, living in a repair order), and a TASK is a subdivision of
  labor under one requirement (`TASK-<semver>-<obpi>-<req>-<seq>`). This
  revises the earlier "cartridge likely matches gzkit's Task": a cartridge
  carries TASKs rather than being one. See T9.
- Filled form (Operator, 2026-11-27): a cartridge is reusable only as a
  structure. Each cartridge is a "filled form" naming its own TASKs and their
  REQ or FIX, and its execution leaves a receipt. So the cartridge itself
  names its TASK and REQ/FIX ids (answers the first half of T9). Open
  follow-ups: does a filled cartridge replace the run request (one form in,
  one receipt out)? Does Gate 5 attest each filled form, or the structure it
  was filled from?
- Skills (was Q4): a cartridge cites skills; it is not a kind of skill. GovZero's
  rule is thin skills, fat tools. (Owner, 2026-11-27.)
- Name (was Q6): a cartridge is both hot-swappable and sealed. The idea is close
  to containerization, applied to bounding a work unit. (Owner, 2026-11-27.)
- Machines: the Mac mini runs local models; `tux` might orchestrate
  (gzcontractor and/or gzfactory). A cartridge never names a machine. The run request
  names the model; a cartridge may list the models it was shaped for, and then
  the run request must pick one (Operator, 2026-11-27). (Owner, 2026-11-27.)

## Tickets

| ID | Question | Mode | Blocks |
| --- | --- | --- | --- |
| T1 | Outcome states: adopt MCP task states as is, or extend with gzyoke's `blocked` and `limit_exceeded`? | HITL, settled 2026-11-27: gzyoke edition 3's five plus `rejected` (records-0.1.md section 5); ADR to follow | Outcome record |
| T2 | Versioning: semver for the spec, and how each document declares its version | HITL, settled 2026-11-27: `spec_version` (semver) and `kind` in every document; breaking minor bumps allowed before 1.0 (records-0.1.md section 2); ADR to follow | Every document |
| T3 | Attestation: what exact bytes a Gate 5 digest covers, and its format | HITL, settled. Digest settled 2026-11-27: `sha256:<hex>` over RFC 8785 canonical JSON of the whole cartridge (records-0.1.md section 6); minimal attestation record (digest, role, time, note; no signatures) settled the same day; ADR to follow | Gate 5 records |
| T4 | Survey MCP task states and A2A task states in depth; propose a mapping | AFK, done 2026-11-27 | T1 |
| T5 | Read Oracle Open Agent Spec closely; list what to map to or reuse | AFK, done 2026-11-27 | Schema shape |
| T6 | Check OpenAPI, JSON Schema 2020-12 and OCI digest practice (current docs) | AFK, done 2026-11-27 | T2, T3 |
| T8 | Pydantic in the hex core: gzkit's kernel uses frozen Pydantic models; the Operator's recent direction keeps Pydantic out of the core. Settle across repos; gz-cartridge is provisionally stdlib-only in the core | HITL, cross-repo | Validator ADR |
| T9 | TASK binding: does a cartridge name its TASK and REQ/FIX ids itself, or does the run request bind a reusable cartridge to them? And is the record still a "run request" under gzkit terms? | HITL; first half settled 2026-11-27 (the filled cartridge names them); run request and Gate 5 follow-ups open | Cartridge and run request records |
| T7 | Mission and run-request contents (run request drafted in records-0.1.md; mission stays out of 0.1) | HITL, after gzcontractor design | Shared records |

## Sources

- Cartridge brief, draft 0.7 (2026-11-26): `docs/design/cartridge-brief.md`.
  Written for gzfactory before this repository existed; its [GUESS] items
  are not settled, and the tickets above supersede its open questions.

- Prior-art survey (2026-11-26), project doc "gz-cartridge prior art".
- Book-impact discussion, project doc "AI Agents in Action vs gzfactory and gzyoke".
