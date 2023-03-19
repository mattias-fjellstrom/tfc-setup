# GitHub Actions for setting up environment variables for Terraform Cloud

With this action you can make available environment variables for further interaction with Terraform Cloud using actions such as the [create workflow action](https://github.com/mattias-fjellstrom/tfc-create-workspace/blob/main/action.yaml).

- This action populates the following environment variables
  - `TERRAFORM_CLOUD_TOKEN` with an API-token for authentication (required)
  - `TERRAFORM_CLOUD_ORGANIZATION` with the name of the Terraform Cloud organization
  - `TERRAFORM_CLOUD_PROJECT` with the name of the Terraform Cloud project to use
  - `TERRAFORM_CLOUD_WORKSPACE` with the name of the Terraform Cloud workspace to use
- If no value is provided for a property then you will need to provide the corresponding value as input to further actions instead.

## Sample workflow

Below is a full sample workflow that sets up a new workspace in Terraform Cloud when a pull-request is opened, and deletes the workspace once the pull-request is closed. If the pull request is opened for a branch named `feature-1` the resulting workspace will be named `my-workspace-feature-1` in this sample.

```yaml
name: Terraform Cloud workspaces for pull-requests

on:
  pull_request:
    types:
      - opened
      - closed

env:
  ORGANIZATION: my-terraform-cloud-organization
  PROJECT: my-terraform-cloud-project
  WORKSPACE: my-workspace-${{ github.head_ref }}

jobs:
  create-workspace:
    if: ${{ github.event.action == 'opened' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: mattias-fjellstrom/tfc-setup@v1
        with:
          token: ${{ secrets.TERRAFORM_CLOUD_TOKEN }}
          organization: ${{ env.ORGANIZATION }}
          project: ${{ env.PROJECT }}
          workspace: ${{ env.WORKSPACE }}
      - uses: mattias-fjellstrom/tfc-create-workspace@v1
        with:
          directory: infrastructure
          branch: ${{ github.head_ref }}
  delete-workspace:
    if: ${{ github.event.action == 'closed' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: mattias-fjellstrom/tfc-setup@v1
        with:
          token: ${{ secrets.TERRAFORM_CLOUD_TOKEN }}
          organization: ${{ env.ORGANIZATION }}
          project: ${{ env.PROJECT }}
          workspace: ${{ env.WORKSPACE }}
      - uses: mattias-fjellstrom/tfc-delete-workspace@v1
```