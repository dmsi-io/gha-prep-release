# GHA Prep Release

[![actions-workflow-main][actions-workflow-main-badge]][actions-workflow-main]
[![release][release-badge]][release]

This GitHub Action is a composite action that can handle back merging a released merge back into the default branch as well as creating an auto-release branch with the incremented semver tag.

## Inputs

| NAME                  | DESCRIPTION                                                      | TYPE     | REQUIRED | DEFAULT                                                                                               |
| --------------------- | ---------------------------------------------------------------- | -------- | -------- | ----------------------------------------------------------------------------------------------------- |
| `GITHUB_TOKEN`        | GitHub Action Token or PAT.                                      | `string` | `true`   | `''`                                                                                                  |
| `skip_create_branch`  | Flag to skip prep auto branch creation.                          | `bool`   | `false`  | `false`                                                                                               |
| `skip_back_merge`     | Flag to skip the back merge step.                                | `bool`   | `false`  | `false`                                                                                               |
| `back_merge_branch`   | Branch to back merge into.                                       | `string` | `false`  | `github.event.repository.default_branch`                                                              |
| `semver_tag`          | Semver Override Tag. This value will always be used if provided. | `string` | `false`  | `null`                                                                                                |
| `patch_pattern`       | Patch keyword to search for in determining semver increment.     | `string` | `false`  | `[PATCH]`                                                                                             |
| `minor_pattern`       | Minor keyword to search for in determining semver increment.     | `string` | `false`  | `[MINOR]`                                                                                             |
| `major_pattern`       | Major keyword to search for in determining semver increment.     | `string` | `false`  | `[MAJOR]`                                                                                             |
| `initial_version`     | The initial version.                                             | `string` | `false`  | `v0.0.0`                                                                                              |
| `release_prefix`      | Prefix for automation release branch.                            | `string` | `false`  | `auto-release`                                                                                        |
| `commit-message`      | The commit message to use for the back merge.                    | `string` | `false`  | `Merge branch '${{ inputs.back_merge_branch }}' into '${{ github.event.repository.default_branch }}'` |
| `commit-author-name`  | The author name to use for the back merge.                       | `string` | `false`  | `github-actions`                                                                                      |
| `commit-author-email` | The email to use for the back merge.                             | `string` | `false`  | `github-actions@github.com`                                                                           |

If both `semver_tag` is not provided and no semver increment pattern is found in the head commit, then an auto release branch will not be created.

The general usage of this action is with the [git flow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow#:~:text=What%20is%20Gitflow%3F,branches%20and%20multiple%20primary%20branches.&text=Under%20this%20model%2C%20developers%20create,until%20the%20feature%20is%20complete.) branching pattern in which a `main` and `develop` branch are used. The `main` branch houses released, production ready code and `develop` houses currently in-development code.

When it's time to release, a `release/*` branch will be created from `develop` with a Pull Request to merge into `main`. The Pull Request title should include one of the semver increment patterns (`[MAJOR]`, `[MINOR]`, `[PATCH]`). Upon merging the PR into `main`, this action would be triggered to first automatically back merge `main` into `develop`. If one of the semver increment patterns are included or the `semver_tag` is provided as an input, then this action will also create a new branch from `main` labeled `auto-release/<semver-tag>`. The author of this branch will match the owner of the provided `GITHUB_TOKEN`.

It is then recommended to perform actual release automation on the created `auto-release/<semver-tag>` branch with a check in place to make sure the branch author matches the one set here.

## Outputs

| NAME    | DESCRIPTION                                                                                                                                         | TYPE     |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| `tag`   | The tag value used to create prep release branch. If `semver-tag` is not provided and no semver increment pattern is found, this value will be `''` | `string` |
| `level` | The semver increment level found from commit message {major, minor, patch}                                                                          | `string` |

## Example

```yaml
name: Prepare Auto Release Branch

on:
  push:
    branches:
      - main

jobs:
  prep-release:
    runs-on: ubuntu-latest
    name: Test gha-prep-release
    steps:
      - uses: alehechka/gha-prep-release@main
        id: commit-tag
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Print Tags
        run: |
          echo "Commit Tag: ${{ steps.commit-tag.outputs.tag }}"
```

For a further practical example, see [.github/workflows/release.yml](.github/workflows/main.yml).

<!-- badge links -->

[actions-workflow-main]: https://github.com/alehechka/gha-prep-release/actions/query?workflow%3ATest%20gha-prep-release
[actions-workflow-main-badge]: https://img.shields.io/github/workflow/status/alehechka/gha-prep-release/Test%20gha-prep-release?label=Test%20gha-prep-release&style=for-the-badge&logo=github
[release]: https://github.com/alehechka/gha-prep-release/releases
[release-badge]: https://img.shields.io/github/v/release/alehechka/gha-prep-release?style=for-the-badge&logo=github
