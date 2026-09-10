# Repository guide

Home Assistant **blueprint** — a single-file automation blueprint plus its
changelog, tests and CI. Not a Python package.

## Layout

- `mqtt-connection-state-monitor.yaml` — the blueprint (all logic lives here)
- `CHANGELOG.md` — one section per release; `README.md` only links to it
- `test/automations.yaml` — CI fixture: drives the blueprint with every input set
- `.github/workflows/validate.yml` — yamllint + HA `check_config` on push / PR
- `test/README.md` — local validation + the dev → HA-re-import loop

## Conventions

- Branch `dev` → PR → `main`. Conventional Commits.
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
