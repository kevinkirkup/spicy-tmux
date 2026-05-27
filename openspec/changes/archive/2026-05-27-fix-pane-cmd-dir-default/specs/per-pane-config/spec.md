## MODIFIED Requirements

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
