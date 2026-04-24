
---

# CURRENT STATE — What We Already Have (Verified)

## Core Architecture

✔ Modular entity component architecture
✔ Movement owned only by Mover
✔ Combat executed by items
✔ Equipment slots validated
✔ Shared player/enemy systems
✔ Interaction system
✔ Pickup swapping algorithm
✔ Momentum-preserving attacks

## Capability System

✔ CAN_MOVE, CAN_ATTACK, CAN_CAST, CAN_BLOCK, CAN_SWITCH
✔ Multi-reason stacking block/unblock (block(cap, reason) / unblock(cap, reason))
✔ States only lock capabilities, never control physics directly

## Combat Features

✔ Melee hitboxes (executor pipeline: windup → active → recovery)
✔ Ranged projectiles (hitscan, beam, projectile modes)
✔ Shields applying stats
✔ Initial loadouts via LoadoutAssigner (seeded, deterministic)
✔ Buff spells: BattleTrance, StoneSkin, Haste, EnemySlowAura
✔ Debuff spells: Frailty, Slow — projectile and AOE delivery modes
✔ EntityVisualController shader params: debuff_intensity, poison_amount, freeze_amount, burn_amount, shield_amount, hurt_flash
✔ SpellEffect base class with duration/potency scaling, timer helpers, visual setters
✔ Spell cooldown per-item (not global), read via spell.get_cooldown_remaining()
✔ Reflection attribute (reflected projectiles reset lifetime)

## Item System

✔ ItemDef / ItemInstance / BaseItemType / ItemTypeRegistry
✔ SpellItemDef with spell_type (BUFF/DEBUFF), core_effect, projectile config, AOE config
✔ Attribute pool system (ItemAttributeBus, ItemAttribute subclasses)
✔ Spell modifier attributes: SpellReducedCooldown, SpellExtendedDuration, SpellAmplifiedPotency
✔ Attribute-based rarity (attribute count → rarity tier)
✔ ItemSpawner — deterministic drops from run_seed + spawn_id + drop_index
✔ ItemTooltipFormatter for expanded item details

## Procedural World

✔ Deterministic seeded floor generation (FloorGenerator)
✔ Graph topology (room graph, START + BOSS selection)
✔ Room database with exit mask matching
✔ Normal rooms (1×1, 800×800 world units) — 16 template variants
✔ XL rooms (2×2, 1600×1600 world units) — 15 template variants
✔ EnemySpawn markers in all 31 room templates (5 per normal, 9 per XL)
✔ PickupSpawn markers in all room templates
✔ Door colliders open/close per exit mask
✔ Player spawn placement at START room

## Room Encounter System

✔ RoomController: IDLE → ACTIVE → CLEARED state machine
✔ Room trigger area (Area2D) covering full room, detects player via "player" group
✔ Seeded per-room combat probability (combat_chance + boss_combat_chance exports on FloorSpawner)
✔ BoI-style entry: block player input → momentum carries player in → lock doors + spawn enemies → unblock
✔ Enemy spawn from EnemySpawn_* markers using seeded RNG
✔ Health.died tracking per enemy, _enemies_remaining counter
✔ encounter_cleared signal — unlocks doors, frees detection area
✔ Room presence tracker: FloorSpawner emits player_room_changed(coord) signal via per-room Area2D

## Debug HUD

✔ Tabbed sections: STATS, SHIELD, BUFFS, ENEMY AI, INV
✔ Map panel separated to top-right corner
✔ Minimap (CanvasLayer-based, draws from FloorPlan)
✔ Player cell tracked via player_room_changed signal (not position math)
✔ Inventory tab: EquipList + item detail panel (ItemTooltipFormatter)
✔ HUD layout: 25% width × 50% height max, tab buttons, font size tuned

## Display

✔ Base resolution 1280×720
✔ stretch/mode = canvas_items, aspect = expand
✔ Fills window at any size; 1.5× on 1080p, 2× on 1440p, 3× on 4K

---

# NEXT PHASE — Dungeon Gameplay Layer

## Priority 1 — Camera Room System

Currently camera follows player freely — no room framing.

Add:
* Camera snaps/lerps to current room bounds on room entry
* Locks to room rect while encounter is ACTIVE
* Smooth transition when moving between rooms

