# docs: remove stale docs left over from the scryglass rename

## What do you want to build?

Several docs describe a state of the repo that's no longer true:

- `mkdocs.yml`: `site_url: "https://efischer19.github.io/scryglass/"`. The repo
  is now `scrymat`.
- `docs-src/index.md`: says "The repository, package names, and deployed URLs
  still use the legacy `scryglass` identifier". The repo and packages are
  renamed; only the infrastructure (S3 bucket, CloudFront, IAM role, live
  domain) still uses it.
- `docs-src/feature-request-automation.md`: documents
  `scripts/create-feature-issues.sh`, `meta/plans/extracted-feature-requests.json`
  and `meta/plans/ai-driven-feature-request-creation.md`, none of which exist.
  The real mechanism is `.github/workflows/create-planned-issues.yml`.
- `.github/copilot-instructions.md`: refers to "the `scryglass` project".

## Acceptance Criteria

- [ ] `mkdocs.yml` `site_url` points at `https://efischer19.github.io/scrymat/`
- [ ] The `docs-src/index.md` note accurately lists what still uses the legacy identifier (infrastructure only)
- [ ] `docs-src/feature-request-automation.md` is rewritten to describe `create-planned-issues.yml`: the folder layout (`NN-slug.md` tickets, `00-*.md` and `README.md` skipped), the required sections (H1 title, "What do you want to build?", "Acceptance Criteria" with at least one checkbox, optional "Implementation Notes"), the inputs (branch or commit_sha, file_path or folder_path, `human_reviewed`), and the duplicate-title skip
- [ ] `.github/copilot-instructions.md` says Scrymat
- [ ] `grep -rni scryglass docs-src mkdocs.yml .github/copilot-instructions.md` returns only intentional, explained references
- [ ] `scripts/build-docs.sh` still builds and `markdownlint-cli2` passes

## Implementation Notes (Optional)

- Out of scope: whether to republish the docs site. GitHub Pages is enabled, but no workflow deploys it, and it currently serves a stale "Static JS App" page. That's an owner decision tracked in `meta/plans/reset-to-useful/housekeeping.md` (H3).
- Don't edit historical ADRs or merged plan files. They're records of their time.
