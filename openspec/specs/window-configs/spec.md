# Window Configs Specification

## Purpose

This domain provides static tmux scripts that define reusable named window layouts for specific workflows (development, IRC, notes, vim configuration). Each script creates a window with a fixed pane arrangement, sets working directories per pane, and fires initialization commands. Session templates compose multiple window configs into a full named session by sourcing them in sequence.

## Requirements

### Requirement: Vim Modeline Header
Every window config file SHALL begin with the comment `# vim: set filetype=tmux:` as its first line to enable tmux syntax highlighting in Vim/Neovim.

#### Scenario: Config file opened in Vim
- GIVEN a window config file is opened in Vim or Neovim
- WHEN the editor reads the modeline on line 1
- THEN syntax highlighting is applied using the `tmux` filetype

---

### Requirement: Window Naming
Each window config SHALL assign a unique, descriptive name to the window it creates via the `-n` flag.

#### Scenario: Window config is sourced
- GIVEN a window config script is sourced inside a running tmux session
- WHEN the `new-window` (or `new`) command executes
- THEN the new window is identifiable by its assigned name in the tmux window list

#### Scenario: Default dev session window name
- GIVEN the `default-dev-session` config is sourced
- WHEN the session is created
- THEN the initial window is named `linux-dev`

#### Scenario: IRSSI window name
- GIVEN the `irssi` config is sourced
- WHEN the window is created
- THEN the window is named `IRSSI`

#### Scenario: tmux-dev window name
- GIVEN the `tmux-dev` config is sourced
- WHEN the window is created
- THEN the window is named `tmux-dev`

#### Scenario: notes window name
- GIVEN the `notes` config is sourced
- WHEN the window is created
- THEN the window is named `notes`

#### Scenario: vim-config window name
- GIVEN the `vim-config` config is sourced
- WHEN the window is created
- THEN the window is named `vim-config`

---

### Requirement: Working Directory at Window Creation
Each window config SHALL set the working directory (`-c`) at window creation time so that new panes inherit a sensible default directory.

#### Scenario: default-dev-session working directory
- GIVEN the `default-dev-session` config is sourced
- WHEN the session window is created
- THEN the initial working directory is `$HOME`

#### Scenario: notes working directory
- GIVEN the `notes` config is sourced
- WHEN the window is created
- THEN the initial working directory is `$HOME/vaults/`

#### Scenario: vim-config working directory
- GIVEN the `vim-config` config is sourced
- WHEN the window is created
- THEN the initial working directory is `$HOME/.homesick/repos/AstroNvim_config/`

---

### Requirement: Two-Column Pane Layout (Standard)
Window configs with a two-column layout SHALL create a primary right-hand column by splitting horizontally first, then optionally split the right column vertically.

#### Scenario: IRSSI two-column layout
- GIVEN the `irssi` config is sourced
- WHEN layout commands execute
- THEN pane 0 occupies the left half (50%) and pane 1 occupies the right half
- AND pane 1 is split vertically into panes 1 and 2 at 50% of the right column

#### Scenario: tmux-dev two-column layout
- GIVEN the `tmux-dev` config is sourced
- WHEN layout commands execute
- THEN pane 0 occupies the left half (50%) and pane 1 occupies the right half
- AND pane 1 is split vertically into panes 1 and 2 at 50% of the right column

#### Scenario: vim-config two-column layout
- GIVEN the `vim-config` config is sourced
- WHEN layout commands execute
- THEN pane 0 occupies the left half (50%) and pane 1 occupies the right half
- AND pane 1 is split vertically into panes 1 and 2 at 50% of the right column

---

### Requirement: Asymmetric Pane Layout (Wide Left)
Window configs that follow a wide-left layout SHALL allocate a larger left column (70%) for the primary editing pane and a narrower right column for auxiliary panes.

