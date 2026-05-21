# Dev-Windows Specification

## Purpose

`dev-windows` is a Python CLI tool that generates machine-local tmux window configuration
files from a YAML manifest (`~/.tmux/local/windows.yaml`) using Jinja2 templates. It also
manages a `bindings.conf` file of tmux command-aliases so that each generated window config
can be loaded from the tmux command prompt. The tool eliminates hand-editing repetitive tmux
script boilerplate while keeping window definitions in a single, version-control-friendly YAML
file.

---

## Requirements

### Requirement: File Locations

The system SHALL resolve all runtime paths from the following defaults, each of which MAY be
overridden by the `TMUX_LOCAL_DIR` environment variable.

| Path                                   | Purpose                                             |
|----------------------------------------|-----------------------------------------------------|
| `~/.tmux/local/`                       | Local directory root (`TMUX_LOCAL_DIR`)             |
| `~/.tmux/local/windows.yaml`           | YAML manifest of configured windows                 |
| `~/.tmux/local/bindings.conf`          | tmux command-alias file managed by the tool         |
| `~/.tmux/local/<name>`                 | Generated tmux script for window `<name>`           |
| `~/.tmux/templates/`                   | Shared Jinja2 templates directory                   |
| `~/.tmux/local/templates/`            | Machine-local Jinja2 template overrides             |

#### Scenario: Default path resolution
- GIVEN the `TMUX_LOCAL_DIR` environment variable is not set
- WHEN any command is executed
- THEN the tool uses `~/.tmux/local/` as the local directory root

#### Scenario: Override via environment variable
- GIVEN `TMUX_LOCAL_DIR=/custom/path` is set in the environment
- WHEN any command is executed
- THEN the tool resolves all local paths under `/custom/path/`

---

### Requirement: Template Discovery

The system SHALL discover Jinja2 templates by scanning `~/.tmux/local/templates/` (local
overrides) and `~/.tmux/templates/` (shared), in that priority order.

#### Scenario: Local template overrides shared template
- GIVEN a file `~/.tmux/local/templates/default.j2` exists
- AND a file `~/.tmux/templates/default.j2` also exists
- WHEN the template named `default` is requested
- THEN the local template is used and the shared template is ignored

#### Scenario: Shared template used when no local override exists
- GIVEN no file `~/.tmux/local/templates/custom.j2` exists
- AND a file `~/.tmux/templates/custom.j2` exists
- WHEN the template named `custom` is requested
- THEN the shared template is used

#### Scenario: Template not found
- GIVEN a template name `nonexistent` has no corresponding `.j2` file in either template directory
- WHEN any command tries to render that template
- THEN the tool exits with an error message naming the missing template and both search paths

---

### Requirement: Template Variables

The system SHALL pass the following variables to every Jinja2 template during rendering.

| Variable | Source                                                                 |
|----------|------------------------------------------------------------------------|
| `name`   | The window key from `windows.yaml`                                     |
| `dir`    | The `dir` field of the window entry                                    |
| `init`   | Shell initialization string derived from `dir` (see Init String below) |
| `L0`     | `panes.L0` from the entry, or empty string if absent                   |
| `L1`     | `panes.L1` from the entry, or empty string if absent                   |
| `L2`     | `panes.L2` from the entry, or empty string if absent                   |
| `L3`     | `panes.L3` from the entry, or empty string if absent                   |

#### Scenario: Pane labels passed through
- GIVEN a window entry defines `panes: {L0: Watch, L1: Logs}`
- WHEN the template is rendered
- THEN the template variable `L0` equals `Watch` and `L1` equals `Logs`

#### Scenario: Absent pane labels default to empty string
- GIVEN a window entry defines no `panes` key
- WHEN the template is rendered
- THEN `L0`, `L1`, `L2`, and `L3` are all empty strings

---

### Requirement: Init String Derivation

The system SHALL derive the `init` variable from the window's `dir` field using the following
logic: if `<dir>/envsetup.sh` exists on the filesystem, `init` is
`<dir> && source envsetup.sh && clear`; otherwise `init` is `cd <dir> && clear`.

#### Scenario: Directory contains envsetup.sh
- GIVEN a window entry with `dir: ~/repos/my-project`
- AND `~/repos/my-project/envsetup.sh` exists
- WHEN the template is rendered
- THEN `init` equals `~/repos/my-project && source envsetup.sh && clear`

#### Scenario: Directory does not contain envsetup.sh
- GIVEN a window entry with `dir: ~/repos/my-project`
- AND `~/repos/my-project/envsetup.sh` does not exist
- WHEN the template is rendered
- THEN `init` equals `cd ~/repos/my-project && clear`

---

### Requirement: Built-in Templates

The system SHALL ship five built-in Jinja2 templates with the following pane layouts and
default labels.