Why:
Room framing is a foundational feel improvement for dungeon pacing.

---

## Priority 2 — Navigation Between Rooms

Currently doors open freely — no transition handling.

Add:
* Door transition trigger areas
* Brief transition (fade or pan) when exiting room through door
* Prevent re-entering a room mid-encounter through a different door

---

## Priority 3 — Room Type Behaviors

Room kinds exist (START, NORMAL, BOSS) but only BOSS and START have non-combat behavior.

Add metadata behavior per kind:
* TREASURE — guaranteed item drop, no combat
* SHOP — NPC interaction, buy with currency
* EVENT — scripted one-off encounters

Room generator already supports kind field — kinds now need behavior nodes.

---

# CONTENT GENERATION PHASE

## Room Footprint Expansion

Currently: 1×1 (normal) and 2×2 (XL).

Planned:
* 2×1 wide
* 1×2 tall
* L-shaped

FloorGenerator occupancy grid must reserve cells before placement. Requires new room template sets.

---

# COMBAT DEPTH PHASE

## Attack Timeline Formalization

Executor pipeline exists (windup → active → recovery) but phases are implicit per executor.

Formalize:
* Explicit timing config on ItemDef
* Hitstop (freeze frames on hit)
* Charged attack variant

---

## Stagger / Interrupt System

Required for melee readability and difficulty scaling.

---

## AI Combat Behavior Expansion

Current AI: alert → chase → basic attack.

Add:
* Maintain optimal distance (ranged enemies)
* Dodge windows (reaction to player attack windup)
* Attack timing variation
* Multi-phase boss patterns

---

# ITEM SYSTEM EXPANSION

## Procedural Loot Generator

Framework exists (BaseItemType, ItemTypeRegistry, ItemSpawner).

Missing:
* Runtime loot roller from enemy death
* Room-depth-scaled difficulty curve
* Drop rate tables per enemy type

Flow: Pool → Archetype → Rarity → Modifiers → Bake seed → Spawn pickup

---

## Attribute Expansion

Current attributes: SpellReducedCooldown, SpellExtendedDuration, SpellAmplifiedPotency, Reflection.

Planned:
* Conditional triggers (on_hit, on_kill, on_low_health)
* Projectile converters (change projectile type)
* Melee-specific: lunge, bleed, AoE on kill

---

# PROGRESSION LAYER

## Pickups Expansion

Currently only equipment pickups exist.

Add:
* Health pickup
* Temporary buff consumables (uses existing SpellEffect infrastructure)

---

## Inventory / Equipment UI

Debug HUD has basic inventory view. Game-facing UI needed.

Required before: shops, builds, strategy decisions.

---

## Shops / NPC

Interaction system already supports it. Needs:
* ShopNPC scene
* Currency resource
* Buy/preview flow

---

# POLISH & GAME FEEL

## Hitstop & Screen Feedback

After attack timeline formalization.
* Hit freeze frames
* Screen shake on heavy hits
* Camera impulse

## Audio & Telegraphing

Required because animations don't communicate attacks.
* Attack windup audio cue
* Hit audio
* Death audio

---

# PERFORMANCE & SCALABILITY

## Object Pooling

Projectiles must be pooled (currently instantiated per shot).

## Deterministic Simulation Validation

Multiplayer / replay future-proofing.

---

# FINAL CONTENT PHASE

## Boss Systems

* Pattern logic per boss type
* Multi-phase transitions

## Biomes

* Different room template pools per floor depth
* Visual + enemy variety per biome

## Meta Progression

* Unlocks between runs
* Persistent pool expansion

---

# Implementation Order (Updated)

1. ~~Room activation + locking~~ ✔ Done
2. ~~Enemy spawn lifecycle~~ ✔ Done
3. Camera room framing
4. Navigation / door transitions
5. Room type behaviors (TREASURE, SHOP, EVENT)
6. Large room footprints (2×1, 1×2, L-shape)
7. Attack timeline formalization + hitstop
8. Stagger / interrupts
9. AI combat behavior expansion
10. Loot generator (enemy drops)
11. Attribute expansion
12. Pickups expansion
13. Inventory / equipment UI
14. Shops / NPC
15. Boss systems
16. Biomes
17. Meta progression