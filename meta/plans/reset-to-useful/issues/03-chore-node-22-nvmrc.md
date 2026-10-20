# chore: move CI and deploy workflows to Node 22 via a single .nvmrc

## What do you want to build?

Node 20 reached end of life in April 2026, but every workflow still pins it.
Add a root `.nvmrc` and have all workflows read it, so local development and
CI use the same version from one source of truth.

Files that currently hard-code Node 20:

- `.github/workflows/ci.yml` (three `setup-node` steps)
- `.github/workflows/deploy-aws.yml`
- `.github/workflows/deploy-signaling.yml`

## Acceptance Criteria

- [ ] A root `.nvmrc` contains `22`
- [ ] Every `actions/setup-node` step uses `node-version-file: .nvmrc` instead of `node-version`
- [ ] Root `package.json` declares `"engines": { "node": ">=22" }`
- [ ] All CI jobs pass, including Playwright E2E and Lighthouse
- [ ] `CONTRIBUTING.md` or `docs-src/index.md` mentions the Node version and `.nvmrc`

## Implementation Notes (Optional)

- Depends on the Node 25+ test fix (ticket 02) only if you move past 22. Staying on 22 is the safe choice.
- Keep `cache: npm` on the `setup-node` steps.
- Don't change anything else in the deploy workflows.
