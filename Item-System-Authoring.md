# Item System Authoring Guide

This document explains **how to create weapons, attributes, and item drops** in the project.
Follow these steps to keep the system **deterministic, modular, and editor-friendly**.

---

# Architecture Overview

The item system is built around four layers:

```
ItemDef        → Static item template
BaseItemType   → Procedural roll rules
ItemInstance   → Runtime rolled item
ItemAttribute  → Modular behavior units
ItemSpawner    → Deterministic world drop factory
```

**Important rule:**

* **ItemDef = static data**
* **BaseItemType = roll rules**
* **ItemAttribute = behavior**
* **ItemSpawner = drop location**

Never mix these responsibilities.

---

# Creating a New Weapon

## Step 1 — Create an ItemDef (Static Template)

Location example:

```
res://Resources/Items/Defs/
```

Create:

```
ItemDef_<Name>.tres
```

Fields to configure:

* `id`
* `display_name`
* `category` (MELEE / RANGED / SPELL / SHIELD)
* `base_stats`

Example:

```
ItemDef_Shotgun.tres
```

Example values:

```
id = "shotgun"
display_name = "Shotgun"
category = RANGED
base_stats.damage = 10
base_stats.attack_speed = 0.8
```

Rules:

* **Do not configure attributes here**
* **Do not configure roll logic here**

ItemDef should only describe **what the weapon is**.

---

## Step 2 — Create a BaseItemType (Roll Rules)

Location example:

```
res://Resources/Items/BaseTypes/
```

Create:

```
BaseType_<WeaponName>.tres
```

Example:

```
BaseType_Shotgun.tres
```

Set:

```
item_def = ItemDef_Shotgun
min_attribute_count
max_attribute_count
```

Optional:

```
start_stat_levels
```

For ranged weapons you can also configure:

```
use_ranged_mode_roll
locked_ranged_mode
weight_projectile
weight_hitscan
weight_beam
force_mode_attribute
```

Automatic attribute pool configuration:

```
use_auto_attribute_pool = true
auto_attribute_root = "res://Resources/Items/Attributes"
weapon_token = "Shotgun"
auto_include_global = true
```

Rules:

* **All attribute rolling behavior belongs here**
* **Never configure attribute pools in ItemDef**

---

# Creating a New Attribute

Attributes are **behavior modules** implemented as resources.

Location:

```
res://Resources/Items/Attributes/
```

Create:

```
Attr_<Category>_<Name>.tres
```

Example:

```
Attr_Ranged_Common_MoreProjectiles.tres
```

---

## Attribute Naming Conventions

The attribute system uses **prefix filtering**, so naming matters.

### Global

```
Attr_Global_<Name>
```

Example:

```
Attr_Global_CritChance
```

---

### Melee

```
Attr_Melee_Common_<Name>
Attr_Melee_Weapon_<Token>_<Name>
```

Example:

```
Attr_Melee_Common_Lunge
Attr_Melee_Weapon_Sword_Bleed
```

---

### Ranged

```
Attr_Ranged_Common_<Name>
Attr_Ranged_Mode_Projectile_<Name>
Attr_Ranged_Mode_Hitscan_<Name>
Attr_Ranged_Mode_Beam_<Name>
Attr_Ranged_Weapon_<Token>_<Name>
Attr_Ranged_Weapon_<Token>_Mode_<Mode>_<Name>
```

Example:

```
Attr_Ranged_Common_Penetration
Attr_Ranged_Mode_Projectile_Spread
Attr_Ranged_Mode_Hitscan_Chain
Attr_Ranged_Weapon_Shotgun_MorePellets
```

---

## Mode Enabler Attributes

These attributes are **not rolled randomly**.

They are injected automatically by the item roll logic.

```
Attr_Ranged_Mode_Hitscan
Attr_Ranged_Mode_Beam
```

Projectile mode is the default and requires no attribute.

---

# Attribute Implementation Types

Attributes usually fall into two categories.

---

## Stat Modifier Attributes (Recommended)

These modify stats through the modifier pipeline.

Example:

```
+damage
+projectile_count
+attack_speed
+penetration
```

These implement:

```
contribute_modifiers(ctx, out_mods)
```

Stat modifiers are:

* deterministic
* easy to debug
* stack safely

Most attributes should use this model.

---

## Event Attributes

These react to gameplay events through the attribute bus.

Examples:

```
bonus damage on hit
spawn explosion on kill
chain lightning
```

Common hooks:

```
on_hit(evt, ctx)
on_attack_start(ctx)
on_projectile_spawn(projectile, ctx)
```

Use these only when stat modifiers cannot express the behavior.

---

# Creating Item Drops

Item drops are created using **ItemSpawner nodes**.

Place a spawner in a scene and configure:

```
base_type
spawn_id
drop_index
pickup_scene
```

Example deterministic identity:

```
spawn_id = "room_2_3_pickup_1"
drop_index = 0
```

The final item seed is derived from:

```
run_seed
spawn_id
drop_index
```

This guarantees that drops are **deterministic across runs**.

---

# Starter Player Loadout

The player’s starting gear is rolled using:

```
InitialLoadoutRoller
```

Configure in the inspector:

```
starter_melee
starter_ranged
starter_spell
starter_shield
```

Each entry should reference a **BaseItemType**.

Example:

```
starter_melee:
  - BaseType_Knife
  - BaseType_Sword

starter_ranged:
  - BaseType_Pistol
  - BaseType_Shotgun
```

The roller will create deterministic item instances at game start.

---

# Authoring Checklist

When creating new content:

### New Weapon

1. Create `ItemDef`
2. Create `BaseItemType`
3. Configure attribute roll rules
4. Add to spawner or starter loadout

---

### New Attribute

1. Create `.tres` resource
2. Follow naming prefix rules
3. Implement stat modifier or event behavior
4. Verify it appears in rolled items

---

# Debug Tips

If an item is not rolling attributes:

Check:

```
BaseItemType.use_auto_attribute_pool
BaseItemType.auto_attribute_root
BaseItemType.weapon_token
min_attribute_count / max_attribute_count
```

Also verify the attribute resource filename prefix matches the pool rules.

---

# Design Philosophy

The system is designed around these principles:

* **Deterministic item generation**
* **Behavior defined by attributes**
* **Strict separation of template vs procedural vs runtime**
* **Editor-driven content creation**
* **Minimal subclassing**

This allows the system to scale to **hundreds of attributes and many weapon types** without increasing code complexity.

---

End of document.
