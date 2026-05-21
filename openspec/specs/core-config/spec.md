# Core Config Specification

## Purpose

This domain defines the global tmux runtime configuration: prefix key, history, terminal capabilities, Vim-style keybindings, Powerline status bar integration, clipboard interoperability, and pane/window navigation conventions. It establishes the baseline behavior that all other window and session configurations depend on.

## Requirements

### Requirement: Prefix Key

The system SHALL use `C-a` as the tmux prefix key, replacing the default `C-b`.

#### Scenario: Prefix key active
- GIVEN a running tmux session
- WHEN the user presses `C-a` followed by any bound key
- THEN tmux interprets the sequence as a prefixed command

#### Scenario: Nested session passthrough
- GIVEN tmux is running inside another tmux session
- WHEN the user presses `prefix + a`
- THEN the `C-a` keystroke is forwarded to the inner tmux session

---

### Requirement: History Limit

The system SHALL retain up to 50,000 lines of scrollback history per pane.

#### Scenario: Long-running command output
- GIVEN a pane with extensive command output
- WHEN the user enters copy mode and scrolls back
- THEN up to 50,000 lines of history are accessible

---

### Requirement: True Color Terminal Support

The system SHALL configure the terminal to support 24-bit true color.

#### Scenario: True color rendering
- GIVEN the `$TERM` environment variable is set at session start
- WHEN tmux sets `default-terminal` and appends `Tc` to `terminal-overrides`
- THEN applications inside tmux receive 24-bit color capability

---

### Requirement: Escape-Time Optimization

The system SHALL set escape-time to 0 milliseconds to eliminate Vim mode-switching delay.

#### Scenario: Vim escape key responsiveness
- GIVEN a pane running Vim in insert mode
- WHEN the user presses `Escape`
- THEN Vim returns to normal mode without perceptible delay

---

### Requirement: Focus Events

The system SHALL enable focus-events so applications receive terminal focus in/out notifications.

#### Scenario: Application focus tracking
- GIVEN a pane running a focus-aware application (e.g., Vim with autoread)
- WHEN the tmux window gains or loses focus
- THEN the application receives the corresponding focus event

---

### Requirement: Mouse Support

The system SHALL enable full mouse support for pane selection, scrolling, and resizing.

#### Scenario: Pane selection via mouse click
- GIVEN multiple panes are visible
- WHEN the user clicks inside a pane
- THEN that pane becomes the active pane

#### Scenario: Scrollback via mouse wheel
- GIVEN a pane with scrollback history
- WHEN the user scrolls the mouse wheel up
- THEN the pane enters scroll mode and scrolls through history

---

### Requirement: Vim Copy Mode

The system SHALL use vi key bindings in copy mode, with `v` to begin selection and `y` to yank.

#### Scenario: Begin selection in copy mode
- GIVEN the user has entered copy mode
- WHEN the user presses `v`
- THEN text selection begins at the cursor position

#### Scenario: Yank to clipboard on macOS
- GIVEN the user has a selection active in copy mode
- WHEN the user presses `y`
- THEN the selected text is piped through `reattach-to-user-namespace pbcopy` and placed in the macOS clipboard

---

### Requirement: Paste from macOS Clipboard

The system SHALL bind `prefix + ]` on macOS to paste from the system clipboard via `pbpaste`.

#### Scenario: Paste from system clipboard
- GIVEN text has been copied to the macOS clipboard outside of tmux
- WHEN the user presses `prefix + ]` on a macOS host
- THEN the clipboard contents are loaded into the tmux paste buffer and pasted into the active pane

---

### Requirement: Powerline Status Bar

The system SHALL load and display a Powerline status bar at the top of the screen, refreshing every 5 seconds.

#### Scenario: Powerline daemon startup
- GIVEN `POWERLINE_DIR` is set in the environment
- WHEN tmux starts
- THEN `powerline-daemon -q` is launched and the Powerline tmux bindings are sourced from `$POWERLINE_DIR/bindings/tmux/powerline.conf`

