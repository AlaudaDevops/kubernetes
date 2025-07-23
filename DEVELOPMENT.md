# Kubectl alauda Branch Development Guide

## Background

Previously, kubectl, as a general-purpose CLI, was used across multiple plugins, each needing to fix vulnerabilities in kubectl itself.

To avoid duplicate work, we forked the current repository based on [kubectl](https://github.com/kubernetes/kubernetes.git) and maintain it through the `alauda-vx.xx.xx` branches.

We use [renovate](https://gitlab-ce.alauda.cn/devops/tech-research/renovate/-/blob/main/docs/quick-start/0002-quick-start.md) to automatically fix vulnerabilities in corresponding versions.

## Repository Structure

Based on the original code, the following content has been added:

- [alauda-auto-tag.yaml](./.github/workflows/alauda-auto-tag.yaml): Automatically tags and triggers goreleaser when a PR is merged into the `alauda-vx.xx.xx` branch.
- [release-alauda.yaml](./.github/workflows/release-alauda.yaml): Supports triggering goreleaser via tag updates or manual actions (note that this pipeline won't be triggered if a tag is created automatically within an action due to design limitations).
- [reusable-release-alauda.yaml](./.github/workflows/reusable-release-alauda.yaml): Executes goreleaser to create a release.
- [scan-alauda.yaml](.github/workflows/scan-alauda.yaml): Runs trivy vulnerability scans ([rootfs](file://D:\code\alauda-kubernetes-2\cluster\gce\gci\mounter\mounter.go#L30-L30) scans Go binaries).
- [.goreleaser-alauda.yml](.goreleaser-alauda.yml): Configuration file for releasing alauda versions.

## Special Modifications

1. The official project does not provide GitHub Actions for automated testing, so we wrote our own [test-alauda.yaml](.github/workflows/test-alauda.yaml) to execute `go test`.

## Pipelines

### Triggered on PR Submission

- [test-alauda.yaml](.github/workflows/test-alauda.yaml): Executes the official tests with `make test`.

### Triggered on Merge to alauda-vx.xx.xx Branch

- [alauda-auto-tag.yaml](.github/workflows/alauda-auto-tag.yaml): Automatically tags and triggers goreleaser.
- [reusable-release-alauda.yaml](.github/workflows/reusable-release-alauda.yaml): Executes goreleaser to create a release (triggered by `alauda-auto-tag.yaml`).

### Scheduled or Manual Triggering

- [scan-alauda.yaml](.github/workflows/scan-alauda.yaml): Runs trivy vulnerability scans ([rootfs](file://D:\code\alauda-kubernetes-2\cluster\gce\gci\mounter\mounter.go#L30-L30) scans Go binaries).

### Others

Other officially maintained pipelines remain unchanged; some irrelevant pipelines have been disabled on the Actions page.

## Renovate Vulnerability Fixing Mechanism

The renovate configuration file is [renovate.json](https://github.com/AlaudaDevops/trivy/blob/main/renovate.json)

1. renovate detects vulnerabilities in a branch and submits a PR for fixes.
2. Tests are automatically executed upon PR submission.
3. After all tests pass, renovate automatically merges the PR.
4. Upon branch update, an action automatically tags the commit (e.g., v0.62.1-alauda-0, where both patch version and last digit increment).
5. goreleaser automatically publishes a release based on the tag.

## Maintenance Plan

When upgrading to a new version, follow these steps:

1. Create an alauda branch from the corresponding tag, e.g., tag `v0.62.1` corresponds to branch `alauda-v0.62.1`.
2. Cherry-pick previous alauda branch changes to the new branch and push.

Renovate automatic fixing mechanism:
1. After renovate submits a PR, pipelines will run automatically. If all tests pass, the PR will be merged automatically.
2. After merging into the `alauda-v0.62.1` branch, goreleaser will automatically create a `v0.62.2-alauda-0` release (note: not `v0.62.1-alauda-0`, because upgrading the version allows renovate to recognize it).
3. Other plugins configured with renovate will automatically fetch artifacts from the release according to their configurations.
