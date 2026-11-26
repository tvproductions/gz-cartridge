> Moved here on 2026-11-26 as historical input to the 0.1 spec. It was written
> for the gzfactory agent before gz-cartridge existed. Where it conflicts with
> ADR-0001 or `docs/design/map.md`, those win.

# Cartridge: brief for the gzfactory agent

Status: **draft 0.7, 2026-11-26, not approved.** Jeffry (the operator) coined
the term "cartridge" in conversations that are not recorded in this project.
This brief reconstructs the idea from gzfactory's and gzyoke's design records.
The tags below show how far you can rely on each statement:

- **[OPERATOR]**: Jeffry said it. You can rely on it.
- **[RECORD]**: an existing gzfactory or gzyoke document says it. The source is linked.
- **[GUESS]**: the drafter inferred it. Do not design or build on it until the
  operator confirms it. Section 7 lists the open questions.

Do not edit gzfactory product documents, the retained pueo documents, or gzyoke
on the strength of this brief. When the operator confirms the brief, record the
term in a design document or ADR through the normal skills-first design gate.

## 1. The term in one paragraph

A **cartridge** describes one agent's unit of work. **[OPERATOR]**
It is a self-contained work unit that could live inside gzkit or down inside
gzfactory. **[OPERATOR]** (2026-11-26) Its format must therefore not depend on
the layer that holds it. **[GUESS]** In the book's terms, gzyoke is layer 1,
gzfactory is layer 2, and a callable layer-3 facility above them is being
designed in another thread. **[OPERATOR]** A proposed reading **[GUESS]**:
layer 3 (or gzkit) selects or submits a cartridge, a gzfactory workflow step
loads it, and gzyoke runs it. The cartridge is a versioned package. **[GUESS]** The package states the task, the typed input
and output, the guidance the agent works under, the tools and effects that are
authorized, the limits, and the evidence that the run must produce. **[GUESS]**
The metaphor is a game console. The workflow and runtime are the console. They
supply power, control, safety, and recording. The cartridge supplies the
specific work. You can swap a cartridge without a change to the console. **[GUESS]**

## 2. Where a cartridge fits

