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

- `magic_id` and `version`.
- `nodes`: spell definitions keyed by stable ids.
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
