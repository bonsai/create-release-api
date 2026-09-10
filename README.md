# create-release-api

GitHub-native release API for `bonsai/*` projects.

## Release definition

A **Release** is an intentional, versioned, user-consumable artifact or product.

### Releaseになるもの

- browser extension package/product
- CLI or executable
- library/package
- deployable web application
- versioned dataset snapshot
- packaged template

### Releaseにならないもの

- source-only commit
- documentation-only change
- Issue / Pull Request
- workflow or CI-only change
- test-only change
- internal configuration
- draft work

A tag alone is not a release. The caller must explicitly set `release=true`, and
the tag must follow `vMAJOR.MINOR.PATCH` with an optional prerelease suffix.
The canonical policy is in [`release-policy.yaml`](./release-policy.yaml).

## Architecture

```text
client
  │ GitHub REST: workflow_dispatch
  ▼
bonsai/create-release-api
  │ GitHub Actions
  │ validate release policy
  ▼
POST /repos/{target_repo}/releases
  │
  ▼
GitHub Release
```

There is intentionally **no always-on server**. GitHub Actions is the execution
layer and GitHub REST is the transport/API surface.

## Why this design

- No VPS, container, or external hosting.
- OpenAPI is the contract.
- `workflow_dispatch` is the HTTP-triggerable entry point.
- The workflow validates that a requested release is intentional and versioned.
- `RELEASE_TOKEN` is used because the built-in `GITHUB_TOKEN` is scoped to this
  repository and cannot generally create releases in arbitrary target repos.
- The same API can serve `bonsai/hw-msedge-ext` and other releaseable projects.

## Setup

Add a repository secret named `RELEASE_TOKEN` to `bonsai/create-release-api`.
Give the token the minimum `Contents: write` access to the target repositories.
For a reusable long-lived service, a GitHub App installation token is preferred;
a fine-grained PAT is sufficient for a personal setup.

The connected GitHub tool can write the workflow and policy files, but it cannot
set repository secrets or dispatch the workflow itself. After the secret is set,
the following command runs the API end-to-end.

## Run: `hw-msedge-ext` v1.0.0

```bash
gh api \
  --method POST \
  -H 'Accept: application/vnd.github+json' \
  -H 'X-GitHub-Api-Version: 2026-03-10' \
  /repos/bonsai/create-release-api/actions/workflows/create-release.yml/dispatches \
  -f ref=main \
  -f 'inputs[target_repo]=bonsai/hw-msedge-ext' \
  -f 'inputs[tag_name]=v1.0.0' \
  -f 'inputs[release]=true' \
  -f 'inputs[name]=Hello World 1.0.0' \
  -f 'inputs[generate_release_notes]=true'
```

The dispatch is asynchronous. Check the Actions run for execution status and
then the target repository's Releases page for the created release.

## OpenAPI

See [`openapi.yaml`](./openapi.yaml).