| Layer | Owns | Relation to a cartridge |
| --- | --- | --- |
| gzkit | Governance facts and human acceptance | Optional. It can accept the evidence that a cartridge run produces. **[RECORD]** ([charter](https://github.com/tvproductions/gzyoke/blob/main/docs/charter.md): "GZKit governs, GZFactory orchestrates, GZYoke executes") |
| gzfactory workflow (Prefect) | Step order, retries, run identity, state | An agentic step names one cartridge, which it loads and runs. **[GUESS]** |
| gzfactory agentic step (Pydantic AI) | The agent loop, tools, model, typed result | The step receives the content of the cartridge as its task input, tools, limits, and output type. **[GUESS]** Pydantic AI as the executor is **[RECORD]** ([continuation design](https://github.com/tvproductions/gzfactory/blob/55a89854761fdf44b801912860c8247dea458d51/docs/superpowers/specs/2026-11-01-pueo-continuation-design.md)). |
| GovZero guidance | Guidance for governed agentic work | A cartridge references guidance packs by identity and version. It does not copy them. **[GUESS]** |
| gzyoke (owned inner loop) | One bounded model–tool–model run | The operator decided (2026-11-26) that gzfactory is the AFK engine: AFK sessions run as ordinary gzfactory workflows, and gzyoke executes inside each step. **[OPERATOR]** ([book comparison, choice B](https://claude.ai/artifact/GfLiP5ZxffGg597YfiWqGi)) So the step loads the cartridge and gzyoke can run it as its run request. **[GUESS]** How this relates to Pydantic AI as the executor is not yet reconciled. The gzyoke prototype still excludes gzfactory integration. **[RECORD]** |
| Skills | "How we work here, and why": norms and rationale | Section 5 gives the proposed relation. |

The unit a cartridge plugs into is a gzfactory workflow step. **[OPERATOR]**
Without gzkit, unattended (AFK) runs are diagnosis-only unless the workflow
carries the operator's explicit approval to make changes (gzkit ADR-0.0.62).
**[OPERATOR]** A cartridge's declared write or external effects therefore need
that approval to run AFK. **[GUESS]** An agent that chooses the next ticket is
deferred; if added, it is one bounded step that picks only AFK tickets.
**[OPERATOR]**

A cartridge is **not** a workflow. A workflow has many steps. A cartridge is the
work of one agentic step. **[GUESS]** A cartridge is also not a deterministic
handler. A deterministic step needs no cartridge. **[GUESS]**

## 3. Proposed contents (outline)

The operator accepted these eleven parts as a good start on 2026-11-26.
**[OPERATOR]** He then added items 12 and 13. **[OPERATOR]** Field names and
the format are still **[GUESS]**.

**Where the format lives (decided 2026-11-26).** The cartridge gets its own new
repository, `gz-cartridge`: a specification for an exchangeable agentic work
unit, written "in the spirit of Pydantic and Astral". It holds the schema as
plain versioned JSON and JSON Schema, so gzyoke can read it without Pydantic;
the shared records (mission, run request, outcome with stop reason); and a
reference validator. It depends on nothing. gzfactory, gzyoke, the layer-3
"gzcontractor", gzkit and gz-starter all consume it. Version 0.1 is shaped only
by real readers. **[OPERATOR]** The repository does not exist yet. gzfactory
may still wrap the schema in Pydantic models for its typed Python workflows,
but the JSON Schema is the authority. **[GUESS]** This list of parts is an
input to gz-cartridge v0.1, not its schema.

1. **Identity**: name, version, author, and content hash. The run record stores
   the hash.
2. **Purpose**: one sentence that states the job and the result that counts as done.
3. **Contract**: an input model and an output model. The output model is the
   Pydantic AI `output_type`.
4. **Instructions**: the task prompt for this unit of work only. It contains no
   general norms. General norms come from skills and guidance (item 5).
5. **Guidance references**: the skills and GovZero packs that apply, each with
   an identity and a version, and their precedence with operator and project
   rules. The precedence design is still open. **[RECORD]**
   ([pueo reset handoff](https://github.com/tvproductions/gzfactory/blob/55a89854761fdf44b801912860c8247dea458d51/docs/handoffs/2026-11-01-pueo-architecture-reset.md))
6. **Tools and effects**: the tools that the agent can call, the effect category
   of each tool (read, propose, write, external), and what needs human approval.
   The runtime enforces this list before dispatch. The cartridge only declares it.
7. **Capabilities required**: for example, reasoning or reliable tool calls.
   The executor or model is selected from this list. This field comes from
   pueo `Step.capabilities` **[RECORD]**
   ([pueo spec language](https://github.com/tvproductions/gzfactory/blob/55a89854761fdf44b801912860c8247dea458d51/docs/history/pueo/docs/spec-language.md)).
8. **Limits**: model requests, tool dispatches, wall time, and output size.
   They must be finite. gzyoke uses the same kind of limits. **[RECORD]**
   (gzyoke edition-2 minimal runtime draft, project file `gzyoke-edition-2/minimal-runtime.md`, not in a repository)
9. **Knowledge and memory**: the Graphify corpus that the agent can query, the
   Graphiti scope that it can read, and the memory writes that it can propose.
10. **Evidence and acceptance**: what the run must record (inputs, guidance
    identities, tool calls, trace, output) and the check that verifies the
    output. The record states whether a human must accept the result.
    Completed, verified, and accepted are three different states. **[RECORD]**
11. **Fixtures**: at least one sample input and its expected properties, so
    that a person can test a cartridge with a fake model before a real run. **[GUESS]**
12. **Rationale**: a short statement of why this unit of work exists. It
    matches the operator's view of skills as "how we work here, and why".
    **[OPERATOR]** It states the purpose of the work, not the norms. The norms
    stay in skills. **[GUESS]**
13. **Failure handling**: what happens when a limit is reached or a check
    fails: retry, escalate to the operator, or stop. **[OPERATOR]** Proposed
    rules **[GUESS]**: the cartridge declares the response for each failure
    kind, and the runtime applies it. A retry never repeats a completed or
    uncertain external effect. An escalation records a blocked fact and waits
    for a human decision; it never invents consent.

## 4. Load and run (proposed lifecycle)

Step 1 is **[OPERATOR]**. The other steps are **[GUESS]**.

1. **Author.** Cartridges are written with a cartridge-writing skill. A human
   reviews and attests each version at Gate 5 before it can run. The "thin
   skills, heavy tools" rule applies: the skill holds the norms and the
   rationale, and tools do the work of validating and minting. **[OPERATOR]**
   Proposed split **[GUESS]**: the skill says what a good cartridge is and why;
   tools scaffold the file, validate it (step 2), compute the content hash,
   and record the Gate 5 attestation. An unattested version cannot run.
2. **Register.** gzfactory discovers the cartridge and validates it: the
   references resolve, the tools exist, the limits are finite, and the input
   and output models import.
3. **Bind.** A workflow step names the cartridge by name and pinned version.
4. **Load.** At dispatch, the step resolves the version, loads the guidance
   that the cartridge references, and records the identities in the run before
   any model call.
5. **Run.** The executor inside the step (gzyoke, per the AFK decision, or Pydantic AI) runs the loop inside the limits. The
   runtime, not the cartridge, enforces tools and effects.
6. **Eject.** The step validates the typed output, stores the evidence, and
   returns the result to the workflow. A rejected output does not move the
   workflow forward.

## 5. Cartridge and skills: proposed division

Jeffry treats skills as a durable record of "how we work here, and why", not as
disposable how-to text. **[OPERATOR]** Guardrails belong in runtime
enforcement. Intent belongs in written artifacts and human gates. **[OPERATOR]**

Proposed rule **[GUESS]**:

- A **skill** holds norms and rationale that apply across many units of work.
  A skill outlives any single cartridge.
- A **cartridge** holds one unit of work. It *cites* skills and does not
  restate them. If a cartridge needs a norm that no skill records, write or
  change the skill. Do not put the norm in the cartridge.
- **Runtime policy** enforces the tool and effect limits that a cartridge
  declares. Text in a skill or cartridge is not enforcement.

Alternatives that the operator can select instead **[GUESS]**: (a) a cartridge
*is* a skill with an executable contract attached, or (b) a cartridge bundles
copies of its skills so that it is fully portable.

## 6. Rules for the gzfactory agent until the brief is confirmed

1. Use "cartridge" only in the sense of section 1. Do not apply the word to
   workflows, skills, plugins, or guidance packs.
2. Treat sections 3–5 as a proposal. The cartridge format belongs to
   `gz-cartridge`; do not define one in gzfactory. Do not create a `Cartridge`
   class or module before the operator approves a feature design.
3. When a design asks "what does one agentic step receive?", give the cartridge
   outline as a candidate answer and mark it unconfirmed.
4. Keep the boundaries that gzfactory already records: workflows are typed
   Python, Pydantic AI is the executor, the runtime enforces authority before
   dispatch, and human acceptance stays separate.
5. Build with the gz-skills-first starter. Shape the identity, evidence, and
   acceptance records so that they can map onto gzkit later. Do not make gzkit
   a dependency.

## 7. Open questions for the operator

1. **Contents.** Answered: the eleven fields are a good start, plus a
   rationale and failure handling (items 12 and 13).
2. **Authors.** Answered: a cartridge-writing skill, with human Gate 5
   review and attestation; thin skills, heavy tools. Open: may an agent run
   the skill unattended up to Gate 5, and which tools are first?
3. **Load and run.** Answered in part: a cartridge plugs into one gzfactory
   workflow step, and gzyoke executes inside the step. Open: does one step load
   exactly one cartridge? Can a
   cartridge contain several steps, or call other cartridges?
4. **Skills.** Is the division in section 5 correct (a cartridge cites skills),
   or is a cartridge a type of skill?
5. **Portability.** Answered by `gz-cartridge`: one dependency-free JSON
   Schema that every layer consumes. Earlier note: the cartridge is self-contained and can
   live in gzkit or gzfactory, so it must be portable across layers. Open: must the same cartridge run in both gzfactory (Pydantic AI)
   and gzyoke? Must it run on the Mac mini and on `tux`?
6. **Origin.** Does the name carry a specific meaning that this brief misses,
   for example hot-swap, sealed and read-only, or one per machine?
