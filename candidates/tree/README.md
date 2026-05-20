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
  "format": "tree",
  "entry_point": "root",
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
  "root": {
    "spell": 0,
    "children": []
  }
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

This magic performs a bounded Thermal Lift. The generated spell selects a target volume, adds no more than the declared heat limit, converts the heated air into an upward force, and removes the force after three seconds. The magic structure composes the spell by the root node of the spell tree, so validation proves the performer can execute the spell without relying on a generated spell id.

## Implementability and extensibility check

This candidate is implementable when the interpreter can validate its topology, resolve structural spell references, check capability requirements, and produce a bounded performer input structure. It is extensible because spell types, attributes, requirements, generation metadata, and extension payloads are namespaced and can be preserved even when a performer only partially supports them.
