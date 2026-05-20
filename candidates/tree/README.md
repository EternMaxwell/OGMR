# Tree Magic Format Candidate

## Summary

The Tree Magic Format represents magic as a rooted tree. Every node is a spell,
and parent-child relationships describe containment, specialization, sequencing,
or control flow. This is the most readable candidate for authored magic because
it mirrors common design concepts such as "cast fire" containing "select area",
"add heat", and "push air".

## Core concepts

- **Spell**: a node with a type, parameters, optional requirements, and metadata.
- **Magic**: a root spell plus any number of descendant spells.
- **Performer**: traverses the tree, interprets each node according to its
  declared role, and maps abstract effects to the host game's simulation.

## Structural model

Each spell node should contain:

- `id`: stable local identifier.
- `type`: namespaced spell kind such as `ogmr.force`, `ogmr.thermal.add`, or
  `game.example.spawn_particle`.
- `role`: optional relationship hint such as `scope`, `modifier`, `condition`,
  `effect`, `duration`, or `fallback`.
- `attributes`: open key-value bag for scalar values, vectors, curves, formulas,
  tags, and references.
- `children`: nested spells.
- `requirements`: performer capabilities needed to evaluate the node.
- `metadata`: authoring, versioning, documentation, and tool hints.

## How it describes physical worlds

The tree does not assume a specific physics engine. A spell can declare abstract
operations such as:

- add or remove heat from a target volume;
- apply force, impulse, torque, pressure, or field gradients;
- transform material state such as phase, density, viscosity, elasticity, or
  fracture threshold;
- emit particles, fluids, rigid bodies, soft bodies, or constraints;
- sample world state such as gravity, temperature, velocity, contact, and
  containment.

The performer decides whether those operations affect a detailed simulation,
gameplay stats, visual-only effects, or another world model.

## Extensibility

- Namespaced `type` values allow third-party spell vocabularies.
- Unknown child roles can be skipped, preserved, or passed to plugins.
- Capability requirements let performers negotiate support.
- Metadata can include editor-only hints without changing runtime semantics.

## Representation and interpreter

- **JSON representation**: a nested object where each spell node owns its
  `children` array. This should be the primary source-control and editor format.
- **Dedicated tightened format**: a canonical pre-order node stream with compact
  symbol tables for spell types, roles, attribute names, and requirement names.
  Each node can store parent index, first-child index, child count, and attribute
  range so loaders do not need recursive parsing.
- **Interpreter output**: a `MagicTree`-style structure containing the root node
  index, a flat node array, resolved child ranges, interned strings, typed
  attributes, requirement sets, extension payloads, and diagnostics. A performer
  can traverse this structure without knowing whether the source was JSON or the
  tightened format.

## Strengths

- Easy for humans and tools to read.
- Natural fit for nested scopes, modifiers, and fallback branches.
- Simple to serialize as JSON, YAML, TOML, XML, or a custom text format.

## Tradeoffs

- Shared dependencies are duplicated unless references are added.
- Cross-branch interactions can become hard to reason about.
- Parallel execution requires explicit role or dependency rules.

## Best use cases

- Authored spells and editor-friendly magic definitions.
- Magic with clear nested structure.
- Games that need simple performer implementations first, with room to add more
  advanced semantics later.
