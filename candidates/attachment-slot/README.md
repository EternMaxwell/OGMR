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

- **Spell structure**: one reusable or procedurally generated unit of intent. It may be an operation record, query, formula, component patch, schedule, resource debit, or other candidate-specific shape; it declares inputs/outputs when relevant, capability requirements, validation bounds, provenance, and extension payloads without requiring a usable id.
- **Magic structure**: a composition/container with format version, the candidate-specific topology, entry points, performer policies, global requirements, and interpreter diagnostics. It may name the composition, but spell addressing inside it must use topology/local handles rather than generated spell ids.

The interpreter keeps this separation while resolving many spells into one performer-facing magic object. It validates and lowers data, but it does not apply effects to a game world.

## Validation examples

Each candidate includes at least five concrete validation examples. Every example separates generated spell data from the candidate-specific magic container, shows performer-facing pseudocode, and describes the intended runtime behavior. The examples intentionally compose each magic from multiple spells and use varied spell records such as scopes, component reads, field patches, schedules, formulas, and resource debits rather than a single mandatory attribute-only spell shape.

The validation examples use a concrete but still low-level example world. `world.gravity` is a gravity map with mass-derived potential cells plus a background base potential map; the host simulation may solve the Poisson equation after performers write mass or potential fields. `world.fluid` combines grid cells and particles for liquid quantities. `world.sand` is a falling-sand grid for fast powder and simple dynamic solid elements. `world.body` stores rigid-body, solid-body, and softbody objects and node fields. `world.heat` stores temperatures and a simplified air-fluid layer that can exchange heat with the other components. These components expose accessors and mutators for cells, particles, objects, fields, and scheduled events only; examples must not call direct magic-like helpers such as `repairGradually`, `shapeBarrier`, or `applyForce`.

### Example 1: Gravity-Heated Updraft

#### JSON spell representation

```json
{
  "spell_encoding": "slot_payload_operation_records",
  "slot_payloads": [
    {
      "slot": "slot_0",
      "handle": "air_column",
      "record": {
        "op": "scope.volume",
        "handle": "air_column",
        "component_reads": [
          "heat_map.air",
          "gravity_map.potential"
        ],
        "selector": {
          "kind": "cylinder",
          "center": "caster.forward_2m",
          "radius_m": 1.2,
          "height_m": 4.0
        },
        "limits": {
          "max_cells": 96
        }
      }
    },
    {
      "slot": "slot_1",
      "handle": "warm_air",
      "record": {
        "op": "thermal.cell_delta",
        "handle": "warm_air",
        "input": "air_column",
        "write": {
          "component": "heat_map",
          "field": "temperature_c"
        },
        "delta_c": 12,
        "limits": {
          "max_total_joules": 1800
        }
      }
    },
    {
      "slot": "slot_2",
      "handle": "lift_targets",
      "record": {
        "op": "body.force_patch",
        "handle": "lift_targets",
        "input": "air_column",
        "reads": [
          "gravity_map.potential_gradient",
          "solid_soft_body.mass"
        ],
        "force": {
          "direction": "against_local_gravity",
          "newtons": 45
        },
        "duration_s": 3
      }
    }
  ]
}
```

#### JSON magic representation

```json
{
  "version": 1,
  "format": "attachment_slot",
  "spell_pool": [
    {
      "op": "scope.volume",
      "handle": "air_column",
      "component_reads": [
        "heat_map.air",
        "gravity_map.potential"
      ],
      "selector": {
        "kind": "cylinder",
        "center": "caster.forward_2m",
        "radius_m": 1.2,
        "height_m": 4.0
      },
      "limits": {
        "max_cells": 96
      }
    },
    {
      "op": "thermal.cell_delta",
      "handle": "warm_air",
      "input": "air_column",
      "write": {
        "component": "heat_map",
        "field": "temperature_c"
      },
      "delta_c": 12,
      "limits": {
        "max_total_joules": 1800
      }
    },
    {
      "op": "body.force_patch",
      "handle": "lift_targets",
      "input": "air_column",
      "reads": [
        "gravity_map.potential_gradient",
        "solid_soft_body.mass"
      ],
      "force": {
        "direction": "against_local_gravity",
        "newtons": 45
      },
      "duration_s": 3
    }
  ],
  "anchor": "caster_cast_frame",
  "slots": [
    {
      "name": "slot_0",
      "spell": 0,
      "order": 0
    },
    {
      "name": "slot_1",
      "spell": 1,
      "order": 1
    },
    {
      "name": "slot_2",
      "spell": 2,
      "order": 2
    }
  ]
}
```

