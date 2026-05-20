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

## Validation examples

Each candidate includes at least five concrete validation examples. Every example separates the generated spell data from the candidate-specific magic container, shows performer-facing pseudocode, and describes the intended runtime behavior.

### Example 1: Thermal Lift

#### JSON spell representation

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

#### JSON magic representation

```json
{
  "version": 1,
  "format": "tree",
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
  "entry_point": "root",
  "root": {
    "spell": 0,
    "children": []
  }
}
```

#### Performer pseudocode

```text
function performThermalLift(magic, world, caster):
    interpreted = interpreter.validateAndResolve(magic)
    spell = interpreted.spells[0]  # structural position, not a generated id
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

#### What the magic does

The spell selects a target volume, adds bounded heat, converts the heated air into upward force, and removes the force after the declared duration. In this candidate, the magic composes the spell through a root tree node, so validation can prove the performer has the required capabilities, bounded parameters, and resolvable local structure before runtime effects are applied.

### Example 2: Water Wall

#### JSON spell representation

```json
{
  "spell_type": "ogmr.water_wall",
  "attributes": {
    "source": "nearby_water",
    "shape": "wall",
    "length_m": 5,
    "height_m": 2,
    "duration_seconds": 6
  },
  "requirements": [
    "ogmr.capability.material.water",
    "ogmr.capability.shape_barrier"
  ],
  "bounds": {
    "max_volume_liters": 800,
    "max_length_m": 5,
    "max_height_m": 2,
    "max_duration_seconds": 6
  },
  "provenance": {
    "generator": "example.validation",
    "seed": 5150
  }
}
```

#### JSON magic representation

```json
{
  "version": 1,
  "format": "tree",
  "spells": [
    {
      "spell_type": "ogmr.water_wall",
      "attributes": {
        "source": "nearby_water",
        "shape": "wall",
        "length_m": 5,
        "height_m": 2,
        "duration_seconds": 6
      },
      "requirements": [
        "ogmr.capability.material.water",
        "ogmr.capability.shape_barrier"
      ],
      "bounds": {
        "max_volume_liters": 800,
        "max_length_m": 5,
        "max_height_m": 2,
        "max_duration_seconds": 6
      },
      "provenance": {
        "generator": "example.validation",
        "seed": 5150
      }
    }
  ],
  "entry_point": "root",
  "root": {
    "spell": 0,
    "children": []
  }
}
```

#### Performer pseudocode

```text
function performWaterWall(magic, world, caster):
    interpreted = interpreter.validateAndResolve(magic)
    spell = interpreted.spells[0]  # structural position, not a generated id
    requireCapability("ogmr.capability.material.water")
    requireCapability("ogmr.capability.shape_barrier")
    water = world.collectMaterial(spell.attributes.source, spell.bounds.max_volume_liters)
    assert spell.attributes.length_m <= spell.bounds.max_length_m
    assert spell.attributes.height_m <= spell.bounds.max_height_m
    barrier = world.shapeBarrier(water, spell.attributes.shape, spell.attributes.length_m, spell.attributes.height_m)
    world.addCohesion(barrier, until=spell.attributes.duration_seconds)
    world.scheduleRelease(barrier, spell.attributes.duration_seconds)
