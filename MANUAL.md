# insensical: the manual

insensical is a terminal multiplexer with a native window, for running many terminals
and coding agents and knowing which of them needs you.

## Contents

1. [What it is](#1-what-it-is)
2. [Getting it running](#2-getting-it-running)
3. [A first session](#3-a-first-session)
4. [Concepts](#4-concepts)
5. [The window](#5-the-window)
6. [Projects and the sidebar](#6-projects-and-the-sidebar)
7. [Tabs](#7-tabs)
8. [Panes](#8-panes)
9. [The terminal](#9-the-terminal)
10. [States and attention](#10-states-and-attention)
11. [Agents](#11-agents)
12. [Notifications](#12-notifications)
13. [Keys](#13-keys)
14. [The command palette and the overview](#14-the-command-palette-and-the-overview)
15. [Settings and themes](#15-settings-and-themes)
16. [The command line, `isc`](#16-the-command-line-isc)
17. [Stopping, restarting and new versions](#17-stopping-restarting-and-new-versions)
18. [When something goes wrong](#18-when-something-goes-wrong)
19. [Reference](#19-reference)

---

## 1. What it is

insensical lets you run many terminals and agents side by side and see which one needs
you. A background server, the *daemon*, owns every program. The window (`insensical`) and
the command line (`isc`) are its clients, so closing the window stops nothing. Open it
again and everything is still there.

Panes, tabs and splits are real UI elements, and a program's output reaches the window
untouched.

What you get:

- Programs that keep running with no window open, and through an update.
- Every project, tab, split, floating pane and minimized pane back after the daemon has
  been stopped. Each pane returns in its directory, showing what it showed before.
- Tabs grouped into projects, listed in a sidebar as a tree of projects, tabs and panes.
- Panes you can float above the layout, minimize to a dock, or drag to rearrange.
- Pictures drawn with the Kitty graphics protocol.
- A state on every pane: working, waiting for you, done or failed. One key takes you to
  the most urgent.
- A notice when a pane starts waiting, finishes or fails, unless you're looking at it:
  inside the window while it's in front, a desktop notification when it isn't.
- Memory use per project, and sleep for a project you don't need: its programs end, and
  its panes and what they showed are kept.
- Every setting on one sheet.
- Everything the window does, from the command line too.
- The whole interface in a terminal, with `isc` alone: on your own machine, on one with
  no desktop, or over `ssh`.

---

## 2. Getting it running

Here's how to install insensical and start it.

### Installing

Releases are at <https://insensical.com/download>, each file with its SHA-256 beside it.

**Arch Linux.** Install from the AUR:

```sh
yay -S insensical-bin
```

**Debian 12 and later, Ubuntu 24.04 and later**, on x86-64. Download the `.deb`, then:

```sh
sudo apt install ./insensical_*_amd64.deb
```

There's no apt repository. To update, download the new version and install it the same
way.

**Other Linux**, on x86-64 with glibc 2.36 or later. Unpack the tarball and put
`insensical` and `isc` together in a directory on your `PATH`. It also holds the desktop
entry, the icon, the user unit and completions for bash, zsh and fish. Put them where
your system looks for them.

The window is built for x86-64 only. On an ARM machine, install `isc` by itself.

**Without the window, on any Linux**, x86-64 or arm64, with or without a desktop. `isc`
alone is insensical with no window: the daemon, the command line and
[the whole interface in a terminal](#the-whole-interface-in-a-terminal), in one file that
needs nothing installed. The packages above install the same `isc` beside the window.

```sh
curl -fsSL https://insensical.com/install.sh | sh
```

It downloads `isc-x86_64-linux` (`isc-aarch64-linux` on ARM) and its checksum from the
latest release, and installs the file only if they agree. Run the same line to update.

| Set | Effect |
|---|---|
| `ISC_VERSION=0.3.1` | Installs that release instead of the latest. `isc --version` prints yours. |
| `ISC_DIR=~/.local/bin` | Installs there, with no password. The default is `/usr/local/bin/isc`, using `sudo` if needed. |
| `ISC_QUIET=1` | Asks nothing and restarts nothing. It stops if the directory needs a password. Also the behaviour when there's no terminal to ask on. |

Put the variable before `sh`, as in `curl -fsSL https://insensical.com/install.sh |
ISC_DIR=~/.local/bin sh`.

- If you already have `isc` on your `PATH`, the new one goes over it there. It won't
  write over `/usr/bin/isc`, which a package put there: update that with your package
  manager.
- If a daemon is running, it asks whether to restart it, and Enter says yes. Restarting
  ends nothing in your panes. Say no and run `isc restart-server` when it suits you.
- Completions aren't installed. `isc completions bash`, `zsh` or `fish` prints them.
- The checksum comes from the same release as the file. It catches a damaged download,
  not a wrong release.

The same by hand:

```sh
cd "$(mktemp -d)"
curl -fLO https://github.com/mah3uz/insensical-release/releases/latest/download/isc-x86_64-linux -O https://github.com/mah3uz/insensical-release/releases/latest/download/isc-x86_64-linux.sha256
sha256sum -c isc-x86_64-linux.sha256 && install -m755 isc-x86_64-linux ~/.local/bin/isc
isc
```

That address gives the latest release. On ARM the file is `isc-aarch64-linux`. For one
release, put `download/v0.3.1` where it says `latest/download`. A machine that can't reach
the release needs only those two files, copied to it. Replacing the file doesn't stop a
running daemon; `isc restart-server --yes` moves it to the new version with everything
still running.

**macOS 13 and later, on Apple silicon.** With Homebrew, which also puts `isc` on the
`PATH`:

```sh
brew install --cask mah3uz/tap/insensical
```

Or open the `.dmg` and drag the application to Applications. It isn't signed with a
Developer ID or notarised, so macOS won't open a downloaded copy until you clear the
quarantine flag (Homebrew does this for you):

```sh
xattr -dr com.apple.quarantine /Applications/insensical.app
```

`isc` is at `/Applications/insensical.app/Contents/MacOS/isc`. There's no build for an
Intel Mac.

**Starting the daemon at login.** The package installs a user unit but doesn't enable it.
To enable it, run `systemctl --user enable --now insensical`. You don't have to: the
first window or command that needs the daemon starts it. Upgrading the package only
replaces files; the running daemon carries on until you move it (see
[A new version](#a-new-version-installed-underneath)).

**What any uninstall leaves.** Removing insensical takes the two programs, the launcher's entry, the icons and the completions. It leaves:

- your settings, in `~/.config/insensical`;
- the saved layout, what the panes showed, the window's size and your recent palette
  choices, in `~/.local/state/insensical` (on a Mac,
  `~/Library/Application Support/insensical`);
- what `isc hub` keeps (its keys, paired devices and joined machines), in
  `~/.local/share/insensical/hub` (on a Mac, `~/Library/Application
  Support/insensical/hub`);
- hooks added to an agent's own settings. Run `isc integrate AGENT --remove` before `isc`
  is gone;
- **a daemon that is running.** It carries on with every program in its panes. Run
  `isc kill-server` first to end it.

Delete the two directories to be rid of everything.

### What it needs

- **Linux:** x86-64 with glibc 2.36 or later for the window; `isc` alone also runs on arm64. The
  window needs Wayland or X11 and Vulkan.
- **macOS:** 13.0 or later.
- **Fonts:** none in particular. The default is `JetBrainsMono Nerd Font`. The window draws
  its own icons, and an icon font inside the application covers the Nerd Font icons programs
  print.

### The two binaries must be together

The window finds the daemon in this order: the file `$INSENSICAL_ISC` names, the `isc`
beside the window, then `isc` on `PATH`. If none exists, it can't start a daemon and
says the server cannot be reached.

### The icon

The icon is a raised terminal chip with a prompt on a glossy
blue tile. A launcher shows it from the desktop entry. A taskbar or switcher shows it for
a window only if the desktop can match the window to that entry, so the window names
itself `insensical`.

- Run without installing, the window gets a generic icon, as there's no entry to match.
- An old `icon-theme.cache` in `~/.local/share/icons/hicolor` can hide the icon. Rebuild
  it with `gtk-update-icon-cache --force --ignore-theme-index ~/.local/share/icons/hicolor`.
  A running launcher or panel may keep the old picture until you restart it.

### Starting it

Start `insensical` from a launcher or a terminal. If no daemon is running, the window
starts one, and without a settings file the defaults apply.

Where you start it matters twice. The first project is made from the directory you start
it in (see [First launch](#first-launch)), and the daemon takes the environment of
whatever started it (see [The daemon's environment](#the-daemons-environment)).

---

## 3. A first session

This walks you through the basics. The keys are the Linux defaults; section 19 lists all
of them, the macOS keys and the leader keymap too.

1. **Open it.** Start `insensical`. You get one project, one tab and one pane running
   your shell. It's an ordinary terminal.
2. **Split.** Press `Ctrl+Shift+O` to split the pane to the right, or `Ctrl+Shift+E` to
   split downwards. The new pane opens in the same directory and takes half the space.
   `Ctrl+Alt+arrow` moves between panes. Drag a divider to resize.
3. **Open a tab.** Press `Ctrl+Shift+T`. The shell opens in the project's root, or in the
   current pane's directory if the project has none. `Ctrl+Tab` and `Ctrl+Shift+Tab` go
   to the next and previous tab.
4. **Open a project.** Press `Ctrl+Shift+N`, leave the name empty, type a directory that
   exists, such as `~/Projects/quote`, and press Enter. The project is named after the
   directory. `Ctrl+Alt+PageDown` and `Ctrl+Alt+PageUp` move between projects.
5. **Float a pane.** `Ctrl+Shift+D` opens a new floating pane. `Ctrl+Shift+G` floats the
   current pane, or puts a floating one back. Drag a float by its header and resize it
   from any edge or corner.
6. **Minimize a pane.** `Ctrl+Shift+M` puts the pane in the dock at the bottom, still
   running. Click its chip or press `Ctrl+Shift+R` to bring it back where it was.
7. **Find the pane that needs you.** Run `sleep 40` in one pane, then switch to another
   tab. When it ends, the pane becomes `done`: its tab, its sidebar row and the count at
   the top of the sidebar show it, and a desktop notification arrives. Press
   `Ctrl+Shift+A` to go there. Looking at the pane clears the mark.
8. **Close the window.** Everything keeps running. Open `insensical` again and it's all
   there.

Two things to know from the start:

- **Closing a pane never asks**, whatever is running in it. Closing a tab or a project
  shows what would end and asks first.
- `Ctrl+Shift+/` shows every key on one sheet, and `Ctrl+Shift+P` opens the command
  palette, which lists every command with its key.

---

## 4. Concepts

These are the words the rest of the manual uses, and how the daemon behaves.

### Words

| Word | Meaning |
|---|---|
| Project | A named group of tabs with a root directory, usually one codebase. |
| Tab | One layout of panes inside a project. |
| Pane | One terminal running one program. |
| Placement | Where a pane is: *tiled* in the layout, *floating* above it, or *minimized*. A pane moves between the three without restarting its program. |
| Dock | The strip at the bottom of the window that holds minimized panes. |
| State | What a pane is doing: `idle`, `running`, `waiting`, `done`, `failed`, `exited`. |
| Attention | A pane that needs you: one that's `waiting`, `done` or `failed`. |
| Daemon | The background process that owns everything. The window calls it the *server*. |
| Hub | The one machine the others connect through. It runs `isc hub`, which serves the page and lets your devices and other machines in. |
| Machine | A computer running insensical: this one, one joined to your hub, or one reached over `ssh`. |
| Device | Something you look from that has no insensical of its own, such as a browser, paired with your machines by its key. |

### The daemon

- The first window or `isc` command that needs it starts it. It runs in its own session
  and outlives whatever started it. You don't need to enable anything at login.
- `isc daemon` runs it in the foreground. A second daemon on the same socket refuses to
  start and says "a daemon is already running". A running daemon holds a lock on
  `daemon.sock.lock`, beside its socket.
- It owns the projects, tabs, panes and layout. Anything a window does, a command can
  do, and each sees the other's changes at once.
- It does no work while idle.
- On Linux with systemd, a daemon started by a window or command runs in a scope of its
  own (`insensical-daemon-…` in `systemctl --user`), so closing the terminal that started
  it, or ending that session, takes neither it nor your programs. A logout that stops
  your user manager does, unless lingering is on (`loginctl enable-linger`). On a
  different socket, `INSENSICAL_SCOPE=1` gives the same scope and `INSENSICAL_SCOPE=0`
  turns it off.
- **Stopping it kills your programs.** `isc kill-server` or a termination signal ends
  every program in every pane and keeps the layout for the next start. Moving it to a
  new version stops nothing. Section 17 covers both.

**Who can reach it.**

- Only your user can connect. It listens on its socket and on no network.
- **Any program running as you can drive it through the socket**: open and close panes,
  type into them and read them.
- For another machine, go through `ssh`: see
  [Other machines, over ssh](#other-machines-over-ssh). For a browser, start `isc hub`,
  a separate program that listens on this machine only: see [In a browser](#in-a-browser).
  `isc` and the interface in a terminal can use another machine's daemon, and the window
  shows other machines' projects beside this computer's: see
  [Other machines in the window](#other-machines-in-the-window).

**The socket.**

- Its directory must be owned by you and closed to others (`0700`). The daemon creates it
  that way. It refuses to start, and says why, if the directory is wider, is a link, or
  holds something that isn't a socket where the socket goes.
- Paths are limited to about 108 bytes on Linux and 104 on macOS. A longer
  `INSENSICAL_SOCKET` fails with "path must be shorter than SUN_LEN".
- With neither `INSENSICAL_SOCKET` nor `XDG_RUNTIME_DIR` set (`TMPDIR` on macOS), `isc`
  fails with "no per-user runtime directory".

### The daemon's environment

**The daemon has the environment of whatever started it, and every pane inherits it.**

- Started from a launcher, panes get the launcher's `PATH`, not a login shell's. A
  command other terminals find might not be found.
- **On macOS, a new pane's shell starts as a login shell**, like the Mac's other
  terminals, so it reads `/etc/zprofile` and `~/.zprofile`. A `shell` setting runs exactly
  as written and needs its own `-l`.
- A variable you change in a shell profile reaches new panes through the shell's own
  start-up files, or after `isc kill-server` and a restart from an environment that has
  it. That kills every program.
- Moving the daemon to a new version keeps its environment, as it's the same process.
- `INSENSICAL_HOST` isn't passed on, so `isc` in a pane means that pane's own daemon.
- Ignored signals aren't passed on. Programs in panes start with Ctrl-C, Ctrl-Z and
  hang-up meaning what they usually mean.
- **Started by the user unit**, the daemon has systemd's user manager environment, which
  reads no shell profile. `PATH` is usually short, and variables from `~/.profile` or a
  shell's start-up file are missing. `systemctl --user show-environment` prints what it
  has. To add to it, run `systemctl --user import-environment PATH` before the unit
  starts, or put `Environment=` lines in a drop-in from `systemctl --user edit insensical`.

---

### The hub, machines and devices

These let you reach your machines from one place.

- One of your machines is your **hub**: where `isc hub` runs, reachable by your devices
  on your network, your tailnet or the internet.
- Your other **machines** join it: `isc hub invite` on the hub makes a line, and
  `isc hub join` on the other machine takes it. A joined machine listens on nothing. Its
  `isc hub` dials the hub, and your devices reach it there.
- A **device** pairs with the hub once, by a code, and is known by its key from then on.
  It reaches every joined machine with no further code, and its page lists them.
- What a device and a machine say to each other is sealed for the two of them. The hub
  passes it on and can't read it. It vouches for paired devices, and each joined machine
  lets those in.
- The daemon still listens on nothing but its socket. `isc hub` is one more client of it,
  like the window. See [In a browser](#in-a-browser) and
  [Joining another machine to your hub](#joining-another-machine-to-your-hub).

## 5. The window

The window is where you see and work in your projects. This section covers what's on
screen, how the window opens, and how it shows other machines.

### First launch

- With no daemon running, the window opens one project with one tab and one pane running
  your shell.
  - Start it in a directory other than your home directory, and that directory becomes the
    project, named after it.
  - Start it from a launcher and it opens in your home directory (on a Mac, `/`). The
    project has no root and gets a generated two-word name such as `amber-fjord`. Its panes
    start in your home directory.
- With a daemon already running, the window shows what the daemon has.
- The window opens centred, at the size it had last time, or 1200×800 the first time. It
  doesn't remember where on the screen it was.
- Each `insensical` process has one window. Two processes give you two windows on the same
  daemon: see [Two windows](#two-windows).

### Parts

- **Sidebar**, on the left: a row of tools, a field, then the project tree. Shown with
  `tabs = "sidebar"`.
- **Top bar**, shown with `tabs = "top"`: the sidebar button, then where you are (the
  project's tile and name, a slash, the project's tabs), then `+` for a new tab. On the
  right: a settings problem, the update notice, the notice about agents that don't report
  yet (section 11), the count of panes that need you, and a count and switch for floating
  panes when the tab has any.
- **The tab's area**: tiled panes with dividers, and floating panes above them.
- **Dock**, at the bottom, only while the tab has minimized panes.

Where this manual says something is "in the top bar", that's where it is with `tabs =
"top"`. With tabs down the side, the same things are at the top and bottom of the sidebar.

**Tabs run along the top or down the side, never both.** The `tabs` setting chooses.
`Ctrl+Shift+B` hides them, and shows them again in the same place.

| `tabs` | What you get |
|---|---|
| `"sidebar"` (default) | The sidebar lists every tab of every project, which suits watching work across several projects. No top bar: the count of panes that need you, the floating-panes switch and `+` are at the top of the sidebar, and notices at its bottom. |
| `"top"` | The top bar shows the current project's tabs only, which suits working in one project. No sidebar. |
| Hidden | Panes get the whole window. The count and notices are off screen, but `Ctrl+Shift+A` still goes to the pane that needs you and the palette still lists them. The window opens with tabs hidden next time. |

### With nothing open

A window with nothing to show offers four buttons, each with its key: a new tab, a new
project, the command palette, and the sheet of every key. A setting that couldn't be used
shows in red at the foot of the sidebar, or in the top bar; click it to see every such
problem.

### What resizes terminals and what does not

Terminals resize when their room changes: showing or hiding the sidebar, dragging its
edge, the dock appearing, hiding pane headers, or changing `sidebar`, `pane-header`,
`density` or `pane-gap`. Things that float over them (the palette, the overview, search,
zoom, the leader's panel, the restart question) don't.

### What the window does not remember

The window remembers its size, whether tabs were hidden, the sidebar's width, and which
projects and tabs you folded. It doesn't remember its position, and a new window always
opens with its keys unlocked.

### Two windows

A pane has one size, so two windows of different sizes can't both fit it. **A pane is the
size that fits the window you used last.** The same goes for a window and `isc attach`.

- Click or switch to a window and the panes it shows take its size. The other window keeps
  showing them at that size.
- A pane sized by another window has a "Sized elsewhere" mark in its header. Press it to
  size the pane for this window.
- The pane shows at its real size. If it's larger than its space you see its foot, where
  the cursor is. Hold Shift and turn the wheel to move across it, or Alt and the wheel to
  move up and down.

### Other machines in the window

One window shows this computer's projects and those of every machine you've added. A pane
on another machine works like any other: type in it, split it, float it. It runs there and
keeps running if you close the window or the connection drops. It all goes through your
own `ssh`: see [Other machines, over ssh](#other-machines-over-ssh).

**Adding a machine.** Press "Add" at the foot of the sidebar and choose Machine. "Add a
machine" in the palette and "Add Remote Host" in the settings sheet do the same.

1. Under the Address field, pick `Tailscale` (your tailnet's machines, those that are on
   first) or `SSH config` (the names in `~/.ssh/config`). It opens on `Tailscale` when your
   tailnet has other machines.
2. Choose a machine, or type to narrow the list. Anything else you type goes to `ssh` as
   it is: a host, or `user@host`. `Tab` moves between the field and the two tabs, Left and
   Right switch tabs, Up and Down move through the list.
3. Press Enter. The dialog checks the machine, as `isc host check` does, and a failing
   line says what to do.
4. A machine that's ready is added at once, and its projects appear in the sidebar.

- **If `ssh` has a question, press `Answer`.** That covers a new host, a key or password,
  and a Tailscale sign-in. It opens a tab on this computer running `isc host answer` for
  that machine, where you answer `ssh` itself; insensical never sees what you type. When it
  ends well the machine is met again. The button is also under the machine's name in the
  sidebar and in Remote Hosts. A changed host key has no `Answer`: its words say what to
  find out first.
- **If insensical is missing or older, one button puts it there**, like "Put insensical
  0.3.0 on atlas". It writes `~/.local/bin/isc`, no sudo, only when you press it, and does
  what `isc host put` does.

**In the sidebar,** each machine is a heading over its projects once you've added one:
"This computer" first, then the others in your settings' order.

- A heading has the machine's tile and name in its colour, the link's mark, and how many
  of its panes need you. Press it to fold the machine.
- **The link's mark is three bars:** three under 60 ms round trip, two under 200 ms, one
  beyond. Point at it for the figure. It's struck through and red while the machine is out
  of reach, and pointing says why.
- **A machine with another version of insensical says so under its heading**, as in "atlas
  has insensical 0.2.1, and this computer has 0.3.0". "Update it" offers to put this version
  there. If the machine has the newer one, update this computer, then press "Try again".
- The filter, going to the pane that needs you, "Next project" and "Previous project" all
  cover every machine. The overview groups projects under each machine, and the palette says
  which machine a pane is on.

**Which machine you're typing into.** A pane on another machine shows the machine's chip
(tile and name, in its colour) in its header, with a line in that colour along the top. In
a narrow pane the chip is the tile alone. This computer's panes carry neither. A machine's
colour comes from its name unless you set one.

**A machine marked careful has a filled chip, and a paste of more than one line there asks
first**, whatever the program asked for. See [Pasting](#pasting). Mark the machine you'd
least like to paste into by mistake.

**One machine is in front at a time.** Keys and commands go to the machine whose pane, tab
or project you last went to.

- A pane, tab or project stays on the machine that runs it.
- "New project" asks which machine once there's more than one. `Alt+M` goes to the next,
  and the directories offered are that machine's.
- What you ask of a machine is never done on another. If it can't be asked, nothing
  happens and a line says so.
- Only you change the machine in front. Another machine can tell you a pane needs you, but
  can't take the keyboard.

**When the link is lost.** A train, a closed lid, a machine that restarts.

- The machine's panes keep what they last showed, dimmed, with a line at their foot: "Link
  to atlas lost · trying again".
- **Nothing you type there is sent, or kept for later.** The line beats once so you know
  the key wasn't taken. Commands that act on that machine do the same. Nothing you typed
  into a dead link turns up at an agent when the network returns.
- It retries by itself, and coming back to the window retries at once.
- When it's back, the panes are drawn afresh and say "Connected" for a second. What ran
  there never noticed.

**Settings.** The Remote Hosts page of the settings sheet has one card per machine. A
folded card shows its picture, name, state and `Check`. Press the row to open it. Open, a
card has:

| Control | What it does |
|---|---|
| Colour | One of ten, or the one its name gives |
| Icon | On a tile of its colour: server, laptop, desktop, mini PC, NAS, board (a small computer such as a Raspberry Pi), phone, container or cloud |
| careful | On or off |
| `Check` | Meets the machine again, as adding it did |
| Pencil | Renames it in its row: Enter keeps, Escape leaves. Link, folds and place in front stay. |
| `Forget` | Forgets it. What runs there runs on, and adding it again finds it. |

From the keyboard, Up and Down go between machines and "Add Remote Host", Left and Right
go along a machine's controls, and Enter presses the marked one.

The settings a server reads (the shell, scrollback, notifications and so on) and the
agents' hooks belong to each machine. In Hub, Terminal, Notifications and Agents, a
chooser at the top right says whose you're looking at, and `Alt+M` goes to the next.
Everything else on the sheet is this window's.

Limits:

- A pane or project can't be moved between machines, and two machines' panes can't be
  shown side by side.
- The interface in a terminal shows one machine at a time. A careful machine is marked in
  the top row, and a multi-line paste asks first.
- In "Go to a machine", a machine `ssh` has a question for says `ssh asks` on its row.
  Press `a` and a tab opens on this computer running `isc host answer` for it. A machine not
  reached for another reason says why, and `a` does nothing there.

**When this computer's server stops,** only its part of the window says so: its heading
gets the struck-through mark and "its server has stopped", and its panes are gone until it
starts. Other machines carry on. Press the heading for the way to start it again.

### Appearance

- **Fonts.** The UI uses the system's proportional font at 13 pixels, whatever `font-size`
  is. `interface-font` names another. If you name the terminal's font, the UI uses it too.
- **A program is shown as a tile**: its own mark (Claude's, Neovim's, Docker's and so on)
  on a small glossy card in the program's colours, so you can find a pane by colour before
  you read its name. Programs without a mark get a terminal's. Tiles appear in the sidebar,
  pane header, tab strip, dock, overview and palette.
- **A tab is shown as a deck**: the tile of the pane that has or last had the keyboard,
  with up to two more behind it. Floating and minimized panes count.
- **The foot of the sidebar** has Settings, Show every key, the overview of every tab, and
  "Add", a small menu of Project or Machine. With the sidebar hidden, these are at the right
  end of the top bar.
- **Menus** take Up and Down, Home and End, Enter, Escape, and a letter to jump to the
  next row that begins with it. In "Add", `P` and `M` choose.
- `density = "comfortable"` (default) draws each pane as a rounded card with space around
  it. `"compact"` puts panes edge to edge with a hairline between them.
- `pane-gap` sets the space between panes, 0 to 40 pixels. Without it, the gap is 6 when
  comfortable and 0 when compact.
- The focused pane has an accent border. Unfocused panes aren't dimmed.
- **Motion is small.** Sheets and notices fade in, a pane you float or bring back clears
  from a light wash, a cursor blinks and a working mark turns. `motion = "off"` stops all of
  it. `"system"` (default) follows the desktop's reduced-motion setting, and on macOS Reduce
  Motion.
- A state is never shown by colour alone: each has a mark and a word.
- **Screen readers** hear the window's chrome: buttons by what they do, tabs, the sidebar
  as a tree, switches, menus, and each sheet and question by name. Terminal content isn't
  exposed.
- **The keyboard in a sheet or a question.** `Tab` and `Shift+Tab` go through its controls
  and round, never out to what's behind. `Space` or `Enter` presses the control with the
  ring, and `Escape` closes. When it closes, the keys return to where they were.

---

## 6. Projects and the sidebar

Projects keep your work in groups, and the sidebar lets you see all of it and jump to any
part.

### Projects

A project is a name, an optional root directory, and tabs. It stays until you close it,
even with nothing running, and comes back after the daemon restarts.

**Closing a tab or a project asks first**, because it kills every program in it. The
window lists the panes that would close, each with its program, directory and state (just
a count above six). Enter, "Close tab" or "Close project" closes. Escape, "Keep it" or a
click outside doesn't. A project with nothing running closes without asking. Closing a
pane doesn't ask, and neither do `isc tab close` or `isc project close`: typing the
command is your yes.

**Opening one.** Press `Ctrl+Shift+N`, or click the project's name in the top bar. You're
asked for:

- **Name**, optional. With a directory and no name, the project is named after the
  directory. With neither, it's given a name.
- **Directory**, where its tabs open. It starts as the directory of the pane you're in.
  Type a path beginning with `/` or `~`, or leave it empty for none.

As you type a path, up to six matching directories are listed (hidden ones once you type a
dot). Up and Down select one, and `Tab` goes into it, so a path takes a few letters and
`Tab` at each step.

If the path doesn't exist but could, the card shows the whole path it worked out and the
button reads "Make the directory". Enter asks once more ("Make it and create"), and Enter
again makes the directory and any missing ones above it, then opens the project in it.
It's made on the machine the project is for, with permissions 755 less your `umask`. If it
can't be made, the card says why and nothing is made.

`Tab` in the name field moves to the directory field, and `Shift+Tab` moves back. Enter or
"Create" makes the project and opens a shell in it. Escape, "Cancel" or a click outside
makes nothing.

`Ctrl+Shift+L` puts the keyboard in the sidebar's field, showing the sidebar if hidden
(with `tabs = "top"` it opens the palette instead). That field only narrows the tree:
Enter on an existing project's name goes there, and nothing typed makes a project.

From the command line:

```sh
isc project new [DIR] [--name NAME] [--make]
isc run --new-project
```

`--make` creates a `DIR` that isn't there, with the ones above it, on the machine the
command is for. To open a project with all its tabs and panes at once, see [A project in a
file](#a-project-in-a-file).

**Where new panes open.**

| New | Opens in |
|---|---|
| Tab | The project's root. With no root, the directory of the project's front pane, and your home directory only if there's none. |
| Split | The directory of the pane it splits |
| Floating pane | The focused pane's directory |

A project's name doesn't follow its panes: a project named `insensical` stays that if its
shells move to `~/Projects/quote`. Tab and pane rows show where they really are.

**Renaming.** Press the pencil on the project's sidebar row, run `isc project rename OLD
NEW`, or choose "Rename this project" in the palette. The name becomes a selected field:
Enter keeps it, Escape leaves the old name, and an empty name isn't allowed. If the row
isn't on screen, a small card asks instead.

**A project's icon.** Every project has a tile beside its name. By default it's the first
letter of the name on a colour that comes from the name. To choose, press the tile on the
project's sidebar row. Each pick applies at once.

- One of ten colours, or "By name" for the one it had.
- A mark: type to find one, such as `git`, `rust` or `docker`. The first tile is the
  letter again. Next are insensical's own marks: `folder`, `code`, `web`, `database`,
  `docs`, `mobile`, `package`, `rocket`, `lab`, `server`, `design`, `chart` and `shield`.
  After them are the marks programs are shown by.
- **"Your SVG"** takes a `.svg` file of yours, 256 KB at most, drawn whole in a square.
  The daemon keeps a copy in `~/.config/insensical/icons/`, so you can move or delete the
  original.
- "Reset" goes back to the letter and the name's colour.

A drawing with an `<image>` in it, or a gzip-packed one, isn't taken.

From the command line, `isc project icon NAME --mark rust --colour '#ff8800'` or `--svg
logo.svg`. What you leave out is left unchanged, so `isc project icon NAME` alone resets
it. The file is read where you run the command, so `isc --host build project icon api
--svg logo.svg` takes a `logo.svg` from this machine. The icon is kept with the layout,
and every window shows it.

| To | Do |
|---|---|
| Put projects in order | Drag a project's row onto another's, or `isc project move NAME INDEX` (counted from 0) |
| Close | Press the cross on its row (asks first), or `isc project close NAME` |
| Move between projects | `Ctrl+Alt+PageDown` or `Ctrl+Alt+PageUp`, click a row, or `isc project select NAME` |

On the command line, a project is its name, or its number as `isc ls` prints it (`4` or
`project:4`). Two projects can share a name; the name then means the first listed, and the
number tells them apart. A number always means that project, never one named that.

### A project in a file

Write a project down, with its tabs, panes and what each runs, and **`isc project open`
makes the whole thing in one go.** Keep the file in the project's directory as
`.insensical.toml` and commit it, and everyone who clones the repository gets the same
project with the same command.

```sh
isc project open                      # the .insensical.toml in this directory
isc project open ~/code/shop          # the one in that directory
isc project open ~/layouts/shop.toml  # a file by name, kept anywhere
```

It prints the project as `project:N` and goes to it.

It prints the project as `project:N` and goes to it.

**Two panes side by side.** This is a whole file:

```toml
name = "shop"

[[tab]]
columns = ["nvim", "cargo watch -x test"]
```

**One big pane and two stacked beside it.** A list inside a list goes the other way, so
inside `columns` it's a stack. `size = 2` gives the editor twice the room of the stack.
`""` is a shell.

```toml
name = "shop"

[[tab]]
name = "work"
columns = [{ run = "nvim", size = 2 }, ["cargo watch -x test", ""]]
```

**A morning's work.** Two tabs, two agents each in its own directory, a floating pane, and
a pane that starts minimized:

```toml
name = "shop"

[[tab]]
name = "work"
columns = [
  { run = "nvim", size = 2 },
  [
    "cargo watch -x test",
    { run = "claude", dir = "api", name = "api agent", front = true },
    { run = "claude", dir = "web", name = "web agent" },
  ],
]

[[tab.float]]
run = "lazygit"
at = [0.1, 0.1, 0.8, 0.8]

[[tab]]
name = "servers"
rows = [
  { run = ["sh", "-c", "npm run dev | tee dev.log"], dir = "web" },
  { run = "docker compose up", minimized = true },
]
```

**What a file can say.** Every key is optional. A key it doesn't know is a mistake, not
skipped.

| Where | Key | What it means |
|---|---|---|
| The project, at the top | `name` | The project's name. Without it, the project is named after its directory. |
| | `root` | The directory its panes start in. Without it, the directory the file is in. A path not beginning with `/` is taken from where the file is, and `~` is your home. |
| A tab, under `[[tab]]` | `name` | The tab's name |
| | `front = true` | This tab is in front. Without it, the first tab is. |
| | `columns = [ ]` | Its panes side by side, left to right |
| | `rows = [ ]` | Its panes stacked, top to bottom. A tab has `columns` or `rows`, not both. |
| | `run`, `dir` | For a one-pane tab without a list: what it runs and where. A tab with none of these is one shell. |
| A pane, in a list | `"command"` | The command, in quotes. `""` is a shell. |
| | `{ run = "command" }` | The same, with room for the keys below |
| | `dir` | Where it starts: a directory from the root, or a whole path |
| | `name` | The pane's name |
| | `size` | Its share of the list's room. Size 2 has twice the room of size 1, which is the default. |
| | `front = true` | This pane has the keyboard. Without it, the tab's first pane has. |
| | `minimized = true` | It starts in the dock, with its program running |
| A list in a list | `[ ]` | Panes going the other way: rows inside columns, columns inside rows. Nest as deep as you need. |
| | `{ rows = [ ], size = 2 }` | The same when the group needs a size, or `{ columns = [ ] }` |
| A floating pane, under `[[tab.float]]` | `run`, `dir`, `name`, `front`, `minimized` | As for any pane |
| | `at = [x, y, width, height]` | Where it floats, in parts of the tab from 0 to 1. Without it, `[0.2, 0.15, 0.6, 0.7]`. |
| | `pinned = true` | It stays above floating panes that aren't pinned |
| | `everywhere = true` | It shows over whichever tab of the project is in front |

**A command is words, not a shell line.** `run = "cargo watch -x test"` starts `cargo`
with three arguments, and quotes keep a word whole. Nothing else means anything: no `&&`,
no `$HOME`, no `~`. For those, write `run = ["sh", "-c", "make && ./serve"]`. A pane
closes when its program ends, as with `isc run`.

**It asks before it starts someone else's programs.** The first time you open a file,
`isc` lists every program it would start, with each directory, and asks. Only `y` starts
them.

- Your yes is remembered for that file at that path. If it changes, or you copy it
  elsewhere, you're asked again.
- `--yes` skips the question and counts as your yes for next time.
- With nobody to ask (a script, a pipe), an unapproved file isn't opened. A file that only
  opens shells isn't asked about.

**Opening it twice doesn't make two.** If a project with the file's name and root is
already open, `isc project open` goes to it and changes nothing. `--again` opens a second
one.

**A mistake makes nothing.** The whole file is checked first. A misspelt key, a wrong kind
of value, broken TOML or a missing `dir` is reported with its line, and no tab is made:

```text
isc: /home/me/shop/.insensical.toml, line 5: a pane has no `sise`. It has `run`, `dir`,
`name`, `front`, `minimized`, `size`. Nothing was opened
```

If the daemon refuses something partway, what was made is closed again and you're told why.

**Saving one.** Arrange a project the way you like it, then:

```sh
isc project save              # writes .insensical.toml in the project's directory
isc project save shop.toml    # or a file you name
isc project save -            # or prints it
```

It writes the project in front: tabs, panes, split sizes, floating and minimized panes,
names, and which tab and pane are in front. A file saved in the project's own directory
names no `root`, so it works for whoever has the directory. It won't overwrite a file
without `--force`.

- Each pane is saved with the command it was started with. A pane started as a shell is
  saved as a shell, and if it's running something, `isc project save` tells you which pane
  so you can add a `run`.
- Not saved: the scratch pane, a zoomed pane, floating panes you hid, the project's icon,
  and what is muted.

**In the window.**

- **Make a new project (`Ctrl+Shift+N`) in a directory that holds `.insensical.toml`, and
  the window opens it as the file arranges it.** If the file starts programs, it lists each
  and asks. "Open as written" (Enter) opens the file. "Just the directory" (`D`) makes an
  ordinary one-shell project. Escape makes nothing.
- Your yes is shared with the command line, and a changed file asks again.
- "Open a project file" in the palette asks for a directory and opens the file in it.
  "Save this project as a file" writes `.insensical.toml` in the project's directory, asking
  first if one is there.
- A mistake in the file shows in a notice with its line, and nothing is made.
- The window always makes a new project, even if one from the file is open. A name you
  type is used if the file gives none.

**Limits.**

- The window opens and saves files on this computer only, and only `.insensical.toml` in a
  directory. For another machine or file name, use `isc project open FILE` and `isc project
  save`.
- The window doesn't tell you which saved shells are running something. `isc project save`
  does.
- A file starts 100 panes at most.
- With `--host`, the file is read here and the project made there. `root` must then be a
  whole path on that machine, and directories aren't checked first. `isc project save` needs
  the file's name there, and always writes `root`.
- A file can't keep a pane open after its program ends.

### Putting a project to sleep

A project you aren't working on still holds what its programs hold: a few megabytes for a
shell, several hundred for an agent. **Putting it to sleep ends its programs and keeps
everything else.**

- Tabs, splits, floating and minimized panes stay, and so does what each pane showed,
  which is written to disk. In the sidebar the heading says `asleep` and its rows are
  dimmed.
- **Waking it** puts a shell in each pane, in its directory, under what it showed. What
  ran there is typed at the prompt for you to run again with Enter: an agent's own resume
  command if it gave one, otherwise the command the pane was started with. This is also what
  happens to every pane when the daemon restarts ([section
  17](#17-stopping-restarting-and-new-versions)).
- A project that's asleep when the daemon stops is asleep when it starts.

To sleep one, hover its heading and press the sleep mark; or choose "Put this project to
sleep, or wake it" in the palette (command `sleep-project`, no default key); or run `isc
project sleep NAME`. The window shows what would end and asks first. To wake it, select it
and press Enter or `Wake it`, or use the same mark, command or `isc project wake NAME`.

- insensical never puts a project to sleep by itself.
- You can't start a pane in a sleeping project. Wake it first.
- Closing a sleeping project removes it and what was kept of its panes.
- With `keep-scrollback = false` a waking project has its panes and directories but not
  what they showed.

### The sidebar

The sidebar is a tree of project, tab and pane. It shows while `tabs = "sidebar"`, and
`Ctrl+Shift+B` hides and shows it. Drag its right edge to change its width, from 190 to
520 pixels (at least 240 are left for the panes). The next window opens at the width you
left.

| Row | Shows |
|---|---|
| Project | A fold mark and the name, brighter for the current project. On the right: memory its programs hold, and its pane count. Folded, instead: how many of its panes need you, with the most urgent mark. |
| Tab | A fold mark, the tab's deck of tiles, then its name, or what its focused pane is called. Folded, on the right: its most urgent pane's mark. |
| Pane | The program's icon, then what the pane is called. On the right: a mark if floating or minimized, and its state. |

**What a pane is called** is the same in the sidebar, on a dock chip, in the palette and
in a close question:

1. the name you gave it, if any;
2. otherwise the title its program set, unless it's a shell (an agent's title says what
   it's working on);
3. otherwise the program and the last part of its directory, such as `nvim · api`. A shell
   is called by its directory alone, such as `api`. The home directory is `~`.

- A tab with one pane and no name has no row: its pane's row stands in for it.
- **A pane with something to say shows it on a second line**, starting with the state in
  the state's colour: "wants permission", "asks a question", "needs you to sign in",
  "waiting", "failed" or "done". The program's own words follow. Hover for the whole text.
- A working pane shows progress and words after its name when they fit, as in "40% ·
  Pushing image", and on their own line when they don't. A pane only running a command, or
  at rest, has its mark and name.
- **A row says how long its pane has needed you**: `now` for the first minute, then `2m`,
  `1h`, `3d`. After ten minutes waiting, the figure turns bold in the waiting colour.
- Click a row to go to it. Click a minimized pane's row to bring the pane back.
- **What an agent started is listed under it**, one level in and joined by a line,
  whichever tab it's in. A tab whose panes all sit under their agents has no row of its own.
  Folding the agent's tab or project folds its team.

**Sharing.** An eye appears beside the pencil on a project's row under the pointer. It
shares the project with a guest, or lists who has it: see [Sharing one project or one
terminal](#sharing-one-project-or-one-terminal). A shared project always shows an eye, or
a keyboard if a guest can type in it.

**Naming.** Press the pencil at the end of the row under the pointer. The name becomes a
selected field: Enter keeps it, Escape leaves the old name. For a tab or pane, an empty
name removes the name. Names survive a restart and appear in the tab strip, the pane's
header, notifications and `isc ls`.

- From the command line: `isc tab rename TAB NAME` or `isc rename PANE NAME`.
- From the palette: "Name this pane" or "Name this tab" (`name-pane`, `name-tab`, no
  default keys). Where the name isn't written on screen, such as a one-pane tab with no
  name, a small card asks.

**Folding.** The mark at the start of a project or tab row hides what's under it. The next
window opens with them folded. While the filter has text, matches show even inside
something folded.

**Filtering.** Type in the field to narrow the tree to projects, tabs and panes whose
name, title, program or directory contains what you typed. Case doesn't matter.

**What is listed.** The `sidebar` setting chooses which levels appear under a project's
name: `["tabs", "panes"]` (default), `["tabs"]`, `["panes"]`, or `[]` for project names
alone.

**Showing only activity.** Press the pulse-line button at the head of the sidebar, or
choose "Show only activity in the sidebar, or everything" in the palette (`activity`,
`Ctrl+Shift+Y`), to list only panes that are agents or have something to say. Shells,
editors and monitors go. Press it again and everything is back. The window remembers your
choice. The number beside the button is how many panes need you.

Each row shows the pane's name, how long it has been as it is, and under the name its
state and what its program says, as `wants permission · Allow the edit?`. An agent another
agent started says whose it is, as `waiting · from Claude`, and one that started others
counts them, as `working · 3 started`. Press a row to go to that pane. In each project,
panes that need you come first, then those at work, then finished, then agents at rest. A
machine with nothing to list says "nothing at work".

The list is kept calm: it doesn't flicker as work starts and stops, and rows don't move
while your pointer is moving over it.

- A long command at a shell that marks its commands (`isc shell-integration` sets bash and
  zsh up) gets a row with what you typed and how long it has run.
- A finished or failed pane stays until you look at it. A waiting pane stays until its
  program says otherwise. If the program is gone and the row is stale, point at it and press
  the arrow "Forget what this pane's program said it was doing".

The filter and arrows work as in the full list. Folds and the `sidebar` setting don't
apply. Set `activity-idle = false` to leave out agents at rest. Two commands walk the list
from anywhere, round at the ends and across machines: `next-activity` (`Ctrl+Shift+J`) and
`previous-activity` (`Ctrl+Alt+/`). While the sidebar shows activity, `Ctrl+Shift+A` goes
to the next pane that needs you or has finished, in the order drawn.

With `tabs = "top"` the button is in the bar and the list shows beside the panes while
it's on. The interface in a terminal has the same list under the same rules: see [The
whole interface in a terminal](#the-whole-interface-in-a-terminal). So does the page `isc
hub` serves (see "In a browser"), though there a finger counts as the pointer, agents at
rest are always listed, and there's no filter or forget arrow.

**From the keyboard.** `Ctrl+Shift+L` (leader `e`) puts the keyboard in the field. Down
and Up move a highlight through the rows, or the matches after you've typed. Left folds
the highlighted project or tab, Right unfolds it. Enter goes to the row and gives the
keyboard back to the pane; with none highlighted, it goes to the project whose name you
typed. Escape leaves the field.

**Drag a row to put it in order.** A project dragged onto another takes its place, and a
tab dragged onto another tab of the same project takes that tab's place. A one-pane tab
with no name is dragged by its pane's row. Panes aren't ordered here: their order is the
layout's.

---

## 7. Tabs

Tabs let you keep a separate layout of panes for each thing you're working on. A tab lasts
as
long as it holds a pane: close its last pane and the tab closes.

| To | Do |
|---|---|
| Open a tab | `Ctrl+Shift+T`, the `+` in the top bar, or `isc tab new`. It opens a shell in the project's root, or where the pane in front is if the project has no root. |
| Switch | Click the tab; `Ctrl+Tab` / `Ctrl+Shift+Tab`; `Ctrl+PageDown` / `Ctrl+PageUp`; `Alt+1` to `Alt+9` for the project's first to ninth tab (`⌘1` to `⌘9` on macOS, leader `1` to `9`); `isc tab next`, `isc tab prev`, `isc tab select TAB`. |
| Close | The cross on the tab, a middle click, `Ctrl+Shift+Q`, or `isc tab close [TAB]`. The window asks first. |
| Move | `Ctrl+Shift+PageUp` / `Ctrl+Shift+PageDown`; `isc tab move TAB INDEX`, counted from 0; or drag the tab onto another, along the top or in the sidebar. |
| Name | The pencil on its sidebar row, "Name this tab" in the palette, or `isc tab rename TAB NAME`. |

- **Closing a tab closes every pane in it.** The window shows what would end and asks first.
  `isc tab close` doesn't ask.
- Drop a tab onto another and it takes that tab's place.
- A tab shows its panes as tiles, with the focused pane's program in front, and its name or
  what that pane is called ([section 6](#6-projects-and-the-sidebar)). It shows a state mark
  only when one of its panes needs you.

---

## 8. Panes

A pane is one running program in a terminal. You can split, float, minimize and move panes
to arrange your work the way you like.

### The life of a pane

A pane runs the shell, or the command you give to `isc run` or `isc split`.

- **When its program ends, the pane closes.** A pane started with `isc run --keep` stays
  open
  and readable, and shows how the program ended (`done`, or `failed` with the exit code).
  Press Enter in it to close it.
- **Closing a pane ends its program, without asking**, even if something is running. The
  program gets a hang-up, then its terminal closes after 300 ms, and its process group is
  killed after three seconds.
- Close a pane with `Ctrl+Shift+W`, the close button in its header, or `isc close [PANE]`.
- When the focused pane closes, the keyboard goes to the pane that takes its space.
- A fault inside one pane closes only that pane.

Limits: a window or terminal shows at most 512 panes at once (the rest say "Not shown yet"
and
retry on their own), and a pane isn't shown if its terminal would have more than 1,024,000
cells.

### Which shell runs

A pane with no command runs the first of these that applies:

1. The `shell` setting, if set: a program and its arguments split at spaces, such as
   `"fish"` or `"/bin/zsh -l"`. A path can't contain a space.
2. `$SHELL` as the daemon sees it.
3. `/bin/sh`.

- A change to `shell` applies to the next pane, with no restart; open panes keep theirs.
- If your shell can't start, the login shell starts instead and the top bar says what went
  wrong.
- `shell` changes what new panes run, not `$SHELL` inside them.

### What is known about a pane

- **Program**: the foreground process's name. A pane that has printed and been sent
  nothing has
  no known program yet and shows the generic terminal icon.
- **Directory**: read from the pane's own process, so it's right with any shell and needs no
  setup.
- **Title**: what the program set, without a busy mark such as an agent's spinner, because
  the
  pane's state has its own mark. `isc ls` shows the title exactly as set.
- **Tile**: chosen from the program's name. About six hundred programs have their own mark,
  in their maker's colour; anything else gets the terminal's. To give your own program one,
  see [Icons for your own programs](#icons-for-your-own-programs). The list is fixed.

  | Kind | Programs |
  |---|---|
  | Agents | claude, codex, gemini, copilot, cursor-agent, opencode, pi, amp, goose, hermes, openclaw, nemoclaw, cline, qwen, kiro, kilo, roo, kimi, trae, codebuddy, qoder, jules, junie, devin, openhands, grok, antigravity, zencoder, vibe, and aider, crush, droid, auggie and paperclipai, which share one mark in different colours |
  | Models | ollama, lms, vllm, deepseek, hf, nvidia-smi |
  | Editors | nvim, vim, emacs, hx, zed, codium, subl, the JetBrains editors, xcodebuild |
  | Version control | git, lazygit, tig, gitui, jj, gh, glab, hg, svn, git-lfs, tea, gt |
  | Rust, Go, Zig | cargo, rustc, rustup, go, zig |
  | JavaScript | node, npm, pnpm, yarn, bun, deno, tsc, vite, webpack, esbuild, next, nuxt, astro, ng, eslint, prettier, jest, vitest, turbo, nx, tailwindcss, gulp, babel, biome, sass, mocha, puppeteer, electron, tauri, expo, nest, ember |
  | Python | python, ipython, pip, uv, ruff, pytest, jupyter, poetry, conda, django (`manage.py`), flask, uvicorn, gunicorn, celery, streamlit, scrapy, pdm, rye |
  | Sites and services | hugo, jekyll, gatsby, docusaurus, mkdocs, vitepress, wp, drush, firebase, stripe, shopify, newman, http (HTTPie), rover |
  | Ruby and PHP | ruby, irb, rake, bundle, rails, gem, php, artisan (Laravel), composer, symfony |
  | Other languages | lua, perl, R, julia, nim, crystal, elixir, erlang, ghc, ocaml, kotlin, swift, scala, clojure, dart, flutter, dotnet, java, gradle, mvn, cmake, make, gcc, clang, gleam, gfortran, racket, sbcl, elm, solc, octave, sage, groovy, typst, latex, pandoc, bazel, conan, just |
  | Databases | psql, pgcli, mysql, mariadb, sqlite3, redis-cli, mongosh, clickhouse, duckdb, supabase, prisma, cqlsh, influx, cypher-shell, cockroach, etcdctl, rabbitmqctl, nats, pscale, turso, surreal, snowsql, bq, trino |
  | Data and learning | spark-submit, pyspark, airflow, prefect, kafka-topics, hadoop, mlflow, dvc, wandb, tensorboard, kaggle, modal, quarto |
  | Containers and clouds | docker, podman, kubectl, k9s, helm, terraform, tofu, ansible, vagrant, pulumi, gcloud, wrangler, vercel, netlify, fly, nginx, caddy, tailscale, doctl, hcloud, railway, serverless, sst, k3s, istioctl, argocd, flux, nerdctl, lxc, incus, qemu, VBoxManage, consul, vault, nomad, packer, prometheus, grafana, traefik, httpd, jenkins, act |
  | Security and networks | op, bw, keepassxc-cli, openvpn, mullvad, tor, tshark, msfconsole, openssl, certbot, ngrok, snyk, trivy |
  | The system | pacman, yay, apt, dnf, nix, brew, flatpak, tmux, curl, ffmpeg, mpv, htop, gpg, snap, zypper, apk, emerge, rpm, yum, pkg, port, rclone, borg, syncthing |
  | Devices and apps | adb, fastboot, arduino-cli, pio, ros2, pod, fastlane, yt-dlp, vlc, blender, godot, firefox, obsidian |
  | Remote | ssh, mosh, sftp, scp, rsync |
  | Monitors | btop, top, nvtop, glances, btm |
  | Shells | sh, bash, zsh, fish, nu, dash, ksh, pwsh |

  A tool run by a language's program is known by its own name: Vite runs as `node` but the
  pane is called `vite`, and likewise `rails`, `artisan` and `manage`. A script of your own
  stays `node` or `python`. A shell script is known by its own name, `deploy` for
  `./deploy`.

### The header

The header shows, in order: the icon; the title, or the name you gave the pane; the
directory, unless the title has it; a mute mark; a crossed-out robot while the pane keeps
agents out (see [Agents](#11-agents)); an eye or a keyboard while a guest can look at the
pane or type in it; the state as a word, when the pane isn't idle; a percentage, when the
program reports progress; what the state is about; and "Enter closes" on a kept pane whose
program has ended.

Buttons show on the focused pane and the pane under the pointer: split right, split down,
float (on a float: put into the layout, keep on top, show over every tab), minimize, fill
the
tab, share (an eye: see [Sharing one project or one
terminal](#sharing-one-project-or-one-terminal)),
close. A pane on another machine has no share button.

A two-pixel line under the header shows the state as a colour.

The `pane-header` setting:

| Value | Header |
|---|---|
| `full` (default) | Everything above |
| `name` | The same, without the buttons |
| `hidden` | No header; the terminal uses the space |

A floating pane keeps its name even with `hidden`, because you drag it by that bar. Changing
this setting resizes the terminals.

### Splitting and focus

- **Split right / down**: `Ctrl+Shift+O` / `Ctrl+Shift+E`, the header buttons, or
  `isc split right|down [PANE] [-- COMMAND…]`. The new pane takes half of the pane it
  splits.
  With a floating pane focused, the tiled layout is split.
- **Focus by direction**: `Ctrl+Alt+arrow`, or `isc focus left|right|up|down`, across tiled
  panes.
- **Next / previous pane**, floats included: `Ctrl+Shift+]` / `Ctrl+Shift+[`.
- Click a pane to focus it. `isc focus PANE` gives a pane the keyboard.

### Resizing

**With the mouse**, drag the divider between panes. Ratios stay between 5% and 95%.

**With the keyboard**, `Ctrl+Alt+Shift+arrow` moves the focused pane's edge that way, by 5%
of the space being divided. From the command line:
`isc resize left|right|up|down [PANE] [--by PERCENT]`.

- The arrow says which way the divider goes, not whether the pane grows. The divider that
  moves is the innermost one on that axis that holds the pane, so "right" widens a pane
  whose
  divider is on its right and narrows one whose divider is on its left.
- A pane as wide as the tab has no divider to move, so the key does nothing.
- On a floating pane, the same keys move its right and bottom edges.

### Moving and swapping

- "Move this pane left / right / up / down" swaps a tiled pane with its neighbour on that
  side, and the pane keeps the keyboard. On a floating pane it moves it a twentieth of the
  tab, up to the tab's edges.
- The commands (`move-left`, `move-right`, `move-up`, `move-down`) have no direct key. Use
  the palette or bind one in `[keys]`. After the leader they're `Ctrl+H`, `Ctrl+J`,
  `Ctrl+K`, `Ctrl+L`, and they repeat.
- `isc swap A B` swaps the places of two panes.
- `isc put PANE left|right|above|below --of OTHER` moves a pane to that side of another,
  half
  each. Without `--of` the pane takes that half of the whole tab, or of the tab `--tab`
  names.

**Drag a tiled pane by its header.** Where you let go says what happens:

| Let go | What happens |
|---|---|
| In the middle of another pane | The two swap places |
| Near a side of another pane: within a quarter of it, 160 pixels at most | The pane takes that half of the other's place |
| Within 24 pixels of the tab's edge | The pane becomes a column, or a row, at that side of everything |
| On a tab, along the top or in the sidebar | The pane moves to that tab, at the right of what's there |
| Anywhere else | Nothing changes |

- **You see it before it happens.** The panes glide to where they'd be, and the pane you
  carry is ringed, with a word in its middle for what letting go does: "Swap", "Top half",
  "Left of everything". You can let go as soon as the word is right.
- **Escape puts it back** while you're still holding it.
- No terminal is resized until you let go.
- A tab left with no pane is gone. You can't drag while one pane fills the tab.

### Filling the tab (zoom)

Press `Ctrl+Shift+Enter`, use the header button, or run `isc zoom [PANE] [--off]`. The pane
fills the tab and the others keep their size underneath.

### Floating panes

A floating pane sits above the layout and works like any other pane. You can float as many
as you like.

| To | Do |
|---|---|
| Open a new float | `Ctrl+Shift+D`, or `isc run --float [--rect X,Y,W,H]` |
| Float the focused pane, or put it back | `Ctrl+Shift+G`; `isc float [PANE] [--rect X,Y,W,H]`, `isc tile [PANE]` |
| Move | Drag the header |
| Resize | Drag any edge or corner |
| Keep on top of other floats | `Ctrl+Shift+K`, the header button, `isc pin [PANE] [--off]` |
| Put over the left half, right half, middle or whole tab | The palette commands `float-left`, `float-right`, `float-centre`, `float-fill`. They have no default keys. |
| Hide or show every float in the tab | `Ctrl+Shift+H`, the switch in the top bar, `isc floats show\|hide` |
| Hide one float, and bring it back | `isc hide [PANE]`, `isc show PANE` |

- A float stays inside the tab and is at least a tenth of it each way.
- `--rect` takes fractions of the tab: `--rect 0.5,0,0.5,1` is the right half. The default
  is
  `0.2,0.15,0.6,0.7`.
- Floating a tiled pane gives its space to its neighbours. Putting it back places it beside
  the focused tiled pane.
- Hidden floats keep running. A float closes when its command exits, like every pane.

**Over every tab.** To make a float show over whichever tab of the project is in front, in
the same place and size, use the button beside "keep on top" in its header, "Show this
floating pane over every tab" in the palette, leader `I`, or `isc follow [PANE] [--off]`.

- Closing a tab doesn't close it while the project has another tab.
- Minimized, it sits in the dock of the tab it was over.

### The scratch pane

A quick shell for a one-off command, without disturbing your layout. Press `` Ctrl+` ``
and a
shell floats across the lower part of the tab with the keyboard. Press it again to put it
away and give the keyboard back. After the leader the key is `` ` ``; from the command line,
use `isc scratch [--project NAME]`.

- There's one scratch shell per project, the same over every tab. What runs in it keeps
  running while it's away.
- Its header and sidebar row say "scratch".
- If it's on screen but another pane has the keyboard, the key gives it the keyboard
  instead.
- It's an ordinary pane: move, resize, rename, tile, minimize or close it.
- Programs in panes don't get `` Ctrl+` ``. To give it back, bind it to `none` in `[keys]`:
  ``"ctrl-`" = "none"``.

### Minimizing and the dock

Minimize a pane to get it out of the way while its program keeps running. Press
`Ctrl+Shift+M`, use the header button, or run `isc minimize [PANE]`. The pane leaves the
layout and keeps its size, so the program isn't disturbed.

- It waits in the dock as a chip: icon, title, percentage if reported, state mark, and a
  border in its state's colour. Chips stay in the order they were minimized.
- **Restore**: click the chip; `Ctrl+Shift+R` restores the one minimized last;
  `isc restore PANE`. A tiled pane returns beside its old neighbours with its old share. A
  float returns to its rectangle.
- A middle click on a chip closes the pane without asking.
- `minimize-others` and `restore-all` are palette commands with no default keys.
- Beyond about one chip per 170 pixels of the tab's width, idle minimized panes share one
  chip,
  "3 idle", which opens the palette. Working, waiting, done or failed panes keep their own.

---

## 9. The terminal

The terminal is libghostty-vt, Ghostty's terminal core, so escape sequences behave as they
do
in Ghostty. This section covers what you see and do in a pane: text, selecting, pasting,
links and pictures.

### `TERM`

`TERM` is `xterm-ghostty`, Ghostty's description of the terminal. It comes with insensical:

- If your system already has it, that one is used. Otherwise the daemon compiles its own
  with
  your system's `tic` the first time a pane starts.
- With no `tic`, `TERM` is `xterm-256color`.
- `term = "xterm-256color"` in the settings makes it that for every pane started afterwards.

**Over `ssh`** to a machine without that description, programs complain about an unknown
terminal. Install it there (`infocmp -x xterm-ghostty | ssh host tic -x -`), set
`TERM=xterm-256color` for that command, or set `term` if most of your work is on other
machines.

Every pane also has `COLORTERM=truecolor`, `TERM_PROGRAM=insensical`,
`TERM_PROGRAM_VERSION`,
`INSENSICAL_PANE` and `INSENSICAL_SOCKET`.

### Scrollback

Each pane keeps 10 MB of history, an amount of memory rather than a number of lines. The
`scrollback` setting changes it, from 1 to 200 megabytes, for panes started afterwards. The
daemon holds the history; section 17 covers what happens to it when the daemon stops.

- Quiet panes cost almost nothing, and a pane you aren't looking at costs the window
  nothing.
  The programs cost more than the panes: a shell is some megabytes, an agent can be several
  hundred. insensical doesn't limit or stop them.
- **The window shows what each project's programs hold.** On the right of a project's
  heading
  in the sidebar is the memory of everything running in it, and its share of a processor
  when
  it's using one: `1.2 GB · 40% cpu`. The overview shows it for each pane, and `isc usage`
  prints it for every pane.

**History from another machine comes a part at a time.** With `--host`, a pane shows its
screen and the newest half megabyte of history at once. Scroll to the top and the next four
megabytes are fetched, and so on.

- The top of the scrollbar's track is marked while there's more to come.
- Searching looks through what has come.
- If the pane changes size before it has all arrived, it's fetched afresh.

**From command to command.** `Ctrl+Shift+Up` scrolls to the prompt of the command before
what
you're looking at, and `Ctrl+Shift+Down` to the one after (`⌘↑` and `⌘↓` on macOS).

- It needs a shell that marks its prompts: fish 4 does, and bash and zsh do with the line
  `isc shell-integration` prints. Without marks the keys do nothing.
- After the last command, `Ctrl+Shift+Down` goes to the bottom.

### Text

- **Font.** `font-family` names the terminal's font. `font-size` is in points: the default
  is
  12, or 13 on macOS. On Linux a point is 1⅓ pixels, as most terminals count it, so the size
  from your other terminal looks the same here. The text around the terminals has its own
  font: see [Appearance](#appearance).
- **There's no font weight setting.** Text uses the family's regular face, and bold and
  italic use theirs. If your other terminal is set to semibold, text here looks thinner.
- **Ligatures.** Symbols side by side are drawn as the font's ligature, if it has one: `->`,
  `!=`, `<=`, `===`. `font-ligatures = false` turns that off. Only symbols join: letter
  ligatures such as `www` or `fi` aren't formed, nor are symbols a program gave different
  colours or weights.
- Characters the font lacks, emoji included, come from the system's fallback fonts. A wide
  Nerd Font glyph can overlap its neighbour.
- **Cursor**: block, bar or underline as the program asks, in the theme's cursor colour or
  the
  program's.
  - It blinks until a program asks for a steady one. Typing makes it visible and restarts
    the
    blink.
  - Only the pane your typing goes to has a solid cursor. Every other pane shows an outline
    that doesn't blink, as does every pane while the window isn't in front or a panel such
    as
    the palette has the keyboard.
- **Half-drawn screens are held.** When a program says it's mid-draw (synchronized output),
  its screen shows when it says it's done, or after one second.

### Colours

- The sixteen named colours come from the theme. 256 colours and true colour show exactly as
  the program gives them.
- A program that asks for the terminal's colours, or whether it's dark, gets the window's
  theme.
- **On a light theme**, text with less than 3:1 contrast against its background is painted a
  darker shade of the same hue. What you copy is unchanged. Dark themes are painted as
  asked.

### Selecting and copying

- Drag to select. Click twice to select the word, three times for the line. Hold Alt while
  you
  drag to select a rectangle. Hold the pointer beyond the top or bottom to scroll.
- Selecting copies to the selection clipboard on systems that have one (Linux).
- Press `Ctrl+Shift+C` to copy to the clipboard. The selection turns the accent colour for a
  second and "Copied" (or "Copied 14 lines") shows at the bottom of the pane. With nothing
  selected, nothing is copied. Wrapped lines are joined.
- `select-all` selects everything in the pane. It has no default key.
- Typing returns the view to the bottom and clears the selection.
- A selection belongs to the window you made it in and stays on its text as output arrives.

### Selecting with the keyboard

Move through a pane's screen and history, select and copy, without the mouse. Start with
`Ctrl+Shift+X` (`⇧⌘X` on a Mac), the leader then `v`, or "Select and copy with the keyboard"
in the palette. The same key leaves.

- **While it's on, every key is its own.** Nothing you press reaches the program.
- **It starts at the program's cursor**, usually the line you're looking at. A strip at the
  foot of the pane shows the keys that work right now.
- Output doesn't move you: your place and selection stay on their text.
- **Copying leaves.** `y` or Enter copies and ends it.
- **Escape lets go first, then leaves.** With something selected it drops the selection;
  with nothing selected it leaves. `q` always leaves. Pasting also leaves.

**Moving**

| Keys | Where the cursor goes |
|---|---|
| `h` `j` `k` `l`, or the arrows | Left, down, up, right |
| `w` `b` `e` | The next word, the word before, the end of the word |
| `W` `B` `E` | The same, counting everything between blanks as one word |
| `0` `^` `$`, `Home` `End` | The start of the line, its first character, its end |
| `g` `G` | The top of the history, and the end |
| `H` `M` `L` | The top, middle and foot of what's shown |
| `{` `}` | The blank line before, and after |
| `PageUp` `PageDown`, `Ctrl+B` `Ctrl+F` | A screen up or down |
| `Ctrl+U` `Ctrl+D` | Half a screen |
| `f` `t`, then a character | To that character on the line, or just short of it |
| `F` `T`, then a character | The same, going back |
| `;` `,` | To that character again, and the other way |
| `[` `]` | The command before, and the command after |
| A number first | Does the next key that many times: `5j` |

- A wrapped line is one line.
- The wheel and scrollbar still scroll, and a click puts the cursor there.

**Selecting**

| Key | What it selects |
|---|---|
| `v` | From here, as you move. Press it again to stop |
| `V` | Whole lines |
| `Ctrl+V` | A block |
| `o` | With something selected: jumps to the other end of it |
| `o` | With nothing selected: the whole output of the command you're in. At a prompt, the output of the command before |
| `c` | The command line: what was typed at the prompt you're in or under |
| `p` | The path or web address under the cursor, or the nearest on the line |
| `y`, Enter | Copies and leaves. With nothing selected, copies the line |

- `o`, `c`, `[` and `]` need a shell that marks its prompts (OSC 133). The shells insensical
  sets up do.
- `p` goes by shape and doesn't ask the disk.

**Labels**

Press `Space` and every path, web address, commit and machine address in view gets a letter.
Type the letter to copy that one and return to your prompt.

- The newest get the keys under your fingers: `a`, `s`, `d`, `f`. With more than 26 in view,
  labels are two letters.
- Hold Shift with the letter to select without copying, so you can stretch the selection
  before `y`.
- Escape, or any key that isn't a label, puts the labels away.

**Searching**

- `/` opens the pane's search, starting from the nearest match above the cursor.
- Enter puts the cursor on the match and returns to selecting. A selection you'd started now
  reaches the match.
- Escape closes the search and restores everything. Up and Down step through matches while
  the field is open.
- `n` goes to the match before, `N` to the one after.

**Limits**

- In a full-screen program it works on what's shown, with no history.
- If the program takes over the screen or gives it back while you select, the selection is
  dropped and the cursor starts again at the program's.
- In the window, pressing the leader twice sends the leader's key to the program and leaves.
- When the link to another machine is lost, it ends. The pane keeps its last picture,
  dimmed.
- On another machine, history that hasn't arrived is fetched when the cursor reaches the top
  of what's here, and keys keep working.
- There's no `isc` command for it.

### Pasting

- `Ctrl+Shift+V` pastes the clipboard. `Shift+Insert` or the middle button pastes what you
  last selected.
- **A paste that would do more than it looks like is held, and you're asked first.** That's:
  - more than one line, into a program that hasn't asked for pastes to be marked (bracketed
    paste), which would run each line as it arrives;
  - text containing the sequence that ends a marked paste;
  - more than one line, into a pane on a machine you marked careful, whatever the program
    asked for. See [Other machines in the window](#other-machines-in-the-window).
- Who copied it makes no difference. What a program put on the clipboard (see
  [The clipboard and programs](#the-clipboard-and-programs)) is pasted like your own copy: a
  command an agent copied lands at your prompt, where you read it before pressing Enter.
- **The question shows what you'd paste.** A card at the bottom of the pane gives the line
  count and the reason, and the text in the terminal's font. Control characters show as
  their
  sign (`␛` for Escape, `␉` for Tab), and invisible or direction-changing characters by
  number, as in `<U+202E>`.
- Press Enter or click "Paste" to paste. Press Escape or click "Cancel" to drop it.
- Most shells ask for marked pastes, so they get multi-line pastes straight away.

### Mouse and wheel

- A program that asks for the mouse gets presses, releases, motion and the wheel. Hold Shift
  to select text instead.
- The wheel scrolls history. In a full-screen program that didn't ask for the mouse it sends
  arrow keys; in one that did, it's reported to the program.
- The wheel moves three rows per notch.
- A scrollbar shows while the view is scrolled back, and when the pointer is at the pane's
  right edge. Drag its thumb.

### Finding text

Press `Ctrl+Shift+F` to open a field over the pane's top right corner. It moves to the
bottom
right when the match would be under it.

- It finds text anywhere in the history, ignoring case, shows which match you're on and how
  many there are, and brings each match to the middle, selected, so you can copy it.
- Enter goes to the previous (older) match, Shift+Enter to the next. Escape returns to where
  the program is writing.
- Text that wraps at the pane's edge is found across the wrap, over two rows but not three.
- There are no regular expressions.

### Web addresses and paths

- `Ctrl`+click a web address to open it (`⌘`-click on macOS).
- `Ctrl`+click a file's path to open it or show it in your file manager, by the kind of file
  (see "What a click on a path does" below).
- **A web address is underlined under the pointer**, and a note in the pane's bottom right
  shows where it leads, as in "Ctrl+click opens localhost:5173". A plain click doesn't open
  it, because a click selects.
- **A path is marked only while you hold `Ctrl`**, and the note says what the click will do:
  "Ctrl+click opens main.rs in src/".
- Only `http://` and `https://` addresses are opened. Brackets and punctuation around one
  are
  left out.
- A path is marked only if it exists. It's read from the pane's directory, and `~/` is the
  home
  directory. A line and column, as in `src/main.rs:12:5`, go to the editor you chose with
  `open-files-with`; your desktop's opener opens the file at its beginning. A path on a
  network or automounted file system isn't looked up, so pointing at one can't make your
  machine wait.
- An address or path that continues onto the next row is found whole.
- **Links a program marks** (OSC 8), such as an agent's "open the pull request" or the file
  names of `ls --hyperlink`, work the same way. Only a web address, or a `file://` link to a
  file that exists on this machine, is followed. Other kinds are ignored.
- **The note names the machine first**, then the rest of the address, so you can see where a
  click leads. Names with letters outside ASCII are shown in their network form
  (`xn--mnchen-3ya.example`), and a name and password before the machine aren't shown.
- **A click follows only what the note said.** If the words changed under the pointer, the
  click opens nothing and the note shows the new destination. Click again to follow.
- An address or path with an invisible or direction-changing character is never marked or
  opened.
**What a click on a path does.** Clicking something a program printed can't start
anything, so
a path opens only when opening it just shows it.

- With `open-files-with` at `system`, plain text, source code, pictures and PDFs open in
  your
  desktop's usual program. `csv` and `tsv` don't, because a spreadsheet works out formulas.
- Everything else is shown in your file manager: directories, programs, scripts, web pages,
  installers and files with no ending.
- So is a file someone else could replace, such as one in `/tmp` or in a directory your
  group
  can write to, and a file whose content doesn't look like its name.
- With an editor chosen in `open-files-with`, any file opens in it. A directory is still
  shown
  in the file manager.
- A path in a pane on another machine is that machine's. With `nvim`, `vim`, `hx` or `nano`
  chosen, `Ctrl`+click opens it in that editor in a new pane on that machine; otherwise the
  click copies the path. A web address opens in your browser from any pane.

**Choosing the editor.** `open-files-with` says what a `Ctrl`+clicked file opens in:

| Value | Opens the file |
|---|---|
| `system` (the default) | With whatever your desktop uses for that kind of file, at its beginning, if it's one of the kinds listed above |
| `code`, `cursor`, `zed`, `subl`, `idea` | In that editor, at the line and column written after the path |
| `nvim`, `vim`, `hx`, `nano` | In that editor in a new pane, split to the right of the one you clicked in, at that line |
| a command with `{file}` in it | By running it, with `{file}`, `{line}` and `{column}` filled in: `"myedit --goto {line} {file}"` |

- With no line after the path, the line is 1.
- The settings sheet lists the editors it finds on your `PATH`.
- A command of your own is cut into words at spaces and run directly, not by a shell. If it
  can't start, the file is handled as with `system`.

### The clipboard and programs

A program can put text on the clipboard (OSC 52). That's how an editor on another machine,
reached over `ssh`, copies: `"+y` in Neovim there lands on the clipboard here.

- **Only the program in the pane you're looking at can do this**: the pane with the
  keyboard,
  in a window that's in front. A program in another tab or a minimized pane is ignored, so
  nothing can change what you're about to paste without you knowing.
- **Pasting it is like pasting anything else**, with no extra question. [Pasting](#pasting)
  says when a paste is held.
- Text only, up to a megabyte, and only the clipboard: requests for the primary selection
  (OSC 52 with `p` or `s`) change nothing.
- **No program can read the clipboard.** OSC 52 with `?` gets no answer, and no setting
  allows
  it.
- Bursts are limited: at most three texts at once, then one a second. The rest are dropped.
- `clipboard = "never"` turns it off. Set on this computer, it holds for every machine's
  panes.

### Pictures

Programs can show pictures in a pane: a plot, a thumbnail, a file manager's preview.
insensical draws pictures sent by the **Kitty graphics protocol**, which `kitty +kitten
icat`,
`chafa`, `timg`, `viu`, matplotlib's kitty backend and many others use.

- A picture is part of the screen: it scrolls with the text, goes into the history, and is
  cut
  off at the pane's edge. It's still there when you return to the tab or `isc attach`.
- One too large for its pane is made smaller so you see all of it, in its own shape. It's
  never stretched or cropped. Make the pane larger and it grows, up to the size asked for.
- **Unicode placeholders** work, so programs such as Yazi can show pictures. The browser
  page
  doesn't draw pictures. The interface in a terminal shows them where your terminal can: see
  [The whole interface in a terminal](#the-whole-interface-in-a-terminal).

Limits:

- **Sixel** pictures aren't shown.
- A picture must be sent in the escape sequences themselves, not named by a file or shared
  memory. `icat` falls back to sending it.
- Each pane keeps 64 MB of pictures; the oldest off-screen ones are forgotten first.
- `isc capture` and copying give the text only.

### What the terminal does not do

- **Input methods (IME) are off unless you turn them on.** With `input-method = true`, your
  system's input method (fcitx, ibus, the Mac's) composes text in a pane, shown underlined
  at
  the cursor. Turn it off again if typing misbehaves.
- Sixel images aren't shown. Kitty graphics are: see [Pictures](#pictures).
- The words of a command running longer than a few seconds are read from the system, so
  everyone who may see your panes can see them: don't put a secret on a command line.
- Narrowing a pane rewraps its lines. A full-screen program's screen is cut off at the new
  edge, and the program draws it again.

---

## 10. States and attention

Every pane shows whether its program is working, waiting for you, or finished, so you can
see
at a glance what needs you and jump straight to it. The daemon keeps each state, so it's
right
even when no window is open.

| State | Means | Counts as needing you |
|---|---|---|
| `idle` | At a prompt, or nothing has reported | |
| `running` | A command or agent is working. Shows a percentage if one is reported | |
| `waiting` | Blocked on you: a question, a permission, a password | ✓ |
| `failed` | Ended with an error, and you haven't looked yet | ✓ |
| `done` | Finished, and you haven't looked yet | |
| `exited` | The program ended and the pane was kept (`isc run --keep`) | |

Each state has its own mark in its own colour. `running` is a spinning ring, and it's
the only mark that moves. `waiting` is a ringed dot, `done` a tick, `failed` a cross, and
`exited` a ring with a bar across it.

`done` and `failed` clear when you look at the pane: when it has the keyboard in a window
that's in front.

### Where a state shows

- **On the pane**: its header says the state in words, what the program said about it, and
  how long it has needed you. A pane that waits glows in the waiting colour, so you catch it
  from the corner of your eye.
- **Everywhere else the pane appears**: its tab, its sidebar row, its dock chip. Hover over
  a mark to read the words.
- **As counts**, at the top of the sidebar or in the top bar: how many panes need you, and
  how many finished. The tray, the Mac's menu bar and `isc tui` show the same numbers.
- **On a folded tab or project**: the most urgent state of what's inside, in this order:
  `waiting`, `failed`, `done`, `running`. Unfolded, the rows beneath show their own states.

A waiting pane says what it waits for when its program tells it: "wants permission",
"asks a question" or "needs you to sign in". An agent's permission question names the tool
and what it would act on, such as `Needs permission: Bash · git push origin main`.

When the dock has more chips than room, the `idle` panes share one chip, such as "3 idle".
Click it to open the palette. A pane with a state always keeps its own chip.

Nothing is ever reordered because of a state. Tabs, rows and chips stay where they are.

> [!TIP]
> Turn off the glow with `attention-glow = false`. It's also left out when `pane-gap = 0`,
> since there's no gap for it to fill.

### Going to the pane that needs you

Press `Ctrl+Shift+A`, or click the count in the top bar. You go to the most urgent pane,
restored if it was minimized. Press it again to step through the rest: first what needs
you, then what finished.

You can also:

- Open the palette: the panes that need you are listed first.
- Open the overview: every tab of every project, with what each waiting pane asks and for
  how long.
- Show only activity in the sidebar: press the pulse-line button at its head. See
  [The sidebar](#the-sidebar).

### Where a state comes from

insensical works out a state from four things. The strongest wins.

1. **A report.** The program says it in its own terminal (see
   [Programs that report by themselves](#programs-that-report-by-themselves)), or an
   agent's hooks or a script run
   `isc status running|waiting|done|failed|idle [-m "what about"]`. A `running` or
   `waiting` report stands until the next report or until the command ends. In a pane you
   started with a command, such as `isc run -- claude`, it stands until the next report
   or until the program ends.
2. **What the program writes to its terminal.**
   - **Progress** (OSC 9;4): a percentage or an unknown amount is `running`; removed is
     `done` if the pane was running; error is `failed`; paused is `waiting` until the
     command ends or you look.
   - **A notification** (OSC 9, 777 or 99) is `waiting` with its text, until you look. It
     doesn't change a `done` or `failed` that was reported.
   - **The bell**, while nobody is looking, is `waiting` until you look. It marks the pane
     and never sends a desktop notification, because shells ring it on every failed
     completion.
   - **A password prompt** is `waiting`, with the line it asks on, such as
     `[sudo] password for you:`. insensical knows because the terminal stops showing
     what you type while still taking a whole line, as `sudo`, `ssh`, `git` and `read -s`
     do. It stops when typing shows again or the command ends.
3. **The foreground command ending**: `done` if the pane was `running`, or if the command
   ran for 30 seconds or more (or as long as `long-command` says). This needs no shell
   integration.
4. **The program itself ending** in a kept pane: `failed` if it exited with a non-zero
   status.

States aren't only for agents. Any command that runs for 30 seconds gives its pane one.

| Command | Does |
|---|---|
| `isc status explain [--pane N]` | Tells you why a pane is in its current state. |
| `isc status clear [--pane N]` | Forgets what was reported and puts the pane at rest. Use it when a pane says `running` or `waiting` for something that's gone. |

The palette has the same as "Forget what this pane's program said it was doing"
(`forget`). Looking at a pane never clears `running` or `waiting`, because looking doesn't
answer a question.

### Programs that report by themselves

A program can tell its terminal what it's doing, and insensical listens. It needs no hook
and no setup, and it works over `ssh` and inside a container, where `isc status` can't
reach. This is the Program Status Protocol, an escape sequence numbered OSC 7501, read at
revision 0.4.

Claude Code speaks it from version 2.1.295. So can any script:

```sh
status() {
  printf '\e]7501;state=%s:msg=%s\e\\' "$1" "$(printf '%s' "$2" | base64 | tr -d '\n')"
}
status working "Syncing photos"
rsync -a ~/Photos backup:/photos && status done "Photos synced" || status error "rsync failed"
```

| The program says | The pane shows |
|---|---|
| `idle` | `idle` |
| `working`, with a percentage if it gives one | `running` |
| `blocked` | `waiting` |
| `done` | `done`, until you've seen it |
| `error` | `failed`, until you've seen it |

A program can also:

- **Say what it's blocked on**: `permission`, `question` or `auth`. The pane shows "wants
  permission", "asks a question" or "needs you to sign in", in `isc ls` too.
- **Give its name**, as Claude Code gives `claude-code`. If insensical has an icon for it,
  or you gave one under `[icons]`, the pane uses that icon.
- **Report on tasks of its own**, such as an agent's helpers. The pane shows the most
  urgent one, and the overview adds a count like "3 tasks · 1 waiting".
- **Take back what it said** (`state=clear`).

It outranks an agent's own hooks while it says working or waiting, and the hooks count
again after it says anything else, so you don't need to remove them. If it has been silent
while working, a hook that says waiting, done or failed is believed. Working and waiting
end when the command ends at your shell's prompt. `done` and `failed` stay until you've
seen them.

**Good to know**

- A program inside `tmux` or `screen` isn't heard.
- A malformed report, an unknown state or an unknown key is ignored.
- A pane can keep saying `running` or `waiting` after its program was killed, if it ran
  through `ssh`, `docker exec` or `su`, under a shell inside a shell, or in a pane started
  with a command that isn't a shell. Run `isc status clear`, choose Forget in the palette,
  or type `printf '\e]7501;state=clear\e\\'` in the pane. insensical never ends a state
  for being silent: a build says nothing for an hour.
- `isc shell-integration` (below) makes the shell end working and waiting at each prompt.
- A shell function that reports and then waits with `wait` or `read` looks like a shell
  at its prompt. Put the work in a script.

### Commands that fail, and short commands

- **A long command is one that ran for 30 seconds.** `long-command` changes that, from 1
  second to 3600.
- **With shell integration, a failed command is `failed`**, with a note such as "exited
  with 2 after 1m 35s". Without it, a failed command shows as `done`.
- **A short command isn't marked**, even if it fails.
- **An exit status of 130** (Ctrl+C) counts as finished, not failed.

To turn shell integration on:

| Shell | Add |
|---|---|
| bash 4.4 or later | `eval "$(isc shell-integration bash)"` in `~/.bashrc` |
| zsh | `eval "$(isc shell-integration zsh)"` in `~/.zshrc` |
| fish older than 4 | `isc shell-integration fish \| source` in `~/.config/fish/config.fish` |
| fish 4 and later | Nothing: it does this itself |

Each does nothing outside a pane. An older bash, such as the Mac's `/bin/bash` (3.2), is
left alone. Other terminals' integrations that write the same marks (OSC 133 C and D) work
too. In a pane you started with a command, a shell's marks don't take down what was
reported.

---

## 11. Agents

insensical learns a coding agent's state in two ways, and lets the agent use insensical too.

- **With no setup**, from what agents already write to their terminal: their own reports
  (Claude Code makes them from version 2.1.295), progress, notifications and the bell, as
  in section 10.
- **Exactly**, from the agent's own hooks, which call `isc status`. insensical can add
  hooks for Claude Code, Codex, Gemini CLI, opencode, pi, Cursor CLI, Copilot CLI and
  Hermes.

The same setup lets an agent open, read and type into panes with `isc`. Ask it in your own
words:

| You ask | The agent |
|---|---|
| "Open nvim with top in a narrow pane on the right." | Opens a tab with the editor, and `top` in a narrow pane beside it. |
| "Start the dev server in another tab and tell me when it's ready." | Starts it without taking your focus, and waits until it says it's listening. |
| "Run the tests beside you and fix what fails." | Runs them in a split, waits for them to end, and reads the failures. |
| "Run `\dt` in the psql pane." | Types it into the pane named psql, presses Enter, and reads the answer. |
| "Show me that project, with bigger text." | Brings the project to the front and raises `font-size`. |
| "Close everything you opened." | Closes the panes it opened, and none of yours. |

Three things go in, per agent:

- **Hooks** that report its state.
- **A skill**: a short file the agent reads when a request is about terminals, panes or
  tabs. It tells the agent to run `isc guide`, so a new version of insensical teaches
  every agent at once.
- **A note at the start of each session**, inside a pane only: where the session runs, and
  that `isc guide` says how to use insensical. An agent you started yourself is asked to
  name an unnamed tab after your task.

Where an agent acts follows your words. "Here", "beside you" or no place means its own
pane and tab. "The focused pane" means the one with the keyboard. A name always wins. An
agent types only into panes it opened or that you named, and asks you first otherwise.
Another agent's question, seen in its pane, it tells you about, and answers only when you
ask.

`isc guide` prints the guide: where the agent is, where to act, and the topics.
`isc guide layout`, `reading`, `acting`, `waiting` and `agents` print one topic each. For
an agent insensical doesn't set up, tell it to run `isc guide`.

**Keeping agents out of a pane.** For a pane with a production database, a secret or
anything else no agent should see, choose Keep agents out of this pane in the palette (the
`protect` command), or run `isc protect PANE`. To let them in, choose Let agents into this
pane, or run `isc protect PANE --off`. It's kept with the layout, and the pane shows a
crossed-out robot.

- A program in another pane can't read it (`isc capture`, `isc history`, `isc attach`,
  `isc status explain`), wait on it (`isc wait`) or type into it (`isc send`, `isc key`,
  `isc ask`). `isc` exits with status 4, and the guide tells agents to tell you.
- `isc ls` in another pane still lists it, ending in `keeps agents out`, with its title,
  tasks and running command left out.
- You aren't refused anything: the window, `isc tui` and `isc` in a terminal outside
  insensical work as ever. `isc` run in another pane is refused, whoever typed it. A
  terminal you open from a pane is in that pane too.
- Only you let agents in again: `isc protect --off` run in a pane is refused.
- It holds back an agent that follows what it's told. It isn't a wall: a program that
  runs as you, or that leaves the pane altogether, can't be told from you.

> [!WARNING]
> Letting an agent run `isc` lets it do anything you can. `isc run`, `isc split` and
> `isc agent start` open panes outside any sandbox the agent has, with your whole home
> and the network. Allow `isc` only in an agent you'd let run commands as you, and keep a
> pane that matters out of its reach.

**Agents that start other agents.** An agent can hand work to another: Claude Code to
Codex, Codex to Claude Code, either to Gemini CLI. Ask "have Codex review this diff" and it
runs `isc agent start codex "Review the diff"`. Codex opens in a new tab with the task as
its first prompt, so you can watch it and take over. The starter waits with
`isc agent wait`, which prints `done` and the whole reply, or `waiting` and the question.
Both need their hooks set up.

| Command | Does |
|---|---|
| `isc agent start AGENT TASK` | Opens the agent in a new tab, leaving your keyboard alone; `--front` brings it forward. `--name NAME` refuses while a pane of that name is there. |
| `isc agent wait` | Waits for a turn to end. `--timeout` sets how long; it exits with status 124 when it runs out. |
| `isc agent tell PANE "MESSAGE"` | Pastes a message as the agent's next prompt. While the agent works, it goes in when the turn ends. |
| `isc agent reply PANE` | Prints the last whole reply again. |
| `isc agent answer PANE yes`, `no` or words | Answers a permission prompt. Words refuse it and go in as the next prompt. |
| `--quiet` on `start` | Runs the agent's one-question mode (`claude -p`, `codex exec`, `gemini -p`) with no pane and prints only its answer. |
| `isc ls --opened` | Lists the agents started from this pane. |
| `isc close --opened` | Ends the whole team, but for a pane you've typed into or focused. |

- `isc agent start` won't start an agent whose hooks aren't set up. It tells the asker to
  ask you to set it up. Agents never run `isc integrate` themselves.
- If a started agent's hooks say nothing for a while, the pane turns to waiting for you
  with the likely reason: Codex runs no hook you haven't trusted with `/hooks`, and Claude
  Code runs none in a folder you haven't trusted.
- `isc agent` knows Claude Code, Codex and Gemini CLI. `answer` presses Enter or Esc for
  Claude Code, `y` or Esc for Codex, and `1` or Esc for Gemini CLI. An agent whose prompt
  takes other keys has to be answered in its pane.
- A program can't answer, or type into, the agent in its own pane, nor in a pane it
  opened: its agent's questions are yours.
- A started agent can ask its starter something by ending its turn with the question. The
  starter's `isc agent wait` returns at once, and it answers with `isc agent tell`.
- Only an agent's own hooks say which agent it is and what it replied.

So a mistake can't run away, one agent can have 8 agents open at once, and agents started
by agents go 2 levels deep. Change them under Agents in the settings sheet, or with
`agents-at-once` and `agents-deep` in `config.toml`.

### Setting up from the window

When an agent on the system isn't set up, the foot of the sidebar says so: "Codex can use
insensical · Set up".

- Press it to list the agents, with each file that changes and what goes in it.
- Press Enter to add them. **Nothing is ever added without that Enter.**
- Press Escape to leave it for later.
- Press `N`, or click the cross on the notice, to say never.
- After adding, a line tells you what's still yours to do: for Codex, trusting the hooks
  with `/hooks` (the dialog has an Open Codex to trust them button); for Hermes, enabling
  the plugin.
- "Set up the agents on this system to use insensical" in the palette asks at any time,
  and includes the agents you declined.

`isc tui` offers them too: `Ctrl+Space` then `Shift+A` asks, `Enter` sets up, `Esc` leaves
the notice and `N` says never. Declined agents are kept in
`~/.local/state/insensical/agents-declined`, one per line. Delete a line to be offered
that agent again.

### Setting up from the command line

| Command | Does |
|---|---|
| `isc integrate` | Finds the agents on this system, shows what would change for each, and asks. |
| `isc integrate claude` | The same for one agent: `claude`, `codex`, `gemini`, `opencode`, `pi`, `cursor`, `copilot` or `hermes`. |
| `isc integrate --yes` | Adds every agent found, without asking. |
| `isc integrate --remove` | Removes them all, or the one you name. |
| `isc integrate --force` | Replaces a skill called `insensical` that insensical didn't write. With `--remove`, removes it. |
| `isc integrate --status` | Prints what was found and how each agent's hooks, skill and note stand, as JSON. Changes nothing. |

- An agent counts as found when its program is on `PATH` or its settings directory exists.
- With no terminal to ask on, `isc integrate` changes nothing unless you pass `--yes`, and
  exits with status 1.
- **With `--host`, it's the other machine's agents**, and you must give `--yes` or
  `--remove --yes`: nobody can be asked there. Its daemon must be running.
- `--status` can't be combined with `--remove`, `--yes` or `--force`. For each agent it
  prints `agent`, `title`, `found`, `integrated`, `stale`, `file`, `items` and `then`.
  `items` has `hooks`, `skill` and `note`, each with a `file` and a `state`: `current`,
  `outdated`, `missing`, `foreign` or `unsupported`.
- A hook of your own that isn't `isc hook AGENT` is never changed or removed. An agent's
  settings file over 16 MB, or one that can't be read as text, is left as it is.
- `CLAUDE_CONFIG_DIR` and the other agents' variables count only when they hold a whole
  path, from `/`.

> [!WARNING]
> The hooks name `isc` by the full path it had when you ran `integrate`. If you move or
> reinstall `isc`, the agent's state stops being reported and the window offers to set it
> up again. Run `isc integrate` for that agent to rewrite them.

What each agent gets:

| Agent | Hooks or plugin | Skill | Note at a session's start | Resume offered |
|---|---|---|---|---|
| Claude Code | `~/.claude/settings.json`, or under `$CLAUDE_CONFIG_DIR` | `~/.claude/skills/insensical/SKILL.md` | A hook | `claude --resume <session>` |
| Codex | `~/.codex/hooks.json`, or under `$CODEX_HOME` | `~/.agents/skills/insensical/SKILL.md` | A hook | `codex resume <session>` |
| Gemini CLI | `~/.gemini/settings.json`, or `$GEMINI_CLI_HOME/.gemini/settings.json` | `~/.agents/skills/insensical/SKILL.md` | A hook | None |
| opencode | `~/.config/opencode/plugins/insensical.js` (under `$XDG_CONFIG_HOME`) | `~/.agents/skills/insensical/SKILL.md` | None | `opencode --session <id>` |
| pi | `~/.pi/agent/extensions/insensical.ts`, or under `$PI_CODING_AGENT_DIR` | `~/.agents/skills/insensical/SKILL.md` | None | `pi --session` |
| Cursor CLI | `~/.cursor/hooks.json` | None | A hook in `hooks.json` | None |
| Copilot CLI | `~/.copilot/hooks/insensical.json`, or under `$COPILOT_HOME` | `~/.agents/skills/insensical/SKILL.md` | A hook | `copilot --resume <session>` |
| Hermes | Plugin in `~/.hermes/plugins/insensical`, or under `$HERMES_HOME` | `~/.hermes/skills/insensical/SKILL.md`, or under `$HERMES_HOME` | The plugin, at the first turn | `hermes --resume <session>` |

- Hook files you already have are kept, with a copy saved once beside them as
  `*.before-insensical`. `--remove` takes only insensical's hooks out. Files insensical
  writes whole (opencode, pi, Copilot CLI, Hermes) are deleted.
- A skill called `insensical` that insensical didn't write is yours: `--force` replaces
  it. Five agents share the one skill in `~/.agents/skills`, and removing one leaves it
  while another is set up.
- Hooks print nothing, never fail and do nothing outside a pane.

What each agent reports:

| Agent | `running` | `waiting` | `done` | `failed` | `idle` |
|---|---|---|---|---|---|
| Claude Code | A prompt; a tool has run | A permission or a question | It finished responding, with its first line | It stopped on an error | A session starts or ends |
| Codex | A prompt; a finished tool | A request for approval | End of a turn, with its first line | Not reported | An interrupted turn |
| Gemini CLI | A prompt; a finished tool | Asking to use a tool | End of a turn, with its first line | Not reported | None |
| opencode | Working | A permission or a question | Going quiet after working | An error, with its message | A turn you stopped |
| pi | Working | A dialog an extension puts up | Finishing, with what it said | Ending in an error | Stopping it yourself |
| Cursor CLI | A prompt; a finished tool | Never | End of a turn | A turn that errors | A turn you stopped |
| Copilot CLI | A prompt; a tool | A request for permission | End of a turn | An error it can't get past | None |
| Hermes | Working | A command or a question | Finishing, with its first line | Same as `idle` | A stopped turn, or none ended without an answer |

### Claude Code

Claude Code 2.1.295 and later also reports in its own terminal (section 10), and that's what
the pane shows; the hooks' state reports are set aside. Keep the hooks: they name the
session to resume (section 17).

When Claude runs several tools at once and one finishes while another still waits for your
answer, the row keeps saying `waiting` until that tool has run, a new tool starts, the
turn ends or you send a prompt. Codex, Gemini CLI and Copilot CLI work the same way.

### Codex

**Codex does not run a hook until you trust it.** After integrating, open Codex, run
`/hooks`, and trust the new ones. Until you do, nothing is reported. If `isc` moves and
you add the hooks again, trust them again.

### Gemini CLI

A failed or cancelled turn isn't reported, so the pane stays `running` until the next
prompt.

### opencode

Sessions that opencode starts itself (subagents) aren't reported.

### pi

pi reports only when it's running with a screen. It never asks before using a tool, so
`waiting` comes only from a dialog an extension shows.

### Cursor CLI

**It never says `waiting`.** Cursor has no event for a question it asks you, so the pane
stays `running` until you answer. What it said at the end of a turn isn't shown.

### Copilot CLI

What it said at the end of a turn isn't shown. Your own hook files in `~/.copilot/hooks`
aren't touched.

### Hermes

**Hermes doesn't load a plugin until you enable it.** After integrating, run
`hermes plugins enable insensical`. Until you do, nothing is reported. Agents that Hermes
starts itself (subagents) aren't reported.

### Any other agent or script

Any agent or script can call `isc status` and `isc notify` from its own hooks.

```sh
isc status running
isc status waiting -m "Allow \`cargo publish\`?"
isc status done
isc status running --resume "mytool --continue 42"
isc notify "Build" "Finished"
```

- `--resume` says how to pick the work up again. The pane offers that command at its
  prompt after the daemon has stopped.
- Inside a pane they need nothing more. Outside one they need `--pane`.
- A message that begins with a hyphen works: `isc status done -m "- fixed"`. In a script,
  `--message="$text"` keeps the text and the option together.

### Agents that open their helpers in tmux panes

Some agents start helpers, each in a pane of its own, by running `tmux` commands, but only
inside tmux. With the `tmux-shim` setting on, those helpers open as panes here, beside the
agent, and you can move, name and zoom them like any pane.

```sh
isc config set tmux-shim true
```

or switch on "Answer as tmux" in the settings sheet, under Terminal. It's off until you
turn it on, and applies to panes started afterwards.

> [!NOTE]
> While it's on, `tmux` in a pane is `isc`, not the real tmux. If you use tmux yourself
> inside insensical, run it by its full path, such as `/usr/bin/tmux`, or leave the
> setting off. Nothing outside insensical's panes changes.

A session is a project, a window is a tab, and a pane is a pane. Targets are written as in
tmux: `%5`, `@7`, `$3`, `api:0.1`, `api:logs`, `:1`.

- **Understood**: `split-window`, `new-window`, `new-session`, `respawn-pane -k`,
  `send-keys`, `kill-pane`, `kill-window`, `select-pane`, `select-window`, `rename-window`,
  `list-panes`, `list-windows`, `list-sessions`, `display-message -p`, `capture-pane -p`,
  `resize-pane`, `select-layout`, `has-session` and `-V`. Common flags work (`-h`, `-b`,
  `-l`, `-p`, `-c`, `-e`, `-d`, `-P -F`, `-n`, `-s`, `-S`, `-E`, `-Z`). Commands can be
  chained with `;`.
- **Formats**: `pane_id`, `pane_index`, `pane_title`, `pane_current_command`,
  `pane_current_path`, `pane_start_command`, `pane_active`, `pane_width`, `pane_height`,
  `pane_dead`, `pane_dead_status`, `window_id`, `window_index`, `window_name`,
  `window_active`, `window_panes`, `window_zoomed_flag`, `session_name`, `session_id`,
  `session_windows`, `session_attached`, `client_control_mode`, `version`, `#S`, `#I`,
  `#W`, `#P`, `#D`, `#T` and `#{?name,this,that}`.
- **Accepted and ignored**: `set-option`, `set`, `set-window-option`, `bind-key`,
  `unbind-key`, `source-file`, `set-hook`, `set-environment`, `start-server` and
  `refresh-client`.
- **Refused**: `kill-server`, which would stop every pane you have. Only
  `isc kill-server` does that.
- Anything else is `unknown command`, with exit status 1.

Limits:

- A pane closes when its program ends: `remain-on-exit` isn't kept.
- `respawn-pane` needs `-k` and a command. `capture-pane` needs `-p`. A session can't be
  attached to.
- `main-vertical` and `main-horizontal` give the main pane half the tab.
- Opening a pane in a tab you aren't looking at shows you that tab briefly; `-d` returns
  the keyboard.
- If your shell's start-up file replaces `PATH` instead of adding to it, a program you
  start won't find `tmux`.

---

## 12. Notifications

insensical tells you when a pane needs you, so you can walk away from your agents and
come back when it matters. You're told when a pane becomes `waiting`, `done` or `failed`,
when a program sends a notification sequence, or when something runs `isc notify TITLE
[BODY]`.

With nothing attached, the daemon can run a command or post to a web address instead: see
[When nobody is attached](#when-nobody-is-attached).

**What you'll notice.** The daemon gathers what your panes say before you're told, so every
way of being told agrees.

- Panes that wait, fail or finish together are one notification, such as `10 panes failed`.
  Click it to go to the first pane.
- A finish is held for about 15 seconds. If you look at the pane or type in it first, you
  aren't told. An agent that pauses and wakes by itself isn't told of until it stops for
  good.
- A pane that says the same thing again isn't told of again until it says something
  different or you've looked.
- A pane that waits for you is never held back.

**Where you're told** depends on where you are. With insensical in front, you get a
notice inside the window and nothing on the desktop. Otherwise, or with no window open,
you get a desktop notification.

**The notice in the window** sits at the foot of the sidebar, or over the window's bottom
right corner when the sidebar is hidden or the tabs are along the top. It never resizes a
terminal.

- It shows the state mark, `pane · project` and what the pane says. Click it to go to the
  pane; `×` closes it.
- A finish goes after six seconds; a wait or failure stays until you've seen the pane.
- Three show at once; a line above them counts the rest and opens the overview.
- When the daemon refuses something you asked for, a notice headed "It could not be done"
  says why.

**Sound.** A pane that waits or failed makes a short sound of two rising notes. A finish
makes a softer single note, if you ask for it.

| `sound` | Plays |
|---|---|
| `needs-you` (default) | Waiting and failed |
| `all` | Waiting, failed and finished |
| `off` | Nothing |

The settings sheet has a button to hear each. Linux needs `pw-play`, `paplay` or `aplay`,
macOS uses `afplay`. You can't use sounds of your own.

**Turning them off.** `notifications = false` stops all of them. `notify-waiting`,
`notify-failed` and `notify-done` turn off one kind each. Panes still show their state.

**You aren't told** when you're looking at the pane, when it's muted, or when the cause is
the bell.

**What a desktop notification holds.** Each pane has one, and a new one replaces the old.
The title is `project · pane`; the body is what the program said, or "Waiting for you",
"Done" or "Failed". A title is cut to 120 characters and the text to 600. They're sent at
normal urgency, so do-not-disturb applies.

**Muting a pane.** Use the `mute` command in the palette (no default key), or
`isc mute [PANE] [--off]`. The pane's states still show, and a mark appears in its header.

**Muting a project.** Use "Silence this project's notifications" in the palette (the
`mute-project` command, no default key), or `isc project mute NAME [--off]`. No pane in the
project sends a notification, including panes you open later. Panes muted on their own
stay muted.

**On Linux** the daemon sends them over D-Bus, so they arrive with no window open.

**Clicking one** goes to the pane, restores it if it was minimized, and asks a window to
come forward. With no window open, one opens on the pane. On Wayland the window may stay
behind: a compositor lets a window raise itself only with a token the toolkit can't use.
The pane is still selected. Hyprland brings it forward with `misc:focus_on_activate` on.

**On Linux, a tray icon says what needs you**, with no window open too. It has an amber
dot while any pane waits or failed. Its menu lists those panes, and picking one goes to
it. **Quit insensical** in the menu asks first and says which panes would end. The tray
needs a desktop that shows status notifier items (GNOME needs the AppIndicator extension).
`tray = false` takes the icon away; **Show in the tray** under **Notifications** in the
settings is the same switch.

**On macOS** the application posts them, even with the window closed. The system asks once
whether to allow them.

**On macOS, the menu bar and the Dock say what needs you**, even with the window closed.

- The menu bar item counts panes that wait or failed, and its menu lists them.
- The Dock icon shows the same number as a badge. If it doesn't appear, turn on **Badge
  application icon** for insensical in System Settings, Notifications.
- At the top of **Notifications** in the settings, a line says whether macOS lets
  insensical notify you and badge its Dock icon. **Open System Settings** takes you there.
- `menu-bar = false` removes the menu bar item and `dock-badge = false` the Dock number,
  both under **Menu bar and Dock** in the settings, a section only a Mac has.

### When nobody is attached

An agent on a server stops to ask you something at two in the morning, and no window is
open. The daemon can still tell you, by running a command of yours or posting to a web
address of yours. Both go outward only: nothing listens.

**The two settings** belong to the daemon, so set them on the machine the agent runs on.
From another machine, put `--host NAME` before each command.

```sh
isc config set notify-command /home/you/bin/tell-me
isc config set notify-url https://ntfy.sh/a-topic-only-you-know
```

Set one or both, then try them:

```sh
isc notify --test
```

That sends one through each, whoever is attached, and prints how it went:

```
notify-command: ran, exit status: 0, in 0.3 s
notify-url: posted, HTTP 200, in 0.4 s
```

It exits with an error if either failed. `isc notify --sinks` shows what's set, how the
last one to each went, and how many weren't sent.

**What gets sent** is what would have been a notification, with the same mutes, switches
and gathering. For several panes the title is the count, and the pane, project and program
are the first pane's.

**When it's sent.** By default only when you're away: no window, terminal interface or
browser is attached to that daemon. `isc events`, a script, a program in a pane and
anything attached with only the `look` right don't count as someone being there. To be
told every time:

```sh
isc config set notify-command-when always
isc config set notify-url-when always
```

**The command** is a program and its arguments, cut at the spaces, with no shell: `|`,
`$VAR` and quotes mean nothing. Put anything longer in a script and name the script. What
happened is in the command's environment:

| Variable | What it holds |
|---|---|
| `INSENSICAL_EVENT` | `waiting`, `failed` or `done` |
| `INSENSICAL_TITLE` | `project · pane`, as on a notification |
| `INSENSICAL_BODY` | What the pane says |
| `INSENSICAL_KIND` | What a wait is for, if the program said: `permission`, `question` or `auth`. Otherwise empty |
| `INSENSICAL_PANE` | The pane's number |
| `INSENSICAL_PROJECT` | The project's name |
| `INSENSICAL_MACHINE` | The name of the machine the daemon is on |
| `INSENSICAL_PROGRAM` | The program in the pane |

A title or message is whatever a program wrote, so keep it in double quotes in your
script: `"$INSENSICAL_BODY"`.

The rest of the environment is the daemon's, so an `isc` your script runs reaches the same
daemon. `INSENSICAL_PANE` is the pane the notification is about, so an `isc` command that
acts on "this pane" acts on that one.

**The web address** is posted this, as JSON:

```json
{"title": "api · claude", "message": "Wants permission: Allow edit to main.rs?",
 "event": "waiting", "kind": "permission", "pane": 41, "project": "api",
 "machine": "build", "program": "claude"}
```

With `notify-url-body = "text"`, the message alone is the body and the title goes in a
`Title` header. Headers of your own go in a table of the settings file, or one at a
time from the command line:

```sh
isc config set notify-url-headers "Authorization: Bearer tk_yourtoken"
```

`isc config get notify-url-headers` shows each value masked, and `isc config reset
notify-url-headers` takes them all out.

For a topic that needs you to sign in, give a token, or a user and password:

```sh
isc config set notify-url-token tk_yourtoken
isc config set notify-url-user me
isc config set notify-url-password "your password"
```

Other users of the machine can read a command line while it runs, so give a secret on
standard input: `printf %s "$TOKEN" | isc config set --stdin notify-url-token` reads the
value to the end of its input, less one final newline.

A token is sent as `Authorization: Bearer …`, a user and password as Basic. With both set,
the token is used, and either stands in place of an `Authorization` header you wrote.
They're secrets: `isc config get`, `isc notify --sinks` and the settings sheets show only
that one is set (a token by its last four characters).

They're never sent over plain `http://` to another machine, nor is an `Authorization`
header you wrote: use an `https://` address. An address on this machine (`127.0.0.1`,
`localhost`, `[::1]`) may be `http://`. Keep the settings file to yourself (`chmod 600`).

**Who sees the address and the command.** Only a connection with the `manage` right sees
both in full, which is every connection of your own unless you asked for fewer. Without it,
such as a browser paired as a device that only looks, `notify-url` shows as its scheme and
host (`https://ntfy.sh`) and `notify-command` as the program's name (`tell-me`).

**ntfy.** Pick a topic nobody would guess; it's the only secret. In the settings file on
the machine the agent runs on:

```toml
notify-url = "https://ntfy.sh/a-topic-only-you-know"
notify-url-body = "text"

[notify-url-headers]
Priority = "high"
Tags = "robot"
```

Subscribe to the topic in the ntfy app, then run `isc notify --test`.

**Pushover, Telegram, Slack and anything else** want a body of their own shape, so let
`curl` shape it in the command. For Pushover, save this as `/home/you/bin/tell-me` and
`chmod +x` it:

```sh
#!/bin/sh
exec curl -fsS --max-time 8 \
  --form-string "token=YOUR_APP_TOKEN" \
  --form-string "user=YOUR_USER_KEY" \
  --form-string "title=$INSENSICAL_TITLE" \
  --form-string "message=$INSENSICAL_BODY" \
  https://api.pushover.net/1/messages.json
```

Then `isc config set notify-command /home/you/bin/tell-me` and `isc notify --test`.

**A desktop you're logged in to, with a daemon that can't reach it.** A daemon started over
`ssh` doesn't know the desktop's session; tell `notify-send` where it is:

```sh
#!/bin/sh
export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$(id -u)/bus"
exec notify-send --app-name=insensical "$INSENSICAL_TITLE" "$INSENSICAL_BODY"
```

**Good to know.**

- A window left open while you're at lunch counts as you being there. Use `always` to have
  your phone told too.
- The command and the address each get ten seconds. The command runs in your home
  directory, so name it by its full path: `~` isn't expanded, and a path with a space can't
  be named.
- Four are sent at once at most. One more is dropped and counted, and `isc notify --sinks`
  shows the count.
- `https` trusts the authorities built into `isc`, not your system's. For a certificate of
  your own making, use the command with `curl`.
- `https` needs an x86-64 processor from about 2014 or later, or an ARM one with the AES
  and SHA-2 extensions (a Raspberry Pi 4 lacks them). Use the command with `curl`, or plain
  `http`.
- To post through a proxy, start the daemon with `ALL_PROXY`, `HTTPS_PROXY` or
  `HTTP_PROXY` in its environment, and `NO_PROXY` for hosts to reach directly. See
  [The daemon's environment](#the-daemons-environment).

---

## 13. Keys

Every command has a key, and you can change any of them. There are two keymaps.
`keymap = "direct"` is the default and gives every command its own chord.
`keymap = "leader"` puts every command on a key you press after a leader key. Section 19
has the full table for both.

**The window spells a key out in full**, joined by `+`: `Ctrl+Shift+T`, `Ctrl+Alt+Left`. A
key is named by what you press, so `?` is shown as `Shift+/`. A capital letter never stands
for Shift: after the leader, `T` is the `t` key alone and `Shift+H` is the capital.

**On macOS, keys are shown in the signs your menus use**, in this order: Control `⌃`,
Option `⌥`, Shift `⇧`, Command `⌘`, then the key, as in `⇧⌘N` and `⌃⌥←`. Keys pressed one
after another are a space apart: `⌃Space T`.

On Linux, the direct keys are `Ctrl+Shift` with a letter. On macOS, they're `⌘`-based.

**On macOS, closing the window doesn't quit the application, and neither does `⌘Q`.** They
leave insensical in the menu bar, still telling you when a pane needs you. Open it again
from the menu bar item, the Dock or Spotlight. To quit, choose
**Quit insensical** or press `⌥⌘Q`. If a pane is running or waiting, it asks first: `Esc`
goes back, `W` closes the window and leaves the application running, `Enter` quits. With
`menu-bar` off there's no item to go to, and `⌘Q` quits.

`⌘,` opens Settings. `⌘H`, `⌥⌘H` and `⌥⌘Q` are the application's and don't reach a program
in a pane.

On Linux, closing the window quits. The `quit` command has no key there; find it in the
palette, or add a line to `[keys]`, such as `"ctrl-alt-q" = "quit"`.

**On macOS, Option isn't Alt unless you say so.** Option with a key types the character
the layout gives it (Option+B is `∫`), so a shell's Alt+B doesn't answer. Set
`option-as-alt = true` and Option is Alt to the program in the pane, and you can't type the
layout's Option characters there. If the system's own `⌃Space` ("select the previous input
source") is on, it takes the key before the leader can.

**A chord in the direct keymap doesn't reach the program in the pane.** `` Ctrl+` ``,
`Ctrl+Shift+Space`, `Ctrl+Shift+I` and `Alt+1` to `Alt+9` are among them. To hand a chord
back, set it to `none` in `[keys]`, use the leader keymap, or lock the keys while the
program needs them.

### Seeing every key

To see what's bound to what, press `Ctrl+Shift+/` or `Ctrl+F1`, or leader `?`, or choose
"Show every key" in the palette.

A sheet opens over the window with every command under its heading and its keys on caps,
for the keymap in use with your own changes.

- A command with no key is listed too, greyed. If your own `[keys.direct]` or
  `[keys.leader]`
  table took its key, the sheet says `not in your [keys.direct]`.
- Typing narrows the sheet by a command's words, heading or keys: `split`, `ctrl alt`.
- **If you press a chord while the sheet is open, it describes the chord instead of
  running it.** The foot shows what you pressed and the command it runs, or says it has
  none. This is also how to find out how a key is written.
- Escape, Enter, "Close" or a click outside closes it.

### Locking the keys

When a program needs every key, such as an editor, another multiplexer or a remote desktop,
press `Ctrl+Alt+Shift+L`, in either keymap, or choose "Give every key to the program, or
take them back" in the palette.

**While the keys are locked, every key goes to the program in the pane**, including the
window's own chords and the leader.

- The window keeps only the key that locks, and that key unlocks. Copy, paste and find go
  to the program too.
- The pane with the keyboard shows a "Locked" mark in its header, or over its corner when
  headers are hidden. Click it to unlock.
- The pointer works as usual. The lock belongs to the window and is off the next time it
  opens.
- A lock key you set in `[keys]` must be a single chord; one that starts with the leader
  can't unlock, because the leader goes to the program while locked.

### The leader keymap

Use the leader keymap to keep the `Ctrl+Shift` chords for your programs.

```toml
keymap = "leader"
leader = "ctrl-space"   # the default
```

- The direct keymap's chords then reach the programs in the panes.
- Copy, paste, paste-selection and find keep their chords in both keymaps.
- While the leader waits, a panel at the bottom of the window lists every key that can
  follow.
- If nothing follows the leader for about a second, the leader goes to the program. Press
  the leader twice to send its own key; Escape cancels it.
- Setting `leader` alone changes nothing. `keymap = "leader"` turns the leader keymap on.

If the desktop takes `Ctrl+Space` (input method switchers do), the leader never arrives, so
choose another with `leader = "…"`.

**Repeat.** After you move a pane (`Ctrl+H` `Ctrl+J` `Ctrl+K` `Ctrl+L`), move its edge
(`H` `J` `K` `L`) or move a tab (`<` `>`), the same keys work without the leader for one
second: leader, `L`, `L`, `L`.

### Changing keys

Add a line under `[keys]` to rebind a command:

```toml
[keys]
"ctrl-shift-y" = "split-right"   # another key for a command
"leader y" = "split-right"       # after the leader, in either keymap
"alt-enter" = "zoom"
"ctrl-shift-m" = "none"          # leave this key to the program
```

- Write what you hold first: `ctrl`, `shift`, `alt`, `cmd` or `super`, joined by hyphens,
  as in `ctrl-shift-t` or `ctrl-alt-left`.
- On Linux `ctrl-shift-]` and `ctrl-}` are the same key, and so are `ctrl-shift-/` and
  `ctrl-?`.
- Separate keys in a sequence with spaces. The word `leader` stands for the key `leader`
  is set to.
- Your keys are added to the defaults and win where they overlap. To remove a default, set
  its key to `none`. The command names are in section 19.
- A name that doesn't exist, or a key that can't be read, is reported in the top bar; the
  rest still apply.

The top bar also reports conflicts between your own keys: the same key for two commands
(`ctrl-shift-y` and `shift-ctrl-y` are the same key), a key that's also the start of a
sequence, and the leader bound to a command itself.

**To see every key in one place and move any of them**, write out a whole keymap:

```toml
[keys.direct]                  # what keymap = "direct" binds
"ctrl-shift-t" = "new-tab"
"ctrl-t" = "split-right"

[keys.leader]                  # what keymap = "leader" binds: the key after the leader
t = "new-tab"
"shift-h" = "resize-left"
```

- A table lies over that keymap's defaults, and your lines win. A key or command the table
  doesn't mention keeps its default, so keys added in a later version work without your
  touching the file.
- To take a default key away, write it with `none`: `"ctrl-shift-t" = "none"`. Taking a
  line out of the table brings the default back.
- Each table is read only in its own keymap, and `[keys]` is read in both, on top of the
  table.
- Copy, paste, paste-selection, find and the key that locks aren't in either table; change
  them in `[keys]`.
- `isc config` prints both tables in full, with unbound commands as lines to uncomment.

**You can make commands of your own.** A command is a name and a list of steps under
`[commands]`. You bind it to a key like any other, and the palette lists it. If one is
written wrongly, the top bar says so and only that command is left out.

```toml
[commands]
tidy = ["minimize-others", "zoom"]
tests = [{ run = "cargo watch -x test", in = "right" }]

[commands.review]
title = "Review: lazygit over the tab"
steps = [
  { run = "lazygit", in = "float", cwd = "~/src/app" },
  "float-fill",
]

[keys]
"ctrl-alt-t" = "tests"
"leader shift-t" = "tests"
```

A step is one of three things:

- **A command's name**, in quotes: any command in section 19, such as `"split-right"`.
- **`{ run = "…" }` runs a program in a new pane.** `in` says where: `"tab"` (a new tab,
  the default), `"right"` or `"down"` (beside the pane in front), or `"float"`. `cwd` is the
  directory it starts in, and `~` is your home on the machine the pane is on. `keep = true`
  leaves the pane open when the program ends. `run = ""` runs your shell. The words are
  split at spaces; when an argument has a space, write a list:
  `run = ["sh", "-c", "make && ./serve"]`.
- **`{ type = "…" }` types text into the pane in front**, as if at its keyboard. End it
  with `\n` to press Enter: `{ type = "git status\n" }`.

What to know:

- The steps run in order. After a step that opens a pane, the pane in front is the new one:
  `[{ run = "", in = "right" }, { type = "npm test\n" }]` types into the new shell.
- If a step fails, the rest don't run, and a notice says which step stopped. A step that
  would ask first, such as `close-tab` with something running, asks, and the steps after it
  don't run.
- A name is letters, digits and hyphens, and can't be a command that already exists.
- The palette and the leader's panel ("Your commands") list yours; the sheet of keys
  doesn't.
- - `isc do NAME` runs one from a shell: see [Running things](#running-things).
- A step can't be another command of your own, and there are no conditions or loops.

**A mode makes plain keys into commands.** For things you do many times in a row, like
moving a pane's edge, a mode is a set of keys that run commands with no leader and no
chord, for as long as the mode is on. One comes ready: **resize mode**. Turn it on, press
the arrows or `H` `J` `K` `L` as often as you like to move the edges of the pane in front,
and press Escape when you're done.

- **Turning it on.** In the leader keymap, and in a terminal, it's leader, then `Shift+R`.
  In the direct keymap it has no key: choose "Resize mode" in the palette, or give
  `resize-mode` a key under `[keys]`.
- While it's on, its keys aren't sent to the program.
- Escape or Enter turns it off. Any other key turns it off too, and then does what it
  always does, so if you start typing, the first letter is typed.
- A small sign with the mode's name stays at the bottom until the mode is off.

You can write modes of your own, and change the resize mode, under `[modes]`:

```toml
[modes.resize]
title = "Resize mode"
"left" = "resize-left"
"down" = "resize-down"
"up" = "resize-up"
"right" = "resize-right"
"h" = "resize-left"
"j" = "resize-down"
"k" = "resize-up"
"l" = "resize-right"

[modes.tabs]
title = "Walk the tabs"
timeout = 3
"n" = "next-tab"
"p" = "previous-tab"
"x" = "close-tab"

[keys]
"leader shift-t" = "tabs-mode"
```

- The command that turns a mode on is its name with `-mode` after it: `resize-mode`,
  `tabs-mode`. Bind it like any other.
- Each key runs one command: one of insensical's, or one of your own from `[commands]`. A
  key is written as under `[keys]`, and is one key, not a sequence.
- `timeout` is seconds with no key before the mode turns itself off. Without it the mode
  stays on until you leave it.
- `[modes.resize]` in your settings replaces the one that comes ready, whole: write every
  key you want it to have.
- A mode's name is letters, digits and hyphens. `NAME-mode` can't be a command that already
  exists, so there can't be a mode called `copy`.
- A key in a mode can't turn on another mode. The page in a browser has no modes.

### Keys that are fixed

| Where | Keys |
|---|---|
| The sidebar field, the palette, the search field | The keys of a field below, Enter, Escape |
| A held paste | Enter pastes, Escape drops it |
| A kept pane whose program has ended | Enter closes it |
| The restart question | Enter restarts, Escape doesn't |
| The question before a tab or a project closes | Enter closes, Escape doesn't |
| The new project's name and directory | The keys of a field below, Tab, Shift+Tab, Up, Down, Enter, Escape |
| The question about agents | Enter adds the hooks, Escape doesn't, `N` stops the offer |
| Adding a machine | The keys of a field below, Up, Down; Enter meets the marked machine, then gives the answer of the coloured button; Escape leaves |
| Where one of several machines is chosen: a new project, the settings sheet | `Alt+M` goes to the next machine |

Every place you type a line is a proper field: a name, a search, the palette, an address, a
token. Pressing puts the caret there, dragging selects, and in the window a double press
takes a word and a triple press the line. In the palette and other lists, Up, Down, Tab and
Enter stay the list's, and Left, Right, Home and End are the field's.

| | In the window (Cmd for Ctrl on a Mac) | In a terminal |
|---|---|---|
| Move a character | `Left`, `Right` | `Left`, `Right`, `Ctrl+B`, `Ctrl+F` |
| Move a word | `Ctrl+Left`, `Ctrl+Right` (`Option` on a Mac), `Alt+B`, `Alt+F` | `Ctrl+Left`, `Ctrl+Right`, `Alt+Left`, `Alt+Right`, `Alt+B`, `Alt+F` |
| Start, end | `Home`, `End` (`Cmd+Left`, `Cmd+Right` on a Mac) | `Home`, `End`, `Ctrl+A`, `Ctrl+E` |
| Select | `Shift` with any move; `Ctrl+A` selects all | `Shift` with any move; `Cmd+A` or `Ctrl+Shift+A` selects all, where your terminal sends them |
| Delete | `Backspace`, `Delete` | `Backspace`, `Delete` |
| Delete a word | `Ctrl+Backspace`, `Ctrl+W`, `Ctrl+Delete`, `Alt+D` (`Option` on a Mac) | `Ctrl+Backspace`, `Alt+Backspace`, `Ctrl+W`, `Ctrl+Delete`, `Alt+D` |
| Delete to the start, to the end | `Ctrl+U`, `Ctrl+K` (`Cmd+Backspace`, `Cmd+Delete` on a Mac) | `Ctrl+U`, `Ctrl+K` |
| Undo, redo | `Ctrl+Z`, `Ctrl+Shift+Z` or `Ctrl+Y` | `Ctrl+Z` or `Ctrl+/`, `Ctrl+Shift+Z` or `Ctrl+Y` |
| Copy, cut, paste | `Ctrl+C`, `Ctrl+X`, `Ctrl+V`, `Shift+Insert` pastes | Paste with your terminal's paste; there's no copy or cut |

A paste is always one line. A field for a secret, such as a token, shows dots and copies
and cuts nothing to the clipboard. Where a key above already means something on that
surface, such as `Ctrl+Left`
and `Ctrl+Backspace` on a setting that has a value to change or reset, it keeps that
meaning, and the field has `Alt+B`, `Alt+F` and `Ctrl+W` instead.

Every question that opens over the window is a card with buttons at its foot, each naming
the key that gives the same answer. Enter gives the coloured button on the right: the accent
colour when it doesn't end anything, red when it does.

---

## 14. The command palette and the overview

The palette gets you to anything by typing a few letters. The overview shows every tab of
every project at once, so you can spot what needs you and jump to it.

### The palette

Press `Ctrl+Shift+P`, or leader `Space`. With nothing typed, it lists everything under
headings, in this order: **Needs you** (the panes that need you), **Recent** (the last five
commands and settings you chose), **Commands**, **Settings**, **Projects**, **Panes** and
**Themes**.

- Your own commands from `[commands]`, and the command that turns on each mode from
  `[modes]`, are under **Commands**.
- Each row has an icon: a pane's program (or its state mark if it needs you), a folder for
  a project, a cog for a setting, a palette for a theme.
- A command shows its keys, a setting its value, and a pane its project and, if it needs
  you, what it says.
- **Typing narrows the list** by any of your words, in any order, and the headings go. A
  word at the start of a name ranks highest, then the start of a later word, then inside a
  word. Of two equal fits, the one you chose more recently comes first.
- Type `pane`, `project`, `setting` or `theme` to ask for that kind of thing.
- `Enter` runs the command, goes to the project or pane, or chooses the theme.
- **`Enter` on a setting opens it on the settings sheet**, with its list open if it has one.
  The palette never changes a setting to a value you haven't seen.
- Themes listed are the built-in ones, those in your `themes` directory, and `ghostty` when
  Ghostty's configuration names a theme.
- **Themes are previewed.** Highlight one and the whole window shows it. `Enter` keeps it
  and
  writes `theme = "…"` to the settings, leaving the rest of the file, comments included,
  alone. `Esc` goes back to the theme in use. If the settings file can't be read, the theme
  is applied but not written, and the top bar says why.

### The overview

Press `Ctrl+Shift+Space` or leader `w`, or choose "See every tab of every project at once"
in the palette. It covers the window with each project's name and its tabs drawn small, in
their layouts: tiled panes where they sit, floating panes over them, minimized panes as a
row of icons. Projects sit side by side while there's room.

- A pane shows its icon, name and a border in its state's colour, and the memory its
  programs hold once that's a megabyte or more.
- Under that is the pane's screen, small and live. You can't read it; it shows what kind of
  thing is there (an editor, a scrolling log, a prompt) and whether it's moving.
- A pane that's waiting, done or failed also shows what it says, readable, over the foot of
  its picture. You can read an agent's question without going there.
- A project's name is followed by how many of its panes need you.
- The overview opens on the tab in front, and the pane you came from says "here". `Left` and
  `Right` step through every tab in order, across projects. `Up` and `Down` go to the
  nearest tab in the row above or below.
- In the tab the keyboard is on, a ring marks the pane `Enter` goes to: the first that needs
  you, or else the one that had the keyboard. `Space` moves the ring to the tab's next pane.
- A sleeping project is marked "asleep". `Enter` on one of its tabs wakes it and takes you
  there, the same as "Wake it" on the project's page.

| Key | Does |
|---|---|
| `Enter` | Go to the ringed pane |
| Click a pane, or a tab's title | Go to that pane, or that tab |
| `Tab` | Show only tabs with a pane that needs you; again for all. If none do, it says "Nothing needs you." |
| `Esc`, or a click elsewhere | Close, changing nothing |

Looking never resizes a pane or tells its program anything.

Good to know: the small screens are live only while the overview is open, and only for the
panes in view, so a very busy pane's picture may lag. A dropped link shows "link lost"
over a
machine's panes. `isc tui` has an overview of layouts and words, with no small screens.

### Finding a file

Type a file's path into the pane without typing the path. Press `Ctrl+Shift+I` or leader
`/`, or choose "Find a file under the pane's directory and type its path" in the palette.

A small list opens at the cursor, showing the files and directories under the focused
pane's directory, each with an icon. With a daemon on another machine, they're that
machine's files. Type a few letters, then press `Enter` to type the path into the pane.

- The path `Enter` would type is shown dimmed at the cursor as you move through the list.
  Nothing reaches the program until you press `Enter`.
- Your letters must appear in the path in order, in any case; the nearer path wins ties.
- `Up` and `Down` choose among up to ten paths. `Enter` types the path and a space. `Esc`,
  or
  a click outside, types nothing.
- **`Tab` goes into the marked directory.** The list then holds only what's in it.
  `Shift+Tab`, or `Backspace` with nothing typed, comes back out. The path typed is still
  from the pane's directory.
- The path is relative, as in `src/main.rs`; a directory's ends in a slash. It's quoted when
  a shell would read it as more than one word, as in `'my notes/to do.md'`.
- **What git ignores isn't listed.** Inside a repository the list leaves out everything git
  ignores. The daemon asks git once, as the list opens, and git is told to run nothing,
  fetch nothing and write nothing. Outside a repository, or without git, only the skipped
  directories below are left out.

Good to know: directories named `target`, `node_modules` or `__pycache__` and those starting
with a dot are skipped, and at most 30 000 entries are listed, nearest first. A file whose
name
holds a control or text-direction character isn't listed.

---

## 15. Settings and themes

Everything about how insensical looks and behaves lives in one file you can edit by hand
or through the settings sheet.

Settings live in `~/.config/insensical/config.toml`
(`$XDG_CONFIG_HOME/insensical/config.toml`
if that's set; `INSENSICAL_CONFIG_DIR` overrides both). You needn't create anything: without
the file, the defaults apply.

**The window makes the file the first time it starts**, if there isn't one. It explains
every
setting and shows its default, but sets none: each setting line begins with `#`, so a
default
that changes in a later version still reaches you. Remove the `#` to set a line, or use the
settings sheet, which writes its line under the one that shows the default.

- A file that's already there is never written over.
- `isc` on its own doesn't make the file. The window does, and so does `isc config set`,
  with its one line.
- The keymaps aren't in it. `isc config` prints them.

`isc config` prints a complete file: every setting at its default with a note and its
possible values, the `[tui]` table, and both keymaps.

```sh
isc config > ~/.config/insensical/config.toml
```

To read or change one setting without opening the file:

```sh
isc config get font-size        # what the file gives it, or its default
isc config set font-size 13
isc config set sidebar tabs,panes
isc config reset font-size      # takes the line out, which leaves the default
```

- `set` changes that one line and keeps the rest of the file as you wrote it. If there's no
  file, it makes one with that line.
- A setting shown in a comment, such as `# shell = "fish"`, is written under that comment.
  `reset` takes out only that line.
- A value the setting can't have is refused, and the message says what it can be.
- `colours` and `mouse`, which only the interface in a terminal has, are read, set and reset
  under `[tui]`.
- These work for the settings in the table in section 19, not for the `[keys]` tables.

**Some settings are the daemon's, and the rest are the interface's.** The daemon's are
`shell`, `agents-at-once`, `agents-deep`, `term`, `scrollback`, `keep-scrollback`,
`tmux-shim`, `clipboard`, `notifications`, `notify-waiting`, `notify-failed`, `notify-done`,
`sound`, `long-command`, `tray`, `notify-command`, `notify-command-when`, `notify-url`,
`notify-url-when`, `notify-url-body`, `notify-url-headers`, `notify-url-token`,
`notify-url-user` and `notify-url-password`. On one machine they share a file. With a daemon
on another machine, its settings are in the file there and the interface's are here:

```sh
isc --host build config get scrollback      # what the file on build gives it
isc --host build config set scrollback 50   # written to the file on build
isc --host build config set theme paper     # refused: theme is this machine's
```

The daemon there makes the change, so it must be running. `isc --host build config` alone
prints the same complete file as without `--host`.

An example:

```toml
theme = { light = "paper", dark = "fjord" }   # or "ghostty", or "ghostty:Tokyo Night"
font-family = "JetBrainsMono Nerd Font"
font-size = 12
shell = "fish"
keymap = "leader"
```

The settings table is in section 19.

### Icons for your own programs

Give a program insensical doesn't know its own tile, under `[icons]`, by the name it runs
under:

```toml
[icons]
gitu = "git"                                    # as git is shown
myagent = "claude"
deploy = { mark = "docker", colour = "#ff8800" }   # Docker's mark on a tile of your colour
```

- The value names a program that has a tile (`git`, `nvim`, `claude`, `docker`) or a mark
  (`agent`, `activity`, `remote`, `shell`, `terminal`).
- `colour` is the tile's own, as `#rrggbb`. Without it the tile keeps the named program's
  colour.
- A line here beats what's built in, so you can change a known program too: `node = "vite"`.
- The name is the one in the pane's header and in `isc ls`; a name the program reports
  (section 10) is tried first.

### The settings sheet

The sheet lets you change settings by clicking and see the result at once. Press `Ctrl+,`
(`⌘,` on macOS) or leader `,`, or choose "Settings" in the palette. It opens over the
window,
which stays in view. Nothing is typed as a value: a setting is switched, picked or
stepped. A
number steps by one, the background's opacity by 0.05, and `long-command` (up to 3600
seconds) by five.

- **Pages are down the left**, under the search field: **Hub**, then Interface and Themes,
  then Terminal, Shortcuts and Remote Hosts, then Notifications and Agents. On a Mac, Menu
  bar and Dock follows Themes. The sheet opens on the page you last looked at. Arrows by the
  page's name go back and forward through pages you've visited. `Esc` closes the sheet.
- **A change applies at once** and is written to that setting's one line in `config.toml`.
- **Type to find a setting** by name or what it does. Results are listed under their page.
- **To reset a setting** your file changes, put the pointer on its row and press the small
  arrow that appears left of the control, or press `Ctrl+Backspace`. Its line leaves the
  file.
- **Showing** is at the top right of Hub, Terminal, Notifications and Agents once you've
  named another computer. It says which computer the page is about: press it for the list,
  or `Alt+M` to step through them. A computer that can't be reached can be chosen; its page
  says so and waits.

| Control | For | Change it |
|---|---|---|
| A switch | on or off | Click, or `Enter` |
| Two words on a track | one of two, such as the density or where the tabs are | Click one, or `Enter` for the other |
| A word with `⌄` | one of more than two, such as the pane header, motion, or how long a device stays paired | Click or `Enter` for a menu with the current value ticked. Arrows, `Home`, `End`, a letter, `Enter` to choose, `Esc` to leave it |
| Buttons side by side | any of a few, such as what the sidebar lists | Click one to tick or untick it, or `Enter` to go through every combination |
| A number with `−` `+` | text size, space between panes | Click `−` or `+`, or `Ctrl+Right` and `Ctrl+Left` |
| A value with `›` | a theme, a font, a shell | Click or `Enter` for a list in place of the rows. Type to narrow, `Enter` chooses, `Esc` goes back |
| Key caps | the leader | Click or `Enter`, then press the key. It needs `Ctrl`, `Alt` or `Cmd` held (`⌃`, `⌥`, `⌘`). `Esc` cancels |

| Key | Does |
|---|---|
| `Up`, `Down` | Move between rows |
| `Enter` | Change the marked setting, or open its list |
| `Ctrl+Right`, `Ctrl+Left` | Next or previous value; one more or one less |
| `Ctrl+Backspace` | Reset the marked setting |
| `Tab`, `Shift+Tab` | Next or previous control on the page |
| `Space`, `Enter` on a control | Press it |
| `Ctrl+Tab`, `Ctrl+Shift+Tab` | Next or previous page |
| `Esc` | Close the sheet, or leave a list |

The pages:

- **Hub** is the first page, for this computer's server and who can reach it. Four things
  live here; the hub sections have the details.
  - **Server and Connected**: the server's state, version and the paths of its socket and
    settings file, each with a copy button, and `Restart Server`, which asks first. Under
    **Connected** is everything looking at this computer's terminals, with what each can do
    and for how long. `Disconnect` ends one; for a named browser or phone, `Disconnect and
    Remove Device` also ends its pairing.
  - **Access from your other devices**: the switch **Open in a browser or on your phone**
    starts and stops `isc hub` for this computer. While it's on you can open a browser
    here with `Open in Browser`, copy a sign-in link for another, and turn on **Reach it
    from anywhere on your tailnet**. **Notifications on your devices** counts the devices
    that asked for them, and `Send a Test` tries each. See [Reaching isc hub from another
    device](#reaching-isc-hub-from-another-device).
  - **Your devices**: each paired device, with its icon, name and when it was last used or
    ends. Press one for **Pair Again** or **Remove**. A guest has `End`. `Add a Device`
    asks for a name, what it may do (Everything, Look and type, or Only look), how long it
    stays paired and its icon, then shows a QR code and a link. See [Pairing a
    device](#pairing-a-device) and [Sharing one project or one
    terminal](#sharing-one-project-or-one-terminal).
  - **Joined machines** (on a hub) lists computers joined to it, with `Remove` and `Invite
    a Machine`. **Join a hub** (on any computer that joined none) has `Join a Hub`, where
    you paste the invite line. See [Joining another machine to your
    hub](#joining-another-machine-to-your-hub).
  - A computer that joined a hub shows **Joined to the hub at** in place of the switch:
    the hub's name, whether it's connected, `Start` if `isc hub` isn't running, and
    `Leave`. It has no browser link, Tailscale row, Joined machines or Add a Device, since
    your devices reach it through the hub.
- **Interface** has four cards: **Panes** (density, space between panes, pane header),
  **Tabs and sidebar** (where the tabs are, what the sidebar lists), **Terminal text**
  (terminal font, text size, ligatures) and **Window** (interface font, motion). The
  terminal font list shows only fonts of one width; any other can be named in the file.
- **Themes** has two cards: **Colours** (the theme, the interface colours) and **Behind the
  window** (the background's opacity and blur). A theme is shown on the whole window as you
  move through the list; `Esc` puts back the one you had. A theme for each appearance,
  `{ light = …, dark = … }`, shows as "By the desktop's appearance". Picking a theme from
  the list replaces the pair with one theme; to have a pair again, write it in the file.
- **Terminal** has four cards: **Shell** (the shell, the terminal type; the shell list holds
  the shells in `/etc/shells`), **Typing** (input method, Option as Alt, type ahead),
  **History** (how much each pane keeps, and whether it's kept when the server stops) and
  **Files and clipboard** (what opens a file, and whether programs may write the
  clipboard).
- **Remote Hosts** lists the computers you've named, one card each. The first row shows its
  state (Connected, Can't be reached, or that its server can't be used) and where ssh goes,
  with a `Check` button that meets it and says what's wrong in plain words. Press the row to
  fold or open the card. Open, it has the colour and icon (dropdowns), whether it's careful
  (it asks before a paste of more than a line), `Rename`, the address (`Change`, type it,
  `Enter` keeps it, `Esc` leaves it) and `Forget`, which asks once and changes nothing on
  that computer.
  - A card starts open if it's your only one, just added, or needs you; otherwise folded.
  - When a check finds insensical missing or older there, a row offers `Update There` or
    `Restart Its Server`. Where `ssh` has a question, the row has an `Answer` button.
  - `Add Remote Host` opens a dialog with the machines of your tailnet and the names in your
    ssh configuration under two tabs; anything you type is given to ssh.
- **Notifications** has three cards: **Being told** (notifications at all, the tray's icon),
  **What you're told of** (a pane that waits, something that fails, something that
  finishes, and how long a command counts as long) and **Light and sound** (the glow around
  a waiting pane, the sound, and a `Play` for each of the two sounds).
  - **When you're away** is how you're told on your phone when something needs you and no
    window or terminal is open. Choose **ntfy** and give the server (the free
    `https://ntfy.sh` unless you change it), a topic, and an access token if your server
    needs one; or choose **Another web address** and give the address. **Only when I'm
    away** is a switch: off, you're told every time. `Send a Test` says how it went, and a
    quiet line says how the last alert went.
  - **More options** holds how an address is sent, a user and password, a token and a
    command to run. Secret fields show only dots.
  - The settings behind it are `notify-url`, `notify-url-token`, `notify-url-body`,
    `notify-url-when`, `notify-url-user`, `notify-url-password`, `notify-command` and
    `notify-command-when`. Giving an ntfy topic also sets `notify-url-body` to `text`.
    `notify-url-headers` is set only with `isc config set`.
- **Agents** starts with how many agents one agent may start and how many levels deep, each
  a number between − and +. Under them is each agent it knows and whether it reports what
  it's doing: it does, it doesn't, its hooks are stale (they name an `isc` that's gone), or
  the agent isn't on this system. `Set up` asks before adding hooks, as in
  [section 11](#11-agents).
- **Shortcuts** has the keymap and the leader under **Keys**. The keys themselves are tables
  in the file; under **Every key** are buttons for the sheet of every key and for opening
  the file. The `[keys]` tables and a theme's own file aren't on the sheet.
- With more than one machine, pages that hold a server's settings say whose they are; see
  [Other machines in the window](#other-machines-in-the-window).

### When a change applies

- **Saved changes apply within a second.** The window checks the file once a second; you
  needn't restart.
- The daemon reads `shell` and `keep-scrollback` when the next pane starts. A pane already
  open keeps its values.
- The daemon reads `clipboard` each time a program asks, so it applies right away.
- Changing `sidebar`, `pane-header`, `density` or `pane-gap` changes how much room terminals
  have, so they're resized.

### When the file is wrong

- **One unreadable line voids the whole file.** A misspelt or unknown key, a value of the
  wrong type or broken TOML shows in the top bar with its line number, and *every* setting
  falls back to its default until you fix it.
- A wrongly written command under `[commands]` or mode under `[modes]` is reported and left
  out alone.
- A value that reads but can't be used (a font size of 500, a theme that doesn't exist, a
  leader that isn't a key, a shell that isn't a program) is reported, and only that setting
  falls back.
- **The daemon is more forgiving.** It reads its own settings one at a time: a value it
  can't
  read leaves only that setting at its default, and a broken file leaves the settings it
  last
  read. It tells you what it couldn't use in a notification, at the end of `isc notify
  --sinks`, and through `isc --host NAME config get`.

### Themes

A theme is a terminal's colours: foreground, background, the sixteen named colours, and
optionally cursor and selection. The window's colours are derived from them, so any terminal
theme styles the whole application. Text is shifted in lightness until it's readable.

Eight themes come with insensical:

| Name | Looks like | Has |
|---|---|---|
| `fjord` | Cool blue-grey, with a sea-green accent | dark |
| `ember` | Warm brown, with an amber accent | dark |
| `paper` | Warm off-white, with dark ink | light |
| `tidepool` | Deep sea-green water, with coral | dark and light |
| `heather` | A moor at dusk: plum, with a heather-pink accent | dark and light |
| `lichen` | Moss on stone: olive green, with a lichen-yellow accent | dark and light |
| `lantern` | An indigo night lit by amber; by day, ink on rice paper | dark and light |
| `quartz` | Quiet grey with soft colours and a touch of rose | dark and light |

A theme with both follows the desktop: `theme = "lantern"` is dark when the desktop is dark
and light when it's light, and changes as the desktop does. It can't be held to one side.

A theme name is looked up like this:

| Name | Means |
|---|---|
| `ghostty` | The theme named in Ghostty's own configuration for the desktop's current appearance |
| `ghostty:Name` | The Ghostty theme with that name |
| anything else | `themes/NAME.toml` beside the settings file, then the themes that come with insensical |

- Without `theme`, you get `fjord` on a dark desktop and `paper` on a light one.
- A pair, `{ light = "…", dark = "…" }`, follows the desktop's appearance.
- Ghostty's themes are read in place from where Ghostty keeps them, on macOS inside
  `Ghostty.app`. **Without Ghostty installed, `ghostty` and `ghostty:…` are errors.**
- **Only the theme comes from Ghostty.** Its font, font size, font style, padding, opacity,
  blur, cursor style and `command` aren't used.
- The terminals' colours always come from the theme. The window around them is grey unless
  you set `interface-style = "terminal"`. A program's tile is always in its own colours.

### A neutral interface

By default the window around the terminals is grey: near-black on a dark desktop, near-white
on a light one, never lighter than the theme's own step under its panes. That leaves the
colour to the terminals. `interface-style = "terminal"` (the switch Follow terminal theme in
the settings) works the window out from the theme instead, so a warm theme gives a warm
sidebar. `isc tui` is the other way round: it's in the theme's colours unless you write
`interface-style = "insensical"`.

- In grey, the sidebar, tabs, headers, borders and sheets are grey; the theme's accent, the
  state colours and the terminals stay as they are.
- Surfaces a theme file chose under `[interface]` aren't used in grey; its `accent`,
  `running`, `waiting`, `done` and `failed` are.

### A see-through window

`background-opacity` sets how much of what's behind the window its background hides, from
`0.2` to `1`. At `1`, the default, the window is solid but for the little `background-blur`
lets through. Below that, your desktop shows through behind the text:

```toml
background-opacity = 0.85
background-blur = true
```

- Only the background thins. Text, cells a program coloured, tiles, borders, sheets and
  questions stay solid.
- A floating or zoomed pane stays solid, since it lies over other panes.
- `background-blur` is on unless you turn it off. It asks the desktop to blur what shows
  through, and even at an opacity of `1` it lets a little through the sidebar and the space
  around the panes. On Hyprland and other compositors that blur by their own rules, turn
  blur
  on for the window there (the class and title are both `insensical`). On a desktop that
  doesn't blur, turn it off if you'd rather the sidebar be fully solid.
- With no compositor there's nothing to show through, and the window is drawn over black.
- Both apply as you save. Text over a busy picture is harder to read, and nothing corrects
  for that.

### A theme file

Put your own theme in `~/.config/insensical/themes/<name>.toml`:

```toml
name = "Fjord"

[terminal]
foreground = "#d0d4dc"
background = "#101216"
cursor = "#d0d4dc"        # optional: the foreground if left out
selection = "#7fd1c7"     # optional: the accent if left out
palette = [               # the sixteen named colours: eight normal, then eight bright
    "#1b1e25", "#f0706a", "#7fd18b", "#f0b35a", "#6fa8ff", "#c58fe8", "#7fd1c7", "#c5cad3",
    "#4c5464", "#ff8f8a", "#9be3a5", "#ffc978", "#93beff", "#d9aef5", "#9fe6dc", "#eef1f6",
]
```

Light or dark is worked out from the background, unless `appearance = "light"` or `"dark"`
at the top of the file sets it.

One file can hold a theme for both. Write each side under `dark` and `light`, and the
desktop's appearance picks:

```toml
name = "Mine"

[dark.terminal]
foreground = "#d0d4dc"
background = "#101216"
palette = [ … ]

[light.terminal]
foreground = "#2b2a27"
background = "#f6f3ec"
palette = [ … ]

[light.interface]         # optional, as [interface] is
accent = "#1f7a78"
```

- A file has either `[terminal]`, or both `[dark.terminal]` and `[light.terminal]`. One side
  alone, or mixing the two forms, is an error.
- A theme needs only `[terminal]` with `foreground`, `background` and `palette`. You can
  leave out `name` (it's called "unnamed"), `appearance`, `cursor`, `selection` and all of
  `[interface]`.
- A colour is six hexadecimal digits, with or without the `#`. Shorter forms such as `#fff`
  and colour names are errors.
- `palette` must have exactly sixteen colours.
- An unknown key is an error. If a theme file can't be read, the top bar reports it by name
  and the default theme is used.

To set any interface colour yourself instead of having it derived:

```toml
[interface]
canvas = "#0a0b0e"       # behind and between panes
raised = "#171a20"       # under the pointer
selected = "#1e222b"     # the current tab, project or pane in a list
border = "#242933"
text = "#d0d4dc"
text-muted = "#767f90"
text-faint = "#4c5464"
accent = "#7fd1c7"       # the focused pane, the current item
running = "#6fa8ff"      # pane states
waiting = "#f0b35a"
done = "#7fd18b"
failed = "#f0706a"
```

What you leave out is derived from the theme: the canvas is the background a step darker,
muted and faint text sit between the foreground and the canvas, the accent is the
palette's blue, and the state colours are its blue (cyan if the accent took the blue),
yellow, green and red. Text that wouldn't stand out enough is shifted in lightness until
it does, colours you set included.

---

## 16. The command line, `isc`

`isc` is the daemon and its command line in one binary. Anything you do in the window you
can do from a shell, and scripts can wait for a pane or follow events. The full list of
commands is in section 19.

### Naming things

Point `isc` at a pane, tab or project and it works out the rest.

- A pane is its number as `isc ls` prints it (`3` or `pane:3`), or the name you gave it
  with `isc rename` or `--name`. A tab works the same way (`7`, `tab:7` or its name). Names
  ignore case, and a number is always a number.
- A project is its name or its number. If two projects share a name, the name means the
  first one listed. See [Projects](#projects).
- A layout command with no pane acts on the pane it's run in (`$INSENSICAL_PANE`), or
  outside any pane, the one with the keyboard. `isc status` and `isc notify` never guess:
  outside a pane they need `--pane`.
- Every command takes `--socket PATH`, which names the daemon's socket in place of
  `$INSENSICAL_SOCKET` and the default.
- `isc --help` and `isc COMMAND --help` describe the commands. `isc --version` prints the
  version of `isc` only.

### Which commands start the daemon

A command about panes, tabs or projects starts the daemon if it isn't running. So do `isc
status explain`, `isc notify --test`, `isc notify --sinks`, and `isc restart-server` when
no daemon is running.

These never start it: `daemon`, `kill-server`, `version`, `status` with a state, `notify`
with a title, `integrate`, `shell-integration`, `config`, `completions`, `host` and the
agents' hooks. With `--host`, the same holds on the other machine.

Nothing is written to a log file. Where you read what insensical says depends on what
started it:

| Started by | Where it prints |
|---|---|
| `isc` | Where you ran it |
| The window | Whatever started the window. Run `insensical` in a terminal to read it. Mistakes in the settings file are among it, and the settings sheet lists the same ones |
| The user unit | The journal: `journalctl --user -u insensical` |
| On demand, by a window or command | Nowhere. To read it, stop the daemon and run `isc daemon` in a terminal |

### Listing

`isc ls` (or `isc list`) shows your projects, tabs and panes, and marks the current
project, tab and focused pane with `*`.

- A project's line has its number, name and root, and `asleep` while it sleeps.
- A tab's line has its number, its name if it has one, and `zoomed` while a pane fills it.
- A pane's line has its number, state, program, directory (`~` for home), name or title,
  and what its state is about, each after two spaces. Empty parts are left out. The state is
  one word and the program is the process's name, so a script can count columns up to there.

After those, each after two spaces, come the parts that are only sometimes there:
`floating` or `minimized`; what a waiting pane waits for and how long it has been
`waiting`, `failed` or `done` (`wants permission, 2h`); the name a program gives itself,
in brackets; how many things it reports; `opened by pane:N` for a pane that `isc run` or
`isc split` opened from inside another pane that's still open; and `keeps agents out`.
Scripts should read `isc ls --json` for these. If an update has been installed under the
running daemon, `isc ls` says so on standard error.

```text
* project:5  api  /home/you/code/api
    tab:4  zoomed
      * pane:3  waiting  node  ~/code/api  Refactor the auth middleware  Allow the edit to session.rs?  wants permission, 2h  (claude-code)
        pane:6  idle  nvim  ~/code/api  src/session.rs  opened by pane:3
  * tab:8  servers
      * pane:10  idle  fish  ~/code/api  floating
        pane:7  waiting  cargo  ~/code/api  minimized
  project:9  web  /home/you/code/web  asleep
      tab:12
        pane:11  idle  fish  ~/code/web
```

Narrow the list with these. Panes that fit are shown under the projects and tabs that hold
them, and the command fails with nothing listed when none fits. Given together, a pane
must fit them all.

| Option | Lists |
|---|---|
| `--program NAME` | Panes running that program, by process name (`psql`) or the name it reports (`claude-code`), in any case |
| `--state STATE` | Panes in that state: `idle`, `running`, `waiting`, `done` or `failed`. `isc ls --state waiting` is everything waiting for you |
| `--opened` | Panes opened by the pane `isc` runs in |
| `--here` | Panes of the tab the pane `isc` runs in |

`isc ls --json` prints everything the daemon knows, and is the stable format for scripts.
`isc usage`, `isc clients` and `isc api` take `--json` too, and `isc integrate --status`
always prints JSON.

### Running things

`isc run` opens a command in a new pane, or your shell if you give none, and prints
`pane:N`.

```sh
isc run [--cwd DIR] [--keep] [--name NAME] [--background] [--wait] [--json] [WHERE] [-- COMMAND…]
```

The working directory is the one you run `isc` in. Give only one place:

| `WHERE` | Place |
|---|---|
| nothing | A new tab in the current project |
| `--project NAME` | A new tab in that project |
| `--new-project [NAME]` | A new project for the working directory, called `NAME` if you give one |
| `--split right\|down [--beside PANE]` | Beside a pane, sharing its space. `--beside` names the pane when it isn't the one `isc` runs in or the focused one |
| `--float [--rect X,Y,W,H]` | Floating above the current tab |

`--keep` leaves the pane open when the command ends so you can read what it printed. Press
Enter in it, or run `isc close`, to close it.

`isc run` and `isc split` share these options:

| Option | Does |
|---|---|
| `--name NAME` | Names the pane from the start, as `isc rename` would |
| `--background` | Opens the pane behind, leaving the keyboard, tab and project you're looking at where they are |
| `--wait` | Waits for the command to end, then prints the end of its output (the last whole lines that fit in 4 KB) and a last line with its status (`exit 7`, or `exit 7, 1980 earlier lines left out`). It exits with that status, so `isc run --wait -- make test && echo passed` works. The pane closes afterwards unless you add `--keep`. If you stop `isc` while it waits, the pane stays open |
| `--json` | Prints `{"pane":12,"tab":7}` in place of `pane:N`. With `--wait` it's printed once the command ends, with `status`, `output` and, when lines were left out, `left_out` |

A pane you open from inside another pane remembers that pane as its opener. `isc close
--opened`, run in that pane, closes every pane it opened and every pane those opened, and
nothing else. An agent can tidy up after itself without touching your panes. A pane you've
typed into or focused stays, with what it opened, and the command says `pane:9 left open:
the user is using it`. `isc close --opened 3` does the same for pane 3. A pane whose
opener has closed is no one's.

`isc do NAME` runs a command by name: one of your own from `[commands]` in the settings
file on this machine (see [Changing keys](#changing-keys)), or one of the commands in
section 19.

```sh
isc do tests            # your own
isc do split-right      # one that exists already
isc --host build do tests
```

- Steps run in order on the daemon the command is for. If one fails, `isc do` stops there,
  says which step, and exits with an error.
- A command only an interface can do, such as `palette` or `copy`, is refused, and so is a
  list at the step that holds one.
- A step that would end something running, such as `close-tab`, is refused unless you say
  `--yes`.

### Typing into a pane and reading it

These commands let a script drive a pane and read what came back.

| Command | Does |
|---|---|
| `isc send PANE TEXT` | Types text. Text may begin with a hyphen. If it's one of `isc`'s own options (`--host`, `--rights`, `--socket`), put `--` first: `isc send 3 -- --host`. The same goes for a title given to `isc notify`, which also has `--test`, `--sinks` and `--pane` |
| `isc key PANE KEY…` | Presses keys by name, in order, encoded as the pane's program expects: `isc key 3 escape : w q enter`. Write a key as in `[keys]`: `ctrl-c`, `enter`, `escape`, `shift-tab`, `space`, `a`, with `ctrl`, `shift`, `alt`, `cmd` or `super` held |
| `isc upload PANE FILE…` | Sends files to the pane's machine and types each path at the cursor, as if you'd dragged the file in |
| `isc ask PANE TEXT` | Types a line, presses Enter, waits and prints only the answer |
| `isc capture PANE` | Prints the end of the screen and scrollback as text |
| `isc history PANE` | Prints the commands run in the pane |

**Uploading.** Nothing is pressed after the paths, which are separated by a space and
quoted when they hold a space or a sign the shell reads. `isc upload` prints where each
file was kept.

- Files go in `.insensical-uploads` in the pane's directory, made on first use with a
  `.gitignore` that keeps it out of git.

Good to know: files are limited to 50 MB, and a file keeps its name without replacing
another.

**Asking.** What `isc ask` waits for depends on the pane:

- At a shell that marks its commands, it waits until the command ends, prints the end of
  its output with a closing line as `isc capture --last` does, and exits with the command's
  status: `isc ask build 'make test'`.
- Anywhere else, such as a database prompt, it waits until the pane has printed nothing
  for a second (`--quiet SECONDS` changes that) and prints what came after the line: at most
  4 KB with the end kept, and a closing line with a token. `isc ask psql '\dt'`. An answer
  that pauses for longer is cut short.

`isc ask --timeout SECONDS` gives up with status 124. `--anyway` types into a busy pane.

**Capturing.** `isc capture PANE` prints nothing your terminal would act on: a control
character a program left there is printed as an escape like `\u{1b}`. Trailing spaces and
trailing empty lines are left out. It prints at most 4 KB, the end kept, and always closes
with a line starting `--`: how many earlier lines were left out, if any, and a token to
read on from, as in `-- 1980 earlier lines left out; --since 3fa2c1d0`.

| Option | Prints |
|---|---|
| `--all` | All of it |
| `--screen` | Only what's in view |
| `--tail N` | Only the last `N` lines |
| `--last [N]` | Only what the latest command printed, without its prompt or command line. The closing line says how it ended (`exit 1`, or `running`). `--last 2` is the one before. It goes by the marks your shell writes around each command (see [Commands that fail, and short commands](#commands-that-fail-and-short-commands)) and says so when the shell writes none. The daemon remembers a pane's last 100 commands and holds the output of the last 20, so `--last 21` says it's no longer held |
| `--grep PATTERN` | Only lines matching a regular expression. `--context N` adds `N` lines around each, with an empty line between groups. With no match it prints only the closing line and exits with 1 |
| `--since TOKEN` | Only what came after the read whose closing line gave the token: the line the cursor was on then, and every line after it. Reading a log again this way prints nothing twice. A pane keeps the places of its last 8 reads. If the place is no longer held (it scrolled out, the daemon restarted, or 8 later reads were made), all of it is printed and the closing line says so |
| `--vt` | All of it with colours and styles as escape sequences, and no others. It takes none of the other options. The title, links, palette, modes and cursor position are never printed |

These combine: `isc capture 4 --last --grep error --context 2`. `--since` can't be used
with `--last` or `--screen`.

**What was run in a pane.** `isc history PANE` prints the commands run there, newest last,
one line each: when it started, how long it ran, how it ended (`exit 1`, or `running`),
the directory (`~` for home), and what was typed at the prompt.

```text
14:02:11  1.2s  exit 0  ~/code/api  cargo fmt
14:02:20  38s  exit 101  ~/code/api  cargo test
14:03:05  4m12s  running  ~/code/api  npm run dev
```

- It prints at most 4 KB, the newest kept, then a line saying how many earlier ones were
  left out. `--all` prints every one the daemon remembers (the last 100). `--json` prints
  them with times in milliseconds since 1970.

### Waiting

`isc wait` pauses a script until a pane is ready.

```sh
isc wait PANE [--until STATE]… [--timeout SECONDS]
```

- It succeeds when the pane reaches a state you name with `--until` (`idle`, `running`,
  `waiting`, `done`, `failed`, `closed`), and prints which.
- Without `--until` it waits for `done`, `failed`, `waiting` or `closed`.
- It exits with 124 when time runs out. The time can be a fraction of a second. `inf` is
  refused.

```sh
pane=$(isc run -- claude -p "fix the failing test")
isc wait "${pane#pane:}" --timeout 1800 && isc capture "${pane#pane:}" | tail -20
```

Or wait for one specific thing. Give one of these, and none with `--until`. Each fails if
the pane closes first and exits with 124 on timeout.

| Option | Waits for |
|---|---|
| `--output PATTERN` | A line on the screen or in the history that matches a regular expression, and prints that line. It succeeds at once if the line was printed before you asked. It reads line by line, so a line the pane's width broke in two is two lines: wait for a shorter piece. Colours don't matter |
| `--prompt` | The command running at the shell to end, by your shell's marks. It prints the end of the output and a closing line as `isc capture --last` does, and exits with the command's status. With nothing running, it does that at once for the last command. A shell that writes no marks is an error |
| `--quiet SECONDS` | The pane to print nothing for that long. Prints nothing |
| `--address` | The pane to show an address on this machine with a port, as a server prints where it listens (`localhost:5173`, `127.0.0.1:8000`, `0.0.0.0:3000`, `[::1]:4000`). It prints the last one shown, as `http://localhost:5173/`. One printed before you asked counts |

```sh
pane=$(isc run -- npm run dev)
isc wait "${pane#pane:}" --output "ready in" --timeout 60 && xdg-open http://localhost:5173
```

**Agents.** These start an agent and wait for it. See section 11.

```sh
isc agent start claude|codex|gemini TASK [--split right|down | --tab] [--front] [--cwd DIR] [--name NAME] [--json]
isc agent start claude|codex|gemini QUESTION --quiet [--cwd DIR] [--timeout SECONDS]
isc agent wait PANE [--timeout SECONDS] [--all]
isc agent reply PANE [--all]
isc agent tell PANE MESSAGE
isc agent answer PANE yes|no|TEXT
```

- `isc agent start` opens the agent in a new tab, or beside your pane with `--split`, with
  TASK as its first prompt, and prints `pane:N`. The keyboard stays where it is unless you
  give `--front`. It refuses an agent whose hooks aren't set up, and a `--name` that an open
  agent pane already has.
- With `--quiet` it runs the agent's own one-question mode on this machine, with no pane,
  prints only the answer and exits with the agent's status.
- `isc agent wait` waits until the agent's turn ends, then prints `done` or `failed` with
  its whole reply, or `waiting` with its question. It waits 30 minutes unless you give
  `--timeout`, then exits with 124, saying the agent is still working. It fails sooner,
  saying why, if the agent's hooks have said nothing 30 seconds after it started.
- A reply is cut to its last 4 KB, with a closing line saying how many lines were left
  out, unless you say `--all`.
- `isc agent answer` prints the state the agent moved on to, and fails if it still asks 10
  seconds later.
- All of them take `--host`, except `--quiet`.

### Any request, as JSON

Everything the window does is a request to the daemon, and `isc api` sends one. Use it for
what the commands don't cover.

```sh
isc api '"State"'                                  # the whole state, as `isc ls --json` gives it
isc api '{"Mute":{"pane":3,"on":true}}'
isc api '{"Float":{"pane":3,"rect":{"x":0.5,"y":0.1,"w":0.45,"h":0.8}}}'
echo '{"Zoom":{"tab":2,"on":true}}' | isc api
```

- A request is one JSON value: a name alone if it takes nothing, or an object with the
  name as its key. The answer is JSON: `"Done"`, `{"Pane":7}`, `{"State":{…}}`,
  `{"Text":"…"}`.
- What the window asks of the daemon's machine is here too: `"Clients"`,
  `{"List":{"dir":"~/src","deep":false}}` for a directory's names, `"Hooks"` and
  `"Settings"`.

**To see what's available, ask `isc api`.** It sends nothing and starts no daemon.

```sh
isc api                 # every request
isc api Float           # what one holds
isc api --json          # the same list, as JSON
isc api Float --json
```

With no request it lists every request, one to a line: its name, the right it needs, and
what it does. `isc api NAME` describes one. `NAME` is the request's name alone, without
quotes; capitals don't matter, and `hide-floats` is `HideFloats`.

```
Float: Lifts a pane out of the layout to float, or moves one that floats. The rectangle is in fractions of the tab.
Needs the right to arrange.

  pane  a whole number
  rect  {x: a number, y: a number, w: a number, h: a number}

Example:  isc api '{"Float":{"pane":3,"rect":{"x":0.5,"y":0.1,"w":0.45,"h":0.8}}}'
Answers:  "Done"
```

- `--json` gives the same as data: `name`, `needs`, `says`, `fields` (each with `name`,
  `is`, `optional` and `defaulted`), `example` and `answers`.
- `isc api '"?"'` fails and lists the names. So does piping in an empty standard input.

### Events

`isc events` prints what happens, one JSON object per line. Each has an `event` and
usually a `pane`. Pipe it into a script to react to your panes.

| `event` | Other fields |
|---|---|
| `opened`, `status` | `status`, `note`, `progress`, `program`, `kind` (`permission`, `question`, `auth` or null), `app` |
| `kind` | The same fields. What a waiting pane waits for changed, and its `status`, `note` and `progress` didn't |
| `closed` | |
| `focus` | `pane` is null when no pane has the keyboard |
| `bell` | |
| `notification` | `title`, `body` |
| `activate` | You picked the pane from a notification |
| `did` | `did` (what was asked, such as `mute` or `close`), `revision`, and `by`: the `client`'s number, its `kind`, its `name` and its `pane`. It has no `pane` of its own |

`did` is printed after a client changed something: opened, closed, moved, named, muted,
put to sleep, changed a daemon setting, or stopped it. It says who:

```json
{"event":"did","did":"mute","revision":41,"by":{"client":12,"kind":"command-line","name":"desk","pane":null}}
```

`by.pane` is the pane the command was run in, when the daemon could check that, and null
otherwise. Looking, typing and refused requests aren't reported.



`isc events` ends if you stop reading for long enough; run it again.

### Who is connected

`isc clients` shows everything connected to the daemon, so you can see who can do what.

```
client:12  window  on desk  0.3.0  look,type,arrange,manage  pid 4121  looking at pane:3  for 512s
client:19  program  on desk  0.3.0  look,type,arrange,manage  pid 9120  in pane:7  for 0s
```

Each line has the client's number, its kind (`window`, `terminal` for the interface in a
terminal, `command-line`, `program` for a command run inside a pane, `web` or `other`),
the machine it runs on, its version, its rights, the process at the other end of the
socket, the pane it's looking at, and how long it's been connected. A window makes a
connection for each pane it shows, so it has several lines.

- A guest's browser, paired with `isc hub device add --only`, shows what it's kept to:
  `only project 4` or `only pane:12`.
- `isc clients --json` prints all of it, with each client's `scope`: `{"project": 4}`,
  `{"pane": 12}`, or `null` for a client that sees everything.

**Putting a client off.** `isc disconnect 12` (or `client:12`) cuts that client's
connection and tells it you ended it. It needs the right to `manage`. With `--host NAME`
it acts on that machine's server.

- A browser paired through `isc hub` shows as `web`, one line for each connection. Putting
  those off doesn't unpair it. To keep it out, remove its device with `isc hub device rm`.

**Rights.** A connection has up to four:

| Right | Lets it |
|---|---|
| `look` | See projects, tabs and panes, each pane's screen and history, what programs use, who is connected, a directory's names, the daemon's settings (with `notify-url` and `notify-command` cut short), how the last notification sent outward went, and agents' hooks |
| `type` | Type and paste into a pane, set the size it's shown at, report a state or a notification for it, and count as you: what it looks at is seen |
| `arrange` | Open a pane with any command, close, split, move, float, focus, name, mute, put to sleep and wake, and choose icons |
| `manage` | Stop the daemon, change its settings, add or remove agents' hooks, and send a test notification outward |

You get all four unless you ask for fewer. `--rights`, before any command, asks for fewer:

```sh
isc --rights look ls
isc --rights look,type attach 3
```

- A command that needs a right the connection wasn't given is refused, and says which:
  "this connection was not given the right to arrange, which `close` needs. Its rights are:
  look". Typing without the right to type is dropped, silently.
- `look` reads everything: every screen and its history, with any password, key or token a
  program printed there. It runs and changes nothing.
- `type`, `arrange` and `manage` each let a client run programs as you. `type` types into
  your shell. `arrange` opens a pane with any command, so it needs no `type` to run one.
  `manage` changes what the daemon runs: your shell, the command it tells you with, agents'
  hooks. Give any of them only to what you'd give your account.

### Attaching from a terminal

`isc attach [PANE]` shows a pane in the terminal you run it in and sends what you type to
it. Press `Ctrl+]` to leave.


You reach a pane on another machine the same way:

```sh
ssh build-host isc attach 3
isc --host build attach 3
```

The second runs `isc` here and needs the machine named first: see [Other machines, over
ssh](#other-machines-over-ssh).

### Other machines, over ssh

Use the daemon on another machine as you use this one: list its panes, run things in them,
attach, or show the whole interface in your terminal. It all goes through your own `ssh`,
and the daemon there opens nothing to the network.

You need two things:

- **`ssh` to that machine asks nothing.** insensical runs `ssh` where nobody can answer,
  so use a key (loaded in your agent if it has a passphrase) and connect once by hand to
  accept the host's key. If `ssh build true` prints nothing and asks nothing, you're set.
- **`isc` is there, in the same version as here.** On Linux, `isc host put` puts it there
  (see "Putting insensical on a machine" below). Or install it there yourself with the
  one-line install, which takes a version: see [Installing](#installing). `~/.local/bin`,
  `/usr/local/bin` and Homebrew's `bin` are looked in even when a shell started by `ssh`
  doesn't.

Name the machine once, then check it:

```sh
isc host add build me@build.example.com
isc host check build
```

The second word is whatever you'd give `ssh`: a host, `user@host`, or a `Host` from your
`~/.ssh/config`. Your ssh configuration, keys, agent and jump hosts all apply.

| Command | Does |
|---|---|
| `isc host add NAME WHERE` | Names a machine. Takes `--color`, `--icon` and `--careful`, and `--no-node` (see below) |
| `isc host check NAME` | Meets the machine, prints four lines, changes nothing and starts nothing. It fails unless the machine is ready to use |
| `isc host ls [--json]` | Lists the machines you've named, with where `ssh` goes and what's set on each |
| `isc host set NAME` | Changes a named machine. `--color`, `--icon`, `--careful` set; `--no-color`, `--no-icon`, `--no-careful` clear; `--ssh` changes where `ssh` goes; `--name NEW` renames. Only what you give changes |
| `isc host rm NAME` | Forgets a machine. Nothing changes on it |
| `isc host put NAME` | Puts this version of `isc` on a Linux machine: see below |
| `isc host restart NAME` | Restarts the server there: see "What it doesn't do" |
| `isc host answer NAME` | Lets `ssh` ask you what it has to ask: see "When `ssh` has a question" |

A name may use ASCII letters and digits and `.` `-` `_` `+` `@` `:` `%` `[` `]`, with
`ssh://` before it if you like (`ssh://me@build:2222`). Anything else is refused, in the
settings file too: it's listed among the settings' problems and never tried. A `Host` in
your ssh configuration with other characters isn't offered; give it a second name there.

`isc host check` prints four lines:

```
✓ Reached build · 42 ms
✓ insensical 0.3.0 is there
✓ Its server is running
✓ 3 projects, 1 wants you
```

They say whether `ssh` got there and how long a round trip takes, whether insensical is
there in a version that works with this one, whether its server (the daemon) is running,
and what's open there. `✓` is a line that holds and `✗` one that doesn't, with what to do
about it underneath. Lines after a failure aren't tried and are marked `·`. `→` marks what
`isc host put` can put right. No server running yet isn't a failure: the first command
that needs one starts it.

The names are lines under `[hosts]` in your settings file, which you can write yourself:
`build = "me@build.example.com"`. A machine can also carry a colour, an icon and a mark
that it's one to be careful on:

```toml
build = { ssh = "me@build.example.com", color = "teal", icon = "server", careful = true }
```

- `color` is `#rrggbb` or one of `blue`, `green`, `orange`, `purple`, `pink`, `teal`,
  `gold`, `grey`, `rust` and `indigo`. With none set, a machine gets one of those ten from
  its name, the same every time.
- `icon` is one of `server`, `laptop`, `desktop`, `mini`, `nas`, `board`, `phone`,
  `container` and `cloud`. Any other word is left in the file, and the machine is shown as a
  server.
- A machine of your tailnet that `isc host add` found there also keeps its node: `build =
  { ssh = "me@build", node = "nABC123CNTRL" }`. See "A machine of your tailnet" below.

**Putting insensical on a machine.** If a Linux machine has no `isc`, or an older one,
`isc host put build` puts this version there. It works on x86-64 and arm64 Linux; on a Mac
it gives you the line to run there yourself.

- A server already running there is left running, on the older version. You're told what a
  restart would come to. `--restart-server` does it, as does a yes to a second question at a
  terminal, or `isc host restart build` afterwards. A server that can move to the new
  version keeps everything in its panes. One that can't is stopped and started, which ends
  what runs in its panes, and you're told how many panes first. A server asked to move that
  can't refuses and runs on.

Then put `--host` before any command:

```sh
isc --host build ls
isc --host build run -- make test
isc --host build attach 3
isc --host build
```

The last one is the whole interface, in this terminal, on that machine's projects.

- **From inside the interface you can go to another machine without leaving.** "Go to a
  machine" in the palette lists this computer and every machine you've named. Each is tried
  while you look and says `reached` with the version of `isc` there, or `not reached`; mark
  that one and the line below says why. `Enter` shows the marked machine's projects. While
  another machine is shown, its name stands at the left of the top row. One machine is shown
  at a time, and what runs on the others runs on. A machine that can't be reached leaves you
  where you were, with a notice.
- `INSENSICAL_HOST=build` in a shell does the same for every command you run there, except
  the ones that end something: `isc kill-server`, `isc close`, `isc tab close`, `isc project
  close` and `isc project sleep`. With only the variable set, those are refused and say
  which machine it names. Add `--host build` to the command to act there.
- What belongs to that machine is that machine's. The file finder lists its files. A new
  project's directory is a directory there, and `~` is your home there. A project's drawing
  is kept by the daemon there. `isc --host build integrate` sets up the agents there, and
  `isc --host build config set` changes the daemon's settings there: see [section
  11](#11-agents) and [section 15](#15-settings-and-themes).

**When the machine can't be reached.** A machine that has sent nothing for four seconds is
given up on. A command says "the link to the other machine was lost" and ends. Nothing
typed into the silence is delivered later: see [Other machines in the
window](#other-machines-in-the-window).

The interface dims your panes and puts a small card over them: "build is out of reach",
how long ago it was last heard once that's over ten seconds, and the key to leave. It
tries again after a second, then two, then four, up to every thirty seconds, with no key
pressed, and shows your panes again when the machine answers. Any key tries at once.
Nothing there was touched, and no daemon is started on this machine in its place.

If the machine won't let you in any more, because `ssh` would ask for a password or
doesn't know the machine's key, the interface tries once and stops, since a machine that
counts failed logins might lock you out. It says what `ssh` needs. Put that right, then
press a key.

**Typing on a far machine.** When the round trip takes 70 milliseconds or more, a letter
you type is drawn at once, a little fainter than the rest, and turns to its usual colour
when the program shows it. Your keys still go to the program at once, unchanged.

- `type-ahead = "off"` turns it off. `"on"` draws ahead over any link to another machine,
  however quick. A pane on this machine never has it.

**From a browser.** `isc hub` on this computer can carry a paired browser on to these
machines over the same `ssh`. Only a device you made with `--far` gets that; see "Your
other machines, through this computer" under [In a browser](#in-a-browser).

**Tailscale, Headscale and WireGuard** need nothing of their own. Give `ssh` the machine's
name or address on that network:

```sh
isc host add build me@build.tailnet-name.ts.net
isc host add nas me@100.64.0.7
isc host add office me@10.8.0.2
```

With Tailscale SSH there's no key to set up; the rule is the same, `ssh build true` must
ask nothing.

**A machine of your tailnet.** A name on a tailnet stops working when the machine or the
tailnet is renamed, or when this computer doesn't use the tailnet's DNS. The machine's
address goes on. So when you `isc host add` one of your tailnet's machines, its node's id
is written beside it, and it's dialled at its address:

```sh
isc host add build me@build
```

- `--no-node` keeps it as typed. So does a name your ssh configuration gives a machine,
  which leads where your configuration says. The window's `Add Remote Host` dialog keeps the
  node of a machine you choose under `Tailscale` or type by its tailnet name, and changing a
  machine's address in the settings sheet looks for its node as `isc host set build --ssh
  me@build` does.

**When `ssh` has a question.** A link is made by `ssh` told to ask nothing, so each thing
it would have asked is a machine that isn't reached. Each has its own words:

| It says | What happened | What to do |
|---|---|---|
| `ssh hasn't met build before` | `ssh` has no host key for it yet | `isc host answer build`, and say yes once you've compared the fingerprint |
| `build's host key is not the one ssh has for it` | The key changed. A machine set up again does this, and so does another machine standing in its place | Find out why first. When you're sure, `ssh-keygen -R build` forgets the old key |
| `ssh needs your key or password for build` | It wants a key, a passphrase or a password | `isc host answer build` shows what it asks. A key in your agent (`ssh-add`) or on the machine (`ssh-copy-id`) lets it in without asking |
| `Tailscale wants you to sign in again before it lets ssh through` | The tailnet's rules ask for a fresh sign-in | `isc host answer build` shows the link to sign in at |
| `Tailscale's rules don't let you in as root on build` | The tailnet's access rules refuse that user, or you | Whoever looks after the tailnet changes its rules |
| `build is off` | Tailscale says it isn't online | Turn it on, or start Tailscale there |
| `build did not answer in time`, `build could not be found`, `build refused the connection`, `build cannot be reached from this network` | As they say | The line under each says what to check |

`isc host check` prints `ssh`'s own last line under these, after `` `ssh` said: ``.

`isc host answer build` runs your own `ssh` to the machine at your terminal, free to ask,
running nothing there but `true`. You see `ssh`'s real question and answer it yourself;
insensical doesn't read, keep or pass on what you type. Then it meets the machine and
prints the four lines. It needs a terminal and succeeds only if the machine was then
reached. For a machine you haven't named, give it where `ssh` goes: `isc host answer
me@build`. In the window, the `Answer` button runs it for you in a new tab on this
computer, and in the interface in a terminal `a` does in "Go to a machine".


**What it doesn't do:**

- **A link can't ask you anything.** A host that wants a password, a passphrase or a yes
  to its key is refused, with what to do about it.
- **The two versions must match** (the same protocol and terminal engine, which `isc
  version` prints). If they don't, you're told which machine has which. `isc host put` puts
  this version on the other machine. If the other machine's is newer, update this one.
- **`--host` is for `isc` and the interface in a terminal.** The window shows every
  machine you've named at once: see [Other machines in the
  window](#other-machines-in-the-window).
- **Some commands are about this machine only**, and say so if you give them a host:
  `daemon`, `proxy`, `restart-server` and `host`. To restart the server elsewhere, use `isc
  host restart build`: it meets the machine, tells you what a restart comes to, and asks. If
  the server there already runs the installed insensical, it says so and restarts nothing.
  Don't run `ssh build isc restart-server`: a shell started by `ssh` often can't find the
  server. `isc --host build kill-server` asks the daemon there to stop, and can't signal one
  that doesn't answer.
- **`config` reaches the daemon's settings there and no others.** A setting of the
  interface, such as `theme`, is read from this machine's file, and `isc --host build config
  set theme paper` is refused. Leave `--host` out for those.
- **A path you click there is the other machine's.** Only web addresses are followed on
  this computer. With `nvim`, `vim`, `hx` or `nano` in `open-files-with`, the click opens
  the file in a pane on that machine; otherwise it copies the path. See [What a click on a
  path does](#web-addresses-and-paths).
- `INSENSICAL_SSH` names a program to run in place of `ssh`, given the same arguments.
  `INSENSICAL_TAILSCALE` does the same for `tailscale`.

### In a browser

`isc hub` lets you use insensical from a phone, a tablet, or a laptop with nothing
installed. On a phone it opens on what needs you, with each agent's question and keys to
answer it, and it can notify you when you're away. On a wide screen it's the sidebar and
the tab in front. Anywhere, you can open a pane, read it, scroll back, and type into it.

It's a separate program that runs only when you ask for it (you start it, join a hub, or
turn on autostart), and a client of the daemon like the window. The daemon still listens
on nothing but its socket.

### Starting isc hub

The guide sets everything up one step at a time:

```
isc hub setup
```

1. **Hub or join.** Choose whether this computer is the hub, where your browsers and
   phones go, or joins a hub you already have. A computer that's already joined says which
   hub and skips to step 3. `isc hub leave` makes it a hub of its own again.
2. **Reach** (the hub). Choose how you'll reach this computer: Tailscale, a name of your
   own behind your proxy, an ssh tunnel, or this computer only. A proxy name must be
   `https://`, since a browser can't pair over plain `http://`.

   **Join** (a joined computer). Paste the line from the hub and press `Enter`. **Invite a
   Machine** on the hub's Hub page gives it, and so does `isc hub invite NAME`. The guide
   hands it to `isc hub join` and nowhere else, so it stays out of your shell history and
   scrollback. A joined computer dials the hub, so there's no Reach step.
3. **Keep running.** Where systemd runs for your user, choose whether the hub starts after
   a restart. **Yes** is `isc hub autostart on`.
4. **Add a device.** **Yes** opens the form that asks what to call your phone and shows
   the code to scan. On a joined computer, the code pairs at the hub's address.
5. **Done.** It shows the address, who's paired, and the plain commands that do the same
   without the guide. The summary stays on your screen. The line you pasted isn't in it.

`Enter` goes on, `Shift`+`Tab` goes back, and `Esc` leaves. Nothing changes until you
choose a way in step 2 or press `Enter` on the line. The guide needs a terminal; in a
script, use the commands below.

To start the hub yourself:

```
isc hub
```

It listens at `http://127.0.0.1:7433`, this machine only. `--port 8000` picks another
port. It runs until `Ctrl`+`C` or `isc hub stop`, which also stops the window starting it
again. `isc hub status` says whether it's running and where; `--json` adds a `tailscale`
object with `state`, `address` and `serves_port`.

**Open it in your browser on this computer**

```
isc hub open
```

This opens the hub in your default browser (`xdg-open` on Linux, `open` on a Mac), paired
as the device `This computer` with every right. It stays signed in until you remove the
device. Run it again for another address for the same device; removing the device signs
out every browser paired this way.

The address works once, for 10 minutes, and only for a request from this machine that asks
for `127.0.0.1`, `localhost` or `[::1]`. From anywhere else it's refused and not used up,
so a leaked link is no use off this computer. A port forward of yours that ends here (`ssh
-L`) does pass, so keep the address to yourself.

- `--print` prints the address and how long it works, and opens nothing.
- `--json` prints `{"url": …, "code": …, "until": …}` and opens nothing. `until` is in
  seconds since the epoch.

A browser that opens `http://127.0.0.1:7433` unpaired sees the pairing screen, which tells
you to press Open in Browser under Settings > Hub in insensical. A code made there can
still be typed.

### Pairing a device

Pair a phone, a tablet, or a browser on another computer so it can use the hub. Nobody
else can until you do. A device pairs by a code.

**Pair your phone**

1. In the window, open Settings > Hub and press **Add a Device**. The Hub page of the
   settings in `isc tui` has the same.
2. Give the device a name, say what it may do, and press **Add**.
3. On the device, point its camera at the QR code, open the code's link, or type its
   letters (twelve groups of five) on the page's pairing screen.

From a shell:

```
isc hub device add --name phone
```

In a terminal you can leave the name out and get the same form. Anything on the command
line, such as `--rights look`, is where the form starts. With `--name`, with `--json`, or
with no terminal, it prints the code: the link on one line and the letters on the next.
`--json` gives `url`, `code` (the same link) and `typed`. `--url
https://desk.example.ts.net` prints the address and code for a name other than this
machine's own.

**Icon and colour.** A device has an icon (phone, tablet, laptop, desktop or browser) and
a colour (blue, green, orange, purple, pink, teal, gold, grey, rust or indigo). Add a
Device has an **Icon** row to pick both. Left alone, the icon is the one the device's
browser reports and the colour comes from its name. Your choice is never replaced by a
guess.

```
isc hub device add --name slate --icon tablet --color teal
isc hub device set slate --icon laptop
isc hub device set slate --no-icon --no-color
```

`--no-icon` and `--no-color` take a choice away. `isc hub device ls` shows both; `--json`
has `icon` and `color`, null where none was chosen.

**How many, and for how long.** A code pairs one device, within 10 minutes, unless you say
otherwise. In the window that's under **More options**.

```
isc hub device add --name family --uses 3 --expires 1h
isc hub device add --name desk --uses any --expires never
```

| Flag | Value |
|---|---|
| `--uses` | A number of devices, or `any` |
| `--expires` | How long the code pairs: minutes, hours or days (`10m`, `1h`, `7d`), or `never` |

Once the code is used up or its time is up, it pairs nothing more, and neither does a copy
of its link or letters. The device that paired stays paired. Five failed tries close a
code for good. `isc hub device ls --json` gives each device's `code` as `open`, `used`,
`closed` or `expired`.

**When a program in a pane asks.** An agent in a pane could otherwise make itself a phone
that types anywhere and keep it after it's gone. So when one of these runs inside an
insensical pane, you're asked first:

- `isc hub device add`, `isc hub device pair-again`, `isc hub device set --far`
- `isc hub open`
- the device step of `isc hub setup`

The window, or `isc tui` in a terminal of its own, shows the question over everything,
naming the pane and its program. The command waits.

- **Allow**, or `A`, lets it go on. For the first second neither choice counts, so a key
  meant for a pane can't allow anything.
- **Deny**, `Enter` or `Esc` stops it. So does not answering within two minutes.
- Only the window and an `isc tui` outside insensical's panes can answer. A program in a
  pane can't, and a browser never is asked.
- With no window open, you're told on the desktop (a notification, and a mark on the tray
  icon or the menu bar), and clicking either opens the question.
- If there's nothing to show or tell you, the command is refused at once and says to use
  **Add a Device** on the Hub page, or a terminal outside insensical.

**Typing the code on the page.** Case, spaces and dashes don't matter, and `I`, `L` and
`O` are read as `1`, `1` and `0`. The field groups letters in fives and counts them.
Nothing is sent until the code checks, so a typo uses nothing up.

**Good to know**

- **A browser keeps its own key.** It's known by that key alone, and stays paired through
  restarts until you press **Forget This Browser** on the page or remove its device. Nothing
  lapses by itself.
- **Clearing site data loses the key**, and the browser needs a new code. A browser is
  paired at one address: another name or port is a different browser.
- **A plain-HTTP address can't pair.** A browser gives keys only to a page over HTTPS, or
  at `localhost` or `127.0.0.1`. Use an `https://` address (Tailscale gives one) or an ssh
  tunnel.
- **If another server answers later at the same address with a different key**, the page
  says so and stops. This guards against a swapped server, not against the one that served
  the page.
- **One device can pair several browsers.** `isc hub device ls` shows each device, what it
  may do, when it was last used, how its code stands, when the device ends, and its paired
  browsers.
- **Scan the Code** on the page opens the camera, where the browser may use one.
- **A home-screen app is a browser of its own.** On an iPhone, the page you add to the
  home screen doesn't share Safari's pairing. Give it a code of its own, and scan with
  **Scan the Code** in the app, not the Camera app, which would open Safari and use the code
  up.
- **A paired browser opened at a link with a code** says it's already paired. The code
  isn't used and still pairs the device it was meant for.

### What a device may do

A device carries rights, the same four the daemon gives every client: `look`, `type`,
`arrange` and `manage`. See [Who is connected](#who-is-connected). Left alone, a device
has all four.

> [!WARNING]
> `type`, `arrange` or `manage` is enough to run programs as you, so a device with one is
> a key to your account on that machine. `manage` is the one to think about most: it can
> change what the server runs to tell you (`notify-command`). Give **Can do everything**, or
> `manage`, only to devices that are yours.

```
isc hub device add --name hallway-screen --rights look
isc hub device add --name guest --rights look,type --expires 1h --uses 2
```

The second code pairs up to two browsers in the next hour. Rights are set on the device:
`isc hub --rights look` is refused.

A device with only `look` can read every pane and its history, and can't type, resize or
change anything. The daemon refuses, so a page somebody wrote themselves gets no further.
It doesn't count as you looking: a pane open in that browser still notifies you. See [Who
is connected](#who-is-connected). `isc clients` lists a browser as `web`, under its
device's name.

**Make a device that ends by itself.** To pair a co-worker's or client's browser for a
while without having to remove it:

```
isc hub device add --name client --rights look --stays 1d
```

`--stays` is how long the device stays paired, counted from when you add it: minutes,
hours or days (`30m`, `1h`, `7d`), or `always`, the default. When time is up, every
browser paired as the device is signed out and the device leaves `isc hub device ls`.
Until then the list says "ends in 40m", and `--json` has `stays_until` in seconds since
the epoch, or `null`. `--expires` is only the code: a browser that paired stays paired
after it expires.

**Remove or re-pair a device**

- `isc hub device rm phone` removes a device at once, and every browser paired with it
  closes. Do this for a phone you've lost.
- `isc hub device pair-again phone` gives a device a new code and link without removing
  it, for a phone you've replaced or a browser that forgot its pairing. `--json` is for
  scripts, and `--expires` sets how long the code works (10 minutes if left out). The device
  keeps its name, icon, colour, rights and how long it stays. The browser that pairs takes
  the place of the ones it had, which stay paired until then. The window's and `isc tui`'s
  device lists (`p`) and the page's list have **Pair Again**. A guest is ended and shared
  again instead.

A device sees every terminal on this computer unless you make it with `--only`, as in the
next section.

### Sharing one project or one terminal

Show a co-worker or client one thing and nothing else.

**Share one project**

1. Press the eye on the project's row in the sidebar, or in the terminal's header. Or
   choose "Share this project" or "Share this terminal" in the palette.
2. Set the options below.
3. Press `Make the Link` and send them the link. It comes with a QR code and a button that
   copies it.

They open it in a browser and install nothing. The link works once, for a day, or until
the share ends if that's sooner.

| Option | Default | Other choices |
|---|---|---|
| **What they see** | Fixed: only this project and anything you open in it, or only this terminal wherever you move it | |
| **They can** | `Look` | `Look and type`: shows a warning, and the link can't be made until you press `Let Them Type`. Typing in a terminal here is typing as you |
| **Where they are** | `On my network` | `Anywhere on the internet`, see [Sharing with someone who isn't on your network](#sharing-with-someone-who-isnt-on-your-network) |
| **Ends** | `1 day` | `1 hour`, `1 week`, `When I end it`. When time is up they're signed out |

If "Open in a browser or on your phone" is off, the button reads `Turn On and Make the
Link` and turns it on first. If the link's address opens only on this computer, the dialog
says so: someone elsewhere needs Tailscale turned on, on the Hub page.

While something is shared, its sidebar row and the header of each terminal a guest sees
carry a small sign: an eye where guests look, a keyboard where one can type. The same
button lists who it's shared with. **End** signs one out at once and stops their link, and
`Share With Someone Else` makes another link. Settings lists every guest on the Hub page.

**Good to know**

- You can share only what's on this computer, not a project on another machine.
- The window names the guest after what's shared: `api guest`, then `api guest 2`.

### Sharing with someone who isn't on your network

A client outside your network or tailnet can't open that link. In the dialog, set **Where
they are** to `Anywhere on the internet`. Then **They can** is `Look` only, and **Ends**
is `1 hour`, `1 day` or `1 week`: a public link always ends, a week at most.

Press `Open a Public Link`. It has `isc hub` answer to this computer's Tailscale name at
its guest door (`isc hub --guest-name`, which restarts it and drops no paired device),
makes the guest (`isc hub device add --public`), and opens the address (`isc hub public
on`). Then it shows the public link and its QR code. Whoever opens the link first gets in,
and it works once. Don't post it where others can read it.

- **What it exposes.** The address is Tailscale's, made by Tailscale Funnel on port 8443
  of this computer's Tailscale name. Anyone on the internet can reach its sign-in page. Only
  someone with a link you made gets past it, and only to look at what you shared. Nothing of
  insensical's own listens on the internet.
- **If it can't be opened, the dialog says why** and offers the fix: `How to Install`,
  `Sign In`, or `Turn On Funnel`. If Tailscale already serves something else on port 8443,
  it's left as it is. If a step fails, the guest it made is removed.
- **The window shows when a public link is open.** A `Public link open` sign stands in
  amber in the bar, at the sidebar's foot, or over the window's corner. Press it to see
  what's shared, each with `End`.
- **To close it**, press the sign and then `Close Public Link`, or press `Close` on the
  `Public link` row of the Hub page. That's `isc hub public off`: every public guest is
  signed out and Tailscale no longer leads to this computer. Ending the last public guest
  closes it too.

**Share from the command line**

```
isc hub device add --name sam --only project:api --stays 1d
isc hub device add --name kim --only pane:12 --rights look,type --stays 1h
```

That device is a guest. Send them the address it prints. You can also make a guest on the
page: **Settings**, **Your devices**, **Add a Device**, and under **What it sees** choose
**One project** or **One terminal**. See [Managing isc hub from the
page](#managing-isc-hub-from-the-page). `isc hub device ls` shows what each device sees:
`everything`, `only project 4 (api when made)` or `only pane 12 (claude in api when
made)`. `--json` has `scope`, `sees` and `reset`.

**What a guest sees.** With `project:api`, that project: its tabs and terminals, and
anything you open in it later. A terminal you move out is gone from their page. With
`pane:12`, that one terminal wherever you move it. Nothing else on this computer is listed
or shown: no other project or terminal, not who else is connected, not your settings, not
your home directory. They do see this computer's name. They can scroll back through
everything the terminal has shown, including what was printed before you shared it.

**What a guest can do.** Look, and that's all unless you say `--rights look,type`.
`arrange` and `manage` are refused with `--only`: a guest never opens, closes, moves or
renames anything, and never changes a setting.

**What a guest never gets.** Notifications, the Settings screen, the list of devices, and
making a device. A guest looking at a terminal doesn't mark it seen, so you're still
notified.

> [!WARNING]
> A guest who can type has your account. Someone who can type in a terminal can read your
> files and run anything as you, here and wherever this computer can reach, including `isc`
> to see every other terminal. `--only` keeps their page to one project; it doesn't confine
> what they type. Share typing only with someone you'd hand your keyboard to. A guest who
> can only look runs nothing, so for them "only this terminal" is exact.

**Good to know**

- **A share is to the number you named.** `project:api` is looked up when you make the
  device, and the share follows that project's number. Rename the project and the guest sees
  the new name; call another project `api` later and the guest doesn't move to it. If two
  projects are called `api`, `isc hub device add` refuses and gives their numbers: say
  `--only project:4`. A terminal is always by its number, as `isc ls` shows it. The project
  or terminal has to be open when you make the device.
- **Give it an end** with `--stays 1h`, `1d` or `7d`. Without it a guest stays until you
  remove it.
- **To end it now**, `isc hub device rm sam`. If you close the terminal or project
  instead, the guest has nothing left to see, and the device stays paired to nothing until
  it ends or you remove it.
- **Pane numbers come from one counter**, so a new terminal's number tells a guest roughly
  how many terminals you've opened in all. Never what they are.
- **A share ends if insensical's saved state is lost or can't be read.** Numbers start
  again from one, so every guest device then stops working and shows as `ended` in `isc hub
  device ls`. Remove it and make a new one. A device made by an earlier version of `isc hub`
  ends this way too.
- **A guest's page** has a bar across the top saying what they are and when the share
  ends. Under it is what you shared and nothing else, with no Home, Settings, bell or
  notifications. They see each terminal at the size and colours you have it, and can still
  scroll, find, copy, and change the letter size on their own screen. **Leave** is in the
  menu at the end of the bar.
- **When it's over**, because you closed what they saw, removed the device, or time is up,
  their page says so and to ask for a new link.

**Share from the command line to someone outside your network.** Add `--public`, then open
the address:

```
isc hub device add --name sam --only project:api --stays 1d --public
isc hub public on
```

This puts an address on the public internet that shows that one project or terminal to
whoever has the link. Read these limits first.

- **It's look only.** `--rights look,type` is refused with `--public`. Someone who types
  in a terminal is using your account, and that's never left one link away from the
  internet. To type together, share on your own network.
- **It ends within a week.** `--stays` is required and is 7 days at most. The link pairs
  one browser, for ten minutes, unless you say `--uses 2` or `--expires 1h`. `--uses any`
  and `--expires never` are refused.
- **Anyone can reach the sign-in page.** The address shows up in public lists of
  certificates and gets scanned. A stranger gets the page and nothing else. Only the link
  gets in, and only to what you shared.
- **The address is Tailscale's.** `isc hub public on` asks your own Tailscale to open it
  with Funnel, at `https://your-machine.your-tailnet.ts.net:8443`. Nothing of insensical's
  own listens on the internet, and nobody of ours is in between. Your own devices don't work
  at that address at all: a lost phone or leaked code of yours is worth nothing there.
- **Close it with one command.** `isc hub public off` removes every public device, signs
  them out, and takes the address away. `isc hub device rm sam` ends one. When the last one
  ends the address stays with Tailscale until you run `off`. If Tailscale won't close it,
  `off` still removes the devices, says what Tailscale said, and fails, so you know to run
  it again.

Start `isc hub` with the name Tailscale gives this computer, as `isc hub --guest-name
desk.example.ts.net`. `isc hub public on` tells you if it's missing, and gives the link to
turn Funnel on if your tailnet doesn't allow it yet. Without Tailscale you can point a
tunnel of your own at `127.0.0.1:7434`, but whoever runs a tunnel that ends the encryption
reads everything through it, the link included. `isc hub device ls` marks these devices
`public`.

**Check a public address.** `isc hub public status` says whether the address is open and
for which devices, open with nothing shared (close it with `isc hub public off`), closed,
not known because Tailscale isn't answering, or held under a name this computer no longer
has. It succeeds when the address is open and fails otherwise, so a script can ask.
`--json` always succeeds and has `state`, `open` (null when Tailscale can't be asked),
`address`, `port`, `leads_to`, `devices` (each with `name` and `ends`), `record` and
`record_without_device`. `isc hub status --json` carries the same under `door`.

### Your other machines, through this computer

Let a device reach your other machines too: the ones `isc host ls` lists on the computer
`isc hub` runs on.

```
isc hub device add --name phone --far
isc hub device set phone --far
isc hub device set phone --no-far
```

The second and third give it to a device that's already paired, and take it back.

- It's all of your machines or none.
- `isc hub` reaches a machine the way the window does: your own `ssh`, running `isc proxy`
  there. The other machine needs `isc` and nothing else, and nothing new listens on it. The
  `ssh` forwards nothing, whatever your `~/.ssh/config` says.
- The device has the same rights there as here.
- Only a device that may type can have `--far`. A device that only looks can't.
- **A guest never can.** `--far` with `--only` or `--public` is refused, and a guest's
  browser isn't told you have other machines.
- `--no-far`, removing the device, its time running out, or taking the machine out of the
  settings closes what the device has open there.
- `isc hub device ls` marks these devices `and your other machines`.
- `ssh` runs only while a browser has a terminal of that machine open, or a device with
  `--far` has notifications on: see [Notifications on your
  phone](#notifications-on-your-phone).
- A machine with another version of insensical can't be opened. Install the same version
  on both.
- A machine's name must be letters, digits, `-`, `_` and `.`, as `isc host add` takes it.
  Others aren't offered to a browser.

**What the page shows.** On such a device the page lists your other machines too:

- **Each machine is a heading over its projects.** This computer is first, then the others
  in the order your settings have them. A device without `--far`, and a guest, see nothing
  about other machines.
- **A machine is folded until you open it.** Press the arrow on its heading. That's what
  connects to it, so a machine you never open costs nothing. The browser remembers which you
  had open.
- **Home and the counts cover every open machine.** Each card says which machine it's on,
  and an answer goes to that machine. A folded machine isn't counted.
- **A terminal on another machine has a chip** in its header with the machine's icon and
  name. No chip means this computer. What you type goes only to the terminal you're looking
  at.
- **An address names the machine.** `?pane=3&at=atlas` is terminal 3 on `atlas`. A
  notification about another machine's terminal opens that machine and lands on the card.
- **Devices**, **Add a Device** and the settings on the page are this computer's. You
  can't share, make or move a project or terminal on another machine from the page.

**When a machine has a problem**

- **Out of reach.** The machine's heading says so, its terminals dim, and what you type
  there isn't sent. This computer's terminals carry on. The page keeps trying and the
  machine comes back by itself.
- **Needs a password.** `ssh` would ask for a password or passphrase, so nothing retries
  by itself. Fix it as "When the machine can't be reached" says under [Other machines, over
  ssh](#other-machines-over-ssh), then press **Try again** under the machine's heading.
- **Another version.** Install the same version on both machines, then press **Try again**.
- **Closed: it sent what no machine sends.** That machine sent something no insensical
  sends, or listed more than 4,096 terminals, tabs or projects. The page closes it and shows
  nothing of it, so it can't pass for another of your machines. Fold it and open it again to
  try once more. If it says so again, don't trust that machine until you know why.
- What `ssh` said is shown as one line of at most 300 characters. It can include your user
  name and the machine's address.

### Managing isc hub from the page

From a phone you can do what `isc hub device`, `isc hub push`, `isc notify` and `isc
clients` do. A browser paired as a device that may do everything, or at least change
settings, has **Settings** in the menu under its letter. Other browsers aren't shown it,
and the server refuses if they ask. It covers the computer `isc hub` runs on.

- **This computer** shows its name, that insensical is running, and its version.
- **Your devices** begins with **Add a Device**, then lists each device.
- **Notifications on your devices** lists the browsers that asked for them. **Send a
  Test** says for each whether its push service took it.
- **Alerts when away** sets where the server tells you when nobody is looking: ntfy (a
  server, a topic, and a token if your server needs one) or another web address. It has a
  switch for only when you're away, a test, and under More options the rest, including a
  command.
- **Connected now** says what is connected and what it may do. **Disconnect** cuts one
  off, as `isc disconnect` does. This page's own row has no button.

Nothing else of the server's settings can be changed from the page. The page is never
given a token or password back, only its last few characters, and **Clear** takes it out.
An address is shown shortened, to its host and the first part of its path, and can't be
edited as shown: **Change** empties the field so you can type the whole address.

**Add a device from the page**

1. Press **Add a Device**.
2. Enter a name and choose what it may do: everything, look and type, or only look.
3. Choose how long it **Stays paired**: until you remove it, 1 hour, 1 day or 7 days.
   Under More options, set how many devices the code pairs and for how long.
4. Press **Add**. You get a code to scan and the link, which is this page's own address,
   so a phone on the same network or tailnet can open it.

A device you add sees every terminal on this computer unless you change **What it sees**
to **One project** or **One terminal** and pick which. The device is then a guest: see
[Sharing one project or one terminal](#sharing-one-project-or-one-terminal). A guest can
**Look** or **Look and type**. It starts on Look, ending in 1 day. **Add** stays dim for
Look and type until you press **Let Them Type**.

Limits when adding from a page:

- You can't make a device that may do more than this browser may.
- No more than six a minute in all, and three from one browser.
- A device never stays paired longer than the browser that made it.
- **Everything** is for devices that are yours: a device that may manage can run programs
  as you.

**The device list.** The device this page is paired as comes first, marked "this device".
Each row shows what it may do (for a guest, what it sees, marked "guest"), when it was
last used, how many browsers it paired, and how long its code or the device itself has
left. A device that ends is gone from the list when its time is up, and a page paired as
it is signed out.

**Pair Again** shows a new code for a device. **Remove** asks once and then ends it at
once, like `isc hub device rm`. On the device this page is paired as, it signs this page
out.

On these screens a choice of two shows both side by side: tap one, or use `Left` and
`Right`. A choice of more opens a list: tap it or press `Space`, `Enter` or an arrow key,
then pick, or press `Esc` to close it without choosing.

### Reaching isc hub from another device

`isc hub` listens on this machine only, so a phone needs a way in. `isc hub setup` offers
these ways and sets up the one you choose. Or use one you already trust.

**ssh**

```
ssh -L 7433:127.0.0.1:7433 you@this-machine
```

Then open `http://127.0.0.1:7433` on the device that ran `ssh`. It pairs with a code like
any other.

**Tailscale**

```
isc hub tailscale on
```

This starts `isc hub` if it isn't running, has it answer to this computer's Tailscale
name, and has Tailscale serve it on port 443 of that name, to the devices on your tailnet.
It prints the address, such as `https://desk.example.ts.net`. If no device is paired yet,
it ends with `isc hub device add --name phone`.

It also says what happens after a restart. With `insensical-hub.service` installed and on,
the service starts again when you log in; without it, nothing starts the hub again unless
you open the window. Either way it names `isc hub autostart on` to fix that, and never
changes lingering itself.

If Tailscale isn't installed, isn't signed in, or isn't answering, it says which and
changes nothing. If Tailscale already serves something else on port 443, it says what and
leaves it alone. The one exception is a mapping to a port that nothing answers on, such as
`tailscale serve 7443` typed while `isc hub` is on 7433: that gives a 502, so it's
replaced.

| Command | What it does |
|---|---|
| `isc hub tailscale on` | Sets it up, as above |
| `isc hub tailscale off` | Takes away the mapping `on` made, and nothing else of Tailscale's. A mapping you made by hand is left |
| `isc hub tailscale status` | Says whether Tailscale serves `isc hub` and at what address, or why it doesn't. Ends with 0 only when `isc hub` is served |

`--json` always ends with 0 and has `state` (`on`, `off`, `absent`, `signed out`,
`silent`, `taken` or `mismatch`), `address`, and `serves_port`. It's the same as the
Tailscale switch on the Hub page of the settings, and the two see each other's work.

To do it by hand, `tailscale serve 7433` gives the machine an `https` address inside your
tailnet. Start `isc hub` with that name, `isc hub --name desk.example.ts.net`, and add the
device with `--url` and that address.

**Start by itself**

```
isc hub autostart on
```

This does what this computer needs for `isc hub` to come back after a restart, and says
each thing it did. It does only what's missing, in this order:

1. Writes `insensical-hub.service` to `~/.config/systemd/user/` (or under
   `$XDG_CONFIG_HOME`), running the `isc` you ran. A service a package installed is never
   written over.
2. Switches the service on and starts it. A server running on its own is restarted from
   the service with the same names and port.
3. If your user's services don't run with nobody logged in, runs `loginctl enable-linger`
   for you. If you aren't allowed to, the service is still on and it tells you to run `sudo
   loginctl enable-linger NAME` to start with the computer.

Run again with everything in place, it changes nothing and says so.

| Command | What it does |
|---|---|
| `isc hub autostart off` | Switches the service off, and takes it away only if `autostart on` wrote it. A running server goes on running. Lingering is left as it is: `loginctl disable-linger` is yours to run |
| `isc hub autostart status` | Says whether it starts by itself, and when (at login or with the computer). Ends with 0 only if it does |

`--json` always ends with 0 and has `autostart`, `unit` (`ours`, `packaged` or `null`),
`enabled` and `linger` (`null` when the system doesn't say). This is systemd's: without
one for your user, such as on a Mac, nothing starts `isc hub` by itself. On a Mac, opening
insensical starts it again.

**Headscale, or a proxy of your own.** Point it at `127.0.0.1:7433`, with your own
certificate. It has to speak TLS to the browser, pass the `Host` and `Origin` headers on
as they came, and pass WebSockets. Start `isc hub` with `--name` and the name the browser
will ask for.

**A public address, for a guest.** `isc hub public on` has Tailscale Funnel open one, to a
second door that `isc hub` keeps at `127.0.0.1:7434` (`--guest-port` changes the port).
That door is on this machine only, always. It's open only while a device made with
`--public` exists, lets in only those devices, and they can only look. It answers to the
names you give with `--guest-name` and to `127.0.0.1`, not to `--name`'s names or
`localhost`. `--guest-port` can't be the port `isc hub` itself listens on. A key paired at
one door is nobody at the other. See [Sharing one project or one
terminal](#sharing-one-project-or-one-terminal).

**What the hub accepts.** `isc hub` answers only to names it knows are its own: this
machine's addresses, `localhost`, and each `--name` you give it. Asked by any other name
it gives nothing, not even the page. That keeps another site's page from reaching it under
a name of that site's making.

It won't listen anywhere else without encryption. `--listen 0.0.0.0` is refused unless you
also give `--tls-cert` and `--tls-key`, a certificate and its key as PEM files. With those
it speaks `https` only.

- Over `ssh -L`, the browser's address is plain `http` and `ssh` does the encrypting. Over
  Tailscale, Tailscale does.
- **Other accounts on this machine can reach the port**, where they can't reach the
  daemon's socket. Without a device's key they get the page and nothing else.

### On a phone: Home

**Home** shows what needs you right now, with every project under it.

At the top is a card for each pane that wants you, most urgent first: waiting, then
failed, then finished. To answer one:

1. Read the question in large type. Under it is the foot of the pane's real screen, live,
   so you can see the choices the agent drew. Tap it to open the pane.
2. Press a key under the card, or write an answer in the line.
3. The card says what was sent. When the agent goes back to work the card goes.

| Under the card | What it does |
|---|---|
| `Enter`, `y`, `n`, `1`, `2`, `3`, `Esc` | Each sends that key and nothing else. They're the same on every card, so read the pane's foot to see which key means what. |
| The line | Type or dictate, then press the arrow. It's sent as typed, then `Enter`. |
| Password field | Hides what you type, with `Enter` and `Esc`. |
| **Open the pane**, **Seen** | For a pane that only failed or finished. |

With nothing waiting, the top says **All quiet**.

Under the cards, each machine has a section with a card per project, lit if it needs you.
Tap a card to open the place most worth looking at: the pane that needs you, else the one
at work, else the pane you used last.

**Everything**, at the bottom, lists every project, tab and pane as the window's sidebar
does. The switch over its foot narrows it to **Activity**: only panes at work or with
something for you, kept quiet by the rules in "Showing only activity" under [The
sidebar](#the-sidebar). Your browser remembers the choice.

The number on the title, on **Home** and on the switch is how many panes need you: waiting
and failed. Finished panes are counted beside it, never in it. The page's tab and the
home-screen icon's badge show the same number.

### A pane in the browser

Tap a pane to open it at your screen's width. Tap it again for the keyboard.

- **Scroll back** by sliding down at the top. More history arrives each time you reach the
  top.
- **Resize the letters** by pinching.
- **Copy** by holding a finger on a word, dragging to select more, and pressing **Copy**.
  **Copy everything**, under `…`, copies all the pane holds.
- **Search** with **Find**, under `…`.
- **Open a web address** by tapping it. A sheet shows where it really leads, with **Open**
  and **Copy**, since a pane's words can say one thing and lead to another. Only `http` and
  `https` open. With a mouse, hold `Ctrl` (`⌘` on a Mac) and click.
- **With a keyboard**, `Ctrl+Shift+C` copies what you've selected with the mouse and
  `Ctrl+Shift+V` pastes (`⌘C` and `⌘V` on a Mac).

**The bar of keys** has what a phone's keyboard lacks: `Esc`, `Tab`, `Ctrl`, `Alt`, the
arrows, `Ctrl+C`, `Shift+Tab`, `~`, `/`, `|`, `-`, paste, and a pencil. `Ctrl` and `Alt`
hold for the next key. The pencil opens a line you can type or dictate into, sent whole.

**Add photos and files:**

1. Tap the paperclip in the pane's bar and choose what to add. On an iPhone you can also
   take a photo there.
2. Wait for the ring to fill. **Cancel** on the card drops a file.
3. Type your question and press `Enter` yourself.

Each file is kept on the pane's machine, in the pane's directory, in
`.insensical-uploads`, and its path is typed at the cursor, as `isc upload` does, so an
agent there can read it. A file can be 50 MB at most.

**Fit panes to your screen.** The button beside `…` (also **Give the size back** under
`…`) switches whether the pane has your screen's size. Leaving the pane gives the size
back. **Fit panes as they open**, in the page's menu, is the same choice, and the browser
remembers it.

When the machine is far, what you type is drawn at once, a little fainter, until the
program shows it: see "Typing on a far machine" under [Other machines, over
ssh](#other-machines-over-ssh).

A device that only looks has no paperclip and no fit button.

### On a wide screen

From 900 pixels wide, the list sits at the side as the window's sidebar, and beside it is
a tab with its panes where its layout puts them, each one live.

- **Click a pane** to type in it. The button at the end of its header makes it fill the
  tab, and puts it back. The paperclip and `…` work as on a phone.
- **Click a row** to go to that pane's tab. Which tab you look at is your browser's own
  and doesn't change what's in front at your desk.
- **Home**, **Everything** and **Activity** work as on a phone.
- **Fit to this screen** takes the size of every pane in the tab, and **Give the sizes
  back** returns them.

A device that may arrange can also drag the border between two panes (`Escape` while you
hold it puts it back), or drag a pane by its header onto another: in the middle the two
swap, near a side the pane takes that half, and a word says which. On a touch screen, hold
the header first. Nothing is split, closed or renamed from a browser.

**Text Size**, in the menu under your device's tile, changes the letters of every pane in
this browser: `−` and `+` move it a pixel, between 9 and 28, and **Reset** goes back to
the window's size (a phone's own 14). Pinching on a phone changes the same size. The
window and other browsers keep their own.

**Fonts.** Nerd Font signs come from an icon font the page brings. At a desk, letters use
your `font-family` and `font-size` settings. A browser on another machine without that
font uses the first of `JetBrains Mono`, `Cascadia Mono`, `SF Mono`, `Menlo`, `Roboto
Mono`, `Consolas`, `Liberation Mono` and `DejaVu Sans Mono` that it has.

### Adding the page to your home screen

From the home screen the page opens full screen, like an app.

1. On an iPhone or iPad, in Safari, tap the share button, then **Add to Home Screen**. On
   Android, in Chrome, tap `⋮`, then **Add to Home screen** or **Install app**.
2. Open the page from the new icon.

On an iPhone the app has storage of its own, so it pairs once more with a new code: see "A
home-screen app is a browser of its own" under [In a browser](#in-a-browser).

**After you update insensical**, the page updates itself the next time you open it, and
nothing needs clearing. If you're typing when a newer page is ready, **Reload** takes you
to it.

`isc hub`, which serves the page, follows your daemon to a new version the next time a
browser connects after you restart the server. It keeps its port, so paired devices and
the Tailscale mapping carry on. Until it has, the page says **Not connected**; turning
**Open in a browser or on your phone** off and on in settings does it by hand.

The phone keeps no copy of the page, so it doesn't open while the server can't be reached.

On iOS 26 and later, if the header of a home-screen app looks blurred, turn on **Header
Looks Blurred** in the page's menu.

### Notifications on your phone

The page can tell your phone when a pane waits, fails or finishes while the page isn't in
front of you.

1. On an iPhone, add the page to your home screen first and open it from there.
2. Make sure the page is on `https`: `tailscale serve` gives you that, and so does a
   certificate of your own. Over `ssh -L` the address is `http://127.0.0.1`, which a browser
   also allows.
3. Press the bell, or say yes when the page offers after your first answer.

You're told what the daemon would tell a window: the machine's name, the state, and the
program's words. What you've muted stays quiet. Press one and the page opens at that
pane's card, on the machine it belongs to. You aren't told while the page is open in front
of you.

Turn notifications off under your device's tile. **Forget This Browser** and `isc hub
device rm` turn them off too.

**Who hears what:**

- **This computer's panes** reach any device with notifications on.
- **Machines of `isc host ls`** reach a device made with `--far`, with the machine's name
  first in the title.
- **Machines joined to your hub** reach every device paired with the hub that has
  notifications on. No `--far` needed, and you can't turn off one joined machine and keep
  the rest.
- A guest hears of nothing.

> [!NOTE]
> While a device with `--far` has notifications on, `isc hub` keeps one `ssh` open to each
> machine of `isc host ls`, about 20 MB a day each. On a laptop on battery or a metered
> connection, that's the cost. It ends when the last such device turns notifications off or
> loses `--far`.

**Privacy.** Notifications go through your browser maker's push service (Apple's, Google's
or Mozilla's), encrypted for your browser before they leave your machine. The service sees
that something was sent, when, and how urgent it was. The server posts only to an `https`
address on port 443 whose name resolves to public addresses, and follows no redirect.

| Command | What it does |
|---|---|
| `isc hub push ls` | Lists the browsers that get notifications, and marks the ones that hear your other machines. With `--json`, each names its device as `device`. |
| `isc hub push test` | Sends a test to each and prints what each service answered. |
| `isc hub push test --at NAME` | Sends one as from that machine, to the browsers that hear of it. |

**Your device's tile**, at the top right, opens **This browser**, laid out like a phone's
own settings. It has:

- **Icon**: five icons and ten colours.
- **Theme**: the eight that come with insensical. Left alone, the page follows your
  system: `fjord` when it's dark and `paper` when it's light.
- **Notifications** and **Fit Panes as They Open**, as switches.
- **Text Size**, and the machines this browser reaches.
- **Settings**, for a device that may manage.
- **Forget This Browser**.

### When the link drops

If your connection fails, the page keeps what it last showed, dimmed, says **Link to desk
lost · trying again** (with your computer's name in place of `desk`), reconnects by
itself, then says **Connected**. You can still scroll back, select and copy meanwhile.

**Nothing you type blind reaches an agent.** What you type arrives within a couple of
seconds or never, and nothing is kept to send later. An answer you press on a card while
the link is lost isn't sent: the card says **Not sent** and stays unanswered, so read the
question again and answer then. A line you wrote stays in its field. If you pressed a key
just before the link dropped, the card stops saying **Sent**; when the link is back it's
gone if your answer arrived and still there if not.

A change from a page known by its key (removing a device, a setting) that arrives late
isn't made, and the page tells you to try again.

If the page says the link is lost on a link that works:

- A link slower than about a second and a half there and back is too slow to type over.
- A proxy in front has to pass WebSocket messages on as they come.
- **Not connected · this page is older than its server: load it again** means your browser
  kept an older page. It reloads by itself; reload it yourself if it doesn't.

### The page's addresses

| Add to the address | Opens |
|---|---|
| `?pane=3` | Pane 3 straight away |
| `?card=3` | Pane 3's card |

A finished or failed pane is marked as seen when you press or type something in it, not
when the address opens. A device that only looks never marks a pane as seen.

### Running isc hub all the time

On Linux a package installs a user unit for it, which nothing turns on:

```
systemctl --user enable --now insensical-hub
```

### Limits in a browser

- **Machines.** A device made without `--far` shows only the machine `isc hub` runs on.
  Make it with `--far` to see machines named with `isc host add`, or run `isc hub` on each.
- **Changes.** You can't open, close, split, minimize or rename anything, and only a
  device with `arrange` moves panes. Settings changes only what [Managing isc hub from the
  page](#managing-isc-hub-from-the-page) lists.
- **Cards.** The keys under a card are always the same seven. The page never works out an
  agent's choices for you.
- **Keys.** The browser keeps `Ctrl+W`, `Ctrl+T`, `Ctrl+N` and the like for itself.
- **Selecting.** A phone's own selection handles don't appear over a pane. Hold a finger
  still to select.
- **Drawing.** Kitty graphics aren't shown. No sound is played.
- **Paths.** A press on a path in a pane does nothing, since it's a path on another machine.
- **Devices.** Everyone who pairs by one device's code is the same device to the server.
  To give someone less, share one project or one terminal instead.
- **Encryption.** `isc hub`'s own encryption, with `--tls-cert`, needs an x86-64 processor
  from about 2014 or later, or on arm64 one with the AES and SHA-2 extensions, which a
  Raspberry Pi 4 lacks. Notifications need the same. On those, put a proxy in front.

### Joining another machine to your hub

Another computer of yours can join the `isc hub` your browser goes to, and your devices
then reach it at the hub's address. The joined machine listens on nothing: its `isc hub`
dials the hub. What a device and that machine say to each other is sealed for the two of
them, so the hub carries it and can't read it. The exception is notifications, which the
hub reads, as it reads its own, to tell your devices.

**Join a machine to your hub:**

1. On the hub, name the machine:

   ```
   isc hub invite studio
   ```

   It prints a line: the hub's address, then `#j1.` and letters. In the window, **Invite a
   Machine** on the Hub page gives the same line. It joins one machine, within ten
   minutes.

2. On the other machine, open Settings > Hub, press **Join a Hub** and paste the line. Or
   run:

   ```
   isc hub join
   ```

   and paste the line. It isn't shown as you paste it, and it's never an argument, so it
   stays out of your shell's history. From a script, give it on standard input.

3. Run `isc hub status` there to see which hub it's joined to and whether it's connected
   (`joined` in `--json`: `address`, `name`, `since`, `connected`).

Give the hub an address the other machine reaches: its Tailscale name from `isc hub
tailscale on`, or yours with `--url https://desk.example.com`. Over `https://` the
certificate must be one a public authority signed, as Tailscale's are.

Good to know:

- Three wrong tries close the invite.
- `isc hub join` checks that the server at that address has the key the line names, and
  stops with nothing changed if it doesn't.
- It starts `isc hub` on the joined machine if it isn't running. If that fails, the join
  still holds and you run `isc hub` yourself.
- Hub-only commands are refused on a joined machine: `isc hub open`, `invite`, `machine`,
  `ban`, `unban`, `tailscale on` and `public on`. Run them on the hub, or `isc hub leave` to
  make this machine a hub of its own. `status`, `stop`, `leave`, `rekey`, `device` and
  `push` work as ever.

**Your devices reach it.** A device paired with the hub reaches every joined machine, with
what it may do on the hub. There's no code to make on the joined machine. Remove a device
on the hub, or change what it may do, and it's so on every joined machine at once. A
guest, kept to one project or terminal of the hub, reaches no joined machine.

A joined machine doesn't pair your devices itself: `isc hub device add` there says to pair
the device with the hub. It can still share one project or terminal of its own with a
guest, by `isc hub device add --only` or `isc hub open`, and that code is at the hub's
address. If the hub can't reach this machine, you're told so above the guest's code, since
it won't work until it can. Either the machine isn't joined any more (the hub was given
`isc hub machine rm`, or this machine's key changed), so join it again with `isc hub
invite` and `isc hub join`; or it isn't connected right now, so check that insensical runs
here and the network.

**It appears on your page by itself.** A page paired with the hub lists every joined
machine after the hub's own heading, **online** or **not connected**. Press a heading to
open it; what you type in its terminals goes to it alone. **This browser** lists them
under **Machines**. A machine that's not connected stays in the list, dimmed, and returns
once it dials the hub again. One taken away from the hub, or that leaves it, goes from the
list.

**Notifications from a joined machine.** Turn on **Notifications** in **This browser** on
a page paired with the hub, and a pane that waits, fails or finishes on any joined machine
reaches that device, with the machine's name first in the title. The hub reads these
notifications to post them. A joined machine sends only its own panes' notifications, not
those of machines it reaches over `ssh`, and the test notification on its Hub page reaches
no browser.

**Add a device from another device.** A device that may manage a machine can add another
device by its key, from its page, with no code. The machine decides what the new device
gets, and never more than the device that adds it has: no right it lacks, no later end,
and your other machines only if it reaches them. The machine records who added it (`isc
hub device ls` says "added by phone"), and each open page of a device that may manage it
is told.

On the page it's **Approve a Device**:

1. On the new device, open the hub's address and press **Already paired a phone? Approve
   from it**. It shows a picture to scan, a link to copy, and three groups of four letters
   under **This device**. These hold the key the browser just made and nothing secret. **Pair
   with a Code Instead** goes back to the field.
2. On a device that may manage, open **Settings**, then **Your devices**, press **Approve
   a Device**, and scan the picture or paste the link. The link can't be typed. Anyone can
   send you a link: approve only for a device you're holding.
3. Under **Check the Device**, look at the new device. If it shows the same three groups,
   turn on **The same as on the device**; **Approve** stays off until you do. A key someone
   else made shows other groups.
4. Name it and choose what it may do and how long it stays. **Only look** is chosen first,
   and you're offered only what your own device may do. With machines joined to the hub,
   **On** chooses which one lets it in. If your device reaches your other machines, a switch
   gives the new one that too.
5. Press **Approve**. The new device's page goes on by itself.

A machine takes at most three devices an hour added this way, from all devices together.
The fourth is refused: add it at the machine with `isc hub device add`.

`isc hub device rm phone` lists the devices it added and asks whether to remove them too.
`--with-approved` removes them without asking.

**Take a machine or a device away:**

| Command | What it does |
|---|---|
| `isc hub machine ls` | On the hub, lists the joined machines, whether each is connected now (`connected` in `--json`) and when each last dialled. `full` marks a machine that adds no more devices, which usually means it has turned bad. |
| `isc hub machine rm studio` | Cuts the machine off at once. It joins again only by a new invite. |
| `isc hub ban ID` | On the hub, carries nothing more for a device, by its id as `isc hub device ls --json` shows it. Each machine still lists it, and the devices it added aren't banned: run `isc hub device rm NAME --with-approved` on each machine. |
| `isc hub unban ID` | Lifts a ban. |
| `isc hub leave` | On a joined machine, leaves the hub. Its `isc hub` listens again as a hub of its own, and each device pairs again at its own address. |
| `isc hub rekey` | Gives a machine a new key and id, for one whose `hub` folder may have been copied. Each device pairs again. A joined machine leaves its hub too and joins again by a new invite. |

**Security.**

- If the hub can't read `members.json` (a file you edited badly, or one closed to you), it
  says so in its log and changes nothing until it can. No ban is lifted by a file it can't
  read, and a hub that never read it carries nobody.
- A machine joins one hub. The hub carries a device only to machines that keep it, and a
  stranger only to a code that's open.
- The hub serves the page every device runs, so it's trusted as the page is: keep it on a
  computer of yours.

### The whole interface in a terminal

Run `isc` with nothing after it and insensical draws in the terminal you're in, with no
window. It's handy when you'd rather stay in a terminal, on a machine with no desktop, and
on one you reach over `ssh`:

```sh
ssh build-host -t isc
isc --host build
```

The first runs the interface on the other machine and sends its screen through `ssh`. The
second runs it here, on the other machine's daemon: see [Other machines, over
ssh](#other-machines-over-ssh).

It's another client of the same daemon: what you do in it shows in a window at once, and
the other way round. Press `Ctrl+Space` then `D` to leave; nothing ends.

- `isc tui` is the same thing in full. It takes the options `isc` takes before a command:
  `--host`, `--socket` and `--rights`.
- `isc --help` lists the commands. So does `isc` alone where there's no terminal to draw
  in (a script, a pipe) and inside one of insensical's own panes.

**What it needs:**

- **A terminal to draw in.** It works from 20 columns by 6 rows. Under 76 columns the list
  of projects is left out.
- **A Nerd Font**, since every icon is one. Ghostty, kitty, WezTerm and insensical's own
  panes carry the icons themselves; in another terminal, set its font to a Nerd Font or the
  icons show as empty boxes. If icons are cut off at the right, use the "Mono" variant
  (`JetBrainsMono Nerd Font Mono`, say).
- **Colours.** `colours = "auto"` works them out from `COLORTERM` and `TERM`. Over `ssh`,
  `COLORTERM` is often not passed on, so if colours look coarse set `colours = "truecolor"`
  under `[tui]`.

**What's on the screen:**

- **The top row** has the tabs of the project in front, and a `+` to open a tab. At the
  right are the counts of what finished and what needs you, and the leader's key. Press the
  counts to go to the next pane that needs you.
- **The list at the left** is your projects, one to a row, each with the state of what
  runs in it.
- **Each pane is in a box**, with its name and its state at the top, and how long it has
  needed you. The focused pane's box is in the accent colour, and a waiting one in its
  waiting colour.
- **The line at the foot** says what your keys are doing (`PANE`, `MENU` after the leader,
  `LIST`, `LOCKED`), where the focused pane is, and the keys you'll want most.
- **The dock**, under the panes, has a pill for each pane you've minimized in the tab.

**Show only activity.** Press `Ctrl+Space` then `Y` (`activity`) to turn the list at the
left into the panes that are at work or need you, under their projects' names. The same
key turns it back. It follows the rules in "Showing only activity" under [The
sidebar](#the-sidebar). Click a row to go to its pane.

`Ctrl+Space` then `}` and `{` walk the rows (`next-activity`, `previous-activity`),
whether the list is on or not. While it's on, `Ctrl+Space` then `A` goes to the next pane
that needs you or has finished.

**Keys.** Commands follow the leader, `Ctrl+Space`, and the keys after it are those of
[the leader keymap](#the-leader-keymap): `T` opens a tab, `Shift+S` and `S` split right
and down, `H` `J` `K` `L` move between panes. While the leader waits, a panel at the foot
lists every key that can follow.

| Keys | Does |
|---|---|
| `Ctrl+Space` then `Space` | The palette |
| `Ctrl+Space` then `Shift+/` | Lists every key |
| `Ctrl+Space` then `C` | Copy |
| `Ctrl+Space` then `Shift+V` | Paste |
| `Ctrl+Space` then `Ctrl+F` | Find text |
| `Ctrl+Space` then `Ctrl+A` | Select everything |
| `Ctrl+Space` then `Shift+Up` / `Shift+Down` | Go to the command before and after |
| `Ctrl+Space` then `PageUp` / `PageDown` | Scroll (`scroll-up`, `scroll-down`) |
| `Shift+PageUp` / `Shift+PageDown` | Scroll without the leader |
| `Ctrl+Space` then `Shift+X` | Give every key to the program |
| `Ctrl+Space` then `Shift+A` | Ask to set up the agents |
| `Ctrl+Space` then `Shift+U` | The restart question: restarts the server onto the installed version, or says what to run |
| `Ctrl+Z` | Goes to the program in the pane |

- **The keys are the window's.** Your `keymap`, `leader`, `[keys]`, `[keys.leader]` and
  `[keys.direct]` are read as the window reads them.
- **The leader always works here**, whichever keymap you use, because your terminal may
  keep a chord for itself. Your own chords work too where the terminal tells keys apart
  (Ghostty, kitty, WezTerm, foot and others that speak the Kitty keyboard protocol) and
  doesn't take the chord first. To free a chord, unbind it in that terminal.
- **To get the keys back when locked**, press the leader then `Shift+X`, click `LOCKED` at
  the foot, or press the chord that locks (`Ctrl+Alt+Shift+L` by default).
- **To stop the interface itself**, send it `SIGTSTP` (`kill -TSTP`) and bring it back
  with `fg`.
- If you unbind the key that leaves, `Ctrl+Space` then `Shift+D`, `Shift+Q` or `Ctrl+D`
  leaves, or else `D`.
- Closing a tab, closing a project and putting a project to sleep ask first. A project
  that's asleep says so where its panes would be; `Enter` wakes it.

**The palette** lists what the window's lists, in the same order: see [The
palette](#the-palette). Here it also has the `terminal` theme (see Themes below), copy,
paste, select-all and find, and **Recent**, the last five commands you chose. These
commands exist only here and have no default key; give one a key under `[keys.leader]` if
you use it often.

| Palette entry | Command | What it does |
|---|---|---|
| Rename this project | `rename-project` | |
| Close this project and everything in it | `close-project` | |
| Move this project up / down the list | `move-project-up`, `move-project-down` | Moves it in the list, as dragging its row does. |
| Move this pane to another tab | `move-to-tab` | Lists every other tab; `Enter` puts the pane at the right of that tab's panes and takes you there. |
| Choose this project's icon | `project-icon` | See below. |
| Go to a machine; the Remote Hosts and Hub pages | `go-to-machine`, `settings-hosts`, `settings-server` | See below. |

**Choose a project's icon:**

1. Open the palette and choose "Choose this project's icon". Colours are across the top
   and marks below.
2. Type to narrow the marks (`git`, `rust`, `docker`). `Up` and `Down` move through the
   marks, `Left` and `Right` through the colours. The first colour, "by name", is the one
   that comes of the project's name.
3. Press `Enter` to set it, as `isc project icon` would. `Escape` leaves it as it was.

The first mark is the folder. A project with a drawing of yours (an SVG) shows "Keep your
drawing" first. Only marks a Nerd Font has a sign for are listed.

**A new project** asks for a directory to work in, or only a name. A directory is written
from `/` or from `~`, and `Tab` goes into the one marked. One that isn't there is made
after one more `Enter`.

**Lists and sheets** open over the panes and take your keys until they close; the line at
the foot says `LIST`. `Esc` closes any of them.

| List | Keys | Does |
|---|---|---|
| The palette | `Up`, `Down`, `Tab`; `PageUp`, `PageDown`; `Enter` | Moves and chooses. |
| Every tab (`overview`), `Ctrl+Space` then `W` | Arrows or `H` `J` `K` `L`; `Enter`; `Tab` | A card for each tab of every project. `Enter` goes to the tab; `Tab` narrows to tabs with a pane that needs you, and back. |
| Find a file (`pick-path`), `Ctrl+Space` then `/` | `Up`, `Down`; `Enter`; `Tab`; `Backspace` | Type a few letters of a file or directory under the pane's directory. `Enter` types the path into the pane, `Tab` goes into the directory marked, `Backspace` on an empty field comes out of it. |
| Find text, `Ctrl+Space` then `Ctrl+F` | `Enter` or `Up`; `Shift+Enter` or `Down`; `Esc` | The match is selected, so `C` copies it. `Enter` or `Up` goes to the one before, `Shift+Enter` or `Down` to the one after. |
| Every key | `Up`, `Down`, `PageUp`, `PageDown`; `Esc` or `Enter` | Type to narrow it. |
| Go to a machine | `Up`, `Down`, `K`, `J`, `Tab`; `Enter` | Lists this computer and each machine under `[hosts]`, each tried as the list opens: `reached`, `not reached`, `ssh asks`. The machines you leave carry on. |

**The settings sheet** is `Ctrl+Space` then `,`. It has three pages: **Settings**,
**Remote Hosts** and **Hub**. `1`, `2` and `3` go to a page, as does a click on its name.
The bottom line says what the keys do on the row you're on, and `Esc` closes the sheet.
The palette opens the other two pages directly: "Remote hosts: add, rename or forget a
machine" and "Hub: browsers, phones and devices".

*Settings* lists every setting in sections, and a change is written to your file at once.
A dot after a name means your file changes it.

| Setting type | Keys |
|---|---|
| Any | `Up` and `Down` (or `K` and `J`) move; `Tab` goes to the next heading; `Backspace` puts the setting back to its default. |
| On or off | `Right` on, `Left` off, `Space` or `Enter` turns it over. |
| Two values | `Left` takes the first, `Right` the second, `Space` or `Enter` the other. |
| More values | `Space`, `Enter` or `Right` opens the list; `Up`, `Down`, `Home`, `End` or a letter move; `Enter` or `Space` chooses; `Esc` closes without choosing. |
| A number | `−` and `+`, or `Left` and `Right`. |

The mouse works too: click a value to choose it or open its list.

*Remote Hosts* lists the machines under `[hosts]`, each tried when you open the page:
`reached`, `not reached` or `ssh asks`.

| Key | Does |
|---|---|
| `Enter` | Tries the machine again |
| `N`, or `Enter` on "Add a remote host" | Adds one. Type what you'd give `ssh`, or pick from the machines of your tailnet or the names in your ssh configuration (`Tab` changes between the lists). |
| `R` | Renames the machine |
| `Backspace` | Forgets it, after asking |
| `A` | Answers what `ssh` asks, in a new tab on this computer |

*Hub* is how this computer is reached from a browser or a phone, as the window's Hub page
has it: see [In a browser](#in-a-browser). With another machine shown, it only says "This
is set on that computer itself."

- **Open in a browser or on your phone** starts and stops `isc hub`, and says the address.
- **This computer's browser** opens insensical in the browser here, signed in.
- **Tailscale** says how it stands. `Enter` turns it on or off, or signs in. Under it are
  the two ways in that need no switch: an `ssh -L` line and a proxy of your own.
- **Each paired device** is a row with its name, what it may do, when it was last used and
  when it ends. `Backspace` removes a device, after asking.
- **Add a device** asks for a name, what it may do (Everything, Look and type, or Only
  look), whether it reaches your other machines, how long it stays paired, how many devices
  the code pairs and how long the code works. You then get the code to scan, the address to
  open, and the code as twelve groups to type instead.
- **Send a test notification** says what each device answered.
- **On a hub**, **Joined machines** lists each joined computer. `Backspace` removes one,
  after asking. **Invite a machine** gives the line to paste on that computer, and `C`
  copies it. See [Joining another machine to your
  hub](#joining-another-machine-to-your-hub).
- **On a computer that joined no hub**, **Join a hub** takes the line from another hub's
  Invite a machine and runs `isc hub join`.
- **On a computer joined to a hub**, **Joined to** names the hub and whether it's
  connected, **Start isc hub** starts it when it isn't running, and **Leave the hub** leaves
  it, after asking.

The code is drawn black on white whatever your theme, because cameras read nothing else.
It needs about 49 columns and 25 rows for an address on your tailnet; in a smaller
terminal you get the address alone and the size it needs.

**If the server stops** while you're in it, a card says "The server has gone away". The
panes are empty and what you type reaches nothing, but you don't have to leave: when a
server runs on that socket again, the card goes and the panes come back, as they were if
the server kept them: see [What survives what](#what-survives-what). Start one with any
`isc` command or `isc daemon`; nothing is started for you, since the server may have been
stopped on purpose. If the server becomes another version (after an update and a restart),
it says so: leave, and run `isc` again.

**The mouse** goes to the pane you press, selects text, scrolls, and drags the edge where
two panes meet.

- Press a tab, a project or a dock pill to go there, and the `+` after the tabs to open a
  tab.
- Press a notice to go to its pane, or its cross to drop it.
- Drag a project's row over another's to move it in the list.
- Drag a floating pane by the top of its box, and its bottom, right side or corner to
  resize it.
- Press the cross on the tab under the pointer, or use the middle button on the tab, to
  close it.
- The middle button in a pane pastes what you last selected here.
- Hold `Shift` and the mouse is your terminal's own again, to select across the whole
  screen.
- A web address in a pane is passed to your terminal as a link, and your terminal opens
  it. Only `http://` and `https://` addresses are passed on. Paths aren't opened here.

**A pane that a window also shows** is the size that fits whichever you used last, as with
[two windows](#two-windows). When that's the window, the pane says "sized by another
window" at the bottom of its box. To take the size back, press a key or a mouse button
here, press those words, or run "Size the panes for this window, over any other that shows
them" in the palette (`lead`).

**Copying and pasting** use your terminal's clipboard, so they work across `ssh`.

- The leader and then `v` selects with the keyboard, with the window's keys: see
  [Selecting with the keyboard](#selecting-with-the-keyboard).
- The copy command is sent as OSC 52, which a terminal can be set to refuse and which
  Terminal on a Mac doesn't take at all.
- Selecting alone sends your terminal nothing. What you selected is kept here for the
  middle button.
- Pasting with your terminal's own key always works. The paste command asks the terminal
  for its clipboard, which many refuse; if yours doesn't answer, a notice says so, and you
  use the terminal's own key.
- A paste is held and asked about as in the window: see [Pasting](#pasting).
- A program can put text on your terminal's clipboard only while you're known to be here:
  while your terminal says it has the keyboard, or for a minute after you last pressed a key
  or a mouse button here.

**Pictures** a program draws with the Kitty graphics protocol are shown where your
terminal shows them itself: Ghostty, kitty, WezTerm and others that answer when asked. See
[Pictures](#pictures). A terminal that doesn't answer gets no picture, and inside `tmux`
or `screen` there are none. A picture is shown whole or not at all: while a list, a
question, a notice or a floating pane lies over it, it's taken away.

**Notifications.** A pane that needs you says so in a notice at the foot of the list of
projects, or at the top right of the panes when that list isn't shown. On a machine with
no desktop, the notification is also sent to your terminal, which shows it as its own if
it knows how (OSC 777), with a bell. A pane that waits or failed keeps its notice until
you've gone to it. When a program in any pane of the tab in front rings its bell, your
terminal rings, unless you've silenced the pane or project. Titles and text are cut and
bursts are held back, as [section 12](#12-notifications) says of the application.

**Themes.** It uses your insensical theme, the same as the window: the light or dark one
of a pair, by your terminal's own background. One more theme exists only here, `terminal`,
which is your terminal's own background, text colour and sixteen colours, so it looks like
everything else you run there. Choose one in the palette: type `theme`. A theme chosen
here is the window's too, and the other way round. Only `terminal`, and any theme once
`[tui]` already names one, are written under `[tui]` for a terminal alone.

```toml
[tui]
theme = "terminal"
```

- Under `terminal` your terminal's background is never painted over, so a transparent or
  pictured background stays.
- Under any other theme your terminal's background is set to the theme's while it runs,
  and put back when you leave.
- Put `theme = "terminal"` under `[tui]`, not at the top of the file: the window has no
  terminal to take colours from.

**Settings.** It reads the window's own settings for the theme, the keys and motion; there
is no second set. Notifications, sounds, scrollback and the rest of what the daemon does
are the daemon's. A `[tui]` table in `config.toml` can say otherwise for a terminal alone
about `theme`, `activity-idle`, `interface-style`, `motion`, `attention-glow`,
`type-ahead`, `clipboard`, `keymap` and `leader`, and holds the two settings only a
terminal has:

```toml
[tui]
theme = "terminal"     # or any of those seven, to differ here
colours = "auto"       # "truecolor", "256" or "16"
mouse = true
```

- The window never reads `[tui]`.
- Only the eleven settings named above are read from `[tui]`; another name there changes
  nothing. `[keys]` and its tables are read from the top of the file only.
- `colours` and `mouse` are read from `[tui]` only.
- Every one applies within a second of saving.
- A line this interface can't use is said in a notice headed Settings, and only that
  setting falls back to its default. Broken TOML leaves every setting at its default here.

The settings sheet lists every setting that takes effect here and is set without typing:
`activity-idle`, `interface-style`, `motion`, `attention-glow`, `keymap`, `notifications`
and the three `notify-` switches, `long-command`, `sound`, `notify-command-when`,
`notify-url-when`, `notify-url-body`, `tray`, `scrollback`, `term`, `type-ahead`,
`clipboard`, `keep-scrollback`, `tmux-shim`, `agents-at-once`, `agents-deep`, and under
"This terminal", `colours` and `mouse`. `leader`, `shell`, `notify-command`, `notify-url`
and your own keys are set in the file.

**Limits:**

- One machine is shown at a time; "Go to a machine" changes which.
- A pane has no buttons: closing, zooming, floating and putting away are commands, and a
  double-click on a pane's name fills the tab with it. A pane isn't dragged with the mouse;
  the move commands, `move-to-tab` and `isc put` do it.
- Sharing, opening and saving a project file, and "Add a machine" are the window's. `isc
  host add` and the Remote Hosts page add a machine, and `isc host set` changes its colour,
  icon, address and `careful` mark.
- `isc hub open --print`, `isc hub public` and `isc clients` give what the Hub page
  doesn't: a link for this computer's browser, a public link, and who is connected.
- On a small screen, lists and questions are cut to fit; under 20 columns or 6 rows it
  says "Needs more room".
- A key written as a sign (`?`, `<`) works wherever that sign is on your keyboard. One
  written with `Ctrl` or `Alt` and a sign (`ctrl-}`) is found by where the key is on a US
  keyboard.
- A curly, dotted, dashed or double underline is a plain one unless your terminal is known
  to show every colour or `colours = "truecolor"` says it does.
- It can't be run inside one of insensical's own panes on the same daemon.

### Other commands

| Command | What it does |
|---|---|
| `isc config` | Prints a default settings file. |
| `isc completions SHELL` | Prints completions for `bash`, `zsh` or `fish`. |
| `isc version` | Prints this version and the running daemon's, and whether they can talk to each other. |
| `isc kill-server` | Stops the daemon and every program it runs, and returns once the daemon is gone. |
| `isc restart-server [--yes]` | Moves the daemon to this version with everything still running. If no daemon is running, it starts one. If you answer no, everything stays as it is and the command exits with status 1. |

**Shell completions.** The packages install
them for bash, zsh and fish. On a Mac they're also inside the application, in
`Contents/Resources/completions`. To put them somewhere yourself:

```sh
isc completions fish > ~/.config/fish/completions/isc.fish
isc completions zsh > ~/.local/share/zsh/site-functions/_isc    # any directory in $fpath
isc completions bash > ~/.local/share/bash-completion/completions/isc
```

Pressing Tab then offers commands and their options, and the session's live panes, tabs
and projects wherever a command takes one, plus the states `isc status` takes and the
sides `isc focus` takes. A project whose name a shell wouldn't read as one plain word, or
that another project shares, is offered by its number.

Asking never starts the daemon. If none is running, you still get commands and options. It
asks the daemon at the default socket or `$INSENSICAL_SOCKET`, not one given with
`--socket` on the line you're typing. Completions saved from an older version list
commands only, so print them again.

---

## 17. Stopping, restarting and new versions

Close the window, update insensical or stop the daemon whenever you like. This section
tells you what keeps running and what comes back.

### What survives what

| What happens | Programs | Screens and scrollback | Layout, directories, names |
|---|---|---|---|
| The window is closed | keep running | kept | kept |
| The window is opened again | — | shown as they are | — |
| The daemon moves to a new version (restart from the window, `isc restart-server`) | keep running | kept | kept |
| The daemon is stopped (`isc kill-server`, a logout that stops your user manager, reboot) | **end** | kept, as text and colour | kept |
| The daemon dies with no warning (crash, power cut, `kill -9`) | **end** | **lost** | kept |

### After the daemon has been stopped

Your layout is saved to a file whenever it changes: projects, tabs, splits, floats,
minimized panes, focus, each pane's directory, and names. When the daemon starts, it
rebuilds all of it. Zoomed, floating, minimized, muted and sleeping things all come back
as they were.

Each pane gets a fresh shell in its directory. If the directory is gone, the shell starts
in the project's root, or in your home directory. Nothing runs again by itself: a pane
that was running `nvim` or an agent comes back as a shell, and a program's environment
isn't kept.

Each pane also shows what it showed before, history included, at the same size. A dim
line, "── from before the server stopped ──", separates it from the new prompt.

- You can scroll, search and copy it, but it's only the printed text. The programs that
  printed it are gone.
- It's written when the daemon is told to stop and when a pane's program ends. A crash or
  power cut keeps the layout but loses the screens.

Screens are saved in `~/.local/state/insensical/screens`, readable only by you. They hold
everything a program printed, secrets included. With `keep-scrollback = false`, nothing is
written, and files already there are removed as their panes close.

**What ran in a pane is offered, not run**

- A pane started with a command, as in `isc run -- npm start`, comes back as a shell with
  that command typed at the prompt. Press Enter to run it, or edit it first.
- Only the command the pane started with is offered, not ones you typed in the shell.
- With the Claude Code hooks installed, a pane where Claude was last working gets `claude
  --resume <session>` at the prompt, until you run something else there. Codex, opencode and
  pi are offered their own resume commands (section 11). Gemini CLI isn't.

**A layout file that can't be read**

The daemon sets it aside as `state.json.unreadable`, beside the original, and starts
empty. That means a file from a version this one can't read, or a damaged one. To go back
to it, run `isc kill-server`, mend the file, and put it back as `state.json`.

### A new version installed underneath

Installing replaces files and nothing else. Your daemon and window keep working. The top
bar says "insensical was updated. Restart to use the new version." `isc version` and `isc
ls` say the same.

To move to the new version, do any of these:

- press the notice;
- press `Ctrl+Alt+Shift+R` (leader `U`);
- choose "Restart insensical" in the palette;
- close the application and open it again;
- run `isc restart-server` in a terminal.

Each one asks first. Press Enter to restart, or Escape to leave everything as it is. With
no terminal to ask on, `isc restart-server` does nothing unless you pass `--yes`.

Restarting ends nothing. The daemon becomes the installed version in place, with the same
terminals. Programs, screens and scrollback carry on.

The first move to a version that can do this still ends everything, and the prompt says
so. After that, updates end nothing.

**Good to know**

- If you open the application while the old daemon is still running, the window says so.
  Press Enter to restart the daemon, or Escape to leave it and close the window.
- If the server is newer than the window ("The server is already the new version, and this
  window is from before it"), the server is left alone whatever you press. The new
  application opens by itself or on Enter. Otherwise, close the window and open the
  insensical you installed.
- If the daemon can't make way, nothing is stopped. The window says "The server did not
  move to the new version" and why. Press Enter to go back. `isc restart-server` prints the
  reason and exits with status 1.

Across a move:

- Windows reconnect by themselves. `isc attach`, `isc events` and `isc wait` end, and you
  run them again.
- With a different terminal engine, screens are redrawn: text, colours, history and cursor
  are kept, a redefined colour isn't.

Restarting won't give you a fresh daemon, for instance after it started with the wrong
environment. For that, run `isc kill-server`, which ends every program, then run any
command or open the window.

### When the daemon goes away under an open window

The window looks for a daemon once a second and carries on when one appears. Press Enter
to start one yourself.

- "The server has stopped": the layout comes back when it starts.
- "The server is moving to a new version": the window finds it again by itself.
- "This window fell behind the server and was cut off. It has connected again.": the
  daemon cut the window off after it left 8 MiB unread.
- "This window was disconnected": you ended its connection from another client. Press Enter.
- If the new daemon is a different version from the window, the window starts the
  installed application in its place, or tells you to close and reopen.

### Different versions of `isc`, window and daemon

- They work together when they share a protocol and terminal engine, whatever their
  version numbers. A mismatch is refused with a message that names `isc restart-server`.
- With `--host`, the other machine's `isc` has to match too.
- These always work: `isc version`, `isc status`, `isc notify`, the agents' hooks, `isc
  kill-server`, `isc restart-server`.

---

## 18. When something goes wrong

Find what you see, then do what's next to it. Each entry is the symptom, why it happens,
and the fix.

### Starting, stopping and versions

| What you see | Why | What to do |
|---|---|---|
| "The insensical server cannot be reached" | No daemon is running and none could be started: `isc` isn't beside the application or on `PATH`, or the socket's directory isn't private | Put `isc` beside `insensical`; check the directory's permissions |
| "insensical was updated", at launch | A new version was installed under a running daemon | Press Enter to restart it. Everything keeps running, unless the daemon is too old to make way. Escape leaves it |
| "The server is already the new version, and this window is from before it", at launch | The server moved on while an older application was still running | Press Enter to open the new application, or close the window and open insensical again. Nothing ends |
| `isc: protocol N does not match the running daemon's M` | The same, from the command line | Run `isc restart-server` |
| "The server has stopped" in the window | The daemon was stopped or crashed | Press Enter to start it; the layout comes back |
| "This window was disconnected", or "Disconnected" in a terminal or browser | You ended that connection from another client, with `isc disconnect` or **Disconnect** | Press Enter in the window, any key in a terminal, or reload the page |
| "The server did not move to the new version", or `isc: the daemon was not moved` | The installed `isc` is missing from where the daemon started, or can't take over what it runs | Nothing was lost. Press Enter to go back. Reinstall, or run `isc kill-server` and start again, which ends what's running |
| `daemon: running, but it does not say what it is`, from `isc version` | The daemon is too old to answer, or stuck | Run `isc restart-server` |
| `a daemon is already running` | `isc daemon` was run twice | Nothing, or run `isc kill-server` first |
| `path must be shorter than SUN_LEN` | The socket's path is too long | Set a shorter `INSENSICAL_SOCKET` |
| `isc: ssh needs your key or password for build` | `ssh` to that machine needs something typed, and a link can't type it | `isc host answer build` shows what it asks. Give the machine a key (`ssh-copy-id build`), or load yours with `ssh-add` |
| `isc: ssh hasn't met build before` | `ssh` would have asked about the host's key | Run `isc host answer build` and answer it |
| `isc: build's host key is not the one ssh has for it` | The machine's key changed | Find out why first. `ssh-keygen -R build` forgets the old key |
| `isc: build is off` | A machine of your tailnet that Tailscale says isn't online | Turn it on, or start Tailscale there |
| `` isc: build has no `isc` where a shell finds it `` | insensical isn't installed there, or is somewhere `ssh` doesn't look | Install it there. `~/.local/bin`, `/usr/local/bin` and Homebrew's `bin` are checked |
| `isc: build has isc X … and this machine has isc Y … which cannot speak to each other` | The two machines have different versions | Install the same version on both (`isc host put build` puts this one there), then run `isc host restart build` |
| `isc: this connection was not given the right to …, which … needs. Its rights are: …` | The command ran with `--rights` and left out a right it needs | Run it with that right, or without `--rights` |
| `` isc: `isc integrate` asks before it changes an agent's settings, and on another machine there is nobody to ask `` | `isc --host NAME integrate` without `--status` or `--yes` | Run it in a terminal there, or look with `--status` and then add `--yes` |
| `isc: theme is a setting of the interface, read from the file on the machine the interface runs on` | `isc --host NAME config` was given a setting that isn't the daemon's | Leave `--host` out |
| `` isc: this daemon was not started as `isc daemon`, so it has no `isc` to do that with `` | A test or another program runs the daemon inside itself | Run `isc integrate` or `isc config` on that machine |
| `isc: the link to the other machine was lost`, or "is out of reach" in the interface | The machine stopped answering, or `ssh` ended | Nothing there was lost. Run the command again; the interface retries by itself |
| The update notice appears though nothing was updated | The binaries were rebuilt or reinstalled | Restart when convenient |

### Settings and appearance

| What you see | Why | What to do |
|---|---|---|
| A red message in the top bar | A setting couldn't be used; the message names the file and line | Fix the line; it applies within a second |
| All settings seem ignored | One unreadable line voids the whole file | Same |
| Text smaller or larger than expected | `font-size` is in points | 12 points is 16 pixels on Linux |
| Text thinner than in another terminal | There's no font weight setting | Nothing; the regular face is used |
| `theme = "ghostty"` is an error | Ghostty isn't installed | Use a built-in theme or a theme file |
| The window has a generic icon | It was run without installing, or the icon cache is old | Install it from a package, or rebuild the icon cache (see [The icon](#the-icon)); restart the launcher or panel |

### Panes and shells

| What you see | Why | What to do |
|---|---|---|
| A different shell, prompt or completion than in another terminal | Panes run `$SHELL`; the other terminal runs something else | Set `shell = "…"` |
| Commands found in other terminals aren't found in panes | The daemon started from a launcher with a short `PATH` | Set `PATH` in the shell's start-up files, or run `isc kill-server` and start it from a terminal |
| On another machine, the first letters of a line show late and the rest at once | Typing is drawn ahead only once the program has shown two letters you typed | Nothing: it keeps passwords from being drawn. `type-ahead = "off"` makes every letter wait alike |
| On another machine, a letter you typed shows and then goes | The program answered the key with something else, or nothing | Nothing: what's left is what the program has |
| A new variable in a shell profile is missing in panes | The daemon keeps the environment it started with | Same |
| Programs over `ssh` complain about an unknown terminal | The other machine has no `xterm-ghostty` description | `infocmp -x xterm-ghostty \| ssh host tic -x -`, or `term = "xterm-256color"` |
| A pane shows the generic terminal icon | It has printed nothing and been sent nothing, or its program isn't in the list | Nothing |
| A pasted block waits at the foot of the pane | It has several lines and the program didn't ask for marked pastes, or it holds the sequence that ends one. The card says which | Read the card, then press Enter to paste or Escape to drop |
| `Ctrl`+click on a path shows it in the file manager | It's a directory, a runnable file, or a kind opening could run | Open it yourself, or set `open-files-with` to your editor |
| A path isn't underlined under the pointer | Paths are looked up only while `Ctrl` is held | Hold `Ctrl` |
| An input method does nothing | It's off by default | Set `input-method = true` |
| On a Mac, Alt+B and Alt+F do nothing in a shell, or type `∫` and `ƒ` | Option types the layout's character | Set `option-as-alt = true` |
| Search misses text that is on screen | The text wraps across three rows or more; two rows are found | Search for a part on one or two rows |
| A pane doesn't fit its window, and its header says "Sized elsewhere" | Another window, or `isc attach`, was used last | Click this window, or press the mark |
| `` Ctrl+` `` doesn't reach a program | That key brings up the scratch pane | Set the key to `none` in `[keys]` |

### States, agents and notifications

| What you see | Why | What to do |
|---|---|---|
| A pane never shows `done` | The command ran under 30 seconds and reported nothing | Lower `long-command`, use `isc status` from a hook, or a program that reports progress |
| A failed command shows `done` | The shell doesn't mark its commands | Use fish 4, or the `isc shell-integration` line for bash or zsh |
| A pane is stuck in a state | A hook or the program reported it and nothing has reported since | `isc status explain` says who. Run `isc status clear`, or choose "Forget what this pane's program said it was doing" in the palette |
| `isc status` changes nothing in a pane | The pane's program reports its own state in its terminal, which wins | `isc status explain` says so. It counts again when the program says it's done or idle, or ends |
| A pane stays `running` or `waiting` after its program was killed | The program reported in its terminal and never took it back, somewhere insensical can't see it end: through `ssh`, in a pane started with a command, or under a shell inside a shell. See "Programs that report by themselves" | Same as above |
| A program's reports aren't heard | It runs inside tmux or screen, which don't pass them on | Run it in the pane itself |
| A Gemini CLI pane stays `running` after a failed or cancelled turn | Gemini CLI has no event for either | The next prompt clears it, or run `isc status idle` |
| Hooks stopped reporting | `isc` moved since `integrate` | Run `isc integrate` for that agent again |
| Codex reports nothing | Its hooks aren't trusted | Run `/hooks` in Codex |
| The bell marks a pane but no notification arrives | The bell never notifies | Nothing |
| Nothing reaches your phone | A window, terminal or browser that can type is attached, so you count as there; or the command or address failed; or a `notify-` setting is mistyped | `isc notify --sinks` says how the last one went. Set `notify-command-when` or `notify-url-when` to `always` |
| `isc notify --test` says `https` can't be done on this processor | It lacks what the built-in `https` needs | Use `notify-command` with `curl` |
| A clicked notification selects the pane but the window stays behind | Wayland; see section 12 | Bring the window forward yourself |
| Notifications have no application name or icon | The desktop entry isn't installed | Install it |
| The tray's icon is blank | insensical's icons aren't installed | Install them |
| There's no icon in the tray | The desktop has no tray for status notifier items, or `tray` is off | On GNOME, add the AppIndicator extension |
| The leader does nothing | The desktop took `Ctrl+Space` | Pick another `leader` |

---

## 19. Reference

Look up a key, a setting, a command, a file or a variable here.

### Keys

A "—" means no default key: the command is in the palette, and you can bind it in
`[keys]`. The Command column is the name to use in `[keys]`. A capital letter after the
leader means the letter with Shift, written `shift-h` in `[keys]`.

| Action | Command | Linux | macOS | After the leader |
|---|---|---|---|---|
| New tab | `new-tab` | `Ctrl+Shift+T` | `⌘T` | `t` |
| New project | `new-project` | `Ctrl+Shift+N` | `⇧⌘N` | `o` |
| Find a project, tab or pane in the sidebar | `filter` | `Ctrl+Shift+L` | `⌘K` | `e` |
| Close pane | `close-pane` | `Ctrl+Shift+W` | `⌘W` | `x` |
| Close tab | `close-tab` | `Ctrl+Shift+Q` | `⇧⌘W` | `q` |
| Split right | `split-right` | `Ctrl+Shift+O` | `⌘D` | `S` |
| Split down | `split-down` | `Ctrl+Shift+E` | `⇧⌘D` | `s` |
| Focus the pane left, down, up, right | `focus-left`, `focus-down`, `focus-up`, `focus-right` | `Ctrl+Alt+arrow` | `⌥⌘arrow` | `h` `j` `k` `l`, or the arrows |
| Move the pane's edge left, down, up, right | `resize-left`, `resize-down`, `resize-up`, `resize-right` | `Ctrl+Alt+Shift+arrow` | `⌃⌘arrow` | `H` `J` `K` `L` |
| Move the pane left, down, up, right | `move-left`, `move-down`, `move-up`, `move-right` | — | — | `Ctrl+H` `Ctrl+J` `Ctrl+K` `Ctrl+L` |
| Next / previous pane | `next-pane`, `previous-pane` | `Ctrl+Shift+]` / `Ctrl+Shift+[` | `⌘]` / `⌘[` | `]` / `[` |
| Next / previous tab | `next-tab`, `previous-tab` | `Ctrl+Tab` / `Ctrl+Shift+Tab`; `Ctrl+PageDown` / `Ctrl+PageUp` | `⌃⇥` / `⌃⇧⇥`; `⇧⌘]` / `⇧⌘[` | `n` / `p` |
| Go to the project's first to ninth tab | `tab-1` to `tab-9` | `Alt+1` to `Alt+9` | `⌘1` to `⌘9` | `1` to `9` |
| Next / previous project | `next-project`, `previous-project` | `Ctrl+Alt+PageDown` / `Ctrl+Alt+PageUp` | `⌥⌘]` / `⌥⌘[` | `N` / `P` |
| Move the tab left / right | `move-tab-left`, `move-tab-right` | `Ctrl+Shift+PageUp` / `Ctrl+Shift+PageDown` | the same | `<` / `>` |
| Fill the tab with the pane, or stop | `zoom` | `Ctrl+Shift+Enter` | `⇧⌘↩` | `z` |
| Show or hide the tabs, wherever the settings put them | `sidebar` | `Ctrl+Shift+B` | `⌘B` | `b` |
| Float the pane, or put it back | `float` | `Ctrl+Shift+G` | `⇧⌘G` | `f` |
| New floating pane | `new-float` | `Ctrl+Shift+D` | `⇧⌘F` | `F` |
| Keep a float on top, or stop | `pin` | `Ctrl+Shift+K` | `⇧⌘K` | `i` |
| Show a float over every tab, or over its own | `everywhere` | — | — | `I` |
| Hide or show all floating panes | `floats` | `Ctrl+Shift+H` | `⇧⌘H` | `g` |
| Bring the scratch pane, or put it away | `scratch` | `` Ctrl+` `` | `` ⌃` `` | `` ` `` |
| Minimize the pane | `minimize` | `Ctrl+Shift+M` | `⌘M` | `m` |
| Restore the pane minimized last | `restore` | `Ctrl+Shift+R` | `⇧⌘M` | `r` |
| Go to the pane that most needs you | `next-attention` | `Ctrl+Shift+A` | `⇧⌘A` | `a` |
| Show only activity in the sidebar, or everything | `activity` | `Ctrl+Shift+Y` | `⇧⌘Y` | `y` |
| Go to the next pane in the activity list | `next-activity` | `Ctrl+Shift+J` | `⌃⌘]` | `}` |
| Go to the pane before in the activity list | `previous-activity` | `Ctrl+Alt+/` | `⌃⌘[` | `{` |
| Command palette | `palette` | `Ctrl+Shift+P` | `⇧⌘P` | `Space` |
| Put this project to sleep, or wake it | `sleep-project` | — | — | — |
| Rename this project | `rename-project` | — | — | — |
| Close this project and everything in it | `close-project` | — | — | — |
| Size the panes for this window, when another window or a terminal sized them last | `lead` | — | — | — |
| Name this pane | `name-pane` | — | — | — |
| Name this tab | `name-tab` | — | — | — |
| Show every key | `keys` | `Ctrl+Shift+/`; `Ctrl+F1` | `⇧⌘/` | `?` |
| Settings | `settings` | `Ctrl+,` | `⌘,` | `,` |
| See every tab of every project | `overview` | `Ctrl+Shift+Space` | `⇧⌘O` | `w` |
| Find a file and type its path | `pick-path` | `Ctrl+Shift+I` | `⇧⌘I` | `/` |
| Restart insensical, to move to a new version | `restart` | `Ctrl+Alt+Shift+R` | `⌃⇧⌘R` | `U` |
| Quit the window; what is running carries on. On macOS with insensical in the menu bar, put its windows away and leave it there | `quit` | — | `⌘Q` | — |
| Give the leader's key to the program | | | | the leader |
| Take the leader back | | | | `Esc` |
| Copy the selection | `copy` | `Ctrl+Shift+C` | `⌘C` | same chord |
| Select and copy with the keyboard | `copy-mode` | `Ctrl+Shift+X` | `⇧⌘X` | `v` |
| Paste | `paste` | `Ctrl+Shift+V` | `⌘V` | same chord |
| Paste what was last selected | `paste-selection` | `Shift+Insert`, middle button | — | same chord |
| Find text in the pane | `search` | `Ctrl+Shift+F` | `⌘F` | same chord |
| Scroll to the command before | `previous-prompt` | `Ctrl+Shift+Up` | `⌘↑` | same chord |
| Scroll to the command after | `next-prompt` | `Ctrl+Shift+Down` | `⌘↓` | same chord |
| Give every key to the program, or take them back | `lock` | `Ctrl+Alt+Shift+L` | `⌃⇧⌘L` | same chord |
| Open a web address | | `Ctrl`+click | `⌘`-click | |

These commands have no default key in either keymap:

| Command | What it does |
|---|---|
| `minimize-others` | Minimizes every pane except this one |
| `restore-all` | Restores every minimized pane |
| `float-left` | Floats this pane over the left half |
| `float-right` | Floats this pane over the right half |
| `float-centre` | Floats this pane in the middle |
| `float-fill` | Floats this pane over the whole tab |
| `mute` | Silences this pane's notifications, or stops silencing them |
| `protect` | Keeps agents in other panes from reading or typing into this pane, or lets them in again |
| `forget` | Forgets what this pane's program said it was doing, and puts the pane at rest |
| `select-all` | Selects everything in the pane |
| `integrate` | Gets the agents on this system to report what they're doing |
| `none` | Not a command. In `[keys]`, it leaves the key to the program. |

After the leader, these commands repeat without it for one second: `resize-left`,
`resize-right`, `resize-up`, `resize-down`, `move-left`, `move-right`, `move-up`,
`move-down`, `move-tab-left` and `move-tab-right`.

Resize mode is `resize-mode`: leader, then `Shift+R`. While it's on, `Left` or `H`, `Down`
or `J`, `Up` or `K` and `Right` or `L` move the edges of the pane in front. Escape or
Enter turns it off. See [Changing keys](#changing-keys).

Five commands have no key and are in the window only:

- `open-project-file` ("Open a project file") and `save-project-file` ("Save this project
  as a file"): see [A project in a file](#a-project-in-a-file).
- `share-project` ("Share this project") and `share-pane` ("Share this terminal"): see
  [Sharing one project or one terminal](#sharing-one-project-or-one-terminal).
- `add-machine` ("Add a machine"): see [Other machines in the
  window](#other-machines-in-the-window).

### Settings

Settings go in `config.toml`. Every key is optional, and an unknown key is an error.

| Key | Meaning | Default |
|---|---|---|
| `theme` | A theme name, or one for each appearance of the desktop: `{ light = "paper", dark = "fjord" }`. A name can be a built-in theme (see [Themes](#themes)), a file in `themes`, `ghostty`, or `ghostty:Name`. | `fjord` when the desktop is dark, `paper` when light |
| `font-family` | The terminal font. | `JetBrainsMono Nerd Font` |
| `interface-font` | Font for tabs, sidebar, headers and panels | the system's interface font |
| `interface-style` | `insensical`: greys around the terminals, with the theme's accent and state colours. `terminal`: the theme's colours (the switch Follow terminal theme in the settings). `isc tui` is `terminal` unless this is written. | `insensical` |
| `background-opacity` | How much of the desktop behind the window the background hides, 0.2 to 1. Below 1 the desktop shows through behind the text. | `1` |
| `background-blur` | Whether the desktop is asked to blur what shows through. | `true` |
| `motion` | Whether sheets, notices and moved panes fade in, the working mark turns and cursors blink. `system` follows the desktop; `on` and `off` decide. | `system` |
| `font-ligatures` | Whether symbols side by side are drawn as the font's ligature, such as `->` and `!=` | `true` |
| `font-size` | Terminal text size in points, 6 to 72. Interface text doesn't follow it. | `12`; on macOS `13` |
| `density` | `comfortable`: each pane is a rounded card with space around it. `compact`: panes sit edge to edge with a hairline between. | `comfortable` |
| `pane-gap` | Space between panes in pixels, 0 to 40 | `6`, or `0` when `density` is `compact` |
| `sidebar` | The levels listed under a project's name: `["tabs", "panes"]`, `["tabs"]`, `["panes"]`, or `[]` for names only | `["tabs", "panes"]` |
| `activity-idle` | Whether agents at rest are listed when the sidebar shows only activity | `true` |
| `tabs` | Where tabs show. `sidebar`: down the side, for every project. `top`: along the top, for the current project. | `sidebar` |
| `pane-header` | `full`: what the pane is and does, plus arrange buttons. `name`: the same without buttons. `hidden`: no header; a floating pane keeps its name. | `full` |
| `keymap` | `direct`: each command has its own chord. `leader`: each command is a key pressed after the leader, leaving the chords to the programs. | `direct` |
| `leader` | The key pressed before a command's key when `keymap` is `leader` | `ctrl-space` |
| `shell` | What a pane runs when you give it no command, with arguments: `"fish"`, `"/bin/zsh -l"`. | the login shell, `$SHELL` |
| `input-method` | Whether your system's input method may compose text in a pane, as for Bangla, Chinese or Japanese. | `false` |
| `option-as-alt` | On a Mac, whether Option with a key is Alt to the program, as in a shell's Alt+B and Alt+F, instead of typing the layout's character. | `false` |
| `type-ahead` | Whether what you type in a pane on another machine is drawn at once, a little fainter, before its program shows it. `auto`: when the round trip takes 70 milliseconds or more. `on`: over any link to another machine. `off`: never. See [Other machines, over ssh](#other-machines-over-ssh). | `auto` |
| `open-files-with` | What a file you `Ctrl`+click opens in: `system`, an editor by name (`code`, `cursor`, `zed`, `subl`, `idea`, `nvim`, `vim`, `hx`, `nano`), or a command with `{file}` in it. See [Web addresses and paths](#web-addresses-and-paths). | `system` |
| `term` | The `TERM` a pane's program is told. `xterm-ghostty` gives programs every feature. `xterm-256color` is known on every machine, which matters over `ssh`, and has fewer. | `xterm-ghostty` |
| `scrollback` | History each pane keeps, in megabytes, 1 to 200. | `10` |
| `keep-scrollback` | Whether what panes show is saved when the daemon stops, so it's above a new shell next time. With `false`, nothing is saved and earlier files are removed as panes close. | `true` |
| `agents-at-once` | How many agents one agent can have open at once, started with `isc agent start`, 1 to 64 (section 11). | `8` |
| `agents-deep` | How deep agents can start agents, 1 to 8. | `2` |
| `tmux-shim` | Whether a program in a pane finds a `tmux` that insensical answers, so an agent that opens helpers in tmux panes opens them as panes here (section 11). While on, `tmux` in a pane isn't the real one. | `false` |
| `clipboard` | Whether a program can put text on the clipboard. `focused`: only the program in the pane you're looking at. `never`: no program. No program can read the clipboard. | `focused` |
| `notifications` | Whether you're told when a pane has something for you. | `true` |
| `attention-glow` | Whether a waiting pane takes the waiting colour at its edge and casts a soft light into the gaps around it | `true` |
| `notify-waiting` | Whether you're told when a pane waits for you, or a program asks for you. | `true` |
| `notify-failed` | Whether you're told when a command or agent ends badly. | `true` |
| `notify-done` | Whether you're told when a long command or an agent's turn ends well. | `true` |
| `notify-command` | A program and arguments the daemon runs to tell you. See [When nobody is attached](#when-nobody-is-attached). | none |
| `notify-command-when` | When the command runs. `away`: only with no window, terminal or browser attached. `always`: every time. | `away` |
| `notify-url` | An `http` or `https` address the daemon posts to, to tell you | none |
| `notify-url-when` | When it's posted to: `away` or `always`, as for the command | `away` |
| `notify-url-body` | What's posted. `json`: the title, the message and the rest. `text`: the message alone, with the title in a `Title` header. | `json` |
| `notify-url-token` | A token sent as `Authorization: Bearer …` with what's posted. A secret, shown masked. | none |
| `notify-url-user`, `notify-url-password` | A user and password sent as Basic. The token wins if both are set. Secrets. | none |
| `notify-url-headers` | A table of headers sent with what's posted, for a token. Shown masked. | none |
| `long-command` | Seconds a command runs before its end is news: the pane is marked done or failed, and you're told if you weren't looking. 1 to 3600. | `30` |
| `sound` | Which notifications make a sound. `needs-you`: a pane that waits or failed. `all`: those, and a softer sound for a finish. `off`: none. | `needs-you` |
| `tray` | On Linux, whether insensical has a tray icon, marked while a pane needs you, with a menu of those panes. | `true` |
| `menu-bar` | On a Mac, whether insensical has a menu bar item that counts and lists the panes that need you. | `true` |
| `dock-badge` | On a Mac, whether the number of panes that need you is on the Dock icon. | `true` |
| `[tui]` | What `isc tui` does differently from the window, which never reads this table: `theme` (`terminal` is the terminal's own colours), `activity-idle`, `interface-style`, `motion`, `attention-glow`, `type-ahead`, `clipboard`, `keymap` and `leader`, each overriding the top-level one, plus `colours` (`auto`, `truecolor`, `256`, `16`) and `mouse` | `colours = "auto"`, `mouse = true` |
| `[keys]` | Keys and the command each one runs, or `none`, in either keymap | empty |
| `[modes]` | Modes of your own, and the resize mode: a table for each, with a `title`, a `timeout` and the keys that work in it. See [Changing keys](#changing-keys). | `resize` |
| `[icons]` | A program's name and its tile: a program that has one, a mark, or `{ mark = "docker", colour = "#ff8800" }`. See [Icons for your own programs](#icons-for-your-own-programs). | empty |
| `[commands]` | Commands of your own: a name and a list of steps. See [Changing keys](#changing-keys). | empty |
| `[keys.direct]` | The direct keymap, written out. It lies over the defaults: your lines win, a default you don't mention stays, and `none` takes a default key away. | the keys in the table above |
| `[keys.leader]` | The leader keymap, written out. Each key is the one you press after the leader. It lies over the defaults like `[keys.direct]`. | the keys in the table above |
| `[hosts]` | Other machines by name and where `ssh` goes for each: `build = "me@build.example.com"`. Also `{ ssh = "…", color = "…", icon = "…", careful = true }`: a colour (`#rrggbb` or `blue`, `green`, `orange`, `purple`, `pink`, `teal`, `gold`, `grey`, `rust`, `indigo`), an icon (`server`, `laptop`, `desktop`, `mini`, `nas`, `board`, `phone`, `container`, `cloud`), and a mark for one to be careful on. `isc host add` writes these, and `node = "…"` for a tailnet machine. See [Other machines, over ssh](#other-machines-over-ssh). | empty |

The file `isc config` prints sets `theme` to `{ light = "paper", dark = "fjord" }`, the
same as leaving it out. It shows `shell` and `interface-font` as lines to uncomment, and
writes out `[keys]`, `[keys.direct]` and `[keys.leader]` with their defaults.

For the theme file format, see [A theme file](#a-theme-file).

### `isc` commands

Run these in any terminal. Placeholders and options:

- `PANE` is a pane's number (`3` or `pane:3`) or its name. `TAB` is a tab's number (`7` or
  `tab:7`) or its name. A project is its name or number (`4` or `project:4`).
- Names match whatever their case. A name that fits more than one does nothing and exits
  with status 3.
- Where `[PANE]` is optional, the command acts on the pane you run it in, or outside any
  pane, on the pane with the keyboard.
- Every command accepts `--socket PATH`. Every command about a daemon also accepts `--host
  NAME` to act on a named machine's daemon: see [Other machines, over
  ssh](#other-machines-over-ssh).
- `--rights look,type` asks the daemon for those rights and no others, of `look`, `type`,
  `arrange` and `manage`: see [Who is connected](#who-is-connected).

| Command | What it does |
|---|---|
| `isc ls [--json]` | Lists projects, tabs and panes, a line each, with state, program, directory, name or title, what the state is about, whether it floats or is minimized, who opened it, and whether it keeps agents out. `--json` prints everything the daemon knows. `isc list` does the same. |
| `isc run [--cwd DIR] [--keep] [--name NAME] [--background] [--wait] [--json] [WHERE] [-- COMMAND…]` | Runs a command in a new pane, or the shell if you give none, and prints the pane as `pane:N`. `--cwd` sets the directory (default: where you run `isc`). `--keep` keeps the pane when the command ends; press Enter, or use `isc close`, to close it. `--name` names it. `--background` leaves the keyboard where it is. `--wait` waits for the command, prints the end of its output and exits with its status. `--json` prints the pane's and tab's numbers, and with `--wait` the status and output. Limits: 256 KB for a command and its directory, 4 MB for everything open, 4,096 panes. |
| `WHERE` for `isc run` | Nothing: a new tab in the current project. `--project NAME`: a new tab in that project. `--new-project [NAME]`: a new project for the working directory, called `NAME` if given. `--split right\|down [--beside PANE]`: beside a pane, sharing its space. `--float [--rect X,Y,W,H]`: floating above the current tab. |
| `isc split right\|down [PANE] [--keep] [--name NAME] [--background] [--wait] [--json] [-- COMMAND…]` | The same, beside a pane, in that pane's directory |
| `isc focus PANE`, `isc focus left\|right\|up\|down` | Gives a pane the keyboard: by number, or the focused pane's neighbour on that side |
| `isc float [PANE] [--rect X,Y,W,H]` | Lifts a pane out of the layout, or moves one already floating. The rectangle is in fractions of the tab; default `0.2,0.15,0.6,0.7`. |
| `isc tile [PANE]` | Puts a floating pane back into the layout |
| `isc pin [PANE] [--off]` | Keeps a floating pane above the others. `--off` stops that. |
| `isc follow [PANE] [--off]` | Shows a floating pane over every tab of its project. `--off` shows it over its own tab only. |
| `isc hide [PANE]`, `isc show PANE` | Hides a floating pane while its program keeps running; brings it back to the front |
| `isc scratch [--project NAME]` | Brings the project's scratch pane to the front, or puts it away if it's there with the keyboard. Starts it the first time. `--project` names a project other than the one in front. |
| `isc floats show\|hide` | Shows or hides the current tab's floating panes |
| `isc minimize [PANE]`, `isc restore PANE` | Puts a pane away, program running and size kept; brings it back where it was |
| `isc zoom [PANE] [--off]` | Gives a pane the keyboard and lets it fill its tab. `--off` stops that. |
| `isc swap A B` | Swaps the places of two panes |
| `isc put PANE left\|right\|above\|below [--of OTHER] [--tab TAB]` | Moves a pane to that side of another pane, or of all the panes of a tab |
| `isc resize left\|right\|up\|down [PANE] [--by PERCENT]` | Moves the divider a pane shares with its neighbours in that direction, by 5% of the divided space unless you set `--by`. |
| `isc rename PANE NAME` | Names a pane, shown instead of its directory and title. `""` removes the name. |
| `isc close [PANE]` | Closes a pane and ends its program |
| `isc close --opened [PANE]` | Closes every pane that `isc run`, `isc split` or `isc agent start` opened from inside the pane you run it in (or `PANE`), and those they opened, except one you've typed into or focused |
| `isc tab new` | Opens a tab with a shell in the current project |
| `isc tab next`, `isc tab prev` | Goes to the next or previous tab of the current project, wrapping round |
| `isc tab select TAB` | Goes to a tab |
| `isc tab close [TAB]` | Closes every pane in a tab, or the current tab |
| `isc tab move TAB INDEX` | Moves a tab to a position among its project's tabs, counted from 0 |
| `isc tab rename TAB NAME` | Names a tab. An empty name removes it. |
| `isc project new [DIR] [--name NAME] [--make]` | Opens a project for a directory, or the current one. It's named after the directory unless you set `--name`. With `--make`, a missing `DIR` is made first, with its parents, on the machine the command is for; if one can't be made, nothing is made and no project opens. |
| `isc project open [FILE] [--again] [--yes]` | Makes the project a file describes: see [A project in a file](#a-project-in-a-file). `FILE` is the file, or a directory holding `.insensical.toml`; default the current directory. An open project of the same name and root is gone to, unless `--again`. A file that starts programs asks first, and again when it changes; `--yes` answers for you. |
| `isc project save [FILE] [--force]` | Writes the project in front as such a file: `.insensical.toml` in the project's directory, or `FILE`, or `-` for standard output. It won't write over a file without `--force`. |
| `isc project select NAME` | Goes to a project |
| `isc project rename NAME NEW` | Renames a project |
| `isc project icon NAME [--mark MARK] [--colour #rrggbb] [--svg FILE]` | Chooses what a project is shown by. With none of them, it's the first letter of its name again. |
| `isc project move NAME INDEX` | Moves a project to a position among the projects, counted from 0 |
| `isc project close NAME` | Closes a project and everything running in it |
| `isc send PANE TEXT` | Types text into a pane, as if at its keyboard |
| `isc upload PANE FILE…` | Keeps files in the pane's directory, in `.insensical-uploads`, and types their paths into the pane |
| `isc key PANE KEY…` | Presses keys by name, in order: `ctrl-c`, `enter`, `escape`, `shift-tab`, `a` |
| `isc attach [PANE]` | Shows a pane in this terminal and lets you type into it. `Ctrl+]` leaves. |
| `isc`, `isc tui` | Draws the whole interface in this terminal. `isc` alone does so only for a person at a terminal; in a script, a pipe or one of insensical's own panes it lists the commands. `Ctrl+Space` then `D` leaves. See [The whole interface in a terminal](#the-whole-interface-in-a-terminal). |
| `isc capture PANE [--last [N]\|--screen] [--tail N] [--grep PATTERN [--context N]] [--since TOKEN] [--all]` | Prints the end of a pane's screen and scrollback as text, at most 4 KB, ending with a line that says what was left out and the token `--since` reads on from. `--last` is what the latest command printed and how it ended. |
| `isc ls [--program NAME] [--state STATE] [--opened] [--here]` | Lists only the panes that fit, under the projects and tabs that hold them |
| `isc wait PANE --prompt\|--quiet SECONDS\|--address\|--output PATTERN` | Waits until the running command ends (printing the end of its output and exiting with its status), until nothing is printed for a while, until a local address is shown (printing it), or until a line matches (printing it) |
| `isc agent start AGENT TASK [--split right\|down \| --tab] [--front] [--cwd DIR] [--name NAME] [--json] [--quiet [--timeout SECONDS]]` | Starts claude, codex or gemini with a task in a new pane and prints it; with `--quiet`, alone, printing only its answer. See section 11. |
| `isc agent wait PANE`, `isc agent reply PANE` | Waits for an agent's turn to end and prints its whole reply or its question; prints its last reply |
| `isc agent tell PANE MESSAGE`, `isc agent answer PANE yes\|no\|TEXT` | Gives an agent its next prompt, now or when its turn ends; answers its permission prompt |
| `isc ask PANE TEXT [--quiet SECONDS] [--anyway] [--timeout SECONDS]` | Types a line and Enter, waits for the answer and prints only it; at a shell that marks its commands, exits with the command's status |
| `isc history PANE [--all] [--json]` | Prints the commands run in a pane, newest last: when, how long, how each ended, where, and what was typed |
| `isc capture PANE --vt` | Prints all of a pane's screen and scrollback with colours and styles, as escape sequences |
| `isc status STATE [-m MESSAGE] [--resume COMMAND] [-p PANE]` | Reports what a pane is doing: `idle`, `running`, `waiting`, `done` or `failed`. `-m` (`--message`) says what it's about. `--resume` is a command that picks the work up again, offered at the pane's prompt after the daemon has stopped. `-p` (`--pane`) is required outside a pane. |
| `isc status explain [-p PANE]` | Explains why a pane is in its current state |
| `isc status clear [-p PANE]` | Forgets what a program or hook reported for a pane, and puts it at rest |
| `isc notify TITLE [BODY] [-p PANE]` | Asks for your attention on a pane's behalf |
| `isc notify --test` | Runs `notify-command` and posts to `notify-url` now, and says how each went |
| `isc notify --sinks` | Says what those two are set to, how the last send to each went, and how many weren't sent |
| `isc mute [PANE] [--off]` | Stops a pane sending notifications. `--off` lets it again. Its state still shows. |
| `isc protect PANE [--off]` | Keeps agents out of a pane: a program in another pane can't read it, wait on it or type into it, and `isc` there exits with 4. `--off` lets them in, and is refused in a pane. |
| `isc integrate [AGENT] [--remove] [--yes] [--force]` | Sets an agent up to use insensical (hooks, a skill and a note at each session's start), after showing you the change and asking. With no agent named, offers each one found. `--remove` takes them out. `--yes` skips the question. `--force` replaces a skill of that name you wrote. |
| `isc integrate [AGENT] --status` | Changes nothing. Prints, as JSON, for each agent whether it's found and set up, how its hooks, skill and note stand, and what's left for you |
| `isc guide [layout\|reading\|acting\|waiting\|agents]` | Prints the guide agents are taught: where they are, where to act, one topic at a time. |
| `isc wait PANE [--until STATE]… [--timeout SECONDS]` | Blocks until the pane reaches a state: `idle`, `running`, `waiting`, `done`, `failed` or `closed`. Without `--until`: `done`, `failed`, `waiting` or `closed`. Prints the state reached. Exits with 124 when time runs out. |
| `isc wait PANE --output TEXT [--timeout SECONDS]` | Blocks until the pane's screen or history shows the text. Exits with 124 when time runs out. |
| `isc events` | Prints what happens, one JSON object per line, until interrupted |
| `isc api [REQUEST]` | Sends the daemon one request as JSON, from the argument or standard input, and prints the answer as JSON |
| `isc api [--json]`, `isc api NAME [--json]` | Lists every request, or says what one holds, with an example. Needs no daemon. |
| `isc do NAME [--yes]` | Runs a command by name: one of your own from `[commands]`, or one of the window's. Stops at the first step that fails. `--yes` goes ahead where a step would end what's running. |
| `isc project sleep NAME` | Ends every program in the project and keeps its tabs, panes and what they showed. Doesn't ask. |
| `isc project wake NAME` | Puts a shell in each pane of a sleeping project, in the pane's directory |
| `isc project mute NAME [--off]` | Stops every pane of the project sending notifications, including panes opened later. `--off` lets them again. |
| `isc usage [--json]` | Prints what the programs in each pane use: memory, processor time and process count, with a total for each project |
| `isc clients [--json]` | Lists who is connected to the daemon: each client's kind, machine, version, rights, process, and the pane it's in or looking at |
| `isc disconnect CLIENT` | Puts another client off the daemon, by the number `isc clients` shows (`7` or `client:7`). Needs the right to `manage`. |
| `isc config` | Prints a settings file with every setting at its default and every key of both keymaps |
| `isc config get NAME` | Prints a setting's value from the file, or its default. With `--host`, from that machine's file. |
| `isc config set NAME VALUE` | Gives a setting a value, changing only that line. With `--host`, one of the daemon's settings on that machine. `--stdin` in place of `VALUE` reads it from standard input, out of sight of other users. |
| `isc config reset NAME` | Takes a setting out of the settings file, leaving its default. With `--host`, as `set`. |
| `isc completions bash\|zsh\|fish` | Prints what that shell needs to complete `isc` commands, including the panes, tabs and projects that exist |
| `isc shell-integration bash\|zsh\|fish` | Prints what that shell needs to tell its pane where each command starts and ends, and its exit status. fish 4 and later doesn't need it. |
| `isc host add NAME WHERE` | Names another machine. `WHERE` is what you'd give `ssh`. `--color C`, `--icon I` and `--careful` set those on it. A tailnet machine is kept by its node; `--no-node` keeps it as typed. |
| `isc host set NAME` | Changes what's set on a machine: `--ssh WHERE`, `--color C` or `--no-color`, `--icon I` or `--no-icon`, `--careful` or `--no-careful`, and `--name NEW` to rename it. Only what you give changes. |
| `isc host ls` | Lists the machines you've named, with where `ssh` goes and what's set on each. `--json` includes `node`. |
| `isc host answer NAME` | Runs your `ssh` to the machine at your terminal so it can ask: a host key, passphrase, password or sign-in. Then it meets the machine, and succeeds only if it was reached. `NAME` can be where `ssh` goes. |
| `isc host rm NAME` | Forgets a machine's name |
| `isc host check NAME` | Meets the machine and prints four lines: reached and round trip, a working insensical version, server running, and what's open. A failing line says what to do. It changes nothing there. |
| `isc host restart NAME [--yes]` | Restarts the server on a machine running an older version than its installed `isc`. It says what that comes to and asks; `--yes` doesn't. A server that can move keeps what runs in its panes. |
| `isc host put NAME` | Puts this version's `isc` in `~/.local/bin` on a Linux machine over `ssh`, with no sudo. It says what it would write and asks; `--yes` doesn't. A server running there is left alone unless you say `--restart-server` or answer yes to a second question. |
| `isc proxy` | What `isc --host` runs on the other machine, through `ssh`. You don't run it yourself. |
| `isc hub [--port N] [--listen ADDR] [--tls-cert FILE --tls-key FILE] [--name NAME]… [--guest-port N] [--guest-name NAME]…` | Serves insensical to a browser, on this machine only unless given a certificate. `--name` is another name browsers reach it by. `--guest-port` (7434) and `--guest-name` are the port and names of the door a public address leads to. See [In a browser](#in-a-browser) |
| `isc hub setup` | Walks you through reaching this computer from a browser or phone: hub or joined machine, how you'll reach a hub, starting by itself, and adding a device with its code. It ends on the equivalent commands. |
| `isc hub device add` | In a terminal, with no `--name`: opens a form that asks for the name and the rest, starting from the flags you gave, and ends on the code to scan |
| `isc hub device add --name NAME [--rights …] [--uses N\|any] [--expires 10m\|never] [--stays 1h\|1d\|always] [--only project:NAME\|pane:N] [--far] [--public] [--url BASE] [--json]` | Adds a device and prints its code once, as a link and a code to scan. The link pairs one browser for ten minutes unless told otherwise. `--stays` is how long the device stays paired. `--only` makes a guest that sees one project or terminal and may look, or look and type. |
| `isc hub device pair-again NAME [--expires TIME] [--url BASE] [--json]` | Makes a new one-use code for a paired device; the browser that pairs with it replaces the device's browsers |
| `isc hub device ls [--json]`, `isc hub device rm NAME [--with-approved]` | Lists the devices, with what each sees and which device added it; ends one and closes every browser paired with it, and with `--with-approved` the devices it added from its page |
| `isc hub device set NAME --far\|--no-far` | Lets a paired device open your other machines' terminals through this computer, or keeps it to this one |
| `isc hub push ls [--json]`, `isc hub push test [--at MACHINE] [--json]` | Lists the browsers sent notifications, and which hear your other machines; sends each a test and says how its push service answered. `--at` tests as from that machine. |
| `isc hub open [--print\|--json]` | Opens insensical in your browser on this computer, signed in as `This computer`, from an address that works once, for ten minutes. `--print` or `--json` prints it and opens nothing. |
| `isc hub status [--json]`, `isc hub stop` | Says whether `isc hub` is running and where; stops it |
| `isc hub tailscale on`, `off`, `status [--json]` | Has Tailscale serve `isc hub` to your tailnet at `https://` and this computer's name; takes that mapping away; says whether it's served, and why not |
| `isc hub autostart on`, `off`, `status [--json]` | Sets this computer up to start `isc hub` by itself (a user service, switched on, and lingering); switches it off, removing only a service it wrote; says whether and when it starts |
| `isc hub invite NAME [--url BASE] [--json]` | On the hub, prints a line that joins one machine to it, within ten minutes. See [Joining another machine to your hub](#joining-another-machine-to-your-hub) |
| `isc hub join`, `isc hub leave` | Joins this machine to the hub of a line `invite` printed, read from the terminal or standard input; leaves it, so `isc hub` listens again |
| `isc hub machine ls [--json]`, `isc hub machine rm NAME` | On the hub, lists the machines joined to it; takes one away at once |
| `isc hub ban ID`, `isc hub unban ID` | On the hub, carries nothing more for a device, by its id; lifts that |
| `isc hub rekey` | Gives this machine a new key and id: each device pairs with it again, and a joined machine joins its hub again |
| `isc hub public on`, `off`, `status [--json]` | Has Tailscale Funnel open a public address to devices made with `isc hub device add --public`; removes those devices and the address; says whether one is open, where, and for which devices |
| `isc tmux COMMAND…` | Answers a `tmux` command with insensical's panes, as the `tmux` a pane finds with `tmux-shim` on does. See section 11. |
| `isc daemon` | Runs the daemon in the foreground |
| `isc kill-server` | Stops the daemon and every program it runs, and returns once it's gone. The layout comes back the next time it starts. Also works on a daemon of another version. |
| `isc restart-server [--yes]` | Moves the running daemon to this version while the programs in its panes keep running. It asks first; `--yes` skips the question. |
| `isc version` | Prints this version and the running daemon's, with the protocol and terminal engine of each, the daemon's process number, and whether they can talk to each other |

### Files and directories

| What | Linux | macOS |
|---|---|---|
| Settings | `~/.config/insensical/config.toml` (`$XDG_CONFIG_HOME`) | the same |
| Your own themes | `~/.config/insensical/themes/<name>.toml` | the same |
| Drawings for projects' icons, kept by the daemon, and your own | `~/.config/insensical/icons/<name>.svg` | the same |
| Drawings a window was sent by its daemon | `~/.local/state/insensical/drawings/<name>.svg` | `~/Library/Application Support/insensical/drawings/<name>.svg` |
| Saved layout | `~/.local/state/insensical/state.json` (`$XDG_STATE_HOME`) | `~/Library/Application Support/insensical/state.json` |
| A saved layout that could not be read, set aside | `~/.local/state/insensical/state.json.unreadable` | `~/Library/Application Support/insensical/state.json.unreadable` |
| The highest number given to a pane, tab or project so far, or a little past it, so that none is ever given twice | `~/.local/state/insensical/numbers` | `~/Library/Application Support/insensical/numbers` |
| Which saved layout the numbers belong to: a random name that changes when the daemon starts with no layout to go on from | `~/.local/state/insensical/store-id` | `~/Library/Application Support/insensical/store-id` |
| What each pane showed when the daemon stopped | `~/.local/state/insensical/screens/<pane number>` | `~/Library/Application Support/insensical/screens/<pane number>` |
| Agents declined | `~/.local/state/insensical/agents-declined` | `~/Library/Application Support/insensical/agents-declined` |
| The project files you've said may start their programs, each as a number made of its path and what it holds | `~/.local/state/insensical/project-files-allowed` | `~/Library/Application Support/insensical/project-files-allowed` |
| A project written down, where you keep it | `.insensical.toml` in the project's directory | the same |
| The devices of `isc hub`: each one's rights, its code until it's used, and the keys of the browsers paired with it | `~/.local/share/insensical/hub/access.json` | `~/Library/Application Support/insensical/hub/access.json`  |
| The key `isc hub` signs notifications to browsers with, closed to other users | `~/.local/share/insensical/hub/vapid.key` | `~/Library/Application Support/insensical/hub/vapid.key`  |
| A hub's machines, the devices they keep and its open invites | `~/.local/share/insensical/hub/members.json` | `~/Library/Application Support/insensical/hub/members.json`  |
| The hub a joined machine dials, and its key | `~/.local/share/insensical/hub/joined.json` | `~/Library/Application Support/insensical/hub/joined.json`  |
| Where a running `isc hub` is | `hub.lock`, beside the daemon's socket | the same |
| Which port `isc hub public on` pointed the public address at, until `isc hub public off` has closed it | `~/.local/share/insensical/hub/funnel` | `~/Library/Application Support/insensical/hub/funnel` |
| That the settings sheet's switch left `isc hub` on, and the names it answers to, where no service keeps it started | `~/.local/share/insensical/hub/on` | `~/Library/Application Support/insensical/hub/on`  |
| That the settings sheet's Tailscale switch, or `isc hub tailscale on`, made the `tailscale serve` mapping, which is the only one either turns off | `~/.local/share/insensical/hub/serve` | `~/Library/Application Support/insensical/hub/serve`  |
| The window's size, whether its tabs were hidden, and what was folded in the sidebar | `~/.local/state/insensical/window` | `~/Library/Application Support/insensical/window` |
| What was chosen lately in the palette | `~/.local/state/insensical/palette-recent` | `~/Library/Application Support/insensical/palette-recent` |
| The terminal's description, when the daemon had to compile its own | `~/.local/state/insensical/terminfo/` | `~/Library/Application Support/insensical/terminfo/` |
| The host keys Tailscale holds for a machine of your tailnet that runs its own ssh server, which `ssh` is told to accept for it | `~/.local/state/insensical/tailnet-keys` | `~/Library/Application Support/insensical/tailnet-keys` |
| That the application has said, once, that it keeps running with no window | not written | `~/Library/Application Support/insensical/told-still-running` |
| Socket | `$XDG_RUNTIME_DIR/insensical/daemon.sock` | `$TMPDIR/insensical/daemon.sock` |

`INSENSICAL_CONFIG_DIR` replaces the settings directory. `INSENSICAL_STATE_DIR` replaces
the directory for the layout, screens and declined agents. `INSENSICAL_DATA_DIR` replaces
the directory the hub's folder is in. `INSENSICAL_SOCKET` replaces the socket.

The state directory is closed to other users (`0700`), and so is each file the daemon
writes in it (`0600`), because the layout names your directories and commands. If they're
open when the daemon starts, it closes them. It writes nothing in a state directory that
belongs to someone else.

insensical writes these files in other programs' directories, only when you tell it to,
and nothing else outside the directories above:

| Agent | File |
|---|---|
| Claude Code | `~/.claude/settings.json`, with a copy at `settings.json.before-insensical` (or under `$CLAUDE_CONFIG_DIR`) |
| Codex | `~/.codex/hooks.json`, with a copy at `hooks.json.before-insensical` (or under `$CODEX_HOME`) |
| Gemini CLI | `~/.gemini/settings.json`, with a copy at `settings.json.before-insensical` (or `$GEMINI_CLI_HOME/.gemini/`) |
| opencode | `~/.config/opencode/plugins/insensical.js` (or under `$XDG_CONFIG_HOME`) |
| pi | `~/.pi/agent/extensions/insensical.ts` (or under `$PI_CODING_AGENT_DIR`) |

### Environment variables

| Variable | Read by | Meaning |
|---|---|---|
| `INSENSICAL_SOCKET` | everything | The daemon's socket. Set inside every pane. |
| `INSENSICAL_PANE` | `isc` | Set inside every pane to its number, so commands run there know which pane they're in. |
| `INSENSICAL_STATE_DIR` | daemon, window | Where the layout, what panes showed and the declined agents are kept. |
| `INSENSICAL_CONFIG_DIR` | window, daemon | Where settings and themes are read from. |
| `INSENSICAL_SCOPE` | window, `isc` | `1` starts a daemon on a socket you named in a systemd scope of its own, as the one on the default socket is. `0` starts it in none. See [The daemon](#the-daemon). |
| `INSENSICAL_NOTIFICATIONS` | daemon | `clients` makes the daemon leave desktop notifications to its windows, and put no icon in the tray. |
| `INSENSICAL_WEB_FILES` | `isc hub` | A directory to serve the page from, in place of the files built into `isc`. For working on the page. |
| `INSENSICAL_ISC` | window, `isc` | The `isc` binary to start the daemon from. When set, it's used before the `isc` beside the program and the one on `PATH`. |
| `INSENSICAL_HOST` | `isc` | The name of another machine, as `--host` takes it. Every command about a daemon then acts on that machine's, except the ones that end something (`kill-server`, `close`, `tab close`, `project close`, `project sleep`), which want `--host` on their own line. `--socket` given on the command line comes first. The daemon doesn't pass it on to its panes. |
| `INSENSICAL_SSH` | `isc` | A program to run in place of `ssh` to reach another machine, with the same arguments. |
| `INSENSICAL_TAILSCALE` | `isc`, the application | A program to run in place of `tailscale`, with the same arguments. When set, no other place is looked in. |
| `INSENSICAL_SYSTEMCTL`, `INSENSICAL_LOGINCTL` | `isc`, the application | A program to run in place of `systemctl`, or `loginctl`, when `isc hub tailscale`, `isc hub autostart` and the settings sheet start and keep `isc hub` running, with the same arguments. |
| `INSENSICAL_ISC_X86_64`, `INSENSICAL_ISC_AARCH64` | `isc` | A file on this machine to send as `isc` when insensical is put on another machine with that processor. When set, it's sent in place of the release's file. It has to be an `isc` for Linux in the version you have here. |
| `SHELL` | daemon | What a pane runs when there's no command and no `shell` setting. `/bin/sh` if unset. |
| `TERMINFO`, `HOME` | daemon | Searched for Ghostty's terminal description. |
| `TERMINFO_DIRS` | daemon | Kept, after the daemon's own directory, when it points a pane at the description it compiled. |
| `XDG_CONFIG_HOME` | window, daemon, `isc` | Where the settings directory is, instead of `~/.config`. Ghostty's and opencode's directories are also looked for here. |
| `XDG_DATA_HOME` | `isc`, the application | Where the hub's folder is, under `insensical/hub`, instead of `~/.local/share`. Linux only. |
| `INSENSICAL_DATA_DIR` | `isc`, the application | A folder used in place of `insensical` under the data directory. The hub keeps its keys and devices in `hub` inside it. |
| `XDG_STATE_HOME` | daemon, window | Where the state directory is, instead of `~/.local/state`. Linux only. |
| `XDG_RUNTIME_DIR` | everything | Where the socket's directory is, on Linux. Where `ssh` runs `isc proxy` without it, `/run/user/` and your user's number is used if it's there, and otherwise `run` under the state directory. |
| `TMPDIR` | everything | Where the socket's directory is, on macOS. |
| `PATH` | daemon, window, `isc` | Every pane inherits the daemon's. The window's is searched for `isc` and for the program that `shell` names. `isc integrate` searches its own for agents. |
| `CLAUDE_CONFIG_DIR`, `CODEX_HOME`, `GEMINI_CLI_HOME`, `PI_CODING_AGENT_DIR` | `isc integrate` | Where each agent keeps its settings, instead of its default directory. |

Set in every pane: `TERM`, `COLORTERM=truecolor`, `TERM_PROGRAM=insensical`,
`TERM_PROGRAM_VERSION`, `INSENSICAL_PANE`, `INSENSICAL_SOCKET`.

Set for `notify-command`: `INSENSICAL_EVENT`, `INSENSICAL_TITLE`, `INSENSICAL_BODY`,
`INSENSICAL_KIND`, `INSENSICAL_PANE`, `INSENSICAL_PROJECT`, `INSENSICAL_MACHINE` and
`INSENSICAL_PROGRAM`. See [When nobody is attached](#when-nobody-is-attached).
