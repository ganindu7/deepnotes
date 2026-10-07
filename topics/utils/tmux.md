---
layout: default
title: tmux (Ctrl+f prefix)
parent: Utilities
permalink: /topics/utils/tmux
nav_exclude: true
---

## tmux (Ctrl+f prefix)

Status: Reference · Updated: {{ page.path | file_date | date_to_string }}

Personal reference for my tmux setup, kept here with the [keyboard layers]({{ '/topics/utils/keyboard_layers' | relative_url }}) so it travels with me. It is deliberately left out of the sidebar; search for **tmux** to find it.

The config is `~/.tmux.conf`, symlinked from the dotfiles repo, and written for tmux 2.4+ so the same file works on the Mac and on the Linux boxes (benches, trucks, servers). Four things differ from stock tmux:

1. **Prefix is Ctrl+f** instead of Ctrl+b. With Caps Lock as Ctrl (see keyboard layers), Caps Lock + f is a home-row chord.
2. **Vim keys for panes**: prefix + h/j/k/l moves between panes, prefix + H/J/K/L resizes by 5 cells.
3. **vi copy mode**: prefix + `[` to enter, `v` to select, `y` to copy.
4. **Mouse on**, 102400 lines of scrollback, windows and panes numbered from 1, 10 ms escape time.

