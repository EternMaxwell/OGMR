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

- `magic_id` and `version`.
- `state`: optional persistent values owned by the magic instance.
- `rules`: any number of event-condition-action records.
- `lifecycle`: creation, update, pause, expiry, and cleanup policies.
- `requirements`: event streams and world capabilities needed by the rules.
- `metadata`: authoring and compatibility data.

Each rule should contain:

- `id`: stable local identifier.
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