#### Performer pseudocode

```text
function performGravityHeatedUpdraft(magic, world, caster):
    plan = interpreter.validateAndResolve(magic)
    for cell_ref in plan.local("air_column").cells:
        heat_cell = world.heat.getCell(cell_ref)
        gravity_cell = world.gravity.getCell(cell_ref)
        capped_delta = min(plan.local("warm_air").delta_c, heat_cell.fields.max_safe_delta_c)
        world.heat.setCell(cell_ref, heat_cell.withField("temperature_c", heat_cell.fields.temperature_c + capped_delta))
        world.gravity.setCell(cell_ref, gravity_cell.withField("local_mass_kg", gravity_cell.fields.local_mass_kg + heat_cell.fields.air_mass_kg))
    for body_ref in plan.local("lift_targets").body_refs:
        body = world.body.getObject(body_ref)
        gradient = world.gravity.getCell(body.fields.center_cell).fields.potential_gradient
        force_entry = {"vector": normalize(-gradient) * plan.local("lift_targets").force.newtons, "expires_at_tick": world.tick + 180}
        forces = list(body.fields.get("force_entries", []))
        forces.append(force_entry)
        world.body.setField(body_ref, "force_entries", forces)
        world.appendEvent("body.remove_force_entry", {"body": body_ref, "entry": force_entry, "at_tick": world.tick + 180})
```

#### What the magic does

The magic combines a scope spell, heat-map cell edits, gravity-map sampling, and body force fields. The performer only reads and writes cells, object fields, and events; the gravity Poisson solve and heat/air update run later in the host simulation.

### Example 2: Fluid-and-Sand Water Wall

#### JSON spell representation

```json
{
  "spell_encoding": "slot_payload_operation_records",
  "slot_payloads": [
    {
      "slot": "slot_0",
      "handle": "water_source",
      "record": {
        "op": "fluid.source_scan",
        "handle": "water_source",
        "component_reads": [
          "fluid.grid",
          "fluid.particles"
        ],
        "material": "water",
        "bounds": {
          "max_liters": 700,
          "radius_m": 6
        }
      }
    },
    {
      "slot": "slot_1",
      "handle": "packed_footing",
      "record": {
        "op": "falling_sand.foundation",
        "handle": "packed_footing",
        "component": "falling_sand",
        "material_filter": [
          "wet_sand",
          "clay"
        ],
        "shape": {
          "kind": "line",
          "length_m": 5
        }
      }
    },
    {
      "slot": "slot_2",
      "handle": "wall_volume",
      "record": {
        "op": "fluid.grid_particle_transfer",
        "handle": "wall_volume",
        "from": "water_source",
        "support": "packed_footing",
        "target_shape": {
          "kind": "wall",
          "height_m": 2,
          "thickness_m": 0.35
        },
        "duration_s": 6
      }
    }
  ]
}
```

#### JSON magic representation

```json
{
  "version": 1,
  "format": "attachment_slot",
  "spell_pool": [
    {
      "op": "fluid.source_scan",
      "handle": "water_source",
      "component_reads": [
        "fluid.grid",
        "fluid.particles"
      ],
      "material": "water",
      "bounds": {
        "max_liters": 700,
        "radius_m": 6
      }
    },
    {
      "op": "falling_sand.foundation",
      "handle": "packed_footing",
      "component": "falling_sand",
      "material_filter": [
        "wet_sand",
        "clay"
      ],
      "shape": {
        "kind": "line",
        "length_m": 5
      }
    },
    {
      "op": "fluid.grid_particle_transfer",
      "handle": "wall_volume",
      "from": "water_source",
      "support": "packed_footing",
      "target_shape": {
        "kind": "wall",
        "height_m": 2,
        "thickness_m": 0.35
      },
      "duration_s": 6
    }
  ],
  "anchor": "caster_cast_frame",
  "slots": [
    {
      "name": "slot_0",
      "spell": 0,
      "order": 0
    },
    {
      "name": "slot_1",
      "spell": 1,
      "order": 1
    },
    {
      "name": "slot_2",
      "spell": 2,
      "order": 2
    }
  ]
}
```

