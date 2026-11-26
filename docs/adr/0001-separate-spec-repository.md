# ADR 0001: The cartridge spec lives in its own repository

Status: accepted by the owner on 2026-11-26 in the gzfactory project
discussion of *AI Agents in Action* (2nd ed.). Approval of this written
edition is pending review of the pull request that adds it.

## Context

The book's three loop layers map onto the project family: gzyoke is the inner
loop (layer 1), gzfactory the task loop (layer 2) and gzcontractor the meta
loop (layer 3). A cartridge is the self-contained work unit that layer 3
sequences, layer 2 runs and layer 1 executes. gzkit and gz-starter projects
author cartridges.

A contract read by all of these should belong to none of them. If gzfactory
owned it, gzyoke and gzkit would depend on gzfactory only to read a cartridge.

## Decision

Keep the cartridge specification, its JSON Schema, the shared records the
layers exchange (mission, run request, outcome with its stop reason) and a
reference validator in this repository. It depends on no layer. Every layer
and harness may depend on it.

## Consequences

- The schema is plain versioned JSON Schema, so gzyoke can read it without
  Pydantic, while gzfactory can use generated Pydantic models.
- Version 0.1 is shaped only by real readers: gzfactory, gzyoke and the
  cartridge-writing skill. A universal work unit is the aim, not the first
  release.
- Spec work precedes code. No validator code is written until the 0.1 spec
  draft is reviewed.
- Each cartridge version must pass human Gate 5 attestation before it can run;
  the spec records what is attested, not who runs the gate.

## Reversal

Only an explicit owner decision changes this. Folding the spec into another
repository would need evidence that the separate repository costs more than
the independence it buys.
