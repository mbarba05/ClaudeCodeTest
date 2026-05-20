# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the project

This is a zero-dependency static web project. Open `tictactoe.html` directly in any browser:

```powershell
Start-Process "tictactoe.html"
```

No build step, no server, no package manager.

## Architecture

Everything lives in a single file: `tictactoe.html`. It is structured as inline HTML → inline `<style>` → inline `<script>`, with no external dependencies.

**Game state** is held in three module-level variables:
- `board` — flat 9-element array (`null` | `'X'` | `'O'`), indices 0–8 left-to-right, top-to-bottom
- `current` — whose turn it is (`'X'` or `'O'`)
- `over` — boolean gate that blocks clicks after the game ends

**Win detection** (`checkWin`) iterates the `WINS` constant (8 hard-coded winning triplets) and returns the winning triplet or `null`.

**DOM ↔ state sync** is one-way: `handleClick` writes to `board`, then immediately reflects the change in the DOM. There is no rendering loop — DOM and state are updated together in the same function call.

**Scores** (`{ X, O, D }`) are in-memory only; they reset on page reload.

## Git workflow

**Commit and push after every meaningful unit of work.** Do not batch multiple features or fixes into one commit. The goal is that GitHub always reflects the latest working state so nothing is ever lost and any point can be reverted to.

Commit message convention used in this repo:

| Prefix | When to use |
|--------|-------------|
| `feat:` | new feature or visible capability |
| `fix:` | bug fix |
| `style:` | visual/CSS-only change |
| `refactor:` | restructure with no behavior change |
| `docs:` | documentation only |

```powershell
git add <changed-files>
git commit -m "feat: short description of what changed and why"
git push
```

Remote: https://github.com/mbarba05/ClaudeCodeTest

### When to commit

- After adding a new feature or game mechanic
- After fixing a bug
- After any visual/style change
- After updating CLAUDE.md or other docs
- Before starting a risky or experimental change (so there is a clean rollback point)
