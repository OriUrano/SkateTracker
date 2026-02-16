# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ice Skills Tracker — a single-file PWA for tracking ice skating skill progression. No build system, no framework, no backend. Everything lives in `index.html` (HTML + CSS + JS inline).

## Development

Open `index.html` directly in a browser. No build step, no install, no dev server required.

The service worker (`sw.js`) caches assets with version key `ice-skills-v14` — **bump this version string** whenever `index.html` changes to bust the cache.

## Architecture

**Single-file SPA** with two tab views:
- **Skills tab**: Grid of 59 skills across 11 phases (Adult 1–6, Pre-Bronze through Master), with prerequisite-based unlocking and 5-level progress tracking (Not Started → Learning → Decent → Great → Fluent)
- **Log tab**: Session history, streak heatmap calendar, stat cards (swipeable), blade sharpening tracker

**Theming**: 5 themes (Frost, Midnight, Rink Classic, Aurora, Competition) implemented via `[data-theme]` CSS variable sets. Each theme defines 20+ variables. All UI must use these variables — never hardcode colors.

**Aurora theme caveat**: `--surface` is semi-transparent (`rgba(255,255,255,.05)`). Use `--panel-bg` (opaque) for backgrounds that need to be opaque (e.g., sticky elements), not `--surface`.

**localStorage keys**:
- `ice-skills-progress` — skill level map (`{skillId: level}`, L/R variants as `skillId-L`)
- `ice-skills-theme` — theme name string
- `ice-sessions` — array of `{date, type, duration, startTime}`
- `ice-rankups` — array of `{date, skillId, side, from, to}`
- `ice-sharpenings` — array of `{date, iceTimeAtSharpening}`

**Skill data model** (the `SKILLS` array): each skill has `id`, `n` (name), `p` (phase), `d` (description), `pr` (prerequisite IDs), `lr` (0=single, 1=needs L/R tracking), `ic` (sprite icon key).

**Sprite icons**: 8×8 grid in `new_sprite.png`, positioned via CSS mask. Map in `SPRITE_POS`.

## Key Patterns

- Bottom-sheet modals: slide-up panels with `.open` class toggle + shared `.backdrop`
- State is global JS variables, persisted via `save()`/`saveLog()` to localStorage
- `setLevel()` auto-records rank-up events for the history timeline
- Detail cards are absolutely positioned within phase sections (not the grid) to avoid `overflow:hidden` clipping
- Stat cards use horizontal scroll-snap; first two visible, third (sharpening) revealed by swiping left
