# Dev / validation workflow

Two layers, so changes don't have to be hand-copied onto a live HA to test them.

## 1. Structural validation (automatic, in CI)

`.github/workflows/validate.yml` runs on every push to `main` / `dev` and on every pull request:

- **yamllint** – indentation, tabs, duplicate keys.
- **HA `check_config`** – spins the official Home Assistant image, drops
  `test/automations.yaml` (which drives the blueprint with every input set) and
  runs `python -m homeassistant --script check_config`. Catches broken Jinja,
  bad selectors, unresolved `!input`, `min_version` issues.

Run the same check locally:

```bash
mkdir -p ha/blueprints/automation/mqtt
cp mqtt-connection-state-monitor.yaml ha/blueprints/automation/mqtt/
cp test/automations.yaml ha/automations.yaml
cat > ha/configuration.yaml <<'EOF'
homeassistant:
  name: CI
  time_zone: UTC
automation: !include automations.yaml
EOF
docker run --rm -v "$PWD/ha:/config" \
  ghcr.io/home-assistant/home-assistant:stable \
  python -m homeassistant --script check_config -c /config
```

CI does **not** validate behaviour – the real entities can't be mocked.

## 2. Behaviour validation (manual, via GitHub + Re-import)

No file copying. Work on the `dev` branch (created fresh from `main` for each batch
of changes):

1. One-time per batch: import the blueprint into HA from `dev`:
   `https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/dabo53ck/mqtt-connection-state-monitor/dev/mqtt-connection-state-monitor.yaml`
2. Per iteration: `git push origin dev` → HA **Settings → Automations →
   Blueprints → ⋮ → Re-import blueprint** → **Reload Automations** → inspect the
   trace.
3. Existing automations keep their stored inputs; new inputs take their default;
   removed inputs are ignored.

`raw.githubusercontent.com` caches ~5 min per URL. To skip the wait, re-run the
import link with a cache-buster (`…/mqtt-connection-state-monitor.yaml?v=<ts>`);
it overwrites the same local blueprint file.

**Important:** an import stamps the imported URL as the blueprint's `source_url`, so
after testing from `dev` "Re-import blueprint" keeps following `dev`. Once the PR is
merged and `dev` is deleted, re-import **once from `main`** so HA tracks `main` again
(otherwise the next re-import would 404):
`https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/dabo53ck/mqtt-connection-state-monitor/main/mqtt-connection-state-monitor.yaml`

When happy: open a PR `dev` → `main` with the version badge + CHANGELOG bumped,
squash-merge it, delete `dev`, tag `vX.Y.Z` and publish the release.
