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
