# Project agent instructions

Read [AGENTS.skills-first.md](AGENTS.skills-first.md) for this project's development stack.
Add project-owned rules, verification commands, and operational skill routing here.

## Project rules

- This repository is a specification. Spec prose in `spec/` is normative; schema files in `schema/` must agree with it.
- Write spec before code. Do not add validator code until the 0.1 spec draft has been reviewed by the owner.
- Use JSON Schema 2020-12. Reject unknown fields by default.
- Keep runtime dependencies near zero. Every layer imports this package.
- Record decisions in `docs/adr/` as `ADR-NNNN-EEE-title.md` (decision id, edition, title), with the metadata the gz-starter ADR policy requires. Leave approval fields empty and the state Proposed until the owner approves that edition.
- Open design questions live in `docs/design/map.md`. Read it before design work; do not answer HITL tickets for the owner.
