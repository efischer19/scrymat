# fix: give guests more than 30 seconds to join a hosted room

## What do you want to build?

When a host clicks **Generate Room Code**, `connectAsHost()` publishes an offer
and then calls `pollForDescription('answer')`. Polling uses the same
`DEFAULT_POLL_TIMEOUT_MS = 30_000` as the guest, so the host gives up with
`Timed out waiting for answer.` 30 seconds after the code is generated. That's
not enough time to text a code to a friend and have them open the link. The
signaling worker keeps rooms for `ROOM_TTL_SECONDS = 300`.

Let the host wait about as long as the room lives, and make the waiting state
obvious in the UI.

## Acceptance Criteria

- [ ] The host's answer-wait timeout is separate from the guest's offer-wait timeout, and defaults to just under the worker TTL (about 290 s)
- [ ] The guest's offer-wait timeout stays short (30 s), so a mistyped code fails fast with a clear message
- [ ] While waiting, the lobby shows a "Waiting for opponent…" state with the room code or link and a working Cancel button
- [ ] When the host times out, the message says the room expired and offers to generate a new code
- [ ] Unit tests (fake timers) cover a host waiting more than 30 s and then connecting, a host timeout, and a guest timeout

## Implementation Notes (Optional)

- Relevant code: `packages/pwa/src/networking/WebRtcDataChannelManager.ts` (`pollForDescription`, `connectAsHost`) and `packages/pwa/src/components/RemoteMatchLobby.tsx`.
- Polling every 1 s for 5 minutes is about 300 GETs per hosted room, well within Workers KV free-tier read limits. Optionally back off to 2–3 s after the first 30 s.
- Keep the worker's 300 s TTL in sync. If you add a shared constant, the signaling and PWA packages don't currently share code, so a comment cross-referencing the two is fine.
