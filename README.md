# commitlint mirror

Mirror of [commitlint](https://github.com/conventional-changelog/commitlint) for [pre-commit](https://github.com/pre-commit/pre-commit)

## Using commitlint with pre-commit

Add this to your `.pre-commit-config.yaml`:

```yaml
- repo: https://github.com/aentwist/pre-commit-mirrors-commitlint
  rev: ""  # Use the sha / tag you want to point at
  hooks:
    - id: commitlint
```

Since this is a commit-msg hook (not a typical, pre-commit hook), it needs additional configuration. By default, pre-commit runs `hooks` in all `stages` but only installs one stage, pre-commit. Adding a commit-msg hook we also need to install the commit-msg stage, but then since hooks are run in all stages by default, other hooks in the file will run multiple times. To prevent this, we can flip this logic: run hooks in specific stages and install hooks for all stages (or just the stages we need).

```yaml
# Install hooks for all stages instead of just for the pre-commit stage.
default_install_hook_types: [pre-commit, commit-msg]
# Limit hooks to running in the pre-commit stage unless specified otherwise.
default_stages: [pre-commit]
repos:
  - repo: https://github.com/aentwist/pre-commit-mirrors-commitlint
    rev: ""  # Use the sha / tag you want to point at
    hooks:
      - id: commitlint
        # commitlint is a commit-msg hook
        stages: [commit-msg]
```

See [Configuring hooks to run at certain stages](https://pre-commit.com/#confining-hooks-to-run-at-certain-stages).

Finally, when using plugins with commitlint you'll need to declare them under `additional_dependencies`. Since this overrides the `additional_dependencies` in this hook, you will also need to redeclare the `commitlint` dependency. For example:

```yaml
- repo: https://github.com/aentwist/pre-commit-mirrors-commitlint
  rev: v18.2.0
  hooks:
    - id: commitlint
      additional_dependencies:
        - commitlint@18.2.0
        - "@commitlint/config-conventional@18.1.0"
```

Combining these ideas together, we have a complete example:

```yaml
default_install_hook_types: [pre-commit, commit-msg]
default_stages: [pre-commit]
repos:
  - repo: https://github.com/aentwist/pre-commit-mirrors-commitlint
    rev: v18.2.0
    hooks:
      - id: commitlint
        stages: [commit-msg]
        additional_dependencies:
          - commitlint@18.2.0
          - "@commitlint/config-conventional@18.1.0"
```

## A note on CI

Due to pre-commit limitations, there is currently no good way to run this thing in CI. It should be run against *new* commits using `commitlint --from=my-ref`, not *all* commits. But we would need to specify the "from" ref. pre-commit does not support passing CLI arguments to hook commands and `--from-ref` and `--to-ref` [do not support running hooks that do not take files](https://github.com/pre-commit/pre-commit-mirror-maker/issues/199#issuecomment-1799386682) (anything other than `commitlint --edit`). We would really need support for hooks that do not take files.

Here are some opinionated ideas that may or may not work:

1. Best right now: Forget pre-commit for this in CI. Get a LTS node environment, parse the tool versions out of the pre-commit config to avoid differences between local and CI runs, install the tools, and run commitlint directly with your "from" ref.
1. Theoretical best: `--from-ref` works!!!
1. Actual theoretical best: Allow commitlint to run on a `git log` file either by implementing it in commitlint (unlikely) or writing a wrapper program that calls commitlint many times and aggregates the results. Then call pre-commit once with such a file.
1. pre-commit suggestion: Convert `git log` output to files, call pre-commit once for each file, and aggregate the results. See https://github.com/pre-commit/pre-commit/issues/3056#issuecomment-1799524821.
1. Easiest: Use pre-commit to run commitlint on all commits using `commitlint --from`. The pre-commit hook for this is not yet implemented here.
1. Maybe: Use an environment variable like so `FROM=my-ref pre-commit run [...]` then somehow get it in your hook `commitlint --from=$FROM`.
1. Very unlikely: Maybe come up with a [fancy git hack](https://github.com/newren/git-filter-repo) to limit the refs available to commitlint, then run a pre-commit hook that runs commitlint on "all" commits.

For completeness, here are some ideas for running commitlint against a range of commits since I find the [commitlint CI Setup Guide](https://commitlint.js.org/#/guides-ci-setup) to be lacking in this regard:

###### "From" refs

- Local: default branch / `main`
- GitHub Actions: maybe `$GITHUB_BASE_SHA`
- GitLab CI: `$CI_MERGE_REQUEST_DIFF_BASE_SHA`
