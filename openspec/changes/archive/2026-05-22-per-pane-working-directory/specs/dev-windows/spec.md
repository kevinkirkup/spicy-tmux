## MODIFIED Requirements

### Requirement: Template Variables
The system SHALL pass the following variables to every Jinja2 template during rendering.

| Variable    | Source                                                                         |
|-------------|--------------------------------------------------------------------------------|
| `name`      | The window key from `windows.yaml`                                             |
| `dir`       | The `dir` field of the window entry                                            |
| `init`      | Shell initialization string derived from `dir` (see Init String below)        |
| `L0`        | `panes.P0.label` from the entry, or empty string if absent                    |
| `L1`        | `panes.P1.label` from the entry, or empty string if absent                    |
| `L2`        | `panes.P2.label` from the entry, or empty string if absent                    |
| `L3`        | `panes.P3.label` from the entry, or empty string if absent                    |
| `dir_P0`    | `panes.P0.dir` if set, else window `dir`                                       |
| `dir_P1`    | `panes.P1.dir` if set, else window `dir`                                       |
| `dir_P2`    | `panes.P2.dir` if set, else window `dir`                                       |
| `dir_P3`    | `panes.P3.dir` if set, else window `dir`                                       |
| `dir_P4`    | `panes.P4.dir` if set, else window `dir`                                       |
| `dir_P5`    | `panes.P5.dir` if set, else window `dir`                                       |
| `dir_P6`    | `panes.P6.dir` if set, else window `dir`                                       |
| `init_P0`   | Init string derived from `dir_P0`                                              |
| `init_P1`   | Init string derived from `dir_P1`                                              |
| `init_P2`   | Init string derived from `dir_P2`                                              |
| `init_P3`   | Init string derived from `dir_P3`                                              |
| `init_P4`   | Init string derived from `dir_P4`                                              |
| `init_P5`   | Init string derived from `dir_P5`                                              |
| `init_P6`   | Init string derived from `dir_P6`                                              |
| `cmd_P0`    | `panes.P0.cmd` if set, else empty string                                       |
| `cmd_P1`    | `panes.P1.cmd` if set, else empty string                                       |
| `cmd_P2`    | `panes.P2.cmd` if set, else empty string                                       |
| `cmd_P3`    | `panes.P3.cmd` if set, else empty string                                       |
| `cmd_P4`    | `panes.P4.cmd` if set, else empty string                                       |
| `cmd_P5`    | `panes.P5.cmd` if set, else empty string                                       |
| `cmd_P6`    | `panes.P6.cmd` if set, else empty string                                       |

#### Scenario: Per-pane label variables passed through
- **GIVEN** a window entry with `panes: {P0: {label: Watch}, P1: {label: Logs}}`
- **WHEN** the template is rendered
- **THEN** `L0` equals `Watch` and `L1` equals `Logs`

#### Scenario: Absent pane labels default to empty string
- **GIVEN** a window entry defines no `panes` key
- **WHEN** the template is rendered
- **THEN** `L0`, `L1`, `L2`, and `L3` are all empty strings

#### Scenario: Per-pane dir variables fall back to window dir
- **GIVEN** a window with `dir: ~/repos/main` and no per-pane `dir` overrides
- **WHEN** the template is rendered
- **THEN** all `dir_PN` variables equal `~/repos/main`

#### Scenario: Per-pane dir variable uses pane override
- **GIVEN** a window with `dir: ~/repos/main` and `panes: {P2: {dir: ~/repos/other}}`
- **WHEN** the template is rendered
- **THEN** `dir_P2` equals `~/repos/other` and all other `dir_PN` equal `~/repos/main`

#### Scenario: Per-pane cmd variable is empty when not set
- **GIVEN** a window entry with no `cmd` in any pane
- **WHEN** the template is rendered
- **THEN** all `cmd_PN` variables are empty strings

#### Scenario: Per-pane cmd variable reflects pane cmd
- **GIVEN** a window with `panes: {P0: {cmd: tail -f app.log}}`
- **WHEN** the template is rendered
- **THEN** `cmd_P0` equals `tail -f app.log`

---

## ADDED Requirements

### Requirement: `add` Command Per-Pane Flags
The `add` command SHALL accept `--p0-dir` through `--p6-dir` and `--p0-cmd` through `--p6-cmd` flags to set per-pane directory and command overrides.

#### Scenario: Adding a window with per-pane dir
- **GIVEN** the user runs `dev-windows add -n my-app -d ~/repos/my-app --p2-dir ~/repos/shared`
- **WHEN** the command completes
- **THEN** `windows.yaml` records `panes: {P2: {dir: ~/repos/shared}}` for `my-app`

#### Scenario: Adding a window with per-pane cmd
- **GIVEN** the user runs `dev-windows add -n my-app -d ~/repos/my-app --p0-cmd "tail -f logs/app.log"`
- **WHEN** the command completes
- **THEN** `windows.yaml` records `panes: {P0: {cmd: tail -f logs/app.log}}` for `my-app`

#### Scenario: Combining label, dir, and cmd flags for same pane
- **GIVEN** the user runs `dev-windows add -n svc -d ~/repos/svc --p0-dir ~/repos/svc/logs --p0-cmd "tail -f app.log"`
- **WHEN** the command completes
- **THEN** `windows.yaml` records `panes: {P0: {dir: ~/repos/svc/logs, cmd: tail -f app.log}}` for `svc`
