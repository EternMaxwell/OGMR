# OGMR Format Candidates

This folder collects candidate plans for Open Game Magic Representation (OGMR)
formats. Each candidate keeps the same three core concepts:

- **Spell**: the smallest declarative unit of magic.
- **Magic**: a variable-size composition of spells.
- **Performer**: a game-provided runtime that accepts magic and applies effects
  to the game world. OGMR only describes inputs and semantics; it does not
  implement concrete performers.

The candidates are intentionally game-world-neutral. A performer may target a
simple RPG stat system, a deterministic simulation, or a fully physical world
with rigid bodies, soft bodies, fluids, heat, gravity, falling sand, chemistry,
or any other domain model.

## Candidates

- [Tree Magic Format](tree/README.md): hierarchical spell composition.
- [DAG Magic Format](dag/README.md): dependency graph composition.
- [Inline Sequence Format](inline-sequence/README.md): ordered stream/pipeline composition.
- [Attribute Component Format](attribute-component/README.md): data-oriented spell bundles.
- [Attachment Slot Format](attachment-slot/README.md): socketed spells attached to anchors.
- [Rule Reaction Format](rule-reaction/README.md): event/condition/action magic rules.

## Shared design goals

- Keep OGMR declarative and serializable.
- Make every candidate representable as JSON for tooling, interchange,
  debugging, and schema validation.
- Treat JSON as a portable representation, not necessarily the final runtime
  format. A production OGMR format should also define a dedicated tightened
  representation that is compact, canonical, fast to parse, and suitable for
  real game loading.
- Define an interpreter path for every candidate. The interpreter reads either
  JSON or the tightened representation, validates it, resolves references and
  extensions, and emits a performer-facing data structure.
- Avoid game-specific effect implementations in the format.
- Let performers advertise supported capabilities and reject, approximate, or
  partially apply unsupported spells.
- Use stable identifiers, extensible metadata, namespaced attributes, and version
  fields so each candidate can evolve without breaking existing content.
- Represent physical-world concepts through abstract targets, quantities,
  constraints, fields, events, and capability requirements rather than through
  one fixed engine model.

## Common representation pipeline

Each candidate should support three layers:

1. **Authoring/interchange JSON**: readable and schema-validatable form used by
   tools, editors, tests, examples, and source control.
2. **Dedicated tightened format**: final compact OGMR representation. It may be
   binary, tokenized text, or another canonical encoding, but it should preserve
   the same semantics as JSON while reducing ambiguity, parsing cost, and file
   size.
3. **Performer input structure**: an in-memory structure produced by an
   interpreter. The structure should hold resolved spells, composition shape,
   attributes, requirements, extension data, diagnostics, and any precomputed
   lookup tables the performer needs.

The interpreter is part of OGMR infrastructure, not a concrete game performer.
It should not apply effects to the world. Its job is to make magic data safe,
resolved, and convenient for game-specific performers.
