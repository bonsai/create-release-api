# create-release-api

A minimal GitHub-native release API.

## Architecture

```text
client
  │ GitHub REST API: workflow_dispatch
  ▼
bonsai/create-release-api
  │ GitHub Actions
  ▼
POST /repos/{owner}/{repo}/releases
  │
  ▼
GitHub Release
```

There is intentionally **no always-on server**. GitHub Actions is the execution
layer and GitHub REST is the transport/API surface.

## Why this design

- No VPS, container, or external hosting.
- OpenAPI describes the public contract.
- `workflow_dispatch` is the simple trigger that GitHub provides for running a
  workflow through REST.
- The workflow uses a token stored as `RELEASE_TOKEN` because the built-in
  `GITHUB_TOKEN` is scoped to this repository and is not suitable for creating
  releases in arbitrary target repositories.
- The target repository can be `bonsai/hw-msedge-ext` or another repository
  permitted by the token.

GitHub documents `workflow_dispatch` as a REST-triggerable workflow event, and
its release API as `POST /repos/{owner}/{repo}/releases`.

## Setup

Create a repository secret named `RELEASE_TOKEN` with the minimum permissions
needed for the target repositories. A GitHub App installation token is
preferable for a reusable service; a fine-grained PAT is sufficient for a
personal setup.

## Call

```bash
gh api \
  --method POST \
  -H 'Accept: application/vnd.github+json' \
  /repos/bonsai/create-release-api/actions/workflows/create-release.yml/dispatches \
  -f ref=main \
  -f 'inputs[target_repo]=bonsai/hw-msedge-ext' \
  -f 'inputs[tag_name]=v1.0.0' \
  -f 'inputs[name]=Hello World 1.0.0' \
  -f 'inputs[generate_release_notes]=true'
```

The request is asynchronous: the API dispatches a workflow, and the workflow
performs the release creation. Use the Actions run as the job/result record.

## OpenAPI

See [`openapi.yaml`](./openapi.yaml).
