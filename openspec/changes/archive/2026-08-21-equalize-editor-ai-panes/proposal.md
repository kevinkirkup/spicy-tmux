## Why

The AI terminal pane — the pane where an agent session is actually driven, distinct from
the "AI Agents" dashboard pane — is meant to carry the same weight as the Editor pane. In
`default.j2`, `dev-4pane.j2`, and `monitoring.j2` it's squeezed into a fixed 15-line strip
below a much larger Editor pane. `notebook.j2` already treats Editor and AI as equal
partners (50/50 split); the other templates should match. Separately, `simple.j2` is
missing pane labels for Editor/AI entirely, even though its sibling templates all label
these panes.

## What Changes

- Resize the Editor/AI split in `default.j2`, `dev-4pane.j2`, and `monitoring.j2` from a
  fixed 15-line AI strip to an even 50/50 split, matching `notebook.j2`'s existing pattern.
- Add fixed `Editor` and `AI` `@pane_label` lines to `simple.j2`, and resize its Editor/AI
  split to 50/50 to match.
- `monitoring.j2`'s separate Monitoring/Console split (`-l 80`) is untouched — this change
  only touches the Editor/AI pair.
- `notebook.j2` is untouched — already 50/50.

## Capabilities

### New Capabilities
(none)

### Modified Capabilities
- `dev-windows`: the `simple` template's Built-in Templates scenario currently says the
  template has "no configurable labels" and doesn't document any fixed labels either —
  because today it emits none. After this change it emits fixed `Editor`/`AI` labels
  (still not configurable, so the "no configurable labels" wording stays accurate, but the
  scenario needs to state the labels now exist, matching how `default`'s scenario
  documents its fixed `Editor`/`AI` labels).

## Impact

- `home/.tmux/templates/default.j2`
- `home/.tmux/templates/dev-4pane.j2`
- `home/.tmux/templates/monitoring.j2`
- `home/.tmux/templates/simple.j2`
- `openspec/specs/dev-windows/spec.md` (delta for the `simple` template scenario)

No `dev-windows` CLI/tooling code changes — templates only. These are shared, tracked
configs, not machine-local (`home/.tmux/local/` is gitignored and unaffected).

## Non-goals

- Not resizing or relabeling the "AI Agents" dashboard pane
- Not touching `monitoring.j2`'s Monitoring/Console split
- Not modifying `notebook.j2` (already correct)
- Not updating `home/.tmux/examples/windows.yaml` (its doc comments describe pane
  composition, not split ratios, and don't need to change)
- Not addressing the pre-existing uncommitted Console→AI relabeling already in the working
  tree — that's a separate, already in-flight change, treated here as the baseline to diff
  against rather than something this change touches
