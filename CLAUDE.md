# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the project

Zero-dependency static web project — open any `.html` file directly in a browser:

```powershell
Start-Process "merge-defense.html"   # main game
Start-Process "tictactoe.html"       # secondary game
```

No build step, no server, no package manager. For automated testing use Playwright (installed locally via `npm install playwright`), then run scripts with `node <script>.cjs` from the project root.

## Repository layout

| File | Purpose |
|------|---------|
| `merge-defense.html` | Main game — all HTML, CSS, and JS inline in one file (~2 300 lines) |
| `tictactoe.html` | Simple tic-tac-toe — self-contained single file |
| `CLAUDE.md` | This file |

---

## merge-defense.html — architecture

The entire game is one file structured as: `<style>` block → `<body>` HTML → `<script>` block. No modules, no bundler.

### Screen state machine

`screen` (`'menu' | 'game'`) and `activeMenuTab` (`'play' | 'character' | 'talents'`) are the top-level UI state. `showScreen(s)` toggles visibility of `#menu-overlay`, `#hud`, `#game-area`, `#controls`, etc. `switchMenuTab(tab)` swaps content inside `#menu-tab-content`.

### Persistent profile (`localStorage`)

All cross-session data lives in one object saved under `'mergeDefenseProfile'`:

```js
profile = {
  playerLevel, xp, talentPoints, gold,
  equippedItems: { helmet, chestplate, shoes, weapon, band, gloves }, // item IDs
  inventory: [],        // array of item IDs owned but not equipped
  purchasedTalents: [], // array of talent IDs
  pendingChests: [],    // chests won but not yet opened
}
```

`loadProfile()` / `saveProfile()` handle serialisation. Level unlock state is stored separately under `'mergeDefenseUnlocked'` (array of unlocked level indices).

### Data tables (constants)

| Constant | What it defines |
|----------|----------------|
| `LEVEL_CONFIG[0..5]` | Per-tier emoji and base damage for player fighters (6 tiers) |
| `LEVEL_DEFS[0..4]`   | 5 game levels — wave count, enemy HP/damage scaling, starting fighters |
| `ITEM_POOL`          | 13 equipment items; each has `id, name, slot, tier, hpBonus, dmgBonus, rarity` |
| `TALENT_DEFS`        | 6 purchasable talents with `id, name, desc, cost, effect` (flavor) |
| `CHEST_OPTIONS`      | 7 in-game chest rewards: 4 upgrades + 3 abilities (fire/lightning/plants) |
| `ENEMY_EMOJIS`       | Three enemy tiers keyed by wave range |

### Game state (reset on `init(levelIndex)`)

```
playerHP, playerMaxHP, wave, score, gameOver, turnPhase ('player'|'resolving')
enemies[]       — { id, row, col, hp, maxHp, damage, wave, burning?, frozen?, justArrived? }
playerChars[]   — { id, row, col, level, ability? }
upgrades        — { statBoost, icePending }
equipmentDmgBonus  — flat damage bonus summed from equipped attack slots
currentLevelDef    — reference to the active LEVEL_DEFS entry
```

`init()` reads `profile` from localStorage, applies equipment HP bonuses to `playerMaxHP`, applies talent bonuses (Iron Will, Rapid Merge, etc.), and spawns wave 1.

### Grid

7 columns × (7 enemy rows + 2 player rows). `ATTACK_ROW = 6` (last enemy row). Cells are identified by `data-row` / `data-col` attributes; `getCell(r, c)` is the lookup helper.

Enemy advance direction: row 0 (top) → row 6 (front). Enemies that reach `ATTACK_ROW` set `e.justArrived = true`; they sit visible for one full player turn before attacking on the following turn.

### Turn resolution flow (`resolveTurn` — async)

```
playerAttackStep()       — each column's chars deal damage to frontmost reachable enemy;
                           abilities (fire burn, lightning chain, plants pierce) apply here
→ remove dead enemies, score
→ burn tick              — fire-burned enemies take DoT, then clear
→ wave-clear check       — if empty: grant XP, showOrbRewards(), spawnWave(), renderAll()
→ advanceEnemies()       — enemies move forward one row; frozen enemies skip; justArrived set
→ enemyAttackStep()      — veterans at ATTACK_ROW deal damage and are removed;
                           fresh arrivals (justArrived) are spared this turn
→ player defeat check
→ re-enable End Turn
```

### Orb reward system (`showOrbRewards`)

Called after every wave clear. Generates 2 orbs (3 from wave 5 onward, +1 with Double Rewards talent): always one `'unit'` orb and one `'chest'` orb. Player clicks each one-by-one. Unit orb → `grantUnitOrb()` adds a level-1 fighter. Chest orb → `openChest()` shows 3 random `CHEST_OPTIONS`; chosen option is applied by `applyChestOption(opt)`.

### Damage calculation (per column, in order)

1. Base: sum of `LEVEL_CONFIG[ch.level-1].damage` for all chars in column
2. `+ equipmentDmgBonus` (flat, from equipped weapon/band/gloves)
3. `× 1.3` if any char in column has `ability === 'fire'`
4. `× 1.25` if `upgrades.statBoost`
5. `× 1.1` if talent `berserker` is purchased

### Drag-and-drop merge rules

Chars are draggable only during `turnPhase === 'player'`. Dropping onto an empty cell moves the char. Dropping onto a same-level char merges them (`level + 1`, capped at `MAX_CHAR_LEVEL = 6`). Dropping onto a different-level char shows a red `.drag-invalid` highlight and does nothing. The source char's `ability` field is lost on merge.

### Rendering

Three render functions called independently or together via `renderAll()`:
- `renderEnemyZone()` — rebuilds all enemy cells from `enemies[]`; adds 🔥/❄️ status badges
- `renderPlayerZone()` — rebuilds player cells from `playerChars[]`; adds ability badge, re-attaches drag handlers
- `updateHUD()` — syncs HP bar, wave counter, kill score, upgrade display

---

## Git workflow

**Commit and push after every meaningful unit of work.**

```powershell
git add <changed-files>
git commit -m "feat: short description"
git push
```

| Prefix | When |
|--------|------|
| `feat:` | new feature |
| `fix:` | bug fix |
| `style:` | CSS/visual only |
| `refactor:` | restructure, no behavior change |
| `docs:` | documentation only |

Remote: https://github.com/mbarba05/ClaudeCodeTest
