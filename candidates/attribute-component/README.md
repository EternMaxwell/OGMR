# Attribute Component Format Candidate

## Summary

The Attribute Component Format represents magic as a collection of spell
components attached to entities, scopes, or abstract records. Instead of
emphasizing order, it emphasizes data. A performer queries compatible components
and decides how to schedule or apply them.

## Core concepts

- **Spell**: a component or component bundle that declares intent and data.
- **Magic**: an entity-like record made from any number of spell components.
- **Performer**: selects supported components, resolves their interactions, and
  applies them to the game world.

## Structural model

Each magic record should contain:

- `version` and optional composition name.
- `components`: namespaced component objects.
- `relations`: optional links between components.
- `requirements`: capabilities needed by the whole record.
- `metadata`: editor, schema, and compatibility data.

Component examples:

- `ogmr.target.volume`
- `ogmr.material.filter`
- `ogmr.energy.thermal`
- `ogmr.force.vector`
- `ogmr.duration`
- `ogmr.condition.world_state`
- `ogmr.policy.fallback`

Each component should contain:

- `component_label`: optional authoring-only label, not used as the procedural spell identity.
- `attributes`: open data payload.
- `schema`: optional schema or version reference.
- `requirements`: local capability needs.

## How it describes physical worlds

A physically detailed magic record may contain components for target geometry,
material rules, energy transfer, field generation, constraints, and lifetime.
The performer can map those components onto a rigid-body world, a cellular
falling-sand world, a fluid solver, a heat diffusion model, or a high-level
gameplay approximation.

Because components are declarative, the same magic can be interpreted at
different fidelity levels. A performer with no fluid simulation might interpret
`ogmr.material.filter: water` as a gameplay target tag, while another performer
might resolve actual fluid cells.

## Extensibility

- Components are independently namespaced and versioned.
- Games can add custom component families without changing standard ones.
- Multiple schemas can coexist in one magic record.
- Performer capability negotiation can happen per component.

## Representation and interpreter

- **JSON representation**: an object with a `components` map or array, optional
  `relations`, and per-component schema/version information.
- **Dedicated tightened format**: a canonical component table with interned
  component types, schema ids, attribute names, relation endpoints, and compact
  typed attribute blocks. Components should be sorted by structural reference or type for
  deterministic loading.
- **Interpreter output**: a `MagicComponentSet`-style structure containing
  component records, type-indexed lookup tables, relation lists, typed
  attributes, requirement sets, extension payloads, and diagnostics. A performer
  can query components directly without caring about the serialized source.

## Strengths

- Very data-oriented and friendly to ECS-style engines.
- Good for partial support and graceful degradation.
- Easy to diff, merge, and patch.
- Flexible enough for many world models.

## Tradeoffs

- Execution order is implicit unless relations or policies define it.
- Interactions between components need clear performer rules.
- Less natural for spells that are inherently procedural.

## Best use cases

- ECS or data-driven games.
- Magic that should scale across different simulation fidelity levels.
- Content pipelines that prefer schemas and validation over control flow.

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
  "format": "attribute_component",
  "entry_point": "target_entity",
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
  "entities": [
    {
      "selector": "selected_volume",
      "components": [
        {
          "spell": 0,
          "component": "thermal_lift_effect"
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

This magic performs a bounded Thermal Lift. The generated spell selects a target volume, adds no more than the declared heat limit, converts the heated air into an upward force, and removes the force after three seconds. The magic structure composes the spell by a component attached to the selected target entity, so validation proves the performer can execute the spell without relying on a generated spell id.

## Implementability and extensibility check

This candidate is implementable when the interpreter can validate its topology, resolve structural spell references, check capability requirements, and produce a bounded performer input structure. It is extensible because spell types, attributes, requirements, generation metadata, and extension payloads are namespaced and can be preserved even when a performer only partially supports them.
