## Why

When a pane has `cmd` set but no `dir`, the generated tmux script sends the command directly without first `cd`-ing to the window's default directory. The `init_PN` variable (which contains the `cd <dir> && clear` logic) is bypassed entirely when `cmd` is present, so the pane runs the command from whatever directory the shell initialized to (typically `~`), not from the window's `dir`.

## What Changes

- Templates updated to `cd` to `dir_PN` before executing `cmd_PN`, so pane commands always run from the correct directory
- All existing templates (`default.j2`, `dev-4pane.j2`, `monitoring.j2`, `notebook.j2`, `simple.j2`) updated consistently

## Capabilities

### New Capabilities
<!-- None — this is a bug fix, no new capabilities introduced -->

### Modified Capabilities
- `per-pane-config`: The existing requirement that "pane without dir uses window dir" now correctly applies when `cmd` is set; previously only enforced when falling back to `init_PN`

## Impact

- All generated window config files must be regenerated via `dev-windows generate` after the fix
- No changes to `windows.yaml` schema or the `dev-windows` Python script
- No breaking changes — existing behavior for panes without `cmd` is unchanged

## Non-goals

- Adding a separate `init_cmd` concept that runs both the `cd`/envsetup logic and a custom command with a shell clear between them
- Modifying the `-c` start-directory flag on `split-window` commands (shell init could still override this)
- Changing behavior for panes that explicitly set their own `dir`
