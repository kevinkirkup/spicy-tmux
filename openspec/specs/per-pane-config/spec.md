# Per-Pane Config Specification

## Purpose

Defines the schema and resolution rules for per-pane configuration objects in `windows.yaml`.
Each pane position (P0–P6) may carry an optional `label`, `dir`, and `cmd` field, allowing
templates to use per-pane working directories, labels, and startup commands while falling back
to window-level defaults when fields are omitted.

---

## Requirements

### Requirement: Pane Object Schema
Each entry under a window's `panes` key SHALL be an object with optional `label`, `dir`, and `cmd` fields. All three fields are optional; an empty object `{}` is valid.

#### Scenario: Full pane object
- **WHEN** a pane entry contains `label`, `dir`, and `cmd`
- **THEN** all three values are loaded and made available as template variables

#### Scenario: Label-only pane object
- **WHEN** a pane entry contains only `label`
- **THEN** `dir` falls back to the window-level `dir` and `cmd` is treated as absent

#### Scenario: Dir-only pane object
- **WHEN** a pane entry contains only `dir`
- **THEN** the pane has no label and uses the specified directory

#### Scenario: Empty pane object
- **WHEN** a pane entry is an empty object `{}`
- **THEN** `dir` falls back to the window-level `dir`, `label` and `cmd` are absent

---

### Requirement: Per-Pane Directory Fallback
If a pane object omits `dir`, the resolved directory for that pane SHALL be the window-level `dir` field.

#### Scenario: Pane dir overrides window dir
- **GIVEN** a window with `dir: ~/repos/main` and a pane with `dir: ~/repos/other`
- **WHEN** template variables are derived
- **THEN** the pane's resolved directory is `~/repos/other`

#### Scenario: Pane without dir uses window dir
- **GIVEN** a window with `dir: ~/repos/main` and a pane with no `dir` field
- **WHEN** template variables are derived
- **THEN** the pane's resolved directory is `~/repos/main`

---

### Requirement: Per-Pane Command Override
If a pane object includes `cmd`, the value SHALL override the default command the template would run for that pane. There is no window-level `cmd` fallback. The generated script SHALL `cd` to `dir_PN` before executing the command, ensuring the command runs from the correct working directory.

#### Scenario: Pane cmd overrides template default
- **GIVEN** a pane with `cmd: tail -f app.log`
- **WHEN** the template variable `cmd_P0` is non-empty
- **THEN** the generated script runs `cd <dir_P0> && tail -f app.log` for that pane instead of the template default

#### Scenario: Pane cmd without explicit dir uses window dir
- **GIVEN** a window with `dir: ~/repos/main` and a pane with only `cmd: tail -f app.log` (no `dir`)
- **WHEN** the template is rendered
- **THEN** the generated script runs `cd ~/repos/main && tail -f app.log` for that pane

#### Scenario: Pane cmd with explicit dir uses pane dir
- **GIVEN** a window with `dir: ~/repos/main` and a pane with `dir: ~/repos/other` and `cmd: tail -f app.log`
- **WHEN** the template is rendered
- **THEN** the generated script runs `cd ~/repos/other && tail -f app.log` for that pane

#### Scenario: Absent cmd leaves template default intact
- **GIVEN** a pane with no `cmd` field
- **WHEN** template variables are derived
- **THEN** `cmd_PN` for that pane is an empty string, and templates use their default command (via `init_PN`)

---

### Requirement: Per-Pane Template Variables
For each pane position `N` (0–6), the system SHALL pass `dir_PN`, `init_PN`, and `cmd_PN` to every Jinja2 template.

| Variable   | Value                                                          |
|------------|----------------------------------------------------------------|
| `dir_PN`   | Pane `dir` if set, else window `dir`                          |
| `init_PN`  | Derived from `dir_PN` using the same envsetup.sh logic as `init` |
| `cmd_PN`   | Pane `cmd` if set, else empty string                          |

#### Scenario: Per-pane variables derived correctly
- **GIVEN** a window with `dir: ~/repos/main` and pane P2 with `dir: ~/repos/other` and `cmd: nvim .`
- **WHEN** the template is rendered
- **THEN** `dir_P2` equals `~/repos/other`
- **AND** `init_P2` is derived from `~/repos/other`
- **AND** `cmd_P2` equals `nvim .`

#### Scenario: Per-pane variables fall back for unconfigured panes
- **GIVEN** a window with `dir: ~/repos/main` and no per-pane overrides
- **WHEN** the template is rendered
- **THEN** all `dir_PN` variables equal `~/repos/main`
- **AND** all `cmd_PN` variables are empty strings

#### Scenario: Existing top-level dir and init variables preserved
- **GIVEN** any window entry
- **WHEN** the template is rendered
- **THEN** the top-level `dir` and `init` variables are still passed unchanged

---

### Requirement: Pane Key Naming
Pane objects SHALL be keyed by positional identifiers `P0`–`P6` corresponding to tmux pane indices.

#### Scenario: Positional key maps to pane index
- **GIVEN** a pane entry under key `P2`
- **WHEN** the window config is generated
- **THEN** `dir_P2`, `init_P2`, and `cmd_P2` reflect the values from that entry