```

#### What the magic does

The spell gathers available water or water-tagged entities, shapes them into a bounded barrier, increases cohesion, and releases the water when the duration expires. In this candidate, the magic composes the spell through a root tree node, so validation can prove the performer has the required capabilities, bounded parameters, and resolvable local structure before runtime effects are applied.

### Example 3: Stone Brace

#### JSON spell representation

```json
{
  "spell_type": "ogmr.stone_brace",
  "attributes": {
    "target": "touched_structure",
    "stiffness_multiplier": 1.5,
    "fracture_bonus": 0.25,
    "duration_seconds": 8
  },
  "requirements": [
    "ogmr.capability.material.stone",
    "ogmr.capability.modify_structure"
  ],
  "bounds": {
    "max_mass_kg": 2000,
    "max_stiffness_multiplier": 1.5,
    "max_fracture_bonus": 0.25,
    "max_duration_seconds": 8
  },
  "provenance": {
    "generator": "example.validation",
    "seed": 6161
  }
}
```

#### JSON magic representation

```json
{
  "version": 1,
  "format": "tree",
  "spells": [
    {
      "spell_type": "ogmr.stone_brace",
      "attributes": {
        "target": "touched_structure",
        "stiffness_multiplier": 1.5,
        "fracture_bonus": 0.25,
        "duration_seconds": 8
      },
      "requirements": [
        "ogmr.capability.material.stone",
        "ogmr.capability.modify_structure"
      ],
      "bounds": {
        "max_mass_kg": 2000,
        "max_stiffness_multiplier": 1.5,
        "max_fracture_bonus": 0.25,
        "max_duration_seconds": 8
      },
      "provenance": {
        "generator": "example.validation",
        "seed": 6161
      }
    }
  ],
  "entry_point": "root",
  "root": {
    "spell": 0,
    "children": []
  }
}
```

#### Performer pseudocode

```text
function performStoneBrace(magic, world, caster):
    interpreted = interpreter.validateAndResolve(magic)
    spell = interpreted.spells[0]  # structural position, not a generated id
    requireCapability("ogmr.capability.material.stone")
    requireCapability("ogmr.capability.modify_structure")
    structure = world.resolveStructure(spell.attributes.target)
    assert structure.mass <= spell.bounds.max_mass_kg
    original = world.snapshotMaterial(structure)
    world.scaleStiffness(structure, min(spell.attributes.stiffness_multiplier, spell.bounds.max_stiffness_multiplier))
    world.addFractureThreshold(structure, min(spell.attributes.fracture_bonus, spell.bounds.max_fracture_bonus))
    world.restoreMaterial(structure, original, after=spell.attributes.duration_seconds)
```

#### What the magic does

The spell reinforces a touched stone or structure by bounded stiffness and fracture modifiers, records the original material state, and restores it after expiry. In this candidate, the magic composes the spell through a root tree node, so validation can prove the performer has the required capabilities, bounded parameters, and resolvable local structure before runtime effects are applied.

### Example 4: Gravity Snare

#### JSON spell representation

```json
{
  "spell_type": "ogmr.gravity_snare",
  "attributes": {
    "anchor": "caster_focus",
    "radius_m": 6,
    "pull_newtons": 40,
    "duration_seconds": 4,
    "exclude_tags": [
      "ally"
    ]
  },
  "requirements": [
    "ogmr.capability.gravity_field",
    "ogmr.capability.target_filter"
  ],
  "bounds": {
    "max_radius_m": 6,
    "max_pull_newtons": 40,
    "max_targets": 10,
    "max_duration_seconds": 4
  },
  "provenance": {
    "generator": "example.validation",
    "seed": 7171
  }
}
```

#### JSON magic representation

```json
{
  "version": 1,
  "format": "tree",
  "spells": [
    {
      "spell_type": "ogmr.gravity_snare",
      "attributes": {
        "anchor": "caster_focus",
        "radius_m": 6,
        "pull_newtons": 40,
        "duration_seconds": 4,
        "exclude_tags": [
          "ally"
        ]
      },
      "requirements": [
        "ogmr.capability.gravity_field",
        "ogmr.capability.target_filter"
      ],
      "bounds": {
        "max_radius_m": 6,
        "max_pull_newtons": 40,
        "max_targets": 10,
        "max_duration_seconds": 4
      },
      "provenance": {
        "generator": "example.validation",
        "seed": 7171
      }
    }
  ],
  "entry_point": "root",
  "root": {
    "spell": 0,
    "children": []
  }
}
```

#### Performer pseudocode

```text
function performGravitySnare(magic, world, caster):
    interpreted = interpreter.validateAndResolve(magic)
    spell = interpreted.spells[0]  # structural position, not a generated id
    requireCapability("ogmr.capability.gravity_field")
    requireCapability("ogmr.capability.target_filter")
    anchor = world.resolveAnchor(spell.attributes.anchor)
    assert spell.attributes.radius_m <= spell.bounds.max_radius_m
    targets = world.findBodiesNear(anchor, spell.attributes.radius_m, exclude=spell.attributes.exclude_tags, limit=spell.bounds.max_targets)
    pull = min(spell.attributes.pull_newtons, spell.bounds.max_pull_newtons)
    for body in targets: world.applyForceToward(body, anchor, pull, spell.attributes.duration_seconds)
    world.scheduleCleanup(targets, spell.attributes.duration_seconds)
