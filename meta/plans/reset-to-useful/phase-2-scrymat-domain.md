# Phase 2: Move to `scrymat.cards`

**Status:** Not started · **Estimated effort:** 1–3 hours, mostly waiting on cert
validation and CloudFront propagation · **Depends on:** Phase 1 · [Back to index](./README.md)

## Goal

`https://scrymat.cards` serves the app. `https://scryglass.cards/<anything>`
301-redirects to the same path on `scrymat.cards`. Returning visitors on the old
domain aren't stranded on a stale cached build.

## Background

- `scrymat.cards` is registered and its NS records point at Route 53
  (`ns-724.awsdns-26.net`, `ns-1675.awsdns-17.co.uk`, `ns-373.awsdns-46.com`,
  `ns-1133.awsdns-13.org`). No A/AAAA records exist yet.
- `scryglass.cards` is also on Route 53 and aliases to CloudFront distribution
  `EL8VCFL4HWB2J` (S3 origin bucket `scryglass-prod`).
- The repo variable `CLOUDFRONT_DOMAIN` is the raw
  `d3o3r0h589roll.cloudfront.net` host, so the smoke test isn't tied to either
  custom domain.

### Two traps to design around

1. **Browser storage is per-origin.** Saved decks live in `localStorage`
   (`scryglass:decklists`), and the image cache is IndexedDB
   (`scryglass-image-cache`). Neither carries over to a new domain. With few
   users this is acceptable, but the user should **export their own decks from
   `scryglass.cards` before the redirect goes live**. Keep the storage key
   names as they are; renaming them would wipe data on the new origin too, for
   no gain.
2. **The service worker is cache-first and can pin the old domain.**
   `packages/pwa/src/sw.ts` answers every request from cache first, including
   navigations. If CloudFront redirects *everything* on `scryglass.cards`,
   including `/sw.js`, browsers treat a redirected SW script as an update
   failure. Existing installs would keep serving the cached old app forever and
   never see the redirect. Fix: keep serving a **kill-switch `sw.js`** on the
   old host that unregisters itself and sends open tabs to the new domain.

## Prerequisites

- Phase 1 done: deploys work, and the distribution config is snapshotted.
- AWS CLI session (`! aws login` if needed). ACM certs for CloudFront **must
  be in `us-east-1`**.

## Steps

1. **Confirm the hosted zone matches the delegation.** Run
   `aws route53 list-hosted-zones-by-name --dns-name scrymat.cards` and check
   that the zone's NS set equals the registrar's (above). A stale or duplicate
   zone is a common reason DNS "doesn't work".
2. **Request an ACM certificate** in `us-east-1`, DNS-validated. A
   distribution has exactly one viewer certificate, so it must cover **every
   alias**: `scrymat.cards`, `www.scrymat.cards`, `scryglass.cards`, and
   `www.scryglass.cards` if the current aliases include it (check
   `cf-before.json`). Create the validation CNAMEs in both zones ("Create
   records in Route 53" does this). Wait for `ISSUED`.
3. **Update the distribution** after confirming. Add `scrymat.cards` and
   `www.scrymat.cards` to Aliases (CNAMEs) and switch to the new multi-name
   cert. Snapshot first, then use `--if-match`.
4. **Create Route 53 alias records** in the `scrymat.cards` zone: A and AAAA for
   the apex and `www` → the distribution (alias hosted zone `Z2FDTNDATAQYW2`).
5. **Verify the new domain** before redirecting anything:
   `curl -sI https://scrymat.cards/` shows 200, and
   `https://scrymat.cards/match/ABC123` shows 200.
6. **Ship the kill-switch SW for the old host** as a code change, PR'd and
   merged.
   - In `sw.ts`, or a tiny separate script selected by hostname: when
     `self.location.hostname` is `scryglass.cards`, then on `activate`, call
     `self.registration.unregister()`, clear caches, and `clients.matchAll()` →
     `client.navigate('https://scrymat.cards' + pathname)`.
   - Alternative: an inline check in `index.html` that redirects when
     `location.hostname === 'scryglass.cards'`. It's simpler but only runs once
     the stale cached page has updated, so it doesn't replace the SW fix.
   - Add a unit test for the hostname branch.
7. **Add a CloudFront Function redirect** (viewer-request) on the distribution
   after confirming. If `Host` is `scryglass.cards` or `www.scryglass.cards`
   **and** the URI is not `/sw.js`, return a 301 to
   `https://scrymat.cards${uri}${querystring}`. Also 301
   `www.scrymat.cards` → apex. Test it in the CloudFront Functions console
   before associating it.
8. **Update the repo:**
   - README: domain links, the `[!NOTE]` about the legacy identifier, and the
     status table row.
   - GitHub repo homepage: `gh repo edit efischer19/scrymat --homepage https://scrymat.cards`.
   - `docs-src/deployment.md`: multi-domain cert and redirect function.
   - `CLOUDFRONT_DOMAIN` can stay as the raw CloudFront host.
9. **Keep `scryglass.cards` registered** for at least a year so old links and
   installed PWAs keep redirecting. Note its renewal date in the Log.

## Acceptance criteria

- [ ] `https://scrymat.cards/` and `https://scrymat.cards/match/ABC123` return 200 with a valid cert.
- [ ] `https://www.scrymat.cards/x` → 301 → `https://scrymat.cards/x`.
- [ ] `https://scryglass.cards/deck?x=1` → 301 → `https://scrymat.cards/deck?x=1`.
- [ ] `https://scryglass.cards/sw.js` → 200 (not redirected).
- [ ] In a browser profile that had the old site installed: one visit to
      `scryglass.cards` ends up on `scrymat.cards`, and DevTools shows no
      service worker left registered for `scryglass.cards`.
- [ ] README and repo homepage point at `scrymat.cards`.

## Rollback

- Detach the CloudFront Function, and the old domain serves the app directly
  again.
- Restore the distribution config snapshot.
- The DNS records on `scrymat.cards` are additive. Delete them to take the new
  domain offline.

## Out of scope

- Migrating users' saved decks across origins. Not worth building for current
  usage; the user exports and re-imports by hand.
- A new bucket or distribution named for Scrymat.

## Log

*(Sessions: append dated notes here.)*