#### Performer pseudocode

```text
function performFluidAndSandWaterWall(magic, world, caster):
    plan = interpreter.validateAndResolve(magic)
    remaining = plan.local("water_source").bounds.max_liters
    for particle_ref in world.fluid.listParticles(plan.local("water_source").scan_bounds):
        particle = world.fluid.getParticle(particle_ref)
        if particle.fields.material == "water" and remaining > 0:
            moved = min(particle.fields.volume_liters, remaining)
            world.fluid.setParticleField(particle_ref, "volume_liters", particle.fields.volume_liters - moved)
            remaining -= moved
            target_cell = plan.local("wall_volume").next_target_cell()
            cell = world.fluid.getGridCell(target_cell)
            world.fluid.setGridCell(target_cell, cell.withField("water_liters", cell.fields.water_liters + moved))
    for sand_ref in plan.local("packed_footing").cells:
        sand_cell = world.sand.getCell(sand_ref)
        if sand_cell.fields.material in ["wet_sand", "clay"]:
            world.sand.setCell(sand_ref, sand_cell.withField("packing", min(1.0, sand_cell.fields.packing + 0.25)))
    world.appendEvent("fluid.release_cells", {"cells": plan.local("wall_volume").target_cells, "at_tick": world.tick + 360})
```

#### What the magic does

The magic is a three-spell composition over fluid particles, fluid grid cells, and falling-sand support cells. It creates no magic wall function; it moves water quantities and sand packing fields that existing solvers can process.

### Example 3: Stone Brace Under Load

#### JSON spell representation

```json
{
  "spell_encoding": "slot_payload_operation_records",
  "slot_payloads": [
    {
      "slot": "slot_0",
      "handle": "arch_segment",
      "record": {
        "op": "body.target",
        "handle": "arch_segment",
        "component_reads": [
          "solid_soft_body"
        ],
        "selector": {
          "tag": "damaged_arch"
        },
        "limits": {
          "max_mass_kg": 2500
        }
      }
    },
    {
      "slot": "slot_1",
      "handle": "load_vector",
      "record": {
        "op": "gravity.load_sample",
        "handle": "load_vector",
        "input": "arch_segment",
        "component_reads": [
          "gravity_map.base_potential",
          "gravity_map.mass_potential"
        ]
      }
    },
    {
      "slot": "slot_2",
      "handle": "brace_patch",
      "record": {
        "op": "body.material_patch",
        "handle": "brace_patch",
        "input": "arch_segment",
        "fields": {
          "stiffness_scale": 1.4,
          "fracture_threshold_add": 120
        },
        "restore_after_s": 10
      }
    }
  ]
}
```

#### JSON magic representation

```json
{
  "version": 1,
  "format": "attachment_slot",
  "spell_pool": [
    {
      "op": "body.target",
      "handle": "arch_segment",
      "component_reads": [
        "solid_soft_body"
      ],
      "selector": {
        "tag": "damaged_arch"
      },
      "limits": {
        "max_mass_kg": 2500
      }
    },
    {
      "op": "gravity.load_sample",
      "handle": "load_vector",
      "input": "arch_segment",
      "component_reads": [
        "gravity_map.base_potential",
        "gravity_map.mass_potential"
      ]
    },
    {
      "op": "body.material_patch",
      "handle": "brace_patch",
      "input": "arch_segment",
      "fields": {
        "stiffness_scale": 1.4,
        "fracture_threshold_add": 120
      },
      "restore_after_s": 10
    }
  ],
  "anchor": "caster_cast_frame",
  "slots": [
    {
      "name": "slot_0",
      "spell": 0,
      "order": 0
    },
    {
      "name": "slot_1",
      "spell": 1,
      "order": 1
    },
    {
      "name": "slot_2",
      "spell": 2,
      "order": 2
    }
  ]
}
```

#### Performer pseudocode

