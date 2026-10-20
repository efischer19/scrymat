# Plan: Reset Scrymat to "something useful" on the public site

**Status:** Not started · **Written:** 2026-10-10 · **Owner:** Eric Fischer

The repo was renamed `scryglass` → `scrymat` in August 2026, and the
infrastructure cutover stalled partway through. The code is healthy: CI is green
and core plus PWA have extensive tests. The public site, however, is frozen on an
old build, and remote play has never run in production. This plan gets the
working part (single-device play) live under the new name, then makes remote
play real.

Each phase document is self-contained and written for a fresh Claude Code
session. Work them in order. Phase 1 is a prerequisite for everything else, and
Housekeeping can be picked up any time.

| # | Document | Outcome | Needs | Status |
| :- | :------- | :------ | :---- | :----- |
| 1 | [Phase 1: Unblock deploys](./phase-1-unblock-deploys.md) | Current Scrymat build live on `scryglass.cards`; deep links work | AWS console/CLI | Not started |
| 2 | [Phase 2: Move to `scrymat.cards`](./phase-2-scrymat-domain.md) | App served on `scrymat.cards`; old domain redirects cleanly | AWS (ACM, CloudFront, Route 53); Phase 1 | Not started |
| 3 | [Phase 3: Remote play](./phase-3-remote-play.md) | Two people on different networks can play a match | Cloudflare account; code changes; Phase 1 (Phase 2 recommended) | Not started |
| H | [Housekeeping](./housekeeping.md) | Repo and docs consistent with the rename | Nothing external | Not started |

## Code tickets

The repo-only pieces of every phase are written up as issue-ready tickets in
[`issues/`](./issues/00-executive-summary.md), for the
`create-planned-issues.yml` workflow. Run it with
`folder_path=meta/plans/reset-to-useful/issues`, not this folder. The phase docs
cover the infrastructure steps around them and reference tickets by number.

## State as of 2026-10-10 (verified, not assumed)

- **Deploys:** `Deploy to AWS` has failed on every push since 2026-08-03 with
  `Not authorized to perform sts:AssumeRoleWithWebIdentity` for
  `arn:aws:iam::954257023024:role/scryglass-deploy`.
- **`scryglass.cards`:** serves the 2026-08-03 build (`<title>Scryglass</title>`).
  `/`, the root path, returns 200. Every other path, including `/deck`,
  `/match/ABCD` and `/api/room`, returns 403.
- **`scrymat.cards`:** registered and delegated to Route 53
  (`ns-724.awsdns-26.net`, …), but the zone has no A/AAAA records.
- **Signaling worker:** `Deploy Signaling Worker` fails (no `CLOUDFLARE_API_TOKEN`
  secret), and `packages/signaling/wrangler.toml` still has placeholder KV ids.
- **Remote play code gaps:** the app calls a same-origin `/api/room` with no
  config hook. `RTCPeerConnection` is created with no ICE servers, so it works
  on a LAN only.
- **README:** honest status section merged in #212; corrections in #214.

## Ground rules for sessions working this plan

- **Confirm before any AWS or Cloudflare change.** Show the exact command or
  console change and wait for a yes. These are shared, live resources.
- **Read before writing.** Dump the current config (trust policy, distribution
  config, DNS records) to the scratchpad before modifying it, so it can be
  rolled back.
- **Update this index** when a phase starts or finishes: set the Status
  column, add the date, and note anything surprising in the phase doc under a
  `## Log` heading.
- **Update the README "Project status" table** when a phase changes what a
  visitor can actually do.
- **Local test note:** on Node ≥ 25, about 86 PWA tests fail locally because
  Node's built-in `localStorage` shadows jsdom's. Use Node 20/22 (CI uses 20)
  or see the Housekeeping doc.
