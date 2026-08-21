## MODIFIED Requirements

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
  - Pane 5: `AI`
- AND pane 4 runs `{{ init }} && nvim .`
- AND all other panes run `{{ init }}`

#### Scenario: `dev-4pane` template — 4 panes
- GIVEN the `dev-4pane` template is rendered
- THEN the generated script creates 4 panes with default labels:
  - Pane 0: `L0` or `Log`
  - Pane 1: `L1` or `Debug`
  - Pane 2: `Editor`
  - Pane 3: `AI`
- AND pane 2 runs `{{ init }} && nvim .`

#### Scenario: `simple` template — 3 panes
- GIVEN the `simple` template is rendered
- THEN the generated script creates 3 panes with no configurable labels, but fixed labels
  on two of them:
  - Pane 0: (unlabeled)
  - Pane 1: `Editor`
  - Pane 2: `AI`
- AND pane 1 runs `{{ init }} && nvim .`
- AND panes 0 and 2 run `{{ init }}`

#### Scenario: `monitoring` template — 6 panes
- GIVEN the `monitoring` template is rendered
- THEN the generated script creates 6 panes with default labels:
  - Pane 0: `L0` or `Log`
  - Pane 1: `L1` or `Misc`
  - Pane 2: `L2` or `Monitoring`
  - Pane 3: `Console`
  - Pane 4: `Editor`
  - Pane 5: `AI`
- AND pane 4 runs `{{ init }} && nvim .`

#### Scenario: `notebook` template — 3 panes
- GIVEN the `notebook` template is rendered
- THEN the generated script creates 3 panes with default labels:
  - Pane 0: `L0` or `Server`
  - Pane 1: `Editor`
  - Pane 2: `AI`
- AND pane 1 runs `{{ init }} && nvim .`
