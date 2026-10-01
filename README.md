# IdentyClaw A2A overlay for Hermes

Hermes **platform plugin** (`identyclaw-a2a`) that overlays bundled A2A with
Passport JWT inbound auth (`token_id` identity), peer `/api/login` +
`/api/login/timestamp`, and outbound `a2a_call` that mints per-peer JWTs via the
**auth sidecar**.

Requires the host auth package ([hermes-identyclaw-auth](https://github.com/discernible-io/hermes-identyclaw-auth))
— CLI + sidecar — **not** a Hermes plugin.

Follows the stock [Hermes Plugins](https://hermes-agent.nousresearch.com/docs/user-guide/features/plugins/) flow:
`hermes plugins install owner/repo` → enable (opt-in) → capability consent.

## Naming (important)

| Surface | Value |
|---------|-------|
| GitHub repo | `discernible-io/hermes-identyclaw-a2a` |
| Plugin id / install dir | `identyclaw-a2a` |
| Bundled Hermes A2A | key `platforms/a2a`, yaml name `a2a-platform` |

Do **not** reuse yaml name `a2a-platform`. `hermes plugins enable a2a-platform`
resolves to the **bundled** adapter (`platforms/a2a`) first.

Catalog submission is optional. `owner/repo` installs are the standard
unreviewed Git path (same as any third-party plugin).

## Install (default Hermes UX)

Sidecar must be healthy first (`curl -fsS http://127.0.0.1:9910/health`).

Interactive (matches the Plugins guide — install, then answer the prompts):

```bash
export HERMES_HOME="${HERMES_HOME:-$HOME/.hermes}"

hermes plugins install discernible-io/hermes-identyclaw-a2a
# Enable 'identyclaw-a2a' now? → y
# Grant tools.override? → y

# Bundled A2A auto-loads; turn it off so this overlay owns the tools:
hermes plugins disable platforms/a2a
```

Confirm:

```bash
hermes plugins list --plain --no-bundled
hermes plugins capabilities identyclaw-a2a
```

Restart the gateway if it is already running (`hermes gateway restart`).

### Scripted / non-interactive

Same CLI flags the guide documents (`--no-enable` / `--enable`). Non-interactive
sessions fail closed on capability consent — pass `--allow-tool-override` on enable:

```bash
hermes plugins install discernible-io/hermes-identyclaw-a2a --no-enable
hermes plugins disable platforms/a2a
hermes plugins enable identyclaw-a2a --allow-tool-override
```

Or one-shot enable after install (still grant override separately if there was no TTY):

```bash
hermes plugins install discernible-io/hermes-identyclaw-a2a --enable
hermes plugins disable platforms/a2a
hermes plugins enable identyclaw-a2a --allow-tool-override
```

### Config equivalent

```yaml
plugins:
  enabled:
    - identyclaw-a2a
  disabled:
    - platforms/a2a
  entries:
    identyclaw-a2a:
      enabled: true
      allow_tool_override: true
      # or: granted_capabilities: [tools.override]
```

`optional_env` from `plugin.yaml` is prompted on `hermes plugins install` when unset.

### Optional: true path-key override

To shadow bundled A2A by the same discovery key (`platforms/a2a`), keep a copy
under `$HERMES_HOME/plugins/platforms/a2a/` (directory name must be `a2a`) and
`hermes plugins adopt platforms/a2a`. Prefer the unique `identyclaw-a2a` id +
disable bundled for stock installs.

## Full playbook

```bash
bash "$HERMES_HOME/hermes-identyclaw-auth/scripts/install-stock-hermes.sh" \
  --a2a-public-url "https://YOUR.PUBLIC.HOST"
```

## Env

| Variable | Role |
|----------|------|
| `NEAR_CREDENTIALS_FILE_PATH` | JWT `aud` via RoditClient (sidecar) |
| `IDENTYCLAW_AUTH_PORT` | Sidecar port (default `9910`) |
| `A2A_PUBLIC_URL` | Public HTTPS base (Agent Card / discovery) |
| `A2A_PORT` / `A2A_HOST` | Inbound listen |
| `IDENTYCLAW_JWT_AUDIENCE` | Optional fallback only |
