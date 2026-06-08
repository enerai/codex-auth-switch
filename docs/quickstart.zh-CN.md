# 快速开始

[English](quickstart.md) | 中文

这份文档说明 `codex-auth-switch` 的首次安装, 首次保存账号, 添加另一个账号, 以及日常切换账号流程.

## 1. 安装命令

在仓库根目录执行:

```bash
./codex-auth-switch install
```

默认安装路径是:

```text
$HOME/.local/bin/codex-auth-switch
```

如果这个目录不在 `PATH` 中, 可以使用完整路径:

```bash
$HOME/.local/bin/codex-auth-switch version
```

你也可以安装到自定义目录:

```bash
./codex-auth-switch install --bin-dir "$HOME/bin"
./codex-auth-switch install --prefix "$HOME/.local"
```

仓库升级后, 再运行一次 `./codex-auth-switch install`. 已安装副本不是 symlink.

## 2. 初始化 Codex file-backed auth

运行:

```bash
codex-auth-switch init --fix
codex-auth-switch --json doctor
```

`init --fix` 会创建 switcher state 目录, 并确保 Codex 配置为:

```toml
cli_auth_credentials_store = "file"
```

如果 `doctor` 报告了 `ERROR`, 先修复这些问题, 再保存账号快照.

## 3. 保存第一个账号

先用 Codex 登录第一个账号:

```bash
codex login
```

把当前 active `auth.json` 保存成命名快照:

```bash
codex-auth-switch save personal
codex-auth-switch list
codex-auth-switch current
```

预期结果:

- `accounts/personal.auth.json` 出现在 switcher state 目录下.
- `current` 输出 `personal`.
- `list` 只展示账号别名, 保存时间, current 标记和短 hash, 不展示 token 值.

## 4. 添加另一个账号

为第二个账号开始登录流程:

```bash
codex-auth-switch begin-login work
```

然后登录这个账号:

```bash
codex login
```

完成 switcher 流程:

```bash
codex-auth-switch finish-login
codex-auth-switch list
```

预期结果:

- `begin-login work` 会把之前 active 的 `auth.json` 隐藏到 `before-login/`.
- `finish-login` 会把新 active `auth.json` 保存成 `accounts/work.auth.json`.
- `current` 变成 `work`.
- `work` 对应的 pending login 文件被清理.

## 5. 切换账号

切回第一个账号:

```bash
codex-auth-switch use personal
```

重启 Codex CLI 或 Codex App, 让进程读取恢复后的 `auth.json`.

切到第二个账号:

```bash
codex-auth-switch use work
```

再次重启 Codex.

`use <name>` 会先尝试把当前 active `auth.json` 存回 current 对应快照, 再恢复目标快照.

如果 active `auth.json` 已损坏, 或形状不像 ChatGPT auth, `use` 会输出 warning, 跳过这次保存, 然后继续切到目标快照. symlink 和非普通文件仍会被拒绝.

## 6. 取消未完成登录

如果你运行了 `begin-login work`, 但没有完成登录:

```bash
codex-auth-switch abort-login
```

预期结果:

- 如果 active `auth.json` 缺失, `abort-login` 会恢复隐藏的 pre-login auth 文件.
- 成功恢复后, 它会删除 `pending-login` 和匹配的调试备份.
- 如果 active `auth.json` 已存在, 它会拒绝覆盖, 除非你传入 `--force`.

只在确认哪个 auth 文件应该成为 active 后, 再使用 `--force`.

## 7. 检查本地状态

常用命令:

```bash
codex-auth-switch paths
codex-auth-switch version
codex-auth-switch --json doctor
codex-auth-switch list
codex-auth-switch current
```

检查重点:

- `doctor` 没有 `ERROR`.
- Codex 已配置为 file-backed auth.
- `accounts/*.auth.json`, `meta/*.json`, `current`, `pending-login` 使用 `600` 权限.
- state 目录使用 `700` 权限.
- state 路径不在云同步目录中.

## 8. 删除快照

只删除已保存快照, 不删除 active auth:

```bash
codex-auth-switch remove personal
```

同时删除快照和 active auth 文件:

```bash
codex-auth-switch remove work --active-too
```

只有当你确认当前 active `auth.json` 属于要删除的账号时, 才使用 `--active-too`.

## 9. 测试工具

测试只使用临时目录, 不会触碰真实 `~/.codex`:

```bash
zsh tests/test_codex_auth_switch.sh
```
