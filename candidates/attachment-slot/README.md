# Attachment Slot Format Candidate

## Summary

The Attachment Slot Format represents magic as spells attached to anchors,
sockets, layers, or channels. Attachments can modify, trigger, wrap, or constrain
other attachments. This candidate is useful when magic is assembled from runes,
items, body parts, terrain anchors, rituals, or modular gameplay equipment.

## Core concepts

- **Spell**: an attachment with a role, anchor, parameters, and compatibility
  rules.
- **Magic**: a set of attachments arranged around anchors and slots.
- **Performer**: resolves attachments for the host game object or world anchor
  and applies the resulting effects.

## Structural model

Each magic attachment set should contain:

- `version` and optional composition name.
- `anchors`: abstract places where spells can attach.
- `attachments`: spell entries bound to anchors or other attachments.
- `compatibility`: optional slot constraints, tags, and conflict rules.
- `resolution`: priority, stacking, override, and fallback policies.
- `metadata`: authoring and compatibility data.

Each attachment should contain:

- optional authoring-only label: if present, used only by tools and never required for generated spells.
- `type`: namespaced spell kind.
- `anchor`: target anchor or slot handle.
- `role`: enhancer, trigger, condition, emitter, limiter, converter, or effect.
- `attributes`: parameters.
- `requirements`: performer capabilities.

## How it describes physical worlds

Anchors can be abstract or physical:

- caster hand, staff socket, item rune slot, terrain point, fluid region,
  simulation cell group, rigid body, soft-body vertex group, heat source,
  gravity well, or moving frame of reference.

Attachments can declare how they modify physical behavior, such as adding a
force channel to an anchor, converting heat into pressure, constraining emitted
particles to a surface, or triggering when a body collides with a volume. The
performer chooses the concrete representation.

## Extensibility

- New anchor kinds and slot rules can be namespaced.
- Attachments can target other attachments to build modifier chains.
- Resolution policies can be extended for stacking and conflict handling.
- Games can expose custom anchors without changing OGMR's core concepts.

## Representation and interpreter

- **JSON representation**: an object with `anchors`, `attachments`,
  `compatibility`, and `resolution` sections. Human-readable anchor and slot handles
  should remain visible for tools.
- **Dedicated tightened format**: compact anchor and attachment tables with
  interned anchor kinds, slot names, attachment types, role names, compatibility
  tags, and resolution policy tokens. Attachment targets should be encoded as
  integer references.
- **Interpreter output**: a `MagicAttachmentSet`-style structure containing
  anchor records, attachment records, slot occupancy maps, target links,
  resolved compatibility data, stacking policies, requirement sets, extension
  payloads, and diagnostics. A performer can resolve active attachments without
  reparsing slot rules.

## Strengths

- Natural for modular magic construction.
- Good fit for equipment, rituals, enchantments, and world-bound magic.
- Supports incremental changes at runtime.
- Makes compatibility and conflicts explicit.

## Tradeoffs

- Requires clear resolution rules to avoid ambiguous stacking.
- Less suitable for pure computation-heavy magic.
- Attachment graphs can become complex if every modifier targets another
  modifier.

## Best use cases

- Rune/socket systems.
- Item enchantments and body/world anchored effects.
- Games where players assemble magic from interchangeable parts.

## Procedural generation

This candidate supports procedural spell generation. Generators create spell structures from templates, random seeds, player choices, world queries, or authored rule sets, then a separately authored or interpreted magic structure composes those generated spells. Generated spells must not rely on ids as usable fields; composition uses structural position, local handles, content-derived references, or candidate-specific links. Generated spells must preserve provenance, deterministic seeds when useful, version fields, capability requirements, and hard bounds for area, duration, intensity, recursion, spawned objects, and simulation cost.

## Separate spell and magic structures

Spell and magic are different structures.

- **Spell structure**: one reusable or procedurally generated unit of intent with `spell_type`, typed attributes, declared inputs/outputs when relevant, requirements, metadata, validation bounds, generator provenance, and extension payloads. It must not require a usable id.
- **Magic structure**: a composition/container with format version, the candidate-specific topology, entry points, performer policies, global requirements, and interpreter diagnostics. It may name the composition, but spell addressing inside it must use topology/local handles rather than generated spell ids.

The interpreter keeps this separation while resolving many spells into one performer-facing magic object. It validates and lowers data, but it does not apply effects to a game world.

## Validation examples

