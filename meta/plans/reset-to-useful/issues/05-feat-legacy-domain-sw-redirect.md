# feat: redirect installs on the legacy scryglass.cards domain to scrymat.cards

## What do you want to build?

The site is moving from `scryglass.cards` to `scrymat.cards`. CloudFront will
301-redirect the old domain, **except `/sw.js`**. Browsers that already have
the old service worker installed answer navigations from cache and would
never see that redirect.

Change the service worker and the page bootstrap so a browser on the legacy
host cleans itself up and moves to the new domain:

- **Service worker:** when `self.location.hostname` is `scryglass.cards` or
  `www.scryglass.cards`:
  - On `install`, call `self.skipWaiting()`. The normal flow waits for a user
    `confirm()` (see `register-sw.ts`), and that must not block this update.
  - On `activate`, delete all caches and call `self.registration.unregister()`.
    Then, for each window client from `self.clients.matchAll({ type: 'window' })`,
    call `client.navigate('https://scrymat.cards' + pathname + search)`.
  - Don't intercept `fetch` on the legacy host.
- **Page bootstrap:** in `main.tsx` or `register-sw.ts`, if
  `location.hostname` is a legacy host, `location.replace(...)` to the same
  path on `scrymat.cards` before rendering. This covers visitors without a
  service worker.

**⚠️ Do not merge until `https://scrymat.cards` serves the app** (Phase 2, step 5).
Merging earlier redirects every visitor to a domain that doesn't resolve.

## Acceptance Criteria

- [ ] On `scrymat.cards` and `localhost`, behaviour is unchanged (unit-tested)
- [ ] On a legacy host, the SW skips waiting, clears all caches, unregisters itself and navigates open clients to the same path and query on `scrymat.cards` (unit-tested with mocked `self`, `caches` and `clients`)
- [ ] On a legacy host, the page bootstrap redirects before rendering (unit-tested)
- [ ] The legacy-host list and target origin are defined once, as shared constants
- [ ] The PR description repeats the "don't merge until scrymat.cards is live" warning

## Implementation Notes (Optional)

- Builds on ticket 04 (navigation fallback), which also edits `sw.ts`.
- Saved decks in `localStorage` (`scryglass:decklists`) are per-origin and **will not** carry over. That's an accepted trade-off. Do not try to migrate them in this ticket, and do not rename storage keys.
- Background: `meta/plans/reset-to-useful/phase-2-scrymat-domain.md` ("Two traps to design around").
