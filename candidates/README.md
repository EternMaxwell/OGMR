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
- [Layer Stack Format](layer-stack/README.md): ordered layers, masks, blends, overrides, and accumulators.
- [State Machine Format](state-machine/README.md): states, transitions, guards, actions, and lifecycle phases.
- [Grammar Template Format](grammar-template/README.md): procedural grammars and templates that expand into concrete spell structures.
- [Constraint Solver Format](constraint-solver/README.md): variables, constraints, objectives, bounds, and solver policies.
- [Field Network Format](field-network/README.md): sources, sinks, transforms, samplers, and spatial or abstract fields.
- [Timeline Track Format](timeline-track/README.md): timed tracks, clips, curves, markers, and synchronization rules.
- [Blackboard Format](blackboard/README.md): shared facts read and written by scheduled spell rules.
- [Entity Recipe Format](entity-recipe/README.md): entity creation, component mutation, links, and cleanup recipes.
- [Message Passing Format](message-passing/README.md): spell actors, channels, handlers, and typed messages.
- [Algebraic Expression Format](algebraic-expression/README.md): typed expressions, functions, formulas, and effect constructors.
- [Behavior Tree Format](behavior-tree/README.md): selectors, sequences, decorators, conditions, and action leaves.
- [Patch Delta Format](patch-delta/README.md): declarative patches that add, replace, scale, or remove world properties.

## Shared design goals

- Keep OGMR declarative and serializable.
- Spell procedural generation should be a first-class use case: generators create spell structures, while magic structures compose generated or authored spells. Generated spells must use structural position, local handles, or content-derived references instead of ids, and must include provenance, bounds, deterministic seeds where useful, and validation before performer use.
- Keep spell structures and magic structures separate: spells are reusable units of intent, while magic is the composition/container that arranges many spells.
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
- Use structural references, local handles, extensible metadata, namespaced attributes, and version
  fields so procedurally generated spells do not depend on ids and each candidate can evolve without breaking existing content.
- Represent physical-world concepts through abstract targets, quantities,
  constraints, fields, events, and capability requirements rather than through
  one fixed engine model.
- Make validation examples concrete: show a JSON spell representation, a JSON magic representation, performer pseudocode, and a description of what the magic does.

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
