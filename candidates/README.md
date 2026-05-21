# OGMR Format Candidates

This folder collects candidate plans for Open Game Magic Representation (OGMR)
formats. Each candidate keeps the same three core concepts:

- **Spell**: the smallest declarative unit of magic.
- **Magic**: a variable-size composition of spells.
- **Performer**: a game-provided runtime that accepts magic and applies effects
  to the game world. OGMR only describes inputs and semantics; it does not
  implement concrete performers.

The candidates are intentionally game-world-neutral. A performer may target a
simple RPG stat system, a deterministic simulation, or a fully physical world
with rigid bodies, soft bodies, fluids, heat, gravity, falling sand, chemistry,
or any other domain model.

## Candidates

- [Tree Magic Format](tree/README.md): hierarchical spell composition.
- [DAG Magic Format](dag/README.md): dependency graph composition.
- [Inline Sequence Format](inline-sequence/README.md): ordered stream/pipeline composition.
- [Attribute Component Format](attribute-component/README.md): data-oriented spell bundles.
- [Attachment Slot Format](attachment-slot/README.md): socketed spells attached to anchors.
- [Rule Reaction Format](rule-reaction/README.md): event/condition/action magic rules.
- [Layer Stack Format](layer-stack/README.md): ordered layers, masks, blends, overrides, and accumulators.
- [State Machine Format](state-machine/README.md): states, transitions, guards, actions, and lifecycle phases.
- [Grammar Template Format](grammar-template/README.md): procedural grammars and templates that expand into concrete spell structures.
- [Constraint Solver Format](constraint-solver/README.md): variables, constraints, objectives, bounds, and solver policies.
- [Field Network Format](field-network/README.md): sources, sinks, transforms, samplers, and spatial or abstract fields.
- [Timeline Track Format](timeline-track/README.md): timed tracks, clips, curves, markers, and synchronization rules.
- [Blackboard Format](blackboard/README.md): shared facts read and written by scheduled spell rules.
- [Entity Recipe Format](entity-recipe/README.md): entity creation, component mutation, links, and cleanup recipes.
- [Message Passing Format](message-passing/README.md): spell actors, channels, handlers, and typed messages.
- [Algebraic Expression Format](algebraic-expression/README.md): typed expressions, functions, formulas, and effect constructors.
- [Behavior Tree Format](behavior-tree/README.md): selectors, sequences, decorators, conditions, and action leaves.
- [Patch Delta Format](patch-delta/README.md): declarative patches that add, replace, scale, or remove world properties.

## Projectile explosion stress test

Candidates must be tested against a multi-component projectile magic before they
are considered viable. The stress-test magic emits a hot ball with initial
velocity, either as a particle group or solid body. While moving, it emits hot
fire particles that contribute heat to the heat map, and the ball itself also
contributes heat. On collision, it explodes: the explosion can remove or damage
falling-sand elements, damage solid-body and softbody structures, emit heat,
spawn high-speed fragments and fire particles, and apply collision/pressure
force either through particle collision when available or through a direct
impulse fallback.

The reference decomposition for this test is:

```json
{
  "stress_test": "ogmr.projectile_explosion",
  "required_spell_roles": [
    "spawn_projectile_body_or_particle_group",
    "set_initial_velocity",
    "attach_heat_source_to_projectile",
    "emit_hot_fire_particles_while_moving",
    "sample_collision_event",
    "damage_falling_sand_cells",
    "damage_solid_body_and_softbody_regions",
    "emit_explosion_heat",
    "spawn_high_speed_fragments",
    "spawn_explosion_fire_particles",
    "apply_particle_collision_force_when_supported",
    "apply_direct_impulse_pressure_fallback"
  ],
  "required_world_components": [
    "gravity_map",
    "grid_particle_fluid",
    "falling_sand",
    "solid_soft_body",
    "heat_map_air_interaction"
  ]
}
```

Candidate status for this stress test:

