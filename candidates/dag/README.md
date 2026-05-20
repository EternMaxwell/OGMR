# DAG Magic Format Candidate

## Summary

The DAG Magic Format represents magic as a directed acyclic graph. Spell nodes
produce and consume abstract values such as targets, fields, materials, energy,
constraints, events, and world queries. Edges describe data flow or dependency
ordering.

## Core concepts

- **Spell**: a graph node with typed inputs, outputs, parameters, and
  requirements.
- **Magic**: a graph of spell nodes with no dependency cycles.
- **Performer**: resolves the graph, evaluates supported nodes, and applies the
  resulting effects to the host world.

## Structural model

Each magic graph should contain:

- `version` and optional composition name.
- `nodes`: spell definitions keyed by structural references.
- `edges`: connections from one node output to another node input.
- `inputs`: external values supplied by the game, caster, item, environment, or
  scripting layer.
- `outputs`: effects, diagnostics, or values exposed back to the performer.
- `requirements`: graph-level capabilities.
- `metadata`: editor and compatibility information.

Each spell node should declare:

- `type`: namespaced spell kind.
- `inputs`: named ports with expected abstract types.
- `outputs`: named ports with produced abstract types.
- `attributes`: constants, curves, policies, formulas, and tags.
- `execution`: hints such as pure query, effectful operation, deterministic
  operation, async operation, or approximation-safe operation.

## How it describes physical worlds

Physical concepts can be modeled as values flowing through the graph:

- target volumes, surfaces, particles, bodies, cells, joints, and fields;
- heat, pressure, mass, charge, velocity, acceleration, viscosity, stiffness,
  density, and gravity vectors;
- simulation operations such as sample, filter, integrate, constrain, convert,
  emit, destroy, and apply.

For example, a graph may sample a cone-shaped volume, filter water cells,
increase pressure, reduce temperature, and output a freezing operation. A simple
performer may turn that into a status effect, while a detailed performer may
modify fluid and thermal simulations directly.

## Extensibility

- New node types can be added without changing the graph container.
- New abstract port types can be namespaced.
- Subgraphs can be packaged as reusable spells.
- Performers can optimize pure subgraphs, cache queries, or parallelize
  independent branches.

## Representation and interpreter

- **JSON representation**: an object with `nodes`, `edges`, `inputs`, and
  `outputs`, using explicit node ids and named ports for clarity.
- **Dedicated tightened format**: a canonical graph table with interned node
  types, port names, value types, and edge endpoints encoded as integer indices.
  The tightened form should include a validated topological order so runtime
  loading does not need expensive graph analysis.
- **Interpreter output**: a `MagicGraph`-style structure containing node records,
  port descriptors, edge lists, adjacency tables, topological execution order,
  typed constants, requirement sets, extension payloads, and diagnostics. A
  performer receives an already-resolved acyclic graph.

## Strengths

- Excellent for reusable computation and shared dependencies.
- Good fit for visual node editors.
- Makes data dependencies explicit.
- Supports validation before runtime.

## Tradeoffs

- Less readable than a tree in raw text.
- Requires cycle detection and port type validation.
- Effect ordering must be explicit when multiple effectful nodes interact.

## Best use cases

- Complex magic with shared calculations.
- Simulation-heavy games where inputs and outputs need strong validation.
- Tooling that already uses graph or visual scripting concepts.

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
