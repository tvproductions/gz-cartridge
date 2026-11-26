# gz-cartridge

An open spec for exchangeable agentic work units: what a unit does, its inputs
and outputs, limits, effects, human gates and evidence. JSON Schema, strict by
default, with a fast reference validator.

Status: **pre-0.1, spec-first.** Nothing here is a released format yet.

## What a cartridge is

A cartridge is one self-contained unit of agentic work. It declares its
purpose and rationale, typed inputs and outputs, the tools and effects it may
use, its limits and stop rules, where a human must approve, the evidence it
must produce, and what happens when it fails.

The spec is a contract, not a runtime. On its own it executes nothing. Its
value is that every layer agrees on it:

| Reader | Role |
| --- | --- |
| gzcontractor (layer 3) | Chooses and sequences cartridges for a mission |
| gzfactory (layer 2) | Runs cartridges inside workflow steps |
| gzyoke or Pydantic AI (layer 1) | Executes the agent loop for one cartridge |
| gzkit, gz-starter skills | Author cartridges; a human attests each version before it runs |

## Design rules

1. Strict by default: unknown fields are rejected.
2. Errors that teach: each names the path, what was expected, what was found
   and how to fix it.
3. One schema, many bindings: JSON Schema 2020-12 is the source of truth;
   language bindings (for example Pydantic models) are generated from it.
4. Fast and tiny: the reference validator has near-zero runtime dependencies
   and runs as one command.
5. Docs come from the schema, so they cannot drift.
6. Versioned and attestable: every document states its spec version, and an
   attested cartridge version is identified by a content digest.

## Layout

- `spec/` - the written specification (prose, normative)
- `schema/` - JSON Schema files generated from or checked against the spec
- `docs/adr/` - architectural decisions

## Prior art

Agent Skills (SKILL.md), the Model Context Protocol and its tasks extension,
A2A agent cards and task states, Oracle's Open Agent Specification, OpenAPI,
JSON Schema and the OCI image spec. Each covers a slice; none describes limits,
effects, human gates, evidence, rationale and failure handling together.

## Development

Built with [gz-starter](https://github.com/tvproductions/gz-starter) (see
`AGENTS.md`), aligned with gzkit for later adoption.
