---
title: The Way of the Console Cowboy
date: 2023-01-10 10:30
categories: [General]
tags: [Linux]
--- 

## Tips and Tricks for Navigating Linux

### tmux

Tmux is a powerful screen multiplexer, meaning it lets you handle multiple panes in a command line interface, which is super useful for sys-admin tasks, hackthebox, and really anything on linux that involves multitasking. I found myself using it alot in my time as a linux admin when handling multiple remote SSH connections to various servers, and it is a fantastic tool.

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

## Grep

Grep is a very powerful command line tool used to find matching patterns. You can use it to find a file in a directory, to find a specific string inside of a file, or you can even use it on the output of another command with the | character. The basic syntax of a grep command is 

`grep [options] [pattern] [file]`

So, to find a file within a directory, you can use

`ls [/path/to/directory/] | grep [pattern]`

Because linux is case sensitive, you can use the -i flag to grep for results in a non case sensitive fashion, as such

`grep -i [pattern] [/path/to/file.txt]`

To make grep search through files recursivly in a directory, type 

`grep -r [pattern] [/path/to/directory/]`

To make grep invert the sense of matching, and exclude a specific pattern, use the -v flag

`grep -v [pattern] [/path/to/directory/]`

There are tons more flags and options for grep, but these are a few that I find myself using most often. These can also be chained together to do more cool stuff. For example,

`ls /var/www | grep -iv config` 

Will search for file names within the /var/www/ directory that do not contain config. To view the full power of grep, you can always use

`man grep`