# Grammar Template Format Candidate

## Summary

This candidate represents magic through procedural grammars and templates that expand into concrete spell structures. It is intended to be serializable, procedurally generatable, interpretable into performer-facing data, and usable by games ranging from simple rule worlds to detailed physical simulations.

## Core concepts

- **Spell**: one declarative unit in the procedural grammars and templates that expand into concrete spell structures model, with typed attributes, requirements, validation bounds, metadata, and extension data.
- **Magic**: the complete procedural grammars and templates that expand into concrete spell structures composition containing many spells, entry points, policies, spell provenance summaries, and diagnostics.
- **Performer**: a game-provided runtime that consumes the interpreted magic structure and maps abstract effects to the concrete world.

## Structural model

Each spell structure contains `spell_type`, typed attributes, requirements, metadata, bounds, generator provenance, and extension payloads, but not a required id. Each magic structure contains `version`, the procedural grammars and templates that expand into concrete spell structures topology, entry points, policies, spell provenance summaries, global requirements, and validation diagnostics.

## How it describes physical worlds

The format describes targets, quantities, materials, fields, constraints, state, events, and operations abstractly. A performer can map the same magic to gameplay tags, entity components, rigid bodies, soft bodies, fluids, heat diffusion, gravity fields, falling-sand cells, chemistry, or other game-specific systems.

## Extensibility

- Namespaced spell and magic fields allow standard and game-specific vocabularies.
- Capability requirements allow rejection, approximation, fallback, or partial support.
- Extension payloads survive interpretation for tools and game plugins.
- Procedural spell-generation bounds keep generated spells safe for real-time worlds.

## Representation and interpreter

- **JSON representation**: a readable object containing the procedural grammars and templates that expand into concrete spell structures topology, spell structures, magic structure, requirements, policies, spell provenance summaries, and metadata.
- **Dedicated tightened format**: a compact canonical encoding with interned strings, typed attribute blocks, integer references, prevalidated topology tables, and bounded resource headers.
- **Interpreter output**: a `MagicGrammarTemplate` structure containing resolved spells, candidate topology, lookup tables, typed attributes, requirements, spell provenance summaries, extension payloads, and diagnostics.

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
