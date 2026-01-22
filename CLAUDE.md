# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a tmux configuration repository managed via [homesick](https://github.com/technicalpickles/homesick). Files in `home/` are symlinked to `~` when the castle is linked.

## Repository Structure

- `home/.tmux.conf` - Main tmux configuration file
- `home/.tmux/` - Window configuration scripts
  - Shared configs: `default-dev-session`, `tmux-dev`, `irssi`, etc.
  - `local/` - Machine-specific window configs (gitignored except `.gitignore`)
  - `local/sessions/` - Session templates that source multiple window configs
- `home/bin/` - Helper scripts (symlinked to `~/bin`)

## Configuration Architecture

### Window Configuration Files

Window configs are tmux scripts (no `.conf` extension) that define window layouts using tmux commands:
- `new-window` - Create named window at specific directory
- `split-window -h/-v` - Horizontal/vertical splits with size (`-l`)
- `select-pane -t N -T 'Title'` - Set pane titles
- `send-keys -t N 'command' Enter` - Run commands in panes

All window configs should start with: `# vim: set filetype=tmux:`

### Session Files

Session templates in `local/sessions/` create full sessions by sourcing multiple window configs:
```tmux
new -s SessionName -c $HOME
source-file ~/.tmux/local/window-config
kill-window -t 0  # Remove default window
source-file ~/.tmux/local/another-window
```

### Command Aliases

The main config defines aliases for quick window loading via `:alias-name`:
```tmux
set -s command-alias[0] ml='source-file ~/.tmux/machine-learning'
```

## Key Bindings

- **Prefix**: `C-a` (not default `C-b`)
- **Reload config**: `prefix + r`
- **Splits**: `prefix + -` (horizontal), `prefix + \` (vertical)
- **Pane navigation**: `prefix + h/j/k/l` (vim-style)
- **Window navigation**: `C-left/C-right` or `C-j/C-k`
- **Last window**: `prefix + C-a`

## Adding New Configurations

1. Create window config in `home/.tmux/` (shared) or `home/.tmux/local/` (machine-specific)
2. For shared configs, add a command alias in `.tmux.conf`
3. For sessions, create template in `local/sessions/` that sources window configs

## Environment Dependencies

- Requires `POWERLINE_DIR` environment variable for status bar
- Uses `reattach-to-user-namespace` on macOS for clipboard integration
