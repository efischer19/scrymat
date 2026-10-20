# Phase 3: Make remote play work

**Status:** Not started · **Estimated effort:** 1–3 sessions · **Depends on:** Phase 1
(Phase 2 recommended first so the CORS/origin story is settled) · [Back to index](./README.md)

## Goal

Two people on **different networks** can open the public site, host and join a
room by code or `/match/:code` link, and play a synchronized game. The README
"Remote play" row moves from *Partial* to *Works*, with stated limitations.

## Background: what exists vs. what's missing

**Exists and tested:**

- `packages/signaling`: Cloudflare Worker `scrymat-signaling`.
  `POST/GET /api/room/:code/(offer|answer)`, stored in KV binding
  `SCRYMAT_SIGNALING` with a 300 s TTL, CORS `*`.
- `packages/pwa/src/networking/WebRtcDataChannelManager.ts`: host/guest
  handshake through polling, an ordered `RTCDataChannel`, room codes, and
  non-trickle ICE (waits for `iceGatheringState === 'complete'`).
- `useWebRtcMatch`, `RemoteMatchLobby`, `/match/:code` routing, presence
  indicators, and host → guest `GameState` resync.

**Missing (each is a step below):**

1. **Worker never deployed.** `.github/workflows/deploy-signaling.yml` fails
   because the `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID` secrets
   aren't set. `wrangler.toml` has `REPLACE_WITH_KV_NAMESPACE_ID`
   placeholders. Dependabot PR #213 bumps `cloudflare/wrangler-action` v3 → v4.
2. **The app can't reach the worker.** `DEFAULT_SIGNALING_BASE_URL = '/api/room'`
   is same-origin, and `App.tsx` calls `useWebRtcMatch({ onMessage })` with no
   `signalingBaseUrl`. On CloudFront, `/api/room` hits S3 (403 today, and
   `index.html` after Phase 1).
3. **No ICE servers.** `new RTCPeerConnection()` takes no config, so only host
   candidates are gathered. Connections across NATs, which covers essentially
   every real remote game, will fail. This is the sneakiest gap: same-machine
   and same-LAN tests pass.
4. **Possible join-window problem (verify).** `DEFAULT_POLL_TIMEOUT_MS = 30_000`.
   If the host gives up waiting for an answer after 30 s, the guest must join
   within 30 s of the host clicking "Generate Room Code". That's too tight for
   sharing a code over text. The worker keeps rooms for 300 s.

## Decisions to make at the start of the session (ask the user)

- **How the app finds the worker.** Recommended: a build-time
  `VITE_SIGNALING_URL`, defaulting to `/api/room` for dev. Set it in
  `deploy-aws.yml` from a repo variable, pointing at
  `https://scrymat-signaling.<account>.workers.dev/api/room`. It's a small code
  change, needs no AWS work, and the worker already sends CORS `*`. The
  alternative is a CloudFront behavior `/api/*` → workers.dev origin, which
  keeps it same-origin with no code change but adds CloudFront config
  (POST/OPTIONS allowed, caching disabled, `Host` header handling).
  A custom domain like `signal.scrymat.cards` would need the zone on
  Cloudflare. It's on Route 53, so skip that.
- **NAT traversal.** Recommended for now: public STUN only
  (`stun:stun.cloudflare.com:3478` and `stun:stun.l.google.com:19302`), and
  document that some strict/symmetric-NAT networks (some mobile carriers,
  corporate Wi-Fi) won't connect. TURN, for example Cloudflare's TURN service,
  needs short-lived credentials minted server-side. That could live in the
  same worker, but it's a separate, later step with its own cost; check current
  pricing before proposing it.

## Steps

1. **Cloudflare setup** (user-driven; confirm each):
   - `npx wrangler kv namespace create SCRYMAT_SIGNALING`, and the same again
     with `--preview`. Put the IDs in `packages/signaling/wrangler.toml`. KV
     IDs aren't secrets, so committing them is fine.
   - Create an API token scoped to *Workers Scripts: Edit* and *Workers KV:
     Edit* on the one account. Have the user add it themselves:
     `! gh secret set CLOUDFLARE_API_TOKEN -R efischer19/scrymat` and
     `! gh secret set CLOUDFLARE_ACCOUNT_ID -R efischer19/scrymat`. Never paste
     tokens into the chat.
2. **Merge Dependabot #213** (wrangler-action v4) once CI is green, then make
   sure `deploy-signaling.yml` still works with v4 inputs.
3. **Deploy and smoke-test the worker:**

   ```bash
   curl -X POST https://scrymat-signaling.<acct>.workers.dev/api/room/TEST01/offer \
     -H 'content-type: application/json' -d '{"sdp":"x"}'
   curl https://scrymat-signaling.<acct>.workers.dev/api/room/TEST01/offer
   ```

   Check the exact request shape in `packages/signaling/src/index.ts`.
4. **Code PR: configurable signaling URL and ICE servers.**
   - Read `import.meta.env.VITE_SIGNALING_URL` (add the type to `vite-env.d.ts`)
     and pass it into `useWebRtcMatch` in `App.tsx`, or default it inside
     `WebRtcDataChannelManager`.
   - Add an `iceServers` option with the STUN defaults above, passed to
     `new RTCPeerConnection({ iceServers })`.
   - In `deploy-aws.yml`, set `VITE_SIGNALING_URL: ${{ vars.SIGNALING_URL }}`
     on the Build step, and create that repo variable.
   - Unit tests: the manager uses the configured URL and passes `iceServers`
     through.
5. **Investigate and fix the join window** (gap 4). Read `connectAsHost` and
   `pollForDescription`. If the host times out at 30 s, raise its answer-wait to
   roughly the worker TTL (about 5 min), keep the guest's offer-wait short, and
   make sure the UI shows "waiting for opponent…" rather than an error. Add or
   adjust tests.
6. **Optional hardening:** restrict worker CORS from `*` to the site origins
   (`https://scrymat.cards`, `https://scryglass.cards`,
   `http://localhost:5173`). It's low risk either way, since rooms are
   short-lived and hold only SDP.
7. **Real-world test** (needs the user and a second device):
   - Laptop on home Wi-Fi hosts; phone **on cellular, Wi-Fi off** joins via
     the `/match/:code` link.
   - Play a few actions both ways, kill the phone's connection briefly, and
     confirm the host resync works.
   - Record results, including any network that failed to connect, in the
     Log.
8. **Update docs:** README status row and Quick Start "Remote Host / Join"
   (remove the WARNING if it works, state the STUN-only limitation), plus a
   short section in `docs-src/deployment.md` for the worker setup.

## Acceptance criteria

- [ ] `Deploy Signaling Worker` is green on `main`.
- [ ] The live site's network tab shows signaling requests going to the worker,
      not `/api/room` on CloudFront.
- [ ] Host on Wi-Fi plus guest on cellular connect and sync actions both ways.
- [ ] Guest can join at least 2 minutes after the host generated the code.
- [ ] README "Remote play" row is updated with honest limitations.

## Explicitly not in this phase

- **Wiring commit-reveal and seeded shuffling into the app.** Today remote sync
  sends plain actions and full `GameState`, so a peer with DevTools can see
  hidden cards. That fits the "trusted casual play" model for now. If it gets
  picked up later, write it as its own plan (Phase 4), since it touches the
  sync protocol, and add an ADR if the design changes.
- **The local AI agent** (`agent/SKILL.md`, `ws://127.0.0.1:8765`).

## Log

*(Sessions: append dated notes here.)*
