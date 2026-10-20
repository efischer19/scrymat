# Epic: Reset Scrymat to "something useful" (code tickets)

## Overview

These are the repo-only pieces of the [reset-to-useful plan](../README.md):
changes a coding agent can make in a PR, with no AWS or Cloudflare access.
The infrastructure steps aren't issues and stay in the phase docs, because
they need console access and a human confirming each change: the IAM trust
policy, CloudFront error pages, ACM and DNS, Cloudflare KV and secrets,
real-device testing, and branch pruning.

Create issues from this folder only, **not** the parent folder, because the
phase docs there aren't in ticket format and would fail the parser:

```bash
gh workflow run create-planned-issues.yml -R efischer19/scrymat \
  -f branch=<branch> -f folder_path=meta/plans/reset-to-useful/issues -f human_reviewed=yes
```

## Ticket sequence

| # | Ticket | Phase | Blocked by |
| :- | :----- | :---- | :--------- |
| 01 | [Deployment guide: OIDC subject and SPA errors](./01-docs-deploy-oidc-and-spa-errors.md) | 1 | — |
| 02 | [Fix PWA tests on Node 25+](./02-fix-tests-node-webstorage.md) | H1 | — |
| 03 | [Move workflows to Node 22 via `.nvmrc`](./03-chore-node-22-nvmrc.md) | H2 | — (02 only if going past Node 22) |
| 04 | [SW: app shell for offline navigations](./04-fix-sw-navigation-fallback.md) | H4 | — |
| 05 | [SW: legacy-domain redirect kill switch](./05-feat-legacy-domain-sw-redirect.md) | 2 | 04; **merge only after `scrymat.cards` is live** |
| 06 | [Configurable signaling worker URL](./06-feat-configurable-signaling-url.md) | 3 | — |
| 07 | [STUN servers for peer connections](./07-feat-webrtc-stun-servers.md) | 3 | — |
| 08 | [Longer join window for hosted rooms](./08-fix-host-join-window.md) | 3 | — |
| 09 | [Restrict signaling worker CORS](./09-chore-signaling-cors-allowlist.md) | 3 | 06 |
| 10 | [Document signaling worker setup](./10-docs-signaling-worker-setup.md) | 3 | 06 |
| 11 | [Remove stale docs from the rename](./11-docs-stale-rename-references.md) | H3 | — |
| 12 | [Rename user-facing "scryglass" labels](./12-chore-rename-deck-format-labels.md) | H6 | — |

Tickets 06–08 are independent and can run in parallel. Ticket 05 is the
only one that is harmful if merged early: it sends `scryglass.cards`
visitors to `scrymat.cards`.
