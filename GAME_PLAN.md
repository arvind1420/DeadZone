# DEAD ZONE: LAST COLONY
## Complete Game Design & Technical Plan

---

## 1. EXECUTIVE SUMMARY

**Genre:** Base-Building + Real-Time Defense Strategy
**Target Platform:** Mobile (PWA — installable on iOS & Android home screen)
**Monetization:** Freemium with IAP (premium currency, hero gacha, speed-ups, VIP)
**Inspiration:** State of Survival, Last Shelter: Survival, Rise of Kingdoms
**Tech Stack:** Pure HTML5 + CSS3 + Vanilla JavaScript — zero dependencies, single-file deliverable
**Target Devices:** iOS 14+, Android 9+, modern desktop browsers

**Core Fantasy:** You are the commander of the last survivor colony after civilization collapsed. Build your base, recruit legendary heroes, and defend against endless zombie hordes, random zombie stampedes, and ruthless rival factions fighting over resources.

---

## 2. PLAYER EXPERIENCE GOALS

| Goal | Implementation |
|------|---------------|
| Immediate satisfaction | Base looks alive on first load — buildings animated, resources ticking |
| Strategic depth | 6 hero classes, 15 research nodes, 4 event types create complex decisions |
| Tension & urgency | Herd warnings, rival raid alerts, low-resource danger states |
| Progression reward | Each upgrade visually improves the building's appearance |
| Monetization feel | Premium elements feel rewarding, not punishing — pay-to-convenience not pay-to-win |
| Session loops | 2-minute battle + 10-minute base management = satisfying mobile session |

---

## 3. FILE STRUCTURE

```
/Games
├── index.html          ← Main game (HTML + CSS + JS, ~5000 lines, self-contained)
├── manifest.json       ← PWA manifest (name, icons, orientation, colors)
├── sw.js               ← Service worker (offline caching, background sync)
├── icon-192.png        ← PWA icon 192×192 (generated from canvas, saved once)
├── icon-512.png        ← PWA icon 512×512 (generated from canvas, saved once)
└── GAME_PLAN.md        ← This file
```

> **Note:** All sprites, sounds, and assets are generated programmatically in JavaScript at runtime. No external asset files are needed. This keeps the game self-contained and PWA-installable from a static file host.

---

## 4. GAME SCREENS / VIEWS

The game has 5 primary views, all rendered into the same canvas + HTML layer combination. Navigation is via the bottom tab bar.

```
┌──────────────────────────────────────────────────────────────┐
│  🍖 2,400  🔫 1,100  ⚙️ 1,800  💎 80    [⚙] [👑 VIP]       │  ← Resource Bar (HTML)
├──────────────────────────────────────────────────────────────┤
│  ⚠️ ZOMBIE HERD APPROACHING! ETA: 45s  [PREPARE DEFENSE]     │  ← Alert Bar (HTML, hidden normally)
├──────────────────────────────────────────────────────────────┤
│                                                              │
│                                                              │
│                 MAIN CANVAS (480×480)                        │  ← Canvas (scales with CSS)
│            Renders active view content                       │
│                                                              │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│   [🏠 BASE]  [🗺️ MAP]  [⚔️ HEROES]  [🔬 TECH]  [💎 SHOP]   │  ← Nav Bar (HTML)
└──────────────────────────────────────────────────────────────┘
```

### 4.1 BASE VIEW (Default)

The player's colony rendered top-down. Buildings are placed at fixed positions within the canvas, each with unique pixel-art-style sprites drawn via canvas 2D API.

**Layout (480×480 canvas):**

```
     ┌──────────────────────────────────────────────────────────┐
  20 │  ╔══════════════════ OUTER WALL ═══════════════════════╗ │
     │  ║  [ARMORY]         [WATCHTOWER]        [HOSPITAL]    ║ │
     │  ║                                                     ║ │
     │  ║  [FARM]        [COMMAND CENTER]      [SCRAPYARD]    ║ │
     │  ║                                                     ║ │
     │  ║  [BARRACKS]    [POWER PLANT]         [WORKSHOP]     ║ │
     │  ║                                                     ║ │
     │  ╚═════════════════════════════════════════════════════╝ │
 460 └──────────────────────────────────────────────────────────┘
```

**Building Slot Positions (center x, center y):**
| Building | Center X | Center Y | Width | Height |
|----------|----------|----------|-------|--------|
| Command Center | 240 | 240 | 72 | 78 |
| Farm | 90 | 240 | 76 | 58 |
| Scrapyard | 390 | 240 | 76 | 58 |
| Barracks | 120 | 340 | 68 | 62 |
| Hospital | 360 | 130 | 68 | 62 |
| Workshop | 120 | 350 | 68 | 62 |
| Armory | 360 | 350 | 68 | 62 |
| Watchtower | 240 | 90 | 52 | 72 |
| Power Plant | 240 | 390 | 64 | 58 |
| Outer Wall | perimeter | perimeter | full | full |

**Interactions:**
- Tap/click a building → opens Building Modal (details, upgrade, info)
- Long-press empty area → context menu (decor info)
- Buildings glow/pulse when ready to upgrade
- Construction animation while upgrading (scaffolding overlay)

### 4.2 BATTLE VIEW (Triggered by events)

The base view transforms into the battle arena. Enemies flood in from outside the wall.

```
  Enemies spawn       ┌──────────────────────────┐  Enemies spawn
  from top edge  →    │  ╔════ WALL ════════╗     │  ← from right
                      │  ║  🏛️  BUILDINGS   ║     │
                      │  ║     in center    ║     │
  Enemies spawn  →    │  ╚═════════════════╝     │
  from left edge      │                          │
                      └──────────────────────────┘
                             ↑ Enemies spawn from bottom
```

**Battle HUD (overlaid HTML):**
```
┌────────────────────────────────────────────────────────────┐
│ [Marcus⚔️ READY] [Rena🛡️ 8s] [Dr.Kim💊 READY] [Ghost🎯 4s] │  ← Hero portraits + ability buttons
│ ⚔️ Wave 3   👥 Enemies: 18   🧱 Wall: 67%   [RETREAT ↩️]  │  ← Battle info bar
└────────────────────────────────────────────────────────────┘
```

