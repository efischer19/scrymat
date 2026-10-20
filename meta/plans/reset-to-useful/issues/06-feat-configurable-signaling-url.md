# feat: make the signaling worker URL configurable at build time

## What do you want to build?

Remote play can't reach the signaling worker in production.
`WebRtcDataChannelManager` defaults to `DEFAULT_SIGNALING_BASE_URL = '/api/room'`,
which is same-origin, and `App.tsx` calls `useWebRtcMatch({ onMessage })` without
a `signalingBaseUrl`. The PWA is served from S3 and CloudFront, so `/api/room`
never reaches the Cloudflare Worker.

Add a build-time `VITE_SIGNALING_URL` that the app uses when it's set, falling
back to `/api/room` for local development, and wire it into the AWS deploy
build.

## Acceptance Criteria

- [ ] `VITE_SIGNALING_URL` is declared in `packages/pwa/src/vite-env.d.ts` (`ImportMetaEnv`)
- [ ] When set, signaling requests go to `${VITE_SIGNALING_URL}/<ROOM>/offer|answer`; when unset, the current `/api/room` behaviour is unchanged
- [ ] The URL is resolved in one place, so components and hooks don't read `import.meta.env` directly
- [ ] The `Build` step in `.github/workflows/deploy-aws.yml` sets `VITE_SIGNALING_URL: ${{ vars.SIGNALING_URL }}`, and an empty or unset variable is safe
- [ ] Unit tests cover both the configured and default URLs
- [ ] `docs-src/deployment.md` lists the new `SIGNALING_URL` repository variable in its variables table

## Implementation Notes (Optional)

- The expected production value is `https://scrymat-signaling.<account>.workers.dev/api/room`. The human sets the repo variable once the worker is deployed; don't hard-code it.
- The worker already sends `Access-Control-Allow-Origin: *`, so a cross-origin call works without worker changes.
- `normalizeBaseUrl` in `WebRtcDataChannelManager.ts` already handles trailing slashes, so reuse it.
- This was chosen over a CloudFront `/api/*` → Worker behaviour to avoid extra AWS configuration. See `meta/plans/reset-to-useful/phase-3-remote-play.md`.
