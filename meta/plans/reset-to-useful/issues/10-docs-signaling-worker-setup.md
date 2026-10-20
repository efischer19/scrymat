# docs: document signaling worker setup and deployment

## What do you want to build?

The Cloudflare signaling worker (`packages/signaling`) has a deploy workflow
(`.github/workflows/deploy-signaling.yml`) but no setup docs. It has never
deployed successfully because the secrets and KV namespace were never created.
Add a "Signaling worker (Cloudflare)" section to `docs-src/deployment.md` so a
human can do the one-time setup.

## Acceptance Criteria

- [ ] KV namespace creation is documented: `npx wrangler kv namespace create SCRYMAT_SIGNALING`, plus `--preview`, and where the IDs go in `packages/signaling/wrangler.toml` (KV IDs aren't secret and are committed)
- [ ] The API token's minimum scopes are documented (Workers Scripts: Edit and Workers KV Storage: Edit, single account)
- [ ] Setting the repo secrets is documented with `gh secret set CLOUDFLARE_API_TOKEN` and `gh secret set CLOUDFLARE_ACCOUNT_ID`, run by the human and never pasted into an issue or chat
- [ ] How to find the deployed `*.workers.dev` URL is documented, as is setting it as the `SIGNALING_URL` repository variable (from ticket 06)
- [ ] A curl smoke test for `POST`/`GET /api/room/:code/offer` is included, with the request shape checked against `packages/signaling/src/index.ts`
- [ ] The workflow's trigger paths are explained (it deploys only on changes under `packages/signaling/**` or the workflow file, or through `workflow_dispatch`)

## Implementation Notes (Optional)

- Docs only. Don't commit real KV IDs or account IDs.
- Check the docs against `cloudflare/wrangler-action@v4` inputs. Dependabot PR #213 upgrades from v3.
