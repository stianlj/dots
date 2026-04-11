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
chezmoi diff       # preview changes before applying
chezmoi apply      # write source state to ~
chezmoi edit <file> # open a target file in $EDITOR
chezmoi cd         # shell into the source directory
```

## Niri

Main config: `dot_config/niri/config.kdl`

Uses `include` directives to pull in shared, machine-type, and Dank Material Shell configs. Switch between desktop and laptop by (un)commenting `include` lines in `config.kdl`:

```kdl
// --laptop
// include optional=true "./type-laptop/input.kdl"
// include optional=true "./type-laptop/layout.kdl"
// include optional=true "./type-laptop/spawns.kdl"
// --desktop
include optional=true "./type-desktop/input.kdl"
include optional=true "./type-desktop/output.kdl"
include optional=true "./type-desktop/spawns.kdl"
```

Machine-specific overrides go in per-host subdirs (toggled the same way).

## Neovim

- **Plugin manager:** [lazy.nvim](https://lazy.nvim.org/), auto-bootstrapped from `lua/settings.lua`
- **Plugin structure:** individual Lua files in `lua/plugins/` returning spec tables, loaded via `{ import = "plugins" }`
- **LSP:** uses the new `vim.lsp.config()` / `vim.lsp.enable()` API (not nvim-lspconfig setup style)
- **Completion:** blink.cmp (enter-to-accept preset)
- **LSP servers:** auto-installed via mason-tool-installer
- **Formatting:** conform.nvim on save (stylua, prettierd, black+isort, shfmt, etc.)
- **Leader:** `<Space>` (both leader and local leader)
- **Legacy:** `lua/plugins-legacy/` holds older configs still referenced by telescope and dap

## Tmux

- Prefix: `C-Space` (not the default `C-b`)
- Vi copy mode, split panes inherit `pane_current_path`
- Plugins managed by tpm (vendored in `plugins/`)
- Smart pane navigation (`C-h/j/k/l`) with vim awareness
- Session picker bound to `prefix + K`

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