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

- optional authoring-only label: if present, used only by tools and never required for generated spells.
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

## Procedural generation

This candidate supports procedural spell generation. Generators create spell structures from templates, random seeds, player choices, world queries, or authored rule sets, then a separately authored or interpreted magic structure composes those generated spells. Generated spells must not rely on ids as usable fields; composition uses structural position, local handles, content-derived references, or candidate-specific links. Generated spells must preserve provenance, deterministic seeds when useful, version fields, capability requirements, and hard bounds for area, duration, intensity, recursion, spawned objects, and simulation cost.

## Separate spell and magic structures

Spell and magic are different structures.

- **Spell structure**: one reusable or procedurally generated unit of intent with `spell_type`, typed attributes, declared inputs/outputs when relevant, requirements, metadata, validation bounds, generator provenance, and extension payloads. It must not require a usable id.
- **Magic structure**: a composition/container with format version, the candidate-specific topology, entry points, performer policies, global requirements, and interpreter diagnostics. It may name the composition, but spell addressing inside it must use topology/local handles rather than generated spell ids.

The interpreter keeps this separation while resolving many spells into one performer-facing magic object. It validates and lowers data, but it does not apply effects to a game world.

## Quick validation examples

| Magic | Behavior | Example implementable performer logic | Implementability / extensibility check |
| --- | --- | --- | --- |
| Thermal Lift | Select a target volume, add bounded heat, convert part of that energy into upward pressure, and expire before overheating. | A performer validates thermal and force capabilities, resolves the target, applies capped heat/impulse, or approximates with a lift status. | Implementable because bounds and capabilities are explicit; extensible with new heat models and pressure solvers. |
| Water Wall | Gather nearby water or water-tagged entities, shape them into a barrier, increase cohesion, and release them after duration. | A performer maps selection to fluid cells, particles, or gameplay entities, then applies cohesion and collision rules. | Implementable at many simulation fidelities; extensible with material filters and wall-shape plugins. |
| Stone Brace | Target a body or surface, increase stiffness and fracture threshold, add mass penalty, and cleanly restore original values. | A performer stores original material data, applies bounded modifiers, and restores or blends them on expiry. | Implementable for stats, rigid bodies, or soft bodies; extensible with more material attributes. |
| Gravity Snare | Create a local gravity field that pulls eligible targets toward an anchor while excluding allies and clamping acceleration. | A performer samples affected bodies each tick and applies clamped impulses according to performer physics rules. | Implementable because target masks and clamps are declared; extensible with custom field equations. |
| Soft Repair | Find damaged soft or living material, spend available energy, restore integrity gradually, and stop at a safety threshold. | A performer maps repair to health, tissue, mesh constraints, or soft-body coefficients depending on game systems. | Implementable without mandating one health model; extensible with domain-specific repair vocabularies. |

## Implementability and extensibility check

This candidate is implementable when the interpreter can validate its topology, resolve structural spell references, check capability requirements, and produce a bounded performer input structure. It is extensible because spell types, attributes, requirements, generation metadata, and extension payloads are namespaced and can be preserved even when a performer only partially supports them.
