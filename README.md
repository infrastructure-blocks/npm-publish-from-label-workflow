# npm-publish-from-label-workflow

This workflow publishes npm packages based on PR labels. It can be run any events, although some default values
for inputs only make sense on certain events. See the [usage](#usage) section for recommended set ups.

It starts by retrieving the PR associated with the SHA input. It then delegates to
[check-has-semver-label-workflow](https://github.com/infrastructure-blocks/check-has-semver-label-workflow). If the
PR check succeeds and the label associated to the PR is different than "no version", then the following occurs:
- The workflow dispatches to
[npm-publish-prerelease-workflow](https://github.com/infrastructure-blocks/npm-publish-prerelease-workflow) if 
`prerelease` is set to true
  - The distribution tags used are `git-sha-<input-sha>` and `gh-pr-<current-pr-number>`
- The workflow dispatches to [npm-publish-workflow](https://github.com/infrastructure-blocks/npm-publish-workflow) if
`prerelease` is set to true
  - The distribution tags used are `latest`, `git-sha-<input-sha>` and `gh-pr-<current-pr-number>`

The outcome of the release is reported as a
[status report](https://github.com/infrastructure-blocks/status-report-action).

## Inputs

|    Name    | Required | Description                                                                                                                                                                                                                                                                                                                               |
|:----------:|:--------:|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|    sha     |  false   | The commit SHA to tag. Defaults to the ${{ github.sha }}. If the event triggering this workflow is of type pull_request, be sure to set this parameter to either ${{ github.event.pull_request.head.sha }} or ${{ github.event.pull_request.base.sha }}. You probably don't want to tag the PR's default SHA, which is on a merge branch. |
| prerelease |  false   | Whether the label is to be interpreted as a prerelease or a full release. Defaults to false.                                                                                                                                                                                                                                              |
|  skip-ci   |  false   | Whether to include `[skip ci]` in the commit created by `npm version` when `prerelease` is false. This is especially useful if using this workflow on a push event with a GitHub PAT, for example. Defaults to false.                                                                                                                     |

## Secrets

|     Name     | Required | Description                                                                                                                                                       |
|:------------:|:--------:|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| github-token |   true   | The GitHub token used to configure the Git CLI. It should have the rights to push code and tags. When the branch or the tags are protected, this should be a PAT. |
|  npm-token   |   true   | The NPM token used to publish the package.                                                                                                                        |

## Outputs

N/A

## Permissions

|     Scope     | Level | Reason                                                                                                   |
|:-------------:|:-----:|----------------------------------------------------------------------------------------------------------|
|   contents    | write | Required to push code when `prerelease` is false and the `github-token` provided is ${{ github.token }}. |
| pull-requests | write | Required to post comments about the status of this workflow.                                             |

## Concurrency controls

N/A

## Timeouts

N/A

## Usage

### Prerelease

```yaml
name: NPM Publish Prerelease From Label

on:
  pull_request:
    types:
      - opened
      - reopened
      - synchronize
      - labeled
      - unlabeled

jobs:
  npm-publish-prerelease:
    uses: infrastructure-blocks/npm-publish-from-label-workflow/.github/workflows/workflow.yml@v3
    permissions:
      contents: write
      pull-requests: write
    with:
      sha: ${{ github.event.pull_request.head.sha }}
      prerelease: true
    secrets:
      # Should use the default token here, since we are not pushing anything.
      github-token: ${{ github.token }}
      npm-token: ${{ secrets.NPM_PUBLISH_TOKEN }}
```

### Release

```yaml
name: NPM Publish Release From Label

on:
  push:
    branches:
      - <you-release-branch>

jobs:
  npm-publish-release:
    uses: infrastructure-blocks/npm-publish-from-label-workflow/.github/workflows/workflow.yml@v3
    permissions:
      contents: write
      pull-requests: write
    with:
      skip-ci: true
    secrets:
      github-token: ${{ secrets.PAT }}
      npm-token: ${{ secrets.NPM_PUBLISH_TOKEN }}
```

### Releasing

The releasing is handled at git level with semantic versioning tags. Those are automatically generated and managed
by the [git-tag-semver-from-label-workflow](https://github.com/infrastructure-blocks/git-tag-semver-from-label-workflow).
