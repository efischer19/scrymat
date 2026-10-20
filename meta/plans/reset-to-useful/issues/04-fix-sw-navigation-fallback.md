# fix: service worker should serve the app shell for offline navigations

## What do you want to build?

The README promises "works offline after first load", but deep links don't
work offline. `packages/pwa/src/sw.ts` answers every request with
`caches.match(request) ?? fetch(request)`. A navigation to `/match/ABC123` or
`/deck` isn't in the precache, so offline it falls through to `fetch` and
fails.

For navigation requests, the service worker should serve the cached
`/index.html` so the client-side router (ADR-009) can handle the path.

## Acceptance Criteria

- [ ] Requests with `request.mode === 'navigate'` are answered network-first, falling back to the cached `/index.html` when the network fails
- [ ] Non-navigation requests keep the current cache-first behaviour
- [ ] Scryfall requests are still not intercepted
- [ ] A Playwright test covers it: load the app, go offline (`context.setOffline(true)`), navigate to `/match/TEST01`, and confirm the app shell renders
- [ ] Unit or E2E coverage confirms an online navigation still gets the fresh `index.html`

## Implementation Notes (Optional)

- Network-first for navigations also helps updates: users get the new `index.html` as soon as a deploy lands, instead of waiting for the SW update cycle.
- `/index.html` is already in `__PRECACHE_ENTRIES__` (see `serviceWorkerPlugin` in `packages/pwa/vite.config.ts`).
- Ticket 05 also edits `sw.ts`. Land this one first.
