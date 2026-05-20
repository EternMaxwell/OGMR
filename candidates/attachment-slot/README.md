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
