Perfect — I’ve now read your design document and compared it against what you’ve implemented so far in the project you shared + the systems we built during this session.

Below is a **real production-grade TODO list**, not just features — it’s organized by **architecture maturity order** so you won’t accidentally break your own rules.

(Important: I’m aligning strictly to your architecture rules — entity orchestrator, items drive combat, deterministic systems, etc.) ([GitHub][1])

---

# CURRENT STATE — What You Already Have (Verified)

## Core Architecture

✔ Modular entity component architecture
✔ Movement owned only by Mover ([GitHub][1])
✔ Combat executed by items ([GitHub][1])
✔ Equipment slots validated ([GitHub][1])
✔ Shared player/enemy systems ([GitHub][1])
✔ Interaction system
✔ Pickup swapping algorithm ([GitHub][1])
✔ Momentum preserving attacks ([GitHub][1])

## Combat Features

✔ Melee hitboxes
✔ Projectiles
✔ Spell casting
✔ Shields applying stats
✔ Initial loadouts ([GitHub][1])

## Procedural World (new work)

✔ Deterministic seeded floors
✔ Graph topology generation
✔ Start + boss selection
✔ Room database exit matching
✔ Door collider opening
✔ Player spawn placement
✔ Random seed testing mode

We now officially have:

> A functioning gameplay sandbox engine

---

# NEXT PHASE — Dungeon Gameplay Layer

## Priority 1 — Room Activation System (Critical)

### Needed

* Room trigger area (enter detection)
* Room state machine:

  * UNVISITED
  * ACTIVE
  * CLEARED
* Door lock during combat
* Enemy spawn trigger
* Clear detection

### Why

We currently generate a dungeon but it has no gameplay pacing.

This implements:

> exploration → encounter → reward loop

---

## Priority 2 — Enemy Lifecycle

Our architecture supports enemies, but dungeon needs orchestration.

### Add

* EnemySpawner component (room-owned)
* Spawn tables per room depth
* Room listens for enemy deaths
* Combat clear signal

### Required for

Boss rooms, pacing, progression

---

## Priority 3 — Navigation Between Rooms

Add:

* Door transition triggers
* Camera room snapping
* Player confinement while locked
* Prevent backtracking exploit during combat

---

# CONTENT GENERATION PHASE

## Large Room Footprints

Implement occupancy grid:

* 1×1 small
* 2×1 wide
* 1×2 tall
* 2×2 XL
* L shaped

Spawner must reserve multiple cells before placement.

This unlocks:

> real combat arenas

---

## Room Types (Logic, not art)

Add metadata per room:

* START (safe)
* COMBAT
* TREASURE
* SHOP
* BOSS
* EVENT

Room generator already supports kinds — now they gain behavior.

---

# COMBAT DEPTH PHASE

## Attack Timeline System

We currently attack — but not with standardized phases.

Implement:
Windup → Active → Recovery → Cooldown ([GitHub][1])

Needed for:

* hitstop
* interrupts
* stagger
* charged attacks

---

## Capability Locks Integration

States should only lock capabilities — not logic 

Add:

* CAN_ATTACK
* CAN_MOVE
* CAN_CAST
* STAGGERED

Then:
Combat reads capabilities instead of states.

---

## Stagger / Interrupt System

Required for melee readability and difficulty scaling.

---

## AI Combat Behaviour

Currently we only have boilerplate Chase AI

Add enemy behaviors:

* maintain distance
* dodge windows
* attack timing
* weapon awareness

---

# ITEM SYSTEM EXPANSION


## Attack Behaviour Objects

Weapons must select behaviours, not contain logic ([GitHub][1])

Create reusable behaviours:

* Arc
* Thrust
* Projectile
* Hitscan
* Aura
* Cast

This is REQUIRED before procedural weapons.

---

## Modifier System

Procedural mutation layer ([GitHub][1])

Add:

* Stat modifiers
* Behaviour modifiers
* Projectile converters
* Conditional triggers

---

## Loot Generator

Seeded item rolling ([GitHub][1])

Flow:
Pool → Archetype → Rarity → Modifiers → Bake seed

---

# PROGRESSION LAYER

## Pickups Expansion

Right now we only have equipment pickups.

Add:

* health
* mana
* ammo
* temporary buffs ([GitHub][1])

---

## Inventory UI

From planned systems ([GitHub][1])

Needed before:
shops, builds, strategy

---

## Shops / NPC

Interaction system already supports it.

---

# POLISH & GAME FEEL

## Hitstop & Screen Feedback

Needed after attack timeline.

## Camera Room Framing

Locks camera per room (huge feel improvement)

## Minimap (real UI)

Replace debug draw with CanvasLayer map

## Audio & Telegraphing

Required because animations don't communicate attacks ([GitHub][1])

---

# PERFORMANCE & SCALABILITY

## Object Pooling

Projectiles must be pooled ([GitHub][1])

## Behaviour Resources Shared

No runtime allocations ([GitHub][1])

## Deterministic Simulation Validation

Multiplayer / replay future proofing

---

# FINAL CONTENT PHASE

## Boss Systems

Pattern logic + multi-phase

## Biomes

Different room pools per floor

## Meta Progression

Unlocks + pools

---

# Suggested Implementation Order (Very Important)

1. Room activation + locking
2. Enemy spawn lifecycle
3. Camera room system
4. Large room footprints
5. Attack timeline
6. Stagger / interrupts
7. Behaviour-based weapons
8. Procedural modifiers
9. Loot generation
10. Inventory / UI
11. Shops / NPC
12. Bosses
13. Meta progression

---


[1]: https://github.com/nikoeklof/Roguelike-Design-Document/tree/main "GitHub - nikoeklof/Roguelike-Design-Document"