```

#### What the magic does

The spell creates a local gravity field around an anchor, filters excluded targets, clamps pull strength and target count, and applies temporary inward force. In this candidate, the magic composes the spell through a root tree node, so validation can prove the performer has the required capabilities, bounded parameters, and resolvable local structure before runtime effects are applied.

### Example 5: Soft Repair

#### JSON spell representation

```json
{
  "spell_type": "ogmr.soft_repair",
  "attributes": {
    "target": "damaged_soft_material",
    "repair_points": 30,
    "energy_cost": 12,
    "duration_seconds": 5,
    "stop_at_integrity": 0.9
  },
  "requirements": [
    "ogmr.capability.repair",
    "ogmr.capability.soft_material"
  ],
  "bounds": {
    "max_repair_points": 30,
    "max_energy_cost": 12,
    "max_duration_seconds": 5,
    "max_integrity": 0.9
  },
  "provenance": {
    "generator": "example.validation",
    "seed": 8181
  }
}
```

#### JSON magic representation

```json
{
  "version": 1,
  "format": "tree",
  "spells": [
    {
      "spell_type": "ogmr.soft_repair",
      "attributes": {
        "target": "damaged_soft_material",
        "repair_points": 30,
        "energy_cost": 12,
        "duration_seconds": 5,
        "stop_at_integrity": 0.9
      },
      "requirements": [
        "ogmr.capability.repair",
        "ogmr.capability.soft_material"
      ],
      "bounds": {
        "max_repair_points": 30,
        "max_energy_cost": 12,
        "max_duration_seconds": 5,
        "max_integrity": 0.9
      },
      "provenance": {
        "generator": "example.validation",
        "seed": 8181
      }
    }
  ],
  "entry_point": "root",
  "root": {
    "spell": 0,
    "children": []
  }
}
```

#### Performer pseudocode

```text
function performSoftRepair(magic, world, caster):
    interpreted = interpreter.validateAndResolve(magic)
    spell = interpreted.spells[0]  # structural position, not a generated id
    requireCapability("ogmr.capability.repair")
    requireCapability("ogmr.capability.soft_material")
    target = world.resolveSoftMaterial(spell.attributes.target)
    assert caster.energy >= spell.attributes.energy_cost
    repair = min(spell.attributes.repair_points, spell.bounds.max_repair_points)
    limit = min(spell.attributes.stop_at_integrity, spell.bounds.max_integrity)
    world.spendEnergy(caster, spell.attributes.energy_cost)
    world.repairGradually(target, repair, limit, spell.attributes.duration_seconds)
```

#### What the magic does

The spell repairs damaged soft or living material gradually, spends bounded energy, and stops before exceeding the declared integrity threshold. In this candidate, the magic composes the spell through a root tree node, so validation can prove the performer has the required capabilities, bounded parameters, and resolvable local structure before runtime effects are applied.

## Implementability and extensibility check

This candidate is implementable when the interpreter can validate its topology, resolve structural spell references, check capability requirements, and produce a bounded performer input structure. It is extensible because spell types, attributes, requirements, generation metadata, and extension payloads are namespaced and can be preserved even when a performer only partially supports them.
