
---

# 🎯 Item System & Attribute-Driven Rarity (Design)

## Overview

Modern roguelike variety arises primarily from item diversity and emergent behaviour combinations. This section defines a **data-driven item system** that supports:

* **Modular item behaviours** (“attributes”) that mutate combat effects
* Stat upgrades distinct from behaviour modifiers
* **Rarity derived from attribute count** (not fixed tiers)
* Integration with existing combat and equipment architecture

> Items are first-class game objects with *both numeric stats and behaviour modules*. Attribute mechanics are triggered via combat and projectile events rather than hardcoded loops.

---

## 1) Item Definition vs Item Instance

### ItemDef (Resource)

A reusable template defining:

* Item Category (MELEE, RANGED, SPELL, SHIELD)
* Base stats (damage, cooldown, range, etc.)
* Allowed attribute types
* Visual metadata (icons, names, descriptions)

ItemDef resources are lightweight definitions that drive generation and UI.

### ItemInstance (Ref/Resource)

A concrete runtime item that contains:

* Reference to an ItemDef
* Rolled numeric stat upgrades
* A list of AttributeResources
* Unique seed for deterministic properties
* Computed rarity based on attribute list

*ItemInstance* encapsulates both stats and behaviour modules without being tied to scene nodes.

---

## 2) Attribute Model

Attributes are modular Resources representing discrete mechanics:

* **Stat modifiers** (e.g., +10% damage)
* **Behaviour triggers** (e.g., on_hit, on_attack, on_projectile_spawn)
* **Conditional effects** (e.g., effects only vs bosses, vs stunned)

Each attribute must:

1. Define a description for UI
2. Declare applicable hooks (on_attack, on_hit, on_block, etc.)
3. Provide serialization for save/loot generation

Attributes do *not* contain execution loops — they respond to signals passed from item execution logic.

---

## 3) Rarity Calculation

Rarity is *derived* from attribute composition:

```
rarity_tier = attribute_list.size()
```

Optionally:

* Attribute cost is weighted for powerful mechanics
* Higher total cost → higher rarity label (Common → Rare → Epic → Legendary)

Rarity is recomputed when:

* Attributes are added or removed
* Item is first rolled

Rarity affects:

* UI display borders
* Naming weaks vs strong affixes
* Drop rate from enemies

---

## 4) Upgrade Model

### Stat Upgrades

ItemInstances contain upgrade levels for numeric stats:

* `damage_level`
* `cooldown_level`
* `speed_bonus`

Upgrades use arithmetic or percentage formulas derived from ItemDef defaults.

### Adding Attributes

Attributes are added via upgrade rolls or player actions:

* New attributes increase rarity
* Attribute list is constrained by allowed types from ItemDef
* Cannot add duplicate attributes (unless specified)

Upgrading and attribute addition persist through save/respawn.

---

## 5) Execution Hooks

Items trigger behaviour via signals:

| Hook                  | Trigger                    |
| --------------------- | -------------------------- |
| `on_attack_start`     | Before attack executes     |
| `on_projectile_spawn` | When projectile is emitted |
| `on_hit`              | When attack hits a target  |
| `on_block`            | When shield blocks         |
| `on_parry`            | On successful parry        |

Each attribute defines handlers for the hooks it supports. The weapon/spell logic dispatches events to attributes, which then modify gameplay.

---

## 6) Integration With Current Architecture

Items are still scene node based in the world:

```
Pickup (scene)
  └─ ItemNode (refers to ItemInstance)
```

On pickup:

1. ItemInstance is added to Inventory component
2. Equipment component swaps the instance into slot
3. Combat invokes item’s execute method
4. ItemInstance dispatches hooks to attributes

This keeps **existing node logic + component structure intact** — attributes augment, not replace, current attack behaviour. ([GitHub][1])

---

## 7) UI Implications

Inventory and equipped item UI must display:

* Base stats
* Attribute list with descriptions
* Current rarity label

Attributes should be clickable for detail popups.

---

## 8) Generation and Determinism

Item generation is seed-driven:

1. Choose ItemDef from loot pool
2. Roll base stat upgrades
3. Determine number of attributes based on difficulty/seed
4. Randomly pick allowed attributes from ItemDef list
5. Bake ItemInstance with seed for reproducibility

This ensures deterministic runs for debugging and replay.

---

## 9) Balance Rules

* Stats scale slowly (linear or low exponent)
* Behaviour attributes scale effects drastically → more variety
* Avoid power stacking by limiting attribute slots

Rarity scales with behaviour, not just numbers.

---

## 10) Future Extensions

* Attribute affinities (helps synergy)
* Affix prefixes/suffixes for naming
* Conditional attributes (only trigger vs bosses, low health, etc.)

---

## Alignment with Core Principles

This design adheres to the project’s architectural goals:

* **Items are nodes/resources** — not transient data blobs ([GitHub][1])
* **Combat executes via equipment** (item behaviour is the executor) ([GitHub][1])
* **State machines don’t control logic** — attributes hook into events, not states ([GitHub][1])
* **Determinism preserved** through seeded item generation

---

## Summary

The item system extends modular architecture with:

* Data-driven item definitions
* Modular behaviours (attributes)
* Rarity tied to attribute count
* Stat upgrades separate from behaviours
* Deterministic procedural variety

This structure prepares the ground for scalable content without compromising architecture stability. ([GitHub][1])

---
