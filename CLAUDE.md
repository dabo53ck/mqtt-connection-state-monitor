# Repository guide

Home Assistant **blueprint** — a single-file automation blueprint plus its
changelog, tests and CI. Not a Python package.

## Layout

- `mqtt-connection-state-monitor.yaml` — the blueprint (all logic lives here)
- `CHANGELOG.md` — one section per release; `README.md` only links to it
- `test/automations.yaml` — CI fixture: drives the blueprint with every input set
- `.github/workflows/validate.yml` — yamllint + HA `check_config` on push / PR
- `test/README.md` — local validation + the `dev` → HA-re-import loop

## Conventions

- Work happens on a `dev` branch that is created fresh from `main` for each batch of
  changes → PR → `main` (squash-merge). `dev` only lives until the release: delete it
  after the merge; the next batch starts a new `dev` from `main`. Conventional Commits.
- `main` is what users import (the blueprint's `source_url` and the README badge point
  at it): a change to `mqtt-connection-state-monitor.yaml` is merged **together with its
  release** — version badge + CHANGELOG date in the PR, tag `vX.Y.Z` right after the
  merge. Docs- or CI-only changes need no release.
- One `CHANGELOG.md` section per release; release = git tag `vX.Y.Z` + GitHub
  release. An unreleased section is still a draft — keep editing it until tagged.
- The `## [vX.Y.Z] - DATE` line uses the release / commit date, not the day the
  work was drafted.
- Bump the version badge in the blueprint `description:` **and** `README.md`.
- Blueprint edits must keep CI green (`check_config` against `test/automations.yaml`);
  a new input also needs adding to that fixture.

## Commit messages

Do **not** add AI / assistant attribution to commit messages or PR descriptions:
no `Co-Authored-By:` line naming an AI, no `Claude-Session:` line, no
"Generated with …" footer. Commits are authored solely by the repository owner.

A `commit-msg` hook enforces this — enable it once per clone:

```sh
git config core.hooksPath .githooks
```
