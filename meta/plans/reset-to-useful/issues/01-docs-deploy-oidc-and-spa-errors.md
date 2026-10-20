# docs: cover OIDC immutable subjects and SPA error responses in the deployment guide

## What do you want to build?

Update `docs-src/deployment.md` so that setting up deploys from scratch, or
after a repo rename, works the first time.

The AWS deploy broke after the `scryglass` → `scrymat` rename because the IAM
role's OIDC trust policy no longer matched the token's `sub` claim. This repo
uses GitHub's **immutable subject** format
(`repo:efischer19@7373924/scrymat@1200416986:ref:refs/heads/main`), but the
guide only shows the name-based form, if it shows one at all. The guide also
needs to stress the CloudFront 403/404 → `/index.html` error responses, which
are missing from the live distribution and make every deep link return 403.

## Acceptance Criteria

- [ ] The IAM trust policy example uses a `StringLike` condition on `token.actions.githubusercontent.com:sub` and shows both the immutable form and the name-based form
- [ ] The guide explains how to read the repo's current subject format: `gh api repos/<owner>/<repo>/actions/oidc/customization/sub`
- [ ] A short "Renaming the repository" note says the trust policy must be checked after a rename, and names the symptom: `Not authorized to perform sts:AssumeRoleWithWebIdentity`
- [ ] The CloudFront section has a troubleshooting line: "deep links return 403 → custom error responses missing"
- [ ] `markdownlint-cli2` passes on the changed file

## Implementation Notes (Optional)

- Docs only. Do not change workflows or infrastructure.
- Background and verified evidence: `meta/plans/reset-to-useful/phase-1-unblock-deploys.md`.
- Owner ID `7373924` and repo ID `1200416986` are public, so it's fine to include them in docs.
