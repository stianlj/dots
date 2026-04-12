# dotfiles

> Previously: [stianlj/dotfiles](https://github.com/stianlj/dotfiles/)

Managed with [chezmoi](https://www.chezmoi.io/). Source state lives in `~/.local/share/chezmoi` and is applied to the target system with `chezmoi apply`.

## Tools

| Role | Tool |
|---|---|
| Shell | Fish (vi keys) |
| Terminal | Ghostty |
| Multiplexer | tmux (prefix: `C-Space`) |
| Editor | Neovim |
| Prompt | Starship |
| CD | zoxide (`cd`) |
| Env | direnv |
| History | atuin |
| Session | sesh |
| Idle/lock | stasis + hyprlock |
| Compositor | niri |
| Theme | Catppuccin Mocha |

## Setup

```sh
sh -c "$(curl -fsLS get.chezmoi.io)"
chezmoi init git@github.com:stianlj/dots.git
chezmoi apply
```

After applying, install tmux plugins on first launch (or `prefix + I` via tpm). Neovim plugins and Mason LSP servers install automatically on first open.

## Workflow

```sh
chezmoi diff        # preview changes before applying
chezmoi apply       # write source state to ~
chezmoi edit <file> # open a target file in $EDITOR
chezmoi cd          # shell into the source directory
```

Neovim also runs `chezmoi.commands.__edit`.watch()` on chezmoi source files, so editing them directly in `~/.local/share/chezmoi/` in Neovim will auto-sync changes to the target.

## Bin scripts

`bin/` contains executable scripts that chezmoi installs to `~/bin/`:

| Script | Purpose |
|---|---|
| `sesh-sessionizer` | fzf-based session picker and repo cloner for sesh (supports GitHub/GitLab) |
| `tmux-update-env` | syncs environment variables into tmux after client reconnect (called by tmux hook) |
| `github-status-notifications.sh` | waybar module — GitHub notification count |
| `github-status-pr.sh` | waybar module — GitHub PR review count |
| `pomodoro-status.sh` | waybar module — pomodoro timer status |

## Niri

Main config: `dot_config/niri/config.kdl.tmpl`

Uses `include` directives to pull in shared, machine-type, and Dank Material Shell configs. The `machineType` chezmoi variable (`"laptop"` or `"desktop"`) controls which type-specific includes are active:

```kdl
// Type-specific
{{- if eq .machineType "laptop" }}
include optional=true "./type-laptop/input.kdl"
include optional=true "./type-laptop/layout.kdl"
include optional=true "./type-laptop/spawns.kdl"
{{- else if eq .machineType "desktop" }}
include optional=true "./type-desktop/input.kdl"
include optional=true "./type-desktop/output.kdl"
include optional=true "./type-desktop/spawns.kdl"
{{- end }}
```

Set `machineType` in `chezmoi.toml`:

```toml
[data]
  machineType = "desktop"
```

Machine-specific overrides go in per-host subdirs under `dot_config/niri/`.

## Neovim

- **Plugin manager:** [lazy.nvim](https://lazy.nvim.org/), auto-bootstrapped from `lua/settings.lua`
- **Plugin structure:** individual Lua files in `lua/plugins/` returning spec tables, loaded via `{ import = "plugins" }`
- **LSP:** uses the new `vim.lsp.config()` / `vim.lsp.enable()` API (not nvim-lspconfig setup style)
- **Completion:** blink.cmp (enter-to-accept preset)
- **LSP servers:** auto-installed via mason-tool-installer
- **Formatting:** conform.nvim on save (stylua, prettierd, black+isort, shfmt, etc.)
- **Leader:** `<Space>` (both leader and local leader)
- **Legacy:** `lua/plugins-legacy/` holds older configs still referenced by telescope and dap

Formatting config: `.stylua.toml` sets Lua to 2-space indent with spaces. `.editorconfig` enforces LF line endings and final newlines. Neovim's `.luarc.json` configures lua_ls for the plugin structure.

## Tmux

- Prefix: `C-Space` (not the default `C-b`)
- Vi copy mode, split panes inherit `pane_current_path`
- Plugins managed by tpm (vendored in `plugins/`)
- Smart pane navigation (`C-h/j/k/l`) with vim awareness
- Session picker bound to `prefix + K`
- `tmux-update-env` auto-syncs env vars (SSH agent, Wayland, API keys) on client-attached

## Chezmoi filename prefixes

These prefixes control how files are installed on the target system. Do not replace them with literal dots.

| Prefix | Effect |
|---|---|
| `dot_` | Prepends a dot (`dot_config/` → `~/.config/`) |
| `executable_` | Sets +x on the target file |
| `private_` | Restricts permissions (600/700) |
| `readonly_` | Makes the target file read-only |

## What is not tracked

See `.chezmoiignore` (chezmoi skips) and `.gitignore` (git skips). Notable entries:

- `fish_variables`, `fish/completions/` — machine-generated
- `tmux/plugins/`, `tmux/resurrect` — vendored / runtime state
- `yazi/plugins/` — third-party package installs
- `niri/dms/`, `hypr/dms/` — Dank Material Shell generated files
- `nvim/lazy-lock.json`, `nvim/lua/packer_compiled.lua` — auto-generated
- `qutebrowser/` bookmarks, quickmarks, private config
