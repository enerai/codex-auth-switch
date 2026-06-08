# Troubleshooting

Start with:

```bash
codex-auth-switch --json doctor
```

`doctor` returns a bounded JSON envelope with `structuredContent.result.issues`. Each issue has a severity, code, message, and path when a path is relevant.

## Common Doctor Issues

### `auth_missing`

`auth.json` does not exist.

Fix:

```bash
codex login
codex-auth-switch save <name>
```

### `config_not_file_backed`

Codex is not configured with file-backed auth.

Fix:

```bash
codex-auth-switch init --fix
```

If you know you want the tool to rewrite the config non-interactively:

```bash
codex-auth-switch init --fix --force
```

### `invalid_json`

An active auth file or saved snapshot is not valid JSON.

Fix:

- If the active `auth.json` is damaged, run `codex login` again.
- If a saved snapshot is damaged, remove it and save it again.

```bash
codex-auth-switch remove <name>
codex login
codex-auth-switch save <name>
```

### `symlink_refused`

The tool found a symlink where it expects a real file or directory.

Fix:

- Inspect the path.
- Replace the symlink with a real file or directory.
- Do not use symlinks for auth snapshots or switcher state.

### `bad_permissions`

A file or directory has weaker permissions than expected.

Expected modes:

```text
directories: 700
auth.json / *.auth.json / meta / current / pending-login: 600
```

Fix:

```bash
chmod 700 "$CODEX_AUTH_SWITCH_HOME"
chmod 700 "$CODEX_AUTH_SWITCH_HOME/accounts"
chmod 700 "$CODEX_AUTH_SWITCH_HOME/meta"
chmod 700 "$CODEX_AUTH_SWITCH_HOME/before-login"
chmod 600 "$CODEX_AUTH_SWITCH_HOME"/accounts/*.auth.json
```

Run `doctor` again after fixing permissions.

### `pending_login`

A previous `begin-login` flow is still open.

Fix after a successful `codex login`:

```bash
codex-auth-switch finish-login
```

Fix if you want to restore the previous active auth:

```bash
codex-auth-switch abort-login
```

### `cloud_synced_path`

The switcher state path looks like it is inside a cloud-synced directory.

Fix:

- Move the state directory to a local-only path.
- Set `CODEX_AUTH_SWITCH_HOME` to that local-only path.
- Save fresh snapshots if needed.

Example:

```bash
export CODEX_AUTH_SWITCH_HOME="$HOME/.codex-auth-switch"
codex-auth-switch init --fix
```

### `codex_running`

Codex App or Codex CLI appears to be running.

Fix:

- Finish the switch.
- Restart Codex CLI or Codex App so it reloads `auth.json`.

### `installed_missing`

The installed command was not found at the expected location.

Fix:

```bash
./codex-auth-switch install
```

### `installed_version_mismatch`

The installed command has a different version from your current checkout.

Fix:

```bash
./codex-auth-switch install
```

### `installed_hash_mismatch`

The installed command has the same version but different file contents.

Fix:

```bash
./codex-auth-switch install
```

### `installed_path_symlink`

The installed command path is a symlink.

Fix:

```bash
./codex-auth-switch install --force
```

Only use `--force` after confirming the target path is safe to replace.

## Common Command Errors

### `invalid_name`

Account names must match:

```text
^[A-Za-z0-9][A-Za-z0-9._-]{0,63}$
```

Use names like:

```text
personal
work
client-a
team_1
```

### `snapshot_missing`

The requested account snapshot does not exist.

Fix:

```bash
codex-auth-switch list
codex login
codex-auth-switch save <name>
```

### `not_chatgpt_auth`

The active `auth.json` does not look like Codex ChatGPT auth.

Fix:

```bash
codex login
codex-auth-switch save <name>
```

Use `--force` only if you inspected the file and know it is safe.

### `lock_exists`

Another `codex-auth-switch` process is running or a stale lock directory exists.

Fix:

- Wait for the other command to finish.
- If you are certain the lock is stale, inspect and remove the lock directory manually.

## Sensitive Output

Do not paste full auth files into issues. Redact:

- `access_token`
- `refresh_token`
- `id_token`
- Full `auth.json`
- Full `*.auth.json`

It is safe to share issue codes, command names, and redacted paths.
