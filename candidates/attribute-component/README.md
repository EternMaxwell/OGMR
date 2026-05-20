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

- `magic_id` and `version`.
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

- `component_id`: stable local identifier.
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
  typed attribute blocks. Components should be sorted by stable id or type for
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

This candidate supports procedural generation. Generators can create spell structures from templates, random seeds, player choices, world queries, or authored rule sets, then assemble them into a magic structure. Generated magic must preserve provenance, deterministic seeds when useful, stable ids, version fields, capability requirements, and hard bounds for area, duration, intensity, recursion, spawned objects, and simulation cost.

## Separate spell and magic structures

Spell and magic are different structures.

- **Spell structure**: one reusable unit of intent with `spell_id`, `spell_type`, typed attributes, declared inputs/outputs when relevant, requirements, metadata, validation bounds, and extension payloads.
- **Magic structure**: a composition/container with `magic_id`, format version, the candidate-specific topology, entry points, performer policies, generation provenance, global requirements, and interpreter diagnostics.

The interpreter keeps this separation while resolving many spells into one performer-facing magic object. It validates and lowers data, but it does not apply effects to a game world.

## Quick validation examples

| Magic | Behavior | Example implementable performer logic | Implementability / extensibility check |
| --- | --- | --- | --- |
| Thermal Lift | Select a target volume, add bounded heat, convert part of that energy into upward pressure, and expire before overheating. | A performer validates thermal and force capabilities, resolves the target, applies capped heat/impulse, or approximates with a lift status. | Implementable because bounds and capabilities are explicit; extensible with new heat models and pressure solvers. |
| Water Wall | Gather nearby water or water-tagged entities, shape them into a barrier, increase cohesion, and release them after duration. | A performer maps selection to fluid cells, particles, or gameplay entities, then applies cohesion and collision rules. | Implementable at many simulation fidelities; extensible with material filters and wall-shape plugins. |
| Stone Brace | Target a body or surface, increase stiffness and fracture threshold, add mass penalty, and cleanly restore original values. | A performer stores original material data, applies bounded modifiers, and restores or blends them on expiry. | Implementable for stats, rigid bodies, or soft bodies; extensible with more material attributes. |
| Gravity Snare | Create a local gravity field that pulls eligible targets toward an anchor while excluding allies and clamping acceleration. | A performer samples affected bodies each tick and applies clamped impulses according to performer physics rules. | Implementable because target masks and clamps are declared; extensible with custom field equations. |
| Soft Repair | Find damaged soft or living material, spend available energy, restore integrity gradually, and stop at a safety threshold. | A performer maps repair to health, tissue, mesh constraints, or soft-body coefficients depending on game systems. | Implementable without mandating one health model; extensible with domain-specific repair vocabularies. |

## Implementability and extensibility check

This candidate is implementable when the interpreter can validate its topology, resolve spell ids, check capability requirements, and produce a bounded performer input structure. It is extensible because spell types, attributes, requirements, generation metadata, and extension payloads are namespaced and can be preserved even when a performer only partially supports them.
