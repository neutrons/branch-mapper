# GitHub Action: neutrons/branch-mapper
![test](https://github.com/neutrons/branch-mapper/workflows/test/badge.svg)

This GitHub Action that maps a `github.ref` to a friendlier name.
`branch-mapper` main use case is in workflows for triggering deployments to map the `github.ref` to a deployment environment.
The output `name` is a simple joining of `prefix` and the calculated `suffix`.

The mapping of `github.ref` is
- `refs/tags/v*` to the value of the input `suffix-release` with an example tag being v1.2.3
- `refs/tags/v*rc*` to the value of the input `suffix-release-candidate` with an example tag being v1.2.3rc1.
- everything else goes to the value of the input `suffix-default`

### Example

This can be used in a workflow with `uses: neutrons/branch-mapper@main`.
An example with triggering a deployment through githlab is
```yaml
jobs:
  deploy-software:
    runs-on: ubuntu-latest
    steps:
    - name: Determine deploy environment
      uses: neutrons/branch-mapper@v1
      id: deploy-name
      with:
        prefix: super-awesome-software
    - name: Trigger deployment
      uses: eic/trigger-gitlab-ci@v2
      id: trigger
      with:
        project_id: 12345
        token: ${{ secrets.TOKEN }}
        variables: |
          CONDA_ENV="${{ steps.deploy-name.outputs.name }}"
    - name: Annotate commit
      uses: peter-evans/commit-comment@v2
      with:
        body: |
          GitLab pipeline for ${{ steps.deployname.outputs.name }} has been submitted for this commit: ${{ steps.trigger.outputs.web_url }}
```
The other actions are for [triggering a gitlab workflow](https://github.com/eic/trigger-gitlab-ci) and [annotating a commit with information](https://github.com/peter-evans/commit-comment).

### Developing locally

Install the wonderful [act](https://github.com/nektos/act) and run the github action using it.
