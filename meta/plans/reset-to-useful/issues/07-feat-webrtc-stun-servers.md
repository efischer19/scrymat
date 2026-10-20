# feat: configure STUN servers so remote play works across networks

## What do you want to build?

`WebRtcDataChannelManager.connect()` creates `new RTCPeerConnection()` with no
configuration. With no ICE servers, browsers gather only host candidates, so
two peers can connect only on the same LAN. Every same-machine and same-Wi-Fi
test passes, yet a real remote game between two homes, or with a phone on
cellular, will almost always fail.

Pass public STUN servers by default, and allow overriding them through an
option.

## Acceptance Criteria

- [ ] `WebRtcDataChannelManagerOptions` (and `useWebRtcMatch` options) accept an optional `iceServers: RTCIceServer[]`
- [ ] The default is `[{ urls: ['stun:stun.cloudflare.com:3478', 'stun:stun.l.google.com:19302'] }]`, defined as a named constant
- [ ] `new RTCPeerConnection({ iceServers })` receives the configured or default list (unit-tested with a mocked `RTCPeerConnection`)
- [ ] The README "Remote play" status row, or a code comment, records that STUN-only won't connect on some symmetric-NAT networks and that TURN is out of scope for now
- [ ] Existing networking tests still pass

## Implementation Notes (Optional)

- Signaling is non-trickle: the code waits for `iceGatheringState === 'complete'` before publishing the SDP. With STUN this takes noticeably longer, roughly 1–5 s. Check that `waitForIceGatheringComplete` has a sensible timeout, and that the lobby UI shows a "preparing connection…" state rather than looking frozen.
- No TURN in this ticket. TURN needs server-minted credentials and has a cost; it'll be its own ticket if needed.
