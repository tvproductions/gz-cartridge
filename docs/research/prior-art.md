> Moved into the repository on 2026-11-28 from the project document "gz-cartridge
> prior art" (written 2026-11-26, before this repository existed). Kept as the
> survey behind ADR-0001 and the design map; later decisions live there.

# gz-cartridge prior art

Nov 26, 2026 · Operator

No existing spec covers a whole agentic work unit. Each one describes a slice: skills describe instructions, MCP describes tools and long tasks, A2A describes agents talking, and Open Agent Spec describes agents and flows. gz-cartridge can reuse their best parts, especially JSON Schema 2020-12 and a shared task-state vocabulary, and add what none of them has: limits, effects, human gates, evidence and a rationale.

## Specs compared

The first four rows come from the current spec pages, read today. The OpenAPI, JSON Schema and OCI rows are from memory and need checking.

| Spec | What it defines | Shape and versioning | Take for gz-cartridge | Leave |
| --- | --- | --- | --- | --- |
| [Agent Skills](https://agentskills.io/specification.md) | A skill: a folder with `SKILL.md` (YAML frontmatter plus Markdown), optional scripts and references | Fields `name`, `description`, `license`, `compatibility`, `metadata`, experimental `allowed-tools`. No version field; validated by `skills-ref validate` | Folder-per-unit layout, a strict name rule, progressive disclosure (metadata first, body on use), and a one-command validator. A cartridge should cite skills, not restate them | No declared inputs or outputs, no limits, no approval |
| [MCP](https://modelcontextprotocol.io/specification/2025-11-25/changelog.md) | Tools, resources and prompts between a client and a server | Date-based versions. JSON Schema 2020-12 is the default dialect since 2025-11-25 | JSON Schema 2020-12 for inputs and outputs. Date or semver versioning stated in every document | Transport and session details |
| [MCP tasks extension](https://modelcontextprotocol.io/seps/2663-tasks-extension.md) | Long-running work a client polls | States `working`, `input_required`, `completed`, `cancelled`, `failed`; fields `taskId`, `status`, `statusMessage`, `ttlMs`, `pollIntervalMs` | A ready vocabulary for run outcomes. `input_required` is close to a human gate | Polling mechanics |
| [A2A v1.0](https://aaif.io/blog/a2a-v1-0-a-builder-s-guide-part-1-discovery-tasks-and-clients) | Agents finding and calling each other | Agent card at `/.well-known/agent-card.json`; version sent in an `A2A-Version` header | Its nine task states include `AUTH_REQUIRED` and `REJECTED`. A cartridge's "what I can do" summary could map to an agent card skill, so gzcontractor could publish its catalog this way later | Network discovery for v0.1 |
| [Open Agent Spec](https://arxiv.org/html/2510.04173v4) (Oracle) | Portable agents, tools and flows of nodes with control and data edges | JSON or YAML, a `component_type` on every part, a declared spec version, JSON Schema inputs and outputs, `$component_ref` links | Closest prior art. Typed components with a type tag, references by id, and parsers that check declared against actual schemas | Flows and graphs belong to gzfactory and gzcontractor, not to the unit |
| OpenAPI (from memory) | HTTP APIs | One document, a stated `openapi` version, JSON Schema for bodies | Write the spec as one readable document with examples, plus generated reference docs | HTTP specifics |
| JSON Schema (from memory) | Data shapes | Dialect URI in `$schema` | Use it as the schema language rather than inventing one. gzyoke reads it as plain JSON | Nothing |
| OCI image spec (from memory) | Container images | Content-addressed by digest, with a manifest | Identify each attested cartridge version by a content digest, so Gate 5 attests exact bytes | Layers and registries |

**What none of them covers:** limits and stop rules, declared effects with their risk class, human gates, required evidence, rationale and failure handling. Those are your 13 parts that make a cartridge different, and they are the core of v0.1.

## In the spirit of Pydantic and Astral

The spec is plain JSON Schema so any language can read it. The Pydantic and Astral spirit shows up in how it feels to use. These are my reading of what both teams are known for, offered as design rules for you to accept or change.

1. **Strict by default.** Unknown fields are rejected, as gzyoke already does for tool arguments. Loosening is an explicit, versioned choice.
2. **Errors that teach.** Every validation error names the exact path, what was expected, what was found and how to fix it, the way Pydantic errors read.
3. **One schema, many bindings.** JSON Schema is the source of truth. An optional Python package generates Pydantic models from it, so gzfactory gets Pydantic and gzyoke can stay on plain dataclasses.
4. **Fast and tiny.** The reference validator has almost no dependencies, installs with `uv`, and runs as one command, `gz-cartridge check <path>`, in milliseconds. That is the Astral habit of tools quick enough to run on every save.
5. **Docs are part of the product.** Every field has a description and an example, and the spec site is generated from the schema so the two never drift.
6. **Versioned and attestable.** Each document declares its spec version. Each attested cartridge version has a content digest, so a Gate 5 attestation points at exact bytes.

## Open questions

- [ ] Should run outcomes adopt the MCP task states as is, or extend them with `blocked` and `limit_exceeded` from gzyoke?
- [ ] Should a cartridge's public summary be shaped so it can later be published as an A2A agent-card skill?
- [ ] Does Open Agent Spec deserve a closer read, to map to it or reuse parts, before v0.1 is drafted?
- [ ] Still open in the cartridge thread: one cartridge per step or composable (Q3), cites skills or is a skill (Q4), and what the name implies (Q6).

The repo does not exist yet. You'd create `tvproductions/gz-cartridge` on GitHub, since this session can't create repos.
