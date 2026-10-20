# chore: restrict signaling worker CORS to known site origins

## What do you want to build?

`packages/signaling/src/index.ts` answers every request with
`Access-Control-Allow-Origin: *`. Once the PWA calls the worker cross-origin
(ticket 06), restrict this to the origins that actually host Scrymat, so other
sites can't use the worker as a free relay.

## Acceptance Criteria

- [ ] Allowed origins are configurable through a `wrangler.toml` `[vars]` entry (for example `ALLOWED_ORIGINS`, comma-separated), with defaults of `https://scrymat.cards`, `https://scryglass.cards` and `http://localhost:5173`
- [ ] When the request `Origin` is allowed, the response echoes it in `Access-Control-Allow-Origin` and includes `Vary: Origin`
- [ ] When the request `Origin` is not allowed, the response has no `Access-Control-Allow-Origin`; a preflight `OPTIONS` returns 403
- [ ] Requests with no `Origin` header (curl, server-to-server) still work, so smoke tests keep working
- [ ] `packages/signaling/src/index.test.ts` covers allowed, disallowed and missing origins; existing tests are updated

## Implementation Notes (Optional)

- Low risk, low priority: rooms live 5 minutes and hold only SDP blobs.
- Don't break local development: `http://localhost:5173` must stay in the defaults, and `vite preview`'s port (`4173`) is worth adding as well.