### 4.3 WORLD MAP VIEW

A procedurally generated map grid (9×9 tiles = 81 tiles) showing the surrounding territory.

**Tile Types:**
- 🏚️ Ruins — Scavenge for metal
- 🌲 Forest — Scavenge for food
- 🏪 Store — Scavenge for ammo
- 🔴 Rival Base — Attack for all resources + loot
- 🟡 Supply Drop — Free timed resource pickup
- 🆘 SOS Signal — Rescue mission (chance to recruit hero)
- ⚫ Unexplored — Fog of war, send scout first
- 🏠 Your Base — Center tile (always revealed)

**Interactions:**
- Tap tile → shows tile info and action button
- Send hero squad on timed missions (timer: 1–10 minutes real-time)
- Attack rival bases (triggers auto-battle, hero vs. defenders)
- Supply drops expire after 2 hours (shows countdown)

### 4.4 HEROES VIEW

A card-based roster of all recruited heroes.

**Layout:**
- Hero cards in a scrollable 2-column grid
- Each card shows: portrait, name, class, rarity stars, level, HP bar
- Tap card → hero detail modal (stats, ability description, equip gear, level up)
- Bottom: [RECRUIT x1 — 100💎] [RECRUIT x10 — 900💎] buttons
- Gacha animation plays when recruiting (card flip reveal with rarity effects)

**Hero Detail Modal:**
```
┌─────────────────────────────────┐
│  [HERO PORTRAIT]  Ghost         │
│   ★★★ EPIC       Lv. 12        │
│   Class: Sniper                 │
│   ━━━━━━━━━━━━━━━━━━━━━━━━      │
│   HP: 90/90  DMG: 55  RNG: 200 │
│   XP: 450/600  ──────────░      │
│   ━━━━━━━━━━━━━━━━━━━━━━━━      │
│   ABILITY: Headshot             │
│   Instantly kills 1 enemy.      │
│   Cooldown: 8s                  │
│   ━━━━━━━━━━━━━━━━━━━━━━━━      │
│   [⚔️ ADD TO ROSTER]            │
│   [📈 LEVEL UP — 200🥫]         │
└─────────────────────────────────┘
```

### 4.5 RESEARCH VIEW (TECH TREE)

A visual node tree rendered on canvas with three branches.

```
                    [START]
                   /   |   \
              [MIL]  [ENG]  [MED]
             /   \   / \    / \
           [dmg1][trp] [bld][wall] [heal][hp]
            /             \         \
         [dmg2]          [wall2]   [revive]
          /               |          \
       [armor1]        [turret1]    [medic1]
                          |
                       [prod1]
```

**Node States:** Locked (gray) → Available (glowing outline) → Researching (progress bar) → Unlocked (colored, checkmark)
**Interactions:** Tap node → shows research modal with cost, time, effect, [RESEARCH] button
**Requires Workshop building at appropriate level**

### 4.6 SHOP VIEW

Professional IAP storefront rendered in HTML.

**Tabs:** Credits | Speed-Ups | Resources | VIP | Heroes

```
┌─────────────────────────────────┐
│  💎 CREDITS                     │
│ ┌─────────────┐ ┌─────────────┐ │
│ │  💎 80      │ │  💎 500     │ │
│ │  Starter    │ │  Explorer   │ │
│ │  $0.99      │ │  $4.99      │ │
│ │  [BUY]      │ │  [BUY]      │ │
│ └─────────────┘ └─────────────┘ │
│ ┌─────────────┐ ┌─────────────┐ │
│ │  💎 1,200   │ │  💎 2,800   │ │
│ │  Commander  │ │  Elite      │ │
│ │  $9.99      │ │  $19.99     │ │
│ │  [BUY]      │ │  [BUY ⭐]   │ │
│ └─────────────┘ └─────────────┘ │
└─────────────────────────────────┘
```

---

## 5. GAME SYSTEMS — DETAILED SPECS

### 5.1 RESOURCE SYSTEM

**Four resource types:**

| Resource | Icon | Produced By | Capacity | Initial | Used For |
|----------|------|-------------|----------|---------|----------|
| Food | 🍖 | Farm | 5,000 | 2,000 | Hero upkeep, building some structures |
| Ammo | 🔫 | Armory | 3,000 | 1,000 | Battle cost, building defenses, research |
| Metal | ⚙️ | Scrapyard | 5,000 | 1,500 | All construction, walls |
| Credits | 💎 | Purchased / rare drops | unlimited | 80 | Premium actions |
| Rations | 🥫 | Gameplay rewards | 99,999 | 500 | Soft currency for upgrades, hero level-up |

**Production formula:**
```
productionPerSecond = buildingLevel × baseRate × powerMultiplier × researchBonus
powerMultiplier = (Power Plant level > 0) ? 1.0 + (0.1 × powerLevel) : 1.0
researchBonus = 1.0 + sum(all active production research bonuses)
```

**Production rates per second (at level 1):**
- Farm: +0.08 food/sec (288/hour)
- Armory: +0.05 ammo/sec (180/hour)
- Scrapyard: +0.06 metal/sec (216/hour)

**Danger state:** When food < 10% of capacity, all hero HP regeneration stops and a warning displays.

**Resource loot on rival attack:** Attacker steals up to 20% of target's resources if they win.

### 5.2 BUILDING SYSTEM

**10 Buildings, each upgradeable Level 0→10 (Level 0 = not built):**

#### Command Center
- **Function:** Gating mechanic — max level of all other buildings = CC level
- **Visual:** 3-story stone fortress, lit windows, radio antenna, flag, barricades at base
- **Special:** Level 5 unlocks World Map. Level 8 unlocks Alliance features.
- **Upgrade cost formula:** `baseCost × 1.8^(level-1)`

