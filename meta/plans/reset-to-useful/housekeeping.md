# Housekeeping

**Status:** Not started · **Depends on:** nothing, so pick items off any time ·
[Back to index](./README.md)

Small, independent cleanups left over from the rename. Each item can be its own
PR. Tick items off here as they land.

## Items

### H1. Fix local tests on Node ≥ 25

- **Problem:** on Node 26.4 (the user's machine), 86 PWA tests fail with
  `TypeError: Cannot read properties of undefined (reading 'clear')`.
  Node's built-in experimental `localStorage` global shadows jsdom's. CI
  passes on Node 20.
- **Fix options:** pin the Node version with an `.nvmrc` and `engines` in the
  root `package.json`. Or, better, make tests independent of it by passing
  `--no-experimental-webstorage` via `NODE_OPTIONS` in the PWA vitest config,
  or by stubbing `localStorage` in the vitest setup file. Confirm which one
  actually works on Node 26.
- **Done when:** `npm test` is green on Node 26 locally and CI stays green.

### H2. Move CI and deploy workflows off Node 20

- Node 20 reached end of life in April 2026. `ci.yml` (three jobs),
  `deploy-aws.yml` and `deploy-signaling.yml` all pin `20`.
- Bump to 22 (or 24), ideally from a single `.nvmrc` via
  `node-version-file`. Check that Playwright and Lighthouse jobs still pass.
- Coordinate with H1 so local and CI use the same version.

### H3. Docs site: fix or retire it

- `mkdocs.yml` still has `site_url: "https://efischer19.github.io/scryglass/"`.
- GitHub Pages is enabled (build type: workflow), but **no workflow publishes
  it anymore**. `https://efischer19.github.io/scrymat/` serves a stale page
  titled "Static JS App".
- `docs-src/index.md` says "The repository, package names, and deployed URLs
  still use the legacy `scryglass` identifier". The repo and packages are
  renamed now, so only the infrastructure still uses it.
- **Ask the user:** keep a docs site, adding a Pages deploy workflow that runs
  `scripts/build-docs.sh`, or disable Pages
  (`gh api -X DELETE repos/efischer19/scrymat/pages`) and treat `docs-src/` as
  in-repo docs only? Either way, fix `site_url` and the stale note.

### H4. SW: serve the app shell for navigations

- `packages/pwa/src/sw.ts` does `caches.match(request) ?? fetch(request)` for
  everything. An offline deep link (`/match/X`, `/deck`) misses the cache and
  fails, which undercuts the offline-first claim.
- **Fix:** for `request.mode === 'navigate'`, respond with the cached
  `/index.html` (network-first or stale-while-revalidate). Add a test. If
  Phase 2 is in flight, coordinate with its kill-switch change in the same
  file.

### H5. Prune stale branches

- About 100 merged `copilot/*` branches and a few `efischer19-patch-*` ones
  remain on `origin`. Local branches `meow` and `pivot` exist too.
- List merged ones with `git branch -r --merged origin/main`, **show the list to
  the user**, and delete only with approval:
  `git push origin --delete <branch>`. Consider enabling "Automatically delete
  head branches" in repo settings.

### H6. Leftover legacy identifiers in code (decide, don't blindly rename)

- User-visible: the deck format is labelled "scryglass" (`DeckInput.tsx`,
  `ExportDropdown.tsx`, file name `deck-scryglass.txt`). Rename the *labels* to
  "Scrymat" but **keep accepting the old format name and files**. The format is
  also the ADR-006 import format, and users may have exported files.
- **Do not rename** storage keys (`scryglass:decklists`, `scryglass-theme`,
  `scryglass-image-cache`, `scryglass-shell-*`). That would silently drop
  users' saved data. Add a code comment saying they're intentionally legacy.
- Internal names (`parseCommandersFromScryglassText`, the
  `scryglass-service-worker` vite plugin) are optional and cosmetic.

### H7. Local repo tidy-up

- The local checkout lives at `~/code/scryglass`, and its `origin` URL is still
  `git@github.com:efischer19/scryglass.git`, which works through GitHub's
  redirect. Update it with
  `git remote set-url origin git@github.com:efischer19/scrymat.git`.
  Renaming the directory is optional, and Claude Code project memory is keyed
  on the path.
- `handoff_readme_status.md` in the repo root is untracked and its work is
  done (#212, #214). Delete it or move it out of the repo.

## Log

*(Sessions: append dated notes here.)*
