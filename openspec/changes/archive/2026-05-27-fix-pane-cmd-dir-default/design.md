## Context

The `dev-windows` tool renders Jinja2 templates with per-pane variables (`dir_PN`, `init_PN`, `cmd_PN`). Templates currently use a conditional:

```
send-keys -t N '{% if cmd_PN %}{{ cmd_PN }}{% else %}{{ init_PN }}{% endif %}' Enter
```

`init_PN` expands to `cd <dir> && clear` (or with envsetup.sh sourcing). When `cmd_PN` is truthy, the template sends **only the raw command**, skipping the `cd`. The pane's working directory then depends entirely on where the shell initialized — typically `~`.

The `resolve_pane_vars` function in `dev-windows` correctly resolves `dir_PN` to the window-level `dir` when the pane has no `dir` key. The bug is purely in template rendering: the `cd` step is elided when `cmd` is present.

## Goals / Non-Goals

**Goals:**
- Pane with `cmd` set runs the command from `dir_PN` (window's `dir` when pane has no `dir`)
- Behavior identical to current for panes without `cmd`
- All 5 shared templates fixed consistently

**Non-Goals:**
- Running envsetup.sh or `clear` before the custom command
- Changing the `dev-windows` Python script or `windows.yaml` schema
- Fixing the unrelated missing `cd` in `get_init` for the envsetup.sh branch

## Decisions

### Prepend `cd {{ dir_PN }} &&` before `cmd_PN` in templates

**Decision:** Change template conditionals from:
```
'{% if cmd_PN %}{{ cmd_PN }}{% else %}{{ init_PN }}{% endif %}'
```
to:
```
'{% if cmd_PN %}cd {{ dir_PN }} && {{ cmd_PN }}{% else %}{{ init_PN }}{% endif %}'
```

**Rationale:** Minimal, targeted fix. Only the failing branch changes. The no-`cmd` path (`init_PN`) is untouched.

**Alternative considered — use `-c '{{ dir_PN }}'` on each `split-window`:**
Sets the tmux start-directory rather than relying on shell `cd`. Cleaner in theory, but the shell's own init (`~/.zshrc`, etc.) can override it after startup, making it less reliable than an explicit `cd` via `send-keys`. Also requires touching every `split-window` line.

**Alternative considered — generate a `dir_cd_PN` variable in `resolve_pane_vars`:**
Would let templates stay DRY if more complex logic were needed. Overkill for a one-liner fix that only touches the `cmd`-present branch.

## Risks / Trade-offs

- **Generated files become stale** → Users must re-run `dev-windows generate` after the fix; no auto-regeneration exists.
- **envsetup.sh not sourced with `cmd`** → By design (non-goal), but worth documenting: custom commands run after a plain `cd`, not after the full env setup.
- **Long commands with single quotes** → If a `cmd` value contains a single-quote character, it will break the tmux `send-keys` quoting. Pre-existing issue, not introduced by this fix.
