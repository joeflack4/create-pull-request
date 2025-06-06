# <img width="24" height="24" src="docs/assets/logo.svg"> Create Pull Request
[![CI](https://github.com/peter-evans/create-pull-request/workflows/CI/badge.svg)](https://github.com/peter-evans/create-pull-request/actions?query=workflow%3ACI)
[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Create%20Pull%20Request-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=github)](https://github.com/marketplace/actions/create-pull-request)

A GitHub action to create a pull request for changes to your repository in the actions workspace.

Changes to a repository in the Actions workspace persist between steps in a workflow.
This action is designed to be used in conjunction with other steps that modify or add files to your repository.
The changes will be automatically committed to a new branch and a pull request created.

Create Pull Request action will:

1. Check for repository changes in the Actions workspace. This includes:
   - untracked (new) files
   - tracked (modified) files
   - commits made during the workflow that have not been pushed
2. Commit all changes to a new branch, or update an existing pull request branch.
3. Create or update a pull request to merge the branch into the base&mdash;the branch checked out in the workflow.

## Documentation

- [Concepts, guidelines and advanced usage](docs/concepts-guidelines.md)
- [Examples](docs/examples.md)
- [Updating to v7](docs/updating.md)
- [Common issues](docs/common-issues.md)

## Usage

```yml
      - uses: actions/checkout@v4

      # Make changes to pull request here

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v7
```

You can also pin to a [specific release](https://github.com/peter-evans/create-pull-request/releases) version in the format `@v7.x.x`

### Workflow permissions

For this action to work you must explicitly allow GitHub Actions to create pull requests.
This setting can be found in a repository's settings under Actions > General > Workflow permissions.

For repositories belonging to an organization, this setting can be managed by admins in organization settings under Actions > General > Workflow permissions.

### Action inputs

All inputs are **optional**. If not set, sensible defaults will be used.

| Name | Description | Default |
| --- | --- | --- |
| `token` | The token that the action will use to create and update the pull request. See [token](#token). | `GITHUB_TOKEN` |
| `branch-token` | The token that the action will use to create and update the branch. See [branch-token](#branch-token). | Defaults to the value of `token` |
| `path` | Relative path under `GITHUB_WORKSPACE` to the repository. | `GITHUB_WORKSPACE` |
| `add-paths` | A comma or newline-separated list of file paths to commit. Paths should follow git's [pathspec](https://git-scm.com/docs/gitglossary#Documentation/gitglossary.txt-aiddefpathspecapathspec) syntax. | If no paths are specified, all new and modified files are added. See [Add specific paths](#add-specific-paths). |
| `commit-message` | The message to use when committing changes. See [commit-message](#commit-message). | `[create-pull-request] automated change` |
| `committer` | The committer name and email address in the format `Display Name <email@address.com>`. Defaults to the GitHub Actions bot user on github.com. | `github-actions[bot] <41898282+github-actions[bot]@users.noreply.github.com>` |
| `author` | The author name and email address in the format `Display Name <email@address.com>`. Defaults to the user who triggered the workflow run. | `${{ github.actor }} <${{ github.actor_id }}+${{ github.actor }}@users.noreply.github.com>` |
| `signoff` | Add [`Signed-off-by`](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---signoff) line by the committer at the end of the commit log message. | `false` |
| `branch` | The pull request branch name. | `create-pull-request/patch` |
| `delete-branch` | Delete the `branch` if it doesn't have an active pull request associated with it. See [delete-branch](#delete-branch). | `false` |
| `branch-suffix` | The branch suffix type when using the alternative branching strategy. Valid values are `random`, `timestamp` and `short-commit-hash`. See [Alternative strategy](#alternative-strategy---always-create-a-new-pull-request-branch) for details. | |
| `base` | Sets the pull request base branch. | Defaults to the branch checked out in the workflow. |
| `push-to-fork` | A fork of the checked-out parent repository to which the pull request branch will be pushed. e.g. `owner/repo-fork`. The pull request will be created to merge the fork's branch into the parent's base. See [push pull request branches to a fork](docs/concepts-guidelines.md#push-pull-request-branches-to-a-fork) for details. | |
| `sign-commits` | Sign commits as `github-actions[bot]` when using `GITHUB_TOKEN`, or your own bot when using [GitHub App tokens](docs/concepts-guidelines.md#authenticating-with-github-app-generated-tokens). See [commit signing](docs/concepts-guidelines.md#commit-signature-verification-for-bots) for details.  | `false` |
| `title` | The title of the pull request. | `Changes by create-pull-request action` |
| `body` | The body of the pull request. | `Automated changes by [create-pull-request](https://github.com/peter-evans/create-pull-request) GitHub action` |
| `body-path` | The path to a file containing the pull request body. Takes precedence over `body`. | |
| `labels` | A comma or newline-separated list of labels. | |
| `assignees` | A comma or newline-separated list of assignees (GitHub usernames). | |
| `reviewers` | A comma or newline-separated list of reviewers (GitHub usernames) to request a review from. | |
| `team-reviewers` | A comma or newline-separated list of GitHub teams to request a review from. Note that a `repo` scoped [PAT](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token), or equivalent [GitHub App permissions](docs/concepts-guidelines.md#authenticating-with-github-app-generated-tokens), are required. | |
| `milestone` | The number of the milestone to associate this pull request with. | |
| `draft` | Create a [draft pull request](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests#draft-pull-requests). Valid values are `true` (only on create), `always-true` (on create and update), and `false`.  | `false` |
| `maintainer-can-modify` | Indicates whether [maintainers can modify](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/allowing-changes-to-a-pull-request-branch-created-from-a-fork) the pull request. | `true` |

#### token

The token input defaults to the repository's `GITHUB_TOKEN`.

> [!IMPORTANT]  
> - If you want pull requests created by this action to trigger an `on: push` or `on: pull_request` workflow then you cannot use the default `GITHUB_TOKEN`. See the [documentation here](docs/concepts-guidelines.md#triggering-further-workflow-runs) for further details.
> - If using the repository's `GITHUB_TOKEN` and your repository was created after 2nd February 2023, the [default permission is read-only](https://github.blog/changelog/2023-02-02-github-actions-updating-the-default-github_token-permissions-to-read-only/). Elevate the [permissions](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token#defining-access-for-the-github_token-permissions) in your workflow.
>   ```yml
>   permissions:
>     contents: write
>     pull-requests: write
>   ```

Other token options:
- Classic [Personal Access Token (PAT)](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) with `repo` scope.
- Fine-grained [Personal Access Token (PAT)](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) with `contents: write` and `pull-requests: write` scopes.
- [GitHub App tokens](docs/concepts-guidelines.md#authenticating-with-github-app-generated-tokens) with `contents: write` and `pull-requests: write` scopes.

> [!TIP]  
> If pull requests could contain changes to Actions workflows you may also need the `workflows` scope.

#### branch-token

The action first creates a branch, and then creates a pull request for the branch.
For some rare use cases it can be useful, or even necessary, to use different tokens for these operations.
It is not advisable to use this input unless you know you need to.

#### commit-message

In addition to a message, the `commit-message` input can also be used to populate the commit description. Leave a single blank line between the message and description.

```yml
          commit-message: |
            the first line is the commit message

            the commit description starts
            after a blank line and can be
            multiple lines
```

#### delete-branch

The `delete-branch` feature doesn't delete branches immediately on merge. (It can't do that because it would require the merge to somehow trigger the action.)
The intention of the feature is that when the action next runs it will delete the `branch` if there is no diff.

Enabling this feature leads to the following behaviour:
1. If a pull request was merged and the branch is left undeleted, when the action next runs it will delete the branch if there is no further diff.
2. If a pull request is open, but there is now no longer a diff and the PR is unnecessary, the action will delete the branch causing the PR to close.

If you want branches to be deleted immediately on merge then you should use GitHub's `Automatically delete head branches` feature in your repository settings.

#### Proxy support

For self-hosted runners behind a corporate proxy set the `https_proxy` environment variable.
```yml
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v7
        env:
          https_proxy: http://<proxy_address>:<port>
```

### Action outputs

The following outputs can be used by subsequent workflow steps.

- `pull-request-number` - The pull request number.
- `pull-request-url` - The URL of the pull request.
- `pull-request-operation` - The pull request operation performed by the action, `created`, `updated`, `closed` or `none`.
- `pull-request-head-sha` - The commit SHA of the pull request branch.
- `pull-request-branch` - The branch name of the pull request.
- `pull-request-commits-verified` - Whether GitHub considers the signature of the branch's commits to be verified; `true` or `false`.

Step outputs can be accessed as in the following example.
Note that in order to read the step outputs the action step must have an id.

```yml
      - name: Create Pull Request
        id: cpr
        uses: peter-evans/create-pull-request@v7
      - name: Check outputs
        if: ${{ steps.cpr.outputs.pull-request-number }}
        run: |
          echo "Pull Request Number - ${{ steps.cpr.outputs.pull-request-number }}"
          echo "Pull Request URL - ${{ steps.cpr.outputs.pull-request-url }}"
```

### Action behaviour

The default behaviour of the action is to create a pull request that will be continually updated with new changes until it is merged or closed.
Changes are committed and pushed to a fixed-name branch, the name of which can be configured with the `branch` input.
Any subsequent changes will be committed to the *same* branch and reflected in the open pull request.

How the action behaves:

- If there are changes (i.e. a diff exists with the checked-out base branch), the changes will be pushed to a new `branch` and a pull request created.
- If there are no changes (i.e. no diff exists with the checked-out base branch), no pull request will be created and the action exits silently.
- If a pull request already exists it will be updated if necessary. Local changes in the Actions workspace, or changes on the base branch, can cause an update. If no update is required the action exits silently.
- If a pull request exists and new changes on the base branch make the pull request unnecessary (i.e. there is no longer a diff between the pull request branch and the base), the pull request is automatically closed. Additionally, if [`delete-branch`](#delete-branch) is set to `true` the `branch` will be deleted.

For further details about how the action works and usage guidelines, see [Concepts, guidelines and advanced usage](docs/concepts-guidelines.md).

#### Alternative strategy - Always create a new pull request branch

For some use cases it may be desirable to always create a new unique branch each time there are changes to be committed.
This strategy is *not recommended* because if not used carefully it could result in multiple pull requests being created unnecessarily. If in doubt, use the [default strategy](#action-behaviour) of creating an updating a fixed-name branch.

To use this strategy, set input `branch-suffix` with one of the following options.

- `random` - Commits will be made to a branch suffixed with a random alpha-numeric string. e.g. `create-pull-request/patch-6qj97jr`, `create-pull-request/patch-5jrjhvd`

- `timestamp` - Commits will be made to a branch suffixed by a timestamp. e.g. `create-pull-request/patch-1569322532`, `create-pull-request/patch-1569322552`

- `short-commit-hash` - Commits will be made to a branch suffixed with the short SHA1 commit hash. e.g. `create-pull-request/patch-fcdfb59`, `create-pull-request/patch-394710b`

### Controlling committed files

The action defaults to adding all new and modified files.
If there are files that should not be included in the pull request, you can use the following methods to control the committed content.

#### Remove files

The most straightforward way to handle unwanted files is simply to remove them in a step before the action runs.

```yml
      - run: |
          rm -rf temp-dir
          rm temp-file.txt
```

#### Ignore files

If there are files or directories you want to ignore you can simply add them to a `.gitignore` file at the root of your repository. The action will respect this file.

#### Add specific paths

You can control which files are committed with the `add-paths` input.
Paths should follow git's [pathspec](https://git-scm.com/docs/gitglossary#Documentation/gitglossary.txt-aiddefpathspecapathspec) syntax.
File changes that do not match one of the paths will be stashed and restored after the action has completed.

```yml
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v7
        with:
          add-paths: |
            *.java
            docs/*.md
```

#### Create your own commits

As well as relying on the action to handle uncommitted changes, you can additionally make your own commits before the action runs.
Note that the repository must be checked out on a branch with a remote, it won't work for [events which checkout a commit](docs/concepts-guidelines.md#events-which-checkout-a-commit).

```yml
    steps:
      - uses: actions/checkout@v4
      - name: Create commits
        run: |
          git config user.name 'Peter Evans'
          git config user.email 'peter-evans@users.noreply.github.com'
          date +%s > report.txt
          git commit -am "Modify tracked file during workflow"
          date +%s > new-report.txt
          git add -A
          git commit -m "Add untracked file during workflow"
      - name: Uncommitted change
        run: date +%s > report.txt
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v7
```

### Auto-merge

Auto-merge can be enabled on a pull request allowing it to be automatically merged once requirements have been satisfied.
See [enable-pull-request-automerge](https://github.com/peter-evans/enable-pull-request-automerge) action for usage details.

## Reference Example

The following workflow sets many of the action's inputs for reference purposes.
Check the [defaults](#action-inputs) to avoid setting inputs unnecessarily.

See [examples](docs/examples.md) for more realistic use cases.

```yml
jobs:
  createPullRequest:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Make changes to pull request
        run: date +%s > report.txt

      - name: Create Pull Request
        id: cpr
        uses: peter-evans/create-pull-request@v7
        with:
          token: ${{ secrets.PAT }}
          commit-message: Update report
          committer: github-actions[bot] <41898282+github-actions[bot]@users.noreply.github.com>
          author: ${{ github.actor }} <${{ github.actor_id }}+${{ github.actor }}@users.noreply.github.com>
          signoff: false
          branch: example-patches
          delete-branch: true
          title: '[Example] Update report'
          body: |
            Update report
            - Updated with *today's* date
            - Auto-generated by [create-pull-request][1]

            [1]: https://github.com/peter-evans/create-pull-request
          labels: |
            report
            automated pr
          assignees: peter-evans
          reviewers: peter-evans
          team-reviewers: |
            developers
            qa-team
          milestone: 1
          draft: false
```

An example based on the above reference configuration creates pull requests that look like this:

![Pull Request Example](docs/assets/pull-request-example.png)

## License

[MIT](LICENSE)
