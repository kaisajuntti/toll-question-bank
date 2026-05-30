# Toll — Question Bank Tool

## What this is
A standalone single-file HTML tool (`index.html`) for managing all quiz content for the Toll iOS app. Opens in Safari. Uses Claude API for AI question generation. Exports JSON files that drop into Xcode.

## Hosting
- GitHub Pages: kaisajuntti.github.io/toll-question-bank
- Repo: github.com/kaisajuntti/toll-question-bank (public)
- Single file: index.html — all HTML, CSS, JS in one file

## Data storage
- All questions stored in browser localStorage key: `toll_qb`
- API key stored in localStorage key: `toll_api_key`
- No backend, no database

## Pack structure
Categories: School 📚 / Languages 🌐 / Popular 🌍 / IQ 🧠 / Specials ⭐
Each pack has levels. Each level has questions.
Question format: { id, q, answers[], correct (index) }

## JSON export format
One file per pack → dropped into Xcode at /Resources/Packs/
See existing buildPackJSON() function for structure.

## Working methodology
- Edit index.html only
- After every working change:
  git add -A
  git commit -m "type: description"
  git push origin main
- GitHub Pages updates in ~1 minute after push

## Update log
- [ ] #1 Import / backup
- [ ] #2 New category + new pack creation
- [ ] #3 Default questions + time per level
- [ ] #4 Export with category metadata + manifest.json
- [ ] #5 Duplicate detection
- [ ] #6 Generate all levels at once
- [ ] #7 Manual add + edit question
