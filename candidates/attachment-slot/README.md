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

- `magic_id` and `version`.
- `anchors`: abstract places where spells can attach.
- `attachments`: spell entries bound to anchors or other attachments.
- `compatibility`: optional slot constraints, tags, and conflict rules.
- `resolution`: priority, stacking, override, and fallback policies.
- `metadata`: authoring and compatibility data.

Each attachment should contain:

- `id`: stable local identifier.
- `type`: namespaced spell kind.
- `anchor`: target anchor or slot id.
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
  `compatibility`, and `resolution` sections. Human-readable anchor and slot ids
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
