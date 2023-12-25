# github-actions-workflow-template

This repository is a template for creating reusable GitHub Actions Workflows. Go through the below checklist
upon instantiating this template:
- Rename and replace the content of [the placeholder](.github/workflows/reusable-workflow.yml) for your reusable workflow.
- Edit this section and the usage section and replace with a meaningful description of your workflow

## Usage

```yaml
name: Template Usage

on:
  push: ~

# This needs to be a superset of what your workflow requires
permissions:
  pull-requests: read

jobs:
  example-job:
    uses: infrastructure-blocks/github-actions-workflow-template/.github/workflows/reusable-workflow.yml@v1
    with:
      example-input: Nobody cares
    secrets:
      example-secret: ${{ secrets.EXAMPLE }}
```

### Releasing

The releasing is handled at git level with semantic versioning tags. Those are automatically generated and managed
by the [git-tag-semver-from-label-action](https://github.com/infrastructure-blocks/git-tag-semver-from-label-action).