| Level | Food | Ammo | Metal | Build Time | Bonus |
|-------|------|------|-------|------------|-------|
| 2 | 500 | 200 | 800 | 2 min | Unlocks Armory slot |
| 3 | 900 | 360 | 1,440 | 4 min | +500 all resource caps |
| 4 | 1,620 | 648 | 2,592 | 8 min | Unlocks Watchtower slot |
| 5 | 2,916 | 1,166 | 4,666 | 16 min | Unlocks World Map |
| 6–10 | +80% each | +80% each | +80% each | ×2 each | Various bonuses |

#### Barracks
- **Function:** Trains generic troops (non-hero defenders). Max troops = level × 3
- **Visual:** Military green building, sandbags at entrance, bunk beds visible in window, military flag
- **Troop types:** Militia (weak, free) → Rifleman (stronger, ammo cost) → Elite (best, ammo + metal)
- **Troop training time:** 30 seconds to 5 minutes depending on type

#### Hospital
- **Function:** Heals injured heroes over time. Faster healing at higher levels.
- **Visual:** White building, red cross symbol, ambulance parked outside
- **Heal rate:** `1 HP/sec × level`
- **Special:** Level 5 allows reviving dead heroes (costs rations)

#### Workshop
- **Function:** Required for Research. Level gates which research branches are available.
- **Visual:** Industrial building with gear symbol, pipes, bright interior light
- **Research unlock:** Level 1 = Military branch; Level 3 = Engineering; Level 5 = Medicine

#### Farm
- **Function:** Passive food production
- **Visual:** Green field with crop rows, small farmhouse building, fence
- **Production:** 0.08 × level food/sec

#### Scrapyard
- **Function:** Passive metal production
- **Visual:** Rusty orange building, metal scraps around it, conveyor visible, steam vent
- **Production:** 0.06 × level metal/sec

#### Armory
- **Function:** Passive ammo production + increases max ammo cap
- **Visual:** Dark stone fortress-style, metal door with lock, ammo crates stacked outside, arrow slits
- **Production:** 0.05 × level ammo/sec
- **Cap bonus:** +500 ammo per level

#### Watchtower
- **Function:** (1) Increases event warning time by 10s per level. (2) With Research: fires auto-turret at enemies during battle.
- **Visual:** Tall narrow tower, wooden/metal frame, spotlight on top, ladder detail
- **Warning time base:** 30 seconds + (10s × level)
- **Auto-turret stats:** 12 dmg/shot, 1 shot/sec, unlimited ammo

#### Power Plant
- **Function:** Multiplies production of all other buildings
- **Visual:** Dark blue industrial building, lightning bolt symbol, generators visible, power lines going out
- **Multiplier:** 1.0 + (0.1 × level) (Level 5 = 1.5× all production)

#### Outer Wall
- **Function:** First line of defense. Divided into 4 sections (N/S/E/W). Each section has independent HP.
- **Visual:** Stone/concrete perimeter rectangle with battlements, corner watchtowers, gates on each side
- **Wall HP per section:** 100 × level (Level 1 = 100 HP, Level 5 = 500 HP per section)
- **Repair cost:** 10 metal per 10 HP restored
- **Special:** When a section reaches 0 HP, enemies breach through and target buildings directly

---

### 5.3 HERO SYSTEM

**6 hero classes, recruited via gacha:**

#### Hero Roster

| ID | Name | Class | Rarity | HP | DMG | Range | Ability | Cooldown |
|----|------|-------|--------|-----|-----|-------|---------|----------|
| gunner | Marcus "Ironside" | Gunner | ⭐ Common | 120 | 22 | 130px | **Suppressing Fire** — Fires 8 rapid shots in a 45° cone dealing 50% damage each. Duration 3s. | 12s |
| defender | Rena "Bulwark" | Defender | ⭐ Common | 220 | 12 | 60px | **Shield Wall** — Absorbs up to 150 damage for all nearby heroes for 4s. | 15s |
| medic | Dr. Kim | Medic | ⭐⭐ Rare | 100 | 10 | 100px | **Emergency Heal** — Restores 40 HP to all deployed heroes instantly. | 18s |
| engineer | Tomas "Builder" | Engineer | ⭐⭐ Rare | 140 | 18 | 110px | **Deploy Turret** — Places a temporary turret at his position dealing 20 dmg/shot for 8s. | 20s |
| sniper | Ghost | Sniper | ⭐⭐⭐ Epic | 90 | 55 | 200px | **Headshot** — Instantly kills 1 enemy (up to Giant tier). Cannot kill boss-tier rivals. | 8s |
| commander | Col. Fox | Commander | ⭐⭐⭐⭐ Legendary | 160 | 20 | 120px | **War Cry** — Increases all deployed heroes' damage by 50% for 6s. | 25s |

#### Hero Gacha Pull Rates
| Rarity | Rate | Pull Cost |
|--------|------|-----------|
| Common | 60% | 100💎 single / 900💎 x10 |
| Rare | 28% | — |
| Epic | 10% | — |
| Legendary | 2% | — |

**Pity system:** Guaranteed Legendary every 50 pulls. Counter persists across sessions.

#### Hero Progression
- **XP gain:** +10 XP per kill, +50 XP per wave survived
- **Level up cost:** `100 × level` Rations
- **Level bonuses:** +5% HP and DMG per level, max level 30
- **Hero HP regenerates** in Hospital between battles at rate of `1 HP/sec × hospitalLevel`

#### Defense Roster
- Up to 4 heroes can be assigned to the defense roster
- Each hero defends one wall sector (N/S/E/W)
- Unassigned heroes do NOT fight in base defense battles
- Heroes assigned to World Map missions are unavailable for defense (strategic risk)

---

### 5.4 ENEMY SYSTEM

**6 enemy types with distinct behaviors:**

