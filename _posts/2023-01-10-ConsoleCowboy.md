---
title: The Way of the Console Cowboy
date: 2023-01-10 10:30
categories: [General]
tags: [Linux]
--- 

## Tips and Tricks for Navigating Linux

### tmux

To start a new tmux session in your current directory,type
 `tmux new -s <name>`

The default prefix key for tmux commands is `ctrl+b`

To open a new window in tmux, press `prefix+c`

Switch between tmux windows with `prefix+<Window #>`

To rename a tmux window, `prefix+,`

To detatch from your tmux session, press `prefix+d`

List tmux sessions with `tmux ls`

Join a tmux session with `tmux attach -t <session name>`

Horizontally split your terminal with `prefix+"`

Vertically split your terminal with `prefix+%`

Move between split panes using `prefix+<arrow key>`

Zoom in and out of split panes using `prefix+z`

To adjust the size of split panes use `prefix+<hold arrow key>`

To switch the position of panes use `prefix+{` or `prefix+}`

To cycle through different layouts of panes use `prefix+<space bar>`

