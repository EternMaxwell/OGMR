# Rule Reaction Format Candidate

## Summary

The Rule Reaction Format represents magic as rules: when events occur and
conditions match, spell actions are emitted. It is designed for reactive magic,
environmental magic, traps, fields, enchantments, auras, and long-lived effects.

## Core concepts

- **Spell**: a rule fragment, condition, query, action, or reaction.
- **Magic**: a set of rules with shared state and lifecycle policies.
- **Performer**: subscribes to game events, evaluates conditions, and applies
  emitted actions to the host world.

## Structural model

Each magic rule set should contain:

- `version` and optional composition name.
- `state`: optional persistent values owned by the magic instance.
- `rules`: any number of event-condition-action records.
- `lifecycle`: creation, update, pause, expiry, and cleanup policies.
- `requirements`: event streams and world capabilities needed by the rules.
- `metadata`: authoring and compatibility data.

Each rule should contain:

- optional authoring-only label: if present, used only by tools and never required for generated spells.
- `on`: event pattern such as collision, temperature threshold, time tick,
  material contact, spell received, body entered volume, or custom game event.
- `where`: conditions and world queries.
- `do`: spell actions to emit.
- `limits`: cooldowns, costs, priority, determinism, and recursion guards.
- `requirements`: local performer capabilities.

## How it describes physical worlds

Events and conditions can be abstract enough to cover many simulations:

- fluid entered region;
- solid body exceeded stress threshold;
- soft body stretched beyond ratio;
- local temperature crossed phase-change point;
- sand cell settled;
- gravity vector changed;
- entity or particle crossed a field boundary.

Actions can then emit abstract operations such as apply impulse, add heat,
change material state, spawn constraint, transform velocity field, or request a
game-specific effect. Simple games can provide coarse events, while simulation
heavy games can provide fine-grained physical events.

## Extensibility

- Event names, condition operators, and action types are namespaced.
- Performers can expose event capability manifests.
- Rules can include custom state schemas.
- Recursion and scheduling policies can evolve independently from action
  vocabularies.

## Representation and interpreter

- **JSON representation**: an object with shared `state`, `rules`, lifecycle
  policies, and explicit event-condition-action records.
- **Dedicated tightened format**: a compact rule table with interned event
  patterns, condition operators, action types, state slots, lifecycle tokens,
  and limit policies. The tightened form should pre-resolve rule priorities and
  event subscription groups.
- **Interpreter output**: a `MagicRuleSet`-style structure containing state
  descriptors, rule records, event subscription indexes, condition trees or
  bytecode, action lists, lifecycle policies, requirement sets, extension
  payloads, and diagnostics. A performer can subscribe and evaluate rules
  without parsing authoring syntax.

## Strengths

- Excellent for persistent and environmental magic.
- Keeps event semantics explicit.
- Can model delayed, conditional, and triggered effects without requiring a
  general scripting language.
- Allows game performers to control scheduling and safety.

## Tradeoffs

- Requires event support from the host game.
- Determinism can be difficult if event ordering is not specified.
- Long-lived rule state needs lifecycle and cleanup discipline.

## Best use cases

- Auras, traps, fields, curses, enchantments, rituals, and reactive terrain.
- Games with rich event systems.
- Magic that should respond to detailed world simulation changes over time.

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