| Type | HP | DMG | Speed (px/s) | Size (px) | XP | Special Behavior |
|------|-----|-----|--------------|-----------|-----|-----------------|
| Crawler | 40 | 8 | 30 | 10 | 10 | Horde enemy. Spawns in large groups (20–60). Walks directly toward wall. |
| Runner | 20 | 6 | 90 | 8 | 15 | Bypasses wall if a gap exists. Targets heroes directly. Hard to hit. |
| Brute | 180 | 25 | 18 | 18 | 35 | Attacks wall only. Ignores heroes unless heroes attack it. Knocks heroes back. |
| Spitter | 60 | 14 | 35 | 12 | 25 | Ranged (100px). Stays outside wall, fires acid projectiles. Poisons heroes (3 dmg/s). |
| Giant | 600 | 50 | 12 | 26 | 120 | Boss. Destroys wall section in 3 hits. AoE stomp attack. Screen shakes on stomp. |
| Rival Soldier | 80 | 20 | 50 | 10 | 20 | Uses cover (moves to nearest obstacle). Fires accurate ranged shots. Squad of 4–8. |

#### Enemy Pathfinding
- **Algorithm:** Simple vector-field flow. Each enemy calculates direction to nearest wall section.
- **Wall contact:** When enemy reaches wall, it switches to "attacking wall" state and deals damage to that wall section per attack tick.
- **Wall breached:** When wall section HP = 0, enemies in that lane move toward the Command Center.
- **Separation:** Enemies maintain minimum spacing (push force when overlapping) to avoid piling.

#### Enemy Spawn Positions
Enemies spawn in the 20px margin outside the outer wall:
- **North spawns:** y = 5–15, x = random between 25 and 455
- **South spawns:** y = 465–475, x = random
- **East spawns:** x = 465–475, y = random between 25 and 455
- **West spawns:** x = 5–15, y = random

---

### 5.5 EVENT SYSTEM

**4 types of attack events, spawned on timers:**

#### Event 1: Zombie Wave (Daily)
- **Trigger:** Every 3 real-time minutes (simulating a "day")
- **Warning:** 30s alert banner, red color
- **Composition:** Mostly Crawlers, some Runners, scaling with day number
- **Formula:** `total enemies = 5 + (dayNumber × 3)`, max 80

| Day Range | Composition |
|-----------|-------------|
| 1–3 | 100% Crawlers |
| 4–6 | 70% Crawlers, 30% Runners |
| 7–10 | 50% Crawlers, 30% Runners, 20% Brutes |
| 11–15 | Add 5% Spitters |
| 16+ | Add Giant (1 per 5 days) |

#### Event 2: Zombie Herd (Random)
- **Trigger:** Random timer 5–15 minutes, weighted toward longer gaps
- **Warning:** 60s alert banner, orange color, pulsing animation
- **Scale:** 3× normal wave size — 80 to 150 enemies
- **Approach:** All 4 wall sides attacked simultaneously
- **Reward:** +200% XP and ration rewards on victory
- **Retreat penalty:** Lose 30% of current resources

#### Event 3: Rival Raid
- **Trigger:** Every 8–12 minutes, only after Day 5
- **Warning:** 45s alert banner, purple color
- **Composition:** 8–20 Rival Soldiers with a leader (buffed stats)
- **Behavior:** Rivals use cover mechanics, flanking moves, coordinated assault
- **Leader:** Has 2× HP and uses a "Call for Backup" ability (spawns 4 more soldiers once)
- **Reward:** Capture rival flag item (cosmetic), +500 Rations, loot from their supplies

#### Event 4: Chain Event (Rare)
- **Trigger:** 10% chance when a Zombie Herd event is active and Rival Raid timer expires simultaneously
- **Warning:** "DOUBLE THREAT!" banner with red/purple split color
- **Effect:** Both events play simultaneously — zombie herd from south, rival raid from north
- **This is intentionally overwhelming** — tests player preparation and hero ability synergies
- **Reward:** 500💎 credits + exclusive cosmetic badge if survived

#### Event 5: Infected Outbreak (Non-combat)
- **Trigger:** Random, every 10–20 minutes
- **Effect:** 1–3 random buildings get "infected" status (production halted, building shown with green glow)
- **Resolution:** Player must tap infected building and spend 50 Ammo to clear each infection within 3 minutes
- **Failure penalty:** Building loses 1 level of upgrades (not destroyed, just de-leveled)

---

### 5.6 BATTLE ENGINE — TECHNICAL SPEC

```
Battle State Machine:
  IDLE → [event triggers] → PREPARING (30–60s warning) → ACTIVE → [all enemies dead] → VICTORY
                                                                 → [base HP = 0] → DEFEAT
                                                         → [player retreats] → RETREAT
```

**PREPARING phase:**
- Alert banner shows with countdown
- Player can reposition heroes in roster (drag and drop)
- Player can spend resources on emergency repairs (wall HP)
- "PREPARE DEFENSE" button opens quick prep modal

**ACTIVE phase loop (60fps):**
1. Update all enemy positions (move toward target)
2. Check enemy-wall collision (damage wall sections)
3. Check enemy-hero collision (damage heroes, trigger knockback)
4. Hero auto-attack cycle (target nearest enemy in range, fire projectile)
5. Update all projectiles (move, check hit, apply damage)
6. Check hero ability activations (player tap input)
7. Update all particles (move, decay)
8. Check win/lose conditions
9. Render everything

**Hero auto-attack behavior:**
- Each hero scans for the nearest enemy within their range every 0.5s
- If target found: hero rotates toward it, fires projectile
- Projectile travels at 250px/s toward target's position at time of firing
- On hit: enemy takes damage, small blood particle burst spawns

**Ability activation:**
- Player taps hero portrait in battle HUD
- If cooldown = 0 and hero is alive: ability triggers immediately
- Cooldown starts counting down
- Visual: ability circle around hero + unique effect particles

**Screen shake:** Triggered by Giant stomp, explosions, player base taking heavy damage. Intensity scales with event type.

**Wall breach visual:** When wall section HP drops to 0:
1. Screen shake (medium intensity)
2. Explosion particles at breach point
3. Wall section color changes to broken/rubble appearance
4. "WALL BREACHED — [SECTOR]" toast notification

---

### 5.7 RESEARCH SYSTEM — 15 NODES

