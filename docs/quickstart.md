# Quickstart

English | [中文](quickstart.zh-CN.md)

This guide walks through the first setup and daily account switching flow for `codex-auth-switch`.

## 1. Install The Command

From a cloned repository:

```bash
./codex-auth-switch install
```

The default install path is:

```text
$HOME/.local/bin/codex-auth-switch
```

If that directory is not on your `PATH`, use the full path:

```bash
$HOME/.local/bin/codex-auth-switch version
```

You can install to a custom directory:

```bash
./codex-auth-switch install --bin-dir "$HOME/bin"
./codex-auth-switch install --prefix "$HOME/.local"
```

After upgrading your checkout, run `./codex-auth-switch install` again. The installed copy is not a symlink.

## 2. Initialize File-Backed Codex Auth

Run:

```bash
codex-auth-switch init --fix
codex-auth-switch --json doctor
```

`init --fix` creates the switcher state directory and ensures Codex is configured with:

```toml
cli_auth_credentials_store = "file"
```

If `doctor` reports `ERROR` entries, fix them before saving snapshots.

## 3. Save The First Account

Log in to the first account with Codex:

```bash
codex login
```

Save the active `auth.json` as a named snapshot:

```bash
codex-auth-switch save personal
codex-auth-switch list
codex-auth-switch current
```

Expected result:

- `accounts/personal.auth.json` exists under the switcher state directory.
- `current` prints `personal`.
- `list` shows aliases, timestamps, current markers, and short hashes, but no token values.

## 4. Add Another Account

Start a login window for the second account:

```bash
codex-auth-switch begin-login work
```

Then log in to that account:

```bash
codex login
```

Finish the switcher flow:

```bash
codex-auth-switch finish-login
codex-auth-switch list
```

Expected result:

- `begin-login work` hides the previously active `auth.json` under `before-login/`.
- `finish-login` saves the new active `auth.json` as `accounts/work.auth.json`.
- `current` becomes `work`.
- Pending login files for `work` are cleaned up.

## 5. Switch Accounts

Switch to the first account:

```bash
codex-auth-switch use personal
```

Restart Codex CLI or Codex App so the process reads the restored `auth.json`.

Switch to the second account:

```bash
codex-auth-switch use work
```

Restart Codex again.

`use <name>` first tries to save the currently active `auth.json` back into the current snapshot, then restores the requested snapshot.

If the active `auth.json` is damaged or is not shaped like ChatGPT auth, `use` emits a warning, skips that save, and continues switching to the requested snapshot. Symlinks and non-regular files are still refused.

## 6. Abort An Unfinished Login

If you ran `begin-login work` but did not finish logging in:

```bash
codex-auth-switch abort-login
```

Expected result:

- If active `auth.json` is missing, `abort-login` restores the hidden pre-login auth file.
- It removes `pending-login` and the matching debug backup after successful restore.
- It refuses to overwrite an existing active `auth.json` unless you pass `--force`.

Only use `--force` after confirming which auth file should become active.

## 7. Inspect Local State

Useful commands:

```bash
codex-auth-switch paths
codex-auth-switch version
codex-auth-switch --json doctor
codex-auth-switch list
codex-auth-switch current
```

Check that:

- `doctor` has no `ERROR` entries.
- Codex is configured for file-backed auth.
- `accounts/*.auth.json`, `meta/*.json`, `current`, and `pending-login` use mode `600`.
- State directories use mode `700`.
- The state path is not inside a cloud-synced directory.

## 8. Remove A Snapshot

Remove a saved snapshot without deleting active auth:

```bash
codex-auth-switch remove personal
```

Remove a snapshot and the active auth file:

```bash
codex-auth-switch remove work --active-too
```

Use `--active-too` only when you are sure the active `auth.json` belongs to the account you want to remove.

## 9. Test The Tool

The tests use temporary directories and do not touch your real `~/.codex`:

```bash
zsh tests/test_codex_auth_switch.sh
```