#### Scenario: Status bar position
- GIVEN Powerline is loaded
- WHEN the tmux status bar is rendered
- THEN it appears at the top of the terminal

#### Scenario: Status bar refresh interval
- GIVEN the tmux session is running
- WHEN 5 seconds elapse
- THEN the status bar content is refreshed

---

### Requirement: Terminal Title

The system SHALL set the terminal window title to reflect the current host, session, window, pane, and program.

#### Scenario: Title reflects session context
- GIVEN a terminal that supports title setting
- WHEN a tmux window is active
- THEN the terminal title displays `<hostname>:<session>.<window>.<pane> <window-name> <pane-title>`

---

### Requirement: Automatic Window Rename

The system SHALL automatically rename windows to reflect the currently running program.

#### Scenario: Window renamed on program change
- GIVEN a window running a shell
- WHEN the user launches a program (e.g., `vim`)
- THEN the window name updates to reflect the running program

---

### Requirement: Activity Monitoring

The system SHALL monitor all windows for activity and indicate it visually.

#### Scenario: Background window activity
- GIVEN a window that is not currently visible
- WHEN output appears in that window
- THEN the status bar highlights the window to indicate activity

---

### Requirement: Pane Border Labels

The system SHALL display pane index and label (or title) in the pane border at the bottom of each pane.

#### Scenario: Pane with a custom label
- GIVEN a pane has a user option `@pane_label` set
- WHEN the pane border is rendered
- THEN the border displays `<pane_index> <@pane_label>`

#### Scenario: Pane without a custom label
- GIVEN a pane has no `@pane_label` user option
- WHEN the pane border is rendered
- THEN the border displays `<pane_index> <pane_title>`

---

### Requirement: Aggressive Resize

The system SHALL use aggressive-resize so each window is only constrained by clients actively viewing it, not all clients in the session.

#### Scenario: Multiple clients at different windows
- GIVEN two clients connected to the same session viewing different windows
- WHEN the smaller client is not viewing a particular window
- THEN that window is sized to the larger client's terminal dimensions

---

### Requirement: Pane Splitting