```
MILITARY BRANCH (Workshop Level 1+)
  [Weapon Mods] ─── [Armor Piercing] ─── [Advanced Ordinance]
                          └── [Kevlar Vests]
  [Squad Training] ─── [Elite Units]

ENGINEERING BRANCH (Workshop Level 3+)
  [Fast Builders] ─── [Efficient Systems]
  [Reinforced Steel] ─── [Electric Fence] ─── [Auto Turrets]

MEDICINE BRANCH (Workshop Level 5+)
  [First Aid Kits] ─── [Supplements] ─── [Infection Resist]
              └── [Quick Revival] ─── [Field Medics]
```

**Research Node Details:**

| Node ID | Name | Branch | Requires | Cost (ammo/metal) | Time | Effect |
|---------|------|--------|----------|-------------------|------|--------|
| dmg1 | Weapon Mods | Military | Workshop L1 | 300💰/200⚙️ | 2min | +15% all hero damage |
| dmg2 | Armor Piercing | Military | dmg1 | 600💰/400⚙️ | 5min | +25% additional hero dmg |
| adv_ord | Advanced Ordinance | Military | dmg2 | 1200💰/800⚙️ | 10min | Bullets pierce 2 enemies |
| kevlar | Kevlar Vests | Military | dmg1 | 400🍖/500⚙️ | 4min | +20% hero HP |
| troop1 | Squad Training | Military | Workshop L1 | 400🍖/300⚙️ | 3min | Barracks capacity +2 troops |
| troop2 | Elite Units | Military | troop1 | 800🍖/600⚙️ | 6min | Barracks capacity +3 troops |
| build1 | Fast Builders | Engineering | Workshop L3 | 400⚙️/100💰 | 3min | -20% all build/upgrade time |
| prod1 | Efficient Systems | Engineering | build1 | 700⚙️/200💰 | 8min | +25% all resource production |
| wall1 | Reinforced Steel | Engineering | Workshop L3 | 600⚙️/200💰 | 5min | +50% wall max HP |
| wall2 | Electric Fence | Engineering | wall1 | 1000⚙️/400💰 | 10min | Enemies stunned 1s on wall contact |
| turret1 | Auto Turrets | Engineering | wall2 | 800⚙️/600💰 | 12min | Watchtower fires 12dmg/s at enemies |
| heal1 | First Aid Kits | Medicine | Workshop L5 | 400🍖/200💰 | 3min | Hero heal speed +30% in Hospital |
| hp1 | Supplements | Medicine | heal1 | 600🍖/300⚙️ | 5min | +15% all hero max HP |
| revive1 | Quick Revival | Medicine | heal1 | 800🍖/300💰 | 7min | Hero revival cost/time -50% |
| field_heal | Field Medics | Medicine | revive1 | 1000🍖/400💰 | 10min | Troops slowly regenerate HP in battle |
| infect_im | Infection Resist | Medicine | hp1 | 800🍖/400⚙️ | 8min | Base immune to Infected Outbreak events |

---

### 5.8 WORLD MAP SYSTEM

**Map grid:** 9×9 tiles (81 total), centered on player base
**Fog of war:** Only tiles adjacent to revealed tiles are visible (Manhattan distance)
**Scout:** Send any hero on a 1-minute "scout" mission to reveal a tile

**Mission types:**

| Tile | Mission Name | Hero Required | Time | Resources Gained | Risk |
|------|-------------|---------------|------|-----------------|------|
| 🏚️ Ruins | Scavenge Metal | Any 1 hero | 3 min | +200–500 metal | 20% injury chance |
| 🌲 Forest | Gather Food | Any 1 hero | 2 min | +150–400 food | 10% injury chance |
| 🏪 Supply Store | Loot Ammo | Any 1 hero | 4 min | +300–600 ammo | 30% injury chance |
| 🟡 Supply Drop | Collect Drop | Any 1 hero | 1 min | +100 all resources | 5% injury chance |
| 🆘 SOS Signal | Rescue Mission | 2+ heroes | 5 min | Recruit a hero (Common/Rare) | Hero may take damage |
| 🔴 Rival Base | Raid | 4 heroes | 8 min | Steal 20% rival resources + loot | Heroes may be injured/KIA |

**Injury system:**
- Injured hero loses 30–70% HP and cannot battle or go on missions until healed in Hospital
- KIA (rare, only on Rival Base raids): Hero "unconscious" for 10 real-time minutes, then recoverable at Hospital for 200 Rations

**Rival bases** (4 on the map, progressively harder):
- Bronze Rival: 80 soldiers, mediocre defense, easy loot
- Silver Rival: 150 soldiers, some brutes, moderate loot
- Gold Rival: 250 soldiers, brutes + snipers, high loot
- Diamond Rival: 400+ soldiers, all types, legendary loot

---

### 5.9 MONETIZATION SYSTEM

#### Currency Design
| Currency | Symbol | Source | Use |
|----------|--------|--------|-----|
| Credits | 💎 | IAP, rare events | Gacha pulls, speed-ups, resource packs, VIP |
| Rations | 🥫 | Gameplay | Hero level-up, soft upgrades, troop training |

#### IAP Products

**Credits Bundles:**
| Pack | Credits | Price | Notes |
|------|---------|-------|-------|
| Starter Pack | 80💎 | $0.99 | One-time purchase bonus: 200 extra credits |
| Explorer Pack | 500💎 | $4.99 | Best per-dollar value |
| Commander Pack | 1,200💎 | $9.99 | Includes 1 Rare Hero guaranteed ticket |
| Elite Pack | 2,800💎 | $19.99 | Includes 1 Epic Hero guaranteed ticket |

**Speed-Ups:**
| Item | Cost | Effect |
|------|------|--------|
| Build Skip (30min) | 30💎 | Skip 30 minutes of construction/research |
| Build Skip (1hr) | 50💎 | Skip 1 hour |
| Build Skip (8hr) | 250💎 | Skip 8 hours |
| Full Skip | `ceil(minutes/60) × 50` | Skip entire remaining time |

**Resource Packs:**
| Item | Cost | Contents |
|------|------|----------|
| Food Crate | 30💎 | +1,000 Food |
| Ammo Box | 30💎 | +1,000 Ammo |
| Metal Scrap | 30💎 | +1,000 Metal |
| Supply Bundle | 80💎 | +1,000 each |
| Resource Vault | 200💎 | +5,000 each |

