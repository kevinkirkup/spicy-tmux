## Context

See `proposal.md` - Why. Each affected template's Editor/AI split is currently a `tmux
split-window -v` call with a fixed line count for the AI pane (`-l 15`); the sibling
`notebook.j2` already uses a percentage (`-l 50%`) for the same split and is the pattern
being matched. `simple.j2` additionally has zero `@pane_label` lines at all today - not
even for Editor, which every other template sets.

## Goals / Non-Goals

**Goals:**
- Editor and AI end up visually equal-weight (50/50) in `default.j2`, `dev-4pane.j2`,
  `monitoring.j2`, and `simple.j2`.
- `simple.j2` gains the same fixed `Editor`/`AI` labels its siblings already have.

**Non-Goals:**
- No new template variables or configurability (no `L`-style override for `simple.j2`'s
  panes) - keeps this a pure layout/label fix, not a feature addition.
- No change to how `dev-windows` (the Python CLI) resolves or passes template variables -
  this only touches the `.j2` files themselves.

## Decisions

- **Use `-l 50%` instead of computing a new fixed line count.** `notebook.j2` already
  proves this pattern works, and percentage splits stay correct as the user's terminal
  size changes, whereas a fixed line count (e.g. `-l 40`) would only be right for one
  terminal height. Matching the existing convention beats inventing a new one.
- **Leave `simple.j2` pane 0 unlabeled.** The proposal's Modified Capability only commits
  to documenting fixed `Editor`/`AI` labels; adding a label (configurable or not) to pane 0
  would be new scope not requested and not covered by the spec delta.

## Risks / Trade-offs

- **Mixed fixed/percentage splits within the same template** → `default.j2` and
  `monitoring.j2` still use fixed-line splits for their other panes (e.g. the `Tests`
  strip, the `Monitoring`/`Console` split). Only the Editor/AI pair moves to percentage.
  This is an accepted inconsistency, not a defect - proposal.md's Non-goals explicitly
  excludes those other splits from this change.
- **Generated machine-local configs won't reflect this until regenerated** → editing the
  shared `.j2` templates has no effect on already-generated files under
  `~/.tmux/local/<name>` (gitignored, machine-local) until `dev-windows generate` is run.
  Mitigation: call this out in `tasks.md` as a manual follow-up step, not something this
  repo can automate.
