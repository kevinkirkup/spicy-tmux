## 1. Fix Templates

- [x] 1.1 In `home/.tmux/templates/default.j2`, update all `send-keys` lines that use `cmd_PN` to prepend `cd {{ dir_PN }} && ` before the command
- [x] 1.2 In `home/.tmux/templates/dev-4pane.j2`, apply the same `cd {{ dir_PN }} &&` fix for all `send-keys` lines with `cmd_PN`
- [x] 1.3 In `home/.tmux/templates/monitoring.j2`, apply the same fix
- [x] 1.4 In `home/.tmux/templates/notebook.j2`, apply the same fix
- [x] 1.5 In `home/.tmux/templates/simple.j2`, apply the same fix

## 2. Verify Generated Output

- [x] 2.1 Run `dev-windows generate` (or `dev-windows add` for a test entry) and confirm the generated script contains `cd <dir> && <cmd>` for a pane with `cmd` but no `dir`
- [x] 2.2 Confirm that a pane with no `cmd` still generates the original `init_PN` form (no regression)
- [x] 2.3 Confirm that a pane with both `cmd` and `dir` generates `cd <pane-dir> && <cmd>` (pane dir takes precedence)

## 3. Regenerate Local Configs

- [x] 3.1 Run `dev-windows generate` to regenerate all window configs in `~/.tmux/local/` from the fixed templates