**VIP Subscription ($4.99/month):**
- +50% resource production permanently while subscribed
- Daily free x1 hero gacha pull
- 2× builder queue (build 2 buildings simultaneously)
- VIP badge displayed on profile
- Skip 1 event warning per day (auto-win with resource cost only)
- Access to VIP-exclusive hero skin: "Commander Prestige"

**Other IAP:**
- Extra Builder Slot: 200💎 permanent (build 2 buildings simultaneously)
- Hero Slot Expansion: 100💎 (add 5 hero roster slots, max 3 expansions)
- Cosmetic Base Skins: 150–400💎 each

#### Monetization Hooks (FOMO & Urgency)
1. **Event timers:** "Zombie Herd in 45s!" creates urgency to buy speed-ups or resource packs
2. **10× gacha:** "4,500💎 value for 900💎!" creates perceived value
3. **Limited-time offers:** "FIRST PURCHASE BONUS" label on Starter Pack
4. **Daily free spin:** Login reward → get 1 free Ration pull → reinforces return habit
5. **Battle defeat:** "Your base was destroyed! Rebuild with 50💎?" offer on game over
6. **VIP trial:** Day 5 triggers VIP trial offer (24hr free, then subscribe)
7. **Hero envy:** Rival raid uses Commander (Legendary hero) → "Want this hero? [RECRUIT]" link

---

## 6. TECHNICAL ARCHITECTURE

### 6.1 Canvas Architecture

```
Main Canvas: 480×480 (square, scales to fit available screen space via CSS)

Rendering Layers (drawn in order each frame):
  1. Background (ground texture — pre-cached offscreen canvas, redrawn only on map change)
  2. Outer Wall (base perimeter)
  3. Building sprites (10 buildings with level-appropriate detail)
  4. Ground particles (blood stains, debris — drawn to persistent decal canvas)
  5. Pickups / resource drops
  6. Enemy sprites (all active enemies)
  7. Hero sprites (all deployed heroes)
  8. Projectiles (bullets, acid spits)
  9. Particle effects (blood, explosions, muzzle flash, ability FX)
 10. UI overlays (health bars, cooldown indicators, damage numbers)
 11. Screen flash effects (ability activations, damage taken)

Offscreen canvases (pre-computed, reused):
  - backgroundCache: Ground texture, path lines (redrawn when building changes)
  - decalCache: Persistent blood stains, scorch marks (accumulates during battle)
```

### 6.2 JavaScript Module Structure

```javascript
// All code in single <script> tag, organized into namespaces:

const C = {};           // CONFIG: canvas size, wall coords, timing constants
const BD = {};          // BUILDING_DATA: definitions for all 10 buildings
const HD = [];          // HERO_DATA: definitions for all 6 hero classes
const ED = {};          // ENEMY_DATA: definitions for all 6 enemy types
const RD = [];          // RESEARCH_DATA: all 15 research nodes

let G = {};             // GAME_STATE: master state object (all runtime data)

const Audio = {};       // AudioEngine: Web Audio API, procedural SFX
const Input = {};       // InputManager: touch (tap) + mouse/keyboard
const Save = {};        // SaveSystem: localStorage serialization

const Sprite = {};      // SpriteLib: all procedural drawing functions
const Fx = {};          // Effects: particles, screen shake, floating text

const Res = {};         // ResourceSystem: production ticks, caps, spending
const Build = {};       // BuildingSystem: upgrades, cost calculation, timers
const Hero = {};        // HeroSystem: gacha, leveling, roster management
const Enemy = {};       // EnemySystem: spawn, AI state machine, pathfinding
const Battle = {};      // BattleEngine: real-time combat loop
const Events = {};      // EventSystem: schedule waves, herds, raids, outbreaks
const WorldMap = {};    // WorldMapSystem: tile gen, missions, rivals
const Research = {};    // ResearchSystem: node unlock, effect application

const Render = {};      // CanvasRenderer: base, battle, map, research views
const UI = {};          // UIManager: HTML DOM updates, modals, toasts, HUD
const Shop = {};        // ShopSystem: IAP UI, credit transactions

const Loop = {};        // GameLoop: requestAnimationFrame, update + render
```

### 6.3 Game State Object (G)

```javascript
G = {
  // Meta
  view: 'base',                  // current screen: 'base'|'map'|'heroes'|'research'|'shop'
  inBattle: false,
  paused: false,
  firstPlay: true,
  dayNumber: 1,
  gameTime: 0,                   // total seconds played (accumulates)
  pityCounter: 0,                // gacha pity pulls counter

  // Resources
  res: { food, ammo, metal, credits, rations },
  resCap: { food, ammo, metal },

  // Buildings: keyed by building ID
  buildings: {
    cc:    { level, hp, maxHp, upgrading, upgradeTimer, upgradeTotal },
    bar:   { level, ... },
    hosp:  { level, ... },
    work:  { level, ... },
    farm:  { level, ... },
    scrap: { level, ... },
    arm:   { level, ... },
    watch: { level, ... },
    power: { level, ... },
    wall:  { level, sections: { N, S, E, W } }  // sections = HP values
  },

  // Heroes
  heroes: [                      // all recruited heroes
    { id, name, class, rarity, level, xp, hp, maxHp, status }
    // status: 'available'|'deployed'|'on_mission'|'healing'|'injured'|'kia'
  ],
  roster: ['gunner', null, null, null],  // 4 hero slots for defense

  // Research
  research: { dmg1: true, wall1: true },  // set of unlocked node IDs

  // World Map
  world: {
    tiles: [ [type, revealed, mission, rival_data], ... ], // 9×9 = 81 tiles
    activeMissions: [{ heroId, tileIndex, endsAt, type }]
  },

  // Active Events
  events: [
    { type: 'wave'|'herd'|'rival'|'outbreak'|'chain', warningTimer, active }
  ],
  nextEventIn: 180,              // seconds until next event rolls

  // Battle State (null when not in battle)
  battle: {
    state: 'preparing'|'active'|'victory'|'defeat',
    eventType,
    wave: 1,
    enemies: [{ type, x, y, hp, maxHp, vx, vy, state, attackCd, isDead }],
    heroUnits: [{ id, x, y, hp, maxHp, cooldown, target, attackCd, abilityActive }],
    projectiles: [{ x, y, vx, vy, dmg, type, life }],
    particles: [{ x, y, vx, vy, color, life, maxLife, size }],
    floatingTexts: [{ x, y, text, color, life, vy }],
    shakeAmount: 0,
    totalKills: 0,
    score: 0,
    turrets: []                  // deployed engineer turrets
  },

  // Particles outside battle (base ambient effects)
  ambientFx: [],

  // Notifications queue
  toastQueue: []
};
```

