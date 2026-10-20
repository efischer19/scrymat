# chore: rename user-facing "scryglass" deck format labels to Scrymat

## What do you want to build?

The UI still shows the old name in deck import and export:

- `DeckInput.tsx`: format option label `scryglass (semicolon-delimited)`
- `ExportDropdown.tsx`: option `scryglass format` and download filename `deck-scryglass.txt`

Rename the **user-visible labels** to Scrymat. Keep the old format readable,
and keep stored data where it is.

## Acceptance Criteria

- [ ] Import label reads `Scrymat (semicolon-delimited)`, and the export option reads `Scrymat format`
- [ ] Export filename is `deck-scrymat.txt`
- [ ] Importing a previously exported `deck-scryglass.txt` still works (unit test with fixture content)
- [ ] Storage keys are **unchanged** (`scryglass:decklists`, `scryglass:seeded`, `scryglass-theme`, `scryglass-image-cache`, `scryglass-shell-*`), and each definition has a one-line comment saying the legacy name is intentional so existing users' data survives
- [ ] Tests that assert on the old labels are updated (`DeckInput.test.tsx`, `ExportDropdown.test.tsx`), and E2E selectors are checked as well
- [ ] README and ADR-006 wording are left alone, unless the README refers to the format by its UI label

## Implementation Notes (Optional)

- Renaming internal identifiers (the `'scryglass'` union member in `ImportFormat` and `ExportFormat`, `parseCommandersFromScryglassText`, `exportScryglass`) is optional. If you rename them, do it in a separate commit to keep the label diff reviewable.
- Changing a storage key would silently wipe every user's saved decks and theme. That's the one thing this ticket must not do.
