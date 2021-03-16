---
title:  "Creating a New Window on the Current Working Directory in Tmux"
date:   2019-05-02
categories: Tmux
tags: [Tmux, Unix]
---

# The Issue
Every time I tried to open a new pane (either vertical/horizontal/new window, etc.), I was navigated to my $HOME directory. Here's a simple solution. First, open your tmux.conf file

```console
$ cd ~/kyle/env # cd to wherever your tmux.conf file is.
$ vim tmux.conf
```

Add these lines to tmux conf.

```
bind '"' split-window -c "#{pane_current_path}"
bind % split-window -h -c "#{pane_current_path}"
bind c new-window -c "#{pane_current_path}"
```
where
* `<Leader key> + "` lets you split a new window vertically.
* `<Leader key> + %` lets you split a new window horizontally.
* `<Leader key> + c` lets you create a new window.

The `#{pane_current_path}` part does the trick. Done!