Each candidate includes at least five concrete validation examples. Every example separates the generated spell data from the candidate-specific magic container, shows performer-facing pseudocode, and describes the intended runtime behavior. The pseudocode assumes `world` exposes only low-level accessors and mutators such as `getObject`, `listObjects`, `createObject`, `setField`, and `appendEvent`; all magic-like behavior is computed by the performer from object fields.

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
  "format": "attachment_slot",
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
  "entry_point": "caster_focus",
  "anchors": [
    {
      "handle": "caster_focus",
      "kind": "focus"
    }
  ],
  "attachments": [
    {
      "spell": 0,
      "anchor": "caster_focus",
      "role": "thermal_lift"
    }
  ]
}
```

#### Performer pseudocode

```text
function performThermalLift(magic, world, caster):
    interpreted = interpreter.validateAndResolve(magic)
    spell = interpreted.spells[0]  # structural position, not a generated id
    requireCapability("ogmr.capability.thermal")
    requireCapability("ogmr.capability.force")
    target = world.getObject(spell.attributes.target)
    assert target.fields.area_m2 <= spell.bounds.max_area_m2
    heat_capacity = max(target.fields.get("heat_capacity_j_per_c", 1), 1)
    temperature_delta = min(spell.bounds.max_temperature_delta_c, spell.attributes.heat_joules / heat_capacity)
    heat = min(spell.attributes.heat_joules, temperature_delta * heat_capacity)
    force = min(spell.attributes.lift_newtons, spell.bounds.max_force_newtons)
    world.setField(target, "heat_joules", target.fields.get("heat_joules", 0) + heat)
    force_record = {"vector": [0, force, 0], "expires_after_seconds": spell.attributes.duration_seconds}
    forces = list(target.fields.get("forces", []))
    forces.append(force_record)
    world.setField(target, "forces", forces)
    world.appendEvent("remove_field_entry", {"object": target.handle, "field": "forces", "value": force_record, "after_seconds": spell.attributes.duration_seconds})
```

#### What the magic does

The spell selects a target volume, adds bounded heat, converts the heated air into upward force, and removes the force after the declared duration. In this candidate, the magic composes the spell through a local attachment anchor, so validation can prove the performer has the required capabilities, bounded parameters, and resolvable local structure before runtime effects are applied.

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
  "format": "attachment_slot",
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
  "entry_point": "caster_focus",
  "anchors": [
    {
      "handle": "caster_focus",
      "kind": "focus"
    }
  ],
  "attachments": [
    {
      "spell": 0,
      "anchor": "caster_focus",
      "role": "water_wall"
    }
  ]
}
```

#### Performer pseudocode

```text
function performWaterWall(magic, world, caster):
    interpreted = interpreter.validateAndResolve(magic)
    spell = interpreted.spells[0]  # structural position, not a generated id
    requireCapability("ogmr.capability.material.water")
    requireCapability("ogmr.capability.shape_barrier")
    water_sources = world.listObjects({"material": "water", "near": spell.attributes.source})
    assert spell.attributes.length_m <= spell.bounds.max_length_m
    assert spell.attributes.height_m <= spell.bounds.max_height_m
    remaining_liters = spell.bounds.max_volume_liters
    gathered_liters = 0
    for source in water_sources:
        available = source.fields.get("available_liters", 0)
        taken = min(available, remaining_liters)
        if taken > 0:
            world.setField(source, "available_liters", available - taken)
            gathered_liters += taken
            remaining_liters -= taken
    barrier = world.createObject({"kind": "temporary_barrier", "material": "water"})
    world.setField(barrier, "shape", spell.attributes.shape)
    world.setField(barrier, "length_m", spell.attributes.length_m)
    world.setField(barrier, "height_m", spell.attributes.height_m)
    world.setField(barrier, "contained_liters", gathered_liters)
    world.setField(barrier, "cohesion_until_seconds", spell.attributes.duration_seconds)
    world.appendEvent("delete_object", {"object": barrier.handle, "after_seconds": spell.attributes.duration_seconds})
```

#### What the magic does

The spell gathers available water or water-tagged entities, shapes them into a bounded barrier, increases cohesion, and releases the water when the duration expires. In this candidate, the magic composes the spell through a local attachment anchor, so validation can prove the performer has the required capabilities, bounded parameters, and resolvable local structure before runtime effects are applied.

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
  "format": "attachment_slot",
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
  "entry_point": "caster_focus",
  "anchors": [
    {
      "handle": "caster_focus",
      "kind": "focus"
    }
  ],
  "attachments": [
    {
      "spell": 0,
      "anchor": "caster_focus",
      "role": "stone_brace"
    }
  ]
}
```

#### Performer pseudocode

```text
function performStoneBrace(magic, world, caster):
    interpreted = interpreter.validateAndResolve(magic)
    spell = interpreted.spells[0]  # structural position, not a generated id
    requireCapability("ogmr.capability.material.stone")
    requireCapability("ogmr.capability.modify_structure")
    structure = world.getObject(spell.attributes.target)
    assert structure.fields.mass_kg <= spell.bounds.max_mass_kg
    original = {
        "stiffness": structure.fields.get("stiffness", 1),
        "fracture_threshold": structure.fields.get("fracture_threshold", 1)
    }
    stiffness_multiplier = min(spell.attributes.stiffness_multiplier, spell.bounds.max_stiffness_multiplier)
    fracture_bonus = min(spell.attributes.fracture_bonus, spell.bounds.max_fracture_bonus)
    world.setField(structure, "stiffness", original["stiffness"] * stiffness_multiplier)
    world.setField(structure, "fracture_threshold", original["fracture_threshold"] + fracture_bonus)
    world.appendEvent("restore_fields", {"object": structure.handle, "fields": original, "after_seconds": spell.attributes.duration_seconds})