### 6.4 Sprite Drawing — Building Functions

Each building is a function `Sprite.drawBuilding(ctx, buildingId, cx, cy, level, state)`.

All sprites drawn using canvas 2D primitives only (fillRect, arc, lineTo, etc.). No images. Key visual techniques:
- `ctx.shadowBlur / ctx.shadowColor` for glow effects on windows, ability FX, premium UI
- Multiple `fillRect` layers to create depth (shadow at base, dark body, lighter face, bright details)
- Level indicator badge (circle + number) in top-right of each building
- Damage state: buildings below 50% HP show cracks (cross-hatch pattern overlay at 30% alpha)
- Under construction: scaffolding overlay drawn on top (semi-transparent orange grid)

**Sample building detail levels (lines of canvas ops):**
- Command Center: ~60 operations (flagship building, most complex)
- Watchtower: ~45 operations (tall shape, spotlight glow)
- Hospital: ~40 operations (white building, cross symbol, ambulance)
- Farm: ~35 operations (crop rows, fence, small house)
- All others: ~30–40 operations

### 6.5 Hero Drawing in Battle

Heroes are 16×24px sprites drawn procedurally. Each has:
- Distinctive color scheme (see §5.3)
- Simple humanoid silhouette (head, body, legs, weapon)
- Aim direction indicator (gun line pointing at target)
- HP bar (green → yellow → red)
- Ability cooldown ring (circular progress arc around hero)
- Hit flash (white overlay when taking damage)

### 6.6 Enemy Drawing

Enemies drawn as distinctive shapes:
| Enemy | Shape | Key Visual |
|-------|-------|-----------|
| Crawler | Hunched rectangle with forward lean | Greenish, arms dragging |
| Runner | Thin upright rectangle | Yellow-green, speed lines |
| Brute | Wide dark rectangle with bumps | Armored look, angry eyes |
| Spitter | Medium rectangle with bulge | Teal, acid drip at mouth |
| Giant | 2× size rectangle, jagged edges | Dark green, boss crown |
| Rival Soldier | Upright human with weapon | Gray uniform, helmet, gun |

### 6.7 Audio System

All sounds are procedurally generated using the Web Audio API. No external audio files.

| Sound | Method | Params |
|-------|--------|--------|
| Shoot | OscillatorNode + noise burst | Short (50ms), high-freq noise + 300Hz square |
| Hit | OscillatorNode | 150Hz sine, 60ms decay |
| Explosion | Noise burst | 500ms, filtered bandpass, reverb |
| Ability activate | Chord arpeggio | 3 tones ascending, 150ms each |
| Upgrade complete | 3-tone fanfare | 440/660/880 Hz, 150ms each |
| Alert warning | Repeating pulse | 440/330 Hz alternating, urgent |
| Purchase | 3-tone positive | 523/659/784 Hz (C5-E5-G5) |
| Game over | Descending tone | 440→110 Hz sweep, 1s |
| Victory fanfare | Ascending arpeggio | 5 tones, bright timbre |

### 6.8 Input System

**Touch handling:**
- `touchstart`: detect tap vs. drag start by tracking position
- `touchend`: if total movement < 10px → treat as tap
- `tap` event dispatched with canvas-space coordinates
- No joysticks needed (base view is tap-only; battle is tap-only for abilities)

**Mouse handling:**
- `click` on canvas → same as tap
- `hover` shows tooltip (desktop only)

**Tap routing by current view:**
- `base` view: check building bounds → open building modal
- `battle` view: no canvas taps (abilities via HTML buttons)
- `map` view: check tile bounds → open tile modal
- `research` view: check node bounds → open research modal
- `shop` view: handled entirely by HTML button clicks

### 6.9 Save/Load System

```javascript
Save.SLOT = 'deadzone_save_v1';

Save.save = () => {
  const data = {
    v: 1,                           // save format version
    ts: Date.now(),                 // timestamp
    G: JSON.stringify(G)            // full game state
  };
  localStorage.setItem(Save.SLOT, JSON.stringify(data));
};

Save.load = () => {
  const raw = localStorage.getItem(Save.SLOT);
  if (!raw) return false;
  const data = JSON.parse(raw);
  if (data.v !== 1) { Save.clear(); return false; }
  Object.assign(G, JSON.parse(data.G));
  return true;
};
```

**Auto-save triggers:** Every 30 seconds, on view change, on battle end, on any purchase.

---

## 7. PWA SETUP

### 7.1 manifest.json
```json
{
  "name": "Dead Zone: Last Colony",
  "short_name": "Dead Zone",
  "description": "Build, recruit heroes, and survive the zombie apocalypse.",
  "start_url": "/",
  "display": "standalone",
  "orientation": "any",
  "background_color": "#0a0a0a",
  "theme_color": "#1a0a0a",
  "icons": [
    { "src": "icon-192.png", "sizes": "192x192", "type": "image/png", "purpose": "any maskable" },
    { "src": "icon-512.png", "sizes": "512x512", "type": "image/png", "purpose": "any maskable" }
  ]
}
```

### 7.2 Service Worker Strategy
- **Cache on install:** index.html, manifest.json, sw.js
- **Network-first, cache-fallback:** For all requests (keeps game updated while allowing offline play)
- **Background sync:** Not needed for single-player offline game

