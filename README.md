# Flux Helm Preview

This GitHub Action calculates the diff of the generated YAML of a commit as it would be deployed to a cluster using the Flux GitOps toolkit. By previewing the changes that will be made to your cluster, you can ensure that your GitOps workflow is properly configured and avoid any unexpected disruptions to your system.

[![ci status](https://github.com/USERNAME/REPO/workflows/ci/badge.svg)](https://github.com/USERNAME/REPO/actions)

## Usage

To use this action, you will need to include it in a workflow file in your repository. Here is an example workflow that runs the action on every push to the `master` branch and posts the output as a comment on the associated pull request:

```
---
name: Flux Preview
on:
  pull_request:
    branches:
      - main
    paths:
      - "**.yaml"
env:
  DEFAULT_BRANCH: main
jobs:
  flux-preview:
    name: Flux Preview
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          path: pr
      - name: Checkout default branch
        uses: actions/checkout@v3
        with:
          ref: "${{ env.DEFAULT_BRANCH }}"
          path: default
      - name: Diff
        uses: tobiash/flux-helm-preview-action@main
        id: diff
        with:
          helm: "true"
          repo-a: default
          repo-b: pr
          kustomizations: |
            _preview/apps
            _preview/infra
            clusters/primary/flux-system
          filter: |
            filters:
              - kind: LabelRemover
                labels:
                  - chart
                  - app.kubernetes.io/version
                  - helm.sh/chart
                  - heritage
          write-markdown: diff.txt
      - uses: marocchino/sticky-pull-request-comment@v2
        with:
          path: diff.txt
```

## Output

The action will output the diff of the generated YAML for the current commit. This will include any changes to deployments, services, and other Kubernetes resources that will be made by Flux. If the diff is empty, it means that there are no changes to be made to the cluster.