```

#### What the magic does

The spell reinforces a touched stone or structure by bounded stiffness and fracture modifiers, records the original material state, and restores it after expiry. In this candidate, the magic composes the spell through a local attachment anchor, so validation can prove the performer has the required capabilities, bounded parameters, and resolvable local structure before runtime effects are applied.

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
  "format": "attachment_slot",
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
  "entry_point": "caster_focus",
  "anchors": [
    {
      "handle": "caster_focus",
      "kind": "focus"
    }
  ],
  "attachments": [
    {
      "spell": 0,
      "anchor": "caster_focus",
      "role": "gravity_snare"
    }
  ]
}
```

#### Performer pseudocode

```text
function performGravitySnare(magic, world, caster):
    interpreted = interpreter.validateAndResolve(magic)
    spell = interpreted.spells[0]  # structural position, not a generated id
    requireCapability("ogmr.capability.gravity_field")
    requireCapability("ogmr.capability.target_filter")
    anchor = world.getObject(spell.attributes.anchor)
    assert spell.attributes.radius_m <= spell.bounds.max_radius_m
    targets = []
    for body in world.listObjects({"has_field": "position"}):
        tags = body.fields.get("tags", [])
        if any(tag in tags for tag in spell.attributes.exclude_tags):
            continue
        if distance(body.fields.position, anchor.fields.position) <= spell.attributes.radius_m:
            targets.append(body)
        if len(targets) == spell.bounds.max_targets:
            break
    pull = min(spell.attributes.pull_newtons, spell.bounds.max_pull_newtons)
    for body in targets:
        direction = normalize(anchor.fields.position - body.fields.position)
        force_record = {"vector": direction * pull, "expires_after_seconds": spell.attributes.duration_seconds}
        forces = list(body.fields.get("forces", []))
        forces.append(force_record)
        world.setField(body, "forces", forces)
        world.appendEvent("remove_field_entry", {"object": body.handle, "field": "forces", "value": force_record, "after_seconds": spell.attributes.duration_seconds})
```

#### What the magic does

The spell creates a local gravity field around an anchor, filters excluded targets, clamps pull strength and target count, and applies temporary inward force. In this candidate, the magic composes the spell through a local attachment anchor, so validation can prove the performer has the required capabilities, bounded parameters, and resolvable local structure before runtime effects are applied.

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
  "format": "attachment_slot",
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
  "entry_point": "caster_focus",
  "anchors": [
    {
      "handle": "caster_focus",
      "kind": "focus"
    }
  ],
  "attachments": [
    {
      "spell": 0,
      "anchor": "caster_focus",
      "role": "soft_repair"
    }
  ]
}
```

#### Performer pseudocode

```text
function performSoftRepair(magic, world, caster):
    interpreted = interpreter.validateAndResolve(magic)
    spell = interpreted.spells[0]  # structural position, not a generated id
    requireCapability("ogmr.capability.repair")
    requireCapability("ogmr.capability.soft_material")
    target = world.getObject(spell.attributes.target)
    assert caster.fields.get("energy", 0) >= spell.attributes.energy_cost
    repair = min(spell.attributes.repair_points, spell.bounds.max_repair_points)
    limit = min(spell.attributes.stop_at_integrity, spell.bounds.max_integrity)
    world.setField(caster, "energy", caster.fields.get("energy", 0) - spell.attributes.energy_cost)
    current_integrity = target.fields.get("integrity", 0)
    max_integrity = max(target.fields.get("max_integrity", 1), 1)
    target_integrity = min(limit * max_integrity, current_integrity + repair)
    steps = max(1, int(spell.attributes.duration_seconds))
    delta_per_step = (target_integrity - current_integrity) / steps
    for step in range(1, steps + 1):
        world.appendEvent("field_delta_at", {"object": target.handle, "field": "integrity", "delta": delta_per_step, "at_seconds": step})
```

#### What the magic does

The spell repairs damaged soft or living material gradually, spends bounded energy, and stops before exceeding the declared integrity threshold. In this candidate, the magic composes the spell through a local attachment anchor, so validation can prove the performer has the required capabilities, bounded parameters, and resolvable local structure before runtime effects are applied.

## Implementability and extensibility check

This candidate is implementable when the interpreter can validate its topology, resolve structural spell references, check capability requirements, and produce a bounded performer input structure. It is extensible because spell types, attributes, requirements, generation metadata, and extension payloads are namespaced and can be preserved even when a performer only partially supports them.
