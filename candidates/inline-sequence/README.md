# Inline Sequence Format Candidate

## Summary

The Inline Sequence Format represents magic as an ordered list of spells. Each
spell reads and writes an execution context, similar to a pipeline, command
buffer, or bytecode stream. This candidate favors predictable execution and a
small performer surface.

## Core concepts

- **Spell**: one operation in an ordered stream.
- **Magic**: a sequence of any number of spells plus initial context bindings.
- **Performer**: walks the sequence, updates context, and applies effects when
  effectful spells are encountered.

## Structural model

Each magic sequence should contain:

- `version` and optional composition name.
- `context`: named initial values such as caster, origin, target, power, seed,
  or world query handles.
- `spells`: ordered spell entries.
- `policies`: error handling, unsupported-spell behavior, determinism, and
  rollback expectations.
- `metadata`: authoring and compatibility data.

Each spell entry should contain:

- `label`: optional diagnostic label for tooling; generated spells must not require ids.
- `type`: namespaced operation.
- `reads`: context keys the spell expects.
- `writes`: context keys the spell creates or updates.
- `attributes`: parameters.
- `requirements`: performer capabilities.

## How it describes physical worlds

Physical effects are expressed as ordered operations over context:

1. query nearby bodies, cells, or fields;
2. filter by material, state, temperature, velocity, or tags;
3. derive quantities such as impulse, pressure, energy, or duration;
4. apply operations to the world through performer-defined capabilities;
5. store diagnostics or generated handles for later spells.

The same sequence can be interpreted by very different games because the format
names abstract operations and required capabilities instead of engine APIs.

## Extensibility

- New spell operations can be appended as namespaced types.
- Context values can be strongly typed by convention or by schema extensions.
- Optional labels and jumps may be introduced later for loops and branches.
- Performer policies define whether unknown operations fail, no-op, approximate,
  or defer to game plugins.

## Representation and interpreter

- **JSON representation**: an object with initial `context`, ordered `spells`,
  and execution `policies`. It should preserve source-friendly spell ids and
  context key names.
- **Dedicated tightened format**: a compact instruction stream with an interned
  operation table, typed literal pool, context slot table, and policy header.
  Reads and writes should reference context slots by integer id instead of name.
- **Interpreter output**: a `MagicProgram`-style structure containing ordered
  instructions, context slot descriptors, literal tables, resolved read/write
  sets, requirement sets, extension payloads, and diagnostics. A performer can
  execute or compile the program without parsing authoring data.

## Strengths

- Simple to parse, debug, replay, and record.
- Clear effect order.
- Easy to map to command buffers or scripting VMs.
- Good for deterministic networking if the performer enforces deterministic
  inputs.

## Tradeoffs

- Complex branching and parallelism are less natural than in graph formats.
- Shared computations require context discipline.
- Long sequences can become hard to author manually.

## Best use cases

- Runtime-friendly spell execution.
- Networked or replayable games.
- Games that want a minimal initial OGMR performer.

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
  "format": "inline_sequence",
  "entry_point": "sequence_start",
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
  "sequence": [
    {
      "spell": 0,
      "reads": [
        "caster",
        "target"
      ],
      "writes": [
        "heated_air",
        "lift_force"
      ]
    }
  ]
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

This magic performs a bounded Thermal Lift. The generated spell selects a target volume, adds no more than the declared heat limit, converts the heated air into an upward force, and removes the force after three seconds. The magic structure composes the spell by an ordered sequence entry that references the first spell structurally, so validation proves the performer can execute the spell without relying on a generated spell id.

## Implementability and extensibility check

This candidate is implementable when the interpreter can validate its topology, resolve structural spell references, check capability requirements, and produce a bounded performer input structure. It is extensible because spell types, attributes, requirements, generation metadata, and extension payloads are namespaced and can be preserved even when a performer only partially supports them.