The system SHALL bind `prefix + -` to create a vertical split and `prefix + \` to create a horizontal split in the current pane's directory context.

#### Scenario: Vertical split
- GIVEN an active pane
- WHEN the user presses `prefix + -`
- THEN the pane is split vertically (top/bottom)

#### Scenario: Horizontal split
- GIVEN an active pane
- WHEN the user presses `prefix + \`
- THEN the pane is split horizontally (left/right)

---

### Requirement: Pane Navigation

The system SHALL bind `prefix + h/j/k/l` to move between panes in Vim directional convention.

#### Scenario: Move left
- GIVEN multiple panes are visible
- WHEN the user presses `prefix + h`
- THEN focus moves to the pane to the left

#### Scenario: Move down
- GIVEN multiple panes are visible
- WHEN the user presses `prefix + j`
- THEN focus moves to the pane below

#### Scenario: Move up
- GIVEN multiple panes are visible
- WHEN the user presses `prefix + k`
- THEN focus moves to the pane above

#### Scenario: Move right
- GIVEN multiple panes are visible
- WHEN the user presses `prefix + l`
- THEN focus moves to the pane to the right

---

### Requirement: Pane Promotion and Join

The system SHALL bind `prefix + Enter` to break the active pane into its own window, and `prefix + Space` to join a pane from another window.

#### Scenario: Break pane to window
- GIVEN a split layout with multiple panes
- WHEN the user presses `prefix + Enter`
- THEN the active pane is promoted to a new window

#### Scenario: Join pane from another window
- GIVEN multiple windows exist
- WHEN the user presses `prefix + Space` and supplies a `window.pane` target at the prompt
- THEN the specified pane is joined into the current window

---

### Requirement: Window Navigation

The system SHALL bind `prefix + C-j` / `prefix + C-k` for previous/next window, and `C-left` / `C-right` (no prefix) for the same.

#### Scenario: Next window with prefix
- GIVEN multiple windows in the session
- WHEN the user presses `prefix + C-k`
- THEN focus moves to the next window

#### Scenario: Previous window without prefix
- GIVEN multiple windows in the session
- WHEN the user presses `C-left` (no prefix)
- THEN focus moves to the previous window

#### Scenario: Last active window
- GIVEN the user was previously viewing a different window
- WHEN the user presses `prefix + C-a`
- THEN focus returns to the last active window

---

### Requirement: Window Rename

The system SHALL bind `prefix + A` to prompt the user to rename the current window.

#### Scenario: Rename window
- GIVEN an active window
- WHEN the user presses `prefix + A` and enters a name
- THEN the window is renamed to the entered value

---

### Requirement: Layout Management

The system SHALL provide bindings to switch between predefined pane layouts and rotate panes.

#### Scenario: Active-only layout
- GIVEN multiple panes in a window
- WHEN the user presses `prefix + o`
- THEN only the active pane is shown (maximized)

#### Scenario: Even vertical layout
- GIVEN multiple panes in a window
- WHEN the user presses `prefix + M--`
- THEN panes are arranged in an even vertical (top/bottom) layout

#### Scenario: Even horizontal layout
- GIVEN multiple panes in a window
- WHEN the user presses `prefix + M-|`
- THEN panes are arranged in an even horizontal (left/right) layout

#### Scenario: Rotate window panes
- GIVEN multiple panes in a window
- WHEN the user presses `prefix + M-r`
- THEN panes are rotated within the window

#### Scenario: Cycle to next layout
- GIVEN a window with panes
- WHEN the user presses `prefix + u`
- THEN tmux cycles to the next built-in layout

---

### Requirement: Config Reload

The system SHALL bind `prefix + r` to reload `~/.tmux.conf` without restarting the server.

#### Scenario: Live config reload
- GIVEN the user has edited `~/.tmux.conf`
- WHEN the user presses `prefix + r`
- THEN the configuration is re-sourced and changes take effect in the running session

---

### Requirement: Command Aliases

The system SHALL define command aliases to load shared window configurations by short name.

#### Scenario: Load irssi window config
- GIVEN the user is in the tmux command prompt
- WHEN the user types `irssi` and presses Enter
- THEN `~/.tmux/irssi` is sourced

#### Scenario: Available aliases
- GIVEN a running tmux session
- WHEN the user enters the command prompt (`prefix + :`)
- THEN the aliases `irssi`, `vim-config`, `tmux-dev`, and `notes` are available to source their respective window configs

---

### Requirement: Local Bindings Override

The system SHALL source `~/.tmux/local/bindings.conf` if it exists, allowing machine-specific key binding overrides.

#### Scenario: Local bindings present
- GIVEN the file `~/.tmux/local/bindings.conf` exists on the host
- WHEN tmux loads the configuration
- THEN the local bindings file is sourced after the global bindings

#### Scenario: Local bindings absent
- GIVEN the file `~/.tmux/local/bindings.conf` does not exist
- WHEN tmux loads the configuration
- THEN no error occurs and global bindings remain unchanged

---

### Requirement: Status Key Mode

The system SHALL use Emacs key bindings in the tmux command prompt.

#### Scenario: Command prompt navigation
- GIVEN the user has opened the tmux command prompt with `prefix + :`
- WHEN the user presses Emacs movement keys (e.g., `C-a`, `C-e`, `C-k`)
- THEN the prompt responds with Emacs-style line editing behavior

---

### Requirement: Message Display Duration

The system SHALL display tmux messages for 4 seconds.

#### Scenario: Status message visibility
- GIVEN tmux emits an informational message (e.g., after config reload)
- WHEN the message appears in the status bar
- THEN it remains visible for 4 seconds before disappearing

---

### Requirement: xterm Key Compatibility

The system SHALL enable xterm-keys so that extended key sequences (used by Vim and other TUI apps) are passed through correctly.

#### Scenario: Extended key sequences in Vim
- GIVEN a pane running Vim
- WHEN the user presses function keys or modified arrow keys
- THEN the correct xterm key sequences are delivered to Vim
