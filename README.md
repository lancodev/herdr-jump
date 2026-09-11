# Jump for herdr

A two-pane fuzzy switcher for [herdr](https://herdr.dev): **workspaces on the
left, agents on the right**. Move, filter, manage workspaces, and jump straight
to an agent's pane.

herdr's built-in session navigator is one tree of every workspace, tab and
pane. It cannot be scoped — it re-expands every workspace each time it opens,
so collapsing a row with <kbd>Space</kbd> never sticks. Jump is the flat
alternative: pick a workspace, or cross over and pick an agent directly.

```
╭──────── workspaces ─────────╮╭────────────────────── agents ──────────────────────╮
│ *  1  ●  dotfiles  working││ * ●  dotfiles   t1     opencode › README   working│
│    2  ○  api          idle││   ●  api        t2/p1  claude › billing   blocked│
│    3  ·  web          none││   ○  web        t3     opencode › tests       idle│
╰─────────────────────────────╯╰───────────────────────────────────────────────────────╯
 * current · ● blocked · ● working · ● done · ○ idle · · none
```

- By default, both panes are visible at normal widths; <kbd>h</kbd>/<kbd>l</kbd>,
  <kbd>Tab</kbd>, and <kbd>Shift-Tab</kbd> switch the active pane
- Accent `*` marks the current location; <kbd>Enter</kbd> jumps to the
  highlighted workspace or agent
- Agent rows include workspace, tab (`t2`, or `t2/p1` for a split), identity,
  terminal title, and status
- <kbd>J</kbd>/<kbd>K</kbd> reorder workspaces; <kbd>r</kbd>/<kbd>F2</kbd>,
  <kbd>a</kbd>, and <kbd>d</kbd> rename, create, and close them in place
- Status filters narrow either list, and the agents pane can be scoped by
  workspace, without changing the search query
- Jump history supports a previous-target key and optional recently visited
  ordering
- The bottom legend uses herdr's status colors and the terminal theme

Below about 100 popup columns, Jump degrades to one full-width active list and
hides the other pane. Pane switching still works, and so do <kbd>?</kbd> help
and <kbd>o</kbd> output — they open a bottom split instead of the normal side
one. Only `preview = false` (see Configure) disables them, not narrow width.

## Status glyphs

Each row carries the agent status herdr reports, in the same colors as its
sidebar:

| Glyph | Status |
|---|---|
| red `●` | blocked — the agent is waiting on you (a permission prompt, a question) |
| yellow `●` | working |
| cyan `●` | done — finished and not yet looked at |
| green `○` | idle |
| dim `·` | no agent, or a status herdr does not know |

Workspaces show the rolled-up status of the agents inside them. Accent `*`
marks the current workspace or agent. Each row also ends with a dim status word
(`blocked`, `working`, `done`, `idle`, or `none`); it makes status searchable as
text and is separate from the colored glyph. A label or terminal title long
enough to be truncated in the row itself is also appended in full afterward,
dimmed, so search still matches text that scrolled out of the visible column.

## Requirements

[`fzf`](https://github.com/junegunn/fzf) 0.74.0 or newer and
[`jq`](https://jqlang.org), both on `PATH`.

`nc` with Unix-socket support or `socat` is optional. herdr ships no CLI wrapper
for `workspace.move` or for focusing a pane by id, so those two calls go to the
socket directly. Without either tool the picker still switches normally; it
drops workspace reordering, and an agent sharing a split tab lands on that
tab's focused pane rather than the agent's exact pane.

## Install

```sh
herdr plugin install lancodev/herdr-jump
```

or for a local checkout:

```sh
herdr plugin link /path/to/herdr-jump
```

Then bind the action in `~/.config/herdr/config.toml`. `prefix+g` is the
natural home, which means unbinding herdr's own navigator:

```toml
[keys]
goto = ""

[[keys.command]]
key = "prefix+g"
type = "plugin_action"
command = "lancodev.jump.open"
description = "jump to a workspace or agent"
```

Reload with `prefix+r` or `herdr server reload-config`.

## Keys

Jump starts in normal mode unless `search_first = true`: bare keys are commands,
and <kbd>/</kbd> enters search mode. In search mode bare keys type normally;
<kbd>Esc</kbd> returns to normal mode. Rename, create, and close prompts are
input modes with the actions shown in their prompt.

| Key | Pane / mode | Action |
|---|---|---|
| <kbd>j</kbd> <kbd>k</kbd> | either pane, normal | move down / up |
| <kbd>↓</kbd> <kbd>↑</kbd> <kbd>Ctrl-n</kbd> <kbd>Ctrl-p</kbd> | either pane, normal or search | move down / up |
| <kbd>Ctrl-j</kbd> <kbd>Ctrl-k</kbd> | either pane, normal or search | move down / up |
| <kbd>Ctrl-d</kbd> <kbd>Ctrl-u</kbd> | either pane, normal or search | half-page down / up |
| <kbd>Page Down</kbd> <kbd>Page Up</kbd> | either pane, normal or search | page down / up |
| <kbd>Ctrl-u</kbd> | rename or create input | clear the current field |
| <kbd>g</kbd> <kbd>G</kbd> | either pane, normal | first / last row |
| <kbd>h</kbd> <kbd>l</kbd> | normal | activate workspaces / agents |
| <kbd>Tab</kbd> <kbd>Shift-Tab</kbd> | normal or search | switch pane |
| <kbd>Enter</kbd> | normal or search | jump to the highlighted workspace or agent |
| <kbd>1</kbd>–<kbd>9</kbd> | either pane, normal | jump directly to that workspace number |
| <kbd>J</kbd> <kbd>K</kbd> | workspaces, normal | reorder the highlighted workspace down / up |
| <kbd>r</kbd> | either pane, normal | rename the highlighted workspace or agent |
| <kbd>F2</kbd> | either pane, normal or search | rename the highlighted workspace or agent |
| <kbd>a</kbd> | workspaces, normal | create a workspace: enter a label, then a directory |
| <kbd>d</kbd> | workspaces, normal | close the highlighted workspace after confirmation |
| <kbd>s</kbd> | either pane, normal | cycle status: all → blocked → done → working → idle |
| <kbd>w</kbd> | agents, normal | cycle scope: all → current workspace → highlighted workspace |
| <kbd>Ctrl-r</kbd> | either pane, normal or search | refresh the herdr snapshot without clearing filters or queries |
| <kbd>Ctrl-o</kbd> | either pane, normal | jump back to the previous location, including the one Jump was opened from if you haven't jumped yet this session |
| <kbd>o</kbd> | agents, normal | toggle the highlighted agent's visible terminal output |
| <kbd>/</kbd> | either pane, normal | enter search mode |
| <kbd>?</kbd> | either pane, normal | toggle this help |
| <kbd>y</kbd> <kbd>Enter</kbd> | close confirmation | close the workspace |
| <kbd>n</kbd> <kbd>Esc</kbd> | close confirmation | keep the workspace |
| <kbd>Esc</kbd> | any | close help, leave the current mode, or quit Jump |
| <kbd>q</kbd> | either pane, normal | quit Jump |

In rename mode, <kbd>Enter</kbd> applies and <kbd>Esc</kbd> cancels. An empty
agent name clears its custom name; a workspace label cannot be empty.

Creating a workspace is two steps. The label step shows the default directory
in its header. <kbd>Enter</kbd> then opens a directory field prefilled with the
cwd of herdr's currently focused pane (or `$HOME`); <kbd>Enter</kbd> creates it,
<kbd>Esc</kbd> returns to the label, and another <kbd>Esc</kbd> cancels. An empty
label lets herdr derive the workspace name from the directory.

Closing asks for confirmation with <kbd>y</kbd>/<kbd>Enter</kbd> or
<kbd>n</kbd>/<kbd>Esc</kbd> and reports how many panes will close. When the
workspace hosts the popup, the prompt warns that closing it also closes Jump.

<kbd>J</kbd>/<kbd>K</kbd> move the row the cursor is on, which is not
necessarily the current workspace, and the cursor rides along with it. The
order is herdr's own, so it persists after the popup closes and renumbers the
workspaces everywhere. Reordering only works in the workspaces pane and needs
`socat` or `nc -U`.

Mouse interaction applies only to the active list. The other pane is rendered
as a preview, not a second clickable list. Mouse movement is disabled while an
input or confirmation prompt is active.

## Configure

Optional. Jump reads `$HERDR_PLUGIN_CONFIG_DIR/config.toml`, which is normally
`~/.config/herdr/plugins/config/lancodev.jump/config.toml` for this plugin.
`herdr plugin config-dir lancodev.jump` prints the active directory.

Settings are read once, when the popup opens, and held fixed for that session.
An edit to `config.toml` takes effect the next time you open Jump —
<kbd>Ctrl-r</kbd> only refreshes the workspace/agent data, not settings.

```toml
# Popup dimensions: cells or percentages.
width = "90%"
height = "60%"

# Initial active pane: "workspaces" or "agents".
start_pane = "workspaces"

# fzf colors: named ANSI color, 0-255 index, or #rrggbb.
accent = 6
dim = 8

# Workspace pane share when both panes are visible (20-60).
pane_ratio = 35

# "workspace" uses herdr order; "recent" puts targets visited through Jump first.
order = "workspace"

# Persist each pane's query between opens.
remember_query = false

# Open directly in search mode.
search_first = false

# Show the inactive pane as a preview; false gives the active list full width.
preview = true
```

| Setting | Default | Meaning |
|---|---|---|
| `width` | `"90%"` | popup width in cells or as a percentage |
| `height` | `"60%"` | popup height in cells or as a percentage |
| `start_pane` | `"workspaces"` | initially active pane: `workspaces` or `agents` |
| `accent` | `6` | fzf pointer, prompt, labels, match highlights, and current mark color |
| `dim` | `8` | borders, headers, info, status words, and other dim text color |
| `pane_ratio` | `35` | workspace pane percentage, from 20 through 60 |
| `order` | `"workspace"` | `workspace` follows herdr order; `recent` puts targets recently visited through Jump first, then unvisited rows in workspace order |
| `remember_query` | `false` | persist each pane's last query across picker opens |
| `search_first` | `false` | start in search mode instead of normal mode |
| `preview` | `true` | show the inactive pane; when false, render the active pane full-width |

## Notes

- The data is a snapshot. <kbd>Ctrl-r</kbd> refreshes it while the picker stays
  open; it is not a live dashboard.
- Agents come from herdr's snapshot, so anything herdr detects or that reports
  through an [integration](https://herdr.dev/docs/integrations/) appears
  without per-agent configuration.
- The popup is session-modal, so opening it while Settings or Copy mode is
  active returns `ui_busy`. Close the other modal first.
- herdr 0.9.0 lets attached clients view different workspaces and tabs, but
  its `agent.focus` stopped reaching the client that asked, so a jump landed
  nowhere. Jump composes the hop from `tab.focus` and `pane.focus` instead,
  which still broadcast. That means a jump moves every attached client, the
  same as herdr's own `workspace.focus`.

## License

MIT
