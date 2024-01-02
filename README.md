# npm-publish-from-label-workflow

This action publishes npm packages based on PR labels. It is meant to be run only on pull_request events
and the full recommended list of event types are listed in the [usage](#usage) section.

The action first looks for and verifies that the PR has exactly one of the following labels:
- `no version`
- `patch`
- `minor`
- `major`

In the event it doesn't find one, the action fails. When "no version" is specified, this action won't do anything
beyond checking labels.

It uses [npm-publish-action](https://github.com/infrastructure-blocks/npm-publish-action) to publish the package. Because reusable workflows have some differences to GitHub
actions, such as the inability to pass environment variables from the calling workflow, this workflow takes some
opinionated stances. For example, it *requires* that the library's `.npmrc` uses the NPM_TOKEN environment variable
as an authentication token. Refer to the `npm-publish-action` documentation to understand how to set up the package
settings.

What the action does depends on the event received from GitHub. When we are processing anything but a merge event,
then we run in prerelease mode. We run in release mode otherwise.

## Prerelease (not merging)
In prerelease mode, the `version` parameter is transformed into its corresponding prerelease equivalent:
- patch => prepatch
- minor => preminor
- major => premajor

The action then runs `npm version <prerelease-version>` with `--no-git-tag-version`. The design decision of not writing
commits and pushing commits was made in other to be the least intrusive to developers workflow as possible.

Running the prerelease version command will give use the prerelease lineage of the package. For example, the version
`1.2.3-beta.5` has the lineage `1.2.3-beta.<n>`, where `n` is the prerelease number.

If the lineage doesn't exist on the registry, then the action will publish it with a prerelease number of 0.

If the lineage does exist, then the action will scan the existing packages and find the latest of the lineage. It
will then increment its prerelease number by 1 and publish the resulting version.

In addition to publishing the package version, this action also associates a couple of dist tags to it:
- `gh-pr-<pr-number>`
- `git-sha-<git-sha>` (${{ github.event.pull_request.head.sha }})

It **does not** update the `latest` tag, on purpose.

## Release (merge event)
In release mode, the value of the PR label is kept as is. So, the `major` label will result in the action calling
`npm version major`. This time, we do generate a commit and a git tag, both of which are pushed at the end of the flow.

Because we are pushing commits to what we expect to be a protected branch, we allow the usage of specific GitHub tokens
through inputs. In the event that we are pushing against a protected branch, this needs to be a GitHub [PAT](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).

In the name of laziness, we leverage the [actions/checkout](https://github.com/actions/checkout) action with the provided token to set up the credentials
for authenticated git commands moving forward. You should know this if you expect no side effects after using
this action.

The resulting package is tagged with dist tags `latest` and `git-sha-<git-sha>` (the SHA of the base HEAD commit).

## Reporting
Failures are reported as PR comments. Failures include: missing PR label, unable to publish packages for some reason.

Successes are also reported as PR comments. They include the package version & dist tags published with
links in their registry.

## Notes
- This action has only been tested on the npmjs registry.

## Usage

```yaml
name: NPM Publish From Label

on:
  branches:
    - <your-release-branch>
  pull_request:
    types:
      # Having all these events ensures there is always a PR label.
      - opened
      - reopened
      - synchronize
      - labeled
      - unlabeled
      - closed

jobs:
  npm-publish-from-label:
    permissions:
      contents: read # Required to check out your project.
      pull-requests: write # Required to post comments.
    env:
      NPM_TOKEN: ${{ secrets.NPM_PUBLISH_TOKEN }}
    uses: infrastructure-blocks/npm-publish-from-label-workflow/.github/workflows/npm-publish-from-label.yml@v1
    with:  
      secrets:
        github-pat: ${{ secrets.PAT }}
```

### Releasing

The releasing is handled at git level with semantic versioning tags. Those are automatically generated and managed
by the [git-tag-semver-from-label-workflow](https://github.com/infrastructure-blocks/git-tag-semver-from-label-workflow).
