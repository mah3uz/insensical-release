# Changelog

What changed in each release of insensical, newest first. Each release's section is also
its release notes.

## Unreleased

## 0.1.0 - 2026-10-04 04:35 +06:00

The first release of insensical: a terminal multiplexer with a native window, for running
many terminals and coding agents and knowing which of them needs you. It comes as two
programs: `insensical`, the window, and `isc`, the server and the command line.

It does not redraw what a program prints. A server owns each program and its terminal;
a window that attaches is sent a snapshot and then the program's own bytes, untouched.
Panes, tabs and splits are real interface elements, not characters drawn inside one
terminal.

### Programs that stay

- **They outlive the window.** Close it and open it again: everything is there, still
  running.
- **They come back after a stop.** Restart the machine, and every project returns with
  its tabs, splits, floating and minimized panes, each pane in the directory it was in,
  showing what it showed, with what ran in it typed at the prompt to run again.
- **They outlive an update.** A new version installed under a running one replaces
  nothing. The window says it is there, and the server moves to it when you say so, with
  every program still running and everything it printed still there.

### The window

- **Projects** in a sidebar, as a tree: each project, its tabs and their panes, with what
  runs in them, where, and in what state. Tabs can be along the top instead.
- **Splits** in either direction, resized, moved and swapped by key or by mouse, and one
  pane filling its tab for a while.
- **Floating panes**, moved by the header and resized from any edge, over one tab or over
  every tab of a project. One key brings a scratch shell over whatever you are looking
  at, and puts it away again.
- **Minimize** any pane to a dock. Its program runs on at the size it had.
- **An overview** of every tab of every project, each drawn small as it is laid out.
- **A command palette**, a finder for files, search in a pane's text, and web addresses
  and paths that open on a click.
- **Themes**: a terminal's colours in a small file, and an interface derived from them.
  Any Ghostty theme works by name.
- **Keys your way**: each command on a chord of its own, or after a leader key as in tmux
  and vim, and any of them rebound. One key shows them all, and another gives every key
  to the program in the pane.

### Agents and attention

- **A state on every pane**: working, waiting for you, done or failed, from what agents
  already write to their terminal and from their hooks.
- **A count of the panes that need you**, and one key to go to the most urgent.
- **Hooks for Claude Code, Codex, Gemini CLI, opencode and pi**, set up when you say so,
  from the window or with `isc`. Those for Codex, Gemini CLI, opencode and pi are written
  from the agents' documentation and have not been run under them.
- **Desktop notifications** when a pane starts waiting, finishes or fails, unless you are
  looking at it. They arrive with no window open.

### The command line

`isc` does everything the window does: it makes and arranges projects, tabs and panes,
sends keys, reads what a pane shows, waits for a pane, and follows events. It completes
panes, tabs and projects in bash, zsh and fish.

### Installing

- **Arch Linux**: `insensical-bin` in the AUR.
- **Debian 12 and later, Ubuntu 24.04 and later**: the `.deb`.
- **Other Linux on x86-64**, with glibc 2.36 or later: the tarball.
- **macOS 13 and later, on Apple silicon**: the disk image, or Homebrew. The application
  is not signed with a Developer ID, so macOS has to be told to open it; the manual says
  how.

`MANUAL.md` is the manual. It needs a Nerd Font: the default is JetBrainsMono Nerd Font.
