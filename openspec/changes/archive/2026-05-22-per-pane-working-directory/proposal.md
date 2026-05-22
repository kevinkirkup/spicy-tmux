## Why

All panes in a generated window config currently inherit the same working directory from the window-level `dir` field, and pane commands are fixed inside templates. This forces workarounds like explicit `cd` and hardcoded commands in templates, making them brittle and `windows.yaml` harder to read when panes genuinely live in different directories or need custom startup commands.

## What Changes

- `windows.yaml` pane entries change from flat label strings to objects with optional `label`, `dir`, and `cmd` fields.
  - `dir` falls back to the window-level `dir` if omitted.
  - `cmd` overrides the command the template would normally run for that pane; no window-level fallback.
- Template variables are extended: each pane slot receives `dir_P0`, `dir_P1`, … (resolved directory) and `init_P0`, `init_P1`, … (derived with the same `envsetup.sh` logic), plus `cmd_P0`, `cmd_P1`, … (empty string if not set, letting templates fall back to their default behavior).
- Built-in templates are updated to use per-pane variables.
- `dev-windows add` gains optional `--p0-dir`, `--p1-dir`, … and `--p0-cmd`, `--p1-cmd`, … flags.
- A `dev-windows migrate` command converts existing `windows.yaml` files from old flat string pane entries to the new object format in-place.

Example `windows.yaml`:

```yaml
my-project:
  dir: ~/repos/my-project        # window-level dir fallback
  template: dev-4pane
  panes:
    P0:
      label: Watch
      dir: ~/repos/my-project/logs
      cmd: tail -f app.log
    P1:
      label: Logs
    P2:
      label: Editor
      dir: ~/repos/shared-lib
      cmd: nvim .
    P3:
      label: Console
```

## Non-goals

- Window-level `cmd` fallback (commands are per-pane only).
- Changing the fallback resolution for static window configs in `home/.tmux/` (those are hand-authored).

## Capabilities

### New Capabilities
- `per-pane-config`: Per-pane `label`, `dir`, and `cmd` configuration in `windows.yaml`, with `dir` falling back to window-level `dir` and `cmd` overriding the template's default command for that pane.

### Modified Capabilities
- `dev-windows`: The `windows.yaml` pane schema, template variable set, `add` CLI flags, built-in templates, and a new `migrate` command all change to support per-pane configuration.

## Impact

- `home/bin/dev-windows` — schema loading, template variable derivation, `add` command flags, new `migrate` command
- `home/.tmux/templates/*.j2` — built-in templates consume new per-pane variables
- `home/.tmux/local/windows.yaml` — migrated to new pane object format
- `openspec/specs/dev-windows/spec.md` — delta spec needed for schema and variable changes
