# Publishing GHA Helm Charts

This guide provides step-by-step instructions for publishing new versions of the GitHub Actions Runner controller Helm charts using the automated GitHub Actions workflow.

## Overview

The GHA Helm chart publishing process is handled by two workflows:

1. **gha-bump-and-publish-charts.yaml** - Bumps chart versions and creates a PR
2. **gha-publish-chart.yaml** - Builds the controller image and publishes the Helm charts

The process automatically updates version numbers in the charts, creates a PR for review, and then triggers the publishing workflow to build and publish both the controller image and Helm charts.

## Charts Published

The workflow publishes two Helm charts:

1. **gha-runner-scale-set-controller** - The controller chart
2. **gha-runner-scale-set** - The runner scale set chart

Both charts are published as OCI artifacts to the GitHub Container Registry (GHCR).

## Prerequisites

- Write access to the repository
- Permissions to trigger GitHub Actions workflows

## Step-by-Step Publishing Process

### 1. Trigger the Bump and Publish Workflow

1. Navigate to the Actions tab in the repository
2. Select the "Bump and Publish GHA Helm Charts" workflow
3. Click "Run workflow"
4. Enter the required parameters:
   - **new_version**: The new version number for the charts (e.g., `0.10.2`)
   - **ref**: (Optional) The branch, tag, or SHA to cut a release from (leave empty to use the default branch)
   - **push_to_registries**: Whether to push images to container registries (set to `true` for actual releases)
5. Click "Run workflow"

### 2. Workflow Actions

When the workflow runs, it performs the following steps:

#### a. Chart Version Updates

The workflow automatically:

1. Checks out the specified reference (or default branch)
2. Configures Git with the user information
3. Updates the version and appVersion fields in:
   - `charts/gha-runner-scale-set-controller/Chart.yaml`
   - `charts/gha-runner-scale-set/Chart.yaml`
4. Verifies that the updated versions match using `./hack/check-gh-chart-versions.sh`

#### b. Pull Request Creation

The workflow:
1. Creates a new pull request with:
   - Title: `chore: bump chart versions to {new_version}`
   - Branch name: `bump-chart-version-{new_version}`
   - Base branch: `master`
   - Description detailing which charts are being updated

#### c. Review and Merge

1. Maintainers should review the PR to ensure the version changes are correct
2. After approval, the PR can be merged to the master branch

### 3. Publishing the Charts

After the PR is merged, you need to manually trigger the publish workflow:

1. Navigate to the Actions tab in the repository
2. Select the "(gha) Publish Helm Charts" workflow
3. Click "Run workflow"
4. Enter the required parameters:
   - **ref**: (Optional) Reference to publish from (leave empty to use default branch)
   - **release_tag_name**: Version tag for the controller image (should match the new chart version)
   - **push_to_registries**: Set to `true` to publish images to registries
   - **publish_gha_runner_scale_set_controller_chart**: Set to `true` to publish the controller chart
   - **publish_gha_runner_scale_set_chart**: Set to `true` to publish the runner scale set chart
5. Click "Run workflow"

Alternatively, if you use the automated process, after the PR is merged, the "Bump and Publish" workflow will automatically trigger the "Publish Helm Charts" workflow with the appropriate parameters.

### 4. Workflow Actions for Publishing

The publish workflow performs:

1. Building and pushing the controller image:
   - Builds the image for multiple platforms (linux/amd64, linux/arm64)
   - Tags the image with the specified version
   - Pushes to GHCR

2. Publishing the Helm charts:
   - Packages the Helm charts with the updated version
   - Pushes the charts to GHCR as OCI artifacts

## Verification

After the workflows complete:

1. Verify that the controller image is published to GHCR:
   - `ghcr.io/{org}/gha-runner-scale-set-controller:{version}`
   - `ghcr.io/{org}/gha-runner-scale-set-controller:{version}-{short_sha}`

2. Verify that the Helm charts are published to GHCR:
   - `ghcr.io/{org}/actions-runner-controller-charts/gha-runner-scale-set-controller:{version}`
   - `ghcr.io/{org}/actions-runner-controller-charts/gha-runner-scale-set:{version}`

## Complete Release Process

For a complete release of GHA runner components:

1. Prepare and merge a release PR to update the chart versions
2. Follow the steps above to publish the Helm charts
3. Create a release tag matching the version (format: `gha-runner-scale-set-{version}`)

## Troubleshooting

If you encounter issues during the publishing process:

- Check the workflow run logs for errors
- Ensure you have the necessary permissions
- Verify that the chart versions are correctly updated
- Check that the controller image is built and pushed successfully

If the publish workflow fails after the PR is merged, you can manually trigger it with the correct parameters.

## References

- [Workflow file: gha-bump-and-publish-charts.yaml](https://github.com/actions/actions-runner-controller/blob/master/.github/workflows/gha-bump-and-publish-charts.yaml)
- [Workflow file: gha-publish-chart.yaml](https://github.com/actions/actions-runner-controller/blob/master/.github/workflows/gha-publish-chart.yaml)
- [Contributing Guide](https://github.com/actions/actions-runner-controller/blob/master/CONTRIBUTING.md#release-gha-runner-scale-set-controller-image-and-helm-charts)