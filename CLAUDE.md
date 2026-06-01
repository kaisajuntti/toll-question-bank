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
- N/A levels (skipped in Generate All): `toll_na_levels` (packId → { levelIdx: true })
- No backend, no database

## Categories
- School 📚 — Grade 1–12 levels
- Languages 🌐 — CEFR levels (A1 Beginner → C2 Mastery); packs named "Target / Native" (e.g. "French / English")
- Popular 🌍 — Easy/Medium/Hard/Expert/Legendary
- Assessment Prep 🎯 — Easy/Medium/Hard/Expert (was "IQ"); hiring test style
- Specials ⭐ — hardcoded only

## Pack structure
Each pack has levels. Each level has questions.
Standard question format: { id, q, answers[], correct (index) }
Quotes question format: { id, quote, author } — used when pack.special === "quotes"
Verbal question format: { id, passage, statement, correct } — used when pack.special === "verbal"
Custom pack extra fields: { userDescription, generationInstructions, icon, custom: true }

Language packs extra fields: { targetLanguage, nativeLanguage } — injected into AI prompt automatically

## Pack definition fields
All packs can have optional fields that override global defaults:
- `defaultTime` — default seconds per question (overrides global 30s fallback)
- `defaultQuestions` — default questions per level in the app (overrides global 2)
- `defaultGenCount` — default AI generation count; AI panel and Bootstrap use this when set
- `generationInstructions` — pre-filled in AI panel; user edits saved to toll_pack_instructions

## Special pack types
Packs with a `special` field are hardcoded — they cannot be deleted from the Manage panel and cannot be created via the New Pack form. Adding or removing a special pack requires a code change in `index.html` (PACKS array + QB seed + all `isSpecial`/`isVerbal`/`isQuotes`/etc. branches).

Current specials:
- `binary` — 2-answer Yes/No questions (e.g. The Idiot Question)
- `bluff` — 3 statements, spot the lie (e.g. Bluff)
- `quotes` — no multiple choice; format is { id, quote, author } throughout
- `verbal` — True/False/Cannot Say; format is { id, passage, statement, correct } throughout; modelled on SHL-style verbal reasoning hiring tests

The Manage panel shows special packs with a SPECIAL badge and 🔒 icon. The delete button is hidden and `deletePack()` has a hard guard that rejects attempts to delete them.

## Hardcoded packs (non-deletable)
Only the following packs are hardcoded in PACKS array:
- **Times Tables** (school, algo) — algorithmically generated
- **Maths: Addition & Subtraction** (school, Grade 1–6)
- **Maths: Division & Multiplication** (school, Grade 4–9)
- **Maths: Percentages & Ratio** (school, Grade 6–9)
- **Maths: Geometry** (school, Grade 4–12)
- **Maths: Algebra** (school, Grade 7–12)
- **Maths: Analysis** (school, Grade 10–12)
- **Science** (school, Grade 4–9)
- **Biology / Chemistry / Physics** (school, Grade 7–12)
- **History / Geography** (school, Grade 4–12, European focus)
- **Economics** (school, Grade 10–12)
- **French / English, Spanish / English, German / English** (languages, CEFR A1–C2)
- **Numerical Reasoning, Abstract Reasoning, Verbal Reasoning** (assessment prep, broad)
- **Number Sequences, Odd One Out, Analogies, Logic Puzzles** (assessment prep, focused)
- **The Idiot Question, Bluff, Mindful Quotes** (specials)

All other packs are created via the Manage panel as custom packs.

## Key architecture notes
- `PACKS` array is `const` but mutable — custom packs are pushed at runtime
- `loadCustomPacks()` runs first in `load()`: applies deleted IDs, pushes saved custom packs
- `loadPackInstructions()` runs after: applies instruction overrides to any pack (built-in or custom)
- QB restore pads level slots to match current pack.levels.length (fixes stale localStorage data)
- `saveCustomPacks()` saves only custom packs; `savePackInstructions()` saves instruction overrides for all
- `window._pendingQs` holds the last AI-generated batch (avoids JSON-in-onclick bugs)
- `state.editingQid` tracks which question card is in inline-edit mode
- `state.generatingAll` blocks concurrent Generate All runs
- Build version string is manually incremented in the header subtitle on each commit
- New pack IDs are derived from name: lowercase, alphanumeric+hyphens. Avoid IDs that match old deleted packs (stored in toll_deleted_packs) — use distinct IDs for new hardcoded packs

## AI model
- Model used: `claude-sonnet-4-6` (no date suffix)
- NOTE: `claude-sonnet-4-5` does NOT exist — the family goes 4 → 4.6. Don't use 4-5.
- API called directly from browser using `anthropic-dangerous-direct-browser-access: true`

## AI generation
- Default genCount: 50 (displayed in AI panel stepper, max 100)
- Assessment Prep packs override to defaultGenCount: 25 — AI panel sets this automatically on pack select
- Bootstrap (⚡ button in header) generates for all empty levels across all non-algo packs; uses pack.defaultGenCount per pack
- Character limits enforced in add/edit forms: question ≤ 90 chars, answer ≤ 28 chars
- Character limits also stated in AI prompts for all non-special packs
- School packs: prompt includes grade→age reference and "quick to evaluate, under 10 seconds" instruction
- Assessment Prep packs: prompts reference real hiring test providers (SHL, Korn Ferry, Cubiks, Saville, Watson-Glaser) and lead with "QUALITY MATTERS"
- Language packs: targetLanguage/nativeLanguage injected into prompt automatically

## N/A levels
- Toggle in the level settings bar ("Skip in gen all" button)
- Marked levels show dimmed with "N/A" badge in level tabs
- Generate All Levels and Bootstrap both skip N/A levels
- Stored in toll_na_levels localStorage

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
- Questions exported as:
  - Normal: `{ id, question, answers, correctIndex }`
  - Quotes: `{ id, quote, author }`
  - Verbal: `{ id, passage, statement, correct }`

## Manifest export
`exportManifest()` → manifest.json with version, generated date, and summary entry per pack
(id, name, category, icon, levelCount, questionCount, version: 1)

## Update log — COMPLETED
- [x] #1 Import / backup (Backup ↓ / Restore ↑ buttons in header)
- [x] #2 New category + new pack creation (Manage panel — create/delete packs)
- [x] #3 Default questions + time per level (compact settings bar below level tabs)
- [x] #4 Export with category metadata + manifest.json
- [x] #6 Generate all levels at once
- [x] #7 Manual add + edit question

## Update log — REMAINING
- [ ] #5 Duplicate detection

## New pack form
- Category drives level structure (LEVEL_PRESETS)
- Optional "Start from template" dropdown — filters by category, pre-fills name
- Languages category shows extra fields: target language (required) + native language
- Icon removed — inherits from category automatically
- No special pack types can be created via the form