#### Scenario: `default` template — 6 panes
- GIVEN the `default` template is rendered
- THEN the generated script creates 6 panes with default labels:
  - Pane 0: `L0` or `Watch`
  - Pane 1: `L1` or `Logs`
  - Pane 2: `L2` (label only emitted when `L2` is non-empty)
  - Pane 3: `L3` or `Tests`
  - Pane 4: `Editor`
  - Pane 5: `Console`
- AND pane 4 runs `{{ init }} && nvim .`
- AND all other panes run `{{ init }}`

#### Scenario: `dev-4pane` template — 4 panes
- GIVEN the `dev-4pane` template is rendered
- THEN the generated script creates 4 panes with default labels:
  - Pane 0: `L0` or `Log`
  - Pane 1: `L1` or `Debug`
  - Pane 2: `Editor`
  - Pane 3: `Console`
- AND pane 2 runs `{{ init }} && nvim .`

#### Scenario: `simple` template — 3 panes
- GIVEN the `simple` template is rendered
- THEN the generated script creates 3 panes with no configurable labels
- AND pane 1 runs `{{ init }} && nvim .`
- AND panes 0 and 2 run `{{ init }}`

#### Scenario: `monitoring` template — 7 panes
- GIVEN the `monitoring` template is rendered
- THEN the generated script creates 7 panes with default labels:
  - Pane 0: `L0` or `Log`
  - Pane 1: `L1` or `Misc`
  - Pane 2: `L2` or `Monitoring`
  - Pane 4: `Editor`
  - Pane 5: `Console`
  - Pane 6: `Working`
- AND pane 4 runs `{{ init }} && nvim .`

#### Scenario: `notebook` template — 3 panes
- GIVEN the `notebook` template is rendered
- THEN the generated script creates 3 panes with default labels:
  - Pane 0: `L0` or `Server`
  - Pane 1: `Editor`
  - Pane 2: `Console`
- AND pane 1 runs `{{ init }} && nvim .`

---

### Requirement: `generate` Command

The system SHALL re-render and write all window config files declared in `windows.yaml`, and
update `bindings.conf` for each, when `dev-windows generate` is invoked.

#### Scenario: Successful generation
- GIVEN `~/.tmux/local/windows.yaml` contains two window entries `foo` and `bar`
- WHEN `dev-windows generate` is run
- THEN `~/.tmux/local/foo` and `~/.tmux/local/bar` are written with rendered content
- AND a binding entry is added or updated in `bindings.conf` for each window
- AND the tool prints the count of windows being generated

#### Scenario: Empty config file
- GIVEN `~/.tmux/local/windows.yaml` is empty or does not exist
- WHEN `dev-windows generate` is run
- THEN the tool exits with an error indicating no windows are configured

---

### Requirement: `add` Command

The system SHALL add or update a single window entry in `windows.yaml`, write its config
file, and add or update its binding, when `dev-windows add` is invoked.

#### Scenario: Adding a new window
- GIVEN `windows.yaml` does not contain an entry named `my-app`
- WHEN `dev-windows add -n my-app -d ~/repos/my-app` is run
- THEN `my-app` is written to `windows.yaml` with `dir: ~/repos/my-app` and `template: default`
- AND `~/.tmux/local/my-app` is written with the rendered template
- AND a binding is added to `bindings.conf` for `my-app`
- AND the output says `Added 'my-app' in <path>`

#### Scenario: Updating an existing window
- GIVEN `windows.yaml` already contains an entry named `my-app`
- WHEN `dev-windows add -n my-app -d ~/repos/my-app-v2` is run
- THEN the `my-app` entry in `windows.yaml` is replaced with the new values
- AND the output says `Updated 'my-app' in <path>`

#### Scenario: Specifying a non-default template
- GIVEN the user runs `dev-windows add -n svc -d ~/repos/svc -t dev-4pane`
- WHEN the command completes
- THEN `windows.yaml` records `template: dev-4pane` for `svc`
- AND the generated file uses the `dev-4pane` template

#### Scenario: Specifying pane labels
- GIVEN the user runs `dev-windows add -n svc -d ~/repos/svc --l0 Watch --l3 Tests`
- WHEN the command completes
- THEN `windows.yaml` records `panes: {L0: Watch, L3: Tests}` for `svc`
- AND absent pane options (`--l1`, `--l2`) are omitted from the stored entry

---

### Requirement: `remove` Command

The system SHALL delete a window entry from `windows.yaml`, delete its generated config
file, and remove its binding from `bindings.conf`, when `dev-windows remove` is invoked.

#### Scenario: Removing an existing window
- GIVEN `windows.yaml` contains an entry named `old-app`
- AND `~/.tmux/local/old-app` exists
- WHEN `dev-windows remove old-app` is run
- THEN `old-app` is deleted from `windows.yaml`
- AND `~/.tmux/local/old-app` is deleted from the filesystem
- AND the binding for `old-app` is removed from `bindings.conf`