| Candidate | Status | Reason |
| --- | --- | --- |
| Tree | Pass | Can nest spawn, motion, trail, collision, explosion, fragment, heat, and fallback impulse spells under one projectile magic tree. |
| DAG | Pass | Can model projectile spawn, trail emission, collision gate, explosion effects, and fallback impulse as dependent graph nodes. |
| Inline Sequence | Pass | Can encode setup, update subscription, collision wait, explosion, and cleanup as ordered steps over shared context. |
| Attribute Component | Pass | Can describe the projectile as components for body/particles, velocity, heat emission, collision trigger, explosion payload, damage, fragments, and impulse fallback. |
| Attachment Slot | Pass | Can attach trail emitters, heat payloads, collision triggers, fragment emitters, and impulse fallback slots to a projectile anchor. |
| Rule Reaction | Pass | Naturally represents moving heat emission and on-collision explosion as event-condition-action rules. |
| Layer Stack | Pass | Can layer projectile body emission, heat contribution, trail particles, collision mask, explosion deltas, damage masks, fragment layer, and impulse fallback accumulators. |
| State Machine | Pass | Can represent projectile lifecycle states such as spawned, moving, collided, exploding, fragmenting, and expired. |
| Grammar Template | Pass | Can expand a projectile-explosion template into the required spell roles with bounded generated fragments and particles. |
| Constraint Solver | Pass | Can represent bounded projectile/explosion variables, collision constraints, damage/heat objectives, fragment bounds, and impulse fallback policy. |
| Field Network | Pass | Can connect moving sources, heat fields, particle emitters, collision samplers, explosion fields, damage samplers, fragment sources, and impulse fields. |
| Timeline Track | Pass | Can place projectile setup, moving emission clips, collision-synchronized explosion clips, fragment clips, heat clips, and cleanup markers on tracks. |
| Blackboard | Pass | Can use facts for projectile state, movement, collision, particle-collision support, explosion payload, damage targets, fragments, and fallback impulse decisions. |
| Entity Recipe | Pass | Can create a projectile entity or particle group with velocity, heat/trail components, collision-trigger components, explosion payloads, damage mutations, fragments, and cleanup. |
| Message Passing | Pass | Can model projectile, trail emitter, collision detector, explosion, damage, heat, fragment, and impulse actors exchanging typed messages. |
| Algebraic Expression | Pass | Can compose expressions for projectile creation, movement, heat emission, collision predicate, explosion effects, fragment generation, and impulse fallback. |
| Behavior Tree | Pass | Can sequence spawn/move behavior, run trail emission while moving, wait for collision, select particle-collision or direct-impulse force, then execute explosion effects. |
| Patch Delta | Pass | Can patch in projectile state, velocity, heat/trail emission records, collision-trigger records, explosion damage/heat deltas, fragments, fire particles, and impulse fallback deltas. |

No candidate currently fails this stress test. If a future candidate cannot
represent all required spell roles and component interactions as declarative
spell data that a performer can lower to component reads, writes, and events,
keep it in this list and mark its status as `Fail`. A candidate would fail if it
needed one opaque helper such as `world.castFireball()` or `world.explodeMagic()`
instead of representing projectile spawn, motion, heat emission, collision,
damage, fragments, and impulse fallback as separate inspectable spell roles.

## Shared design goals

- Keep OGMR declarative and serializable.
- Spell procedural generation should be a first-class use case: generators create spell structures, while magic structures compose generated or authored spells. Generated spells must use structural position, local handles, or content-derived references instead of ids, and must include provenance, bounds, deterministic seeds where useful, and validation before performer use.
- Keep spell structures and magic structures separate: spells are reusable units of intent, while magic is the composition/container that arranges many spells.
- Make every candidate representable as JSON for tooling, interchange,
  debugging, and schema validation.
- Treat JSON as a portable representation, not necessarily the final runtime
  format. A production OGMR format should also define a dedicated tightened
  representation that is compact, canonical, fast to parse, and suitable for
  real game loading.
- Define an interpreter path for every candidate. The interpreter reads either
  JSON or the tightened representation, validates it, resolves references and
  extensions, and emits a performer-facing data structure.
- Avoid game-specific effect implementations in the format.
- Let performers advertise supported capabilities and reject, approximate, or
  partially apply unsupported spells.
- Use structural references, local handles, extensible metadata, namespaced attributes, and version
  fields so procedurally generated spells do not depend on ids and each candidate can evolve without breaking existing content.
- Represent physical-world concepts through abstract targets, quantities,
  constraints, fields, events, and capability requirements rather than through
  one fixed engine model.
- Make validation examples concrete: include at least five examples per candidate, and for each example show a JSON spell representation, a JSON magic representation, performer pseudocode, and a description of what the magic does. Example magic must compose multiple spells and use the candidate-specific composition structure rather than one single-spell container. Example spell data should use varied operation records, not one mandatory attribute-only spell shape. Example pseudocode must assume `world` provides only low-level accessors and mutators over a practical component world: a Poisson-style gravity map with base potential, grid/particle fluid simulation, falling-sand grid, solid/soft body objects, and a heat map with simplified air flow. It must not call direct magic-like operations such as gradual repair or barrier shaping.

## Common representation pipeline

Each candidate should support three layers:

1. **Authoring/interchange JSON**: readable and schema-validatable form used by
   tools, editors, tests, examples, and source control.
2. **Dedicated tightened format**: final compact OGMR representation. It may be
   binary, tokenized text, or another canonical encoding, but it should preserve
   the same semantics as JSON while reducing ambiguity, parsing cost, and file
   size.
3. **Performer input structure**: an in-memory structure produced by an
   interpreter. The structure should hold resolved spells, composition shape,
   attributes, requirements, extension data, diagnostics, and any precomputed
   lookup tables the performer needs.

The interpreter is part of OGMR infrastructure, not a concrete game performer.
It should not apply effects to the world. Its job is to make magic data safe,
resolved, and convenient for game-specific performers.