### 7.3 iOS Safari specific meta tags (in index.html)
```html
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="Dead Zone">
<link rel="apple-touch-icon" href="icon-192.png">
```

### 7.4 Icon Generation
Icons are generated once at first launch using an offscreen canvas, then saved as data URIs in localStorage. If icon files don't exist (local file), the game still works — just won't show custom icon on home screen.

---

## 8. VISUAL DESIGN SYSTEM

### 8.1 Color Palette
| Role | Color | Hex |
|------|-------|-----|
| Background (deep) | Near-black warm | #0a0a08 |
| Background (panel) | Dark warm gray | #1a1008 |
| Panel border | Dark rust | #3a1a0a |
| Accent — primary | Rust orange | #c8421a |
| Accent — gold | Warm gold | #c8921a |
| Accent — premium | Bright gold | #ffd700 |
| Accent — danger | Blood red | #c8221a |
| Accent — success | Army green | #4a8a2a |
| Text — primary | Off white | #e8e8e0 |
| Text — secondary | Warm gray | #888880 |
| Text — label | Dim amber | #887860 |
| Building — base | Dark terracotta | #3a2018 |
| Building — roof | Medium brown | #5a3830 |
| Ground | Dark earth | #1e1a14 |
| Path | Dirt track | #2a2218 |

### 8.2 Typography
- **Game font:** `'Courier New', monospace` (retro terminal feel)
- **Title size:** 24–32px
- **HUD labels:** 10–12px bold
- **Body text:** 11–13px regular
- **Floating damage numbers:** 8px bold, color-coded by damage type

### 8.3 CSS Layout (Flex Column)
```
html, body: 100% viewport, no scroll, overflow hidden
#app: flex-column, 100% height
  #top-bar: flex-shrink:0, ~42px (resource bar) + ~28px (alert bar when shown)
  #canvas-wrap: flex:1, relative, centers canvas
    canvas: absolute, centered, scaled by CSS (image-rendering: pixelated)
    #battle-overlay: absolute, bottom:0, full width
  #nav-bar: flex-shrink:0, ~52px
```

---

## 9. IMPLEMENTATION ROADMAP

### Phase 1 — Foundation (Estimated 600 lines)
1. HTML structure (100 lines)
2. CSS design system (250 lines)
3. Config, data constants (BD, HD, ED, RD), game state init (G)
4. Canvas setup + resize handler
5. PWA meta tags, manifest link, service worker registration

### Phase 2 — Core Systems (Estimated 700 lines)
6. Audio Engine (procedural Web Audio API)
7. Input Manager (tap detection, routing)
8. Save / Load system
9. Resource System (production ticks, cap enforcement, display update)
10. Building System (upgrade cost calc, timer tick, unlock checks)

### Phase 3 — Visuals (Estimated 900 lines)
11. Sprite Library — 10 building draw functions
12. Sprite Library — 6 hero draw functions
13. Sprite Library — 6 enemy draw functions
14. Particle / FX system
15. BASE VIEW renderer (ground, wall, buildings, ambient FX)

### Phase 4 — Hero & Enemy (Estimated 700 lines)
16. Hero System (gacha logic, gacha animation, level up, roster management)
17. Enemy System (spawn logic, AI state machine, pathfinding, attack)
18. BATTLE ENGINE (full combat loop, projectiles, abilities, win/lose)

### Phase 5 — Events & World (Estimated 600 lines)
19. Event System (schedule logic for 4 event types)
20. World Map System (tile generation, mission dispatch, rival bases)
21. Research System (node unlock, effect application to G)
22. BATTLE VIEW renderer, MAP VIEW renderer, RESEARCH VIEW renderer

### Phase 6 — UI & Monetization (Estimated 800 lines)
23. UI Manager (HTML DOM updates: resource bar, alert bar, building modal, hero modal)
24. Building upgrade modal
25. Hero management modal + gacha animation canvas
26. Research modal
27. World map tile modal
28. Shop (IAP storefront, credit transactions)
29. VIP modal
30. Settings modal

### Phase 7 — Polish & PWA (Estimated 300 lines)
31. First-time tutorial overlay
32. Game loop with auto-save
33. Toast notification system
34. Screen shake + flash effects
35. Performance tuning (cached backgrounds, object pooling)
36. PWA icon generation script
37. Install prompt handling

**Total estimated:** ~4,600 lines of well-commented, organized code

---

## 10. PERFORMANCE TARGETS

| Metric | Target |
|--------|--------|
| Initial load | < 500ms (single HTML file, no network deps) |
| Frame rate (idle base) | 60fps steady |
| Frame rate (100-enemy herd) | ≥ 45fps on mid-range Android |
| Memory footprint | < 30MB (no textures, only procedural art) |
| Canvas draw calls per frame (battle) | < 800 (use batching by fill style) |

**Optimization strategies:**
- Background ground texture cached to offscreen canvas (re-render only on building change)
- Object pooling for enemies, projectiles, particles (pre-allocate arrays, reuse inactive slots)
- Limit max simultaneous particles to 250
- Sort enemies by state (dead ones skipped early in loop)
- `requestAnimationFrame` with delta-time cap at 50ms (prevents physics explosion on tab resume)

---

## 11. POTENTIAL FUTURE FEATURES (v2.0)

1. **Multiplayer Alliance** — Real backend (Node.js + WebSocket), alliance vs. alliance events
2. **Campaign Mode** — 20 story missions with narrative, unique boss enemies
3. **Hero Equipment** — Craftable gear items with stats (helmet, weapon, armor)
4. **Clan Wars** — Weekly scheduled PvP tournaments
5. **Live Events** — Server-driven seasonal events (Halloween, etc.)
6. **Leaderboards** — Global kill count, wave survival, base value rankings
7. **Chat** — Alliance chat system
8. **Push Notifications** — Real mobile notifications via Web Push API ("Your scavenge mission is done!")
9. **Cosmetic System** — Base skins, hero skins, wall skins
10. **Prestige System** — Reset base for permanent multipliers after Day 30

---

*End of Plan — Dead Zone: Last Colony v1.0*

