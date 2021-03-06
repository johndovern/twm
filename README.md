# twm

The goal of twm is to make tmux behave more like a dynamic tiling window
manager (specifically dwm). Right now `tmux.conf` checks most of those boxes.
- [X] Master slave layout
- [X] New panes spawn as master pane
- [X] Able to swap focused pane with master pane
- [X] Scroll up and down through stack
- [X] Multiple layouts
  - [X] Tiled (horizontal and vertical)
  - [X] Fullscreen (or zoomed)
- [X] Move between windows (tags/workspaces)
- [ ] Able to have multiple masters

In my mind this is the basic functionality of a dynamic tiling window manager.
The only functionality that is not present is the possibility to have two or
more master panes. It may be possible to get this functionality, but I am not
sure of the best way to incorporate this here. Anyways on to the key bindings!
## Keybinds
```
    <key>           <action>
    j               Move down pane stack

    k               Move up pane stack

    h               Move focus to prev (-) window

    l               Move focus to next (+) window

    H               Move the focused pane to the
                    prev (-) window. Focus remains
                    on current window and does not
                    switch to the destination.

    L               Move the focused pane to the
                    next (+) window. Focus remains
                    on current window and does not
                    switch to the destination.

    space           Swap focused window with master.
                    If focused window is the master
                    swap with next slave (top right)

    enter/v/s       Spawn new pane as master

    i               Fullscreen (zoom) focused pane

    f               Focus master pane

    q               Kill focused pane and focus master
```
**All of the above binds must be prefixed, Ctrl+b by default, to work**

If you've used dwm, then the binds for j/k will be familiar to you.
You may be used to h/l doing something else and if that is the case
then feel free to comment those binds out or rebind them.

H and L might be new to dwm users as well. If there is no next or prev window
then nothing will happen. I have it in mind to use the `break-pane` command to
resolve this, but it has not yet been implemented.

Space doesn't work exactly the way I would like at the moment. If you have
three windows open like this
```
_________________________________
|                  |            |
|                  |            |
|                  |            |
|                  | ....2      |
|                  |            |
| .......1         | __________ |
|                  |            |
|                  |            |
|                  | ....3      |
|                  |            |
|                  |            |
| ________________ | __________ |
```
And you press `<prefix> space` while pane 3 is focused, then panes 1 and 3 will
swap places instead of moving pane 3 to the top of the stack and moving all
other panes down one level. Maybe there is a way to achieve this all within
tmux, but I do not know how to do that.

<prefix> enter/s/v should be familar to users of dwm.

<prefix> i zooms the current pane. With this I do not find any need to install
a bunch of plugins for tmux or (n)vim as I can focus a(n) (n)vim pane and use
Ctrl+h/j/k/l to move around my (n)vim splits.

<prefix> f is normally bound to the `find-window` command. I have included the
same command, commented out, as <prefix> F so you can retain that bind. Here
<prefix> f will focus the master pane, something I use often in dwm and now use
similarly often in tmux.
## Installation
```bash
# Using curl
mkdir -p ~/.config/tmux/twm
curl -JL https://github.com/johndovern/twm/raw/master/twm.conf -o ~/.config/tmux/twm/twm.conf
curl -JL https://github.com/johndovern/twm/raw/master/twm-horizontal.conf -o ~/.config/tmux/twm/twm-horizontal.conf
echo "source-file ~/.config/tmux/twm/twm.conf" >> ~/.config/tmux/tmux.conf
# or if you use ~/.tmux.conf
echo "source-file ~/.config/tmux/twm/twm.conf" >> ~/.tmux.conf
# Using git
git clone https://github.com/johndovern/twm.git ~/.config/tmux/twm
echo "source-file ~/.config/tmux/twm/twm.conf" >> ~/.config/tmux/tmux.conf
# or if you use ~/.tmux.conf
echo "source-file ~/.config/tmux/twm/twm.conf" >> ~/.tmux.conf
```
Depending on where you keep your tmux config file these commands should make
tmux act more like a dynamic tiling window manager. Alternatively you can copy
and paste the block below into you tmux config file. This will only give you
access to the vertical layout.
```
# Set main pane size for vertical and horizontal layouts
set-option -g main-pane-height 60%
set-option -g main-pane-width 60%

# move focus through pane stack
bind-key j run "(tmux display-message -p '#{window_zoomed_flag}' | grep -iqE '^1$' && tmux resize-pane -Z) ; (tmux select-pane -t {next})" \; select-layout main-vertical
bind-key k run "(tmux display-message -p '#{window_zoomed_flag}' | grep -iqE '^1$' && tmux resize-pane -Z) ; (tmux select-pane -t {previous})" \; select-layout main-vertical

# move pane up or down in stack
bind-key J run "(tmux display-message -p '#{window_zoomed_flag}' | grep -iqE '^1$' && tmux resize-pane -Z) ; (tmux display-message -p '#{pane_at_left} #{pane_at_bottom}' | grep -iqE '^0 1$' && tmux rotate-window -D && tmux select-pane -t0) || (tmux swap-pane -D)" \; select-layout main-vertical
bind-key K run "(tmux display-message -p '#{window_zoomed_flag}' | grep -iqE '^1$' && tmux resize-pane -Z) ; (tmux display-message -p '#{pane_at_left} #{pane_at_bottom}' | grep -iqE '^1 1$' && tmux rotate-window -U && tmux select-pane -t {bottom-right}) || (tmux swap-pane -U)" \; select-layout main-vertical

# move to next (+) or prev (-) window
# using l and h respectively
bind-key l next-window \; select-layout main-vertical
bind-key h previous-window \; select-layout main-vertical

# send current pane to the next (+) or prev (-) window
# using L and H respectively
# remove the '-d' flag if you want to follow the pane
bind-key L join-pane -fdbh -t:+ \; select-layout main-vertical
bind-key H join-pane -fdbh -t:- \; select-layout main-vertical

# swap current pane with master pane
# if current pane is the master, the next slave is swapped
bind-key space run "(tmux display-message -p '#{window_zoomed_flag}' | grep -iqE '^1$' && tmux resize-pane -Z) ; (tmux display-message -p '#{pane_index}' | grep -iqE '^0$' && tmux swap-pane -dZ -s1) || (tmux swap-pane -s0)" \; select-layout main-vertical

# use <prefix> enter, v, and s to open new panes as master
bind-key enter run "(tmux display-message -p '#{window_zoomed_flag}' | grep -iqE '^1$' && tmux resize-pane -Z) ; (tmux split-window -bh -t0)" \; select-layout main-vertical

# <prefix> i to zoom the focused pane
bind-key i resize-pane -Z

# <prefix> f to focus the master pane
bind-key f select-pane -t0 \; select-layout main-vertical

# <prefix> f is rebound above, this is an alternative if you'd like to remap it
# bind-key F command-prompt "find-window -Z -- '%%'"

# simple binds for pane/window switching
bind-key a last-pane \; select-layout main-vertical
bind-key c new-window \; select-layout main-vertical
bind-key t next-window \; select-layout main-vertical
bind-key T previous-window \; select-layout main-vertical

# Kill pane
bind-key q killp \; select-pane -t0 \; select-layout main-vertical
```
As <prefix> f is a prebound key I have included a replacement bind for it,
commented out, at the bottom. You can replace enter with v/s if you are more
used to using those key combos.
## Disclaimer
twm does not look to incoporate checks for (n)vim when switching panes. If you
know what you are doing and want to suggest changes to enable movement between
panes and splits in a painless manner then I am all ears but this is not a
problem I am looking to solve.
