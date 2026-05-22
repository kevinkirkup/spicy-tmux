## Context

`dev-windows` generates tmux window config scripts from `~/.tmux/local/windows.yaml`. Currently each window entry has a single `dir` and a flat `panes` map of `L0`–`L3` label strings. All panes in a window share that `dir`, and the commands each pane runs are entirely controlled by the Jinja2 template.

This design extends the pane model so each pane can carry its own `dir` and `cmd`, while keeping backward compatibility through a migration command.

## Goals / Non-Goals

**Goals:**
- Per-pane `label`, `dir` (with window-level fallback), and `cmd` (template override, no fallback)
- Template variable set extended with `dir_P0`…`dir_P5`, `init_P0`…`init_P5`, `cmd_P0`…`cmd_P5`
- `dev-windows migrate` converts existing flat string pane entries to the new object format
- `dev-windows add` gains `--p0-dir`/`--p0-cmd` … `--p5-dir`/`--p5-cmd` flags

**Non-Goals:**
- Window-level `cmd` fallback
- Changes to hand-authored static configs in `home/.tmux/`
- Shorthand string syntax for panes (migration handles the conversion; new entries always use objects)

## Decisions

### Pane key scheme: `P0`–`P5` (positional) not `L0`–`L3` (label slots)

The existing `L0`–`L3` keys are label-slot names defined per template, not fixed pane positions. Using positional `P0`–`P5` keys directly maps to tmux pane indices (the same indices used in `send-keys -t N` and `set-option -p -t .N`), making the YAML readable alongside the template layout without needing to cross-reference label slot definitions.

Alternatives considered:
- Keep `L0`–`L3`: Familiar to existing users but ambiguous — `L0` means different pane positions in different templates.
- Numeric bare keys (`0`, `1`, …): Valid YAML but visually easy to misread and not idiomatic.

### Always-object pane entries; no shorthand string fallback

Allowing `P0: Watch` (string) alongside `P0: {label: Watch}` (object) complicates schema loading with no benefit after migration. The `migrate` command handles the one-time conversion; new entries always use the object form.

### `cmd` is a full override, not a prefix/suffix

Templates already compose commands (`init && nvim .`). Allowing partial overrides (prefix/suffix) would require templates to parse `cmd` for intent. A full override is unambiguous: if `cmd_P2` is non-empty, the template emits it verbatim for that pane instead of its default command.

### Template variables: `dir_PN`, `init_PN`, `cmd_PN`

Each pane slot gets three variables:
- `dir_PN` — resolved directory (`dirs.PN` if set, else window `dir`)
- `init_PN` — `cd <dir_PN> [&& source envsetup.sh] && clear` (same derivation as existing `init`)
- `cmd_PN` — raw command string, empty string if not set

Templates check `cmd_PN` with `{% if cmd_P2 %}{{ cmd_P2 }}{% else %}<default>{% endif %}`. The existing `dir` and `init` top-level variables are kept for templates that don't need per-pane overrides.

### Migration: dedicated `migrate` subcommand

Auto-migrating on `generate` would silently rewrite files users haven't opted into changing. A dedicated `dev-windows migrate` command makes the operation explicit, prints what it changed, and can be run once then forgotten.

## Risks / Trade-offs

- **Built-in templates must be updated** — templates that don't use per-pane variables will ignore `dir_PN`/`cmd_PN`. All five built-in templates need updating; custom local templates are unaffected until the user chooses to update them. → Ship template updates alongside the schema change.
- **`P0`–`P5` key range is fixed** — templates with more than 6 panes (e.g. `monitoring` has 7) need `P6`. → Extend variable generation to cover all pane indices actually used by built-in templates (up to `P6`).
- **Migration is one-way** — the old flat string format is not re-emitted after migration. → Document this; the format change is intentional and not expected to be reversed.

## Migration Plan

1. Run `dev-windows migrate` — rewrites `windows.yaml` converting `panes: {L0: Watch}` → `panes: {P0: {label: Watch}}`.
2. Run `dev-windows generate` — regenerates all window config files using the new per-pane template variables.
3. Reload tmux bindings (`prefix + r` or `tmux source ~/.tmux.conf`) — no session restart required.

Rollback: restore `windows.yaml` from git (`git checkout -- ~/.tmux/local/windows.yaml`) and re-run `generate`.

## Open Questions

- Should `migrate` rename `L0`–`L3` keys to `P0`–`P3` using a fixed slot-to-pane mapping per template, or always map positionally (`L0`→`P0`, `L1`→`P1`, …)? Positional mapping is simpler and correct for all existing built-in templates.
