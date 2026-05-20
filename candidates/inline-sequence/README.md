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

- `magic_id` and `version`.
- `context`: named initial values such as caster, origin, target, power, seed,
  or world query handles.
- `spells`: ordered spell entries.
- `policies`: error handling, unsupported-spell behavior, determinism, and
  rollback expectations.
- `metadata`: authoring and compatibility data.

Each spell entry should contain:

- `id`: optional stable identifier for diagnostics and tooling.
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
