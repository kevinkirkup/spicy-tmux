## 1. Schema & Variable Derivation (`home/.tmux/bin/dev-windows`)

- [x] 1.1 Update the YAML loading logic to accept pane entries as either a plain string (legacy) or an object with optional `label`, `dir`, and `cmd` fields
- [x] 1.2 Add a `resolve_pane_vars(window_entry, pane_count=7)` helper that, for each pane index 0–6, resolves `dir_PN` (pane `dir` or window `dir` fallback), `init_PN` (using existing `envsetup.sh` derivation on `dir_PN`), and `cmd_PN` (pane `cmd` or empty string)
- [x] 1.3 Update `render_window` (or equivalent template context builder) to inject `dir_P0`…`dir_P6`, `init_P0`…`init_P6`, and `cmd_P0`…`cmd_P6` alongside the existing `dir`, `init`, and `L0`–`L3` variables
- [x] 1.4 Update `L0`–`L3` extraction to read from `panes.P0.label`…`panes.P3.label` in the new object format (while staying compatible with the legacy string format during migration)

## 2. `add` Subcommand Per-Pane Flags (`home/.tmux/bin/dev-windows`)

- [x] 3.1 Add Click options `--p0-dir` through `--p6-dir` and `--p0-cmd` through `--p6-cmd` to the `add` subcommand
- [x] 3.2 Merge provided per-pane flags into the `panes` map using the new object format (e.g. `--p0-dir ~/logs --p0-cmd "tail -f app.log"` → `P0: {dir: ~/logs, cmd: tail -f app.log}`)
- [x] 3.3 Preserve existing `--l0`–`--l3` label flags by storing them as `label` within the corresponding `P0`–`P3` pane objects

## 4. Update Built-in Templates (`home/.tmux/templates/`)

- [x] 4.1 Update `default.j2` (6 panes) to use `init_P0`–`init_P5` and check `cmd_PN` with `{% if cmd_PN %}{{ cmd_PN }}{% else %}<default>{% endif %}` for each pane
- [x] 4.2 Update `dev-4pane.j2` (4 panes) with per-pane variables
- [x] 4.3 Update `simple.j2` (3 panes) with per-pane variables
- [x] 4.4 Update `monitoring.j2` (7 panes) with per-pane variables including `P6`
- [x] 4.5 Update `notebook.j2` (3 panes) with per-pane variables

## 5. Migrate Local Config (`~/.tmux/local/windows.yaml`)

- [x] 5.1 Convert `home/.tmux/local/windows.yaml` directly: rename `L0`–`L3` keys to `P0`–`P3` and rewrite flat string values as `{label: <value>}` objects
- [x] 5.2 Run `dev-windows generate` to regenerate all window config files and verify output is valid tmux syntax

## 6. Update Example File (`home/.tmux/examples/windows.yaml`)

- [x] 6.1 Rewrite `home/.tmux/examples/windows.yaml` to use the new pane object format (`P0: {label: ..., dir: ..., cmd: ...}`), updating comments to document the new schema