#### Scenario: default-dev-session asymmetric layout
- GIVEN the `default-dev-session` config is sourced
- WHEN layout commands execute
- THEN pane 1 (right column) occupies 70% of the window width
- AND pane 0 (left column) is split vertically three times into fixed-height strips (10, 10, 5 lines)
- AND pane 1 is split vertically once into a 5-line strip

#### Scenario: notes asymmetric layout
- GIVEN the `notes` config is sourced
- WHEN layout commands execute
- THEN pane 1 (right column) occupies 70% of the window width
- AND pane 1 is split vertically with a 20% bottom strip

---

### Requirement: Pane Initialization Commands
Each window config SHALL send initialization commands to relevant panes immediately after the layout is established.

#### Scenario: default-dev-session pane commands
- GIVEN the `default-dev-session` layout is complete
- WHEN initialization commands are sent
- THEN pane 0 runs `top`
- AND pane 2 changes directory to `~/Desktop`
- AND pane 1 opens `nvim .` in the working directory
- AND pane 4 runs `tail -f` (awaiting a filename argument)

#### Scenario: IRSSI pane commands
- GIVEN the `irssi` layout is complete
- WHEN initialization commands are sent
- THEN pane 0 launches `irssi` from `$HOME`
- AND pane 1 opens `$HOME/.irssirc` in nvim
- AND pane 2 opens the irssi `config` file in nvim from `$HOME/.irssi/`

#### Scenario: tmux-dev pane commands
- GIVEN the `tmux-dev` layout is complete
- WHEN initialization commands are sent
- THEN pane 0 opens `nvim .` inside `$HOME/.tmux`
- AND pane 1 opens `$HOME/.tmux.conf` in nvim
- AND pane 2 changes directory to `$HOME` (shell ready)

#### Scenario: notes pane commands
- GIVEN the `notes` layout is complete
- WHEN initialization commands are sent
- THEN pane 0 changes directory to `$HOME/vaults`
- AND pane 1 opens nvim inside `$HOME/vaults`
- AND pane 2 changes directory to `$HOME/vaults` (shell ready)

#### Scenario: vim-config pane commands
- GIVEN the `vim-config` layout is complete
- WHEN initialization commands are sent
- THEN pane 0 opens `nvim .` inside `$HOME/.config/nvim`
- AND pane 1 opens `nvim .` inside `$HOME/.homesick/repos/AstroNvim_config`
- AND pane 2 navigates to `$HOME/.homesick/repos/AstroNvim_config` (shell ready)

---

### Requirement: Active Pane Selection
A window config MAY set the active pane at the end of initialization so the user lands on the most relevant pane.

#### Scenario: default-dev-session active pane
- GIVEN the `default-dev-session` initialization is complete
- WHEN all send-keys commands have run
- THEN focus is placed on pane 1 (the nvim editor pane) via `select-pane -t 1`

---

### Requirement: Session Template Composition
A session template SHALL create a named session and compose its windows by sourcing individual window config files in sequence.

#### Scenario: Template session creation
- GIVEN the `template-session` file is used as a tmux session template
- WHEN `tmux source-file` is called on it
- THEN a new session named `Template` is created with `$HOME` as the base directory
- AND window configs are loaded by sourcing each local config file in order
- AND the default empty window (index 0) is removed via `kill-window -t 0` after the first window config is sourced

#### Scenario: Default window removal
- GIVEN a session template creates a new session (which always starts with one empty window at index 0)
- WHEN the first window config is sourced
- THEN the empty default window is killed to leave only the windows defined by the configs

---

### Requirement: Window Config Scope
Shared window configs SHALL use `new-window` to add a window to the current session. Session templates SHALL use `new` (i.e., `new-session`) to create an entire session.

#### Scenario: Shared config sourced into existing session
- GIVEN any of `irssi`, `tmux-dev`, `notes`, or `vim-config` is sourced
- WHEN the script runs
- THEN a new window is appended to the current session using `new-window`

#### Scenario: Session template creates a session
- GIVEN the `template-session` (or `default-dev-session`) is sourced
- WHEN the script runs
- THEN a new tmux session is created using the `new` command with a session name argument
