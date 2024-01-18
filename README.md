# github-actions-workflow-template

This repository is a template for creating reusable GitHub Actions Workflows. Go through the below checklist
upon instantiating this template:
- Remove the [trigger update from template workflow](.github/workflows/trigger-update-from-template.yml)
- Edit the content of [the placeholder](.github/workflows/workflow.yml) for your reusable workflow.
- Edit this document and update the relevant sections

## Inputs

|     Name      | Required | Description       |
|:-------------:|:--------:|-------------------|
| example-input |   true   | An example input. |

## Secrets

|      Name      | Required | Description        |
|:--------------:|:--------:|--------------------|
| example-secret |   true   | An example secret. |

## Outputs

|      Name      | Description        |
|:--------------:|--------------------|
| example-output | An example output. |

## Permissions

|     Scope     | Level | Reason   |
|:-------------:|:-----:|----------|
| pull-requests | read  | Because. |

## Concurrency controls

Describe concurrency controls of the workflow.

## Timeouts

Describe the timeouts configured, if any.

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
    uses: infrastructure-blocks/github-actions-workflow-template/.github/workflows/workflow.yml@v1
    with:
      example-input: Nobody cares
    secrets:
      example-secret: ${{ secrets.EXAMPLE }}
```

### Releasing

The releasing is handled at git level with semantic versioning tags. Those are automatically generated and managed
by the [git-tag-semver-from-label-workflow](https://github.com/infrastructure-blocks/git-tag-semver-from-label-workflow).