Source of truth for the file is the [dotfiles](https://github.com/ganindu7/dotfiles) repo. The files below are a snapshot from 7 Oct 2026 (dotfiles commit 13b3051).

## Cheat sheet

![tmux cheat sheet]({{ '/topics/utils/tmux/cheat-sheet.png' | relative_url }})

[Print-ready PDF (A4 landscape)]({{ '/topics/utils/tmux/cheat-sheet.pdf' | relative_url }}){: .btn .btn-outline .fs-3 .mb-2 .mr-2 }
[tmux.conf]({{ '/topics/utils/tmux/tmux.conf' | relative_url }}){: .btn .btn-primary .fs-3 .mb-2 .mr-2 }

## Prefix

Every binding below is **prefix, release, then the key**. The prefix is Ctrl+f (Caps Lock + f on the Mac).

| Keys | Action |
|:--|:--|
| Ctrl+f | Prefix |
| Ctrl+f, Ctrl+f | Send a real Ctrl+f to the program in the pane (page forward in vim and less, forward-char in readline) |
| prefix `?` | List all bindings |
| prefix `:` | Command prompt (emacs editing keys) |
| prefix `t` | Clock |

## Panes

| Keys | Action | Note |
|:--|:--|:--|
| prefix `"` | Split below | |
| prefix `%` | Split right | |
| prefix `h` `j` `k` `l` | Move to the pane left / below / above / right | Custom. Not repeatable on purpose: a letter typed just after a move must reach the shell, not tmux. Arrows also work and are repeatable. |
| prefix `H` `J` `K` `L` | Resize the pane left / down / up / right by 5 cells | Custom, repeatable: press prefix once, then keep tapping within 500 ms. Avoids the stock Ctrl+Arrow, which Mission Control grabs on macOS (also over ssh). |
| prefix `z` | Zoom the pane to full window and back | |
| prefix `x` | Kill the pane (asks y/n) | |
| prefix `o` | Next pane | |
| prefix `;` | Back to the last pane | |
| prefix `q` | Show pane numbers, then type one to jump | |
| prefix `Space` | Cycle through layouts | |
| prefix `{` / `}` | Swap with the previous / next pane | |
| prefix `!` | Break the pane out into its own window | |

Mouse: click selects a pane, dragging a border resizes, the wheel scrolls the pane into copy mode. In iTerm2 hold Option while dragging to select text with the terminal instead of tmux.

## Windows

| Keys | Action | Note |
|:--|:--|:--|
| prefix `c` | New window | |
| prefix `n` / `p` | Next / previous window | |
| prefix `1` … `9` | Jump to window by number | Numbered from 1 to match the key row; gaps close when a window is killed. |
| prefix `w` | Pick a window from a list | |
| prefix `,` | Rename the window | |
| prefix `&` | Kill the window (asks y/n) | |
| prefix `.` | Move the window to another index | |
| prefix `f` | Find a window by name | |
| prefix `l` | ~~Last window~~ | Taken by pane-right. Use `w` or a number. |

## Sessions

| Keys | Action | Note |
|:--|:--|:--|
| prefix `d` | Detach; the session keeps running | |
| prefix `s` | Pick a session from a list | |
| prefix `$` | Rename the session | |
| prefix `(` / `)` | Previous / next session | |
| prefix `L` | ~~Last session~~ | Taken by resize-right. Use `s`. |

From the shell:

```shell
tmux new -A -s main          # attach to "main", or create it
tmux ls                      # list sessions
tmux a -t main               # attach
tmux kill-session -t main    # end it
ssh somehost -t 'tmux new -A -s main'   # attach-or-create on a remote box
```

## Copy mode (vi keys)

| Keys | Action | Note |
|:--|:--|:--|
| prefix `[` | Enter copy mode | Scrolling up with the mouse wheel also enters it. |
| `v` | Start selecting characters | Custom; `Space` also works. |
| `V` | Select whole lines | |
| Ctrl+v | Toggle block (rectangle) selection | |
| `y` | Copy the selection and leave copy mode | Custom; `Enter` also works. |
| prefix `]` | Paste | |
| `q` | Leave copy mode without copying | `Esc` only clears the selection. |
| `/` / `?` | Search down / up, then `n` / `N` for the next / previous match | |
| `g` / `G` | Top / bottom of the scrollback | |
| Ctrl+u / Ctrl+d | Half a page up / down | |
| `h` `j` `k` `l` `w` `b` `0` `$` | Move as in vim | |

## Settings

| Option | Value | Why |
|:--|:--|:--|
| `history-limit` | 102400 | Scrollback lines per pane (default 2000). |
| `base-index`, `pane-base-index` | 1 | Windows and panes numbered from 1, matching the key row. |
| `renumber-windows` | on | Close the gap when a window is killed. |
| `escape-time` | 10 | Esc reaches vim at once instead of after 500 ms. |
| `status-keys` | emacs | Editing keys on the prefix + `:` prompt. |
| `mode-keys` | vi | Copy mode uses vi keys. |
| `mouse` | on | See Panes above. |

Reload after editing, in a running server:

```shell
tmux source-file ~/.tmux.conf
```

## Setting it up on a new machine

Needs tmux 2.4 or newer (`tmux -V`); the copy-mode bindings use the `-T copy-mode-vi` syntax introduced there.

Just the one file:

```shell
curl -fsSL https://ganindu7.github.io/deepnotes/topics/utils/tmux/tmux.conf -o ~/.tmux.conf
tmux source-file ~/.tmux.conf   # if a server is already running
```

The full dotfiles, symlinked and with backups of whatever was there:

```shell
git clone git@github.com:ganindu7/dotfiles.git ~/Workspace/dotfiles
~/Workspace/dotfiles/install.sh             # everything
~/Workspace/dotfiles/install.sh tmux.conf   # or only this file
```

Onto a remote box over ssh, from the Mac, with the same backups and a check that tmux loads the file:

```shell
cd ~/Workspace/dotfiles
./deploy.sh <host> tmux.conf
```

## Files in this folder

| File | What it is |
|:--|:--|
| [cheat-sheet.png]({{ '/topics/utils/tmux/cheat-sheet.png' | relative_url }}) | The visual reference above, A4 landscape at 2x. |
| [cheat-sheet.pdf]({{ '/topics/utils/tmux/cheat-sheet.pdf' | relative_url }}) | Same design, print-ready. |
| [cheat-sheet.html]({{ '/topics/utils/tmux/cheat-sheet.html' | relative_url }}) | Static HTML/CSS source of the design. Fixed size, not responsive. Uses the keyboard layers [design tokens]({{ '/topics/utils/keyboard_layers/design-tokens.json' | relative_url }}). |
| [tmux.conf]({{ '/topics/utils/tmux/tmux.conf' | relative_url }}) | The config itself, as deployed. |
| [bindings.json]({{ '/topics/utils/tmux/bindings.json' | relative_url }}) | The bindings, shell commands and options on the sheet as data. |
