# Dev / validation workflow

Two layers, so changes don't have to be hand-copied onto a live HA to test them.

## 1. Structural validation (automatic, in CI)

`.github/workflows/validate.yml` runs on every push to `main` / `dev` and on PRs:

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

No file copying:

1. One-time: import the blueprint into HA from the **`dev`** branch:
   `https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/dabo53ck/mqtt-connection-state-monitor/dev/mqtt-connection-state-monitor.yaml`
2. Per iteration: `git push origin dev` → HA **Settings → Automations → Blueprints
   → ⋮ → Re-import blueprint** → **Reload Automations** → inspect the trace.
3. Existing automations keep their stored inputs; new inputs take their default;
   removed inputs are ignored.

`raw.githubusercontent.com` caches ~5 min per URL. To skip the wait, re-run the
import link with a cache-buster (`…/mqtt-connection-state-monitor.yaml?v=<ts>`);
it overwrites the same local blueprint file.

When happy: merge `dev` → `main`, bump the version badge + CHANGELOG.
