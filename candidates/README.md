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
- Avoid game-specific effect implementations in the format.
- Let performers advertise supported capabilities and reject, approximate, or
  partially apply unsupported spells.
- Use stable identifiers, extensible metadata, namespaced attributes, and version
  fields so each candidate can evolve without breaking existing content.
- Represent physical-world concepts through abstract targets, quantities,
  constraints, fields, events, and capability requirements rather than through
  one fixed engine model.
