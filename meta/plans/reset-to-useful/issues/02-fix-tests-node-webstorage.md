# fix: PWA unit tests fail on Node 25+ because of built-in localStorage

## What do you want to build?

Make the `@scrymat/pwa` Vitest suite pass on current Node releases. On Node
26.4, 86 tests across 7 files fail, for example `useDarkMode.test.ts`,
`deck-storage.test.ts`, `DeckInput.test.tsx` and `App.test.tsx`, with:

```text
TypeError: Cannot read properties of undefined (reading 'clear')
TypeError: Cannot read properties of undefined (reading 'getItem')
```

Node ≥ 25 ships a built-in Web Storage global that shadows the `localStorage`
jsdom provides. CI is unaffected only because it pins Node 20.

## Acceptance Criteria

- [ ] `npm test` passes on Node 26 with no test changes that weaken assertions
- [ ] `npm test` still passes on Node 20 and 22
- [ ] The fix lives in one place, either `packages/pwa/src/test-setup.ts` or the `test` block of `packages/pwa/vite.config.ts`, and has a comment explaining why

## Implementation Notes (Optional)

- Candidate fixes, to be verified on Node 26:
  1. Run the Vitest workers with `--no-experimental-webstorage`, through
     `test.poolOptions` / `execArgv` or `NODE_OPTIONS` in the `test` npm
     script.
  2. In `test-setup.ts`, if `globalThis.localStorage` isn't a working
     `Storage`, install jsdom's (`window.localStorage`) or a minimal in-memory
     `Storage` stub on `globalThis`.
- Prefer option 1 if it works, since it keeps jsdom's real implementation.
- The Vitest config is in `packages/pwa/vite.config.ts` (`test.setupFiles: ['src/test-setup.ts']`).
