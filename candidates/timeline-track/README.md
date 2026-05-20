# Timeline Track Format Candidate

## Summary

This candidate represents magic through timed tracks, clips, curves, markers, and synchronization rules. It is intended to be serializable, procedurally generatable, interpretable into performer-facing data, and usable by games ranging from simple rule worlds to detailed physical simulations.

## Core concepts

- **Spell**: one declarative unit in the timed tracks, clips, curves, markers, and synchronization rules model, with typed attributes, requirements, validation bounds, metadata, and extension data.
- **Magic**: the complete timed tracks, clips, curves, markers, and synchronization rules composition containing many spells, entry points, policies, spell provenance summaries, and diagnostics.
- **Performer**: a game-provided runtime that consumes the interpreted magic structure and maps abstract effects to the concrete world.

## Structural model

Each spell structure contains `spell_type`, typed attributes, requirements, metadata, bounds, generator provenance, and extension payloads, but not a required id. Each magic structure contains `version`, the timed tracks, clips, curves, markers, and synchronization rules topology, entry points, policies, spell provenance summaries, global requirements, and validation diagnostics.

## How it describes physical worlds

The format describes targets, quantities, materials, fields, constraints, state, events, and operations abstractly. A performer can map the same magic to gameplay tags, entity components, rigid bodies, soft bodies, fluids, heat diffusion, gravity fields, falling-sand cells, chemistry, or other game-specific systems.

## Extensibility

- Namespaced spell and magic fields allow standard and game-specific vocabularies.
- Capability requirements allow rejection, approximation, fallback, or partial support.
- Extension payloads survive interpretation for tools and game plugins.
- Procedural spell-generation bounds keep generated spells safe for real-time worlds.

## Representation and interpreter

- **JSON representation**: a readable object containing the timed tracks, clips, curves, markers, and synchronization rules topology, spell structures, magic structure, requirements, policies, spell provenance summaries, and metadata.
- **Dedicated tightened format**: a compact canonical encoding with interned strings, typed attribute blocks, integer references, prevalidated topology tables, and bounded resource headers.
- **Interpreter output**: a `MagicTimelineTrack` structure containing resolved spells, candidate topology, lookup tables, typed attributes, requirements, spell provenance summaries, extension payloads, and diagnostics.

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
  "format": "timeline_track",
  "entry_point": "effect_track",
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
  "tracks": [
    {
      "name": "effect",
      "clips": [
        {
          "spell": 0,
          "start_seconds": 0,
          "duration_seconds": 3
        }
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

This magic performs a bounded Thermal Lift. The generated spell selects a target volume, adds no more than the declared heat limit, converts the heated air into an upward force, and removes the force after three seconds. The magic structure composes the spell by a timed clip on a local track, so validation proves the performer can execute the spell without relying on a generated spell id.

## Implementability and extensibility check

This candidate is implementable if its interpreter can validate the topology, enforce generation bounds, resolve structural spell references, and emit a deterministic performer input structure. It is extensible because new spell types, attributes, policies, and extension payloads can be added under namespaces without changing the performer boundary.

## Strengths

- Supports authored magic composed from generated or authored spells.
- Keeps spell format and magic format separate.
- Has JSON interchange and a tightened production representation.
- Gives performers resolved data instead of raw authoring syntax.

## Tradeoffs

- Requires candidate-specific validation.
- Generated spells need strict bounds.
- Advanced worlds may need game-specific extension vocabularies.

## Best use cases

- Games whose tooling naturally matches this composition model.
- Procedural spell-generation pipelines.
- Performers that want safe, resolved, capability-checked magic data.
