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

## Quick validation examples

| Magic | Behavior | Example implementable performer logic | Implementability / extensibility check |
| --- | --- | --- | --- |
| Thermal Lift | Select a target volume, add bounded heat, convert part of that energy into upward pressure, and expire before overheating. | A performer validates thermal and force capabilities, resolves the target, applies capped heat/impulse, or approximates with a lift status. | Implementable because bounds and capabilities are explicit; extensible with new heat models and pressure solvers. |
| Water Wall | Gather nearby water or water-tagged entities, shape them into a barrier, increase cohesion, and release them after duration. | A performer maps selection to fluid cells, particles, or gameplay entities, then applies cohesion and collision rules. | Implementable at many simulation fidelities; extensible with material filters and wall-shape plugins. |
| Stone Brace | Target a body or surface, increase stiffness and fracture threshold, add mass penalty, and cleanly restore original values. | A performer stores original material data, applies bounded modifiers, and restores or blends them on expiry. | Implementable for stats, rigid bodies, or soft bodies; extensible with more material attributes. |
| Gravity Snare | Create a local gravity field that pulls eligible targets toward an anchor while excluding allies and clamping acceleration. | A performer samples affected bodies each tick and applies clamped impulses according to performer physics rules. | Implementable because target masks and clamps are declared; extensible with custom field equations. |
| Soft Repair | Find damaged soft or living material, spend available energy, restore integrity gradually, and stop at a safety threshold. | A performer maps repair to health, tissue, mesh constraints, or soft-body coefficients depending on game systems. | Implementable without mandating one health model; extensible with domain-specific repair vocabularies. |

## Implementability and extensibility check

This candidate is implementable when the interpreter can validate its topology, resolve structural spell references, check capability requirements, and produce a bounded performer input structure. It is extensible because spell types, attributes, requirements, generation metadata, and extension payloads are namespaced and can be preserved even when a performer only partially supports them.
