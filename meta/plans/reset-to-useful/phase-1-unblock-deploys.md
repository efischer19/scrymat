# Phase 1: Unblock deploys

**Status:** Not started · **Estimated effort:** under 1 hour · [Back to index](./README.md)

## Goal

`main` deploys automatically again. `scryglass.cards` serves the current Scrymat
build, and any deep link (`/deck`, `/match/ABC123`) loads the app instead of
returning 403. No code changes.

## Background

### Deploy failure

`.github/workflows/deploy-aws.yml` assumes `vars.AWS_ROLE_ARN`
(`arn:aws:iam::954257023024:role/scryglass-deploy`) through GitHub OIDC. Every run
since 2026-08-03 fails at "Configure AWS credentials":

```text
Could not assume role with OIDC: Not authorized to perform sts:AssumeRoleWithWebIdentity
```

The repo's OIDC subject customization (`gh api repos/efischer19/scrymat/actions/oidc/customization/sub`) is:

```json
{"use_default": true, "use_immutable_subject": true,
 "sub_claim_prefix": "repo:efischer19@7373924/scrymat@1200416986"}
```

The token's `sub` claim is therefore
`repo:efischer19@7373924/scrymat@1200416986:ref:refs/heads/main`. That uses
immutable owner and repo IDs, not the classic `repo:efischer19/scryglass:...`
form the role was almost certainly set up with in April 2026. Either the rename
or the switch to immutable subjects (or both) broke the match. **Read the trust
policy to confirm which before changing it.**

### 403 on deep links

The site is a SPA with client-side routing (ADR-009), served from S3 through
CloudFront distribution `EL8VCFL4HWB2J`. S3 returns 403 for keys that don't
exist, and the distribution evidently has no custom error responses mapping
403/404 to `/index.html`. `docs-src/deployment.md` (step 4 under CloudFront)
says it should.

## Prerequisites

- AWS CLI session with IAM and CloudFront read/write in account `954257023024`.
  If expired, ask the user to run `! aws login`.
- `gh` CLI authenticated, which it already is on this machine.

## Steps

1. **Snapshot current config** into the scratchpad:

   ```bash
   aws iam get-role --role-name scryglass-deploy \
     --query Role.AssumeRolePolicyDocument > trust-before.json
   aws cloudfront get-distribution-config --id EL8VCFL4HWB2J > cf-before.json
   ```

2. **Diagnose the trust policy.** Look at the
   `token.actions.githubusercontent.com:sub` condition. Expect it to name
   `repo:efischer19/scryglass:*`, or something similarly pre-rename.
3. **Update the trust policy** after confirming with the user. Recommended
   condition, which allows only this repo's `main` branch plus manual dispatch
   from `main`:

   ```json
   "Condition": {
     "StringEquals": {
       "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
     },
     "StringLike": {
       "token.actions.githubusercontent.com:sub": [
         "repo:efischer19@7373924/scrymat@1200416986:ref:refs/heads/main",
         "repo:efischer19/scrymat:ref:refs/heads/main"
       ]
     }
   }
   ```

   The immutable form is what GitHub sends today. The name-based form is a
   harmless fallback in case the customization is ever reset. Apply with
   `aws iam update-assume-role-policy`.
4. **Add SPA error responses** to the distribution after confirming. Add
   `CustomErrorResponses` for 403 and 404 → `ResponsePagePath: /index.html`,
   `ResponseCode: 200`, `ErrorCachingMinTTL: 10`. Use
   `aws cloudfront update-distribution --if-match <ETag>` with the edited
   `DistributionConfig`, or do it in the console. Wait for the status to
   reach `Deployed`.
   - This means a truly missing `/assets/foo.js` also returns `index.html` with
     status 200. That's the standard SPA trade-off and acceptable here.
5. **Re-run the deploy:**
   `gh workflow run deploy-aws.yml -R efischer19/scrymat --ref main`, then
   `gh run watch`.
6. **Verify** (see acceptance criteria).
7. **Update docs:**
   - README "Project status" table: change the live-site row to say it serves
     current `main` and that deep links work.
   - `docs-src/deployment.md`: add the immutable-subject trust condition example
     and a note that renaming the repo requires checking the trust policy.
   - Index status → Done, with the date.

## Acceptance criteria

- [ ] Latest `Deploy to AWS` run on `main` is green, including the smoke test.
- [ ] `curl -s https://scryglass.cards | grep -o '<title>[^<]*'` shows `Scrymat`.
- [ ] `curl -s -o /dev/null -w '%{http_code}' https://scryglass.cards/match/ABC123` → `200`.
- [ ] `last-modified` header on `https://scryglass.cards/` is today.
- [ ] In a real browser (`/run` skill or Chrome), load a deck, draw, and move a
      card to the battlefield on the live site.
- [ ] A browser that previously had the old site picks up the new build after
      one or two reloads. The service worker is cache-first, so a new `sw.js`
      must install before the new shell shows.

## Rollback

- Trust policy: `aws iam update-assume-role-policy --role-name scryglass-deploy --policy-document file://trust-before.json`.
- CloudFront: re-apply the `DistributionConfig` from `cf-before.json` with the
  current ETag.
- Bad build: revert the commit on `main`. The deploy workflow redeploys.

## Out of scope

- Renaming the role, bucket or distribution to `scrymat-*`. That's cosmetic, and
  the README already says the infrastructure keeps the legacy identifier. Revisit
  only if it becomes annoying.
- The new domain (Phase 2).

## Log

*(Sessions: append dated notes here.)*