#### Scenario: Removing a window not in config
- GIVEN `windows.yaml` does not contain an entry named `ghost`
- WHEN `dev-windows remove ghost` is run
- THEN the tool exits with an error indicating `ghost` was not found

#### Scenario: Binding not in bindings.conf during remove
- GIVEN `windows.yaml` contains `old-app`
- AND `bindings.conf` does not contain a binding for `old-app`
- WHEN `dev-windows remove old-app` is run
- THEN the tool emits a warning to stderr that the binding was not found
- AND the rest of removal (config and file) still proceeds

---

### Requirement: `list` Command

The system SHALL print a summary of all configured windows when `dev-windows list` is
invoked.

#### Scenario: Windows are configured
- GIVEN `windows.yaml` contains entries `foo` and `bar`
- WHEN `dev-windows list` is run
- THEN each entry is printed on its own line showing the window name, directory, and template
- AND if an entry has pane labels defined, they are printed on a second indented line

#### Scenario: No windows configured
- GIVEN `windows.yaml` is absent or empty
- WHEN `dev-windows list` is run
- THEN the tool prints a message indicating no windows are configured and names the config file

---

### Requirement: `templates` Command

The system SHALL list all available template names (without the `.j2` extension) from both
template directories when `dev-windows templates` is invoked.

#### Scenario: Templates exist
- GIVEN `~/.tmux/templates/` contains `default.j2`, `simple.j2`
- AND `~/.tmux/local/templates/` contains `custom.j2`
- WHEN `dev-windows templates` is run
- THEN `custom`, `default`, and `simple` are printed (each on its own line)

#### Scenario: Local template shadows shared template in listing
- GIVEN both `~/.tmux/local/templates/default.j2` and `~/.tmux/templates/default.j2` exist
- WHEN `dev-windows templates` is run
- THEN `default` appears exactly once in the output

#### Scenario: No templates found
- GIVEN neither template directory exists or contains `.j2` files
- WHEN `dev-windows templates` is run
- THEN the tool prints "No templates found."

---

### Requirement: `bindings.conf` Management

The system SHALL manage `set -s command-alias[<index>]` entries in `bindings.conf`
following these rules.

#### Scenario: Adding a new binding
- GIVEN `bindings.conf` exists and contains no binding for `new-win`
- WHEN a binding is added for `new-win`
- THEN a new line `set -s command-alias[<N>] new-win='source-file ~/.tmux/local/new-win'` is
  appended, where `<N>` is one greater than the highest existing index, or 100 if no
  command-alias entries exist

#### Scenario: Updating an existing binding
- GIVEN `bindings.conf` already contains `set -s command-alias[123] old-win='source-file ...'`
- WHEN a binding is added for `old-win` (e.g. after a path change)
- THEN the existing line is replaced in-place, preserving index `123`
- AND the output indicates the binding was updated with its index

#### Scenario: bindings.conf absent — add is a no-op
- GIVEN `bindings.conf` does not exist
- WHEN a binding add or remove operation is attempted
- THEN the operation silently succeeds without creating the file

#### Scenario: Removing an existing binding
- GIVEN `bindings.conf` contains a binding for `old-win`
- WHEN `remove_binding("old-win")` is called
- THEN the entire line for `old-win` is deleted from `bindings.conf`

#### Scenario: Index allocation starts at 100 when no aliases exist
- GIVEN `bindings.conf` exists but contains no `command-alias` lines
- WHEN the first binding is added for `first-win`
- THEN the new entry uses index `100`

---

### Requirement: Generated File Format

Each generated config file SHALL begin with a `# vim: set filetype=tmux:` modeline and a
`# Managed by dev-windows` comment, followed by valid tmux commands that use
`set-option -p -t .<N> @pane_label '<label>'` to assign pane labels.

#### Scenario: Generated file is a valid tmux source-file
- GIVEN a window `my-app` with `dir: ~/repos/my-app` is configured
- WHEN the config file is generated
- THEN the file at `~/.tmux/local/my-app` starts with `# vim: set filetype=tmux:`
- AND contains a `new-window -n my-app -c '~/repos/my-app'` command
- AND uses `set-option -p` with the `@pane_label` user option to label panes

---

### Requirement: windows.yaml Persistence

The system SHALL persist `windows.yaml` using block-style YAML (no flow style), preserving
key insertion order and allowing Unicode values.

#### Scenario: File written with correct YAML style
- GIVEN a new entry is added via `add` or `generate`
- WHEN `windows.yaml` is written
- THEN the file uses block YAML format (not inline/flow style)
- AND the key order matches the order entries were inserted

#### Scenario: Parent directory created automatically
- GIVEN `~/.tmux/local/` does not yet exist
- WHEN `save_config` is called for the first time
- THEN the directory (and any missing parents) is created before writing the file
