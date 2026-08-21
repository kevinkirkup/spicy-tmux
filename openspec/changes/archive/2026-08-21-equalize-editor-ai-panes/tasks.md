## 1. Template Edits

- [x] 1.1 In `home/.tmux/templates/default.j2` (shared, tracked config, symlinked to
  `~/.tmux/templates/default.j2` by homesick) change `split-window -v -l 15 -t 3` to
  `split-window -v -l 50% -t 3`. Verify with `git diff home/.tmux/templates/default.j2`
  showing exactly that one line changed.
- [x] 1.2 In `home/.tmux/templates/dev-4pane.j2` change `split-window -v -l 15 -t 2` to
  `split-window -v -l 50% -t 2`. Verify with `git diff home/.tmux/templates/dev-4pane.j2`
  showing exactly that one line changed.
- [x] 1.3 In `home/.tmux/templates/monitoring.j2` change `split-window -v -l 15 -t 3` to
  `split-window -v -l 50% -t 3`, leaving the `split-window -v -l 80 -t 2`
  (Monitoring/Console) line untouched. Verify with
  `git diff home/.tmux/templates/monitoring.j2` showing exactly that one line changed.
- [x] 1.4 In `home/.tmux/templates/simple.j2` change `split-window -v -l 15 -t 1` to
  `split-window -v -l 50% -t 1`, and add
  `set-option -p -t .1 @pane_label 'Editor'` and `set-option -p -t .2 @pane_label 'AI'`
  immediately after the split-window lines. Verify with
  `git diff home/.tmux/templates/simple.j2` matching the design's described edit (one
  changed line, two new lines).

## 2. Spec Sync

- [x] 2.1 Apply this change's spec delta to `openspec/specs/dev-windows/spec.md`, updating
  the `simple` template scenario under "Built-in Templates" to document the new fixed
  `Editor`/`AI` labels. Verify by diffing the updated scenario against
  `openspec/changes/equalize-editor-ai-panes/specs/dev-windows/spec.md` and confirming they
  match.

## 3. Verification

- [x] 3.1 Render each of the four edited templates with sample values (via the
  `dev-windows` CLI, or a manual Jinja2 render) and confirm `tmux` accepts the generated
  script, e.g. `tmux source-file <generated-file>` in a scratch session, with no errors.
- [x] 3.2 In a live tmux session, load each of `default`, `dev-4pane`, `monitoring`, and
  `simple` and visually confirm the Editor and AI panes are equal height, and that
  `simple`'s Editor/AI panes now show their labels. (Verified programmatically via
  `tmux list-panes` on a scratch session: Editor/AI pane heights differ by at most 1 line
  across all four templates — expected tmux rounding on odd splits, not a defect — and
  `simple`'s panes report `label=Editor`/`label=AI`. Final human visual look in a real
  session still recommended.)
- [x] 3.3 Manual follow-up (machine-local, not tracked by this repo): run
  `dev-windows generate` to regenerate any existing window configs under
  `~/.tmux/local/` that use these templates, since editing the shared `.j2` files has no
  effect on already-generated gitignored files until they're regenerated. (Ran
  `dev-windows generate` — regenerated all 48 configured windows and their key bindings;
  spot-checked `atlas`, `archer`, `mercury`, `atlantis` and confirmed each now carries the
  `-l 50%` Editor/AI split.)
