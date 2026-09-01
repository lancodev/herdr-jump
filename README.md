# Jump for herdr

A two-pane fuzzy switcher for [herdr](https://herdr.dev): **workspaces on the
left, agents on the right**, vim keys to move within a pane, swap between them,
or reorder your workspaces.

herdr's built-in session navigator is one tree of every workspace, tab and
pane. It cannot be scoped — it re-expands every workspace each time it opens,
so collapsing a row with <kbd>Space</kbd> never sticks. Jump is the flat
alternative: pick a workspace, or cross over and pick an agent directly.

```
╭──────────── workspaces ─────────────╮╭─────────────── agents ────────────────╮
│ ❯                                   ││   ●  dotfiles  opencode › OC | Limit… │
│   j/k move · J/K reorder · / filt…  ││   ●  api  claude › refactor billing   │
│     1  ●  dotfiles           1 pane ││ ▸ ○  web  opencode › fix scroll bug   │
│     2  ●  api               2 panes ││                                       │
│ ❯ ▸ 3  ○  web                1 pane ││                                       │
╰─────────────────────────────────────╯╰───────────────────────────────────────╯
```

- Both panes are always visible; <kbd>h</kbd>/<kbd>l</kbd>/<kbd>Tab</kbd> move focus between them
- Each pane opens on whatever is currently focused, so <kbd>⏎</kbd> is a no-op and <kbd>j</kbd><kbd>⏎</kbd> is the next workspace
- Selecting an agent jumps workspace, tab and pane in one hop
- <kbd>J</kbd>/<kbd>K</kbd> reorders the highlighted workspace without leaving the picker
- Status glyphs mirror herdr's own sidebar and inherit your terminal theme

## Requirements

[`fzf`](https://github.com/junegunn/fzf) 0.54+ and [`jq`](https://jqlang.org),
both on `PATH`.

Reordering also needs `nc` or `socat`, because herdr ships no CLI wrapper for
`workspace.move` and the call has to go to the socket directly. Without either
one the picker still switches normally and simply drops the reorder key.

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

Jump starts in normal mode, like herdr's navigator: bare keys are commands,
<kbd>/</kbd> starts filtering.

| Key | |
|---|---|
| <kbd>j</kbd> <kbd>k</kbd> <kbd>ctrl-n</kbd> <kbd>ctrl-p</kbd> | move |
| <kbd>ctrl-d</kbd> <kbd>ctrl-u</kbd> | half page |
| <kbd>g</kbd> <kbd>G</kbd> | first / last |
| <kbd>h</kbd> <kbd>l</kbd> <kbd>Tab</kbd> | swap pane |
| <kbd>J</kbd> <kbd>K</kbd> | reorder the highlighted workspace |
| <kbd>/</kbd> | filter; <kbd>Esc</kbd> returns to normal mode |
| <kbd>⏎</kbd> | jump to the selection |
| <kbd>q</kbd> <kbd>Esc</kbd> | close |

In filter mode the bare keys type normally, so searching for `jq` or `agent`
works as expected.

<kbd>J</kbd>/<kbd>K</kbd> move the row the cursor is on, which is not
necessarily the focused workspace, and the cursor rides along with it. The order
is herdr's own, so it persists after the popup closes and renumbers the
workspaces everywhere. The agents pane is a view of that same order rather than
a list of its own, so it re-sorts as you go; there is nothing to reorder from
that side.

## Configure

Optional. Create `config.toml` in the plugin's config dir
(`herdr plugin config-dir lancodev.jump` prints the path):

```toml
# Popup dimensions. Cells or percentages.
width = "90%"
height = "60%"

# Which pane is active when the picker opens: "workspaces" or "agents".
start_pane = "workspaces"

# fzf colors for the accent (pointer, prompt, labels, match highlights) and for
# borders and the header. ANSI index, 256-color index, or hex.
accent = 6
dim = 8
```

## Notes

- The list is a snapshot taken when the popup opens, not a live view. It is a
  switcher you are in for a second, not a dashboard.
- Agents come from `herdr agent list`, so anything herdr detects or that
  reports through an [integration](https://herdr.dev/docs/integrations/) shows
  up — no per-agent configuration here.
- The popup is session-modal, so opening it while Settings or Copy mode is
  active returns `ui_busy`. Close the other modal first.

## License

MIT
