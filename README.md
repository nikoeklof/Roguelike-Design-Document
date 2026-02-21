# Clank Slayer (not final name, working title) Technical Design Document (Godot 4.x)

---

## Project Goal

Create a fully modular, reusable entity architecture where player and enemies share the exact same systems:

* No special‑case player code
* Items exist as real scene nodes (not data blobs)
* Combat is equipment‑driven
* States never control physics
* Animation never controls gameplay

The project prioritizes **architecture stability over content speed**.

---

## Core Principles

### 1) Entity Is Only an Orchestrator

The root entity does not contain gameplay rules.
It only wires components together.

Entity responsibilities:

* Initialize components
* Route events
* Provide component lookup

It does NOT:

* Move the character
* Perform attacks
* Apply damage
* Decide animation

---

### 2) Components Own Behavior

All gameplay logic lives in components:

| System               | Owned By                    |
| -------------------- | --------------------------- |
| Movement physics     | Mover                       |
| Health rules         | Health                      |
| Damage modifiers     | Stats                       |
| Facing visuals       | FacingPointer               |
| Aim direction        | AimRay                      |
| Combat execution     | Combat                      |
| Equipment management | Equipment                   |
| Input/AI             | ControlSource               |
| Logic flow           | FSM (StateHandler + States) |

---

### 3) Items Are Real Scene Nodes

Items are instanced Nodes that:

* can exist in world
* can be equipped
* can be dropped
* can act independently

No duplicated logic between ground and equipped forms.

---

### 4) Physics Authority Rule

Only one system controls velocity:

> Mover is the sole owner of movement physics.

States request permissions via Capabilities, never directly modify velocity.

---

### 5) Aim Separation Rule

| System        | Purpose                      |
| ------------- | ---------------------------- |
| AimRay        | Gameplay targeting direction |
| FacingPointer | Animation direction          |

Gameplay never depends on animation facing.

---

## Entity Architecture

```
CharacterBody2D (Entity)
 ├─ Mover
 ├─ Stats
 ├─ Health
 ├─ FacingPointer
 ├─ AimRay
 ├─ AnimDriver
 ├─ ControlSource
 ├─ Combat
 ├─ Equipment
 ├─ Capabilities
 ├─ Faction/Tags
 └─ StateHandler
```

---

## Finite State Machine

### Core Flow

```
Locomotion
   ↓ attack input
AttackState
   ↓
Combat.try_attack()
   ↓ wait signal
Locomotion
```

States do NOT:

* move entity
* apply forces
* change velocity

States only:

* lock capabilities
* choose actions

---

## Movement System

Movement is permission‑based.

| Capability | Effect                            |
| ---------- | --------------------------------- |
| CAN_MOVE   | Player input acceleration allowed |
| BLOCKED    | Input ignored                     |
| ALWAYS     | Friction still applies            |
| NEVER      | Velocity not zeroed               |

This enables momentum‑preserving attacks.

---

## Combat System

Combat acts as executor only.

```
State → Combat → Equipped Item → Action
```

Combat never knows weapon type details.
Weapons never know FSM state.

---

## Equipment System

Equipment holds slot nodes:

```
Equipment
 ├─ MeleeSlot
 ├─ RangedSlot
 ├─ SpellSlot
 └─ ShieldSlot
```

Slots validate items before equipping.

### Slot Validation Rules

| Slot   | Accepts         |
| ------ | --------------- |
| MELEE  | Weapon (melee)  |
| RANGED | Weapon (ranged) |
| SPELL  | Spell           |
| SHIELD | Shield          |

Invalid swaps are rejected safely.

---

## Item Architecture

### Weapon

Provides:

* try_attack(direction, owner)
* allow_move_during_attack
* pickup_slot_kind

### Spell

Provides:

* try_cast(owner, direction)
* allow_move_during_cast

### Shield

Applies passive stat modifiers on equip.

---

## Interaction System

Interactor Area detects nearby interactables.

```
Player
 └─ InteractionArea (Interactor)
```

Flow:

```
Input → Interactor → interactable.interact(entity)
```

Interactables include:

* pickups
* doors
* NPCs
* future UI prompts

---

## Pickup System

GroundItemPickup contains an actual item node internally.

Swap algorithm:

```
player presses interact
→ pickup determines slot
→ equipment.swap_item_in_slot()
→ old item becomes new pickup
```

Visuals are inherited automatically from the item sprite.

---

## Combat Movement Policy

Movement permissions come from the item, not the state.

| Item Property            | Effect           |
| ------------------------ | ---------------- |
| allow_move_during_attack | Steering allowed |
| false                    | momentum only    |

---

## Current Working Features

* Equipment swapping
* Shared player/enemy combat
* Melee hitboxes
* Projectile weapons
* Spell casting
* Shields applying stats
* Momentum‑preserving attacks
* Interaction system
* Initial loadouts

---

## Planned Systems

### Combat Expansion

* Attack variants (light/heavy/charged)
* Combos
* Stagger / interrupt
* AI combat behavior

### Player Systems

* UI prompts
* Inventory UI
* Equipment screen

### World Systems

* Doors
* NPC dialogue
* Item pickups expanded (health, ammo, mana etc.)

---

## Architectural Rules (Strict)

1. States never control physics
2. Weapons never talk to FSM
3. Animation never drives gameplay
4. AimRay is gameplay authority
5. FacingPointer is visual only
6. Items are nodes, not resources
7. Entity contains no gameplay logic

---

## Long‑Term Scalability Goal

This architecture should support:

* hundreds of enemies
* procedural weapons
* procedural levels with set room layouts 
* deterministic simulation
without rewriting core systems.