```text
function performStoneBraceUnderLoad(magic, world, caster):
    plan = interpreter.validateAndResolve(magic)
    body_ref = plan.local("arch_segment").body_ref
    body = world.body.getObject(body_ref)
    assert body.fields.mass_kg <= plan.local("arch_segment").limits.max_mass_kg
    gravity_cell = world.gravity.getCell(body.fields.center_cell)
    original = {"stiffness": body.fields.stiffness, "fracture_threshold": body.fields.fracture_threshold}
    load_scale = 1 + length(gravity_cell.fields.potential_gradient) / max(body.fields.rest_gravity, 1)
    world.body.setField(body_ref, "stiffness", original["stiffness"] * min(plan.local("brace_patch").fields.stiffness_scale, load_scale + 0.5))
    world.body.setField(body_ref, "fracture_threshold", original["fracture_threshold"] + plan.local("brace_patch").fields.fracture_threshold_add)
    world.appendEvent("body.restore_fields", {"body": body_ref, "fields": original, "at_tick": world.tick + 600})
```

#### What the magic does

The magic composes target selection, gravity-map load sampling, and solid-body material field edits. The performer writes only body fields and an event for restoration; solid-body integration remains a host-world concern.

### Example 4: Dust Vortex Cooling Ring

#### JSON spell representation

```json
{
  "spell_encoding": "slot_payload_operation_records",
  "slot_payloads": [
    {
      "slot": "slot_0",
      "handle": "loose_dust",
      "record": {
        "op": "sand.region_query",
        "handle": "loose_dust",
        "component": "falling_sand",
        "materials": [
          "dust",
          "ash"
        ],
        "limits": {
          "max_cells": 128
        }
      }
    },
    {
      "slot": "slot_1",
      "handle": "cooling_ring",
      "record": {
        "op": "heat.air_velocity_bias",
        "handle": "cooling_ring",
        "component": "heat_map.air",
        "velocity": {
          "tangent_mps": 3.5,
          "updraft_mps": 0.8
        },
        "temperature_delta_c": -4
      }
    },
    {
      "slot": "slot_2",
      "handle": "vortex_particles",
      "record": {
        "op": "sand.velocity_patch",
        "handle": "vortex_particles",
        "input": "loose_dust",
        "field_from": "cooling_ring",
        "duration_s": 4
      }
    }
  ]
}
```

#### JSON magic representation

```json
{
  "version": 1,
  "format": "attachment_slot",
  "spell_pool": [
    {
      "op": "sand.region_query",
      "handle": "loose_dust",
      "component": "falling_sand",
      "materials": [
        "dust",
        "ash"
      ],
      "limits": {
        "max_cells": 128
      }
    },
    {
      "op": "heat.air_velocity_bias",
      "handle": "cooling_ring",
      "component": "heat_map.air",
      "velocity": {
        "tangent_mps": 3.5,
        "updraft_mps": 0.8
      },
      "temperature_delta_c": -4
    },
    {
      "op": "sand.velocity_patch",
      "handle": "vortex_particles",
      "input": "loose_dust",
      "field_from": "cooling_ring",
      "duration_s": 4
    }
  ],
  "anchor": "caster_cast_frame",
  "slots": [
    {
      "name": "slot_0",
      "spell": 0,
      "order": 0
    },
    {
      "name": "slot_1",
      "spell": 1,
      "order": 1
    },
    {
      "name": "slot_2",
      "spell": 2,
      "order": 2
    }
  ]
}
```

#### Performer pseudocode

```text
function performDustVortexCoolingRing(magic, world, caster):
    plan = interpreter.validateAndResolve(magic)
    for air_ref in plan.local("cooling_ring").air_cells:
        air_cell = world.heat.getCell(air_ref)
        velocity = air_cell.fields.air_velocity + plan.local("cooling_ring").velocity.vector_at(air_ref)
        world.heat.setCell(air_ref, air_cell.withFields({"air_velocity": velocity, "temperature_c": air_cell.fields.temperature_c - 4}))
    for sand_ref in plan.local("loose_dust").cells:
        sand_cell = world.sand.getCell(sand_ref)
        if sand_cell.fields.material in plan.local("loose_dust").materials:
            air_cell = world.heat.getCell(sand_cell.fields.air_cell)
            world.sand.setCell(sand_ref, sand_cell.withField("velocity", sand_cell.fields.velocity + air_cell.fields.air_velocity * 0.5))
    world.appendEvent("sand.clear_velocity_bias", {"cells": plan.local("loose_dust").cells, "at_tick": world.tick + 240})
```

