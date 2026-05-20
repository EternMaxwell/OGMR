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

## Validation example

### JSON spell representation

```json
{
  "spell_type": "ogmr.thermal_lift",
  "attributes": {
    "target": "selected_volume",
    "heat_joules": 1200,
    "lift_newtons": 35,
    "duration_seconds": 3
  },
  "requirements": [
    "ogmr.capability.thermal",
    "ogmr.capability.force"
  ],
  "bounds": {
    "max_area_m2": 4,
    "max_duration_seconds": 3,
    "max_temperature_delta_c": 20,
    "max_force_newtons": 35
  },
  "provenance": {
    "generator": "example.validation",
    "seed": 4242
  }
}
```

### JSON magic representation

```json
{
  "version": 1,
  "format": "dag",
  "entry_point": "lift_force",
  "spells": [
    {
      "spell_type": "ogmr.thermal_lift",
      "attributes": {
        "target": "selected_volume",
        "heat_joules": 1200,
        "lift_newtons": 35,
        "duration_seconds": 3
      },
      "requirements": [
        "ogmr.capability.thermal",
        "ogmr.capability.force"
      ],
      "bounds": {
        "max_area_m2": 4,
        "max_duration_seconds": 3,
        "max_temperature_delta_c": 20,
        "max_force_newtons": 35
      },
      "provenance": {
        "generator": "example.validation",
        "seed": 4242
      }
    }
  ],
  "nodes": [
    {
      "spell": 0,
      "outputs": [
        "heated_air",
        "lift_force"
      ]
    }
  ],
  "edges": []
}
```

### Performer pseudocode

```text
function performThermalLift(magic, world):
    interpreted = interpreter.validateAndResolve(magic)
    spell = interpreted.spells[0]              # structural position, not an id
    requireCapability("ogmr.capability.thermal")
    requireCapability("ogmr.capability.force")
    target = world.resolveVolume(spell.attributes.target)
    assert target.area <= spell.bounds.max_area_m2
    heat = min(spell.attributes.heat_joules, heatForDelta(target, spell.bounds.max_temperature_delta_c))
    force = min(spell.attributes.lift_newtons, spell.bounds.max_force_newtons)
    world.addHeat(target, heat)
    world.applyForce(target, vector(0, force, 0), spell.attributes.duration_seconds)
    world.scheduleCleanup(target, spell.attributes.duration_seconds)
```

### What the magic does

This magic performs a bounded Thermal Lift. The generated spell selects a target volume, adds no more than the declared heat limit, converts the heated air into an upward force, and removes the force after three seconds. The magic structure composes the spell by a DAG node whose outputs are named by the topology, so validation proves the performer can execute the spell without relying on a generated spell id.

## Implementability and extensibility check

This candidate is implementable when the interpreter can validate its topology, resolve structural spell references, check capability requirements, and produce a bounded performer input structure. It is extensible because spell types, attributes, requirements, generation metadata, and extension payloads are namespaced and can be preserved even when a performer only partially supports them.