Below is the same addendum rewritten as **plain Markdown** so you can directly copy-paste into your repo without UI blocks or special formatting.

It intentionally matches a GitHub-friendly style.

---

# Procedural Equipment & Attack System

## Purpose

Extend the modular entity architecture with a scalable item-generation system capable of supporting:

* Shared player/enemy equipment
* Droppable enemy weapons usable by player
* Behaviour-changing upgrades
* Large weapon counts without animation explosion
* Minimal art production cost
* Deterministic generation

The system must produce hundreds of gameplay-distinct weapons while keeping implementation complexity linear.

---

## Core Philosophy

### Items define combat — actors only execute

The entity does not attack.
The item performs the attack.

Flow:

State → Combat → Equipped Item → Attack Behaviour → Effects

This extends the existing rule that animation never drives gameplay into:

**Actors never define attacks**

---

## Separation of Responsibilities

| Layer            | Responsibility            |
| ---------------- | ------------------------- |
| Actor            | Movement, facing, sockets |
| Combat           | Requests attack           |
| Item             | Selects behaviour         |
| Attack Behaviour | Timing + hit logic        |
| Projectile       | Motion & impact           |
| Modifier         | Mutates behaviour         |

This prevents combinatorial explosion:

actors × weapons × upgrades
becomes
actors + weapons + upgrades

---

## Item Structure

An item is composed of five independent parts:

Item Instance

* Base Archetype
* Attack Behaviour
* Projectile (optional)
* Modifiers (0-N)
* Visual Overrides

Items are assembled at spawn time and remain immutable afterward (except durability/charges).

---

## Attack Behaviour Model

Attacks are reusable behaviours parameterized by items.

### Behaviour categories

| Type         | Description       |
| ------------ | ----------------- |
| MELEE_ARC    | Sweeping hitbox   |
| MELEE_THRUST | Forward stab      |
| HITSCAN      | Instant ray       |
| PROJECTILE   | Spawned object    |
| CAST         | Delayed effect    |
| BLOCK        | Defensive state   |
| AURA         | Persistent effect |

Weapons never implement logic — they select behaviours.

---

## Attack Timeline

Every attack follows the same timeline:

Windup → Active → Recovery → Cooldown

The behaviour defines timings, not animation.

Different weapons feel unique by changing:

* Windup
* Hitstop
* Movement lock
* Impulse
* Screen shake

Not by adding new animations.

---

## Projectile System

Projectiles are standalone simulation objects.

They contain:

| Property  | Purpose            |
| --------- | ------------------ |
| Motion    | Velocity model     |
| Collision | Hit rules          |
| Lifetime  | Expiry             |
| Impact    | Effects            |
| Status    | Applied conditions |

An attack only spawns them.

This allows:

same weapon + different projectile = new weapon

---

## Modifier System (Procedural Upgrades)

Modifiers mutate the generated item.

### Modifier types

| Category    | Effect               |
| ----------- | -------------------- |
| Stat        | Numeric scaling      |
| Behaviour   | Change attack params |
| Conversion  | Swap projectile      |
| Conditional | Add triggers         |
| Visual      | Palette / glow       |

Modifiers never contain gameplay loops — only parameter edits.

---

## Modifier Application Order

Order is deterministic and required for reproducibility:

1. Base stats
2. Flat additions
3. Multipliers
4. Behaviour overrides
5. Projectile overrides
6. Visual overrides
7. Naming

This guarantees reproducible items from seeds.

---

## Naming Generation

Names are assembled as:

[prefix] [base name] [suffix]

Example:
Brutal Scrap Pistol of Splitting

Modifiers may contribute:

* Prefix
* Suffix
* Descriptor

---

## Visual System Integration

Art pipeline rule:

**Weapons animate — characters move**

Characters provide:

* Hand sockets
* Facing direction

Weapons provide:

* Attack frames
* Trails
* Flash frames

This guarantees:
New weapon = zero character animation work

---

## Enemy Equipment Integration

Enemies do not have special weapons.

Enemy = Actor + AI + Equipment

They spawn with rolled items from a loot pool.

When killed:
Drop item instance → player equips same object

No conversion required.

---

## Loot Generation

Roll process:

1. Choose pool
2. Choose archetype
3. Roll rarity
4. Roll modifier count
5. Pick allowed modifiers
6. Apply modifiers
7. Bake seed

Result: deterministic item instance

---

## Rarity Scaling

Rarity controls modifier count, not stat multipliers.

| Rarity    | Mods |
| --------- | ---- |
| Common    | 0–1  |
| Rare      | 1–2  |
| Epic      | 2–3  |
| Legendary | 3–5  |

Balance remains predictable.

---

## Balance Rule

Stats scale slowly.
Behaviour changes scale dramatically.

Variety should come from mechanics, not numbers.

---

## Animation Policy

Characters never have attack animations.

Characters only animate:

* Locomotion
* Hitstun
* Death

Attacks are communicated via:

* Weapon motion
* Flashes
* Audio
* Particles
* Timing pauses

This keeps hundreds of weapons readable.

---

## Data-Driven Expansion

Adding a new weapon requires:

* 1 sprite
* 1 JSON definition

No new code.

Adding a new mechanic requires:

* 1 behaviour OR projectile

Then hundreds of weapons can use it.

---

## Performance Considerations

To maintain performance:

* Projectiles must be pooled
* Behaviours shared as resources
* Modifiers applied only at spawn

Runtime combat must not allocate memory.

---

## Long-Term Benefits

This system enables:


* Enemy weapon variety
* Scalable content production
* Future mod support

Without rewriting core combat.

---

## Design Outcome

Three independent axes of variety:

Actors (AI behaviour)
Items (attack behaviour)
Modifiers (rule mutations)

