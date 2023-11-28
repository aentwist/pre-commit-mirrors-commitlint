# commitlint mirror

Mirror of [commitlint](https://github.com/conventional-changelog/commitlint) for [pre-commit](https://github.com/pre-commit/pre-commit)

## Using commitlint with pre-commit

### Derivation

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

### Result

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

## CI

The main hook checks a commit message at the time it is made. To run on a range of commits after they have been made, we need a separate hook.

This hook is `commitlint-all`. It runs commitlint on all commits. Note that it cannot run only on branch commits due to pre-commit limitations. To explore other options see [the discussion](https://github.com/aentwist/pre-commit-mirrors-commitlint/discussions/1).

### Derivation

Adding the `commitlint-all` hook, we have

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
      - id: commitlint-all
        stages: [manual]
        additional_dependencies:
          - commitlint@18.2.0
          - "@commitlint/config-conventional@18.1.0"
```

We can [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) this out with a [YAML anchor](https://support.atlassian.com/bitbucket-cloud/docs/yaml-anchors/):

### Result

```yaml
default_install_hook_types: [pre-commit, commit-msg]
default_stages: [pre-commit]
repos:
  - repo: https://github.com/aentwist/pre-commit-mirrors-commitlint
    rev: v18.2.0
    hooks:
      - id: commitlint
        stages: [commit-msg]
        additional_dependencies: &commitlint-additional-dependencies
          - commitlint@18.2.0
          - "@commitlint/config-conventional@18.1.0"
      - id: commitlint-all
        stages: [manual]
        additional_dependencies: *commitlint-additional-dependencies
```

Run the hook manually with `pre-commit run --hook-stage=manual commitlint-all`.
