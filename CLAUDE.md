# Toll — Question Bank Tool

## What this is
A standalone single-file HTML tool (`index.html`) for managing all quiz content for the Toll iOS app. Opens in Safari. Uses Claude API for AI question generation. Exports JSON files that drop into Xcode.

## Hosting
- GitHub Pages: kaisajuntti.github.io/toll-question-bank
- Repo: github.com/kaisajuntti/toll-question-bank (public)
- Single file: index.html — all HTML, CSS, JS in one file
- Build version shown in header: `QUESTION BANK · YYYY-MM-DD.N` — increment N with each commit to verify cache

## Data storage
- Questions: `toll_qb` (QB object: packId → levelIndex → [questions])
- API key: `toll_api_key`
- Custom packs: `toll_custom_packs` (full pack objects including userDescription, generationInstructions)
- Deleted built-in pack IDs: `toll_deleted_packs`
- Level defaults (questions/time per level): `toll_level_defaults`
- Pack instruction overrides (all packs, incl. built-ins): `toll_pack_instructions`
- No backend, no database

## Pack structure
Categories: School 📚 / Languages 🌐 / Popular 🌍 / IQ 🧠 / Specials ⭐
Each pack has levels. Each level has questions.
Standard question format: { id, q, answers[], correct (index) }
Quotes question format: { id, quote, author } — used when pack.special === "quotes"
Custom pack extra fields: { userDescription, generationInstructions, icon, custom: true }

## Special pack types
- `binary` — 2-answer Yes/No questions (e.g. The Idiot Question)
- `bluff` — 3 statements, spot the lie (e.g. Bluff)
- `quotes` — no multiple choice; format is { id, quote, author } throughout (display, add/edit forms, AI generation, export)

## Key architecture notes
- `PACKS` array is `const` but mutable — custom packs are pushed at runtime
- `loadCustomPacks()` runs first in `load()`: applies deleted IDs, pushes saved custom packs
- `loadPackInstructions()` runs after: applies instruction overrides to any pack (built-in or custom)
- `saveCustomPacks()` saves only custom packs; `savePackInstructions()` saves instruction overrides for all
- `window._pendingQs` holds the last AI-generated batch (avoids JSON-in-onclick bugs)
- `state.editingQid` tracks which question card is in inline-edit mode
- Build version string is manually incremented in the header subtitle on each commit

## AI model
- Model used: `claude-sonnet-4-6` (no date suffix)
- NOTE: `claude-sonnet-4-5` does NOT exist — the family goes 4 → 4.6. Don't use 4-5.
- API called directly from browser using `anthropic-dangerous-direct-browser-access: true`

## Working methodology
- Edit index.html only
- After every working change:
  git add -A
  git commit -m "type: description"
  git push origin main
- GitHub Pages updates in ~1 minute after push
- To force reload in Safari: Cmd+Option+R (hard refresh)

## JSON export format
One file per pack → dropped into Xcode at /Resources/Packs/
See `buildPackJSON()` — includes:
- `categoryMeta: { id, label, icon }`
- `icon`: pack.icon || cat.icon
- Per-level `defaultQuestions` and `defaultTime` from stored levelDefaults
- `algorithmic`, `special` fields
- Questions exported as `{ id, question, answers, correctIndex }` for normal packs, or `{ id, quote, author }` for quotes packs

## Manifest export
`exportManifest()` → manifest.json with version, generated date, and summary entry per pack
(id, name, category, icon, levelCount, questionCount, version: 1)

## Update log — COMPLETED
- [x] #1 Import / backup (Backup ↓ / Restore ↑ buttons in header)
- [x] #2 New category + new pack creation (Manage panel — create/delete packs)
- [x] #3 Default questions + time per level (compact settings bar below level tabs)
- [x] #4 Export with category metadata + manifest.json

## Update log — REMAINING
- [ ] #5 Duplicate detection
- [ ] #6 Generate all levels at once
- [ ] #7 Manual add + edit question (edit is done; manual ADD is not yet built)

## Features built (session summary)
- **Backup/Restore**: header buttons, JSON file with { version, exported, QB }
- **Manage panel**: full-screen overlay, lists all packs with delete, new pack form
  - New pack fields: name, icon, category, userDescription, generationInstructions, auto-filled levels
  - Level presets by category: school (12 levels), popular/iq (5: Easy→Legendary), specials (Pool)
  - Custom packs stored in toll_custom_packs, deletions of built-ins in toll_deleted_packs
- **userDescription**: shown as muted subtitle below pack title in pack header
- **generationInstructions**: auto-populated into AI panel textarea when pack selected
  - "Save instructions" button always visible — saves for ALL packs (not just custom)
- **Level defaults bar**: between level tabs and questions — stepper (1–4 questions) + 15s/30s/45s/60s segmented control; saved immediately to localStorage
- **Export enhancements**: categoryMeta block in each pack JSON; Manifest button in pack header
- **Edit question**: hover → Edit button → inline edit form with textarea + answer inputs + radio for correct; Save/Cancel
- **Add All fixed**: was broken by apostrophes in JSON-in-onclick; now uses window._pendingQs + addPendingQs()
- **AI panel**: label renamed to "AI generation instructions"; save button always visible; resize handle removed
- **Pack instruction overrides**: toll_pack_instructions persists instructions for any pack across reloads
- **Mindful Quotes pack**: new `special: "quotes"` type — { id, quote, author } format throughout; AI prompt generates mindful quotes; display shows italic quote + author; add/edit forms have quote textarea + author input; export uses quotes shape