#### What the magic does

The magic is a composed interaction between falling-sand cells and the heat map air layer. It edits cell velocities and temperatures directly, leaving particle settling and air advection to the normal simulations.

### Example 5: Softbody Repair With Thermal Guard

#### JSON spell representation

```json
{
  "spell_encoding": "slot_payload_operation_records",
  "slot_payloads": [
    {
      "slot": "slot_0",
      "handle": "torn_softbody",
      "record": {
        "op": "softbody.damage_scan",
        "handle": "torn_softbody",
        "component_reads": [
          "solid_soft_body.soft"
        ],
        "selector": {
          "tag": "repair_target"
        },
        "limits": {
          "max_nodes": 64
        }
      }
    },
    {
      "slot": "slot_1",
      "handle": "repair_cost",
      "record": {
        "op": "caster.resource_debit",
        "handle": "repair_cost",
        "resource_field": "energy",
        "amount": 25
      }
    },
    {
      "slot": "slot_2",
      "handle": "integrity_steps",
      "record": {
        "op": "softbody.node_delta",
        "handle": "integrity_steps",
        "input": "torn_softbody",
        "field": "integrity",
        "total_delta": 0.35,
        "steps": 5
      }
    },
    {
      "slot": "slot_3",
      "handle": "thermal_guard",
      "record": {
        "op": "heat.clamp",
        "handle": "thermal_guard",
        "input": "torn_softbody",
        "max_temperature_c": 45
      }
    }
  ]
}
```

#### JSON magic representation

```json
{
  "version": 1,
  "format": "attachment_slot",
  "spell_pool": [
    {
      "op": "softbody.damage_scan",
      "handle": "torn_softbody",
      "component_reads": [
        "solid_soft_body.soft"
      ],
      "selector": {
        "tag": "repair_target"
      },
      "limits": {
        "max_nodes": 64
      }
    },
    {
      "op": "caster.resource_debit",
      "handle": "repair_cost",
      "resource_field": "energy",
      "amount": 25
    },
    {
      "op": "softbody.node_delta",
      "handle": "integrity_steps",
      "input": "torn_softbody",
      "field": "integrity",
      "total_delta": 0.35,
      "steps": 5
    },
    {
      "op": "heat.clamp",
      "handle": "thermal_guard",
      "input": "torn_softbody",
      "max_temperature_c": 45
    }
  ],
  "anchor": "caster_cast_frame",
  "slots": [
    {
      "name": "slot_0",
      "spell": 0,
      "order": 0
    },
    {
      "name": "slot_1",
      "spell": 1,
      "order": 1
    },
    {
      "name": "slot_2",
      "spell": 2,
      "order": 2
    },
    {
      "name": "slot_3",
      "spell": 3,
      "order": 3
    }
  ]
}
```

#### Performer pseudocode

```text
function performSoftbodyRepairWithThermalGuard(magic, world, caster):
    plan = interpreter.validateAndResolve(magic)
    caster_energy = caster.fields.get("energy", 0)
    assert caster_energy >= plan.local("repair_cost").amount
    world.body.setField(caster.handle, "energy", caster_energy - plan.local("repair_cost").amount)
    for node_ref in plan.local("torn_softbody").node_refs:
        node = world.body.getSoftNode(node_ref)
        heat_cell = world.heat.getCell(node.fields.heat_cell)
        if heat_cell.fields.temperature_c <= plan.local("thermal_guard").max_temperature_c:
            per_step = plan.local("integrity_steps").total_delta / plan.local("integrity_steps").steps
            for step in range(1, plan.local("integrity_steps").steps + 1):
                world.appendEvent("softbody.node_field_delta", {"node": node_ref, "field": "integrity", "delta": per_step, "at_tick": world.tick + step * 30})
```

#### What the magic does

The magic uses four spells: scan damaged softbody nodes, debit caster energy, schedule node integrity deltas, and guard against overheating through heat-map cells. There is no world repair function; repair is expressed as low-level node-field events.
