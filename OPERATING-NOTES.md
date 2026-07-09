# QB Tool — Operating Notes for Linus

Not a technical reference (that's `CLAUDE.md`) — this is the "what can I safely
click" briefing. Read this before opening the tool, especially before touching
any Assessment Prep pack. Ask Claude to re-read this file and walk you through
anything before your next session with the tool.

## What the tool can do today

- Generate questions via the AI panel (✦ Generate) for any pack/level — paste
  your Anthropic API key, pick pack + level, hit generate, review, add.
- Bootstrap (⚡ in header) — generates for every empty level across all
  non-algorithmic packs in one go, using each pack's own defaults.
- Manual add/edit/delete individual questions.
- Create new custom packs via the Manage panel — but not special types
  (quotes/verbal/binary/bluff) and not with tier/market tags (see below).
- Backup ↓ / Restore ↑ — full local state, use this before anything risky.
- Dedupe (🧹) — cleans existing duplicate questions in a pack.
- Export a single pack's JSON, or the full manifest.

## What it can't do — the traps

**1. Hardcoded to exactly 4 answers.** The AI generation prompt for every
non-special pack says "4 multiple choice answers" and shows the AI a 4-item
JSON example, no matter what a pack's own instructions say. You cannot
generate a genuine 5-option pack (a faithful ORD, or NOG's 5 fixed answers)
through the live tool as it stands today.

**2. The 5 Högskoleprovet packs were NOT built through this tool.**
`hp-ord`, `hp-xyz`, `hp-mek`, `hp-kva`, `hp-nog` exist as pack *definitions*
in the tool (you'll see them in the sidebar, Assessment Prep category), but
their actual 500 questions live only in `toll/Toll/Resources/Packs/hp-*.json`
in the app repo — never generated into this tool's browser storage.

**⚠️ If you open the QB tool and click into any of these 5 packs, they will
show up EMPTY (0 questions).** That's expected, not data loss. But it means:
- **Do not hit Generate/Bootstrap on these 5 packs** without thinking first —
  you'd be adding content into a pack the tool thinks is empty, with no
  awareness of the 100 real questions already shipped in each. You'd risk
  duplicate/inconsistent content once someone tries to reconcile the two.
- **Do not export these 5 packs from the tool and drop the file into Xcode** —
  the exported JSON would be near-empty (whatever you'd generated in-browser,
  if anything) and would silently overwrite the real 500-question files.
- If you want more Högskoleprovet content, ask Claude to keep generating it
  the same way as last time (direct script generation with correctness
  verification), not through the live AI panel, until trap #1 is fixed.

**3. Doesn't emit `tier` or `market` fields at all.** `buildPackJSON()` has no
concept of Content Profile tags. Every pack that needs `tier: "plusplus"`
(all Assessment Prep) or `market: "se"` (Sweden-specific packs) has had that
field **hand-added to the exported JSON after download** — it's not something
the tool does for you, and it never has been. If you re-export any
tier/market-tagged pack from this tool, the exported file will silently be
missing those fields unless you add them back by hand before dropping it into
Xcode.

**4. Re-exporting can revert a pack's category.** Football Facts and History
(European) exist in the app as custom packs in this tool's storage, but were
manually recategorized after export. Re-exporting them from QB as-is will
revert that. (Documented in `CLAUDE.md`, repeating here since it's the same
class of footgun as #2/#3 — the tool and the shipped app JSON can silently
diverge, and the tool always wins if you export over the real file.)

## Before you use the tool next time

Ask Claude: **"Remind me what's safe before I open the QB tool"** — it should
re-read this file and tell you, in particular, which packs (if any) have
real content living outside the tool's storage that you could accidentally
clobber.

## Known gaps worth fixing eventually (not urgent)

- Make the AI generation prompt answer-count-aware (fixes trap #1) — would
  let you build a faithful 5-option ORD and generate more NOG through the
  live tool instead of by hand.
- Add real `tier`/`market` fields + a Manage-panel UI (fixes trap #3) —
  removes the hand-patching step for every future gated/market pack.
