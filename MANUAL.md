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

A background server, the *daemon*, owns every program and its terminal. The window
(`insensical`) and the command line (`isc`) are its clients. Closing the window doesn't
stop anything. Open it again and everything is still there.

Panes, tabs and splits are real UI elements, not characters drawn inside one terminal.
A program's output goes to the window untouched. It isn't redrawn.

What it does:

- Keeps programs running when no window is open, and through an update to a new version.
- Brings back every project, tab, split, floating pane and minimized pane after the
  daemon has been stopped. Each pane comes back in its directory, showing what it showed
  before.
- Groups tabs into projects. A sidebar lists them as a tree of projects, tabs and panes.
- Lets you float any pane above the layout, or minimize it to a dock while its program
  keeps running, and rearrange the tiled ones by dragging.
- Shows the pictures programs draw by the Kitty graphics protocol.
- Shows on every pane whether its program is working, waiting for you, done or failed.
  It counts the panes that need you (waiting or failed), and one key takes you to the
  most urgent.
- Tells you when a pane starts waiting, finishes or fails, unless you're looking at it:
  with a notice inside the window while insensical is in front, and a desktop
  notification when it isn't.
- Shows what each project's programs hold in memory, and puts a project to sleep when
  you say so: its programs end, and its panes and what they showed are kept.
- Has every setting on one sheet, to be switched or picked without opening a file.
- Does everything the window does from the command line too.
- Draws the whole interface in a terminal, with `isc` alone and no window: on your own
  machine, on one with no desktop, or over `ssh`.

---

## 2. Getting it running

### What it needs

- Rust 1.98.1. The repository's `rust-toolchain.toml` selects it.
- Zig 0.16.x on `PATH`, to build the terminal engine.
- On Linux, what the window toolkit (gpui) needs: Wayland or X11 development files, and
  Vulkan.
- No particular font. The default is `JetBrainsMono Nerd Font`, and any font works: the
  UI draws its own icons, and the Nerd Font icons programs print come from an icon
  font that's part of the application.

On macOS, you need macOS 13.0 or later. To build it, you need the full Xcode
application, not only the Command Line Tools.

### Building

```sh
cargo build --release
```

This produces `target/release/insensical`, the application, and `target/release/isc`,
the daemon and command line.

- The first build downloads the Ghostty source. To use a local checkout instead, set
  `GHOSTTY_SOURCE_DIR`.
- The terminal engine is very slow when it's built without optimisation, so
  `.cargo/config.toml` builds it optimised even in development builds. Don't remove
  that.
- `just` lists the tasks. Among them are `build`, `release`, `run`, `isc …`, `test`, `lint`, `check`,
  `bench`, `banner`, `icons`, `install`, `install-local`, `install-mac`.
- `cargo test --workspace`, or `just test`, runs the tests. The window tests draw
  off-screen and write screenshots to `target/screenshots/`. No test puts anything on
  your desktop or reads your own settings or layout. The window tests can hang at the
  very end of a test, about one run in twelve. This is a fault in off-screen testing;
  the real application doesn't have it. With `cargo-nextest` installed (`cargo install
  cargo-nextest --locked`), `just test` runs the tests side by side, ends a test that
  hangs and runs it again. Without it, `just test` runs under a time limit: if it hangs,
  run it again.
- `just test-changed` runs only the tests your change can have broken. `just gate` runs
  everything, the page in a browser too.

### Installing

Releases are at <https://insensical.com/download>. Each file has its SHA-256 beside it.

**Arch Linux.** Install from the AUR with the helper you use:

```sh
yay -S insensical-bin
```

**Debian 12 and later, Ubuntu 24.04 and later**, on x86-64. Download the `.deb`, then:

```sh
sudo apt install ./insensical_*_amd64.deb
```

There's no apt repository, so to update, download the new version and install it the
same way.

The window is built for x86-64 only on Linux. On an ARM machine, install `isc` by
itself, below.

**Other Linux**, on x86-64 with glibc 2.36 or later. Unpack the tarball and put
`insensical` and `isc` together in a directory on your `PATH`. The tarball also holds
the desktop entry, the icon, the user unit and completions for bash, zsh and fish. Put
them where your system looks for them.

**Without the window, on any Linux**, on x86-64 or arm64, with a desktop or with none.
`isc` by itself is insensical with no window: it's the daemon, the command line and
[the whole interface in a terminal](#the-whole-interface-in-a-terminal), in one file. It
needs nothing installed and runs on any distribution, old or new. It's the same `isc`
the packages above install beside the window. One line installs it:

```sh
curl -fsSL https://insensical.com/install.sh | sh
```

- It downloads `isc-x86_64-linux`, or `isc-aarch64-linux` on an ARM machine such as a
  Graviton or Ampere server or a Raspberry Pi with a 64-bit system, and its checksum
  from the latest release. It installs the file only if the two agree.
- `ISC_VERSION=0.3.1` before `sh` installs that release instead of the latest:
  `curl -fsSL https://insensical.com/install.sh | ISC_VERSION=0.3.1 sh`. Use it to give
  another machine the version you have here; `isc --version` prints it.
- It installs to `/usr/local/bin/isc`, and asks for your password with `sudo` if it
  needs it. `ISC_DIR=~/.local/bin` before `sh` installs it there, with no password:
  `curl -fsSL https://insensical.com/install.sh | ISC_DIR=~/.local/bin sh`.
- Run the same line to update. If you already have `isc` somewhere on your `PATH`, the
  new one goes over it there.
- If a daemon from before is running, it asks whether to restart it, and Enter says
  yes. Restarting ends nothing in your panes. Say no and the daemon is left alone:
  `isc restart-server` moves it when it suits you.
- `ISC_QUIET=1` before `sh` has it ask nothing and restart nothing, and so does running
  it where there's no terminal to ask on, as a script over `ssh` does. It only puts
  the file in place. It won't wait for a password either: if the directory needs one,
  it stops and says so. Set `ISC_DIR` to a directory of your own, which is made if it
  isn't there.
- It won't write over `/usr/bin/isc`, which a package put there. Update that with your
  package manager.
- It has no completions with it. `isc completions bash`, `zsh` or `fish` prints them.
- Running a script straight from a website means trusting the website. The checksum
  comes from the same release as the file, so it catches a download that was cut short
  or damaged, not a release that is the wrong one.

The same by hand:

```sh
cd "$(mktemp -d)"
curl -fLO https://github.com/mah3uz/insensical-release/releases/latest/download/isc-x86_64-linux -O https://github.com/mah3uz/insensical-release/releases/latest/download/isc-x86_64-linux.sha256
sha256sum -c isc-x86_64-linux.sha256 && install -m755 isc-x86_64-linux ~/.local/bin/isc
isc
```

That address always gives the latest release. On an ARM machine the file is
`isc-aarch64-linux`. For one release, put `download/v0.3.1` where the address says
`latest/download`. A machine that can't reach the release needs only those two files,
copied to it: the third line is the same there. Replacing the file doesn't stop a daemon
that's running; `isc restart-server --yes` moves it to the new version with everything
still running.

**macOS 13 and later, on Apple silicon.** With Homebrew:

```sh
brew install --cask mah3uz/tap/insensical
```

Homebrew also puts `isc` on the `PATH`. Or open the `.dmg` and drag the application to
Applications. The application isn't signed with a Developer ID or notarised, so macOS
refuses to open a downloaded copy until its quarantine flag is cleared:

```sh
xattr -dr com.apple.quarantine /Applications/insensical.app
```

Homebrew clears it for you. `isc` is inside the application, at
`/Applications/insensical.app/Contents/MacOS/isc`. There is no build for a Mac with an
Intel processor.

**From the source, on Arch Linux.** `just install` builds the working tree, makes the
package `insensical-bin` from it and installs it with pacman, under `/usr`.
`just uninstall` removes it. If there's a copy under `~/.local`, it's found before the
packaged one until `just uninstall-local` removes it. In `packaging/arch`, `makepkg -si`
builds the package `insensical-git` instead, from the repository as it is on GitHub.

**From the source, on a Mac.** `just install-mac` builds `insensical.app`, puts it in
`/Applications` and links `isc` into `~/.local/bin`. If you have Homebrew, it also links
the completions for zsh, bash and fish where Homebrew's shells look for them, and a
shell you start afterwards completes `isc`. `just uninstall-mac` removes all of it.

The package also installs a user unit, but doesn't enable it. To start the daemon at
login, run `systemctl --user enable --now insensical`. You don't have to: the first
window or command that needs the daemon starts it.

Upgrading the package only replaces files. The running daemon carries on until you move
it to the new version (see [A new version](#a-new-version-installed-underneath)).

**From the source, elsewhere.**

```sh
just install-local
```

This installs:

- both binaries in `~/.local/bin`;
- a desktop entry in `~/.local/share/applications`, which names the binary by its full
  path, because a launcher doesn't always have `~/.local/bin` on its `PATH`;
- the icon in `~/.local/share/icons/hicolor`, as an SVG and as a PNG at each common size;
- completions for bash, zsh and fish.

To install somewhere else, use `PREFIX=/somewhere just install-local`. zsh needs the
completions directory in `fpath` before `compinit`.

`just uninstall-local` removes what `just install-local` installed. It leaves your
settings and the saved layout alone.

**What any uninstall leaves.** Removing insensical, by a package manager or by `just`,
takes the two programs, the launcher's entry, the icons and the completions. It leaves:

- your settings, in `~/.config/insensical`;
- the saved layout, what the panes showed, the window's size and what you chose lately
  in the palette, in `~/.local/state/insensical` (on a Mac,
  `~/Library/Application Support/insensical`);
- hooks added to an agent's own settings. `isc integrate AGENT --remove` takes them out,
  and has to be run before `isc` is gone;
- **a daemon that is running.** It carries on, with every program in its panes, though
  there's no longer an `isc` to stop it with. Run `isc kill-server` first if you want
  it ended.

Delete the two directories to be rid of everything.

### The two binaries must be together

The window looks for the daemon in three places, in order. If `$INSENSICAL_ISC` is set,
it starts the daemon from the file that names. Otherwise it uses the `isc` beside it.
If there isn't one, it uses `isc` on `PATH`. If it finds none of these, the window
can't start a daemon and says that the server cannot be reached.

### The icon

The icon is `logo.svg` in the repository: a rounded teal pane with two cursors for eyes, on a dark tile. A launcher shows it from the desktop entry.
A taskbar or switcher shows it for a window only if the desktop can match the window to
the entry, so the window names itself `insensical`.

- If you run it without installing, there's no entry to match, and the window gets a
  generic icon.
- `~/.local/share/icons/hicolor` may hold an `icon-theme.cache`. A cache written before
  the icon was added hides the icon. `just install-local` rebuilds the cache
  (`gtk-update-icon-cache --force --ignore-theme-index`) and the desktop database. A
  launcher or panel that's already running may keep the old picture until you restart
  it.

### Starting it

Start `insensical` from a launcher or a terminal. If no daemon is running, the window
starts one. You don't need to configure anything first. Without a settings file, the
defaults apply.

Where you start it from matters in two ways. The first project is made from the
directory you start it in (see [First launch](#first-launch)), and the daemon takes the
environment of whatever started it (see
[The daemon's environment](#the-daemons-environment)).

---

## 3. A first session

The keys here are the Linux defaults. Section 19 lists them all, along with the macOS
keys and the keys of the leader keymap.

1. **Open it.** Start `insensical`. The window opens with one project, one tab and one
   pane running your shell. It's an ordinary terminal, so type in it.
2. **Split.** Press `Ctrl+Shift+O` to split the pane to the right, or `Ctrl+Shift+E` to
   split it downwards. The new pane opens in the directory of the pane you split and
   takes half its space. Press `Ctrl+Alt+arrow` to move between panes. Drag the divider
   between two panes to resize them.
3. **Open a tab.** Press `Ctrl+Shift+T` to open a new tab with a shell in the project's
   root. If the project has no root, the shell opens in the directory of the pane you
   were in. `Ctrl+Tab` and `Ctrl+Shift+Tab` go to the next and previous tab.
4. **Open a project.** Press `Ctrl+Shift+N`. It asks for a name and a directory. Leave
   the name empty, type a directory that exists, such as `~/Projects/quote`, and press
   Enter. A project opens there, named after the directory. `Ctrl+Alt+PageDown` and
   `Ctrl+Alt+PageUp` move between projects.
5. **Float a pane.** Press `Ctrl+Shift+D` to open a new floating pane above the layout.
   Press `Ctrl+Shift+G` to float the pane you're in, or to put a floating pane back
   into the layout. Drag a float by its header. Resize it from any edge or corner.
6. **Minimize a pane.** Press `Ctrl+Shift+M` to put the pane away in the dock at the
   bottom of the window. Its program keeps running. To bring it back where it was,
   click its chip in the dock or press `Ctrl+Shift+R`.
7. **Find the pane that needs you.** In one pane, run a command that takes more than 30
   seconds, for example `sleep 40`, then move to another tab. When the command ends,
   the pane becomes `done`. Its tab, its sidebar row and the count at the top of the
   sidebar show it, and you get a desktop notification. Press `Ctrl+Shift+A` to go to
   that pane. Looking at it clears the mark.
8. **Close the window.** Everything keeps running. Open `insensical` again and it's all
   still there.

Two things to know from the start:

- **Closing a pane never asks**, whatever is running in it. Closing a tab or a project
  shows you what would end and asks first.
- `Ctrl+Shift+/` shows every key on one sheet, and `Ctrl+Shift+P` opens the command
  palette, which lists every command with its key.

---

## 4. Concepts

### Words

| Word | Meaning |
|---|---|
| Project | A named group of tabs with a root directory. It's usually one codebase. |
| Tab | One layout of panes inside a project. |
| Pane | One terminal running one program. |
| Placement | Where a pane is: *tiled* in the layout, *floating* above it, or *minimized*. Any pane can move between the three without restarting its program. |
| Dock | The strip at the bottom of the window that holds minimized panes. |
| State | What a pane is doing: `idle`, `running`, `waiting`, `done`, `failed`, `exited`. |
| Attention | A pane that needs you: one that's `waiting`, `done` or `failed`. |
| Daemon | The background process that owns everything. The window calls it the *server*; the command line and this manual call it the daemon. |

### The daemon

- The first window or `isc` command that needs the daemon starts it. It runs in its own
  session, so it outlives whatever started it. You don't need to enable anything at
  login.
- On Linux with systemd, a daemon that a window or command starts is put in a scope of
  its own (`insensical-daemon-…`, which you'll see in `systemctl --user`). Closing the
  terminal it was started from, or ending the session that started it, doesn't take it
  or your programs with it. A logout that stops your user manager does, unless lingering
  is on (`loginctl enable-linger`). If you use a different socket, set
  `INSENSICAL_SCOPE=1` to get the same; `INSENSICAL_SCOPE=0` turns it off.
- `isc daemon` runs it in the foreground. A second daemon on the same socket refuses to
  start and says "a daemon is already running". While it runs, a daemon holds a lock on
  a file beside its socket, `daemon.sock.lock`.
- It owns the projects, tabs, panes and their layout. Anything a window does to the
  layout, a command can do too, and each sees the other's changes immediately.
- It does no work while idle: no timers and no wakeups while nothing is happening.
- Only the same user can connect to it. The daemon listens on its socket and on no
  network. To use the daemon on another machine, go through `ssh`: see
  [Other machines, over ssh](#other-machines-over-ssh). To use it from a browser,
  start `isc web`, a separate program that listens on this machine only: see
  [In a browser](#in-a-browser). `isc` and the interface in a terminal can use
  another machine's daemon, and the window shows other machines' projects beside
  this computer's: see [Other machines in the window](#other-machines-in-the-window).
- **Any program running as you can drive the daemon through its socket**: it can open
  and close panes, type into them and read them.
- The directory that holds the socket must be owned by you and closed to others
  (`0700`). The daemon creates it that way. If the directory already exists with wider
  permissions, or is a link to another directory, the daemon refuses to start and says
  so. It also refuses if something that isn't a socket is where the socket goes.
- Socket paths have a length limit: about 108 bytes on Linux and 104 on macOS. A long
  `INSENSICAL_SOCKET` fails with "path must be shorter than SUN_LEN".
- If neither `INSENSICAL_SOCKET` nor `XDG_RUNTIME_DIR` is set (`TMPDIR` on macOS),
  there's nowhere to put the socket, and `isc` fails with "no per-user runtime
  directory".
- Stopping the daemon (`isc kill-server`, or a termination signal) kills every program
  in every pane and keeps the layout for the next start. Moving it to a new version
  doesn't stop it. Section 17 covers both.

### The daemon's environment

**The daemon has the environment of whatever started it, and every pane inherits it.**

- If a window opened from a launcher started the daemon, panes get the launcher's
  `PATH`, not a login shell's. A command that other terminals find might not be found.
- **On macOS, a new pane's shell starts as a login shell**, the same way the Mac's other
  terminals start it. So it reads the files that set `PATH` there (`/etc/zprofile`,
  `~/.zprofile`), even though the application itself started with almost nothing in it.
  This applies to the login shell only: a `shell` setting runs exactly as written, and
  needs its own `-l`.
- A variable you change in a shell profile reaches new panes only through the shell's
  own start-up files, or after you stop the daemon with `isc kill-server` and start it
  again from an environment that has the variable. Stopping the daemon kills every
  program.
- Moving the daemon to a new version doesn't change its environment, because it's the
  same process.
- One variable isn't passed on: `INSENSICAL_HOST`. If the shell that started the
  daemon had it set, panes don't get it, so `isc` in a pane means that pane's own
  daemon.
- Signals it was told to ignore aren't passed on either. A program in a pane starts
  with Ctrl-C, Ctrl-Z and a hang-up meaning what they usually mean, even if
  the daemon was started by a script's background job, which ignores some of them.
- **Started by the user unit** (`systemctl --user enable --now insensical`), the daemon
  has the environment of systemd's user manager, which reads no shell profile. `PATH` is
  usually short there, and variables you export in `~/.profile` or a shell's start-up
  file are missing. `systemctl --user show-environment` prints what it has. To add to
  it, run `systemctl --user import-environment PATH` before the unit starts, or put
  `Environment=` lines in a drop-in made with `systemctl --user edit insensical`.

---

## 5. The window

### First launch

- If no daemon is running, the window opens one project with one tab and one pane
  running the shell.
  - If you start it in a directory other than your home directory, that directory
    becomes the project, and the project is named after it.
  - If you start it from a launcher, it starts in the home directory (on a Mac, in `/`).
    The project then has no root and gets a generated two-word name such as
    `amber-fjord`.
  - A pane in a project with no root, and with no directory set for the pane, starts in
    the home directory.
- If a daemon is already running, the window shows what the daemon has.
- The window opens centred, at the size it had when it was last open, or 1200×800 the
  first time. It doesn't remember where on the screen it was.
- Each `insensical` process has one window. Two processes give you two windows on the
  same daemon; see [Two windows](#two-windows).
- On Linux, the window asks the desktop's settings portal whether the desktop is dark
  before it opens, so it doesn't open light and then turn dark. If the desktop has no
  portal, or the portal doesn't answer within half a second, the window uses the
  toolkit's answer, and you might see a flash of the other appearance.

### Parts

- **Sidebar**, on the left, 248 pixels wide: a row of tools, a field, then the project
  tree. It's shown with `tabs = "sidebar"`.
- **Top bar**, shown with `tabs = "top"`: the sidebar button, the project's name, the
  current project's tabs, and a `+` for a new tab. On the right: a settings problem if
  there is one, the update notice if there is one, the count of panes that need you, and
  a count and switch for floating panes when the tab has any. The top bar also shows the
  notice about agents that don't report yet (section 11).
- **The tab's area**: tiled panes with dividers between them, and floating panes above
  them.
- **Dock**, at the bottom: shown only while the tab has minimized panes.

When this manual says something is shown "in the top bar" (a settings problem, the
update notice, the notice about agents, the count of panes that need you, the switch
for floating panes, `+` for a new tab), that's where it is with `tabs = "top"`. With
tabs down the side, the same things are at the top and bottom of the sidebar.

Every button is an icon. Hover over one to see what it does and, where a command does
the same thing, that command's keys in the current keymap. A pane's header buttons show
their keys on the focused pane only, and a tab's close button on the front tab only,
because those are what the keys act on.

**Tabs run along the top or down the side, never both.** The `tabs` setting chooses
which. Press `Ctrl+Shift+B` to hide them, and again to show them in the same place.

- `tabs = "sidebar"`, the default: the sidebar lists every tab of every project, so it
  suits watching work spread across several projects. There's no top bar. Everything the
  top bar holds besides tabs is at the top of the sidebar: the count of panes that need
  you, the switch for floating panes, and `+` for a new tab. A settings problem, the
  notice about agents and the update notice are at the bottom, when there are any.
- `tabs = "top"`: the top bar shows only the current project's tabs, which suits working
  in one project. There's no sidebar.
- **When hidden, neither is shown** and the panes get the whole window. The count of
  panes that need you and the notices are then off screen. `Ctrl+Shift+A` still goes to
  the pane that needs you, and the palette still lists them.
- The window remembers that you hid the tabs, and opens with them hidden.

### With nothing open

A window with no pane to show says so, and lists four ways on, each with its key: a new
tab, a new project, the command palette and the sheet of every key. Each is a button.

A setting that couldn't be used is said in red at the foot of the sidebar, or in the top
bar. Click it to open the settings sheet, which lists every such problem.

### What resizes terminals and what does not

Showing or hiding the sidebar, the dock appearing or disappearing, and hiding pane
headers all change how much room terminals have, so the terminals are resized. Changing
the `sidebar`, `pane-header`, `density` or `pane-gap` setting does the same. These count
as you changing the layout.

Nothing else that covers terminals resizes them: the palette, the overview, search,
zoom, the leader's panel and the restart question.

### What the window does not remember

The window remembers its size, whether the tabs were hidden, and which projects and
tabs you folded away in the sidebar: the next window opens with them folded. It doesn't
remember its position. A new window always opens with its keys unlocked, so that it
never opens answering no key but one. Two windows open at once share
what's remembered, and the one you changed last decides it. A desktop
that tiles its windows gives the window whatever size it chooses.

### Two windows

A pane has one size, and two windows of different sizes can't both have it fit. **A pane
is the size that fits the window you used last.** The same goes for a window and
`isc attach`.

- Come to a window, by clicking it or switching to it, and the panes it shows take its
  size. The other window keeps showing them, at that size.
- **A pane that another window sizes says so**: its header has a "Sized elsewhere" mark.
  Press the mark to size the pane for this window without leaving it.
- The pane is shown at its real size, cells as large as ever. If it's larger than its
  space you see its foot, where the cursor is. Hold Shift and turn the wheel to move
  across it, or Alt and the wheel to move up and down it. A sideways swipe moves across
  it too. If it's smaller, the rest of the space is empty.
- A window that stops showing a pane, by going to another tab or closing, no longer has
  a say in its size. The pane then fits whichever window still shows it.
- With pane headers hidden there's no mark. Everything else is the same.

### Other machines in the window

One window shows this computer's projects and those of every machine you've named.
A pane on another machine is used like any other: you type in it, split it, float
it, minimize it. It runs there, and goes on running when you close the window or
the connection drops. It all goes through your own `ssh`: see
[Other machines, over ssh](#other-machines-over-ssh) for what that needs.

**Adding a machine.** Press "Add" at the foot of the sidebar and choose Machine.
"Add a machine" in the palette does the same, and so does "Add Remote Host" under
Remote Hosts in the settings sheet.

- Under the Address field are two tabs, `Tailscale` and `SSH config`, and under them
  the list of the one that's chosen. `Tailscale` lists the machines of your tailnet,
  each by its name there, with its system and whether it's on. The ones that are on
  come first; one that's off is listed after them, dimmer, and you can still choose
  it. `SSH config` lists the names in your `~/.ssh/config`, each with where it leads.
  The dialog opens on `Tailscale` when your tailnet has other machines, and on `SSH
  config` when it doesn't.
- A tab with nothing to list says why in one line: Tailscale isn't installed on this
  computer, Tailscale isn't running, there are no other machines on your tailnet, or
  there are no names in your ssh config.
- Type to narrow the list. Anything else you type is given to `ssh` as it is: a host,
  or `user@host`. That works on either tab. Tab moves the keys from the field to the
  two tabs and back, and Left and Right switch between them; Up and Down move through
  the list.
- A machine you choose from `Tailscale` is kept by its node, so it's dialled at the
  address it has now, whatever its name becomes. So is one you type by a name of your
  tailnet. One you choose from `SSH config` goes where your configuration says.
- Enter meets the machine, in front of you, as four lines that are marked as they
  pass: whether it was reached, whether insensical is there, whether its server
  runs, and what's open there. They're the lines `isc host check` prints. A line
  that fails says what to do under it.
- **If `ssh` has a question, press `Answer`.** A machine `ssh` hasn't met, one that
  wants your key or password, or a sign-in Tailscale wants: the first line says which,
  and the button is `Answer`. It opens a new tab on this computer that runs `isc host
  answer` for that machine, where `ssh` asks and you answer `ssh` itself. insensical
  never asks for the secret and isn't given what you type. When the command ends well
  the machine is met again by itself; if it doesn't, nothing changes, and the tab
  stays so you can read what `ssh` said. The same button stands under the machine's
  name in the sidebar and on its row of Remote Hosts in the settings sheet. A host key
  that changed has no `Answer`: its words say what to find out first.
- A machine that's ready is added at once, named after where `ssh` goes for it.
  Its projects appear in the sidebar under its name.
- **If insensical isn't there, or is older, one button puts it there.** The button
  says what it does, "Put insensical 0.3.0 on atlas", and under the lines you read
  what it will write: `~/.local/bin/isc`, how large it is, no sudo, nothing else.
  Nothing is written until you press it. It does what `isc host put` does.
- **Once it's being put there, it isn't stopped halfway.** The card says "Hide", not
  "Cancel", and Escape or a click outside puts it away while the file goes on, or the
  server restarts. A notice says how it ended, "atlas is ready" or why it wasn't, and
  the card comes back by itself if something is asked, such as whether to restart the
  server. Pressing "Add Remote Host" brings it back too. While it's only looking,
  Escape ends it, and nothing was written.
- **A server already running there is asked about separately.** After the file is
  there you're asked whether to restart the server, and told what that comes to:
  either everything in its panes carries on, or what runs in them would end. "Leave
  it running" leaves it.
- A Mac, or a processor other than x86-64 and arm64, isn't given insensical from
  here. You're shown the line to run there.
- It can't ask you for a password or a passphrase. A machine that wants one says so
  on its first line, with what to do.

**The sidebar.** With no machine named, the sidebar is as it always was. Once one
is named, each machine is a heading over its projects: "This computer" first, then
the others in the order your settings name them.

- A heading has the machine's tile and name, the mark of its link, and how many of
  its panes need you. Press it to fold the machine away. Folded, it shows the most
  urgent state of what it holds.
- **A machine's name is shown as you wrote it**, in the machine's colour, under a
  thin line that sets its section apart. A name too long for its place loses its
  middle, not its end: `atlas-build-server-eu-west` and `atlas-build-server-us-east`
  stay told apart. Point at it for the whole name. The same goes for the chip on a
  pane and for what the window says of a machine.
- **The mark of the link is three bars.** Three under 60 ms for a round trip, two
  under 200 ms, one beyond that. Point at it for the figure in words, as in
  "atlas · 142 ms · typing is drawn ahead" when what you type there is being drawn
  before the machine shows it. It's struck through and red while the machine is out
  of reach, and pointing at it says why.
- A machine that couldn't be reached when the window opened has its heading and
  the struck-through mark. The window doesn't wait for it.
- **A machine with another version of insensical says so under its heading.** It's
  reached, but its server and this window can't speak, and trying again by itself
  would change nothing, so the window doesn't. You read how the two stand, as in
  "atlas has insensical 0.2.1, and this computer has 0.3.0", over one button.
  "Update it" opens the meeting, which offers to put this version there. If the
  machine has the newer one, you're told to update this computer, and "Try again"
  reaches it afresh once you have. Point at the words for everything its server
  said.
- The filter finds panes, tabs and projects on every machine, and a machine's
  name finds everything on it. A machine with nothing that matches is left out.
- Going to the pane that needs you walks every machine's panes: the most urgent
  first, then in the sidebar's order. The count on the activity button is of all of
  them, and a machine's heading counts its own.
- "Next project" and "Previous project" go through every machine's projects.
- The overview has each machine's projects under its name, and the palette says
  which machine a pane or a project is on.

**Which machine you're typing into.** Every pane on another machine says so in its
own header, where your eyes are when you type.

- The header begins with the machine's chip: its tile and its name, in its colour.
  A line in the same colour runs along the header's top. In a narrow pane the chip
  is the tile alone, and pointing at it gives the name.
- This computer's panes carry neither. With one machine nothing is added anywhere.
- A machine's colour comes from its name unless you set one. It's the same on its
  heading, its chips, and what the window says of it.
- **A machine marked careful has a filled chip, and a paste of more than one line
  there asks first**, whatever the program in the pane asked for. See
  [Pasting](#pasting). Mark the machine you'd least like to paste into by mistake.

**One machine is in front at a time.** The panes you see are one machine's: the
one whose pane, tab or project you last went to. Keys and commands go there.

- **A pane, a tab or a project stays on the machine that runs it.** Dropped on
  another machine in the sidebar, it stays where it was, and one line says why.
- "New project" asks which machine, once there's more than one: the machine in
  front is chosen, and `Alt+M` goes to the next. The directories it offers are that
  machine's. A machine that's out of reach, or whose server can't be spoken to,
  isn't offered until it can be asked again.
- **What you ask of a machine is never done on another.** If the machine can't be
  asked, nothing happens anywhere and one line says so, as in "atlas is out of
  reach." If you forget a machine while something was open for it, a new project
  or the choice of an icon, that closes and a line says why.
- What a machine says while you're at the window is said in the window, with the
  machine's name. With the window behind others, or closed, it goes to the desktop,
  and the notification's title begins with the machine's name.
- **A question stays about the machine you asked it of.** If you're asked whether
  to close a tab and another machine comes to the front before you answer, say
  because you picked one of its notifications, your answer still closes the tab
  you were asked about. The path finder closes when the machine in front changes,
  since the pane it was typing into is no longer shown.
- **Only you change the machine in front.** Another machine can tell you that a
  pane needs you. It can't bring itself to the front, raise the window, close it,
  or take the keyboard from the pane you're typing in.

**When the link is lost.** A train, a closed lid, a machine that restarts.

- The machine's panes keep what they last showed, dimmed, with a line at their foot:
  "Link to atlas lost · trying again". After ten seconds it also says how long ago
  the machine was last heard.
- **Nothing you type there is sent.** A key pressed at such a pane isn't kept for
  later either: the line at its foot beats once, so you know it wasn't taken, and
  the machine is tried again at once. Commands that act on that machine do the same.
  The same goes for the moment after the link is back and before the machine's
  server has been spoken to: a key then is refused, not lost.
- The link is tried again after 1 second, then 2, 4, 8, 16, and every 30 seconds
  after that. Coming back to the window tries every machine at once, and so does
  this computer waking from sleep, on Linux.
- The machine's heading stays where it is, with what it held.
- When it's back, and its server has been spoken to, the panes you see are drawn
  afresh and say "Connected" for a second.
  What ran there never noticed.
- **A link is lost after four seconds in which nothing came back from the
  machine**, though `ssh` itself takes about twenty to give up on it. A link that
  was lost is never picked up again: coming back is a new link, and the panes are
  drawn afresh.
- **What you type gets there within about two and a half seconds, or never.** In
  the seconds before the panes say the link is lost, your keys are still taken.
  Those typed in the first second and a half go into the network; the later ones
  wait here. If the machine answers in time, they arrive, as after any stall. If
  it doesn't, the ones that waited are dropped, and the other machine throws away
  whatever reaches it after two and a half seconds of silence. So nothing you typed
  into a dead link turns up at an agent when the network returns.
- A machine that goes on answering and takes nothing you send, as when its server
  has stopped, loses its link within about ten seconds. The window and the other
  machines' panes carry on meanwhile.
- One thing can't be undone: a key typed in the last moment before a link dies may
  already have arrived, though you never saw its echo.
- Pasting a great deal into a program that prints nothing, over a line slower than
  about 60 KiB a second, can lose the link: nothing comes back for four seconds
  while it's carried.

**Settings.** The settings sheet has a page, Remote Hosts, with a card of rows for each:

- Its colour: press for the next of ten, and back to the one its name gives.
- Its icon: laptop, desktop, server or cloud.
- "careful", on or off.
- "Check" meets it again, as adding it did, and offers the same things.
- The pencil renames it in its row, and the settings stay open: the name becomes a
  field, Enter keeps it, Escape leaves it. The arrow keys and Enter reach the pencil as
  they reach the controls beside it. `Forget` forgets it. A renamed machine stays as it was: its
  link, its folds and whether it's in front don't change. Forgetting changes nothing
  on the machine: what runs there runs on, and adding it again finds it.
- If you save the settings file with a mistake that stops it being read at all, the
  machines stay as they were, careful ones included, and the mistake is listed at the
  top of the sheet. They go only when the file reads and doesn't name them.
- From the keyboard: Up and Down go from one machine to the next and to "Add Remote
  Host", Left and Right go along a machine's controls, and Enter presses the
  one that's marked.
- The settings a server reads (the shell, scrollback, notifications and the rest
  of that kind) and the agents' hooks are each machine's own. In Server,
  Terminal, Notifications, Agents and Changed, a chooser at the top right says whose
  you're looking at, and `Alt+M` goes to the next. Everything else on the sheet is this window's.

What it doesn't do:

- **It doesn't move a pane or a project between machines.**
- **It doesn't show two machines' panes side by side.** One machine is in front.
- **A fold on another machine is remembered by the machine's name.** Rename the
  machine and the window you open next has its folds forgotten.
- **The interface in a terminal shows one machine at a time.** There, a machine you
  marked careful has its name filled in the top row, with `· careful` after it where
  there's room, and a paste of more than one line asks first, as in the window. In
  "Go to a machine", the same word stands by its row.
- **In "Go to a machine", `a` answers what `ssh` asks.** A machine `ssh` has a
  question for says `ssh asks` on its row, and why under the list. Press `a` on it
  and a new tab opens on this computer running `isc host answer` for it; from another
  machine you're taken to this computer first. Open the list again afterwards and the
  machine is tried anew. A machine that wasn't reached for another reason says why,
  whole, and `a` does nothing there.
  It stays careful until the settings file says otherwise. If you save the file with
  a mistake in it, or rename the machine or take it out while it's shown, the mark
  stays and a notice headed "Still careful" says why. A `careful` that's neither
  `true` nor `false` counts as `true`.

**When this computer's server stops**, only this computer's part of the window
says so. Its heading in the sidebar gets the struck-through mark and "its server
has stopped" under it, and its panes are gone until it starts. The other machines
carry on: their panes stay, and you go on typing in them. Press this computer's
heading, or go to one of its projects, for the way to start its server again.

### Appearance

- The UI around the terminals uses the system's own proportional font at 13 pixels,
  whatever `font-size` is. On Linux that's the family fontconfig gives for `system-ui`,
  looked up once when the application starts. Use `interface-font` to name another. If
  you name the terminal's font, the UI uses the terminal's typeface.
- Icons in the UI are drawings, not letters of a font, so they look the same whatever
  font you use. Icons a program prints in a terminal come from the terminal's font if
  it's a Nerd Font, and otherwise from the icon font that comes with insensical (Nerd
  Fonts' symbols), so `font-family` doesn't have to be a Nerd Font.
- An icon a program prints is usually drawn wider than one cell. With a blank after it,
  it keeps that width. With a letter or another icon right after it, it's drawn smaller,
  so it stays in its own cell and nothing is drawn over.
- **A program is shown as a tile**: its own mark (Claude's, Neovim's, Docker's, Git's
  and the rest) on a small glossy card in the program's colours, like an application's
  icon. A program insensical has no mark for gets a terminal's. The colours belong to the program and not
  to the theme, so Neovim is green and Claude Code is coral in every theme, and you can
  find a pane by colour before you read its name. The tile is in the sidebar, the pane
  header, the tab strip, the dock, the overview and the palette.
- **The foot of the sidebar** has the window's tools: Settings, Show every key, and the
  overview of every tab, and at its other end "Add". Press it for a small menu:
  Project, or Machine. While the menu is open, `P` and `M` choose, the arrows and
  Enter work too, and Escape closes it. With the sidebar hidden, the three tools are
  at the right end of the top bar.
- **A tab is shown as a deck**: the tile of the pane that has the keyboard, or had it
  last, in front, and the tiles of up to two more of its panes behind it like cards.
  Floating and minimized panes count. A tab with one pane shows one tile. The deck is in
  the sidebar's tab rows and on the tabs along the top; hover over a tab at the top to
  see how many panes it holds.
- `density = "comfortable"`, the default, draws each pane as a rounded card with space
  around it. `compact` puts panes edge to edge with a hairline between them.
- `pane-gap` sets the space between panes directly, in pixels from 0 to 40. If you
  leave it out, it's 6 when comfortable and 0 when compact.
- The focused pane has an accent-coloured border. Unfocused panes aren't dimmed.
- **Little moves.** A sheet, a question and a notice fade in over a moment, and a pane
  you float, put back, bring back from the dock or zoom clears from a light wash so you
  can see where it went. A blinking cursor blinks, and the mark of a pane that's working
  turns. Nothing slides, and what a terminal shows is never animated.
- `motion = "off"` stops all of it: nothing fades, the working mark stands still and no
  cursor blinks. `"system"`, the default, follows the desktop. On Linux that's GNOME's
  `enable-animations` setting, read once as the window starts; on a desktop without it,
  motion is on. On macOS it's Reduce Motion.
- A state is never shown by colour alone: each has a mark and a word.
- Terminal content isn't exposed to screen readers.

---

## 6. Projects and the sidebar

### Projects

A project is a name, an optional root directory, and tabs. It stays until you close it,
even with nothing running in it, and comes back after the daemon restarts.

**Closing a tab or a project asks first.** Closing either one kills every program in it,
so the window lists the panes that would close, each with what it's running, its
directory and its state, and waits. Enter, or "Close tab" or "Close project", closes.
Escape, "Keep it", or a click outside closes nothing.

- With more than six panes, the window counts them instead of listing them all.
- A project with nothing running in it closes without asking.
- Closing a pane doesn't ask, and neither does `isc tab close` or `isc project close`:
  typing the full command counts as your yes.

**Opening one.** Press `Ctrl+Shift+N`, or click the project's name in the top bar.
You're asked for two things:

- **Name**, which you can leave empty. A project with a directory and no name is named
  after the directory. With neither, it's given a name.
- **Directory**, where its tabs open. It starts as the directory of the pane you're in.
  Type a path that begins with `/` or `~`:
  - Below the field are the directories that what you've typed could become, six at
    most. Hidden ones appear only once you type a dot. Up and Down select one.
  - `Tab` goes into the selected one, so entering a path takes a few letters and `Tab`
    at each step.
  - If the field is empty, the project has no directory.
  - If the path isn't a directory but could be one on the machine the project is for,
    this computer or another, the card says so
    in one line, with the whole path it worked out (`~` is your home, `..` is taken),
    and the button reads "Make the directory". Enter, or the button, asks once more:
    "Make it and create". Enter again makes the directory, and the ones above it that
    aren't there either, and opens the project in it. Typing anything in between
    takes the question back. If it can't be made, because you may not write there or a
    file is in the way, the card says why in one line, in the system's words, and
    leaves nothing made. The server of the machine makes the directory, and `~` and
    `..` are worked out there, so on another machine it's made there and not here. It
    makes at most 32 new directories at once, in a path no longer than the system
    allows, and says so and makes nothing when you ask for more. Each is made with
    permissions 755 less your `umask`: you can write in it and others can only look.
  - If the path isn't a directory and can't be made, the window says so, and Enter does
    nothing.
- `Tab` in the name field moves to the directory field; `Shift+Tab` moves back. Clicking
  either field moves to it. Enter, or "Create", makes the project and opens a shell in
  it. Escape, "Cancel", or a click outside makes nothing.

**From the sidebar's field.** Press `Ctrl+Shift+L` to put the keyboard in the field at
the top of the sidebar. This shows the sidebar if it was hidden. With `tabs = "top"`
there's no such field, and the key opens the palette instead, which finds the same
things. The field only narrows the tree: typing the name of an existing project and
pressing Enter goes to that project, and nothing typed there makes a project. Make one
with `Ctrl+Shift+N`, the `+` button or the palette.

From the command line, use `isc project new [DIR] [--name NAME] [--make]`, or
`isc run --new-project`. With `--make`, a `DIR` that isn't there is made first, with the
ones above it, by the server of the machine the command is for. To open a project with
all its tabs and panes at once, see [A project in a file](#a-project-in-a-file).

**Where new panes open.** A new tab opens in the project's root. In a project with no
root, it opens in the directory of that project's front pane, and in the home directory
only when there's no such pane. A split opens in the directory of the pane it splits. A
new floating pane opens in the focused pane's directory.

**A project's name does not follow its panes.** A project named `insensical` is still
called `insensical` even if all its shells have changed directory to
`~/Projects/quote`. The tab and pane rows show where they really are.

**Renaming.** Press the pencil that appears on the project's sidebar row, or use
`isc project rename OLD NEW`, or "Rename this project" in the palette. The name on the
row becomes a field holding the name, selected: type to replace it all, Enter keeps
what's there, Escape leaves the name as it was, and a click elsewhere keeps it. You
can't give a project an empty name. If the row isn't on screen (the sidebar is hidden,
or the filter has left it out) a small card asks instead.

**A project's icon.** Every project has a tile beside its name, in the sidebar, in the
top bar and in the overview. Without your doing anything it's the first letter of the
name, on a colour that comes of the name, so the same project always looks the same.

- To choose, press the tile on the project's sidebar row: a pencil shows on it under
  the pointer. Pick
  one of ten colours, or "By name" for the one it had. Pick a mark from the same set
  programs are shown by, and type to find one: `git`, `rust`, `docker`. The first tile
  in the list is the letter again. Each pick is made at once, and you see it in the
  sidebar behind.
- **"Your SVG" takes a drawing of yours.** Choose a `.svg` file, 256 KB at most.
  It's drawn whole in a square, so a square drawing fills the tile and any other is
  fitted inside it. The daemon keeps a copy in `icons` beside its settings
  (`~/.config/insensical/icons/`, under the file's name and a number made of what it
  holds), so the file you chose can be moved or deleted, and two projects can each have
  a `logo.svg` of their own. Only SVG is taken. The copy is deleted when no project is
  shown by it any more: you chose another icon, reset it, or closed the project. The
  daemon deletes only copies it made. A file you put in `icons` yourself is never
  deleted, whatever it's called.
- The file is read where you chose it and sent to the daemon, so
  `isc --host build project icon api --svg logo.svg` takes a `logo.svg` from this
  machine. A window is sent the drawing by the daemon and keeps it in `drawings` in
  its state directory, so it asks only once.
- A drawing you put in `icons` yourself, under a name of your own, still shows. A
  window asks the daemon for that one each time it starts.
- A drawing that shows a picture from another file (an `<image>` in it) isn't taken,
  and you're told why. Draw it with shapes, or put the picture's own shapes in it.
- A drawing has to be a file of text. One packed with gzip (an `.svgz` given the name
  `.svg`) isn't taken: unpack it first.
- A window keeps the drawings it was sent, 128 of them and 8 MB for each machine. Past
  that the ones shown longest ago are removed, and the next window fetches them again.
- "Reset" goes back to the letter and the name's colour.
- From the command line: `isc project icon NAME --mark rust --colour '#ff8800'`, or
  `--svg logo.svg`. What you don't give is left out, so `isc project icon NAME` alone
  is the reset.
- The icon belongs to the project: it's kept with the layout, and every window shows it.

**Putting projects in order.** Drag a project's row onto another's in the sidebar, or
use `isc project move NAME INDEX`, counted from 0.

**Closing.** Press the cross on the project's row, which asks first, or use
`isc project close NAME`, which doesn't. This closes every pane in the project.

**Moving between projects.** Press `Ctrl+Alt+PageDown` or `Ctrl+Alt+PageUp`, click a
sidebar row, or use `isc project select NAME`.

**On the command line**, you refer to a project by its name, or by its number as
`isc ls` prints it (`4` or `project:4`). Two projects can have the same name. The name
then means the first one listed, and the number tells them apart. A number always
means the project with that number, never a project whose name is that number: a
project called `2024` is reached by its own number.

### A project in a file

You can write a project down: its tabs, the panes in each, and the program each pane
runs. **`isc project open` then makes the whole project in one go.** Keep the file in
the project's directory, call it `.insensical.toml`, and commit it. Everyone who clones
the repository gets the same project with the same command.

```sh
isc project open                      # the .insensical.toml in this directory
isc project open ~/code/shop          # the one in that directory
isc project open ~/layouts/shop.toml  # a file by name, kept anywhere
```

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

**A morning's work.** Two tabs, two agents each in its own directory, a floating pane,
and a pane that starts minimized:

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

**What a file can say.** Every key is optional. A key it doesn't know is a mistake,
not something it skips.

| Where | Key | What it means |
|---|---|---|
| The project, at the top | `name` | The project's name. Without it, the project is named after its directory. |
| | `root` | The directory its panes start in. Without it, it's the directory the file is in, which is what you want for a file kept in a repository. A path that doesn't begin with `/` is taken from where the file is, and `~` is your home. |
| A tab, under `[[tab]]` | `name` | The tab's name |
| | `front = true` | This tab is the one in front. Without it, the first tab is. |
| | `columns = [ ]` | Its panes side by side, left to right |
| | `rows = [ ]` | Its panes stacked, top to bottom. A tab has `columns` or `rows`, not both. |
| | `run`, `dir` | For a tab of one pane, without a list: what it runs and where. A tab with none of these is one shell. |
| A pane, in a list | `"command"` | The command, in quotes. `""` is a shell. |
| | `{ run = "command" }` | The same, with room for the keys below |
| | `dir` | Where it starts: a directory from the root, or a whole path |
| | `name` | The pane's name |
| | `size` | Its share of the list's room. A pane of size 2 has twice the room of a pane of size 1, which is what a pane has without it. |
| | `front = true` | This pane has the keyboard. Without it, the tab's first pane has. |
| | `minimized = true` | It starts in the dock, with its program running |
| A list in a list | `[ ]` | Panes going the other way: rows inside columns, columns inside rows. Nest as deep as you need. |
| | `{ rows = [ ], size = 2 }` | The same when the group needs a size, or `{ columns = [ ] }` |
| A floating pane, under `[[tab.float]]` | `run`, `dir`, `name`, `front`, `minimized` | As for any pane |
| | `at = [x, y, width, height]` | Where it floats, in parts of the tab from 0 to 1. Without it, `[0.2, 0.15, 0.6, 0.7]`. |
| | `pinned = true` | It stays above the floating panes that aren't pinned |
| | `everywhere = true` | It shows over whichever tab of the project is in front |

**A command is words, not a line for a shell.** `run = "cargo watch -x test"` starts
`cargo` with three arguments. Quotes inside keep a word whole:
`run = "sh -c 'npm run dev | tee dev.log'"`. Nothing else means anything: no `&&`, no
`$HOME`, no `~`. For those, run a shell as the example does, or write the words as a
list: `run = ["sh", "-c", "make && ./serve"]`. A pane closes when its program ends, as
with `isc run`.

**It asks before it starts someone else's programs.** A file that came with a
repository you cloned is somebody else's list of programs to run on your machine. The
first time you open a file, `isc` lists every program it would start, with the directory
of each, and asks. Only `y` starts them.

- Your yes is remembered for that file at that path, exactly as it was. If one
  character of it changes, or it's copied somewhere else, you're asked again.
- `--yes` skips the question, and counts as your yes for next time too.
- Where there's nobody to ask (a script, a pipe), a file you haven't said yes to isn't
  opened: `isc` lists what it would start, says to use `--yes`, and fails.
- A file that only opens shells starts nothing of anyone's, so it isn't asked about.
- A no isn't remembered. You're asked again next time.

**Opening it twice doesn't make two.** If a project with the file's name and root is
already open, `isc project open` goes to it, changes nothing in it, and says so. That's
what you want from a command you run every morning. `--again` opens a second one
anyway.

**A mistake makes nothing.** The whole file is read and checked before anything is
opened. A misspelt key, a value of the wrong kind, broken TOML or a `dir` that isn't
there is reported with its line and what belongs there, and no tab is made, not even
the ones that were right:

```text
isc: /home/me/shop/.insensical.toml, line 5: a pane has no `sise`. It has `run`, `dir`,
`name`, `front`, `minimized`, `size`. Nothing was opened
```

If the daemon refuses something partway (a program that isn't installed, say), what was
made of the project is closed again and you're told why.

**Saving one.** Arrange a project the way you like it, then:

```sh
isc project save              # writes .insensical.toml in the project's directory
isc project save shop.toml    # or a file you name
isc project save -            # or prints it
```

It writes the project in front: every tab and pane, the splits with their sizes,
floating and minimized panes, names, and which tab and pane are in front. A file saved
in the project's own directory names no `root` and no path of your machine, so it works
for whoever has the directory. It won't write over a file that's there unless you say
`--force`.

- **Each pane is saved with the command it was started with**, by `isc run`, by a file
  or by the window. A pane that was started as a shell is saved as a shell, whatever
  you typed into it since. When such a shell is running something, `isc project save`
  tells you which pane, so you can give it a `run` in the file yourself.
- Sizes are saved as the smallest whole numbers that say them, to within half a
  percent: a split dragged to two thirds is saved as `size = 2` beside a pane with no
  size.
- Not saved: the scratch pane, a zoomed pane, floating panes you hid, the project's
  icon, and what is muted.

**In the window.** You don't need a terminal for any of this.

- **Make a new project (`Ctrl+Shift+N`) in a directory that holds `.insensical.toml`,
  and the window opens it as the file arranges it.** If the file starts programs, the
  window first lists each one with its directory, exactly as `isc project open` does,
  and asks. "Open as written" (Enter) opens the file. "Just the directory" (`D`) makes
  an ordinary project with one shell and leaves the file alone. Escape makes nothing.
- **Your yes is the same yes as on the command line.** A file you've allowed in either
  place opens in the other with no question, and a file that has changed asks again.
  A file that only opens shells isn't asked about.
- **"Open a project file" in the palette** asks for a directory, as a new project
  does, and opens the file in it. If there's no file there, it says so and makes
  nothing.
- **"Save this project as a file" in the palette** writes the project in front to
  `.insensical.toml` in its directory. If one is there already, it asks before writing
  over it.
- **A mistake in the file is shown in a notice with its line, and nothing is made**,
  not even the ordinary project.
- The window always makes a new project from the file, even when one from the same
  file is open. That's what `--again` does on the command line.
- If you type a name when you make the project, it's used when the file gives none.

**Limits.**

- The window opens and saves a file on this computer only. For a project on another
  machine, use `isc project open` and `isc project save`.
- The window opens `.insensical.toml` in a directory. A file with another name is
  opened with `isc project open FILE`.
- The window doesn't tell you which saved shells are running something. `isc project
  save` does.
- A file starts 100 panes at most, and can be 1 MB at most.
- With `--host`, the file is read on this machine and the project is made on the other
  one. `root` must then be written in the file as a whole path on that machine, and
  the directories aren't checked before the project is made. `isc project save` needs
  the file's name there, and always writes `root`.
- A pane whose program ends the moment it starts closes at once, like any pane. If it
  was the first pane of a tab, opening stops there and says so.
- A file can't ask for a pane to stay open after its program ends.

### Putting a project to sleep

A project you aren't working on still holds whatever its programs hold: a few megabytes
for a shell, several hundred for an agent. **Putting it to sleep ends its programs and
keeps everything else.**

- Its tabs, splits, floating and minimized panes stay as they are, and so does what
  each pane showed, which is written to disk.
- Nothing runs in it and it holds no memory. In the sidebar its heading says `asleep`
  and its rows are dimmed.
- **Waking it** puts a shell in each pane, in the pane's directory, under what the pane
  showed. What ran there is typed at the prompt for you to run again with Enter: an
  agent's own resume command if it gave one, otherwise the command the pane was
  started with. This is what happens to every pane when the daemon restarts
  ([section 17](#17-stopping-restarting-and-new-versions)).
- A project that's asleep when the daemon stops is asleep when it starts.

To put a project to sleep: hover over its heading in the sidebar and press the sleep
mark; or choose "Put this project to sleep, or wake it" in the palette for the project
in front (the command is `sleep-project`, with no default key); or
`isc project sleep NAME`. The window shows what would end and asks first. `isc` doesn't
ask.

To wake it: select the project and press Enter or `Wake it`; or the same mark, command
or `isc project wake NAME`.

- insensical never puts a project to sleep by itself.
- You can't start a pane in a sleeping project. Wake it first.
- Closing a sleeping project removes it and what was kept of its panes.
- **With `keep-scrollback = false` nothing is written to disk**, so a project that
  wakes has its panes and directories but not what they showed.
- A program that ignores being hung up is ended a third of a second later.

### The sidebar

The sidebar is a tree: project, tab, pane. It's shown while `tabs = "sidebar"`. Press
`Ctrl+Shift+B` to hide it, and again to bring it back.

| Row | Shows |
|---|---|
| Project | A fold mark and the name, as a heading over its rows: brighter for the current project. On the right: what its programs hold in memory, and how many panes it holds. When it's folded, this is instead the number of its panes that need you, with the mark and colour of the most urgent, if any do. |
| Tab | A fold mark, the tab's deck of tiles, then the tab's name if it has one, otherwise what its focused pane is called. When it's folded, on the right: the mark of its most urgent pane. |
| Pane | The program's icon, then what the pane is called. On the right: a mark if it's floating or minimized, and its state. |

**What a pane is called** is the same in the sidebar, on a dock chip, in the palette and
in a question about closing:

- the name you gave it, if you gave one;
- otherwise the title its program set, unless the program is a shell. An agent's title
  says what it's working on;
- otherwise the program and the last part of its directory, such as `nvim · api`. A
  shell is called by its directory alone, such as `api`: its icon already says it's a
  shell. The home directory is written `~`.

The pane's own header still shows the title and the whole directory.

- **A tab that holds one pane and has no name has no row of its own.** Its pane's row
  stands in its place, one level up. Split the pane or name the tab and the tab's row
  appears. To name such a tab, choose "Name this tab" in the palette.
- **A pane with something to say says it on its row,** on a second line under its name.
  The line starts with the state as you'd say it: "wants permission", "asks a
  question", "needs you to sign in", "waiting", "failed" or "done". That part is in the
  state's colour. The program's own words follow, and they wrap to a second line before
  they end in "…"; whatever is still cut is shown whole in a hover over the words.
- **A working pane has a second line only when its words don't fit beside its name.**
  If its program said something, how far along it is, a line of words, or both, it's
  after the pane's name on the same line when it fits, small and quiet, as in "40% ·
  Pushing image". When it doesn't fit, the words go on a line of their own under the
  name, like a waiting pane's. A pane that's only running a command has its turning
  mark and its name. So does a pane at rest.
- **The pencil never covers the words.** Under the pointer it takes a place of its own
  at the end of the name's line, and it's the name that gets shorter to make room. The
  words, the age and the state's mark stay in view.
- **A row says how long its pane has needed you.** A pane that's `waiting`, `failed`
  or `done` shows it beside its mark: `now` for the first minute, then `2m`, `1h`,
  `3d`. It changes once a minute at most, never by the second. Once a pane has waited
  ten minutes for you, the figure turns bold and takes the waiting colour. A pane
  that's working or at rest shows no time.
- A pane whose program reports several things, and has no words on its row, shows
  their count beside its name, such as "3 tasks".
- **When you answer a waiting pane, its row clears.** The waiting colour washes off
  the row over most of a second as the pane goes back to work. With `motion = "off"`
  nothing washes.
- A row with a second line is taller, so the rows below it move down while it has one.
  Only a pane that needs you or has finished unseen has one, and the rows of working
  panes never change height.
- Click a row to go to it. Click a minimized pane's row to bring the pane back.
- Each project stands a little apart from the one above it.
- The rename and close buttons take up room on the row even when they're invisible, so
  a long name is cut a little sooner than the row's width suggests.

**Sharing.** An eye appears beside the pencil on a project's row under the pointer. It
shares that project with a guest, or lists who it's shared with: see "Share one project
or one terminal" under [In a browser](#in-a-browser). A project that's shared shows an
eye on its row all the time, or a keyboard if a guest can type in it.

**Naming.** A pencil appears at the end of the row under the pointer. Press it and the
name becomes a field holding the name, selected: type, and press Enter. Escape leaves
the name as it was, and a click elsewhere keeps what you typed. For a tab or a pane, an
empty name removes the name. The daemon keeps names: they survive a
restart and appear in the tab strip, the pane's header, notifications and `isc ls`. From
the command line, use `isc tab rename TAB NAME` or `isc rename PANE NAME`. From the
palette, choose "Name this pane" or "Name this tab" (`name-pane`, `name-tab`; neither
has a default key): the pane's header, the tab's capsule along the top or the tab's
sidebar row becomes the field, and where the name isn't written on screen, such as a tab
that is one pane with no name, a small card asks instead.

**Folding.** The mark at the start of a project or tab row hides what's under it.
The next window opens with them folded again. While the filter field has text, its matches
are shown even inside something folded.

**Filtering.** Type in the field to narrow the tree to the projects, tabs and panes
whose name, title, program or directory contains what you typed. Case doesn't matter.

**What is listed.** The `sidebar` setting chooses which levels appear under a project's
name: `["tabs", "panes"]` (the default), `["tabs"]`, `["panes"]`, or `[]` for project
names alone.

**Showing only activity.** Press the button with the pulse line at the head of the
sidebar, or choose "Show only activity in the sidebar, or everything" in the palette
(`activity`, `Ctrl+Shift+Y`), and the sidebar lists only the panes that are agents
or have something to say. Shells, editors and monitors go, and so does a project with
nothing left in it. Press it again and everything is back. The window remembers which
you chose. The number beside the button is how many panes need you: the ones that ask
something and the ones that failed.

Each row is a pane: its name, how long it has been as it is, and under the name its
state in words and what its program says, as `wants permission · Allow the edit?`. In
each project the panes that need you come first, then those at work, then those that
finished, then agents at rest. Projects and machines keep the sidebar's order. A machine
with nothing to list says "nothing at work" under its heading.

The list is kept quiet on purpose, so a row isn't always what its pane says this second:

- A row keeps saying "working" through a pause of under 4 seconds, and its words change
  at most once a second. A question shows at once.
- Work that a program reports gets a row after it has gone on for 2 seconds. A command
  you run at a shell that marks its commands (`isc shell-integration` sets bash and zsh
  up to) gets one after 10 seconds, with what you typed as its words and how long it has
  run, so a build or a
  deploy is listed while it goes and an ordinary command never is. When it ends, the row
  becomes done or failed by its exit if it ran long enough to be told of
  (`long-command`); a shorter one just goes. A program that takes the whole screen, an
  editor or a pager, isn't a command that works. A pane whose program reports its own
  state, or an agent, is listed by that and never by its shell's marks. At a shell that
  doesn't mark its commands, a command gets no row until it ends.
- A pane that finished or failed stays listed until you look at it. Then its row dims,
  and goes 5 seconds later.
- A pane that waits stays listed until its program says otherwise. Going to it doesn't
  clear it. If the program is gone and the row is stale, point at the row and press the
  arrow that turns back, "Forget what this pane's program said it was doing".
- While your pointer is over the list and has moved in the last 2 seconds, no row
  appears, leaves or changes place. What a row says still changes where it stands.
- Rows move together, a second after the last change and at most 3 seconds after the
  first.
- A pane that finished more than a day ago and was never looked at is dimmed and listed
  last in its project.

The filter field narrows this list as it does the full one, and the arrows walk it.
Folds and the `sidebar` setting don't apply to it: a pane with something to say is
always shown. Set `activity-idle = false` to leave out agents that are at rest, so the
list empties as they finish.

Two commands walk the list from anywhere, round at the ends and across machines:
`next-activity` (`Ctrl+Shift+J`) and `previous-activity` (`Ctrl+Alt+/`). While the sidebar
shows activity, `Ctrl+Shift+A` goes down the list to the next pane that needs you or has finished, in
the order drawn, and not to the most urgent one first.

With `tabs = "top"` there is no sidebar. The button is in the bar, and the list is
shown beside the panes while it's on, which takes room from them as showing the sidebar
does.

The interface in a terminal has the same list, kept by the same rules: see
[The whole interface in a terminal](#the-whole-interface-in-a-terminal).

The page that `isc web` serves has this list too, by the same rules: see "In a browser".
There a finger on the list counts as the pointer does. The page always lists agents at
rest (`activity-idle` is the window's and the terminal's), and it has no filter field and
no arrow to forget what a program said.

**From the keyboard.** Press `Ctrl+Shift+L` (leader `e`) to put the keyboard in the
field. Down and Up then move a highlight through the rows in the order they're listed
or, after you've typed, through the rows that match. Left folds the highlighted project
or tab, and Right unfolds it. Enter goes to the row and gives the keyboard back to the
pane. With no row highlighted, Enter goes to the project whose name you typed, if there is one. Typing clears the
highlight, because the rows have changed. Escape leaves the field.

**Drag a row to put it in order.** A project dragged onto another project takes its
place, and a tab dragged onto another tab of the same project takes that tab's place. A
tab with one pane and no name is shown as that pane's row, and that row is dragged as
the tab. The row under the pointer is outlined while you carry one over it; dropped
anywhere else, nothing moves. Panes aren't put in order here: their order is the
layout's.

---

## 7. Tabs

A tab is one layout of panes. It exists as long as it holds a pane: when you close its
last pane, the tab closes too.

| To | Do |
|---|---|
| Open a tab | `Ctrl+Shift+T`, the `+` in the top bar, or `isc tab new`. The new tab opens a shell in the project's root. If the project has no root, it opens where the pane in front is. |
| Switch | Click the tab; `Ctrl+Tab` / `Ctrl+Shift+Tab`; `Ctrl+PageDown` / `Ctrl+PageUp`; `Alt+1` to `Alt+9` for the project's first to ninth tab (`⌘1` to `⌘9` on macOS, leader `1` to `9`); `isc tab next`, `isc tab prev`, `isc tab select TAB`. |
| Close | The cross that appears on the tab, a middle click on the tab, `Ctrl+Shift+Q`, or `isc tab close [TAB]`. The window asks first. |
| Move | `Ctrl+Shift+PageUp` / `Ctrl+Shift+PageDown`; `isc tab move TAB INDEX`, counted from 0; or drag the tab onto another, along the top or in the sidebar. |
| Name | The pencil on its sidebar row, "Name this tab" in the palette, or `isc tab rename TAB NAME`. |

- **Closing a tab closes every pane in it.** The window shows what would end and asks
  first; `isc tab close` doesn't ask.
- When you drag a tab onto another, it takes that tab's place and the others shift along.
  While you carry a tab over another, the tab under the pointer is outlined. If you drop
  it anywhere else, nothing moves.
- A tab in the strip shows its deck of tiles, with its focused pane's program in front,
  and its name or what that pane is called
  ([section 6](#6-projects-and-the-sidebar)). It shows a state mark only
  when one of its panes needs you.

---

## 8. Panes

### The life of a pane

A pane runs one program: the shell, or the command you give to `isc run` or `isc split`.

- **When its program ends, the pane closes.** A pane started with `isc run --keep` stays
  open and readable, and shows how the program ended (`done`, or `failed` with "exited
  with N"). Press Enter in it to close it. Its header says "Enter closes".
- **Closing a pane always ends its program.** The program is sent a hang-up. After
  300 ms its terminal closes. After three seconds its process group is killed.
  **There is no confirmation, even if something is running.**
- Close a pane with `Ctrl+Shift+W`, the close button in its header, or
  `isc close [PANE]`.
- When the focused pane closes or leaves the layout, the keyboard goes to the pane that
  takes its space.
- A fault inside one pane closes only that pane.
- **A window or a terminal shows 512 panes at once at most.** Any more are tried again
  by themselves, a quarter of a second later at first and then less often, up to every
  eight seconds, until others are closed or put away. In the window such a pane says
  "Not shown yet" and why.
- A pane whose terminal has more than 1,024,000 cells in all, 2000 columns by 512 rows
  for one, isn't shown.

### Which shell runs

If you don't give a pane a command, it runs the first of these that applies:

1. the `shell` setting, if it's set: a program and its arguments, split at spaces, such
   as `"fish"` or `"/bin/zsh -l"`. You can't give a path that contains a space;
2. `$SHELL` as the daemon sees it;
3. `/bin/sh`.

- The daemon reads the setting each time a pane starts, so a change applies to the next
  pane without a restart. A pane that's already open keeps the shell it opened with.
- The setting applies to panes opened from the window, to panes opened by `isc`, and to
  panes brought back after the daemon has been stopped.
- If the shell you named can't start, the login shell starts instead and the top bar
  says what went wrong.
- **The `shell` setting changes what new panes run, not `$SHELL` inside them.**
  Programs in the pane still see the original `$SHELL`.
- If your other terminal is set to run fish while your login shell is zsh, you get zsh
  here, with zsh's prompt and completion messages, until you set `shell = "fish"`.

### What is known about a pane

- **Program**: the name of the foreground process. It's read shortly after output or
  input, never on a timer. **A pane that has printed nothing and been sent nothing has
  no known program yet** and shows the generic terminal icon.
- **Directory**: read from the pane's own process (`/proc` on Linux), so it's correct
  with any shell and needs no setup. A shell's own report of its directory (OSC 7) only
  triggers an earlier read. It's the directory of the pane's *own* program: for a pane
  started as `nvim`, it's nvim's directory.
- **Title**: what the program set. Some programs put a mark before the words, such as
  the spinner or star an agent uses to show it's busy. That mark is left out wherever
  the title is shown, because the pane's state has its own mark. A mark joined to a
  word, as in `*scratch*`, is kept. `isc ls` shows the title exactly as the program set
  it.
- **Tile**: chosen from the program's name. Some two hundred programs have their own
  mark, in their maker's colour. Anything else gets the terminal's, unless you give it
  one: see [Icons for your own programs](#icons-for-your-own-programs).

  | Kind | Programs |
  |---|---|
  | Agents | claude, codex, gemini, copilot, cursor-agent, ollama, and aider, amp, opencode, goose and pi, which share one mark in different colours |
  | Editors | nvim, vim, emacs, hx, zed |
  | Version control | git, lazygit, tig, gitui, jj, gh, glab |
  | Rust, Go, Zig | cargo, rustc, rustup, go, zig |
  | JavaScript | node, npm, pnpm, yarn, bun, deno, tsc, vite, webpack, esbuild, next, nuxt, astro, ng, eslint, prettier, jest, vitest, turbo, nx, tailwindcss |
  | Python | python, ipython, pip, uv, ruff, pytest, jupyter, poetry, conda, django (`manage.py`), flask, uvicorn, gunicorn |
  | Ruby and PHP | ruby, irb, rake, bundle, rails, gem, php, artisan (Laravel), composer, symfony |
  | Other languages | lua, perl, R, julia, nim, crystal, elixir, erlang, ghc, ocaml, kotlin, swift, scala, clojure, dart, flutter, dotnet, java, gradle, mvn, cmake, make, gcc, clang, gleam |
  | Databases | psql, pgcli, mysql, mariadb, sqlite3, redis-cli, mongosh, clickhouse, duckdb, supabase, prisma |
  | Containers and clouds | docker, podman, kubectl, k9s, helm, terraform, tofu, ansible, vagrant, pulumi, gcloud, wrangler, vercel, netlify, fly, nginx, caddy, tailscale |
  | The system | pacman, yay, apt, dnf, nix, brew, flatpak, tmux, curl, ffmpeg, mpv, htop, gpg |
  | Remote | ssh, mosh, sftp, scp, rsync |
  | Monitors | btop, top, nvtop, glances, btm |
  | Shells | sh, bash, zsh, fish, nu, dash, ksh, pwsh |

  **A tool run by a language's program is known by its own name.** Vite runs as
  `node`, Rails as `ruby`, Laravel's `artisan` as `php` and Django's `manage.py` as
  `python`. The daemon looks at what the program was started with and calls the pane
  `vite`, `rails`, `artisan` or `manage`, for the tools it knows. A script of your own
  stays `node` or `python`.

  **A shell script is known by its own name**, `deploy` for `./deploy`, on Linux and on
  a Mac. On a Mac that's also so when you run it as `sh deploy`.

  The list is fixed. You can't add to it in the settings.

### The header

The header shows, in order: the icon; the title, or the name you gave the pane; the
directory, unless the title already contains it; a mute mark; an eye or a keyboard
while a guest can look at the pane or type in it; the state as one word,
when the pane isn't idle; a percentage, when the program reports progress; what the
state is about; and "Enter closes" on a kept pane whose program has ended.

Buttons show on the focused pane and on the pane under the pointer: split right, split
down, float (or, on a float, put into the layout, keep on top and show over every tab),
minimize, fill the tab, share (an eye: see "Share one project or one terminal" under
[In a browser](#in-a-browser)), close. A pane on another machine has no share button.

Under the header, a two-pixel line shows the pane's state as a colour. It doesn't move.

The `pane-header` setting:

| Value | Header |
|---|---|
| `full` (default) | Everything above |
| `name` | The same, without the buttons |
| `hidden` | No header; the terminal uses the space |

A floating pane keeps its name even with `hidden`, because you drag it by that bar.
Changing this setting resizes the terminals.

### Splitting and focus

- **Split right / down**: `Ctrl+Shift+O` / `Ctrl+Shift+E`, the header buttons, or
  `isc split right|down [PANE] [-- COMMAND…]`. The new pane takes half of the pane it
  splits. If a floating pane is focused, the tiled layout is split instead.
- **Focus by direction**: `Ctrl+Alt+arrow`, or `isc focus left|right|up|down`. This
  moves across tiled panes.
- **Next / previous pane**, floats included: `Ctrl+Shift+]` / `Ctrl+Shift+[`.
- Click a pane to focus it. `isc focus PANE` gives a pane the keyboard.

### Resizing

**With the mouse**: drag the divider between panes. Ratios stay between 5% and 95%.
The pointer changes over a divider; nothing else marks it.

**With the keyboard**: `Ctrl+Alt+Shift+arrow` moves the focused pane's edge that way, by
5% of the space being divided. From the command line:
`isc resize left|right|up|down [PANE] [--by PERCENT]`.

- **The arrow says which way the divider goes, not whether the pane grows.** The
  divider that moves is the innermost one on that axis that holds the pane. So "right"
  widens a pane whose divider is on its right and narrows one whose divider is on its
  left: the same key grows one pane and shrinks another.
- A pane as wide as the tab has no divider to move left or right, so the key does
  nothing.
- Each press is calculated from the last layout the window received. Holding the key
  repeats correctly at keyboard speed. Two presses that arrive in the same instant, as
  from a script or a macro, move the edge once.
- On a floating pane, the same keys move its right and bottom edges.
- While you drag, the programs in the panes are resized as you go, at most once every
  25 milliseconds, and once more at the size you let go at.
- **A program is told of a resize in two ways.** Every program gets the usual signal.
  One that asked the terminal for a written report of every resize, as some editors
  and agents do, gets that too, but only while it's the program in the foreground. If
  it's killed without taking its request back, the shell it leaves behind isn't sent
  reports it never asked for.

### Moving and swapping

- "Move this pane left / right / up / down" swaps a tiled pane with its neighbour on
  that side. The pane keeps the keyboard. If there's no neighbour on that side, nothing
  happens.
- On a floating pane, the same commands move it a twentieth of the tab at a time. It
  stops at the tab's edges.
- These commands (`move-left`, `move-right`, `move-up`, `move-down`) have no key in the
  direct keymap. Use the palette or bind one in `[keys]`. After the leader they are
  `Ctrl+H`, `Ctrl+J`, `Ctrl+K`, `Ctrl+L`, and they repeat.
- `isc swap A B` swaps the places of two panes.
- `isc put PANE left|right|above|below --of OTHER` moves a pane to that side of another:
  the two share the space the other had, half each. Without `--of` the pane takes that
  half of the whole tab, or of the tab `--tab` names.
- **With the pointer**: drag a tiled pane by its header. Where you let go says what
  happens:

  | Let go | What happens |
  |---|---|
  | In the middle of another pane | The two panes swap places |
  | Near a side of another pane: within a quarter of it, 160 pixels at most | The pane takes that half of the other's place. So two panes side by side become one above the other |
  | Within 24 pixels of the tab's own edge | The pane becomes a column, or a row, at that side of everything |
  | On a tab, along the top or in the sidebar | The pane moves to that tab, at the right of what's there |
  | Anywhere else | Nothing changes |

  - **You see it before it happens.** While you hold the pane, the panes are shown as
    they'd be if you let go there, gliding to their places as you move from one to the
    next. They wait a moment before they move, so carrying the pane across one place to
    reach another shuffles nothing on the way. Let go as soon as the word is right: you
    don't have to wait for the panes. The pane you're carrying is ringed and washed with the accent colour, with a
    word in its middle for what letting go would do: "Swap", "Top half", "Left of
    everything". Move it back over its own place and everything is shown where it was.
  - **Escape puts it back** while you're still holding it.
  - **No terminal is resized until you let go.** While you hold the pane, each terminal
    is drawn at its current size in the place it's shown in. It's cut off where that
    place is smaller, and has empty space where it's larger. The programs get their new
    sizes once, when you drop the pane.
  - A pane that takes half of another's place gets exactly half. The pane it left gives
    its space to its neighbour. A tab left with no pane is gone.
  - You can't drag while one pane fills the tab, and a floating pane is moved by its
    header as before.

### Filling the tab (zoom)

Use `Ctrl+Shift+Enter`, the header button, or `isc zoom [PANE] [--off]`. The pane fills
the tab. The other panes keep their size underneath. Only the zoomed pane is resized.

### Floating panes

A floating pane is an ordinary pane that sits above the layout. Keyboard, mouse,
selection, copy and paste work exactly as they do in a tiled pane. Any number of panes
can float at once.

| To | Do |
|---|---|
| Open a new float | `Ctrl+Shift+D`, or `isc run --float [--rect X,Y,W,H]` |
| Float the focused pane, or put it back | `Ctrl+Shift+G`; `isc float [PANE] [--rect X,Y,W,H]`, `isc tile [PANE]` |
| Move | Drag the header |
| Resize | Drag any edge or corner |
| Keep on top of other floats | `Ctrl+Shift+K`, the header button, `isc pin [PANE] [--off]` |
| Put over the left half, right half, middle or whole tab | The palette commands `float-left`, `float-right`, `float-centre`, `float-fill`. They have no default keys. |
| Hide or show every float in the tab | `Ctrl+Shift+H`, the switch in the top bar, `isc floats show\|hide` |
| Put one float out of sight, and bring it back | `isc hide [PANE]`, `isc show PANE` |

- A float can't leave the tab's area, and it's at least a tenth of the tab each way.
- Its position is stored as fractions of the tab, so it survives resizing the window.
  The rectangle you give to `--rect` uses the same fractions: `--rect 0.5,0,0.5,1` is
  the right half. Without `--rect`, `isc float` and `isc run --float` use
  `0.2,0.15,0.6,0.7`.
- Clicking or focusing a float raises it.
- When you float a tiled pane, the tiled panes beside it take back its space. When you
  put it back into the layout, it goes beside the focused tiled pane.
- Hidden floats keep running and keep their places.
- A float opened with `isc run --float -- lazygit` closes when its command exits, like
  every pane. This covers pickers and one-shot tools.

**Over every tab.** A float belongs to its tab unless you change that with the button
beside "keep on top" in its header, "Show this floating pane over every tab" in the
palette, leader `I`, or `isc follow [PANE] [--off]`. It then shows over whichever tab of
the project is in front, in the same place and at the same size.

- Closing a tab doesn't close it while the project has another tab. Closing the last
  tab does.
- When it's minimized, it sits in the dock of the tab it was over, and it comes back
  the way it was.

### The scratch pane

Press `` Ctrl+` `` to bring up a shell that floats across the lower part of the tab and
has the keyboard. Press the same key again to put it out of sight and give the keyboard
back to where it was. After the leader the key is `` ` ``; from the command line, use
`isc scratch [--project NAME]`.

- There's one scratch shell per project, and it's the same over every tab. Whatever was
  running in it keeps running while it's out of sight.
- It's called "scratch" in its header and in the sidebar.
- If the scratch pane is on screen but another pane has the keyboard, the key gives the
  scratch pane the keyboard instead of putting it away.
- It's an ordinary pane: you can move, resize, rename, tile, minimize or close it. Once
  its shell ends, the key starts a fresh one.
- Out of sight isn't the same as minimized: the pane has no dock chip. The sidebar
  still lists it, and pressing its row brings it back.
- **`` Ctrl+` `` is not available to programs in panes.** Bind it to `none` in `[keys]`
  to give it back to them. The key is written with the backtick inside the quotes:
  ``"ctrl-`" = "none"``.

### Minimizing and the dock

Use `Ctrl+Shift+M`, the header button, or `isc minimize [PANE]`. The pane leaves the
layout and its program keeps running. The pane keeps the size it had, so the program
isn't disturbed.

- It waits in the dock as a chip: icon, title, percentage if reported, state mark, and
  a border in its state's colour. Chips stay in the order they were minimized.
- **Restore**: click the chip; `Ctrl+Shift+R` restores the pane minimized last;
  `isc restore PANE`. A tiled pane returns beside the panes it was beside, with the
  share of space it had. A float returns to its rectangle.
- A middle click on a chip closes the pane without asking.
- "Minimize every pane but this one" (`minimize-others`) and "Restore every minimized
  pane" (`restore-all`) are palette commands with no default keys.
- A minimized pane costs the window nothing.
- The dock fits about one chip for every 170 pixels of the tab's width. When there are
  more chips than fit, the idle minimized panes share one chip, "3 idle", at the end of
  the dock. Pressing it opens the palette, which lists every pane. Panes that are
  working, waiting, done or failed always keep their own chip.
- The dock resizes the terminals when it appears or disappears.
- Hovering a chip shows no preview of its pane.

---

## 9. The terminal

The terminal engine is libghostty-vt, the terminal core of Ghostty. Escape sequences
behave the way they do in Ghostty.

### `TERM`

`TERM` is `xterm-ghostty`, Ghostty's description of the terminal, which says everything
a pane can do. The description comes with insensical:

- If your system already has it (under `$TERMINFO`, `~/.terminfo`,
  `/usr/share/terminfo`, `/usr/lib/terminfo` or `/etc/terminfo`), that one is used.
- Otherwise the daemon compiles its own copy with your system's `tic`, the first time a
  pane starts, into `terminfo` in the state directory, and points the pane's programs
  at it with `TERMINFO_DIRS`.
- If there's no `tic`, `TERM` is `xterm-256color`.
- `term = "xterm-256color"` in the settings makes it that for every pane started from
  then on.

**Over `ssh`** to a machine that doesn't have that description, programs complain about
an unknown terminal. Install the description there (`infocmp -x xterm-ghostty | ssh
host tic -x -`), or set `TERM=xterm-256color` for that command, or set `term` as above
if most of your work is on other machines.

Every pane also has these set: `COLORTERM=truecolor`, `TERM_PROGRAM=insensical`,
`TERM_PROGRAM_VERSION`, `INSENSICAL_PANE` and `INSENSICAL_SOCKET`.

### Scrollback

Each pane keeps 10 MB of history. That's an amount of memory, not a number of lines.
The `scrollback` setting changes it, from 1 to 200 megabytes, for panes started after
the change; a pane that's running keeps what it was started with. The history is held
in the daemon's memory. Section 17 covers what happens to it when the daemon stops.

**What panes cost.** Measured on Linux with 70 panes open in one daemon:

| | Memory in the daemon |
|---|---|
| A pane at a prompt with little history | about 0.4 MB |
| A pane whose history is full, while it is printing | about 11.5 MB |
| The same pane once it has been quiet for a moment | about 1.6 MB, for output that repeats as a build's does |

- **History is packed away when a pane goes quiet**, in the daemon and in the window
  that shows it, and unpacked when you scroll into it or search it. Nothing is lost and
  you do nothing. Output that repeats packs to a fraction of its size; output that
  doesn't packs less.
- **A window holds up to a gigabyte of one pane's history.** Only a pane with a lot of
  history that's then made several times wider goes past that, because each of its
  lines takes more room than it did. The window then shows that pane's screen without
  its history, which the daemon still holds.
- A pane you aren't looking at costs the window nothing: only the panes of the tab in
  front are drawn, or hold a copy of their history in the window.
- An idle daemon uses no CPU, however many panes it holds.
- **The programs cost more than the panes.** A shell is some megabytes and an agent
  can be several hundred. insensical doesn't limit or stop them.
- **The window says what each project's programs hold.** On the right of a project's
  heading in the sidebar is the memory held by everything running in the project, and
  its share of a processor when it's using one: `1.2 GB · 40% cpu`. It's shown only
  when there's room for all of it after the project's name: a long name in a narrow
  sidebar is shown whole and the figure is left out. The overview says the same for
  each pane. `isc usage` prints it for every pane.
- A pane's figure covers its program and everything that program started.
- The window asks for these figures every five seconds, and only while it's in front.
  The share of a processor is what was used between the last two askings.
- In the daemon a pane is one thread and three open files. The daemon raises its own
  limit on open files as far as the system lets it when it starts. While a pane's
  program prints faster than about fifty megabytes a second, the pane has a second
  thread that does nothing but read; it's given back when the output stops.

If a window falls very far behind a flood of output, it's resynchronised, and the output
nobody could have read is skipped.

**History from another machine comes a part at a time.** With `--host`, a pane shows
its screen and the newest half megabyte of its history at once, which is tens of
thousands of lines. Scroll to the top of that and the next four megabytes are fetched,
and so on back to the start.

- The top of the scrollbar's track is marked while there's more on the other machine,
  and the thumb can't reach the top until it has all come. How much is left is an
  estimate.
- Searching looks through what has come, not through what's still on the other
  machine.
- If the pane changes size while part of its history is still there, the pane is
  fetched afresh: you get the newest half megabyte again, and the rest as you scroll.
- The daemon holds what's left for you in its memory until you've fetched it, or you
  leave the pane. It holds 256 MiB of that at most, for every pane and everyone
  attached together. Past that the oldest is let go, starting with what was held for
  clients that can only look. If yours was let go, the pane is fetched afresh when
  you scroll to the top: the newest half megabyte again, and the rest as you scroll.
- A daemon on this machine sends all of a pane's history at once, as always.

When you switch to a tab, the tab you left stays on screen until every pane of the new
one has received its screen, so you don't see empty panes in between. That's a few
thousandths of a second on this machine, and 150 at most: after that the new tab is
shown as it is, and a pane that still has nothing says so. Meanwhile the sidebar and the
tab bar already show the new tab, and what you type goes to it from the key that
switched: nothing reaches the tab you left. A click on the tab you left does nothing.
With a see-through window (`opacity` below 1) the old tab isn't kept, and a pane of
the new one can show empty for an instant. Keys go to the new tab from the moment you
switch when you switch with a key or by clicking a tab. A switch made any other way,
by clicking a pane's row in the sidebar or with `isc tab select`, takes your keys with
it when the window hears of it. If you scroll back into history that hasn't arrived, nothing shows that
it's missing.

**From command to command.** `Ctrl+Shift+Up` scrolls to the prompt of the command
before what you're looking at, and `Ctrl+Shift+Down` to the one after (`⌘↑` and `⌘↓` on
macOS), so going back through a long session takes one key a command however much
each printed.

- It needs a shell that marks its prompts: fish 4 does, and bash and zsh do with the
  line `isc shell-integration` prints. Without marks the keys do nothing.
- After the last command, `Ctrl+Shift+Down` goes to the bottom.
- A prompt of several lines counts once, at its first line.
- Prompts in what was restored from a stop of the daemon aren't known.

### Text

- **Font.** `font-family` names the terminal's font. `font-size` is in points, and the
  default is 12, or 13 on macOS. On Linux a point is 1⅓ pixels, as most terminals count it. So 12 points is 16 pixels, and the size you set in another terminal
  gives the same text here. On macOS a point is a pixel. A row is 1.3 times the font's
  size high, rounded up. The text around the terminals has its own font and size: see
  [Appearance](#appearance).
- **There is no font weight setting.** Text uses the family's regular face, and bold and
  italic use those faces. If your other terminal is set to semibold, text here looks
  thinner.
- **Ligatures.** Symbols that stand side by side are drawn as the font's ligature for
  them, if it has one: `->`, `!=`, `<=`, `===`. `font-ligatures = false` turns that off.
  Only symbols join. Ligatures a font makes of letters, such as `www` or `fi`, aren't
  formed, and neither are symbols a program gave different colours or weights.
- Characters the font doesn't have come from the system's fallback fonts. That includes
  emoji.
- **Icons in cells aren't fitted to their cell.** A wide Nerd Font glyph can overlap its
  neighbour.
- Line, block and Powerline characters are drawn as shapes, on whole pixels, and don't
  come from the font. So lines join and blocks tile at any size and scale.
- **Cursor**: block, bar or underline, whichever the program asks for, in the theme's
  cursor colour or the program's.
  - It blinks until a program asks for a steady one. Typing makes it visible and restarts
    the blink.
  - **Only the pane your typing goes to has a solid cursor.** In every other pane, the
    cursor is the outline of a block and doesn't blink. The same is true of every pane
    while the window isn't in front, or while a panel such as the palette has the
    keyboard.
  - A pane that was already running when the daemon was updated to the version that
    introduced this keeps a steady cursor until its program asks otherwise.
- **Half-drawn screens are held.** When a program says it's in the middle of drawing
  (synchronized output), its screen is shown when it says it has finished, or after one
  second.

### Colours

- The sixteen named colours come from the theme. 256 colours and true colour are shown
  exactly as the program gives them.
- When a program asks the terminal for its colours, or whether it's dark, it gets the
  theme of the window that's showing it.
- **On a light theme**, text whose contrast with its background is less than 3:1 is
  painted in a darker shade of the same hue. Only the pixels change. What you copy is
  what the program wrote. Dark themes are painted exactly as asked.

### Selecting and copying

- Drag to select. Click a second time to select the word, and a third time to select the
  line. Hold Alt while you drag to select a rectangle. Hold the pointer beyond the top or
  bottom to scroll.
- Selecting copies to the selection clipboard on systems that have one (Linux).
- Press `Ctrl+Shift+C` to copy to the clipboard. For a second, the selection turns the
  accent colour and "Copied" shows at the bottom of the pane, or "Copied 14 lines" when
  it was more than one. If nothing is selected, nothing is copied and no message shows.
  Wrapped lines are joined when you copy them.
- "Select everything in the pane" is a command, `select-all`. It has no default key.
- Typing returns the view to the bottom and clears the selection.
- A selection belongs to the window you made it in. It stays on its text as output
  arrives.

### Selecting with the keyboard

You can move through a pane's screen and its history, select and copy without the
mouse. Start it with `Ctrl+Shift+X` (`⇧⌘X` on a Mac), with the leader and then `v`, or
with "Select and copy with the keyboard" in the palette. The same key leaves it.

- **While it's on, every key is its own.** Nothing you press reaches the program.
- **It starts where the program's cursor is**, which is usually the line you're
  looking at. Its own cursor is a solid block in the accent colour, and the program's
  cursor turns into an outline.
- **A strip at the foot of the pane says it's on and what the keys do right now.** It
  changes as you go: `v` select, `Space` labels, `/` search and `y` copy while you
  move, then `y` copy, `o` other end and `Esc` once something is selected. It floats
  over the pane and resizes nothing. It moves to the top when the cursor is at the
  foot, and in a narrow pane it shows as many keys as fit whole.
- **Output doesn't move you.** The program goes on printing, and your place, your
  cursor and your selection stay on their text. The strip says "new output below".
- **Copying leaves.** `y` or Enter copies and ends it, the selection flashes, and the
  pane says "Copied", or "Copied 14 lines" when it was more than one. That's said
  after `Ctrl+Shift+C` too.
- **Escape lets go first, then leaves.** With something selected, Escape drops the
  selection. With nothing selected, it leaves, and the view goes back to where the
  program is. `q` always leaves.
- Pasting, or running the command again, leaves it too.

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

- **A line that wraps is one line.** Words, `0` and `$`, and `f` go through the wrap,
  and what you copy across it is one line.
- A wide character is one step.
- The view follows the cursor and keeps two rows in hand above and below it.
- The wheel and the scrollbar still scroll, and the cursor comes along. A click puts
  the cursor there, and a drag selects from it.

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

- `o`, `c`, `[` and `]` need a shell that marks its prompts (OSC 133). The shells
  insensical sets up do. Where nothing is marked, the strip says so.
- `p` goes by shape and doesn't ask the disk: a word with a slash in it, a file name
  with a short ending, with a line number after it if there is one.

**Labels**

Press `Space` and every path, web address, commit and machine address in view gets a
letter. Type the letter and that one is copied, and you're back at your prompt.

- The newest get the keys under your fingers: `a`, `s`, `d`, `f`.
- With more than 26 in view, each label is two letters. The first narrows them down,
  and Backspace takes it back.
- **Hold Shift with the letter to select it without copying**, so you can stretch the
  selection before you press `y`.
- Escape puts the labels away. So does any key that isn't a label.
- Plain numbers and ordinary words get no label.

**Searching**

- `/` opens the pane's search. It starts from the nearest match above the cursor.
- Enter puts the cursor on the match and returns to selecting. If you'd started a
  selection, it now reaches the match.
- Escape closes the search and puts everything back as it was. Up and Down step
  through the matches while the field is open.
- `n` goes to the match before, `N` to the one after.

**Limits**

- In a full-screen program it works on what's shown. There's no history there.
- If the program takes over the screen or gives it back while you're selecting, or
  the daemon has to send the pane again, the selection is dropped and the cursor
  starts again at the program's. The strip says the screen changed.
- A line wrapped over more than about thirty rows is read thirty rows to either side
  of the cursor.
- In the window, the leader pressed twice sends the leader's key to the program and
  leaves the mode.
- **When the link to another machine is lost, it ends.** The pane keeps what it last
  showed, dimmed, but that's a picture: there's nothing behind it to move through or
  copy from until the machine is back. `Ctrl+Shift+X` on such a pane is refused the
  way typing is.
- Typing drawn ahead on a far machine is never selected or copied. You get what the
  program has shown.
- On another machine, the history that hasn't arrived yet is fetched when the cursor
  reaches the top of what's here. The strip says "older history is on its way", and
  the keys go on working meanwhile.
- There's no `isc` command for it. It belongs to the window or terminal you're at.

### Pasting

- Press `Ctrl+Shift+V` to paste the clipboard. Press `Shift+Insert` or the middle button
  to paste what you last selected.
- **A paste that would do more than it looks like is held**, and you're asked first.
  That's any of these:
  - More than one line, into a program that hasn't asked for pastes to be marked
    (bracketed paste). Such a program runs each line as it arrives.
  - Text that contains the sequence that ends a marked paste.
  - More than one line, into a pane on a machine you marked careful, whatever the
    program asked for. One line with its line end is one line. See
    [Other machines in the window](#other-machines-in-the-window).
- **Who copied it makes no difference.** What a program put on the clipboard (see
  [The clipboard and programs](#the-clipboard-and-programs)) is pasted the way your
  own copy is. A command an agent copied for you lands at your prompt, where you read
  it before you press Enter.
- **The question shows what you'd be pasting.** A card at the bottom of the pane says
  how many lines there are and which of the reasons applies, and shows the text
  in the terminal's own font.
  - A line ends at a carriage return as well as at a line feed, because that's how the
    program reads it.
  - A long line continues on the next row. It isn't cut.
  - Characters you wouldn't otherwise see are shown: a control character as its sign,
    such as `␛` for Escape and `␉` for Tab, and an invisible or direction-changing
    character by its number, as in `<U+202E>`.
  - When there isn't room for all of it, the last row counts the rest: "and 24 more
    lines".
- Press Enter or click "Paste" to paste. Press Escape or click "Cancel" to drop it.
- Most shells ask for marked pastes, so they get multi-line pastes straight away.
- Text that arrives in one piece with a line end or a control character in it, from an
  input method, dictation or a system service, is treated as a paste and checked the
  same way.

### Mouse and wheel

- A program that asks for the mouse gets presses, releases, motion and the wheel. Hold
  Shift to select text instead.
- The wheel scrolls history. In a full-screen program that didn't ask for the mouse, it
  moves through the file (arrow keys are sent). In a program that did ask, it's reported
  to the program.
- The wheel moves three rows per notch, and at most twelve steps per event.
- A scrollbar appears while the view is scrolled back, and when the pointer is at the
  pane's right edge. You can drag its thumb.

### Finding text

Press `Ctrl+Shift+F` to open a field over the pane's top right corner. When the match
it's showing would be under the field, the field moves to the pane's bottom right.

- It finds text anywhere in the pane's history, ignoring case. It shows which match
  you're on and how many there are, and brings each match to the middle of the view,
  selected, so you can copy it.
- Press Enter to go to the previous (older) match, and Shift+Enter to go to the next.
  Press Escape to return to where the program is writing.
- **Text that wraps at the pane's edge is found across the wrap**, as long as it's on
  two rows. A match can't span three.
- There are no regular expressions.

### Web addresses and paths

- `Ctrl`+click a web address to open it (`⌘`-click on macOS).
- `Ctrl`+click the path of a file to open it, or to see it in your file manager. Which
  one depends on the kind of file (see "What a click on a path does" below).
- **A web address is marked when the pointer is over it.** It's underlined, and a small
  note in the pane's bottom right corner shows the key and where it leads, as in
  "Ctrl+click opens localhost:5173". The note goes after four seconds if the pointer
  doesn't move, and comes back when it does; the underline stays. While you hold
  `Ctrl`, the pointer becomes a hand. A plain click doesn't open it, because a click in
  a terminal selects.
- **A path is marked only while you hold `Ctrl`.** That's when insensical looks on the
  disk for it. The note then says what the click will do: "Ctrl+click opens main.rs in
  src/" or "Ctrl+click shows in the file manager Report.app". On Linux a file the note
  says it opens can still be shown in the file manager: your desktop is asked what
  kind of file it is only when you click (see below).
- For addresses, only `http://` and `https://` are ever opened. Brackets and punctuation
  around the address are left out.
- A path is marked only if it leads to something that exists:
  - It's read from the directory the pane's shell is in. `~/` is the home directory.
  - A path that starts with a slash but doesn't exist from the root is tried from the
    pane's directory. That's how a bundler prints `/src/App.vue`.
  - A line and column after the path, as in `src/main.rs:12:5`, aren't part of the path.
    They're passed to the editor you chose with `open-files-with` (see below). Your
    desktop's own opener can't take them, so with it the file opens at its beginning.
  - A bare word with no slash and no dot in it, such as `src`, isn't looked up.
  - A path on a file system known to be another machine's, or automounted, isn't
    looked up, so pointing at one can't make your machine wait for, or reach out to,
    another. On Linux that goes by the kind of file system: NFS, SMB, 9p, AFS, Ceph,
    Coda, GlusterFS, Lustre, NCP, davfs, autofs, and anything a program keeps through
    FUSE, such as `sshfs`, `rclone` or an encrypted directory. A network file system
    of another kind is looked up like a disk of your own.
- **An address or path that continues onto the next row is found whole**, from either
  row, and both parts are underlined. It's read up to four rows to either side of the
  pointer.
- **Links a program marks** (OSC 8), such as an agent's "open the pull request" or the
  file names of `ls --hyperlink`, work the same way: the marked words are underlined
  under the pointer and `Ctrl`+click follows them. Only a web address, or a `file://`
  link to a file that exists on this machine, is followed. A `file://` link that names
  another machine is ignored, and so is any other kind of link a program marks.
- **The note names the machine the address leads to first.** It's in full when the
  note has room for it; a name longer than the note keeps its end, which says whose it
  is. The rest of the address follows it and is shortened in the middle if it's long.
  A name and password written before the machine's name, as in `https://name@host/`,
  aren't shown: they can be written to look like another site.
  - An address with nothing where the machine's name belongs, such as
    `https:///name@host/`, isn't a link. A browser skips the extra slashes and goes to
    the name after them.
  - A machine's name with letters outside ASCII is shown the way the network is asked
    for it, as in `xn--mnchen-3ya.example`: letters of other alphabets can look exactly
    like these. It's made from the name as written, in lower case. A browser may change
    some letters first, so in rare cases it isn't letter for letter where the browser
    goes; it still tells you the name isn't the plain one it looks like.
- **A click follows only what the note said.** A program can change where its words
  lead at any moment. The click opens nothing, and the note says where they lead now,
  if the note wasn't showing, if it said something else, or if it began to say this
  less than half a second ago in place of something else it said with the pointer on
  the same cell. Click again to follow it. If the words come to lead somewhere else
  while the note is away, the note comes back.
- An address or a path with an invisible or direction-changing character in it is
  never marked or opened. Such a character can make an address read as another.
- While you hold `Ctrl` with the pointer on a link, a program that asked to hear of
  every move of the pointer isn't told of it.

**What a click on a path does.** Clicking something a program printed can't start
anything. A program can print any path, and a directory you cloned or unpacked can
hold anything under any name.

- **A file opens only if opening it just shows it.** With `open-files-with` left at
  `system`, that's a file whose name ends in one of these, that can't be run, that
  nobody else can replace, and that begins as its name says:
  - text and data: `txt`, `text`, `md`, `markdown`, `rst`, `adoc`, `log`, `json`,
    `jsonl`, `yaml`, `yml`, `toml`, `ini`, `cfg`, `conf`, `lock`, `diff`, `patch`
  - source: `rs`, `c`, `h`, `cc`, `cpp`, `cxx`, `hpp`, `hh`, `go`, `java`, `kt`, `swift`,
    `cs`, `ts`, `tsx`, `jsx`, `vue`, `svelte`, `css`, `scss`, `sql`
  - pictures and documents: `png`, `jpg`, `jpeg`, `gif`, `webp`, `bmp`, `pdf`
  - `csv` and `tsv` aren't among them. A spreadsheet works out the formulas in one, so
    they're shown in your file manager.
- **Everything else is shown in your file manager, not opened.** That's every
  directory, since on a Mac a directory can be an application; every file that can be
  run; and every other kind of file: a script, a web page, a `.jar`, a `.desktop`
  file, an installer, a file with no ending.
- **With an editor chosen in `open-files-with`, any file opens in that editor.** An
  editor shows a file whatever it is. A directory is still shown in the file manager.
- **A file someone else could replace is shown in your file manager.** Your desktop is
  handed a path, and opens whatever is there when it gets to it. So the file, and every
  directory on the way to it, must belong to you or to root and be writable by nobody
  else, your own group included: a group can have other people in it. A path through a
  shared directory such as `/tmp`, or one your group can write to, is shown, not opened.
  If that's your own project directory, `chmod g-w` on it and the folders above the
  file makes its files open again.
- **A file that doesn't begin as its name says is shown in your file manager.** Some
  desktops choose what opens a file by what's in it, so a web page named `notes.md`
  would open in the browser. The first four kilobytes are read:
  - A picture or a PDF must begin with its own format's mark.
  - A text or source file must be UTF-8 with no zero byte, and must not begin with `<`,
    `#!` (`#![`, as a Rust file begins, is fine), `%!` or `{\rtf`.
  - Nor may it begin as a mailbox does (`From `), or a calendar (`BEGIN:VCALENDAR`),
    a contact card (`BEGIN:VCARD`) or a launcher (`[Desktop Entry]`,
    `[InternetShortcut]`): each of those has a program of its own that would open it.
  - A text or source file must not hold, in its first 256 bytes, a tag such as `<img`,
    `<div`, `<br`, `<svg` or `<!--`.
  - No file may hold `<html`, `<head`, `<title`, `<script`, `<style`, `<table`,
    `<a href=` or `<!doctype` anywhere in what's read. Capitals, and spaces, tabs and
    line ends inside one, make no difference: `<A` at the end of one line and `HREF=`
    on the next is found.
  - So a `.vue` or `.svelte` file that begins with a tag, and a Markdown file that
    begins with HTML, are shown and not opened. Choose an editor to open those.
- **On Linux your desktop is asked as well, when you click.** The marks above are the
  ones insensical knows, and your desktop may go by others. So `xdg-mime query
  filetype` is asked what kind of file it is, and the file is opened only if the answer
  is a kind that's only shown: plain text, Markdown, the kinds it gives source, JSON,
  YAML, TOML and SQL files, a picture or a PDF. Anything else is shown in your file
  manager: a web page, a saved mail, a shortcut, a program.
  - It's given two seconds. If it isn't installed, fails or doesn't answer, the marks
    above decide alone.
  - A desktop that goes by a file's name takes a `.ts` file for a video or a
    translation and a `.tsx` file for a map. There they're shown, not opened.
  - It isn't asked on a Mac, which opens a file by its name.
- **The file is looked at again when you click.** If the path leads somewhere else
  than when the note was written, or to another file, or the file has been written to
  since, it's shown in your file manager and not opened. A program running as you can
  still swap a file in the instant after that look.
- A symbolic link is judged by the file it leads to, not by its own name.
- **A path in a pane on another machine is that machine's, and nothing on this
  computer is opened for it.** This computer's disk isn't asked about it, even if a
  file of the same name is here. With `nvim`, `vim`, `hx` or `nano` chosen in
  `open-files-with`, `Ctrl`+click opens it in that editor in a new pane on that
  machine, and the note says "opens on its machine". Otherwise the click copies the
  path as it was written, and the note says so. That machine's disk isn't asked
  either, so any word that reads as a path is taken for one. A web address opens in
  your browser from any pane.

**Choosing the editor.** `open-files-with` says what a file you `Ctrl`+click opens in:

| Value | Opens the file |
|---|---|
| `system` (the default) | With whatever your desktop uses for that kind of file, at its beginning, if it's one of the kinds listed above |
| `code`, `cursor`, `zed`, `subl`, `idea` | In that editor, at the line and column written after the path |
| `nvim`, `vim`, `hx`, `nano` | In that editor in a new pane, split to the right of the one you clicked in, at that line |
| a command with `{file}` in it | By running it, with `{file}`, `{line}` and `{column}` filled in: `"myedit --goto {line} {file}"` |

- With no line written after the path, the line is 1.
- The settings sheet lists the editors it finds on your `PATH`.
- A command of your own is cut into words at spaces and run directly, not by a shell.
  If it can't be started, the file is handled as it would be with `system`.

### The clipboard and programs

- **A program can put text on the clipboard** (OSC 52). This is how an editor on another
  machine, reached over `ssh`, copies: `"+y` in Neovim there lands on the clipboard here.
- **Only the program in the pane you're looking at can do this.** That's the pane with
  the keyboard, in a window that's in front. A program in a tab you aren't on, or in a
  minimized pane, is ignored. So nothing can change what you're about to paste without
  you knowing. It stops the moment the window leaves the front or closes, whether or
  not the window can still be seen.
- **Pasting what a program put there is like pasting anything else.** You aren't
  asked because a program copied it. [Pasting](#pasting) says when a paste is held.
- Text only, up to a megabyte.
- Only the clipboard. A program that asks for the primary selection or the selection
  instead (OSC 52 with `p` or `s`, which is what `"*y` is in Neovim) changes nothing:
  it isn't put on the clipboard in its place. A secondary selection or a numbered cut
  buffer (`q`, `0` to `7`) is taken as the clipboard.
- **No program can read the clipboard.** Asking for it (OSC 52 with `?`) gets no answer,
  and no setting allows it. Pasting is always something you do.
- It's made plain before it's put there. Control characters other than a tab and a
  line end are left out, and so are the marks that begin and end a paste: terminals
  differ in what they paste for those.
- At most three texts are put there at once, and one a second after that. The rest
  are dropped.
- `clipboard = "never"` turns it off completely. Set on this computer, it holds for
  every machine's panes: the clipboard is this computer's, so its own setting is
  asked as well as the other machine's, and either can say no.

### Pictures

A program can show a picture in a pane: a plot, a thumbnail, a file manager's preview.
insensical draws pictures sent by the **Kitty graphics protocol**, which is what
`kitty +kitten icat`, `chafa`, `timg`, `viu`, matplotlib's kitty backend and many others
use. Programs find out by asking the terminal, and it answers that it can.

- A picture is part of the screen. It scrolls with the text around it, goes into the
  history, and is cut off at the pane's edge.
- It's drawn at the size the program asked for, counted in cells, so it takes the same
  cells whatever your text size.
- **A picture too large for its pane is made smaller, so you see all of it.** That's a
  picture taller than the pane, or one that runs past the pane's right edge. It's drawn
  whole, in its own shape, inside the part of its place that's in view. It's never
  stretched, never cropped to fill, and a picture that fits is never made larger.
  - The program still counts the picture at the size it asked for. So a wide picture
    made smaller has empty rows under it, and a tall one gets smaller as more lines
    are printed and its place moves up out of view.
  - A picture that fits its pane isn't touched: it scrolls off the top like text.
  - Make the pane larger and the picture is drawn larger, up to the size the program
    asked for.
  - A file manager's preview is fitted by the file manager, and is left as it is.
  - In `isc tui` the smaller picture takes whole cells, so its shape can be off by
    less than a cell.
  - A program is told a cell's size in whole pixels, so a picture can be drawn up to
    a tenth smaller than its pixels.
- A program can put it under the text or over it, show a part of it, show it several
  times, and take it away again.
- **Unicode placeholders** work. A program such as Yazi shows a picture by printing
  special characters where it goes, and the window draws the picture there. The
  characters themselves show nothing. The browser page doesn't draw pictures, so
  there those cells are blank. The interface in a terminal shows them where your
  terminal can: see [The whole interface in a terminal](#the-whole-interface-in-a-terminal).
- **It's still there when you come back.** A pane's pictures are shown again when you
  return to its tab, open another window, or use `isc attach`.

What isn't done:

- **Sixel** pictures aren't shown, and the terminal doesn't say it can show them, so
  programs that check send something else. A Sixel picture sent anyway is ignored and
  leaves the text around it as it was.
- A picture has to be sent in the escape sequences themselves. One a program names by a
  file or by shared memory isn't read. Programs that prefer those fall back to sending
  it: `icat` does.
- Each pane keeps 64 MB of pictures. Past that, the oldest that aren't on screen are
  forgotten.
- A program can show one picture in many places. Of more than 256 places in view, the
  256 nearest the front are drawn.
- The pictures in view go on when the daemon stops and starts again, the same way a
  pane's text does. They go on when the daemon moves to a new version of itself too,
  unless the engine changed between the two.
- When you come back to a pane, a picture whose top edge has scrolled out of view isn't
  shown again, and neither is anything past 32 MB of them. Scroll or redraw brings a
  program's own pictures back.
- A picture much taller than its pane can lie over the lines printed after it: the
  program's place for it reaches below the pane's foot, and it's drawn inside that place.
- `isc capture` and copying give the text only.

### What the terminal does not do

- **Input methods (IME) are off unless you turn them on.** With `input-method = true`,
  your system's input method (fcitx, ibus, the Mac's) composes text in a pane: what
  it's composing shows at the cursor, underlined, and the program is given only the
  finished text. Keys the input method doesn't take are typed as before. It's off by
  default: turn it on, and turn it off again if typing misbehaves. Whether accents
  typed as two keys (dead keys) work with it on depends on the toolkit.
- Sixel images aren't shown. Kitty graphics are: see [Pictures](#pictures).
- Shell-integration marks (OSC 133) are read to learn when a command begins and ends,
  how it ended, and where each prompt is. The words a command running longer than a few
  seconds was started with are read from the system, so they can be seen by everyone
  who may see your panes: don't put a secret on a command line.
- When you make a pane narrower, its lines are rewrapped, even when a program has
  switched line wrapping off. A full-screen program's screen isn't rewrapped: it's cut
  off at the new edge, and the program draws it again.

---

## 10. States and attention

Every pane has one state. The daemon keeps it, so it's correct even when no window is
open.

| State | Means | Shows as |
|---|---|---|
| `idle` | At a prompt, or nothing has reported another state | nothing |
| `running` | A command or agent is working | a mark, and a percentage if one is reported |
| `waiting` | Blocked on you | a mark, counted as attention |
| `done` | Finished since you last looked | a mark, counted as attention |
| `failed` | Finished with an error since you last looked | a mark, counted as attention |
| `exited` | The program ended and the pane was kept (`isc run --keep`). It shows `done` or `failed` until you've seen it, then this. | a mark |

Each state has its own mark, drawn in the state's colour:

- `running`: a ring of eight dots that spins, with the leading dot brightest. It's the
  only mark that moves, so you can tell at a glance what's still working and what has
  stopped. It moves one step every tenth of a second, and every such mark in the window
  spins together.
- `waiting`: a dot with a ring around it.
- `done`: a tick. `failed`: a cross. `exited`: a ring with a bar across it.

While no pane is `running`, nothing spins and the window redraws nothing for it.

`done` and `failed` mean *unseen*. When you look at the pane, it's marked seen and
returns to `idle`. Looking at a pane means it has the keyboard, in a window that's in
front.

### Where a state shows

- On the pane's header and the line under it, its tab, its sidebar row and its dock chip.
  A `running` pane has no coloured line: it has its turning mark, in the theme's blue,
  and no more.
  It also shows as counts at the head of the sidebar, or in the top bar: how many
  panes are waiting or failed, in the colour of the most urgent, and before it, in
  grey with a tick, how many have finished. Either is left out at zero.
- **What needs you and what finished are counted apart, and each is the same number
  wherever it's shown.** A pane needs you when it waits or failed. A pane that finished
  and that you haven't looked at is news: it keeps its tick and its place in the bell's
  count of what finished, and it's never added to the number that need you. So the
  activity button, a machine's heading, a project's row, the tray, the menu bar and the
  interface in a terminal all say, for the same panes, the same number.
- **In the sidebar, a project or a tab shows a state only while it's folded.** Unfolded,
  the rows beneath it show their own states and its row shows none. Folded, it shows the
  most urgent state of what it holds, and a project shows how many of its panes need you,
  or, where none does, how many have finished.
  If the settings list no rows beneath it, it always shows a state.
- **A waiting pane glows.** Its edge and its header take the waiting colour and a soft
  light of that colour fills the gaps around it, so you catch it from the corner of
  your eye among many panes. The light beats three times as the pane begins to wait and
  then stands still; with `motion = "off"` it's still from the start. It never falls on
  another pane's text or on the sidebar. With `pane-gap = 0` there's no gap to fill, and in a
  see-through window a tiled pane has nothing to hide the light behind, so in both
  cases you get the coloured edge and header alone. `attention-glow = false` turns off
  the light, the coloured edge and the coloured header.
- A pane's header shows its state in words, then what the pane says about it, then how
  long it has needed you. The state's words are always whole. The pane's name gives
  way first, and the program's words are cut at their end. In a narrow pane with a
  state to show, the header's buttons appear only under the pointer, and "Sized
  elsewhere" shrinks to its icon.
- The words are the same wherever a state is put in words: "working" for `running`,
  and for the rest the state's own name or what the pane waits for. Hover over a mark
  on a tab or a dock chip to read them, for example "waiting · Needs permission: Bash".
- **A pane that waits says what for, when its program said**: "wants permission", "asks
  a question" or "needs you to sign in" stands in place of "waiting". See "Programs
  that report by themselves" below.
- A tab, a project or the top bar shows the most urgent state of what it holds:
  `waiting`, then `failed`, then `done`, then `running`. The mark of a tab or a project
  gives the words of the pane it stands for.
- A **dock chip** is the small box a minimized pane becomes in the dock, with its icon,
  name and state. When the dock has more chips than it has room for, the chips of panes
  that are `idle` share one chip that reads, for example, "3 idle". Click it to open the
  palette, which lists them. A chip whose pane has a state always keeps its own chip.
- **Nothing is ever reordered because of a state.** Tabs, rows and chips stay where they
  are.

### Going to the pane that needs you

- Press `Ctrl+Shift+A`, or click the count in the top bar, to go to the pane that needs
  you most. If the pane was minimized, it's restored. Do it again to step through the
  others: first the panes that need you, then the panes that finished.
- The palette opens with every pane that needs you at the top, most urgent first. A pane
  that finished is listed with the rest, with its tick.
- The sidebar can list only the panes at work or with something for you: see "Showing
  only activity" under The sidebar.
- The overview shows every tab of every project with each pane's state. For a pane that
  wants you, it also shows what the pane is asking and how long it has waited.

### Where a state comes from

Strongest first:

1. **A report**, in either of two ways. The program can say it in its own terminal
   (see "Programs that report by themselves" below), or an agent's hooks or a script
   can call
   `isc status running|waiting|done|failed|idle [-m "what about"]`. A `running` or
   `waiting` reported this way overrides everything below until the next report or until
   the command ends. In a pane you started with a command, such as `isc run -- claude`
   or `isc run -- bash`, there's no shell of the pane's own for a command to end at:
   there a report stands until the next report or until the program ends. A report
   carries the time it was made. A report that was made earlier and arrives later is
   ignored. A time ahead of the daemon's clock counts as the moment the report
   arrived.
2. **What the program writes to its terminal.**
   - Progress (OSC 9;4): set or indeterminate gives `running` with the percentage;
     removed gives `done` if the pane was running; error gives `failed`; paused gives
     `waiting`, until the command ends or you look at the pane. A full reset of the
     terminal, as `reset` does, removes the progress too.
   - A notification (OSC 9, 777 or 99) gives `waiting` with its text, until you look at
     the pane. It doesn't change a `done` or `failed` that was reported: an agent that
     says it finished, and then announces it, hasn't asked you for anything.
   - The bell, while nobody is looking, gives `waiting` until you look at the pane.
     **The bell marks a pane and never sends a desktop notification**, because shells
     ring it on every failed completion.
   - **A password prompt** gives `waiting`, with the line it asks on as what the pane
     says, such as `[sudo] password for you:`. insensical knows a program is asking
     because the terminal stops showing what you type while still taking a whole line,
     which is how `sudo`, `ssh`, `git` and a script's `read -s` ask. The pane waits
     whether or not you're looking at it, and stops waiting when typing shows again or
     the command ends. Nothing the program prints meanwhile ends the wait. A program
     that asks while drawing the screen itself, or a `sudo` set to show a star for each
     character (`pwfeedback`), can't be told from any other full-screen program and
     isn't noticed.
3. **The command in the foreground ending**: `done` if the pane was `running`, or if the
   command ran for 30 seconds or more, or for as long as `long-command` says. This
   doesn't need shell integration.
4. **The program itself ending** in a kept pane: `failed` if it exited with a non-zero
   status, even if it had reported `done` just before.

States aren't only for agents. Any command that runs for 30 seconds gives its pane a
state. So does any program that reports progress, sends a notification or rings the bell,
and any script that calls `isc status`.

`isc status explain [--pane N]` tells you why a pane is in its current state.

`isc status clear [--pane N]` forgets what was reported for a pane and puts it at
rest. Use it when a pane says `running` or `waiting` for something that's gone. The
palette has the same command: "Forget what this pane's program said it was doing"
(`forget`). Looking at a pane never clears `running` or `waiting`, because looking
doesn't answer a question.

### Programs that report by themselves

A program can tell its terminal what it's doing, and insensical listens. This is the
Program Status Protocol, an escape sequence numbered OSC 7501, which insensical reads
at revision 0.4. It needs no hook and no
setup, and because it travels with the program's output, it works over `ssh` and from
inside a container, where `isc status` can't reach.

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

- **A wait says what it's for.** A program that's blocked can say on what, and the pane
  says so everywhere its state is put in words: "wants permission", "asks a question" or
  "needs you to sign in". In `isc ls` it's after the pane's title: `wants permission, 2h`.
  A notification's text begins with it, and so do the pane's lines in the tray's menu
  and the Mac menu bar's.
- **The program's name picks its icon.** A program can give its name, as Claude Code
  gives `claude-code`. If insensical has an icon for that name, or you gave it one
  under `[icons]`, the pane uses it in place of the icon of the process, which for
  Claude Code is `node`. `isc ls` shows the name in brackets after the pane's title.
- **Several things at once.** A program can report on tasks of its own as well as on
  itself: an agent's helpers, the regions of a deploy. The pane shows the most urgent
  of them, in the usual order (`waiting`, `failed`, `done`, `running`), with that
  task's name and words. The overview adds a count, such as "3 tasks · 1 waiting", and
  hovering over the pane's state lists each task with its own mark. In a terminal, the
  count follows the pane's words in its header where there's room. A pane keeps up to
  64 tasks; with more, the one reported longest ago is dropped. A hover lists the
  first eight.
- **Working and waiting end with the program.** When the command ends at your shell's
  prompt, what it said it was working at or waiting for goes with it, and the pane
  shows how the command ended, as for any command. The same happens when the program
  is killed under a shell you started inside the pane's shell: insensical asks the
  system whether anything of it is left. `done` and `failed` stay until you've seen
  them. A program can also take back what it said (`state=clear`).
- **An agent run with `exec` keeps its state.** After `exec claude` at a prompt, the
  agent is the pane's own program, and what it reports stands as it does in a pane you
  started with a command.
- **A job in the background keeps its state** while it runs and your shell sits at its
  prompt. It ends when the job does, or when the next command you run ends.
- **It outranks the program's own hooks.** If an agent reports this way and also has
  insensical's hooks, the two would say the same things at slightly different times.
  While the program says it's working or waiting, what its hooks say is set aside for
  that pane. `isc status explain` tells you when that happened. When the program says
  neither, the hooks count again: after it reports `done`, `error` or `idle`, a hook's
  report made later is believed. You don't need to remove them.
- **A program that's gone quiet doesn't hide a question.** If the program said it was
  working and has said nothing for 30 seconds, a hook that says the pane is waiting,
  done or failed is believed, and so are the hooks after it until the program reports
  again. Nothing a hook says takes down a `waiting` the program itself reported.
- **What a program finished at ends when the next command starts.** A `done` or
  `failed` it reported stays until you've seen it, or until something else is run in
  the pane.
- **The hook's words fill in for a program that gave none.** A program may say only
  that it's blocked, while its hook carries the question. If the program's report has
  no words and its hook's last report was of the same state and had some, the pane
  shows the hook's words. The state, what it waits for and when it ends are still the
  program's. `isc status explain` says when the words shown are the hook's.
- A progress report (OSC 9;4) from a program that reports this way changes nothing: its
  state already says how far along it is.

What it can't do:

- **A program run inside tmux or screen in a pane isn't heard.** They don't pass the
  sequence on.
- **It can't always tell that a program is gone.** A pane goes on saying `running` or
  `waiting` after the program that reported was killed, in these cases:
  - Through `ssh`, `docker exec`, `su` or anything else that stays in the foreground
    while the program inside it dies, when the shell inside doesn't mark its prompts.
    It ends when that `ssh` ends.
  - In a pane you started with a command that isn't a shell, when something the
    command ran reported and died while the command carries on. It ends when the
    command ends or reports.
  - Under a shell you started inside the pane's shell, while anything the program
    started is still running.
  - At your own shell, after a command so short it was over before insensical saw it,
    when your shell keeps a helper of its own running and doesn't mark its prompts.
    It ends when the next command does.

  In each, `isc status clear`, or "Forget what this pane's program said it was doing"
  in the palette, puts the pane at rest. So does `printf '\e]7501;state=clear\e\\'`
  typed at a shell in the pane. insensical never ends a state for having been silent:
  a build says nothing for an hour.
- **In a pane you started with a command**, such as `isc run -- claude`, working and
  waiting end only when the program says so or exits, since there's no shell of the
  pane's own for a command to end at. If the command is a shell (`isc run -- bash`),
  they also end when the program that reported is gone. A shell that marks its
  prompts (see below) ends them at each prompt wherever it runs, over `ssh` too.
- **A shell function that reports and then waits inside the shell itself**, with
  `wait` or `read`, looks like a shell at its prompt, and what it reported is dropped
  after a moment. Put the work in a script and run that.
- `isc shell-integration` makes bash and zsh mark their prompts, which ends working
  and waiting at each prompt, on the far side of `ssh` too if you set it up there.
- A report that breaks the protocol's limits is ignored whole: more than 4096 bytes,
  a message over 2048 bytes, or text with control characters in it, a newline or a tab
  among them. A state it doesn't know is ignored too, so a program written for a later
  revision does no harm. A key it doesn't know is passed over.
- **A program can ask what's read.** The terminal answers a program's query with the
  states (`idle`, `working`, `done`, `blocked`, `error`) and the kinds (`permission`,
  `question`, `auth`) it takes, and nothing a program has said.

### Commands that fail, and short commands

- **A long command is one that ran for 30 seconds.** `long-command` in the settings
  changes that, from 1 second to 3600. The daemon reads it as each command ends.
- **With shell integration, a command that runs for 10 seconds or more is listed as
  work when the sidebar shows only activity.** That is a different bar from
  `long-command`, which is about telling you when it ends: a command that ends between the
  two is listed while it runs and goes when it ends, with no `done` or `failed` and no
  notification, unless you lower `long-command` to 10 or less.
- **Without shell integration, a failed command shows as `done`, not `failed`.** The end
  of a command is read from the terminal's foreground process, which gives no exit code.
- **With a shell that marks its commands, a failed command is `failed`**, with a note
  such as "exited with 2 after 1m 35s".
  - fish 4 and later does this with no setup.
  - For bash, put `eval "$(isc shell-integration bash)"` in `~/.bashrc`. For zsh, put
    `eval "$(isc shell-integration zsh)"` in `~/.zshrc`. For a fish older than 4, put
    `isc shell-integration fish | source` in `~/.config/fish/config.fish`. Each one does
    nothing outside a pane.
  - bash must be 4.4 or later. An older one, such as the `/bin/bash` on a Mac (3.2), is
    left alone, and a command that fails in it shows as `done`.
  - Other terminals' integrations that write the same marks (OSC 133 C and D) work too.
  - A mark ends a `running` or `waiting` that was reported, or a wait for a password,
    only when the shell has the terminal back. A command that's still running can
    print the marks, and that changes nothing an agent reported.
  - The shell meant there is the one a pane starts by itself. In a pane you started
    with a command, a shell among them (`isc run -- bash`), neither a mark nor the
    command's end takes down what was reported or a wait for a password. A long
    command there still shows `done` or `failed` when nothing was reported.
  - Any program can print a mark, and a file shown with `cat` can hold one. In a shell
    that marks nothing, a command's end is still read from the foreground process:
    when a command ends and no mark says so within a moment, the marks seen so far
    are taken to have been printed by something else.
- An exit status of 130, which is what you get when you press Ctrl+C, counts as
  finished, not failed.
- **A short command isn't marked at all**, even if it fails. If it ran for under 30
  seconds in a pane that wasn't `running`, nothing is shown.

---

## 11. Agents

insensical learns a coding agent's state in two ways.

- **With no setup**, from what agents already write to their terminal: their own
  reports of what they're doing, which Claude Code makes from version 2.1.295, and
  progress, notifications and the bell, all as described in section 10.
- **Exactly**, from the agent's own hooks, which call `isc status`. insensical can add
  those hooks for Claude Code, Codex, Gemini CLI, opencode and pi.

### Setting up from the window

When the window opens and there's an agent on the system that doesn't report yet, the
top bar says so. For example: "Codex does not show its state yet · Set up".

- Press it to list those agents, with the file each change goes in.
- Press Enter to add them. **Nothing is ever added without that Enter.**
- Press Escape to leave it for later. The notice stays.
- Press `N`, or click the cross on the notice, to say never. Those agents aren't
  offered again.
- After adding, a line tells you what's still yours to do. For Codex, that's trusting
  the hooks with `/hooks`.
- "Have the agents on this system say what they are doing" in the palette asks at any
  time, and includes the agents you declined.

The agents you declined are kept in `~/.local/state/insensical/agents-declined`, one
per line. Delete a line to be offered that agent again.

### Setting up from the command line

| Command | Does |
|---|---|
| `isc integrate` | Finds the agents on this system and offers each one. It says which were found and which weren't, then for each one found it shows what would change and asks. |
| `isc integrate claude` | The same for one agent: `claude`, `codex`, `gemini`, `opencode` or `pi`. |
| `isc integrate --yes` | Adds every agent that was found, without asking. |
| `isc integrate --remove` | Removes them all. If you name an agent, removes that one. |
| `isc integrate --status` | Prints what was found and what already reports, as JSON. `stale` is true for an agent whose hooks are there but name an `isc` that is no longer this one. Changes nothing. |

- An agent counts as found when its program is on `PATH` or its settings directory
  exists.
- An agent that's already integrated says "nothing to change".
- Nothing is added without asking, and nothing is added for an agent that isn't on the
  system.
- When there's no terminal to ask on, `isc integrate` changes nothing unless you pass
  `--yes`, and exits with status 1.
- If the agent's settings changed while you were being asked, as when the agent wrote
  to them itself, nothing is written and you're told so. Ask again.
- **With `--host`, it's the other machine's agents.** `isc --host build integrate
  --status` says what's there, and `isc --host build integrate claude --yes` adds the
  hooks there. The daemon on that machine does it, with its own `isc`, so it must be
  running. You have to give `--yes` or `--remove --yes`: nobody can be asked on the
  other machine, so without it the command is refused.
- You can't combine `--status` with `--remove` or `--yes`. For each agent it prints
  `agent` (the name used on the command line), `title`, `found`, `integrated`, `stale`,
  `file` (where its hooks go) and `then` (what's left for you to do afterwards, or
  null).

**The hooks name the `isc` binary by the full path it had when you ran `integrate`.**
If you move `isc` or reinstall it somewhere else, the hooks point at nothing. The
agent's state stops being reported, `--status` says it isn't integrated and that its
hooks are stale, and the window offers to set it up again. Run `isc integrate` for that
agent to rewrite the hooks so they name the `isc` you ran.

- A path with a space or a shell's own characters in it is written between quotes, so
  the agent's shell reads it as one name.
- A hook counts as insensical's when the program it runs is called `isc` and is
  followed by `hook` and the agent's name, and nothing else. A hook of your own that
  only ends the same way, such as `mytool hook claude`, is never changed or removed.
- An agent's settings file is replaced in one step and keeps its permissions, and so
  does the copy saved beside it. If the settings file is a link to a file you keep
  somewhere else, the link stays and that file is the one replaced.
- A settings file of more than 16 MB isn't read, and nothing is changed. The same goes
  for one that isn't a file, such as a pipe or a directory, or that can't be read as
  text, and for the script opencode or pi loads: `isc integrate` says so and leaves
  what's there as it is.
- `CLAUDE_CONFIG_DIR` and the other agents' variables count only when they hold a
  whole path, from `/`. One that holds part of a path is ignored, and the agent's usual
  directory is used.

### Claude Code

`isc integrate claude` adds hooks to Claude Code's settings: `~/.claude/settings.json`,
or under `$CLAUDE_CONFIG_DIR`.

- Hooks that are already there are kept, and stay first.
- A copy of your original settings is saved next to them, as
  `settings.json.before-insensical`. It's written once.
- `--remove` takes the hooks out and leaves what was there before.
- The hook prints nothing, never fails, runs separately from Claude's own work, and
  does nothing outside a pane.

| Claude Code event | State |
|---|---|
| A prompt is submitted; a tool has run | `running` |
| Permission is asked for; it asks a question | `waiting`, with what it's asking |
| It has finished responding | `done`, with the first line of what it said. It stays `done` no matter how long it then sits at its prompt. |
| It stopped on an error | `failed` |
| A session starts or ends | `idle` |

Each event also names Claude's session. That's how the pane can offer
`claude --resume …` after the daemon has stopped (section 17).

When Claude runs several tools at once, one can finish while another is still waiting
for your answer. The finished tool doesn't change the row: it keeps saying `waiting`
and what Claude asked for, until that tool has run, a new tool starts, the turn ends or
you send a new prompt. The hook remembers what was asked in a small file under the state directory,
`hook-asks`. It removes the file when the turn or the session ends, and anything older
than a day. Codex works the same way.

- A tool is told from another by its name and what it was given. Two tools with the
  same name and the same input at once can't be told apart, so one finishing counts as
  the answer.
- If you refuse a request, Claude sends nothing for it. The row goes on saying
  `waiting` until Claude starts its next tool, which a blocked Claude can't do, and
  then it says `running`. A tool that was already running can finish in the meantime
  without changing the row. If Claude has nothing more to run after a refusal, the row
  stays on `waiting` until the turn ends or you send a prompt. A tool that starts within
  two seconds of the question is taken to have been started together with it, and
  changes nothing. Codex and Gemini CLI work the same way, going by when a tool starts.

Claude Code 2.1.295 and later also reports in its own terminal (section 10). Where it
does, that's what the pane shows, and the hooks' reports of its state are set aside.
The hooks are still worth having: they're what names the session to resume.

### Codex

`isc integrate codex` adds hooks to `~/.codex/hooks.json`, or under `$CODEX_HOME`. A
copy of the original file is saved next to it, as `hooks.json.before-insensical`. It's
written once.

- **Codex does not run a hook until you have trusted it.** After integrating, open
  Codex, run `/hooks`, and trust the new ones. Until you do, nothing is reported.
- If `isc` moves and you add the hooks again, you have to trust them again.
- A prompt or a finished tool gives `running`. A request for approval gives `waiting`.
  The end of a turn gives `done`, with the first line of what it said. An interrupted
  turn gives `idle`.
- A turn that fails isn't reported, because Codex has no event for it.
- The pane offers `codex resume <session>` after the daemon has stopped.

### Gemini CLI

`isc integrate gemini` adds hooks to `~/.gemini/settings.json`, or to
`$GEMINI_CLI_HOME/.gemini/settings.json`. A copy of your original settings is saved
next to them, as `settings.json.before-insensical`. It's written once.

- A prompt or a finished tool gives `running`. Asking to use a tool gives `waiting`,
  with its question. The end of a turn gives `done`, with the first line of its answer.
- A failed or cancelled turn isn't reported. Gemini CLI has no event for either, so
  the pane stays `running` until the next prompt.
- Gemini's question isn't always said to be about one tool. When it names the tool, a
  different tool finishing leaves the row `waiting`, as it does for Claude. When it
  doesn't, the first tool to finish counts as the answer.
- No resume command is offered.
- Gemini waits for its hooks. The hook answers at once. If the daemon is stuck, Gemini
  is held up for three seconds at most.

### opencode

`isc integrate opencode` writes one file, `~/.config/opencode/plugins/insensical.js`
(under `$XDG_CONFIG_HOME` if that's set). opencode loads it when it starts. `--remove`
deletes it.

- Working gives `running`. Asking for permission or asking a question gives `waiting`.
  Going quiet after working gives `done`. An error gives `failed`, with its message. A
  turn you stopped gives `idle`.
- Sessions that opencode starts itself (subagents) aren't reported.
- The pane offers `opencode --session <id>` after the daemon has stopped.

### pi

`isc integrate pi` writes one file, `~/.pi/agent/extensions/insensical.ts`, or under
`$PI_CODING_AGENT_DIR`. pi loads it when it starts. `--remove` deletes it.

- Working gives `running`. A dialog that an extension puts up gives `waiting`, with its
  title (pi itself never asks before using a tool). Finishing gives `done`, with what it
  said last. Ending in an error gives `failed`. Stopping it yourself gives `idle`.
- pi reports only when it's running with a screen. Its other modes don't report.
- The pane offers `pi --session` with the session's file, or its id when it has no file,
  after the daemon has stopped.

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
- `isc status` and `isc notify` speak *for* a pane, and never guess which one. Inside a
  pane they need nothing more. Outside one they need `--pane`.
- A message, a title or a body can begin with a hyphen: `isc status done -m "- fixed"`
  reports `- fixed`. In a script, `--message="$text"` keeps the text and the option
  together whatever the text holds.
- A hook that's told more than a megabyte, such as everything a tool printed, reads
  the start of it, where the agent says which moment this is.
- These two commands, and the agents' hooks, work even when `isc` and the daemon are
  different versions.

### Agents that open their helpers in tmux panes

Some agents can run a team: a lead that starts helpers, each in a pane of its own. They
do it by running `tmux` commands, and only when they find themselves inside tmux. In an
insensical pane there's no tmux, so the agent keeps its helpers out of sight or doesn't
start them.

With the `tmux-shim` setting on, the agent's helpers open as panes here. You see each
one beside the agent that started it, in the same tab. You can move it, name it, zoom
it and be told about it like any pane.

**Turn it on:**

```sh
isc config set tmux-shim true
```

or switch on "Answer as tmux" in the settings sheet, under Terminal. It's off until you
turn it on, and it applies to panes started from then on. Start your agent in a new
pane.

**What changes in a pane while it's on:**

- `tmux` is found first on `PATH`, and it's `isc` under that name. `isc tmux ...` is the
  same thing.
- `TMUX` and `TMUX_PANE` are set, which is how a program decides it's inside tmux.
- **The real tmux isn't what `tmux` runs in that pane.** If you use tmux yourself inside
  insensical, run it by its full path, such as `/usr/bin/tmux`, or leave the setting off.
- Nothing outside insensical's panes changes. With the setting off, nothing changes at
  all.

**How tmux's words map onto yours:**

| tmux | insensical | Written as |
|---|---|---|
| session | project | its name, or `$3` for project 3 |
| window | tab | its place in the project counted from 0, its name, or `@7` for tab 7 |
| pane | pane | `%5` for pane 5, or its place in the tab counted from 0 |

A target is written as tmux writes it: `%5`, `@7`, `api:0.1`, `api:logs`, `:1`. With
no target, a command means the pane it's run in.

**The commands that are answered:**

| Command | What it does here |
|---|---|
| `split-window` | Opens a pane beside the target, in its tab. `-h` beside it, otherwise below; `-b` before it; `-l` and `-p` for its size; `-c` for its directory; `-e` for a variable; `-d` to leave the keyboard where it was; `-P -F` to print it. |
| `new-window` | Opens a tab in the same project. `-n` names it; `-c`, `-e`, `-d` and `-P -F` as above. |
| `new-session` | Opens a project. `-s` names it. It never takes your keyboard. |
| `respawn-pane -k` | Runs another command in a pane that `split-window` or `new-window` opened with a command. The pane keeps its number. |
| `send-keys` | Types into a pane. `Enter`, `Escape`, `Tab`, `Space`, `BSpace`, the arrows, `Home`, `End`, `PgUp`, `PgDn`, `F1` to `F12`, `C-c` and `M-x` are keys; anything else is typed as written. `-l` types every word as written. |
| `kill-pane`, `kill-window` | Closes the pane, or the tab. |
| `select-pane`, `select-window` | Gives the pane or tab the keyboard. `select-pane -T` names a pane instead. |
| `rename-window` | Names a tab. |
| `list-panes`, `list-windows`, `list-sessions` | Lists them, one a line, with `-F` for the format. |
| `display-message -p` | Prints a format for the target. |
| `capture-pane -p` | Prints what a pane shows. `-S` and `-E` choose lines, with lines below 0 being what scrolled away; `-e` keeps colours. |
| `resize-pane` | Moves the pane's divider: `-L`, `-R`, `-U`, `-D` by so many cells, `-x` and `-y` to a size in cells or as `30%` of the tab. `-Z` zooms. |
| `select-layout` | Arranges the tab's tiled panes as `even-horizontal`, `even-vertical`, `main-vertical`, `main-horizontal` or `tiled`. |
| `has-session` | Succeeds if there's a project of that name. |
| `-V` | Prints `tmux 3.5a`. |

- **Formats** know `pane_id`, `pane_index`, `pane_title`, `pane_current_command`,
  `pane_current_path`, `pane_start_command`, `pane_active`, `pane_width`, `pane_height`,
  `pane_dead`, `pane_dead_status`, `window_id`, `window_index`, `window_name`,
  `window_active`, `window_panes`, `window_zoomed_flag`, `session_name`, `session_id`,
  `session_windows`, `session_attached`, `client_control_mode` and `version`, the short
  forms `#S`, `#I`, `#W`, `#P`, `#D` and `#T`, and `#{?name,this,that}`.
- **Commands that set tmux up are accepted and do nothing**: `set-option`, `set`,
  `set-window-option`, `bind-key`, `unbind-key`, `source-file`, `set-hook`,
  `set-environment`, `start-server` and `refresh-client`. A pane's border colour is
  tmux's to draw, and there's none here.
- **`kill-server` is refused.** It would stop the daemon and every pane you have. Only
  `isc kill-server` does that.
- Errors are tmux's own, with exit status 1: `can't find pane: %9`,
  `can't find session: work`, `unknown command: frobnicate`.
- Several commands can follow each other with `;` between them.

**Who opened a pane** is in the activity record: it names the pane whose program asked.
A pane opened with a command is named after that command until something names it.

**Limits:**

- A command tmux has that isn't in the table is `unknown command`.
- `pane_pid` and any format name not listed print nothing.
- A pane closes when its program ends. tmux's `remain-on-exit` isn't kept.
- `respawn-pane` needs `-k` and a command, and only works on a pane opened through
  `tmux` with a command.
- `capture-pane` needs `-p`: there are no paste buffers. `-J` doesn't join lines the
  terminal wrapped.
- `main-vertical` and `main-horizontal` give the main pane half the tab. Size it
  afterwards with `resize-pane`.
- Opening a pane, and arranging a tab that isn't in the wanted shape yet, bring that
  tab to the front for a moment. With `-d` the keyboard then goes back where it was. An
  agent working in a tab you aren't looking at will show you its tab briefly each time.
- A new pane's variables set with `-e` reach a pane started with a command, not one
  that starts your shell.
- If your shell's start-up file replaces `PATH` instead of adding to it, a program you
  start from that shell won't find `tmux`. Keep the directories already in `PATH`.
- A session can't be attached to, and `tmux` with no command says so.

---

## 12. Notifications

**When you're told**: a pane becomes `waiting`, `done` or `failed`; a program sends a
notification sequence; or `isc notify TITLE [BODY]` is called.

With no window and no terminal attached, the daemon can run a command or post to a web
address: see [When nobody is attached](#when-nobody-is-attached).

**One thing is said once.** What your panes have to say is gathered before you're
told. The daemon does it, so it's the same on the desktop, in the window, in a browser
and for `notify-command` and `notify-url`.

- **A pane that waits for you or failed is told of a second after it changes.** That
  second is the wait to see whether other panes say the same.
- **A finish is told once it has stood.** A pane that goes `done` is told of when it
  has stayed done for 15 seconds, and a second more for other panes to join it. Its row,
  the activity list and the bell's count show `done` at once: only the notification
  waits. Looking at the pane, or typing in it, during that time means you've seen it,
  and you aren't told.
- **A pause isn't a finish.** An agent that waits on work in the background ends its
  turn, wakes by itself when a part is done, says it's still waiting and ends its turn
  again. If a pane goes back to work by itself before its finish has stood, you aren't
  told of that finish at all. When it stops for good you get one notification, with
  its last words.
- **A pane that pauses is waited on for longer.** Once a pane has gone back to work by
  itself, its next finish is kept for one and a half times its longest recent pause,
  never less than 15 seconds and never more than ten minutes. That ends when one of
  its finishes stands, or when you type in the pane: work you asked for by typing is
  told of after the usual 15 seconds.
- A pane's first pause that lasts longer than 15 seconds is told of as a finish:
  nothing was known of the pane yet. So is a pause longer than the pane is being
  waited on for.
- **A program's own notification about its finish is that finish.** If an agent's
  hooks say it finished and the agent also sends a notification sequence for the same
  stop, you get one notification, in the finish's words. This holds for two minutes
  after the finish.
- **Panes that start waiting, fail or finish together are one notification.** It says
  how many, as in `10 panes failed`. Under that are their words if they all said the
  same, such as a usage limit's message. If they didn't, the first three are named,
  with `and 7 more`. It's sent once no other pane has joined for a second, and three
  seconds after the first one at the latest. Click it to go to the first of them.
  Each pane's own row still shows its own state.
- **Waiting, failed and finished are gathered apart.** So at most one of each arrives,
  and the one about panes that wait for you comes first.
- **The same news from more panes is waited on longer.** After you've been told of a
  failure or a finish, a pane that fails or finishes with the same words within two
  minutes is the rest of that news. Those are gathered until none has joined for ten
  seconds, thirty at the most, and come as `4 more panes failed`. A pane that waits
  for you is never held for this.
- **A pane that says the same thing again isn't told of again.** An agent that retries
  and fails the same way every few seconds is one notification. You're told again when
  it says something different, when it's gone ten minutes without saying it, or once
  you've looked at the pane.
- **A pane says one thing at a time.** If it says two within its second, you get the
  one that needs you more, or else the later.
- No setting changes any of this.

**Where you're told depends on where you are.** If insensical is in front, you get a
notice inside the window and nothing on your desktop. If it isn't in front, or has no
window open, you get a notification on your desktop.

**The notice in the window** shows at the foot of the sidebar. With the sidebar hidden,
or the tabs along the top, it lies over the bottom right corner of the window instead.
It resizes no terminal.

- It has the pane's state mark, `pane · project`, and what the pane says, on up to two
  lines.
- Click it to go to the pane. Its `×` closes it.
- It shows only if the pane still needs you a second later. An agent that finishes and
  starts again at once says nothing.
- **A finish goes after six seconds.** It stays for as long as the pointer is on it,
  and the six seconds don't run while insensical isn't in front.
- **A pane that waits or failed keeps its notice** until you've seen the pane or closed
  the notice. Any notice goes as soon as you've seen its pane.
- **Three show at once at most**, the newest lowest, and fewer in a short window:
  together they take no more than a third of its height. A line above them counts the
  rest, as in `and 2 more`; click it for the overview.
- **When the daemon refuses something you asked for**, such as a new tab in a project
  that's asleep, a notice headed "It could not be done" says why, in the daemon's
  words. It leads to no pane and goes after six seconds.

**Sound.** A pane that waits or failed makes a short sound of two rising notes. With
`sound = "all"` a finish makes a softer single note; with `sound = "off"` nothing makes
a sound. The default is `needs-you`. The settings sheet has a button to hear each.

- Whichever program tells you makes the sound: the window for a notice, and for a
  desktop notification the daemon on Linux or the application on macOS.
- It's played by `pw-play`, `paplay` or `aplay` on Linux, whichever is found first, and
  by `afplay` on macOS. With none of them there's no sound, and nothing says so.
- It isn't silenced by the desktop's do-not-disturb, since insensical plays it itself.
- The window and the application don't start a sound within 300 ms of the last one
  either of them started. The daemon's sounds aren't held to that: each goes with a
  desktop notification, and a pane can send only so many of those (see below).
- You can't use sounds of your own.

**Turning them off.** `notifications = false` stops all of them. `notify-waiting`,
`notify-failed` and `notify-done` turn off one kind each. A pane still shows its state
whatever these say. The daemon reads them each time, so a change holds from the next
one on.

**When you're not told**: you're looking at the pane (it has the keyboard, in a window
that's in front); the pane is muted; or the cause is the bell.

- Each pane has one notification. A new one replaces the pane's previous one, and it's
  removed once you've seen the pane. One about several panes is the first pane's: it's
  removed once you've seen that pane.
- A pane can say at most three things in quick succession, and earns another every
  second and a half. Any beyond that are dropped, before the gathering above.
- Where the application puts them on the desktop and not the daemon, as on a Mac, it
  puts at most five there at once, and two a second after that, for all panes
  together. What's held back isn't lost: the last one from each pane is put there when
  there's room, the pane that has waited longest first. A title is cut to 120
  characters and the text under it to 600.
- The title is `project · pane`. The pane part is the pane's name if it has one,
  otherwise its title, otherwise its program. The body is what the program said, or
  "Waiting for you", "Done", "Failed".
- Text from programs is cleaned up first: one line, no control characters, limited
  length. In the title, `<` and `>` are written as `‹` and `›`, and `&` as `+`, because
  some desktops read a title as markup.
- Whether your desktop reads the body as markup is asked for each notification, so a
  notification service started or changed while insensical runs is written to rightly.
- They're sent at normal urgency, so they respect do-not-disturb.

**Muting a pane.** Use the `mute` command in the palette (it has no default key), or
`isc mute [PANE] [--off]`. The pane's states still show, but it sends no notifications.
The header shows a mute mark.

**Muting a project.** Use "Silence this project's notifications" in the palette (the
`mute-project` command; it has no default key), or `isc project mute NAME [--off]`.
None of the project's panes sends a notification, including panes you open in it
afterwards. Their states still show, and the project's name in the sidebar has a mute
mark. Muting a project doesn't change which of its panes are muted on their own.

**On Linux** the daemon sends them over D-Bus, so they arrive even with no window open.
They name the desktop entry `insensical` and carry its icon. If the entry and the icon
aren't installed, the desktop may show them with no application name or icon.

**Clicking one** goes to the pane, restores it if it was minimized, and asks a window
to come forward.

- **On Wayland the window may not come to the front.** A compositor only lets a window
  raise itself with a token that the notification service hands over, and the toolkit
  can't use one. The pane is selected, but the window may stay where it is. Hyprland
  brings it forward, with `misc:focus_on_activate` on.
- With no window open, one opens on the pane. The daemon starts `insensical` from the
  directory `isc` is in.
- A window on another workspace asks to come forward at once. Whether it's let to is
  the compositor's to say.

**On Linux, an icon in the tray says what needs you**, with no window open too. The
daemon puts it there, so it's there for as long as your programs are.

- **The icon** is insensical's own, with an amber dot on its corner while any pane
  needs you: one that waits or failed. Point at it for how many. A pane that finished
  isn't one of them.
- **Its menu** lists those panes, the most urgent first, each with a sign for its
  state, its project and name, and what it says. Pick one to go to that pane. Nine are
  named and the rest counted. Below them: how many panes are running, and Open
  insensical, which reads Show insensical while a window is open. Clicking the icon
  itself does what that line does.
  A window opened that way is on the project you were last at, on whichever machine
  that was, for as long as insensical has been running. If that machine can't be
  reached at that moment, it opens on this computer.
- **Close the window** is there while a window is open. It closes every window and
  ends nothing: your programs run on, and the icon stays.
- **Quit insensical** stops the server, which ends every program in every pane. It
  asks first, in a window, opening one if there is none: the question names the panes
  that would end and says what's kept. Nothing stops until you answer **End
  everything**; `Esc` leaves it all running. Your projects and panes are kept, and each
  comes back with a new shell when you open insensical again. If the window it opens
  for the question hasn't come within ten seconds, the question is dropped: choose
  **Quit insensical** again.
- **With no window open, one opens**, and with one open it's asked to come forward,
  both as for a clicked notification: on Wayland it may not be able to.
- **insensical running in a terminal isn't a window.** **Close the window** leaves it
  open. With only that open, the menu says Open insensical, and picking a pane opens a
  window on it. If it's the interface you used last, **Quit insensical** asks its
  question there.
- **Not every desktop has a tray.** It needs one that shows status notifier items: KDE
  Plasma, Waybar and most panels do, and GNOME does only with the AppIndicator
  extension. Nothing depends on the icon: notifications arrive without it.
- The icon is found in the installed icon theme. If insensical's icons aren't
  installed, the tray shows a blank or a placeholder.
- `tray = false` takes the icon away, and **Show in the tray** under **Notifications** in
  the settings is the same switch. The icon follows it within a couple of seconds.

**On macOS** the application posts them, not the daemon, and only from an app bundle.
They arrive with the window closed too, for as long as the application is running. The
system asks once whether to allow them. Click one to bring the window to the front on
the pane it's about; with no window open, one opens there. None arrive once you quit
the application.

**On macOS, the menu bar and the Dock say what needs you**, so you can tell with the
window closed or behind other work:

- **In the menu bar**, insensical's mark, with the number of panes that need you
  beside it: those that wait or failed. It's the same number the window shows. With none, the mark is alone; with no server to hear
  from, it's dimmed. It never changes colour and never moves.
- **Its menu** lists those panes, the most urgent first, each with its state's mark,
  the same marks the window draws, its project and name, and what it says. Pick one
  and insensical comes forward on that pane, opening a window if there is none. Nine
  are named and the rest counted; the count opens the overview. Below them: how many
  panes are running, Open insensical, Settings, Restart when an update is installed,
  and Quit.
- **On the Dock icon**, the same number as a badge, and the same panes in the menu you
  get by right-clicking the icon. The system can hide a menu bar item when the bar is
  crowded or behind a notch; the Dock always has it. If the number doesn't appear on
  the icon, turn on **Badge application icon** for insensical in System Settings,
  Notifications: macOS shows a badge only where that's allowed.
- **The settings say what macOS allows.** At the top of **Notifications**, a line says
  whether macOS lets insensical notify you, and whether it lets it put a count on its
  Dock icon. If either is off, the line is marked and says so, since nothing you switch
  on below it arrives until macOS allows it. **Open System Settings** takes you to
  insensical's notifications there. The line follows what you change in System Settings
  within a couple of seconds.
- `menu-bar = false` takes the item out of the menu bar, and `dock-badge = false` the
  number off the Dock icon. Both are in the settings, under **Menu bar and Dock**, a
  section only a Mac has. Dragging the item out of the bar with `⌘` held turns
  `menu-bar` off for you.

There's no count on the application's icon on Linux. You can't choose to get desktop
notifications while insensical is in front, or notices instead of them when it isn't.

### When nobody is attached

An agent on a server stops to ask you something at two in the morning. No window is
open, no terminal shows insensical, so nothing above reaches you. The daemon is still
running, and it can tell you two ways: it runs a command of yours, or it posts to a web
address of yours. Both go outward only. Nothing listens, and there's no service of
insensical's in between.

**The two settings.** They're the daemon's, so set them on the machine the agent runs
on. From another machine, put `--host NAME` before each command.

```sh
isc config set notify-command /home/you/bin/tell-me
isc config set notify-url https://ntfy.sh/a-topic-only-you-know
```

Set one or both. Then try them:

```sh
isc notify --test
```

That sends one through each, whoever is attached, and prints how it went:

```
notify-command: ran, exit status: 0, in 0.3 s
notify-url: posted, HTTP 200, in 0.4 s
```

It exits with an error if either failed. `isc notify --sinks` shows what's set, how the
last one sent to each went, and how many weren't sent.

**What gets sent** is exactly what would have been a notification: the same kinds, the
same mutes, the same `notify-waiting`, `notify-failed` and `notify-done` switches, the
same limit of three in quick succession for a pane, the same gathering of many panes
into one. For several panes the title is the count, as in `10 panes failed`, and the
pane, project and program are the first pane's.

**When it's sent.** By default only when you're away: no window, no interface in a
terminal and no browser is attached to that daemon. `isc events`, a script, a
program in a pane and anything attached with only the `look` right don't count as
someone being there. To be told every time, whoever
is attached:

```sh
isc config set notify-command-when always
isc config set notify-url-when always
```

**The command** is a program and its arguments, cut at the spaces. It's given no shell,
so `|`, `$VAR` and quotes mean nothing in the setting. Put anything longer in a script
and name the script. What happened is in the command's environment:

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

A title or a message is whatever a program wrote, so it's only ever a value in the
environment and never part of a command line. In your script, keep it in double
quotes: `"$INSENSICAL_BODY"`.

The rest of the command's environment is the daemon's. If that has
`INSENSICAL_SOCKET`, an `isc` your script runs reaches the same daemon. The script
isn't running in a pane. `INSENSICAL_PANE` is the pane the notification is about, so
an `isc` command in the script that would act on "this pane" acts on that one, and
`isc clients` lists it as claimed, not verified.

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

`isc config get notify-url-headers` shows each header with its value masked, and so
does everything else that shows a setting. `isc config reset notify-url-headers` takes
them all out. The settings sheets don't list the headers.

A topic that needs you to sign in takes a token or a user and a password:

```sh
isc config set notify-url-token tk_yourtoken
isc config set notify-url-user me
isc config set notify-url-password "your password"
```

A command line can be read by other users of the machine while it runs, so a secret is
better given on standard input: `printf %s "$TOKEN" | isc config set --stdin notify-url-token`
reads the value to the end of its input, less one final newline.

A token is sent as `Authorization: Bearer …`. A user and password are sent as Basic.
With both set, the token is used. They stand in place of an `Authorization` header you
wrote in `notify-url-headers`. They're secrets: `isc config get`, `isc config`,
`isc notify --sinks` and the settings sheets show that one is set (a token by its last
four characters, a user and a password by nothing), never its value, and no error or
log holds it. `isc notify --sinks` says `signed in with a token` or `signed in as a user`.
They're never sent over plain `http://` to another machine, and neither is an
`Authorization` or `Proxy-Authorization` header you wrote in `notify-url-headers`: the
post fails with a line saying so, and you use an `https://` address. An address on this machine
(`127.0.0.1`, `localhost`, `[::1]`) may be `http://`. insensical doesn't check who can
read the settings file, so keep it to yourself (`chmod 600`).

**Who sees the address and the command.** Whoever has the address of an ntfy topic can
read it and post to it, and a command's arguments are where a token usually ends up.
So the daemon shows both in full only to a connection with the `manage` right, which
is every connection of your own unless you asked for fewer. Without `manage`, such as
a browser paired with a look-only token, `notify-url` is shown as its scheme and host
(`https://ntfy.sh`) and `notify-command` as the program's name (`tell-me`), in the
settings and in `isc notify --sinks`. `isc notify --sinks` shows the address without
its path to everyone.

**ntfy.** Pick a topic nobody would guess; it's the only secret. In the settings file
on the machine the agent runs on:

```toml
notify-url = "https://ntfy.sh/a-topic-only-you-know"
notify-url-body = "text"

[notify-url-headers]
Priority = "high"
Tags = "robot"
```

Subscribe to the topic in the ntfy app, then run `isc notify --test`. For a server of
your own with access tokens, add the `Authorization` header as above.

**Pushover, Telegram, Slack and anything else** want a body of their own shape, so
use the command and let `curl` shape it. For Pushover, save this as
`/home/you/bin/tell-me` and make it executable with `chmod +x`:

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

**A desktop you're logged in to, with a daemon that can't reach it.** A daemon started
over `ssh` doesn't know the desktop's session, so it shows no notifications there.
`notify-send` can, told where the session is:

```sh
#!/bin/sh
export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$(id -u)/bus"
exec notify-send --app-name=insensical "$INSENSICAL_TITLE" "$INSENSICAL_BODY"
```

**Limits.**

- **Away means nothing is attached, not that you're not looking.** A window left open
  on your desk while you're at lunch counts as you being there. Use `always` if you
  want your phone to hear of it too.
- **The command gets ten seconds.** After that it's ended, with whatever it started.
  Its output is thrown away. It runs in your home directory with the daemon's
  environment, so name it by its full path: `~` isn't expanded, and a path with a
  space in it can't be named.
- **The address gets ten seconds too.** A redirect isn't followed. The answer is
  ignored, apart from its status.
- **Four are sent at once at most.** One more isn't queued: it's dropped and counted,
  and `isc notify --sinks` shows the count.
- **A failure tells you nothing.** It's kept for `isc notify --sinks` and isn't itself a
  notification, so test after you change anything.
- **`https` checks the certificate against the authorities built into `isc`,** not the
  ones your system trusts. A server with a certificate of your own making is refused:
  use the command with `curl` for it.
- **`https` needs a processor from about 2014 or later** on x86-64 (AES, AVX2 and ADX),
  and on ARM one with the AES and SHA-2 extensions, which a Raspberry Pi 4 lacks.
  Without them `isc notify --test` says so; use the command with `curl`. Plain `http`
  works anywhere.
- **To post through a proxy**, start the daemon with `ALL_PROXY`, `HTTPS_PROXY` or
  `HTTP_PROXY` in its environment, and `NO_PROXY` for hosts to reach directly. See
  [The daemon's environment](#the-daemons-environment). There's no setting for it.
  A user and password in the proxy's address are never shown: not by
  `isc notify --test`, and not by `isc notify --sinks`.
- There's nothing ready-made for any one service, and nothing is sent anywhere unless
  you set one of the two.

---

## 13. Keys

There are two keymaps. `keymap = "direct"` is the default, and gives every command its own
chord. `keymap = "leader"` puts every command on a key you press after a leader key.
Section 19 has the full table for both.

**The window always spells a key out in full**, joined by `+`: `Ctrl+Shift+T`,
`Shift+H`, `Ctrl+Alt+Left`. That's true in tooltips, the palette, the list that follows
the leader, the sheet of keys and the buttons of a question. A key is named by what you
press, so `?` is shown as `Shift+/`. A capital letter is the key and never stands for
Shift, which is always written. After the leader, `T` is the `t` key alone and `Shift+H`
is the capital.

**On macOS, a key is shown in the signs your keyboard and menus use**, with nothing
between them, in the menus' order: Control `⌃`, Option `⌥`, Shift `⇧`, Command `⌘`, then
the key. So you read `⇧⌘N`, `⌃⌥←` and `⇧/`, in the window and in the interface in a
terminal. Return is `↩`, Delete `⌫`, forward Delete `⌦`, Tab `⇥`, and the arrows are
arrows. Esc, Space, Home, End, PageUp, PageDown and the function keys are words. Keys
pressed one after another are a space apart: `⌃Space T`. A press with the pointer is
written `⌘`-click.

On Linux, the direct keys are `Ctrl+Shift` with a letter. They avoid `Super`, which
window managers own. On macOS, they're `⌘`-based.

**On macOS, closing the window doesn't quit the application, and neither does `⌘Q`.**
insensical lives in the menu bar as well as in its windows. `⌘Q`, and **Close to Menu
Bar** in its menu, put every window away and leave it there, counting the panes that
need you and telling you about them. Open it again from the menu bar's item, the Dock
or Spotlight and it gets a window.

To quit it, choose **Quit insensical** from its menu or from the menu bar item's menu,
or press `⌥⌘Q`. If a pane is running or waiting when you quit, insensical asks first:
on a Mac it's the application that tells you when a pane needs you, and once it's quit
nothing does until you open it again. `Esc` goes back, `W` closes the window and leaves
the application running, `Enter` quits. With nothing running, or no window open, it
quits at once. If you've turned `menu-bar` off there's no item to go to, and `⌘Q` quits.

The menu also has About insensical, which shows its version, `Settings` (`⌘,`), which
opens a window for the settings if there is none, the system's Services, and Hide
(`⌘H`), Hide Others (`⌥⌘H`) and Show All. `⌘H`, `⌥⌘H` and `⌥⌘Q` are the application's
and don't reach a program in a pane.

The first time you close its last window, one notification says that insensical is
still running and how to quit it. It's said once and never again. Whether a window is
open or not, the daemon and every program under it keep running, and the next window
shows them. On Linux, closing the window quits. The `quit` command has no key there,
and is in the palette. To give it one, add a line to `[keys]`, such as
`"ctrl-alt-q" = "quit"`.

**On macOS, Option isn't Alt unless you say so.** Option with a key types the
character the keyboard layout gives it (Option+B is `∫`), so a shell's Alt+B and Alt+F,
and a program's other Alt keys, don't answer. Set `option-as-alt = true` and Option
with a key is Alt to the program in the pane. You then can't type the layout's Option
characters in a pane; it's both Option keys or neither. If the system's own `⌃Space`
("select the previous input source") is on, the system takes it before the leader can.

**A chord in the direct keymap doesn't reach the program in the pane.** `` Ctrl+` ``,
`Ctrl+Shift+Space`, `Ctrl+Shift+I` and `Alt+1` to `Alt+9` are among them. To hand a chord back, set it to
`none` in `[keys]`, use the leader keymap, or lock the keys for as long as the program
needs them.

### Seeing every key

Press `Ctrl+Shift+/` or `Ctrl+F1`, or leader `?`, or choose "Show every key" in the
palette.

A sheet opens over the window, with every command under its heading: projects and tabs,
panes, arranging panes, floating and minimized, finding and going, text, the
application. Beside each command are its keys, each key on a cap. They're the keys of
the keymap in use, with your own changes. Keys you press one after another have a mark
between them.

- A command with no key is listed too, greyed, and says so.
- If a command has no key only because your own `[keys.direct]` or `[keys.leader]` table
  gave its key to another command, or to `none`, the sheet says
  `not in your [keys.direct]` beside it, in the colour of something waiting for you.
- Typing narrows the sheet to what matches, by a command's words, its heading or its
  keys: `split`, `float`, `ctrl alt`.
- **If you press a chord while the sheet is open, the sheet describes it instead of
  running it.** The foot of the sheet shows what you pressed, as the keyboard sent it,
  and the command it runs, or says that it has none and goes to the program. This is
  also how to find out how a key is written.
- Escape, Enter, "Close" or a click outside closes it. It doesn't resize anything.
- The foot of the sheet says which table of `config.toml` to change those keys in.

### Locking the keys

Press `Ctrl+Alt+Shift+L`, in either keymap, or choose "Give every key to the program, or
take them back" in the palette.

**While the keys are locked, every key goes to the program in the pane**, including the
window's own chords and the leader. Use it for a program whose keys clash with the
window's: an editor, another multiplexer, a remote desktop.

- **The window keeps only the key that locks, and that key unlocks.** Copy, paste and
  find go to the program too while locked.
- When you lock the keys, the window says so along its top for a moment, with the key
  that unlocks. After that, the pane with the keyboard shows a "Locked" mark in its
  header, or over its corner when headers are hidden. Click the mark to unlock.
- The pointer works as usual: you can still click tabs, buttons and dividers.
- The lock belongs to the window, not to a pane. It stays on as you go from pane to
  pane, and it's off again the next time the window opens.
- If you set your own lock key in `[keys]`, it must be a single chord. A key that starts
  with the leader doesn't unlock, because the leader goes to the program while locked.

### The leader keymap

```toml
keymap = "leader"
leader = "ctrl-space"   # the default
```

- The chords of the direct keymap are then *not* bound, and reach the programs in the
  panes.
- Copy, paste, paste-selection and find keep their chords in both keymaps.
- **While the leader is waiting, a panel at the bottom of the window lists every key
  that can follow and what it does**, including your own keys. It closes when you press
  the next key, or after a second. It doesn't resize anything.
- **If nothing follows the leader for about a second, the leader is passed to the
  program.** So is a leader followed by a key that isn't bound to anything.
- **Press the leader twice to send the leader's own key to the program**: `Ctrl+Space`,
  `Ctrl+Space` gives the program one `Ctrl+Space`.
- Escape after the leader cancels it and sends the program nothing.
- If the `leader` setting can't be read, that's reported and `ctrl-space` is used.
- Setting `leader` alone changes nothing. `keymap = "leader"` is what turns the leader
  keymap on.

**`Ctrl+Space` is often the key that switches input method** (fcitx, ibus) and, in some
editors, the key for completion. If the desktop takes it, the leader never arrives.
Choose another with `leader = "…"`.

**Repeat.** After you move a pane (`Ctrl+H` `Ctrl+J` `Ctrl+K` `Ctrl+L`), move its edge
(`H` `J` `K` `L`) or move a tab (`<` `>`), the same keys work without the leader for one
second: leader, `L`, `L`, `L`.

- Any other key ends the repeat and goes to the program.
- So does waiting: a second later, `L` types an L.
- Nothing else repeats. Focus doesn't, so typing straight after you move focus is never
  taken for a command.

### Changing keys

```toml
[keys]
"ctrl-shift-y" = "split-right"   # another key for a command
"leader y" = "split-right"       # after the leader, in either keymap
"alt-enter" = "zoom"
"ctrl-shift-m" = "none"          # leave this key to the program
```

- Write a key with what you hold first: `ctrl`, `shift`, `alt`, `cmd` or `super`, joined
  by hyphens, as in `ctrl-shift-t` or `ctrl-alt-left`.
- **You can write a key that has two characters either way** on Linux: `ctrl-shift-]`
  and `ctrl-}` are the same key, and so are `ctrl-shift-/` and `ctrl-?`. Keys are shown
  the first way.
- Separate keys in a sequence with spaces. The word `leader` stands for the leader key,
  the one `leader` is set to, in either keymap.
- Your keys are added to the defaults, and yours win where the two are the same, and
  where one begins the other: a default key sequence is dropped when a key of yours is its
  first key, and a default key is dropped when a sequence of yours begins with it. The
  only way to remove a default is to set its key to `none`.
- The command names are in section 19.
- A name that doesn't exist, or a key that can't be read, is reported in the top bar.
  The rest of your keys still apply.

If two of your own keys conflict, the top bar reports it. These are conflicts:

- the same key for two commands, however it's written (`ctrl-shift-y` and
  `shift-ctrl-y` are the same key);
- a key that's also the start of a sequence, which would wait a second to see which one
  you meant;
- the leader bound to a command itself.

A key set to `none` isn't counted. Replacing a default with your own key isn't a
conflict.

**You can write out a whole keymap**, to see every key in one place and move any of
them:

```toml
[keys.direct]                  # what keymap = "direct" binds
"ctrl-shift-t" = "new-tab"
"ctrl-t" = "split-right"

[keys.leader]                  # what keymap = "leader" binds: the key after the leader
t = "new-tab"
"shift-h" = "resize-left"
```

- **A table lies over that keymap's defaults.** Your lines win. A key or command the
  table doesn't mention keeps its default, so a key added in a later version works
  without your touching the file, even if the file came from `isc config` in an older
  version. A default isn't doubled: if your table gives a default's key to another
  command, or puts its command on another key, that default is dropped. To take a
  default key away, write it with `none`: `"ctrl-shift-t" = "none"`. Taking a line out
  of the table doesn't leave its command without a key: the default comes back. To leave
  a key to the programs in a pane, keep its line and give it `none`.
- Each table is read only in its own keymap. So when `keymap = "leader"`, chords written
  under `[keys.direct]` aren't bound, and reach the programs.
- `[keys]` is still read in both keymaps, on top of the table.
- Copy, paste, paste-selection, find and the key that locks aren't part of either table.
  They're the same in both keymaps, and you change them in `[keys]`.
- You can't change the leader pressed twice, or Escape after the leader.
- `isc config` prints both tables in full, with every command that has no key shown as a
  line to uncomment. A file made from that output lists the commands of the version that
  printed it. A command added by a later version gets its default key, unless your table
  has that key or that command; the sheet of keys marks each command your table took the
  key from.
- A command or key that can't be read is reported in the top bar, which names the table.

**You can make commands of your own.** A command of your own is a name and a list of
steps, written under `[commands]`. It's bound to a key like any other command, and
the palette lists it.

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
- **`{ run = "…" }` runs a program in a new pane.** `in` says where: `"tab"` (a new
  tab, the default), `"right"` or `"down"` (beside the pane in front), or `"float"`.
  `cwd` is the directory it starts in, and `~` is your home on the machine the pane
  is on; without it, the pane starts where a new pane there would. `keep = true`
  leaves the pane open when the program ends, so you can read what it printed.
  `run = ""` runs your shell. The words are split at spaces. When an argument has a
  space in it, write a list: `run = ["sh", "-c", "make && ./serve"]`.
- **`{ type = "…" }` types text into the pane in front**, as if at its keyboard. End
  it with `\n` to press Enter: `{ type = "git status\n" }`.

What to know:

- **The steps run in order, and each waits for the one before.** So after a step that
  opens a pane, the next step's pane in front is the new one:
  `[{ run = "", in = "right" }, { type = "npm test\n" }]` types into the new shell.
- **If a step fails, the rest don't run.** A notice says which step stopped and why.
  The steps before it stay done.
- **A step that would ask first, such as `close-tab` with something running, asks, and
  the steps after it don't run.**
- A name is letters, digits and hyphens, and can't be a command that already exists.
  Without `title`, the palette lists it by its name. The palette finds it by either.
- After the leader, the panel lists your commands under "Your commands".
- **A command that's written wrongly is reported in the top bar with the settings'
  other problems, and only that command is left out.** The rest of the file stands,
  and so do your other commands. A key bound to it is reported too.
- `isc do NAME` runs one from a shell: see [Running things](#running-things).
- A step can't be another command of your own. There are no conditions and no loops.
- Your commands aren't on the sheet of keys. The palette shows each one's key.

**A mode makes plain keys into commands.** Some things you do many times in a row, like
moving a pane's edge. A mode is a set of keys that run commands with no leader and no
chord, for as long as the mode is on. One comes ready: **resize mode**. Turn it on, then
press the arrows or `H` `J` `K` `L` as often as you like to move the edges of the pane
in front. Press Escape when you're done.

- **Turning it on.** In the leader keymap it's leader, then `Shift+R`. In the direct
  keymap it has no key of its own: it's "Resize mode" in the palette, or give
  `resize-mode` a key under `[keys]`. In a terminal, where the leader always works, it's
  leader, then `Shift+R`.
- **While it's on, its keys run their commands and aren't sent to the program.** They
  win over every other binding.
- **Escape or Enter turns it off**, and isn't sent to the program.
- **Any key that isn't the mode's also turns it off, and then does what it always
  does.** So if you start typing, the first letter is typed. Nothing is lost.
- **A panel lists the mode's keys when it starts**, where the leader's panel is. It
  goes at the first key you press, so it isn't over what you're resizing. A small sign
  with the mode's name stays at the bottom until the mode is off. In a terminal the
  sign is at the left of the status line. Neither resizes anything.

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

- **The command that turns a mode on is its name with `-mode` after it**: `resize-mode`,
  `tabs-mode`. Bind it like any other command. The palette lists it by its `title`, or
  by that command name if it has no title.
- **Each key runs one command**: one of insensical's, or one of your own from
  `[commands]`. A key is written as it is under `[keys]`, and is one key, not a
  sequence.
- **`timeout` is a number of seconds.** After that long with no key, the mode turns
  itself off. Without it the mode stays on until you leave it.
- **`[modes.resize]` in your settings takes the place of the one that comes ready**,
  whole: write every key you want it to have.
- A mode's name is letters, digits and hyphens. `NAME-mode` can't be a command that
  already exists, so there can't be a mode called `copy`.
- **A mode that's written wrongly is reported in the top bar with the settings' other
  problems, and only that mode is left out.** That covers a command that doesn't
  exist, a key that can't be read and a mode with no keys.
- A key in a mode can't turn on another mode.
- When a mode's key opens something that takes the keyboard, such as the palette, the
  mode turns off.
- The page in a browser has no modes.

### Keys that are fixed

| Where | Keys |
|---|---|
| The sidebar field, the palette, the search field | Typing, Backspace, Enter, Escape |
| A held paste | Enter pastes, Escape drops it |
| A kept pane whose program has ended | Enter closes it |
| The restart question | Enter restarts, Escape doesn't |
| The question before a tab or a project closes | Enter closes, Escape doesn't |
| The new project's name and directory | Typing, Backspace, Tab, Shift+Tab, Up, Down, Enter, Escape |
| The question about agents | Enter adds the hooks, Escape doesn't, `N` stops the offer |
| Adding a machine | Typing, Backspace, Up, Down; Enter meets the marked machine, then gives the answer of the coloured button; Escape leaves |
| Where one of several machines is chosen: a new project, the settings sheet | `Alt+M` goes to the next machine |

Every question that opens over the window looks the same. The window dims behind it, a
card says what's being asked and lists what it's about, and the answers are buttons at
the foot of the card. Each button names the key that gives the same answer, so you can
answer with the pointer or the keyboard. The answer Enter gives is the coloured button
on the right: the accent colour when it doesn't end anything, red when it does.

---

## 14. The command palette and the overview

### The palette

Press `Ctrl+Shift+P`, or leader `Space`. With nothing typed, the palette lists
everything under headings, in this order: **Needs you**, the panes that need you;
**Recent**, the last five commands and settings you chose here; **Commands**;
**Settings**; **Projects**; **Panes**; **Themes**.

- Commands of your own, from `[commands]` in the settings, are listed with the others
  under **Commands**. So is the command that turns on each mode from `[modes]`.
- Each row starts with an icon: a pane's program, or its state mark if it needs you; a
  folder for a project; a cog for a setting; a palette for a theme.
- A command shows its keys at the end of its row, as key caps. A setting shows its
  value. A pane shows its project, and what it says if it needs you.
- **Typing narrows the list** by any of the words you type, in any order, and the
  headings go. The best fit comes first: a word at the start of a name beats one at
  the start of a later word, which beats one inside a word. Of two that fit alike, the
  one you chose more recently comes first.
- You can type what kind of thing you want as a word: `pane`, `project`, `setting`,
  `theme`.
- Enter runs the command, goes to the project or pane, or chooses the theme.
- **Enter on a setting opens it on the settings sheet**, with its list open if it has
  one. The palette never changes a setting to a value you haven't seen.
- What you chose lately is remembered across restarts of the window, in the state
  directory.
- The themes listed are the built-in ones, the ones in your own `themes` directory, and
  `ghostty` when Ghostty's configuration names a theme.
- **Themes are previewed.** Highlight a theme and the whole window shows it straight
  away. Enter keeps it and writes `theme = "…"` to the settings, without touching the
  rest of the file, comments included. Escape goes back to the theme in use.
- If the settings file can't be read as settings, the theme is applied but not written,
  and the top bar says why.
- Copy, paste, select-all and find aren't listed. They act on a terminal that has the
  keyboard, and the palette has taken the keyboard.

### The overview

Press `Ctrl+Shift+Space` or leader `w`, or choose "See every tab of every project at
once" in the palette.

The overview covers the whole window. It shows each project's name and, under it, each
of the project's tabs drawn small, in its layout. Tiled panes are where they are in the
tab, floating panes lie over them, and minimized panes are a row of icons underneath.
Projects stand side by side for as long as the window has room, so many small projects
fill the window as a grid. A project with more tabs than fit across has rows of its own.

- A pane shows its icon, its name, and a border in the colour of its state. Beside the
  name is the memory its programs hold, once that's a megabyte or more.
- Under that is the pane's screen, small, as it is right now. You can't read it and
  aren't meant to: text is drawn as a bar for each word, in its colour, and boxes and
  bars a program drew keep their shape. You see what kind of thing is there, an
  editor, a log scrolling, a prompt, and whether it's moving. The screens appear a
  moment after the overview does.
- A pane that's waiting, done or failed also shows what it says about that, in text
  you can read, over the foot of its picture. So you can read an agent's question
  without going there. A question too long for its tile ends in "…", never cut in the
  middle of a letter.
- The strip along the foot of the overview shows the last rows of one pane at their
  real size: the pane Enter would go to, or the one under the pointer. Move the
  pointer over a pane to read its last lines.
- A project's name is followed by the number of its panes that need you.
- The overview opens on the tab in front, and the pane you came from says "here".
  Left and Right go through every tab in order, across projects. Up and Down go to the
  nearest tab in the row above or below. All four stop at the ends, and a tab you step
  to out of sight is scrolled into view.
- In the tab the keyboard is on, one pane has a ring: that's where Enter goes. It's
  the first pane there that needs you, or else the one that had the keyboard. Press
  Space to move the ring to the tab's next pane.
- A sleeping project is marked "asleep". Press Enter on one of its tabs and the project
  wakes and you go to it. That's the same as "Wake it" on the project's page, which
  asks nothing first.
- Press Tab to show only the tabs that have a pane that needs you, and Tab again to
  show every tab. A project with no such tab is left out, and if there are none at all
  the overview says "Nothing needs you."
- Press Enter to go to the ringed pane. Click a pane to go to that pane, or a tab's
  title to go to the tab. Escape, or a click anywhere else, closes the overview and
  changes nothing.
- It doesn't resize anything. Looking at a pane here never changes its size or tells
  its program anything.

What the small screens cost:

- They're live only while the overview is open, and only for the panes in view. Scroll,
  and the ones that leave are let go. Close it, and nothing is left running.
- A pane already on screen in the tab in front is drawn from what the window has.
- They're redrawn at most four times a second, however fast the panes print.
- At most 48 panes of this computer and 12 of each other machine have a live screen at
  once. Past that, a pane shows its name and its words, as before.
- A pane that prints very fast is looked at again after a second, then after longer
  each time it still prints too fast, up to eight seconds. It isn't followed.
- On another machine, a pane that prints very fast is looked at once. Its picture
  then stays as it was until you move the pointer or the ring onto it. A look at
  such a pane takes up to 32 KB over the link, so the overview doesn't keep looking.
- If the link to a machine is lost, its panes say "link lost" over their last picture,
  and come back when the link does.
- The interface in a terminal (`isc tui`) has an overview of layouts and words, with no
  small screens: a tab there is thirty characters wide.

### Finding a file

Press `Ctrl+Shift+I` or leader `/`, or choose "Find a file under the pane's directory
and type its path" in the palette.

A small list opens at the terminal's cursor. It shows the files and directories under
the focused pane's directory, each with an icon for its kind. The daemon lists them,
so with a daemon on another machine they're that machine's files. Type a few letters
of the one you want, then press Enter to type its path into the pane.

- **The path that Enter would type is shown at the cursor, dimmed**, as you move through
  the list. The window draws it over the terminal. Nothing is sent to the program until
  you press Enter.
- The letters must appear in the path in the order you typed them, in any case. Letters
  count for more when they're in the file's own name, at the start of a word, or next to
  one another. When two paths match equally, the nearer one comes first.
- Up and Down choose among the paths listed, ten at most. Enter types the path, followed
  by a space. Escape, or a click outside the list, types nothing.
- **Tab goes into the marked directory.** The list then holds only what's in it, and
  what you type is looked for in there. Shift+Tab, or Backspace with nothing typed, comes
  back out. The path Enter types is still from the pane's directory.
- The path is relative to the pane's directory, as in `src/main.rs`. A directory's path
  ends in a slash. The path is quoted when it contains anything a shell wouldn't read as
  one word, as in `'my notes/to do.md'`.
- The path is typed as a paste, so it works in a shell, an editor's command line or an
  agent's prompt: anything that takes typing. It's checked like any other paste.
- Directories named `target`, `node_modules` or `__pycache__` are skipped, and so are
  directories whose names begin with a dot, such as `.git`. Files whose names begin with
  a dot are listed. Links to directories aren't followed.
- A file whose name holds a control character, an invisible character or one that
  changes the direction of text isn't listed: what's listed would read as another
  name than the one typed.
- At most 30 000 files and directories are listed, and at most 4 MB of their paths. In
  a directory with more, such as a home directory, only the nearest can be found.
- A directory that can't be read is passed over.
- The directory is where the pane's shell is, not where a program running in the shell
  has gone.
- The list opens under the cursor's row, or above it when there's no room underneath.
- **What git ignores isn't listed.** Inside a git repository the list leaves out
  everything git is told to ignore, by any `.gitignore`, your own excludes or the
  repository's. Git itself is asked by the daemon, once, as the list opens. Outside a
  repository, or without git, nothing is left out but the directories named above.
  - Git is told to run none of the programs a repository's settings can name, to fetch
    nothing, and to write nothing.
  - Git has three seconds to answer. After that the files are listed without it. The
    same happens if what it ignores comes to more than 16 MB of names.

---

## 15. Settings and themes

Settings live in `~/.config/insensical/config.toml`
(`$XDG_CONFIG_HOME/insensical/config.toml` if that's set; `INSENSICAL_CONFIG_DIR`
overrides both). You don't need to create anything. Without the file, or without the
directory, the defaults apply.

**The window makes the file the first time it starts**, if there isn't one. That file
explains every setting and shows its default, and sets none of them: each line that
shows a setting begins with a `#`, so nothing in it is read. That way the file is there
to find and to edit, and a default that changes in a later version still reaches you
for whatever you never chose. Take the `#` from a line to set it, or use the settings
sheet, which writes its line under the one that shows the default.

- A file that's already there is never written over, whatever is in it.
- `isc` on its own doesn't make the file; the window does, and so does `isc config set`
  with the one line it sets.
- The keymaps aren't in it. `isc config` prints them.

`isc config` prints a complete file. It has every setting at its default, with a note on
each and the values it can take. A setting with no default appears as a line to
uncomment, and so does the `[tui]` table. Both keymaps are there too, with every key:

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

- `set` changes that one line and leaves the rest of the file as you wrote it, with
  its comments and order. If the file doesn't exist, it's made with that one line.
- A setting the file shows in a comment, such as `# shell = "fish"`, is written under
  that comment. `reset` then takes only that line out and leaves the comment.
- In a shell with completions installed, `isc config set` and its neighbours complete
  the names of the settings, each with what it does.
- A value the setting can't have is refused, and the message says what it can be.
- `colours` and `mouse`, which only the interface in a terminal has, are read, set and
  reset under `[tui]`, where that interface looks for them.
- A file that can't be read as TOML isn't written to.
- These work for the settings in the table in section 19, not for the `[keys]` tables.

**Some settings are the daemon's, and the rest are the interface's.** The daemon goes
by `shell`, `term`, `scrollback`, `keep-scrollback`, `tmux-shim`, `clipboard`, `notifications`,
`notify-waiting`, `notify-failed`, `notify-done`, `sound`, `long-command`, `tray`,
`notify-command`, `notify-command-when`, `notify-url`, `notify-url-when`,
`notify-url-body`, `notify-url-headers`, `notify-url-token`, `notify-url-user` and
`notify-url-password`.
Every other setting is about how an interface looks and what its keys do. On one
machine they're all in the same file and you needn't care. With a daemon on another
machine, the daemon's settings are in the file there, and the interface's are in the
file here:

```sh
isc --host build config get scrollback      # what the file on build gives it
isc --host build config set scrollback 50   # written to the file on build
isc --host build config set theme paper     # refused: theme is this machine's
```

- The daemon there makes the change, with its own `isc`, so it must be running.
- `isc --host build config`, with nothing after it, still prints the same complete
  file as without `--host`.

Here's an example:

```toml
theme = { light = "paper", dark = "fjord" }   # or "ghostty", or "ghostty:Tokyo Night"
font-family = "JetBrainsMono Nerd Font"
font-size = 12
shell = "fish"
keymap = "leader"
```

The settings table is in section 19.

### Icons for your own programs

A program insensical doesn't know is shown by the terminal's tile. Give it one in the
settings file, under `[icons]`, by the name the program runs under:

```toml
[icons]
gitu = "git"                                    # as git is shown
myagent = "claude"
deploy = { mark = "docker", colour = "#ff8800" }   # Docker's mark on a tile of your colour
```

- The value names a program that already has a tile (`git`, `nvim`, `claude`,
  `docker`) or a mark (`agent`, `activity`, `remote`, `shell`, `terminal`).
- `colour` is the tile's own, written `#rrggbb`. Without it the tile has the colour it
  has for the program you named.
- A line here comes before what's built in, so you can also change how a known program
  is shown: `node = "vite"`.
- It applies as you save the file. A name nothing is called, or a colour that isn't one,
  is listed with the settings' other mistakes, and that line is passed over.
- The program's name is the one in the pane's header and in `isc ls`. A program that
  names itself in what it reports (section 10) is looked up by that name first, so
  `claude-code = "agent"` changes how Claude Code is shown whatever it runs as.

### The settings sheet

Press `Ctrl+,` (`⌘,` on macOS), or leader `,`, or choose "Settings" in the palette. A
sheet opens over the window with every setting on it. Nothing on it is typed as a
value: a setting is switched, picked or stepped. A number steps by one, the
background's opacity by 0.05, and `long-command`, which goes up to 3600 seconds, in
fives.

- **Sections are down the left**, on a panel set into the sheet, with the field "Search
  Settings" above them: **Server**, then Interface and Themes, then Terminal,
  Shortcuts and Remote Hosts, then Notifications and Agents, and **Changed** last. On a
  Mac, Menu bar and Dock comes after Themes. Each
  has a tile in its own colour, and Server has its state ("Running") under its name. A
  number beside a section says how many of its settings your file changes. The sheet
  opens on the page you last looked at. The page's name is at the top, beside two
  arrows that go back to the page you looked at before and forward again, and are
  dimmed where there is nowhere to go. The sheet floats over the window and Esc closes it.
- **The sheet takes most of the window**: about seven tenths of its width and eight
  tenths of its height, never less than 880 by 640 where the window has that much, and
  never more than 1080 by 820. A window smaller than that gives the sheet what it has,
  less a margin. The sections keep a little under a third of the sheet's width.
- **A page is a few cards, each a few settings that belong together**, under a short
  heading. No card has more than four rows. A row is the setting's name, one quiet line
  under it that says what it does, and one control at the right. The quiet line is always
  said whole: where it doesn't fit on one line it goes to a second, and the row grows for
  it.
- **The row the keys are on** has a ring in the accent colour while you're using the
  keyboard, and loses it as soon as you move the pointer. A row, a button or a switch that
  can be pressed is a step lighter under the pointer and a step darker while it's pressed.
- **A change takes effect at once** and is written to `config.toml`, to that
  setting's one line. The window behind the sheet stays in view, so you see a theme,
  a font or a density change as you make it.
- **A setting your file changes** has a small dot in the accent colour after its name.
  Put the pointer on its row, or the keys, and a small arrow that turns back appears
  left of its control: press it, or `Ctrl+Backspace`, to reset the setting, which takes
  its line out of the file. **Changed** lists all of them, under the name of the page
  each is on.
- **Type to find a setting** in any section, by its name or what it does. What's found
  is listed under the name of the page it's on.
- **Showing** is at the top right of Server, Terminal, Notifications, Agents and Changed
  when you have named another computer. It says which computer the page is about, and
  Alt+M steps through them. A computer that can't be reached can be chosen: the page says
  so and has nothing to change until it answers.
- **Server** is the first page. A card gives this computer's server: its state, a `Restart
  Server` button that asks first what would end, its version, the path of its socket
  and the path of the settings file, each with a button that copies it. Click the settings
  file's row to open the file. Under **Connected** is everything that is looking at that
  computer's terminals: **This window** (first, as one row, though it holds a connection
  for each pane it shows), **A terminal (isc)**, **A browser or phone** with its name,
  **Another computer, over ssh**, or **Another program** with its name after a dot when it gives one. When the browser server (`isc web`) is running, the connections it keeps to notify your devices are one row, **Phone and browser access**, with no `Disconnect`: the switch **Open in a browser or on your phone** turns it off. Each says what it can do:
  **Can only look**, **Can look and type**, **Can do everything**, or what it can do in
  words, and how long it has been there. It's asked again every two seconds. Each row but
  this window's and the phone and browser access one has `Disconnect`, which asks once and then ends that connection. It can
  connect again. For a browser or phone with a name, the question also offers
  `Disconnect and Remove Device`, which ends its pairing too once the connection has
  ended; if the pairing can't be ended, this card says so. With another computer chosen,
  the card is that computer's server and can't be restarted from here, and the question
  offers only to disconnect, because the devices paired are on that computer.
- **Access from your other devices** is a card on Server, for opening insensical in a
  browser or on a phone without a terminal. It's for this computer only: with another
  computer chosen the card says that it's set on that computer itself and offers nothing.
  - **Open in a browser or on your phone** is a switch. On starts `isc web` for this
    computer's server and the line under it says where: `On, on this computer only:
    http://127.0.0.1:7433`. Off stops it, as `isc web stop` does. Where the systemd user
    service `insensical-web.service` is installed, the switch enables and starts it, and
    disables and stops it, so it's there at your next login. Where it isn't (a build you
    made yourself, a Mac), the switch starts `isc web` on its own and remembers that in
    the file `web-on` in your settings directory, and the application starts it again
    when it next starts and finds it not running. It never starts a second one: it asks
    `isc web status` first, and if another window got there first it takes that as
    started. If it can't start, or never says it's running, the line says why. Every
    program it runs has a few seconds to answer and is ended if it doesn't. What the
    switch is doing carries on if you close the sheet, and the line says so when you open
    it again; a second switch waits for it. `isc web stop` on the command line also
    forgets `web-on`, so a window you open afterwards doesn't start it again.
  - **This computer's browser** appears while the switch is on, with a row, **Link**, under
    it. `Open in Browser` runs `isc web open`, which makes an address that works once, for
    10 minutes, and opens it in the browser your computer opens links with. That browser
    is paired as the device `This computer` and stays signed in. `Link` shows the address
    without its secret part, and its copy button puts a new address on the clipboard, which
    the row then says works once, for 10 minutes. Nothing is restarted. Press either again
    for another address: it pairs a browser as the same device, so removing `This computer`
    under Your devices signs out every browser that was paired this way. It's listed under
    Connected like any device.
  - **Reach it from anywhere on your tailnet** appears while the switch is on, and runs
    `tailscale status --json` to see how Tailscale stands. It finds `tailscale` on your
    `PATH` and, on a Mac, where Homebrew and the Tailscale application put it, since a
    window opened from the Dock has only the system's `PATH`. Without Tailscale the row
    says so, and `How to Install` opens this manual. Signed out, `Sign In` runs `tailscale up` and opens
    the address it prints; if Tailscale won't sign in from here, the row says to open
    Tailscale. Signed in, the row shows this computer's tailnet name (the first of its
    certificate names, else its DNS name) and a switch. On
    does what "Reach it from another device" under [In a browser](#in-a-browser) has you type:
    it restarts `isc web` answering to that name (with the service installed, through a
    drop-in file `names.conf` for it in `~/.config/systemd/user/insensical-web.service.d`,
    which keeps the service's own command and adds the names to it; if that command
    can't be read the row says so and nothing is changed) and runs `tailscale serve --bg
    7433`; the address in the first row is then `https://` and the tailnet name. A mapping
    that is the one `/` on port 443 to this server's port, and nothing else, is taken as
    this switch's own, whoever made it and under whichever of the computer's names it is
    filed. Under the current name the switch reads as on, and off takes it away. Under a
    name the computer no longer has, the switch is off and the row says so; on takes the old
    mapping away, restarts `isc web` with the current name and makes the new mapping, and if
    a step fails it puts `isc web` back as it was and says what failed. If Tailscale serves
    anything else on 443, or can't say what it serves, it stops and says what is there,
    and touches nothing. If your tailnet hasn't turned Serve
    on, Tailscale waits for you: the row says so, `Turn On Serve` opens the page Tailscale
    gave, and the name is taken off `isc web` again. Off runs `tailscale serve
    --https=443 off` and restarts `isc web` without the name. What a switch says about
    its own row is said under that row, whole, in the failure's colour only when it failed.
    If `isc web` is found answering to a name the computer no longer has while this switch
    is on, it's restarted with the current one.
  - **Notifications on your devices** says how many devices asked for notifications, and
    `Send a Test` sends each of them one and says in a sentence per device how it went.
- **Your devices** lists each device paired, with what it may do, when it was last
  used and, for one that stays paired only for a while, when it ends ("ends in 40m", in the accent colour under ten minutes), and a `Remove` that asks once and ends its pairing. A guest, who sees one project or one terminal, says so in place of that: "Guest · only project api · can look · ends in 23h", or "nothing left to see" once what it was shown is closed, and has `End`, which signs it out at one press without asking. Under **Connected**, a guest's browser reads the same way and has `End` too. `Add a Device...`, in a card of its own under them, asks for
  a name (not one a device has already), what it may do (Everything, Look and type, or Only look), how long it
  stays paired (Until I remove it, 1 hour, 1 day or 7 days: after that the server signs it out and removes it) and, under More
  options, how many devices the code pairs and for how long, and `Add` shows a QR code to
  scan with the device's camera, and the link with a button that copies it. A line under the fields says that a device sees every terminal on this computer
  and that Share, on a project or a terminal, shows someone just that one (see "Share one project or one terminal" under [In a browser](#in-a-browser));
  for a device that may do Everything it says instead that it can run programs as you. The code
  works once for ten minutes unless you changed that, and the line under it says so and, unless it stays until you remove it, how long the device stays. It's made as `isc web token add`
  makes it, and it's kept only while the dialog is open: close it and the link is gone,
  and a lost one is replaced by removing the device and adding it again.
- **Interface** has four cards: **Panes** (density, space between panes, pane header),
  **Tabs and sidebar** (where the tabs are, what the sidebar lists), **Text** (terminal
  font, text size, ligatures, interface font) and **Motion**.
- **Themes** has two: **Colours** (the theme, the interface colours) and **Behind the
  window** (the background's opacity and blur).
- **Terminal** has four: **Shell** (the shell, the terminal type), **Typing** (input
  method, Option as Alt, type ahead), **History** (how much each pane keeps, and whether
  it's kept when the server stops) and **Files and clipboard** (what opens a file, and
  whether programs may write the clipboard).
- **Remote Hosts** lists the computers you've named, each as three cards. The first has
  its state (Connected, Can't be reached, or that its server can't be used) and where ssh
  goes for it, with a `Check` button that meets it and says what's wrong in plain words.
  The second has its colour and icon (press for the next) and whether it's careful (it
  asks before a paste of more than a line). The third has `Rename`, the address
  (`Change`, then type it in the row, Enter keeps it, Escape leaves it), and
  `Forget`, which asks once; nothing on that computer changes.
  When a check finds that insensical there is missing or older than here, or that its
  server runs an older insensical than the one installed there, a row offers `Update
  There` or `Restart Its Server`, and each says what it will do before it does it.
  What a row says of a problem or an offer is said whole, on as many lines as it takes.
  A dialog's sentence is never cut: its head grows to the lines it needs.
  A machine that can't be reached says why on its own row, and where `ssh` has a
  question the row has an `Answer` button.
  `Add Remote Host` opens a dialog: the machines of your tailnet and the names in your
  ssh configuration are listed under two tabs, anything you type is given to ssh, Enter
  meets it, and the four lines that tick say how far it got and what to do. The dialog lies over the sheet, which stays
  behind it, dimmed; Esc closes it and you're back on Remote Hosts.
- **Notifications** holds the settings about what tells you, in three cards: **Being
  told** (notifications at all, and the tray's icon), **What you're told of** (a pane
  that waits, something that fails, something that finishes, and how long a command is a
  long one) and **Light and sound** (the glow around a waiting pane, the sound, and a
  `Play` for each of the two sounds). Under **When you're
  away** is how you're told on your phone when something needs you and no window or terminal
  is open. Choose **ntfy** and give the server (the free `https://ntfy.sh` unless you
  change it), a topic, and an access token if your server needs one; or choose **Another
  web address** and give the address. **Only when I'm away** is a switch: off, you're told
  every time. `Send a Test` says in a sentence how it went, such as `Sent. Check your
  phone.`, `The server said no: the token is wrong or missing (403).`, `Couldn't reach
  ntfy.example.com.` or `Not sent: this address isn't secure (http). Use https.`, and a
  quiet line says how the last alert went. **More options**, closed until you press it,
  holds how an address is sent (JSON or plain text), a user and password, a token for an
  address, and a command to run. A token, user or password is typed into a field that
  shows dots, and afterwards the field shows only dots. The small arrow that appears
  beside the field while the pointer or the keys are on its row, or `Ctrl+Backspace`,
  takes it out. `Ctrl+V` pastes. The settings behind it are `notify-url`, `notify-url-token`,
  `notify-url-body`, `notify-url-when`, `notify-url-user`, `notify-url-password`,
  `notify-command` and `notify-command-when`; the ntfy fields are the address cut at its
  last `/`, and giving a topic also sets `notify-url-body` to `text`, which is what ntfy
  takes. `notify-url-headers` is set only with `isc config set`. Finding "ntfy", "phone"
  or "alert" leads here, and the settings it changes are listed under Changed. The arrow
  keys go through the settings and then the ways of being told as one list.
- A file edited by hand while the sheet is open shows on the sheet as it's saved.
- Mistakes in the file are listed at the top of the sheet. The settings that could be
  read still work.
- The settings file's path is on the Server page. The sheet has no foot: Esc closes it.
- **The list for the terminal's font has only fonts of one width**, since a terminal is
  set in cells. Any other font can still be named in the file.
- A page with more cards than fit scrolls under its name, which stays where it is. It
  fades at its foot while there is more below, and has a hairline under its name while
  there is more above. The row the keys are on is kept in view.
- What is said under the pointer never runs off the window: a long one wraps.

| Control | For | Change it |
|---|---|---|
| A switch | on or off | Click it, or Enter |
| A word with `⌃⌄` | one of a few choices | Click it, or Enter, for the next |
| Buttons side by side | any of a few, such as what the sidebar lists | Click one to tick (in the accent colour) or untick it (grey), or Enter to go through every combination |
| A number with `−` `+` beside it | text size, space between panes | Click `−` or `+`, or `Ctrl+Right` and `Ctrl+Left` |
| A value with `›` | a theme, a font, a shell | Click, or Enter, for a list in place of the rows. Type to narrow it. Enter chooses, Esc goes back. |
| Key caps | the leader | Click, or Enter, then press the key. It needs Ctrl, Alt or Cmd held (`⌃`, `⌥` or `⌘` on macOS). Esc cancels. |

- A theme is shown on the whole window as you move through the list, before you choose
  it. Esc puts back the one you had.
- The font lists hold every font the system has. The shell list holds the shells in
  `/etc/shells`.
- **Agents** isn't settings of insensical's. It lists each agent it knows, and whether
  the agent reports what it's doing: it does, it doesn't, its hooks have gone
  stale (they name an `isc` that's no longer there), or the agent isn't on this
  system. `Set up` asks before adding hooks, as in [section 11](#11-agents).
- **Shortcuts** has the keymap and the leader under **Keys**. The keys themselves are
  tables in the file: under **Every key** are buttons for the sheet of every key and for
  opening the file.
- With more than one machine, the pages that hold a server's settings say whose they are.
  See [Other machines in the window](#other-machines-in-the-window).

| Key | Does |
|---|---|
| `Up`, `Down` | Move between rows |
| `Enter` | Change the marked setting, or open its list |
| `Ctrl+Right`, `Ctrl+Left` | Next or previous value; one more or one less |
| `Ctrl+Backspace` | Reset the marked setting |
| `Tab`, `Shift+Tab` | Next or previous section |
| `Esc` | Close the sheet, or leave a list |

A theme for each appearance of the desktop, `{ light = …, dark = … }`, shows as "By
the desktop's appearance". Picking a theme from the list replaces the pair with that
one theme; to have a pair again, write it in the file. The `[keys]` tables and a
theme's own file aren't on the sheet.

### When a change applies

- **Saved changes apply within a second.** You don't need to restart or press a key. The
  window checks the file once a second.
- The daemon reads `shell` and `keep-scrollback` when the next pane starts. A pane
  that's already open keeps the values it was opened with.
- The window checks that it can find the program `shell` names, either by its path or
  on the window's own `PATH`. If it can't, it says so in the top bar.
- The daemon reads `clipboard` each time a program asks, so a change applies right away.
- Changing `sidebar`, `pane-header`, `density` or `pane-gap` changes how much room
  terminals have, so they're resized.

### When the file is wrong

- **One unreadable line voids the whole file.** A misspelt key, a value of the wrong
  type or broken TOML shows in the top bar with its line number, and *every* setting
  falls back to its default until you fix the line. An unknown key is an error. It
  isn't ignored.
- A command under `[commands]` that's written wrongly is reported, and only that
  command is left out. It never voids the file. The same goes for a mode under
  `[modes]`.
- A value that can be read but not used is reported, and only that setting falls back.
  That covers a font size of 500, a theme that doesn't exist, a leader that isn't a key,
  and a shell that isn't a program.
- The top bar shows only the first problem. All of them are printed to the window's
  standard error.
- **The daemon is more forgiving than the window.** It reads its own settings (listed
  at the start of this section) one at a time. A value it
  can't read, such as `notify-url-when = "allways"`, leaves that one setting at its
  default, and the rest stand. If the file is broken TOML, the daemon keeps the
  settings it last read, so saving a half-edited file doesn't reset anything that's
  running. You're told what it couldn't use: in a notification when the file
  changes, at the end of `isc notify --sinks`, and through `isc --host NAME config
  get`, which shows such a setting at its default.

### Themes

A theme is a terminal's colours: foreground, background, the sixteen named colours, and
optionally cursor and selection. The UI's colours are derived from them, so any terminal
theme styles the whole application. UI text is always shifted in lightness until it's
readable.

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

A theme that has both follows the desktop: `theme = "lantern"` is dark when the desktop
is dark and light when it's light, and changes as the desktop does. It can't be held to
one side on a desktop of the other.

A theme name is looked up in this order:

| Name | Means |
|---|---|
| `ghostty` | The theme named in Ghostty's own configuration for the desktop's current appearance |
| `ghostty:Name` | The Ghostty theme with that name |
| anything else | `themes/NAME.toml` beside the settings file, then the themes that come with insensical |

- If `theme` isn't set, you get `fjord` when the desktop is dark and `paper` when it's
  light.
- A pair, `{ light = "…", dark = "…" }`, follows the desktop's appearance as it
  changes.
- Ghostty's themes are read from where Ghostty keeps them: `~/.config/ghostty/themes`,
  `/usr/share/ghostty/themes`, `/usr/local/share/ghostty/themes`, and inside
  `Ghostty.app` on macOS. Nothing is copied.
- **Without Ghostty installed, `ghostty` and `ghostty:…` are errors.**
- When you save a theme file that's in use, it applies within a second.
- **Only the theme comes from Ghostty.** Its font, font size, font style, padding,
  opacity, blur, cursor style and `command` aren't used.
- The UI's colours always come from the terminal theme, except a program's tile, which
  is in the program's own colours.

### A neutral interface

By default the window around the terminals is worked out from the theme, so a warm theme
gives a warm sidebar. `interface-style = "neutral"` draws it in greys instead, as light
or dark as the theme, and leaves the colour to the terminals.

- The sidebar, tabs, headers, borders, sheets and their text become grey.
- The theme's accent and the colours of the states stay, and so do the terminals.
- Surfaces a theme file chose under `[interface]` aren't used; its `accent`, `running`,
  `waiting`, `done` and `failed` are.

### A see-through window

`background-opacity` says how much of what's behind the window its background hides,
from `0.2` to `1`. At `1`, the default, the window is solid. Below that your desktop
shows through behind the text:

```toml
background-opacity = 0.85
background-blur = true
```

- Only the background thins. Text, the cells a program gave a colour of its own, tiles,
  borders, sheets and questions are as solid as ever.
- The whole window thins alike: the sidebar and the panes are one sheet.
- A floating pane and a zoomed pane stay solid, since they lie over other panes and
  would otherwise show those through.
- `background-blur = true` asks the desktop to blur what shows through. Whether it does
  is up to the desktop. On Hyprland and other compositors that blur by their own rules,
  it has no effect: turn blur on for the window there (in Hyprland, `decoration:blur`
  applies to any window that isn't opaque).
- On a desktop with no compositor there's nothing to show through, and the window is
  drawn over black.
- Both apply as you save, without reopening the window. In the settings sheet the
  opacity steps by 0.05.
- Text over a busy picture is harder to read. Nothing corrects for that.

### A theme file

`~/.config/insensical/themes/<name>.toml`:

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

Whether a theme is light or dark is worked out from its background, unless
`appearance = "light"` or `"dark"` at the top of the file sets it explicitly.

One file can hold a theme for both. Write each side under `dark` and `light` instead,
and the desktop's appearance picks between them:

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

- A file has either `[terminal]`, or both `[dark.terminal]` and `[light.terminal]`.
  One side alone is an error, and so is mixing the two forms.

- A theme only needs `[terminal]` with `foreground`, `background` and `palette`. You
  can leave out `name`, `appearance`, `cursor`, `selection` and all of `[interface]`. A
  theme with no `name` is called "unnamed".
- A colour is six hexadecimal digits, with or without the `#`. Shorter forms such as
  `#fff` are errors, and so are colour names.
- `palette` must have exactly sixteen colours.
- An unknown key is an error, as in the settings file. If a theme file can't be read,
  the top bar reports it by name and the default theme is used.

You can set any UI colour yourself instead of having it derived:

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

How the rest is derived:

- The canvas is the terminal background, a step darker. Raised, selected and border
  step from the canvas toward the foreground. Steps are taken in a colour space where
  equal steps look equal and the tint is kept.
- Muted and faint text sit between the foreground and the canvas.
- The accent is the palette's blue. The state colours are its blue (its cyan, if the accent
  took the blue), yellow, green and red.
- UI text that wouldn't stand out enough from the canvas is shifted in lightness until
  it does. This applies to colours you set too, so no theme can produce a UI you can't
  read.

---

## 16. The command line, `isc`

`isc` is the daemon and its command line in one binary. It does everything the window
does, and it lets scripts wait for a pane or follow events. The full list of commands
is in section 19.

### Naming things

- Refer to a pane by its number, as `isc ls` prints it: `3` or `pane:3`. A tab works
  the same way: `7` or `tab:7`. Refer to a project by its name or its number.
- If you don't give a layout command a pane, it acts on the pane it's run in
  (`$INSENSICAL_PANE`). Outside any pane, it acts on the pane with the keyboard.
- `isc status` and `isc notify` never guess. Outside a pane they need `--pane`.
- If two projects share a name, the name means the first one listed. A number is always
  a project's number, never a name: see [Projects](#projects).
- What `isc` prints to read comes from names, titles and directories, and a directory
  can be called anything. A character your terminal would obey or hide is printed the
  way `\u{1b}` is here, so a name can't clear your screen, write to your clipboard or
  pass for another line. `--json` output, `isc api` and `isc events` keep every name
  exact: such a character is written there as JSON's own escape, `\u001b`, which
  whatever reads the JSON reads as the character.
- Errors go to standard error as `isc: …`, with exit status 1.
- Every command accepts `--socket PATH`. It names the daemon's socket, in place of
  `$INSENSICAL_SOCKET` and the default.
- `isc --help` and `isc COMMAND --help` describe the commands. `isc --version` prints
  the version of `isc` only.

### Which commands start the daemon

**Nothing is written to a log file.** What insensical has to say, it says in the
window, or on standard error:

- `isc` prints its errors where you ran it.
- A daemon started on demand, by a window or a command, has nowhere to print: its
  standard error is discarded. It says little: a pane that failed and was closed, or an
  application it couldn't open. To read it, stop the daemon and run `isc daemon` in a
  terminal yourself.
- A daemon started by the user unit prints to the journal:
  `journalctl --user -u insensical`.
- The window prints to whatever started it. From a terminal, run `insensical` there and
  read it. Mistakes in the settings file are among what it prints, and the settings
  sheet lists the same ones.

A command about panes, tabs or projects starts the daemon if it isn't running. So does
`isc status explain`, `isc notify --test`, `isc notify --sinks`, and `isc restart-server`
when no daemon is running. These never start it: `daemon`, `kill-server`, `version`,
`status` with a state, `notify` with a title, `integrate`, `shell-integration`, `config`, `completions`, `host`, and the agents'
hooks. With `--host`, the same commands start or don't start the daemon on the other
machine.

### Listing

`isc ls` prints projects, tabs and panes. `isc list` is the same command. It marks the
current project, tab and focused pane with `*`. A project's line shows its number, name
and root, and `asleep` while it sleeps. A tab's line shows its number, its name if it
has one, and `zoomed` while a pane fills it. A pane's line shows its number, size in
columns and rows, placement (`tiled`, `floating` or `minimized`), state, program, and
name or title, with two spaces between each. The state is always one word and the
program is the process's name, so a script can count columns up to there.

After the title, each with two spaces before it, come the things that are only
sometimes there: what a waiting pane waits for and how long a pane has been
`waiting`, `failed` or `done`, as in `wants permission, 2h`; the name a program
gives itself in its reports, in brackets; and how many things it reports. Those are
for you to read. A script that wants them reads `isc ls --json`. If an update has been
installed under the running daemon, `isc ls` says so on standard error.

```text
* project:5  api  /home/you/code/api
    tab:4  zoomed
      * pane:3  120x40  tiled  waiting  node  Refactor the auth middleware  wants permission, 2h  (claude-code)
        pane:6  120x40  tiled  idle  nvim  src/session.rs
  * tab:8  servers
      * pane:10  80x24  floating  idle  fish  
        pane:7  96x30  minimized  waiting  cargo  
  project:9  web  /home/you/code/web  asleep
      tab:12
        pane:11  80x24  tiled  idle  fish  
```

`isc ls --json` prints everything the daemon knows. That's the stable format for
scripts. `isc usage`, `isc clients` and `isc api` have `--json` too, and
`isc integrate --status` always prints JSON.

### Running things

```sh
isc run [--cwd DIR] [--keep] [WHERE] [-- COMMAND…]
```

It runs a command in a new pane, or the shell if you don't give one, and prints
`pane:N`. The working directory defaults to the one you run `isc` in. The places are
mutually exclusive: you can give only one of `--project`, `--new-project`, `--split`
and `--float`. Use `--beside` to name the pane to split when it isn't the one `isc`
runs in or the focused one.

| `WHERE` | Place |
|---|---|
| nothing | A new tab in the current project |
| `--project NAME` | A new tab in that project |
| `--new-project [--name NAME]` | A new project for the working directory |
| `--split right\|down [--beside PANE]` | Beside a pane, sharing its space |
| `--float [--rect X,Y,W,H]` | Floating above the current tab |

With `--keep`, the pane stays open when the command ends, so you can read what it
printed. Press Enter in it, or run `isc close`, to close it.

**`isc do NAME` runs a command by its name**: one of your own from `[commands]` in the
settings file on this machine (see [Changing keys](#changing-keys)), or one of the
commands in section 19.

```sh
isc do tests            # your own
isc do split-right      # one that exists already
isc --host build do tests
```

- The steps run in order on the daemon the command is for. If one fails, `isc do`
  stops there, says which step it was, and exits with an error.
- A command that only an interface can do, such as `palette` or `copy`, is refused,
  and so is a list at the step that holds one.
- A step that would end something that's running, such as `close-tab`, is refused
  unless you say `--yes`.

### Typing into a pane and reading it

- `isc send PANE TEXT` types text. Text may begin with a hyphen. If it's one of
  `isc`'s own options (`--host`, `--rights`, `--socket`), put `--` first:
  `isc send 3 -- --host`. The same goes for a title given to `isc notify`, which also
  has `--test`, `--sinks` and `--pane`.
- `isc key PANE KEY…` presses keys by name: `ctrl-c`, `enter`, `escape`, `shift-tab`,
  `space`, `a`. They're encoded the way the program in the pane expects. Write a key
  as you would in `[keys]`, with `ctrl`, `shift`, `alt`, `cmd` or `super` held. Several
  keys are pressed in order: `isc key 3 escape : w q enter`.
- `isc capture PANE [--vt]` prints the screen and scrollback as text, and nothing your
  terminal would act on: a control character a program left there is printed the way
  `\u{1b}` is here. With `--vt`, colours and styles are included as escape sequences,
  and those are the only escape sequences in it. The pane's title, links, palette,
  modes and cursor position are never printed.

### Waiting

```sh
isc wait PANE [--until STATE]… [--timeout SECONDS]
```

- It succeeds when the pane reaches one of the states you name with `--until` (`idle`,
  `running`, `waiting`, `done`, `failed`, `closed`), and prints which one.
- Without `--until`, it waits for `done`, `failed`, `waiting` or `closed`: the states
  that end the pane working on its own.
- It fails if the pane closes before reaching a state you asked for, or if there's no
  such pane.
- It exits with status 124 if the time runs out. The time can be a fraction of a
  second. A time that isn't a length of time, such as `inf`, is refused.

```sh
pane=$(isc run -- claude -p "fix the failing test")
isc wait "${pane#pane:}" --timeout 1800 && isc capture "${pane#pane:}" | tail -20
```

**Waiting for what a pane prints.** `isc wait PANE --output TEXT [--timeout SECONDS]`
succeeds when the pane's screen or its history shows `TEXT`:

```sh
pane=$(isc run -- npm run dev)
isc wait "${pane#pane:}" --output "ready in" --timeout 60 && xdg-open http://localhost:5173
```

- It looks at the screen as it is and at the history the pane keeps, so it succeeds at
  once if the text was printed before you asked, even if it has scrolled off since.
- Text the pane's width broke across two lines isn't found. Wait for a shorter piece
  of it.
- The text is matched as written, with no patterns. It's found whatever colours it's
  printed in.
- It can't be combined with `--until`. It fails if the pane closes first, and exits
  with 124 if the time runs out.

### Any request, as JSON

Everything the window does is a request to the daemon, and `isc api` sends one:

```sh
isc api '"State"'                                  # the whole state, as `isc ls --json` gives it
isc api '{"Mute":{"pane":3,"on":true}}'
isc api '{"Float":{"pane":3,"rect":{"x":0.5,"y":0.1,"w":0.45,"h":0.8}}}'
echo '{"Zoom":{"tab":2,"on":true}}' | isc api
```

- The request is one JSON value: a name alone for a request that takes nothing, or an
  object with the name as its key. The answer is printed as JSON: `"Done"`,
  `{"Pane":7}`, `{"State":{…}}`, `{"Text":"…"}`.
- What the window asks of the daemon's machine is here too: `"Clients"`,
  `{"List":{"dir":"~/src","deep":false}}` for a directory's names, `"Hooks"` and
  `"Settings"`.
- The requests can change between versions. The commands are the steadier way to
  script; `isc api` is for what they don't cover.

**`isc api` with no request lists every request**, one to a line: its name, the right
it needs, and what it does. It asks the daemon nothing, and starts none.

```sh
isc api                 # every request
isc api Float           # what one holds
isc api --json          # the same list, as JSON
isc api Float --json
```

**`isc api NAME` describes one request and sends nothing.** `NAME` is the request's
name alone, without quotes. Capitals don't matter, and `hide-floats` is `HideFloats`.

```
Float: Lifts a pane out of the layout to float, or moves one that floats. The rectangle is in fractions of the tab.
Needs the right to arrange.

  pane  a whole number
  rect  {x: a number, y: a number, w: a number, h: a number}

Example:  isc api '{"Float":{"pane":3,"rect":{"x":0.5,"y":0.1,"w":0.45,"h":0.8}}}'
Answers:  "Done"
```

- Each part is listed with what it may be: `a whole number`, `a number`, `text`,
  `true or false`, a list, an object with its own parts, or `one of` a few words.
  `or null` means it may be `null`, and `may be left out` means you can omit it.
- The example is a request you can send as it's printed, with your own numbers in it.
- **What a request holds is worked out from the requests themselves, by this version
  of `isc`**, so it's never out of date. Describe requests with the `isc` that will
  send them.
- For a part that is one of several forms, such as where a new pane goes, the forms are
  named and the one in the example is described. The others aren't.
- With `--json` you get the same as data, for a script or an agent: `name`, `needs`,
  `says`, `fields` (each with `name`, `is`, `optional` and `defaulted`), `example` and
  `answers`.
- `isc api '"?"'` still fails and lists the names.
- Piped an empty standard input, `isc api` lists the requests too.

### Events

`isc events` prints what happens, one JSON object per line. Each has an `event` and
usually a `pane`.

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

- `did` is printed after a client changed something: opened, closed, moved, named,
  muted, put to sleep, changed a setting of the daemon's, or stopped it. It says who:

  ```json
  {"event":"did","did":"mute","revision":41,"by":{"client":12,"kind":"command-line","name":"desk","pane":null}}
  ```

  `by.pane` is the pane the command was run in, when the daemon could check that, and
  null otherwise. Looking and typing aren't reported, and neither is a request that
  was refused.

- If you stop reading, as when the output is piped to a pager, the daemon keeps a
  limited amount for you, 8 MB. Past that it closes the connection and `isc events`
  ends; run it again. A window that was stopped for long enough to leave that much
  unread loses its connection the same way, and finds the daemon again by itself
  when it runs on.
- A program that sends the daemon requests and never reads the answers loses its
  connection too, once more than 8 MB of answers wait behind the first.
- A program that rings its bell, or changes its title or progress, many times in a
  moment is reported at most twenty times a second, always with what it said last.
  What a program reports of itself and its tasks (section 10) is passed on at most ten
  times a second.

### Who is connected

`isc clients` lists everything connected to the daemon, one to a line:

```
client:12  window  on desk  0.3.0  look,type,arrange,manage  pid 4121  looking at pane:3  for 512s
client:19  program  on desk  0.3.0  look,type,arrange,manage  pid 9120  in pane:7  for 0s
```

- Each line has the client's number, what kind of program it is (`window`, `terminal`
  for the interface in a terminal, `command-line`, `program` for a command run inside
  a pane, `web` or `other`), the machine it runs on, its version, its rights, the
  process at the other end of the socket, the pane it's looking at, and how long it's
  been connected.
- **The kind, the machine and the version are what the client said**, and the daemon
  doesn't check them. The process is what the system says.
- A client that came through `ssh` is marked ``through `isc proxy` (claimed, not
  verified)``, and its process is the proxy's on the daemon's machine.
- `in pane:7` means the command was started in that pane, and the daemon checked it:
  the process is in that pane's own session. A process that only has the pane's number
  in its environment, or left the session, is marked `(claimed, not verified)`.
- A guest's browser, paired with `isc web token add --only`, says what it's kept to:
  `only project 4` or `only pane:12`. `isc clients --json` prints all of it as JSON,
  with each client's `scope`: `{"project": 4}`, `{"pane": 12}`, or `null` for a client
  that sees everything.
- `isc disconnect 12` (or `client:12`) puts that client off: its connection is cut, and
  it's told that you ended it. A window, the interface in a terminal and a page in a
  browser say so and stay disconnected until you connect them again there: Enter in
  the window, any key in a terminal, loading the page again in a browser. A guest's
  page isn't told why and tries again, so end a guest's pairing to keep it out. Each
  pane a client shows has a connection of its own in the list; the one that ends the
  client is its first. It needs the right to `manage`. You can't put off the connection you ask
  on, and a number that's gone is said to be. A browser paired through `isc web` shows
  as `web`, one line for each connection it has; putting those off doesn't unpair it,
  and it connects again. To keep it out, end its token with `isc web token rm`. With
  `--host NAME` it puts a client off that machine's server.
- A window makes a connection for each pane it shows, so it has several lines.

**Rights.** A connection has up to four rights:

| Right | Lets it |
|---|---|
| `look` | See projects, tabs and panes, each pane's screen and history, what programs use, who is connected, a directory's names, the daemon's settings (with `notify-url` and `notify-command` cut short), how the last notification sent outward went, and agents' hooks |
| `type` | Type and paste into a pane, set the size it's shown at, report a state or a notification for it, and count as you: what it looks at is seen |
| `arrange` | Open a pane with any command, close, split, move, float, focus, name, mute, put to sleep and wake, and choose icons |
| `manage` | Stop the daemon, change its settings, add or remove agents' hooks, and send a test notification outward |

- **`type`, `arrange` and `manage` each let a client run programs as you.** `type`
  types into your shell. `arrange` opens a pane with any command it likes, so it
  needs no `type` to run one. `manage` changes what the daemon runs: your shell, the
  command it tells you with, agents' hooks. Give any of them only to what you'd give
  your account.
- **`look` reads everything**: every screen and all its history, with any password,
  key or token a program printed or echoed there. It runs nothing and changes nothing.
- You get all four without asking, so nothing changes unless you ask for fewer.
- `--rights`, before any command, asks for fewer:

  ```sh
  isc --rights look ls
  isc --rights look,type attach 3
  ```

- A command that needs a right the connection wasn't given is refused, and says which:
  "this connection was not given the right to arrange, which `close` needs. Its
  rights are: look". Typing without the right to type is dropped, with nothing said.
- Rights are set when a connection is made, and can't be added to after.
- **Going to a pane needs `arrange`**, because the focus is shared by everyone
  connected. Without it you can't change which pane, tab or project is in front.
- **Only a client that can type counts as you.** A pane it's looking at is taken as
  seen and sends no notification, what a program there copies is sent to it, and
  while it's attached you aren't away. A client with only `look` counts as nobody.
  The daemon can't tell your own second screen from one on a wall, so you're told
  anyway: the pane stays `done` until you look from somewhere you can type, and your
  phone still rings.
- **This isn't a wall against your own programs.** Any program running as you can make
  a connection of its own with every right. `--rights` is for a command, a script or a
  screen you want to be sure does no more than you meant.
- With only `look` you still see everything typed that a program echoes, every screen
  and all its history, and the names in any directory you can list on that machine.
- **What a client with only `look` can still affect:** it has a line in `isc clients`
  under the name it gave; and it makes the daemon work and hold memory for it, since
  every screen it opens is put together for it. However many such clients there
  are, the daemon holds 256 MiB of history kept back and 256 MiB of screens waiting
  to be read for all of them together, and a little more for each. It changes nothing
  else: no pane, no layout, no state, no setting, and nothing about how you're told.

### Attaching from a terminal

`isc attach [PANE]` shows a pane in the terminal you run it in, and sends what you type
to it. Press `Ctrl+]` to leave. It says so in one line above the pane when it starts,
in the terminal's history, and for as long as you're attached it says so in the
terminal's title too: the pane's name, then "Ctrl+] to leave". The title takes no row
from the pane. When you leave, the terminal's own title is put back, if the terminal
allows it.

If you press `Ctrl+C` three times within two seconds, which is what someone who can't
find the way out does, a reversed line over the pane's last row says "Attached to a
pane: Ctrl+C goes to its program. Ctrl+] leaves." After three seconds the row is
drawn again from the pane. The three `Ctrl+C` still reach the program.

When the program in the pane ends, `isc attach` ends too, once its last output is shown,
and the shell's prompt is back.

- The terminal gets the pane's history and then the program's own output. Nothing is
  redrawn or re-encoded on the way, so the terminal's own scrollback, selection and
  search work on it.
- That means every escape sequence the program prints reaches your terminal as it
  would if you had run the program there: titles, links, clipboard requests and all.
  `isc attach` is a terminal for the pane, and it filters nothing. Use `isc capture`
  to read what a pane you don't trust has printed.
- Whatever was on the terminal's screen before moves up into its history. When you
  leave, the terminal is restored: mouse reporting off, cursor shown, and so on.
  Leaving doesn't affect the pane or its program. `Ctrl+]` leaves even when a program
  in the pane has put your terminal into a keyboard mode that sends it as a longer
  sequence.
- **The pane takes that terminal's size**, for everyone looking at it, and follows it
  when the terminal is resized, until you go back to a window that shows the pane. See
  [Two windows](#two-windows).
- You reach a pane on another machine the same way, from a shell there or from here:

  ```sh
  ssh build-host isc attach 3
  isc --host build attach 3
  ```

  The second runs `isc` on this machine and needs the machine named first: see
  [Other machines, over ssh](#other-machines-over-ssh).

### Other machines, over ssh

`isc` can use the daemon on another machine as it uses the one on this machine. You
list its panes, run things in them, attach to one, or show the whole interface in your
terminal, all from here. It goes through your own `ssh`. The daemon there opens
nothing to the network.

You need two things:

- **`ssh` to that machine works without asking you anything.** No password, no
  passphrase to type, no question about the host's key. insensical runs `ssh` where
  there's nobody to answer, so use a key, loaded in your agent if it has a
  passphrase, and connect once by hand first. If `ssh build true` prints nothing and
  asks nothing, you're set.
- **`isc` is there, in the same version as here.** `isc host put` puts it there from
  this machine, on Linux: see "Putting insensical on a machine" below. Or install it
  there yourself. One line installs it by itself, for x86-64 or arm64, and takes a
  version: see [Installing](#installing). `~/.local/bin`, `/usr/local/bin` and
  Homebrew's `bin` are looked in even when a shell started by `ssh` doesn't.

Give the machine a name, once:

```sh
isc host add build me@build.example.com
isc host check build
```

- The second word is whatever you'd give `ssh`: a host, `user@host`, or a `Host` from
  your `~/.ssh/config`. Your ssh configuration, keys, agent and jump hosts all apply,
  because it's your `ssh` that runs.
- It's written in ASCII letters and digits and `.` `-` `_` `+` `@` `:` `%` `[` `]`,
  with `ssh://` before it if you like, as in `ssh://me@build:2222`. Anything else is
  refused wherever it comes from, the settings file included: a machine written
  there with something else is listed among the settings' problems and is never
  tried. A `Host` in your ssh configuration with other characters in its name isn't
  offered; give it a second name there.
- `isc host check` meets the machine and prints four lines:

  ```
  ✓ Reached build · 42 ms
  ✓ insensical 0.3.0 is there
  ✓ Its server is running
  ✓ 3 projects, 1 wants you
  ```

  Whether `ssh` got there, and how long a round trip takes. Whether insensical is
  there in a version that works with this one. Whether its server, the daemon, is
  running. What's open there. `✓` is a line that holds and `✗` one that doesn't, with
  what to do about it on the line under it. Lines after one that doesn't hold aren't
  tried, and are marked `·`. `→` marks what `isc host put` can put right. It changes
  nothing on the machine and starts nothing, and it fails unless the machine is ready
  to use. No server running yet isn't a failure: the first command that needs one
  starts it.
- `isc host ls` lists the machines you've named, with where `ssh` goes and what's set
  on each. `isc host ls --json` prints the same as JSON. `isc host rm NAME` forgets
  one. Nothing changes on the machine.
- `isc host set build --color teal --icon server --careful` sets those on a machine
  you've named. `--no-color`, `--no-icon` and `--no-careful` take them off again,
  `--ssh` changes where `ssh` goes, and `--name NEW` renames it. Only what you give
  changes. `isc host add` takes `--color`, `--icon` and `--careful` too.
- The names are lines under `[hosts]` in your settings file, and you can write them
  there yourself: `build = "me@build.example.com"`. A machine of your tailnet that
  `isc host add` found there also keeps its node:
  `build = { ssh = "me@build", node = "nABC123CNTRL" }`. See "A machine of your
  tailnet" below.
- `isc host answer build` lets `ssh` ask you what it has to ask. See "When `ssh` has a
  question" below.
- A machine can carry a colour, an icon, and a mark that it's one to be careful on.
  Write `build = { ssh = "me@build.example.com", color = "teal", icon = "server", careful = true }`.
  `color` is `#rrggbb` or one of `blue`, `green`, `orange`, `purple`, `pink`, `teal`,
  `gold`, `grey`, `rust` and `indigo`. With none set, a machine gets one of those ten
  from its name, so it's the same every time. `icon` is one of `laptop`, `desktop`,
  `server` and `cloud`; any other word is left in the file and shown as no icon.

**Putting insensical on a machine.** If a Linux machine has no `isc`, or an older
one, `isc host put build` puts this version there:

- It says what it would write, and asks. `--yes` skips the question. With no terminal
  and no `--yes`, it writes nothing.
- It writes one file, `~/.local/bin/isc`, over your `ssh`. No sudo. Nothing else there
  is changed.
- The file is the one for that machine's processor, x86-64 or arm64, in the version
  you have here. It's downloaded from the release onto this machine with `curl`,
  checked against the release's SHA-256, and then sent. The other machine doesn't
  need to reach the internet.
- There, it's written beside the old file, checked again with `sha256sum` or `shasum`
  if the machine has one, and run once to see that it starts and is the right
  version. Only then does it take the old file's place. If any of that fails, the
  `isc` you had there is still there, as it was.
- It only writes over an `isc` when the machine itself said it has none, or which
  version it has. If the `isc` there won't say its version, or it's this version and
  its link couldn't be made just then, `isc host check` says so and `isc host put`
  writes nothing. Try again, or look at it with `ssh build isc --version`.
- After the file is in place it looks once more. If the machine doesn't answer that,
  it says so and carries on to the question about the server: the file is there.
- **A server already running there is left running**, on the older version. Putting
  the file there never restarts it. You're told what a restart would come to, and
  `--restart-server` does it, or a yes to a second question at a terminal, or
  `isc host restart build` afterwards. A server
  that can move to the new version keeps everything that runs in its panes. One
  that can't is stopped and started, which ends what runs in its panes: you're told
  how many panes that is before you say yes. A server asked to move that can't
  refuses, and runs on.
- **It doesn't put insensical on a Mac**, or on a processor other than x86-64 and
  arm64. It says so, with the line to run on a Mac.
- **It doesn't write a second copy behind another.** If `isc` is already somewhere
  that's found before `~/.local/bin` there, such as `/usr/local/bin` or where a
  package put it, that copy is the one that would run. You're told where it is, and
  to update that one.
- **A build that isn't a release has nothing to download.** Name the file to send
  with `INSENSICAL_ISC_X86_64` or `INSENSICAL_ISC_AARCH64`, or install it there by
  hand. A file named that way is sent in place of the release's whenever it's set.

Then put `--host` before any command:

```sh
isc --host build ls
isc --host build run -- make test
isc --host build attach 3
isc --host build
```

- The last one is the whole interface, in this terminal, on that machine's projects.
- **From inside the interface you can go to another machine without leaving.** "Go to
  a machine" in the palette lists this computer and every machine you've named. Each
  is tried while you look, and says `reached` with the version of `isc` there, or
  `not reached`; mark that one and the line below says why. `Enter` shows the marked
  machine's projects. While another machine is shown, its name stands at the left of
  the top row. One machine is shown at a time, and what runs on the others runs on. A
  machine that can't be reached leaves you where you were, with a notice.
- `INSENSICAL_HOST=build` in a shell does the same for every command you run there,
  but for the ones that end something: `isc kill-server`, `isc close`,
  `isc tab close`, `isc project close` and `isc project sleep`. With only the variable
  set, those are refused and say which machine it names. They act on another machine
  when you say `--host build` on the command itself. A variable you set an hour ago
  shouldn't be why a server stopped.
- The daemon there is started by the first command that needs it, as it is here. It
  and its panes go on running when you leave, and when the connection drops.
- A pane runs on that machine: its shell, its files, its environment. With no
  `--cwd`, `isc --host build run` starts in your home directory there.
- **What belongs to that machine is that machine's.** The file finder lists its
  files. A new project's directory is a directory there, and `~` is your home there.
  A project's drawing is kept by the daemon there. `isc --host build integrate` sets
  up the agents there, and `isc --host build config set` changes the daemon's
  settings there: see [section 11](#11-agents) and
  [section 15](#15-settings-and-themes).
- A pane's history comes a part at a time: see [Scrollback](#scrollback).
- A server started for you on the other machine, by the first thing that needed
  one, isn't told of the `ssh` connection that started it: its panes have no
  `SSH_AUTH_SOCK`, `SSH_CONNECTION`, `SSH_CLIENT` or `SSH_TTY`. That connection ends
  long before the server does, and a forwarded agent's socket goes with it. For an
  agent in panes there, set `SSH_AUTH_SOCK` in your shell's profile on that machine
  to a socket that stays, or start the server there yourself from a login that has
  one.
- Each command makes its own `ssh` connection, which takes as long as `ssh` takes. To
  make that quick, turn on connection sharing in your ssh configuration
  (`ControlMaster auto`, `ControlPersist`).

**When the machine can't be reached.** A machine that has sent nothing back for four
seconds is given up on. A command then says "the link to the other machine was lost"
and ends. Nothing typed into the silence is delivered later: see
[Other machines in the window](#other-machines-in-the-window). The interface dims your panes and puts a small card over them: "build is out
of reach", how long ago it was last heard once that is over ten seconds, and the key to
leave. It tries after a second, then two, then four, up to every thirty seconds, with no key
pressed, as the window does; when the machine answers again, it shows your panes again.
Press any key and it tries at once. Nothing
there was touched. No daemon is ever started on this machine in its place.

If the machine won't let you in any more, because `ssh` would ask for a password or
doesn't know the machine's key, the interface tries once and stops. Each try would be a
failed login there, and a machine that counts those locks you out. It says what `ssh`
needs. Put that right, then press a key.

**Typing on a far machine.** When the round trip to the machine takes 70 milliseconds
or more, a letter you type would show that much later. So it's drawn at once, a little
fainter than the rest, and it turns to its usual colour when the program there shows
it. Nothing moves when it does. Your keys still go to the program at once, unchanged.

- It draws letters you type at the end of a line, Backspace over what you've just
  typed, and the left and right arrows over what you've just typed.
- Everything else waits for the program: Enter, Tab, keys with Ctrl or Alt, a paste,
  and a letter typed in the middle of a line.
- It starts once the program has shown two letters you typed, one after the other.
  So the first two or three letters in a pane, and after each Enter, show when the
  program shows them.
- A program that doesn't show what you type never has a letter drawn. A password
  prompt is one.
- If a guess is wrong, it's gone as soon as the program's answer arrives, and the
  next letters wait until two are shown again. A shell that expands an abbreviation
  does this.
- It doesn't happen in a program that takes the whole screen, such as an editor or a
  pager, in one that hides the cursor or has taken the mouse, while you're scrolled
  back, or while you're selecting on that line.
- The last column of the pane is left to the program.
- If a program that was showing your typing stops, with no prompt printed, the next
  letters are drawn until its answer is overdue: two round trips and 150
  milliseconds, two seconds at most. Then they're taken off.
- `type-ahead = "off"` turns it off. `"on"` draws ahead over any link to another
  machine, however quick. A pane on this machine never has it.

**On a slow line** each pane is sent only as fast as it's read, and panes take turns.
A pane that prints megabytes doesn't hold up what you type in another for more than
a moment, and it skips to its latest screen instead of showing you everything it
printed.

**From a browser.** `isc web` on this computer can carry a paired browser on to these
machines, over the same `ssh`. Only a device you made with `--far` gets that; see "Your
other machines, through this computer" under [In a browser](#in-a-browser).

**Tailscale, Headscale and WireGuard** need nothing of their own. The machine has a
name or an address on that network, and you give that to `ssh`:

```sh
isc host add build me@build.tailnet-name.ts.net
isc host add nas me@100.64.0.7
isc host add office me@10.8.0.2
```

With Tailscale SSH there's no key to set up; the rule is the same, `ssh build true`
must ask nothing.

**A machine of your tailnet is kept by its node, and dialled at its address.** A name
on a tailnet stops working when the machine is renamed, when the tailnet is, or when
this computer doesn't use the tailnet's DNS. The machine's address goes on. So when you
`isc host add` a machine and what you typed is one of your tailnet's machines, its
node's id is written beside it:

```sh
isc host add build me@build
# build is a node of your tailnet: it's dialled at the address Tailscale has for it, ...
```

- What you typed can be its name on the tailnet, with or without the tailnet's own name
  after it, the name the machine gives itself, or one of its addresses.
- Each time the machine is dialled, this computer's Tailscale is asked where that node
  is, and `ssh` goes to `me@100.x.y.z`, its IPv4 address, or its IPv6 address if it has
  no other. The user is the one you typed. No address is kept: it's asked for every
  time. Nothing is asked of any other machine.
- `ssh` is told to file the machine's host key under the name you typed
  (`-o HostKeyAlias=build`), so a key you accepted under that name already is found,
  and it's asked about once whatever the address becomes.
- A machine Tailscale says isn't online is **off**, at once. `ssh` isn't run to find
  out again, and nothing waits. It's tried again the next time it's asked for.
- If Tailscale can't be asked, isn't running, or no longer has that node, the machine
  is dialled as you wrote it, as it was before. If that fails, the reason says it was
  dialled as written. A machine that left the tailnet and joined again is a new node:
  you're told, and `isc host add` it again to follow the new one.
- `isc host ls` says `tailnet node, at 100.x.y.z now` for such a machine, or that it's
  off or gone. `isc host check` says the address before its four lines.
- `isc host add build me@build --no-node` keeps it as typed. So does a name your ssh
  configuration gives a machine: that name leads where your configuration says, and
  isn't looked for on the tailnet. The window's `Add Remote Host` dialog keeps the
  node of a machine you choose under `Tailscale` or type by its name on the tailnet,
  and changing a machine's address in the settings sheet looks for its node as
  `isc host set build --ssh me@build` does.
- `isc host set build --ssh elsewhere` sends it somewhere else: the node is looked for
  again, and taken off if there's none. Renaming it keeps the node.
- When this computer's Tailscale has no network interface (userspace networking),
  `ssh` is carried by Tailscale's own program, `-o ProxyCommand='tailscale' nc %h %p`.
- If Tailscale holds host keys for the node, which a machine running Tailscale's own
  ssh server gives it, those are the only keys `ssh` is told to accept for it on port 22, from a
  file under insensical's state directory (`tailnet-keys`). There's then no first
  question to answer.

**When `ssh` has a question.** A link is made by `ssh` told to ask nothing, so each
thing it would have asked is a machine that isn't reached. Each has its own words, and
one thing to do:

| It says | What happened | What to do |
|---|---|---|
| `ssh hasn't met build before` | `ssh` has no host key for it yet | `isc host answer build`, and say yes once you've compared the fingerprint |
| `build's host key is not the one ssh has for it` | The key changed. A machine set up again does this. So does another machine standing in its place | Find out why first. Nothing here says yes for you. When you're sure, `ssh-keygen -R build` forgets the old key |
| `ssh needs your key or password for build` | It wants a key, a passphrase or a password | `isc host answer build` shows what it asks. A key in your agent (`ssh-add`) or on the machine (`ssh-copy-id`) is what lets it in without asking |
| `Tailscale wants you to sign in again before it lets ssh through` | The tailnet's rules ask for a fresh sign-in | `isc host answer build` shows the link to sign in at |
| `Tailscale's rules don't let you in as root on build` | The tailnet's access rules refuse that user, or you | Whoever looks after the tailnet changes its rules |
| `build is off` | Tailscale says it isn't online | Turn it on, or start Tailscale there |
| `build did not answer in time`, `build could not be found`, `build refused the connection`, `build cannot be reached from this network` | As they say | The line under each says what to check |

`isc host check` prints `ssh`'s own last line under these, after `` `ssh` said: ``.

`isc host answer build` runs your own `ssh` to the machine at your terminal: the same
place a link goes, under the same name, but free to ask, and running nothing there but
`true`. You see `ssh`'s real question and answer `ssh` itself. insensical doesn't read
what you type, keep it or pass it on. Then it meets the machine and prints the four
lines. It succeeds only if the machine was then reached, and it needs a terminal. For a
machine you haven't named, give it where `ssh` goes: `isc host answer me@build`.
In the window, the `Answer` button runs it for you in a new tab on this computer, and
in the interface in a terminal `a` does in "Go to a machine".

- For a first connection, compare the fingerprint `ssh` shows with the one the machine
  gives of itself: `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`, run there.
- A password you type gets you in that once. A link is made without you, so a machine
  that only takes a password still isn't reached afterwards: it needs a key.
- The words for Tailscale's refusals are the ones its ssh server sends. The words for
  its sign-in aren't in its published source and are recognised by "requires an
  additional check" and "To authenticate, visit".

What it doesn't do:

- **A link can't ask you anything.** A host that wants a password, a passphrase or a
  yes to its key is refused, with what to do about it. `isc host answer` is where
  `ssh` asks.
- **The two versions must match**: the same protocol and terminal engine, which
  `isc version` prints. If they don't, you're told which machine has which.
  `isc host put` puts this version on the other machine and asks about its server.
  If the other machine's is the newer one, update this machine.
- **`--host` is for `isc` and the interface in a terminal.** The window shows every
  machine you've named at once: see
  [Other machines in the window](#other-machines-in-the-window).
- **Some commands are about this machine only**, and say so if you give them a host:
  `daemon`, `proxy`, `restart-server` and `host`. To restart the server on another
  machine, use `isc host restart build`: it meets the machine, tells you what a
  restart comes to, and asks. If the server there already runs the insensical that is
  installed there, there is nothing to restart it to: it says so, asks nothing and
  restarts nothing. Don't run `ssh build isc restart-server`: a shell
  started by `ssh` often can't find the server. `isc --host build kill-server` asks
  the daemon there to stop. It can't signal one that doesn't answer.
- **`integrate` can't ask you there.** With a host it needs `--status`, or `--yes`.
- **`config` reaches the daemon's settings there and no others.** A setting of the
  interface, such as `theme`, is read from this machine's file, and
  `isc --host build config set theme paper` is refused. Leave `--host` out for those.
- **The settings sheet in the interface is split the same way.** The daemon's
  settings, such as `scrollback`, are shown as the other machine's file has them and
  changed there. The interface's, such as `motion`, are this machine's.
- **The `integrate` command in the interface asks once and sets up every agent found
  there.** It doesn't show what would change in each agent's settings first, as it
  does on this machine. `isc --host build integrate --status` says which agents
  there are.
- **A path you click there is the other machine's, and nothing here is opened for it.**
  Only web addresses are followed on this computer. With `nvim`, `vim`, `hx` or `nano`
  in `open-files-with`, the click opens the file in a pane on that machine; otherwise it
  copies the path. See [What a click on a path does](#web-addresses-and-paths).
- **A pane's own number isn't used.** Inside a pane, `isc split` acts on that pane.
  With `--host` it doesn't, because the number is this machine's: name the pane.
- **Tab completion doesn't go there.** After `--host`, your shell still offers this
  machine's panes, tabs and projects. With `INSENSICAL_HOST` set it offers none.
- `INSENSICAL_SSH` names a program to run in place of `ssh`. It's given the same
  arguments. `INSENSICAL_TAILSCALE` names one to run in place of `tailscale`.

### In a browser

`isc web` serves insensical to a browser: a phone, a tablet, or a laptop that has
nothing installed. On a phone it opens on what needs you, with each agent's own
question and keys to answer it, and it can tell you by notification when you're
away. On a wide screen it's the sidebar and the tab in front, as it's laid out.
Anywhere, you can open a pane, read it, scroll back, and type into it.

It's a separate program that runs only when you start it. The daemon itself still
listens on nothing but its socket. `isc web` is one more client of it, like the
window.

**Start it**

```
isc web
```

It says where it's listening, which is `http://127.0.0.1:7433`: this machine only.
`--port 8000` picks another port. It runs until you stop it with `Ctrl`+`C` or
`isc web stop`. `isc web status` says whether it's running and where. `isc web stop`
also forgets that the settings sheet left it on, so the window doesn't start it again.

**Open it in your browser on this computer.**

```
isc web open
```

This makes an address that works once, for 10 minutes, and opens it in the browser your
computer opens links with (`xdg-open` on Linux, `open` on a Mac). The browser that opens
it is paired as the device `This computer`, with every right, and it stays signed in
until you remove the device. It never ends by itself. Run `isc web open` again for
another address for the same device; `isc web token ls` shows one device, and removing it
signs out every browser paired this way. If `isc web` isn't running, it says so.

`isc web open --print` prints the address and how long it works, and opens nothing. So
does it where no program opens links. `isc web open --json` prints
`{"url": …, "until": …}`, with `until` in seconds since the epoch, and opens nothing. The
address is always the one for this machine, `http://127.0.0.1:7433/#…` (`https` when `isc
web` has a certificate), never one you gave with `--name`.

The address pairs only a request that comes from this machine and asks for `127.0.0.1`,
`localhost` or `[::1]`. From anywhere else, it's refused as not a token and isn't used up,
so a link that leaks is no use off this computer. A port forward of yours that ends on
this machine's own address (`ssh -L` on a computer you reached it from) does pass that
test, so keep the address to yourself for its 10 minutes.

A browser that opens `http://127.0.0.1:7433` without being paired sees the pairing screen
with one more line: on this computer, open Settings > Server in insensical and press Open
in Browser. The same button is in the window.

**Make a token and pair a browser**

Nobody else can use it until you make a token:

```
isc web token add --name phone
```

That prints the token, an address that ends in the token, and the same address as a
code a phone's camera can read. Open the address, or scan the code, and the browser
is paired. The token is shown this once and kept nowhere, so if you lose it, end it
and make another.

**The address pairs one browser, and only for ten minutes.** Once a browser has paired
with it, or when the ten minutes are up, it pairs nothing more, and the page says
which: "This code was already used" or "This code has run out". The browser that
paired stays paired. To make another address, make another token. `--uses` and
`--expires` say otherwise:

```
isc web token add --name family --uses 3 --expires 1h
isc web token add --name desk --uses any --expires never
```

`--uses` is a number of browsers, or `any`. `--expires` is how long the address pairs,
in minutes, hours or days (`10m`, `1h`, `7d`), or `never`. A token made by an older
insensical has neither limit: its address pairs any number of browsers, with no end.

- The browser stays paired after that, through restarts of `isc web` and of the
  daemon. **Forget this browser** on the page ends it for that browser.
- A browser you paired with an older insensical asks for the token once more, and
  so does one that has had its site data cleared. The token still stands: open its
  address again, or type it in. If you no longer have it, make another.
- A browser is paired at one address. Reach `isc web` by another name or port and
  it's a different browser as far as pairing goes.
- One token can pair several browsers. `isc web token ls` shows each token, what it
  may do, when it was last used, how many more browsers its address pairs and for how
  long (or that it's used up or has expired), when it ends if it has an end, and how
  many browsers it has paired. It never shows a token.
- `--url https://desk.example.ts.net` prints the address and the code for a name
  other than this machine's own, when that's how the browser will reach it.
- You can also type the token into the page, or press the camera button beside the
  field and point the phone at the code `isc web token add` printed. The button is
  there where the browser may use a camera, which is `https` or this machine's own
  address. A browser without a QR reader of its own, such as Safari, uses one the page
  brings and fetches only when you press the button.
- **A one-device code also pairs that device's home-screen app.** On an iPhone the page
  you add to the home screen is a separate browser, and it opens with the address the page
  had when you added it. So once a code that pairs one device has paired your phone's
  browser, the same code pairs one home-screen app too: when the app says it is one, it
  comes within fifteen minutes of the browser's pairing, it comes from the same network
  address, and no app has been paired from that pairing yet. The app has the token's
  rights, counts in `isc web token ls` as one more browser of the same device, and ends
  when the device does. A third browser, or one from another network address, or a late
  one, is refused, and the page says so in plain words. Everything else about a used or
  late code stays as it is. A copy of a copy is refused too. Once paired,
  the page tells you how to keep it on your phone: tap Share, then Add to Home Screen.
- **Another way to pair a home-screen copy.** On an iPhone the page you add to the
  home screen has storage of its own, so it doesn't share Safari's pairing, and the
  address you scanned no longer carries the token. In the page's menu (the letter at
  the top), **Add to Home Screen** makes a one-use address of the same token, good for
  ten minutes, and puts it in the page's address. Then tap the share button and **Add
  to Home Screen**; the copy that opens from there pairs itself and takes the code out
  of its address. It counts in `isc web token ls` as one more browser of the token,
  has the token's rights, and ends when the token does. An address that has been used or
  is late says so, as any pairing address does. So "pairs one browser" counts addresses
  used, not copies of one device: any paired browser can pair its own home-screen copy
  this way, after its address is spent. It's bounded: a token makes at most three such
  codes in ten minutes (the page is told to wait), a browser holds one code at a time
  and asking again cancels its own, and a code is cancelled when the browser that asked
  for it leaves or is removed.

**What a token may do**

A token carries rights, the same four the daemon gives every client: `look`, `type`,
`arrange` and `manage`. See [Who is connected](#who-is-connected). Left alone, a
token has all four. Any of `type`, `arrange` and `manage` is enough to run programs
as you, so a token with one of them is a key to your account on that machine.
`manage` is the one to think about most: a device that may manage can change what
the server runs to tell you (`notify-command`), so it can run programs as you. Give
**Can do everything**, or `manage`, only to devices that are yours.

```
isc web token add --name hallway-screen --rights look
isc web token add --name guest --rights look,type --expires 1h --uses 2
```

That address pairs up to two browsers in the next hour. `--expires` ends the address,
not the browsers it paired: a browser that paired stays paired, with these rights,
until you run `isc web token rm guest`, or until the time `--stays` gives it.

**A device that ends by itself.** To pair a co-worker's or a client's browser for a
while and not have to remember to remove it, say how long it stays:

```
isc web token add --name client --rights look --stays 1d
```

`--stays` is how long the device stays paired, counted from when you make the token:
minutes, hours or days (`30m`, `1h`, `7d`), or `always`, which is the default. When the
time comes, every browser paired with the token is signed out at once, and what it had
open is closed. Its notifications stop, any code it made for a home-screen copy stops
pairing, and the token is gone from `isc web token ls`. `isc web token ls` says "ends in
40m" until then, and `--json` has `stays_until` in seconds since the epoch, or `null`.
`--expires` is still only the address: if `--stays` is shorter, the address ends with
the device, and `token add` says so. A browser paired from a home-screen code ends with
its token.

A device sees every terminal on this computer unless you make it with `--only`, which
is next. Either way it must be able to reach this computer.

**Share one project or one terminal**

To show a co-worker or a client one thing and nothing else, share it from the window.
Press the eye on the project's row in the sidebar, or in the terminal's header, or
choose "Share this project" or "Share this terminal" in the palette. One dialog
opens:

- **What they see** is fixed: only this project and anything you open in it, or only
  this terminal wherever you move it.
- **They can** is `Look` unless you change it. `Look and type` shows a warning, and
  the link can't be made until you press `Let Them Type` under it: typing in a terminal
  here is typing as you.
- **Where they are** is `On my network` unless you change it to `Anywhere on the
  internet`. See "From the window, for someone who isn't on your network" below.
- **Ends** is `1 day` unless you change it to `1 hour`, `1 week` or `When I end it`.
  When the time is up they're signed out.

`Make the Link` shows a QR code and a link with a button that copies it. Send them the
link. It works once, for a day, or until the share ends if that's sooner. They open it
in a browser and install nothing. If "Open in a browser or on your phone" is off, the
dialog says so, the button reads `Turn On and Make the Link`, and it turns that on
first, as the switch on the Server page does. If the link's address opens only on this
computer, the dialog says that too: someone elsewhere needs Tailscale turned on, on the
Server page.

While something is shared, its row in the sidebar and the header of each terminal a
guest sees carry a small sign: an eye where guests look, a keyboard where one can type.
The same button then opens the list of who it's shared with, each with `End`, which
signs them out at once and stops their link. `Share With Someone Else` makes
another link. Settings also lists every guest, on the Server page.

- You can share only what's on this computer, not a project on another machine.
- The window names the guest after what's shared: `api guest`, then `api guest 2`.
- The window learns of a guest you make on the command line when it next asks: when it
  starts, when you open the dialog or the Server page.

**From the window, for someone who isn't on your network**

A client who isn't on your network or your tailnet can't open the link above. In the
dialog, set **Where they are** to `Anywhere on the internet`. The dialog then changes:

- **They can** is `Look`, and there's no other choice. Over the internet a guest can
  only look.
- **Ends** is `1 hour`, `1 day` or `1 week`. There's no `When I end it`: a public link
  always has an end, a week at most.
- Under the card it says what the button does, before you press it: it opens an
  address on the public internet, through your own Tailscale, that shows this one
  project or terminal to whoever has the link, to look at and nothing more, until the
  time is up or you close it.

`Open a Public Link` does three things for you, each a command you could type: it has
`isc web` answer to this computer's Tailscale name at its guest door (`isc web
--guest-name`, which restarts it and drops no paired device), makes the guest (`isc web
token add --public`), and opens the address (`isc web public on`). Then it shows the
public link and its QR code. Whoever opens the link first gets in, and it works once.
Don't post it where others can read it.

- **What it exposes.** The address is Tailscale's, made by Tailscale Funnel on port
  8443 of this computer's Tailscale name. Anyone on the internet can reach its sign-in
  page. Only someone with a link you made gets past it, and only to look at what you
  shared. Nothing of insensical's own listens on the internet.
- **If it can't be opened, the dialog says why and what to do.** Tailscale isn't
  installed: `How to Install` opens the instructions. It isn't signed in: `Sign In`. Funnel
  isn't turned on for your tailnet: `Turn On Funnel` opens Tailscale's own page for it.
  Tailscale already serves something else on port 8443: it's left as it is. When the
  last step fails, the guest it made is removed again, so nothing is left half open.
- **While a public link is open, the window says so.** A sign reading `Public link
  open` stands in amber in the bar, or at the sidebar's foot, or over the window's
  corner when you've hidden both. It says how many it's shared with and when the first
  of them ends. Press it to see what's shared publicly, each with `End`.
- **To close it**, press the sign and then `Close Public Link`, or press `Close` on the
  `Public link` row of the Server page. That's `isc web public off`: every public guest
  is signed out and Tailscale no longer leads to this computer. The window says when
  it's closed.
- **When you end the last public guest, the window closes the public link too**, and
  says it did. If it finds the address open with nothing shared, the sign and the
  Server page say so and offer to close it.
- The Server page lists a public guest as `Guest · public link · only project api ·
  can look · ends in 23h`.
- The window reads whether a public link is open from what `isc web` wrote down, when
  you share or end something, when you open the Server page, and every 30 seconds. So a
  link opened on the command line shows within half a minute.
- The tray's menu on Linux doesn't say a public link is open. Its lines come from the
  server, which knows nothing of `isc web`.

The dialog runs the command below for you. To make the device yourself, use `--only`:

```
isc web token add --name sam --only project:api --stays 1d
isc web token add --name kim --only pane:12 --rights look,type --stays 1h
```

That device is a guest. Send them the address it prints, as for any device.

You can make a guest from the page too: **Settings**, **Your devices**, **Add a
Device**, and under **What it sees** choose **One project** or **One terminal**.
See "Manage it from the page" below.

- **What a guest sees.** With `project:api`, that project: its tabs and terminals, and
  anything you open in it later. A terminal you move out of the project is gone from
  their page. With `pane:12`, that one terminal, wherever you move it, and not the name
  of the project it's in. Nothing else on this computer is listed or shown to them: no
  other project or terminal, not who else is connected, not your settings, not your
  home directory. They do see this computer's name.
- **They see all of it.** A guest can scroll back through everything that terminal has
  shown, including what was printed before you shared it, and sees what you type there
  while it's shared.
- **What a guest can do.** Look, and that's all unless you say `--rights look,type`.
  Left out, `--rights` is `look` for a guest. `arrange` and `manage` are refused with
  `--only`: a guest never opens, closes, moves or renames anything, and never changes
  a setting.
- **What a guest never gets.** Notifications: their browser isn't sent any, and can't
  ask for them. The Settings screen, the list of devices, and making a device. Being
  taken for you: a guest looking at a terminal doesn't mark it seen, and you're still
  notified about it.
- **A guest who can type has your account.** Typing in a shell is using your account
  on this computer. Someone who can type in a terminal can read your files and run
  anything as you, here and wherever this computer can reach, and that includes
  running `isc` to see every other terminal. `--only` keeps their page to one project.
  It doesn't confine what they type. Share typing only with someone you'd hand your
  keyboard to. `token add` says so each time you make a guest who can type. A guest
  who can only look runs nothing: for them "only this terminal" is exact.
- **It's the project you named, by its number.** `project:api` is looked up when you
  make the device, and the share is to that project's number from then on. Rename the
  project and the guest sees the new name; call another project `api` later and the
  guest doesn't move to it. `token add` and `isc web token ls` both give the number:
  `only project 4 (api when made)`. If two projects are called `api`, `token add`
  refuses and gives their numbers: say `--only project:4`. A terminal is always by
  its number, as `isc ls` shows it. The project or terminal has to be open when you
  make the device, and the server running.
- **Give it an end.** `--stays 1h`, `1d` or `7d` signs the guest out by itself when the
  time is up, as for any device. Without `--stays` a guest stays until you remove it.
- **To end it now**, `isc web token rm sam`. Their page closes within a second. If you
  close the terminal or the project instead, the guest has nothing left to see, and
  the device stays paired to nothing until its time is up or you remove it.
- **What a guest can still learn.** Pane numbers come from one counter, so the number
  of a new terminal in their project tells them roughly how many terminals you've
  opened in all. Never what they are.
- **A share ends if insensical here is reset.** If insensical starts on this computer
  with nothing saved to go on from (its saved state was deleted or couldn't be read),
  it counts terminals and projects from one again, so a number could mean something
  else. Every guest device then stops working, and `isc web token ls` shows it as
  `ended` and says why. Remove it and make a new one. A device made by an earlier
  version of `isc web` ends this way too. `token add --only` is refused while insensical
  isn't running.
- **What a guest's page looks like.** A bar stands across the top: "You're a guest on
  desk. You can look at api.", or "You can look and type in api.", or "You can look at
  one terminal.", and when the share has an end, "Ends in 23h". Under it is what you
  shared and nothing else. A guest of a project has that project's list and its tabs. A
  guest of one terminal has that terminal, filling the page. There's no "Needs you", no
  "Everything", no Settings, no bell and no offer of notifications. A guest's page never
  takes a terminal's size: there's no "Fit to this screen", and they see each terminal at
  the size and in the colours you have it. They can still scroll, find, copy, and change
  the size of the letters on their own screen. A guest who only looks has no keys on a
  phone, and pressing a key says "You can look, not type." The letter at the end of the
  bar has the text size, the theme, and **Leave**: "You'll need a new link to come back."
- **What they see when it's over.** If you close the terminal or the project, their page
  says "This was closed. What was shared with you isn't open any more. Ask for a new
  link if you still need it." It says the same if you remove the device. When the time
  is up it says "Your time here is up. Ask for a new link." After any of these the page
  stops asking the server for anything. A new link from you starts again.
- **Someone who isn't on your network.** Add `--public`, and then open the address:

  ```
  isc web token add --name sam --only project:api --stays 1d --public
  isc web public on
  ```

  This puts an address on the public internet that shows that one project, or that
  one terminal, to whoever has the link. Read that twice before you use it.
  - **It's look only.** A device for the public address can never type, whatever you
    ask for: `--rights look,type` is refused with `--public`. Someone who types in a
    terminal is using your account, and that's never left one link away from the
    internet. To type together, share on your own network instead.
  - **It ends within a week.** `--stays` is required and is 7 days at most. The link
    pairs one browser, for ten minutes, unless you say `--uses 2` or `--expires 1h`;
    `--uses any` and `--expires never` are refused.
  - **Anyone can reach the sign-in page.** The address shows up in public lists of
    certificates within minutes and gets scanned. A stranger gets the page and
    nothing else. Only the link gets in, and only to what you shared. Don't post the
    link where others can read it.
  - **The address is Tailscale's.** `isc web public on` asks your own Tailscale to
    open it with Funnel, at `https://your-machine.your-tailnet.ts.net:8443`. Nothing
    of insensical's own listens on the internet, and nobody of ours is in between.
    Your devices, with all their rights, don't work at that address at all: a lost
    phone or a leaked code of your own is worth nothing there.
  - **Close it with one command.** `isc web public off` removes every device made
    for the public address, signs them out at once, and takes the address away.
    `isc web token rm sam` ends one. When the last one ends or is removed, `isc web`
    stops listening for the public address by itself, but the address itself stays
    with Tailscale until you run `off`. `off` always closes the address `on` opened:
    it doesn't matter that `isc web` has stopped since, or which `--guest-port` it
    had. If Tailscale won't close it, or isn't running to be asked, `off` still
    removes the devices, tells you what Tailscale said, and fails, so you know to run
    it again.
  - **See how it stands.** `isc web public status` says one of these:
    - `Open at https://… for 1 device (sam, ends in 23h).` The address is open and
      someone can use it.
    - `Open at https://…. The public address is open but nothing is shared: close it
      with isc web public off.` Tailscale still has the address, and no device is left
      for it. Close it: the address leads to a port on this computer, and you don't
      want another program to end up behind it.
    - `Closed.` Tailscale has no public address for insensical, or this computer has no
      Tailscale to ask. If devices are waiting for one, it lists them.
    - `No public address is open to insensical.` and what Tailscale serves on port
      8443 instead, which is yours and is left alone.
    - `Tailscale holds a public address for insensical under` a name this computer
      no longer has. Close it with `isc web public off`.
    - `Not known.` You opened an address and Tailscale isn't answering now. Run `isc
      web public off` once Tailscale is running.

    It succeeds when the address is open and fails otherwise, so a script can ask.
    `isc web public status --json` says the same as data and always succeeds: `state`,
    `open` (true while Tailscale says the address stands, false when it doesn't, null
    when Tailscale can't be asked), `address`, `port`, `leads_to` (the port of this
    computer it points at), `devices` (each with its `name` and `ends`, in seconds since
    the epoch), `record` (the port `on` wrote down) and `record_without_device`.
    A `record` with `open` false is one that was left behind. `isc web status --json`
    carries the same under `door`, with `listening` and `door_port`, and asks Tailscale
    only when there's a device, a record or a listening door to ask about.
    `isc web token ls` marks these devices `public`.

  For this, start `isc web` with the name Tailscale gives this computer, as
  `isc web --guest-name desk.example.ts.net`: `isc web public on` tells you the name
  if it's missing. If Funnel isn't allowed for your tailnet yet, `on` gives you the
  link where you turn it on. If Tailscale already serves something else on port 8443
  it's left alone and `on` says so. Without Tailscale you can point a tunnel of your
  own at `127.0.0.1:7434`, but whoever runs a tunnel that ends the encryption reads
  everything that goes through it, the link included.

`isc web token ls` says what each device sees: `everything`, `only project 4 (api when
made)` or `only pane 12 (claude in api when made)`. `--json` has `scope`, `sees` and `reset`.

`--rights` goes on the token or the login, as you add it. `isc web --rights look`
is refused: the server goes by each token's own rights, and has none of its own.

- A token with only `look` can read every pane and its history and can't type,
  resize or change anything. The page says so. The daemon is what refuses, so a
  page somebody wrote themselves gets no further. It doesn't count as you looking
  either: a pane open in that browser still notifies you, and stays `done` or
  `failed` until you see it somewhere you can type. What it can still do is in
  [Who is connected](#who-is-connected).
- `--expires` ends the address, not the browsers it paired: they stay paired until
  you end the token with `isc web token rm`.
- `isc web token rm phone` ends a token at once. Every browser paired with it is
  closed within a second, whatever it was doing. Do this for a phone you've lost.
- `isc clients` lists a browser as `web`, under its token's name, with its rights.

**Your other machines, through this computer**

A device can be let through to your other machines: the ones `isc host ls` lists on the
computer `isc web` runs on. Make it with `--far`:

```
isc web token add --name phone --far
```

Or give it to a device that's already paired, and take it back:

```
isc web token set phone --far
isc web token set phone --no-far
```

- It's all of your machines or none. You can't pick some.
- `isc web` reaches a machine the way the window does: your own `ssh`, running `isc
  proxy` there. The other machine needs `isc` and nothing else. It doesn't need `isc
  web`, and nothing new listens on it.
- The device has the same rights there as here. The other machine's `isc clients` lists
  it as `web`, under its token's name.
- Only a device that may type can have `--far`. It could already type `ssh` in a terminal
  here. A device that only looks can't have it.
- **A guest never can.** A device made with `--only` or `--public` stays on this
  computer. `--far` with either is refused, and a guest's browser isn't told that you
  have other machines, or what they're called.
- A device made before this stays on this computer until you run `token set`.
- `--no-far` closes what the device has open on another machine, within a second.
  So does removing the device, its time running out, and taking the machine out of
  the settings or sending it somewhere else, each with a sentence that says why.
- The `ssh` that `isc web` runs for this is told to forward nothing: no agent, no X11, no
  ports, whatever your `~/.ssh/config` says. The window's own `ssh` is as it always was.
- `isc web token ls` marks these devices `and your other machines`.
- `ssh` runs only while a browser has a terminal of that machine open, or while a
  device with `--far` has notifications on: see "Notifications on your phone" below.
  When neither holds any longer, it ends.
- If a machine can't be reached, the browser is told it's out of reach within ten
  seconds. `isc web` tries again when the browser next asks: no sooner than a second
  after the first failure, then two, four, and at most thirty.
- If `ssh` would ask for a password or a passphrase, `isc web` doesn't keep trying: each
  try is a failed login on that machine. Fix it as "When the machine can't be reached"
  says under [Other machines, over ssh](#other-machines-over-ssh), then press **Try
  again** under the machine's heading on the page. It can be asked once in ten seconds:
  asking sooner is answered and does nothing.
- What `ssh` or the other machine said when it couldn't be reached is passed on to the
  page as one line of at most 300 characters. It can include your user name and the
  machine's address, and the page of a device you gave `--far` is yours.
- While `isc web` is still reaching a machine for one browser, another that asks for it
  is told so at once and asks again, rather than waiting behind it.
- A machine with another version of insensical can't be opened. Install the same
  version on both.
- A machine's name must be letters, digits, `-`, `_` and `.`, as `isc host add` takes
  it. One you wrote into the settings with other characters isn't offered to a browser.

On such a device the page lists your other machines too:

- **Each machine is a heading over its projects.** This computer is first, then the
  others in the order your settings have them, each with its icon and colour as the
  window shows them. A device without `--far`, and a guest, see the page as it always
  was: no headings, and nothing about other machines.
- **A machine is folded until you open it.** Press the arrow on its heading. That's
  what connects to it, so a machine you never open costs nothing. The browser
  remembers which you had open and opens them again next time. Fold it and the page
  lets go of it: its terminals leave the list, **Needs you** and the counts.
- **Needs you gathers every open machine.** Each card says which machine it's on, and
  an answer goes to that machine. The counts in the title, on the badge and on
  **Activity** cover every open machine. A folded machine isn't counted.
- **A terminal on another machine says so.** Its header starts with a chip: the
  machine's icon and name in its colour. A terminal with no chip is on this computer.
  Two machines can both have a terminal 3. They're never mixed up, and what you type
  goes only to the one you're looking at.
- **An address names the machine.** `?pane=3&at=atlas` is terminal 3 on `atlas`. Press
  a notification about another machine's terminal and the page opens that machine, if
  it wasn't open, and lands on that card. A machine opened that way is open until you
  fold it or leave the page. Only a press on its heading is remembered for next time.
- **A machine you take out of the settings is forgotten by the page** the next time
  it's loaded. If you later give another machine the same name, it starts folded.
- **When a machine can't be reached**, its heading says "out of reach · trying again",
  its terminals dim and say "Link to atlas lost · trying again", and what you type
  there isn't sent: the page says "Not sent". This computer's terminals carry on. The
  page asks again by itself, after a second, then two, four, and at most thirty, and
  the machine comes back by itself.
- **"needs a password"** means `ssh` would ask for one, so nothing tries again by
  itself. Fix it as "When the machine can't be reached" says under [Other machines,
  over ssh](#other-machines-over-ssh), then press **Try again** under the heading.
- **"another version"** means that machine runs a different version of insensical.
  Install the same version on both, then press **Try again**.
- **"closed: it sent a number no machine gives"** means that machine's insensical sent
  the page something no insensical sends. The page closes it and shows nothing of it,
  so that it can't pass for another of your machines. Fold it and open it again to
  try once more. If it says so again, don't trust that machine until you know why.
  **"closed: it sent more than a machine holds"** and **"closed: it sent what no
  machine sends"** mean the same, for a machine that listed more than 4,096 terminals,
  tabs or projects, or something that isn't a list at all.
- What a terminal on another machine has to tell you is said in the page's corner
  with that machine's chip, so you know whose it is before you press it. The page
  says four things at once at most there, and two of them at most about one other
  machine. Older ones go. **Needs you** still has every terminal
  that's waiting.
- **Devices**, **Add a Device** and the settings on the page are this computer's.
  You can't share another machine's project or terminal from here.
- You can't make a project or a terminal on another machine from the page, or move a
  terminal from one machine to another.

**Manage it from the page**

A browser paired with a token that may do everything, or at least change settings, has
**Settings** in the menu under its letter. It opens a screen for the computer `isc web`
runs on, so you can do from a phone what `isc web token`, `isc web push`, `isc notify`
and `isc clients` do:

- **This computer** says its name, that insensical is running, and its version.
- **Your devices** begins with **Add a Device**. It asks for a name and what it may do
  (everything, look and type, or only look) and how long it **Stays paired**: until you
  remove it, 1 hour, 1 day or 7 days. Under More options are how many devices the code
  pairs and for how long. It then shows a code to scan and the link, which is this
  page's own address, so a phone on the same network or tailnet can open it, and says how
  long the code works and how long the device stays paired. A device you add here sees
  every terminal on this computer, and must be able to reach it, unless you change
  **What it sees**. That's **Every terminal on this computer**, **One project** or
  **One terminal**, and with either of the last two you pick which, from what's open
  now. The device is then a guest, as `isc web token add --only` makes one: see "Share
  one project or one terminal" above. A guest can **Look** or **Look and type**, and
  nothing more. It starts on Look, and on ending in 1 day. Choosing Look and type shows
  what typing gives away, and **Add** stays dim until you press **Let Them Type**. Once
  it's made, the dialog says what the guest can do: "sam can look at api." You can't make
  a device that may do
  more than this browser may, no more than six in a minute in all, and no more than three
  from one browser. A device never stays paired longer than the browser that made it:
  if this browser has an end, so does what it makes. A device that may manage can change
  what the server runs to tell you, and so can run programs as you: the dialog says so
  under **Everything**, which is for devices that are yours.
  Under it is each token, the one this page is paired as first and marked "this device":
  what it may do, for a guest what it sees, in the words `isc web token ls` uses ("only
  project 4 (api when made)") and marked "guest", when it was last used ("last used just
  now", "last used 5m ago", or "not used yet" if no browser has paired), how many
  browsers it paired if that's more than one, and, while its code still works, how long it has left. A device with an end says "ends in
  40m", in the accent when it is under ten minutes away, and its row is gone the next time
  the page looks. A page paired as a device that ends is signed out and says "This
  device's time is up. Ask for a new code." **Remove** asks once and then ends it at
  once, like `isc web token rm`; on the device this page is paired as, it signs this page
  out.
- **Notifications on your devices** lists the browsers that asked for them and **Send a
  Test** says for each whether its push service took it. One test is sent at a time,
  and a browser can ask for four a minute.
- **Alerts when away** sets where the server tells you when nobody is looking: ntfy (a
  server, a topic and a token if your server needs one) or another web address, with a switch for only
  when you're away, a test that answers in a sentence, and under More options the
  rest, including a command. The page is never given a token or password back: it
  shows only its last few characters, and **Clear** takes it out. An address is shown
  only as far as its host and the first part of its path, which is a topic's whole
  name: the rest is where a secret is often written, and the page says that the address
  is shortened. A shortened address is shown as it is and can't be edited: **Change**
  empties the field so you can type the whole address, and changing only the token or the
  switch never rewrites the address. The shortened form is refused as a new address. Nothing else of the
  server's settings can be changed from the page.
- **Connected now** says in words what is connected and what it may do, and
  **Disconnect** cuts one off, as `isc disconnect` does. This page's own row has no
  button.

On a wide screen the page is a panel in the middle of the window, larger in a larger
window, with the pages in a list at its left and a search field above them: type a word
and the list keeps the pages that say it and shows the first. Each page is a few small
cards, and the line under a row's name is always shown whole. The arrows at the top step back and forward through the
pages you've looked at. **Escape**, the close button or a press outside closes it. On a
phone it is one page, with a back arrow, and a dialog rises from the bottom. Every button
can be reached with Tab, and a dialog keeps Tab to itself and gives focus back when it
closes.

The screen refreshes every few seconds while it is open. A browser whose token may not
change settings isn't shown Settings, and the server refuses it if it asks.

**Reach it from another device**

`isc web` listens on this machine only, so a phone needs a way in. Use one you
already trust:

- **ssh.** `ssh -L 7433:127.0.0.1:7433 you@this-machine`, then open
  `http://127.0.0.1:7433` on the device that ran `ssh`. It pairs with a token like
  any other.
- **Tailscale.** `tailscale serve 7433` gives the machine an `https` address inside
  your tailnet. Start `isc web` with that name, `isc web --name desk.example.ts.net`,
  and make the token with `--url` and that address.
- **Headscale, or a proxy of your own.** Point it at `127.0.0.1:7433`, with your own
  certificate. It has to speak TLS to the browser, pass the `Host` and `Origin`
  headers on as they came, and pass WebSockets. Start `isc web` with `--name` and the
  name the browser will ask for.

- **A public address, for a guest.** `isc web public on` has Tailscale Funnel open
  one, to a second door that `isc web` keeps for it at `127.0.0.1:7434`
  (`--guest-port` changes the port). That door is on this machine only, always. It's
  open only while a device made with `--public` exists, lets in only those devices,
  and they can only look. It answers to the names you give with `--guest-name`, and
  to `127.0.0.1` so you can check it from this computer. It doesn't answer to
  `--name`'s names, to `localhost`, or to any other address: if a tunnel of your own
  asks for it by another name, give that name with `--guest-name`. `--guest-port`
  can't be the port `isc web` itself listens on, and `isc web` won't start if it is.
  If you open your own public link in a browser that's already paired, it stays
  paired: the two doors keep separate cookies. See "Share one project or one
  terminal" above.

`isc web` answers only to names it knows are its own: this machine's addresses,
`localhost`, and each `--name` you give it. Asked by any other name it says so and
gives nothing, not even the page. That keeps another site's page from reaching it
under a name of that site's making.

`isc web` won't listen anywhere else without encryption. `--listen 0.0.0.0` is
refused unless you also give `--tls-cert` and `--tls-key`, a certificate and its key
as PEM files. With those it speaks `https` only.

- Over `ssh -L`, the browser's address is plain `http`, and it's `ssh` that
  encrypts. Over Tailscale it's Tailscale.
- **Other accounts on this machine can reach the port**, where they can't reach the
  daemon's socket. Without a token they get the page and nothing else.

**Let Tailscale say who you are**

With Tailscale in front, you can skip tokens for yourself:

```
isc web login add you@example.com
isc web --trust-proxy you@example.com
```

`tailscale serve` tells `isc web` which tailnet user is asking, and `isc web` then
trusts the logins you named, with the rights `isc web login add` gave each
(`--rights look` for fewer). It believes this only from a proxy on this machine.
**Use it only on a machine where every account is yours**: any program on the
machine can claim to be that proxy.

**On a phone: what needs you**

The first screen is a card for each pane that wants you, the most urgent first:
panes that wait, then ones that failed, then ones that finished.

- A card says which program, which project and which machine, the state in the
  words the window uses, and **the program's own question** in large type. Beside it
  is how long the pane has waited, in bold once that's more than ten minutes.
- Under the question is **the foot of the pane's real screen**, live. It's what the
  agent drew, so its choices are there to read. Slide a finger across it to see
  what's cut off at the right. Tap it to open the pane.
- **The keys** under it are `Enter`, `y`, `n`, `1`, `2`, `3` and `Esc`. Each sends
  that key and nothing else. They're the same on every card: the page doesn't read
  the screen to guess what an agent is offering, so look at the foot to see which
  key means what.
- **The line** is for a whole answer. Type it, or use your keyboard's microphone,
  and press the arrow: it's sent as you'd typed it, then `Enter`. Nothing is sent
  until you press the arrow.
- A pane that waits for a password gets a field that hides what you type, with
  `Enter` and `Esc`. A pane that only failed or finished gets **Open the pane** and
  **Seen**.
- When you answer, the card says what was sent. When the agent goes back to work,
  the card goes. If it asks something else, the card stays and its question
  changes.
- With nothing waiting, the screen says **All quiet**, and how many panes are at
  work.
- **Everything**, at the bottom, is every project, tab and pane, as the window's
  sidebar lists them.
- The switch over the foot of that list narrows it to **Activity**: only the panes that
  are at work or have something for you, each with its state in words and how long it
  has been so, under its project's name. Tap a row to open its pane. With nothing to
  list it says "Nothing is at work". It's the window's list, kept quiet by the same
  rules ("Showing only activity" under The sidebar): a row doesn't flicker as an agent
  pauses, a pane that finished dims once you've looked at it and goes 5 seconds later,
  and no row appears, leaves or changes place while your finger is on the list or moved
  on it in the last 2 seconds. Your browser remembers which you chose, and the place at
  the bottom is named for it.
- The number on the title, on **Needs you** and on the switch is how many panes need
  you: the ones that wait and the ones that failed. A pane that finished is counted
  beside it, with a tick, and never in it. The page's tab and the home-screen icon's
  badge say the same number.

**A pane**

- A pane opens at your screen's width, from the bottom: its columns are what fit
  across at the current size of letters, its rows what fit between the bar and the
  keys, so there's nothing to slide sideways. Slide down at the top to scroll back.
  History arrives in parts: the newest first, and more each time you reach the top.
  The view follows new output while you're at the end. Pinch to make the letters bigger
  or smaller; the columns and rows follow once your fingers lift. When the keyboard
  comes up or goes, the rows follow it.
- Tap the pane to bring up the keyboard. The page shrinks to what the keyboard
  leaves, and the cursor stays in view.
- **The bar of keys** has what a phone's keyboard lacks: `Esc`, `Tab`, `Ctrl`,
  `Alt`, the arrows, `Ctrl+C`, `Shift+Tab`, `~`, `/`, `|`, `-`, paste, and a pencil.
  `Ctrl` and `Alt` hold for the next key, from the bar or from the keyboard. Hold
  an arrow and it repeats. Pressing a key never puts the keyboard away. The pencil opens a line you can type or dictate into,
  sent whole.
- **Fitting is on.** The button beside `…` (also **Give the size back** under `…`)
  shows whether the pane has your screen's size, and switches it. The window at your
  desk shows the pane at that size while you're in it. Leaving the pane, or putting the
  page away, gives the size back, and coming back fits it again. Turn fitting off and
  panes open at the desk's size, slid over, until you turn it on; this browser
  remembers which. **Fit panes as they open**, in the page's menu, is the same
  choice. A token that only looks has no button, as it can't resize a pane.
- To copy, hold a finger still on a word until it's selected, drag to select more,
  and press **Copy**. **Copy everything**, under `…`, copies all the pane holds.
- To open a web address, tap it. A sheet shows where it leads, in full, with **Open**
  and **Copy**. A tap never opens it: the words in a pane can say one thing and lead
  to another, so you see the real address first. Holding a finger on it selects it
  instead, and beside **Copy** you get **Open** with the name of the machine it leads
  to. With a mouse, an address is underlined when the pointer is over it. Hold `Ctrl`
  (`⌘` on a Mac) and click it, and it opens in a new tab. A token that only looks can
  open addresses too. Only `http` and `https` addresses open, whether the program
  wrote one or marked other words as leading to it.
- **Find**, under `…`, searches everything the pane holds.
- With a keyboard of keys, type as you would anywhere. `Ctrl`+`Shift`+`C` copies
  what you've selected with the mouse and `Ctrl`+`Shift`+`V` pastes (`⌘`+`C` and
  `⌘`+`V` on a Mac).
- When the machine is far, what you type is drawn at once, a little fainter, until
  the program shows it, as it is over `ssh`: see "Typing on a far machine" under
  [Other machines, over ssh](#other-machines-over-ssh). How far it is, is measured
  once, as you open the pane: 70 milliseconds there and back, or more, turns it on.

**On a wide screen**

At 900 pixels wide or more, the list is at the side, as the window's sidebar, and
beside it is a tab with its panes where its layout puts them, each one live.

- Click a pane to type in it. The button at the end of its header makes it fill
  the tab, and puts it back. The `…` beside it opens **Find**, which searches
  everything the pane holds, and **Copy everything**, which copies all of it.
- Click a row in the list to go to that pane's tab. Which tab you look at is your
  browser's own: it doesn't change what's in front at your desk.
- **Fit to this screen** takes the size of every pane in the tab, and **Give the
  sizes back** returns them. Leaving the tab gives them back too.
- **Needs you**, at the top of the list, shows the cards.
- **Everything** and **Activity**, under it, choose what the list shows: every project,
  tab and pane, or only the panes at work or with something for you, as on a phone.
  The number on **Activity** is how many panes need you.
- With a token that may arrange, drag the border between two panes to move it, as in
  the window. The window and every other browser follow as you drag, and **Escape**
  while you hold it puts it back. The cursor changes over a border only for a token
  that may arrange.
- Drag a pane by its header onto another pane of the tab. In the middle of it, the
  two swap places. Near a side of it (a quarter of its width or height, at most 160
  pixels), the pane you carry takes that half of it, and near the tab's own edge it
  goes at that side of everything. A word says which: "Swap", "Left half", "Left of
  everything". Let go outside, or press **Escape**, and nothing changes. A press on a
  header that doesn't move is a click, as before. With a finger, hold the header for
  a third of a second, then drag. A phone's layout has none of this.
- Nothing is split, closed or renamed from a browser.
- The signs programs print with a Nerd Font (a branch, a language) are drawn from an
  icon font the page brings with it, so you don't need one installed. A sign is drawn
  across two cells when the cell after it is empty, as it is in the window, and
  fitted into its own cell when a letter follows. At a desk, letters are drawn in the
  `font-family` and `font-size` your settings name (JetBrainsMono Nerd Font at 12
  points if they name none), so a pane is the size it is in the window. A browser on
  another machine that doesn't have that font uses the first of `JetBrains Mono`,
  `Cascadia Mono`, `SF Mono`, `Menlo`, `Roboto Mono`, `Consolas`, `Liberation Mono` and
  `DejaVu Sans Mono` that it has. A phone keeps its own, smaller letters, which you can
  change by pinching.
- **Text size**, in the menu under your letter, makes the letters of every pane in this
  browser smaller or larger: `−` and `+` change it by a pixel, between 9 and 28. Panes
  change at once, and one fitted to this screen is fitted again, so smaller letters give
  it more columns. The browser remembers the size; the window keeps its own, and so does
  every other browser. **Reset** appears once you've changed it and goes back to the
  window's size, or on a phone to the phone's own 14. Pinching on a phone changes the
  same size. A token that only looks is told so once, in a box at the foot of
  a phone's screen, with an **OK** a thumb reaches.
- The wheel scrolls to the pane's live end, also when the program in it printed more
  after the page opened.

**Add it to your home screen**

From the home screen the page opens full screen, like an app.

- **iPhone or iPad:** in Safari, tap the share button, then **Add to Home Screen**. The
  app pairs itself from the same code if you do this within fifteen minutes, on the same
  network: see "A one-device code also pairs that device's home-screen app" above.
- **Android:** in Chrome, tap `⋮`, then **Add to Home screen** or **Install app**.

A phone's page tells you this once, and you can put the reminder away. A guest isn't told.

**After you update insensical**, the page updates itself the next time you open it, in a
browser or from the home screen, and nothing needs clearing. A page you left open reloads
itself when you come back to it; if you're typing into a pane or have half an answer
written, it says a newer page is ready instead, and **Reload** takes you to it.

The server that serves the page, `isc web`, starts itself again as the installed
version the first time your daemon turns it away for being older. It keeps its port,
and the port of the guest door too, even if you started it with `--guest-port 0` and the
system chose it, so your paired devices and the Tailscale mapping carry on as they were. Until it has,
the page says **Not connected** and, in plain words, that insensical on this computer
was updated and that you can turn **Open in a browser or on your phone** off and on
again in its settings, which does the same by hand. A server started with
`--local-visits`, which `isc web` no longer takes, still starts: the flag is ignored.

The phone keeps no copy of the page: it's fetched from your server each time, so it
doesn't open while the server can't be reached.

On iOS 26 and later the system blurs the top edge of a home-screen app. The page asks for
an opaque status bar, so it begins under the bar and its header is clear of that edge. If
yours still looks blurred, the page's menu has **Header looks blurred** when it runs as a
home-screen app: it moves the header further down, and it's remembered in that browser.
Under it the menu shows what the phone says of its own screen, in small type.

**Notifications on your phone**

The page can tell your phone when a pane waits, fails or finishes while the page
isn't in front of you.

- Press the bell, or say yes when the page offers after your first answer. It
  never asks as it loads.
- It needs `https`. `tailscale serve` gives you that, and so does a certificate
  of your own. Over `ssh -L` the address is `http://127.0.0.1`, which a browser
  also allows.
- **An iPhone needs the page on its home screen first**, opened from there.
- What you're told is what the daemon would tell a window: the machine's name,
  the state, and the program's words. What you've muted or turned off stays
  quiet. A newer notification for a pane replaces the last one. Press one and
  the page opens at that pane's card. If the pane is on another of your machines,
  the page opens that machine and goes to its card there, never to this computer's
  pane of the same number.
- You aren't told while the page is open in front of you on that phone, or about
  something the window at your desk is already showing you.
- **Your other machines too**, for a device that has `--far`. A pane that waits, fails
  or finishes on a machine of `isc host ls` reaches that phone as this computer's do,
  with that machine's name first in the title. Each machine's own daemon decides what's
  worth telling you, by the same rule as here: ten panes that fail together there are
  one notification, and a program that pauses and carries on is none. `isc web` adds
  nothing to that.
- A device without `--far` hears of this computer only, and a guest hears of nothing.
- You're never told that a machine went out of reach or came back. The page says how
  each machine stands when you look.
- **It costs a standing link.** While a device with `--far` has notifications on,
  `isc web` keeps one `ssh` open to each machine of `isc host ls`, all the time, whether
  or not a browser is open. With nothing happening that's three small packets a second
  for each machine, about 15 KB a minute or 20 MB a day with their headers, and almost
  no processor. On a laptop on battery, or on a metered connection, that's what
  notifications from other machines cost you. It ends when the last such device turns
  notifications off or loses `--far`.
- A machine that can't be reached is tried again after a second, then two, four, and
  so on up to every five minutes. So after a machine comes back it can be five minutes
  before it's listened to again, and what a pane said in between isn't told later. One
  whose `ssh` would ask for a password is tried once and not again.
- Notifications travel through your browser maker's push service: Apple's,
  Google's or Mozilla's. They're encrypted for your browser before they leave your
  machine, so that service can't read them. It sees that something was sent, when,
  and how urgent it was.
- A browser says where its notifications go, and the server posts there. It refuses an
  address that isn't `https` on port 443, or whose name resolves to anything but public
  addresses (this machine, a private network, a link-local address). It looks the name up
  again each time it posts, and follows no redirect. A failure is said as the service's
  name and a plain reason, never its address.
- `isc web push ls` lists the browsers that get them, and marks the ones that hear your
  other machines. `isc web push test` sends a test to each and prints what each service
  answered. `isc web push test --at NAME` sends one as from that machine, to the
  browsers that hear of it.
- Turn them off under your letter, at the top right. **Forget this browser** and
  `isc web token rm` turn them off too.

**Yours**

Your letter, at the top right, opens what's this browser's own: the token it's
paired with and what that may do, notifications, a theme, whether panes take
your screen's size as you open them, and **Forget this browser**. The themes are
the eight that come with insensical. Left alone, the page follows your system:
`fjord` when it's dark and `paper` when it's light.

**When the link drops**

In a lift or a tunnel, the page keeps what it last showed, dimmed, and says
**Link to desk lost · trying again**, with your computer's name in place of `desk`. It says so two and a half seconds after it last heard
from your machine. Keys aren't sent while it's lost: the line jumps once to say so.
It reconnects by itself and says **Connected**.

Nothing you type blind reaches an agent:

- What you type arrives within two and a half seconds, or never. `isc web` throws
  away whatever reaches it from a page it hasn't heard from in that time, so
  nothing turns up at an agent when the network returns.
- Nothing arrives after the page has said the link is lost.
- Nothing is kept to send later. An answer you press on a card while the link is
  lost isn't sent. The card says **Not sent**, and it's still unanswered when the
  link is back: read the question again and answer it then. A line you wrote stays
  in its field.
- A key you pressed in the last moment before the link was lost may or may not
  have arrived. The page can't know, so the card stops saying **Sent**. When the
  link is back, the card is gone if your answer arrived and still there if it
  didn't.
- Letters that were drawn ahead of a far program are taken off the pane when the
  link is lost.
- You can still scroll back, select and copy in a pane while the link is lost.
- When you switch to another tab or lock your phone, the page stops sending. When
  you come back it's dimmed until it has heard from your machine again, which is a
  few milliseconds on a good link. It takes keys from then.
- A link that takes more than about a second and a half there and back is too slow
  to type over: the page says it's lost.
- A proxy in front has to pass WebSocket messages on as they come. One that holds
  them back makes the page say the link is lost.
- If the page says **Not connected · this page is older than its server: load it
  again**, your browser kept a page from an older `isc`. It reloads by itself once
  it has fetched the new one; reload it yourself if it doesn't.

**Addresses**

- `?pane=3` on the end of the address opens that pane straight away.
- `?card=3` opens at that pane's card.
- A pane that finished or failed is marked as seen when you look at it. An address
  alone doesn't count as looking: the page waits until you've pressed or typed
  something in it. A token that only looks never marks a pane as seen.

**Run it all the time**

On Linux a package installs a user unit for it, which nothing turns on:

```
systemctl --user enable --now insensical-web
```

**Limits**

- A device made without `--far` shows only the machine `isc web` runs on. Other machines
  you've named with `isc host add` are listed only for a device made with `--far`; without
  it, run `isc web` on each machine you want.
- From a browser you can't open, close, split, minimize or rename anything. Only a
  token with `arrange` moves or reorders panes, at a desk. There's no palette, overview or
  file finder, and Settings changes only what "Manage it from the page" lists.
- The keys under a card are always the same seven. The page never works out an
  agent's choices for you.
- A phone's own selection handles and magnifier don't appear over a pane. Hold a
  finger still to select instead.
- Pictures a program draws with the Kitty graphics protocol aren't shown in a
  browser.
- A press on a path in a pane does nothing there: in a browser it's a path on
  another machine.
- A browser that has none of the fonts above falls to its own monospace font, which
  can be larger or smaller than another browser's.
- The cursor doesn't blink, and prompts aren't jumped between.
- A very large pane is drawn less sharply on a phone, because a phone limits how
  big a drawing can be.
- Whether typing is drawn ahead is settled when a pane is opened and isn't looked at
  again while it stays open. A phone that moves to a slower network keeps what it
  had: open the pane again. There's no setting for it in a browser.
- Keys the browser keeps for itself never reach the pane: `Ctrl`+`W`, `Ctrl`+`T`,
  `Ctrl`+`N` and the like.
- No sound is played.
- A browser signed in through `--trust-proxy`, with no token, can't be sent
  notifications.
- Everyone who holds a token is the same device to the server: it doesn't tell people
  apart. To give someone less, share one project or one terminal instead.
- The encryption `isc web` does itself, with `--tls-cert`, needs a processor from
  about 2014 or later on x86-64, and on arm64 one with the AES and SHA-2 extensions,
  which a Raspberry Pi 4 lacks. On those it says so and you put a proxy in front instead.
  Notifications need the same of the processor.

### The whole interface in a terminal

`isc`, with nothing after it, runs insensical in the terminal you run it in, with no
window. Use it on your own machine when you'd rather stay in a terminal, on a machine
that has no desktop, and on one you reach over `ssh`:

```sh
ssh build-host -t isc
isc --host build
```

The first runs the interface on the other machine and sends its screen through `ssh`.
The second runs it here, on the other machine's daemon, and says so when that machine
goes out of reach: see [Other machines, over ssh](#other-machines-over-ssh).

- `isc tui` is the same thing said in full. It takes the options `isc` takes before a
  command: `--host` for another machine's daemon, `--socket` for a daemon at another
  socket, and `--rights` to ask for fewer rights.
- `isc --help` lists the commands. So does `isc` alone where there's no terminal to
  draw in (a script, a pipe), and inside one of insensical's own panes, where the
  interface would show itself.

It's another client of the same daemon. What you do in it shows in a window at once, and
the other way round. Leaving it ends nothing.

**It needs a terminal to draw in**: with its input or its output piped it says so and
stops. It writes in as many colours as your terminal shows, which `colours = "auto"`
works out from `COLORTERM` and `TERM`: millions where `COLORTERM` says `truecolor` or
`24bit` or `TERM` names Ghostty, kitty, Alacritty, foot, WezTerm or Contour, 256 where
`TERM` says 256, sixteen for `linux`, `dumb` and `vt100`. Over `ssh` `COLORTERM` is
often not passed on; if the colours look coarse, set `colours = "truecolor"` under
`[tui]`. It works at any size down to 20 columns by 6 rows. Under 76 columns the list
of projects is left out, and under 56 the top row loses its name.

It's a terminal program and looks like one.

- **The top row** has the tabs of the project in front, each with its number, after
  the name `insensical` and, on another machine, that machine's name. The one in front
  is a pill, and a `+` after the last opens a tab. At the right end are the counts of
  what finished and what needs you, and the leader's key. Press the counts to go to
  the next pane that needs you.
- **The list at the left** is your projects, one to a row, each with the state of
  what runs in it or how many panes it has. Under 76 columns it's left out.
- **The list can show only activity.** `Ctrl+Space` then `Y` (`activity`) turns the list
  at the left into the panes that are at work or need you, under their projects' names,
  and the same key turns it back. It's the list the window has, kept quiet by the same
  rules: see "Showing only activity" under [The sidebar](#the-sidebar).
  - A row is two lines. The first has the program's icon, the pane's name and, at the
    end, how long it has been as it is (`now`, `2m`, `1h`, `3d`). The second has its
    state in words and what its program says, as `waiting · Allow the edit?`. A name or
    words that don't fit are cut with `…`.
  - The row of the pane in front is filled, with a bar at its edge. A row that's about
    to go, and one that finished more than a day ago, is dimmed.
  - `Ctrl+Space` then `}` and `{` walk the rows (`next-activity`, `previous-activity`),
    round at the ends, whether the list is on or not. While it's on, `Ctrl+Space` then
    `A` goes down the rows to the next pane that needs you or has finished.
  - A click on a row goes to its pane, and a click on a project's name to the project.
  - No row appears, leaves or changes place while your pointer is over the list and has
    moved in the last 2 seconds, or for 2 seconds after you walk it with a key.
  - With nothing to list it says "Nothing is at work or needs you." and the key that
    brings everything back.
  - With more rows than fit, the last line says how many are out of sight, and the row
    of the pane in front is always shown.
  - The list is as wide as the list of projects, 26 columns, so turning it on and off
    resizes no pane. If the list at the left was hidden, turning activity on shows it.
  - A notice that only says what a listed pane's state is isn't shown while the list
    is: its row says it.
  - Under 76 columns there is no list, so there's nothing to narrow. The two keys still
    walk the panes that would be listed, and `A` goes to the most urgent pane first.
  - It shows one machine's panes, the one this interface is on.
- **Each pane is in a box.** Its name is in the top, and its state at the end of the
  top: in words as the window's rows say it, with the program's own words cut to the
  room there is, and then how long the pane has needed you (`now`, `2m`, `1h`,
  `3d`, bold after ten minutes). The focused pane's box is in the theme's accent colour, and a waiting one in its
  waiting colour: your terminal's blue and yellow under the `terminal` theme.
  Panes aren't listed anywhere unless the list shows only activity: they're on the
  screen.
- **The line at the foot** says what your keys are doing (`PANE` when they go to the
  program, `MENU` after the leader, `LIST` in a list, `LOCKED`), where the focused pane
  is, and the keys you'll want most.

**It needs a Nerd Font**, since every icon in it is one. Ghostty, kitty, WezTerm and
insensical's own panes carry those icons themselves, so there it works with any font.
In another terminal, set its font to a Nerd Font, or the icons show as empty boxes.

A Nerd Font's icons are wider than one cell, so each is given the cell after it too.
A terminal that keeps every character inside its own cell cuts them off at the right;
the "Mono" variant of a Nerd Font (`JetBrainsMono Nerd Font Mono`, say) has icons one
cell wide and shows them whole there.

**Keys.** Commands follow the leader, `Ctrl+Space`, and the keys after it are the ones of
[the leader keymap](#the-leader-keymap): `Ctrl+Space` then `T` opens a tab, `Shift+S` and `S`
split right and down, `H` `J` `K` `L` move between panes. While the leader waits, a panel at the foot
lists every key that can follow. It stays until you press one, however long you take;
`Esc` closes it. The line at the foot of the screen names the keys for the palette and
for the list of keys, so you don't have to remember the leader.

- **`Ctrl+Space` then `D` leaves.** The terminal is given back as it was.
- `Ctrl+Space` then `Space` is the palette, and `Ctrl+Space` then `Shift+/` lists every
  key, under the same headings as the window's sheet.
- [The palette](#the-palette) lists what the window's lists, under the same headings
  and in the same order, and typing finds things the same way. A few things differ here. Its themes have one
  more, `terminal`: see Themes below.
  It lists copy, paste, select-all and find, which act on the pane once the palette has
  closed. A command is also found by its name in the settings, such as `split-right`.
  A setting's row doesn't show its value. And **Recent** holds the last five commands
  you chose, not settings, and only until you leave.
- A terminal has a few keys of its own after the leader, for what a window does with
  chords: `C` copies, `Shift+V` pastes, `Ctrl+F` finds text, `Ctrl+A` selects
  everything, `Shift+Up` and `Shift+Down` go to the command before and after,
  `PageUp` and `PageDown` scroll, `Shift+X` gives every key to the program.
- `Shift+PageUp` and `Shift+PageDown` scroll without the leader.
- **The keys are the window's.** Your `keymap`, `leader`, `[keys]`, `[keys.leader]` and
  `[keys.direct]` are read as the window reads them.
- **The leader always works here, whichever keymap you use**, because the terminal you
  run it in may keep a chord for itself: `Ctrl+Shift+T` opens Ghostty's own tab before
  insensical ever sees it, and an older terminal can't tell `Ctrl+Shift+T` from
  `Ctrl+T`.
- **Your chords work too**, wherever the terminal tells keys apart (Ghostty, kitty,
  WezTerm, foot and others that speak the Kitty keyboard protocol) and doesn't take
  the chord first: copy, paste, find and the like in either keymap, and the rest with
  `keymap = "direct"`. To free a chord, unbind it in that terminal.
- **A leader your terminal can't report isn't used.** If `leader` is a chord with
  `Ctrl+Shift` or `Cmd` and the terminal doesn't tell keys apart, the leader here is
  `Ctrl+Space`, and a notice says so.
- **While the keys are locked, there are three ways to have them back.** Press the
  leader and then the key that locks, `Ctrl+Space` then `Shift+X`; or click `LOCKED` at
  the foot of the screen; or press the chord that locks, where your terminal tells it
  apart. The line at the foot names the keys, as you've bound them. `Ctrl+Alt+Shift+L`
  is that chord by default.
- While they're locked, the leader goes to the program together with the key you press
  after it, not before.
- `Ctrl+Z` goes to the program in the pane. To stop the interface itself, send it
  `SIGTSTP` (`kill -TSTP`): it gives the terminal back while it's stopped, and draws
  everything again when you bring it back with `fg`.
- **Closing a tab, closing a project and putting a project to sleep ask first**, with
  the question the window asks, and only when something would end.
- **A new project asks one thing**: a directory to work in, or only a name. A
  directory is written from `/` or from `~`, which is your home on the daemon's
  machine, and `Tab` goes into the one marked. A directory that isn't there on this
  computer, or on the machine the project is for, is made there after one more Enter.
- **A project is renamed and closed from the palette**: "Rename this project" and
  "Close this project and everything in it". They're the commands `rename-project` and
  `close-project`, which have no default key.
- **A project is put higher or lower in the list from the palette**: "Move this
  project up the list" and "Move this project down the list", the commands
  `move-project-up` and `move-project-down`. Or drag its row with the mouse. The order
  is the daemon's, so every window shows it too.
- **A pane goes to another tab from the palette**: "Move this pane to another tab",
  the command `move-to-tab`. It lists every other tab, those of the pane's own project
  first, each with its number, its name, its project and how many panes it has. `Enter`
  puts the pane at the right of that tab's panes and takes you there with it. With no
  other tab it says so.
- **A project's icon is chosen from the palette**: "Choose this project's icon", the
  command `project-icon`. It lists the colours across the top and the marks below, and
  typing narrows the marks: `git`, `rust`, `docker`. `Up` and `Down` move through the
  marks, `Left` and `Right` through the colours (the first, "by name", is the one that
  comes of the project's name), and the project's row shows the choice as you move.
  `Enter` sets it, as `isc project icon` would, and every window shows it. `Escape`
  leaves it as it was. The first mark is the folder, the project's own sign here, the
  letter in the window. A project that has a drawing of yours shows "Keep your drawing"
  first: it's kept, whatever colour you pick, until you pick a mark or the folder.
  Only marks a Nerd Font has a sign for are listed; the window has many more, and a mark
  it chose that isn't listed shows here as the folder.
- Those four commands, and `go-to-machine` (see Lists and sheets below), exist here
  only and have no default key. Give them one under `[keys.leader]` if you use them
  often. `scroll-up` and `scroll-down`, which `PageUp` and `PageDown` run after the
  leader, exist here only too.
- If you unbind the key that leaves, `Ctrl+Space` then `Shift+D`, `Shift+Q` or `Ctrl+D`
  (the first of them nothing else uses) leaves, or else `D`, so there is always a way
  out.
- **A project that's asleep** says so where its panes would be, and `Enter` wakes it.
- **If the server stops** while you're in it (it crashed, or `isc kill-server` was run),
  a card over the screen says "The server has gone away". The list of projects, the
  tabs and the panes' boxes stay, dimmed, and the panes are empty, since nothing in
  them can be vouched for now. What you type reaches nothing. You don't have to leave: it
  looks for a server once a second (a key you press looks at once), and when one runs on that socket again (start it with any `isc`
  command, or with `isc daemon`) the card goes and the panes come back, as they were
  if the server kept them (see [What survives what](#what-survives-what)).
  Nothing is started for you: the server may have been stopped on purpose.
  For a machine you reached over `ssh`, the card says why it is out of reach and what to
  try, and any key tries again.
- **If the server becomes another version** while you're in it (after an update and a
  restart), it says so: leave, and run `isc` again.
- **If you choose to stop everything from the tray** while this is the interface you
  used last, it asks you here what the window would ask, and stops the server if you
  say so.

**Lists and sheets** open over the panes and take your keys until they close, and the
line at the foot says `LIST`. `Esc` closes any of them. Typing in one that has a field
goes into the field, and so does a paste.

- **The palette** moves with `Up`, `Down` and `Tab`, a page at a time with `PageUp` and
  `PageDown`, and `Enter` chooses. A theme is set when you choose it; it isn't shown
  first, as the window shows it.
- **Every tab** is `overview`, `Ctrl+Space` then `W`. Each tab of every project is a
  card, drawn small as it is laid out, with what a pane that needs you asks. The arrows
  or `H` `J` `K` `L` move, `Enter` goes to the tab, and `Tab` narrows the cards to tabs
  with a pane that needs you, and back.
- **Find a file** is `pick-path`, `Ctrl+Space` then `/`. Type a few letters of a file
  or directory under the pane's directory. `Up` and `Down` move, `Enter` types the path
  into the pane, `Tab` goes into the directory marked, and `Backspace` on an empty
  field comes out of it.
- **Find text** is `Ctrl+Space` then `Ctrl+F`. A field lies over a corner of the pane
  and the match is selected, so `C` copies it. It says which match of how many. `Enter`
  or `Up` goes to the match before, `Shift+Enter` or `Down` to the one after, and
  `Esc` closes.
- **Every key** is `Ctrl+Space` then `Shift+/`. Type to narrow it. `Up`, `Down`,
  `PageUp` and `PageDown` scroll, and `Esc` or `Enter` closes it.
- **The settings sheet** is `Ctrl+Space` then `,`. `Up` and `Down` (or `K` and `J`)
  move, `Tab` goes to the next heading, `Left` and `Right` (or `H` and `L`), `Enter`
  and `Space` change the value one step, and `Backspace` puts it back to its default.
  A change is written to the file at once.
- **Go to a machine** lists this computer and each machine under `[hosts]`. Each is
  tried as the list opens and says `reached`, with its version, or `not reached`, or
  `ssh asks`, or `no server yet` for this computer; the one on show says `shown now`.
  `Up`, `Down`, `K`, `J` and `Tab` move and `Enter` goes. The machines you leave carry on.
- **The restart question** is `Ctrl+Space` then `Shift+U`. It asks the server whether it
  can hand its panes to the version installed. If it can, `Enter` restarts it and every
  pane carries on. If it can't, the card says what to run and `Enter` only closes it.
  For a server on another machine it names `isc host restart`.
- **A question** names its keys in its buttons: `Enter` for the first and `Esc` for the
  other. A card that only says something is happening, such as "Stopping the server",
  has no keys.

**The dock** is the row under the panes. Each pane you've minimized in the tab is a pill
there with its state, and `+3` stands for three that didn't fit.

**The mouse** goes to the pane you press, selects text, scrolls, and drags the edge
where two panes meet. Press a tab, a project or a pill in the dock to go there, the `+`
after the tabs to open a tab, and the counts at the top right to go to the next pane
that needs you. Press a notice to go to its pane and drop it, or its cross to drop it
alone. Drag a project's row over another's and it takes that place in the list.

- A floating pane is carried by the top of its box. Drag the bottom of its box to make
  it taller or shorter, the right side to make it wider or narrower, and the corner
  where they meet for both.
- The tab under the pointer shows a cross at its end. Press it to close the tab. The
  middle button on a tab closes it too.
- The middle button in a pane pastes what you last selected here into that pane. If
  you've selected nothing here, or your terminal has said since that it lost the
  keyboard, it asks your terminal for its own selection.
- Hold `Shift` and the mouse is your terminal's own again, to select across the whole
  screen.
- A web address in a pane, printed or marked by a program (OSC 8), is passed to your
  terminal as a link, and your terminal opens it its own way. Only `http://` and
  `https://` addresses are passed on. Words a program marked as leading anywhere else,
  a file included, are shown as plain words. Paths aren't opened here.

**A pane that a window also shows** is the size that fits whichever you used last, as
with [two windows](#two-windows). When that's the window, the pane says "sized by another
window" at the bottom of its box, and you see its foot, where the cursor is.

- Press a key or a mouse button here and the panes take this terminal's size again.
  You don't have to do anything else.
- Pressing those words does the same, and so does "Size the panes for this window, over
  any other that shows them" in the palette, the command `lead`.
- Nothing but your own key or press takes the size back, so a window and a terminal
  that are both open don't take it from each other.

**Copying and pasting** use your terminal's clipboard, so they work across `ssh`.

- The leader and then `v` selects with the keyboard, with the same keys as in the
  window: see [Selecting with the keyboard](#selecting-with-the-keyboard). The strip
  is the pane's last row, what it copies goes to your terminal's clipboard, and a
  notice says "Copied" and how many lines.

- The copy command copies what's selected to the clipboard. It's sent as OSC 52, which
  a terminal can be set to refuse, and which Terminal on a Mac doesn't take at all.
- Selecting alone sends your terminal nothing. What you selected is kept here for the
  middle button. Some terminals have no selection clipboard and would put it on the
  clipboard instead, replacing what you'd copied.
- Pasting with your terminal's own key always works. The paste command asks the
  terminal for its clipboard, which many terminals refuse. If yours doesn't answer
  within a second, a notice says so, once: use the terminal's own key. An answer that
  comes later than that, or that wasn't asked for, is dropped.
- A paste is held and asked about for the same reasons as in the window, and the
  question shows the same lines. See [Pasting](#pasting).
- **A program can put text on your terminal's clipboard only while you're known to be
  here.** That's while your terminal says it has the keyboard, or for a minute after
  you last pressed a key or a mouse button here. A terminal reports the keyboard only
  when it gains or loses it, and some never do, so one left open in a window you
  aren't looking at doesn't count. A program that copies after you've been idle for
  more than a minute, in a terminal that hasn't reported, is ignored: press a key and
  have it copy again.

**Pictures** a program draws with the Kitty graphics protocol are shown where your
terminal shows them itself: Ghostty, kitty, WezTerm and others that answer when asked.
See [Pictures](#pictures) for what a program can draw.

- Your terminal is asked once, as the interface starts. One that doesn't answer is
  sent no picture, and the cells are blank.
- A picture is sent to your terminal once and then placed by its number, so moving
  it, covering it and uncovering it cost nothing. A large one takes as long to
  appear as it takes to send, which over `ssh` can be seconds.
- When its pane leaves the screen (another tab, or another pane filling the tab), your
  terminal is told to drop it, and it's sent again when the pane is back.
- It's cut at its pane's edge and at the top when it scrolls away, as in the window.
- **A picture is shown whole or not at all.** While a list, a question, a notice, the
  leader's panel or a floating pane lies over any part of it, it's taken away, and it
  comes back when that's gone.
- A picture the program sized in cells fills those cells. One it sized in dots is
  shown dot for dot, when this terminal sized the pane; when a window did, it's
  stretched to the cells it has there.
- Inside `tmux` or `screen` there are no pictures: they don't pass the question on.

**Notifications.** While you're in it, a pane that needs you says so in a notice at the
foot of the list of projects, or at the top right of the panes when that list isn't
shown. On a machine with no desktop, the notification is also sent to your
terminal, which shows it as its own if it knows how (OSC 777), with a bell.

- What a program asks you to be told, with `isc notify` or its own notification, stays
  for six seconds when its pane isn't waiting for you. A pane that waits or failed
  keeps its notice until you've gone to it.
- When the daemon refuses something you asked for, a notice headed "It could not be
  done" says why for six seconds. Only what you asked for is reported: a refusal of
  something the interface did by itself, such as sizing a pane that has just closed,
  isn't shown.
- When a program in any pane of the tab in front rings its bell, your terminal rings.
  A pane or a project you've silenced doesn't ring.
- Your terminal is rung at most once in 300 ms, whether for a pane's bell or with a
  notification. It's sent at most five notifications at once and two a second after
  that, for all panes together, with what's held back sent later and titles and text
  cut to 120 and 600 characters, as [section 12](#12-notifications) says of the
  application. The notices inside the interface aren't held back.

**Themes.** It uses your insensical theme, the same one as the window: the light or
the dark one of a pair, by your terminal's own background. One more theme exists here
only, `terminal`, which is your terminal's own colours: its background, its text colour
and its sixteen colours, so it looks like everything else you run there.

Choose one in the palette: type `theme` and it lists them. A theme chosen here is the
window's too, as one chosen in the window is this interface's. Two things are written
under `[tui]` instead, for a terminal alone: `terminal`, and any theme once `[tui]`
already names one.

```toml
[tui]
theme = "terminal"
```

- Under `terminal` the panes stand on your terminal's background, which is never
  painted over, so a transparent or pictured background stays as it is.
- Under any other theme your terminal's background is set to the theme's while it
  runs, so the strip a terminal leaves round its cells isn't another colour. It's put
  back when you leave. A terminal that doesn't tell its colours is left alone.
- Put `theme = "terminal"` under `[tui]`, not at the top of the file: the window has no
  terminal to take colours from.

**Settings.** It reads the window's own settings for the theme, the keys and motion,
and there is no second set. Notifications, sounds, scrollback and the rest of what the
daemon does are the daemon's, so they're the same whichever interface you use. A
`[tui]` table in `config.toml` says otherwise for a terminal alone about `theme`,
`activity-idle`, `interface-style`, `motion`, `attention-glow`, `type-ahead`, `clipboard`, `keymap` and
`leader`, and holds the two settings that only a terminal has:

```toml
[tui]
theme = "terminal"     # or any of those seven, to differ here
colours = "auto"       # "truecolor", "256" or "16"
mouse = true
```

- The window never reads `[tui]`, whatever is in it.
- Of the settings, only the eleven named above are read from `[tui]` here. Another
  setting's name written there, `scrollback` or `density` say, is a name insensical
  knows, so no notice is given, and it changes nothing. A name that is no setting,
  or `[keys]` under `[tui]`, is said in a notice headed Settings.
- `colours` and `mouse` are read from `[tui]` only. At the top of the file they're a
  key the window doesn't know, and this interface tells you where they go: "is written
  under [tui]".
- Every one of them applies within a second of saving, `mouse` included: turn it off
  and your terminal has its mouse back at once.
- A setting the daemon reads, such as `notifications`, changes nothing under `[tui]`:
  the daemon reads the top of the file.
- `[keys]` and its tables are read from the top of the file only.
- A line this interface can't use is said in a notice headed Settings, and only that
  setting falls back to its default: a name it doesn't know, a value of the wrong
  type, one key in `[keys]` that isn't given a command. The rest of the file still
  applies here, even while the window sets the whole file aside for that line. Broken
  TOML leaves every setting at its default here too.

The settings sheet lists every setting that takes effect here and can be stepped:
`activity-idle`, `interface-style`, `motion`, `attention-glow`, `keymap`, `notifications` and the three
`notify-` switches, `long-command`, `sound`, `notify-command-when`, `notify-url-when`,
`notify-url-body`, `tray`, `scrollback`, `term`, `type-ahead`, `clipboard` and
`keep-scrollback`, `tmux-shim`, and under "This terminal", `colours` and `mouse`. It shows a value
from the `[tui]` table where that table has one, and changes it there. A theme is
chosen in the palette. `leader`, `shell`, `notify-command`, `notify-url` and your own
keys are set in the file. The window's density, fonts
and sidebar settings have no meaning here and aren't listed.

**What it doesn't do.**

- A picture that anything lies over isn't shown until it's uncovered, and none is
  shown in a terminal that doesn't answer the question about pictures.
- A pane has no buttons to press. Closing, zooming, floating and putting away are
  commands. Double-click a pane's name to fill the tab with it.
- A pane isn't dragged into another place with the mouse, within its tab or to
  another. The move commands do the first and `move-to-tab` the second. `isc put`
  sets a pane at a chosen side of another.
- A drawing of your own (an SVG) can't be chosen here, and isn't drawn: the project's row
  shows its folder. The question about agents isn't asked: the
  `integrate` command opens `isc integrate` in a floating pane. With another machine
  shown, it asks once and has that machine set up every agent it finds.
- One machine is shown at a time. "Go to a machine" in the palette changes which.
- Sharing a project or a terminal, opening and saving a project file, and "Add a
  machine" are the window's, and aren't in the palette here. `isc host add` names a
  machine.
- Nothing fades in.
- On a small screen a question, a list or the settings sheet is cut to fit: text is
  shortened and a long list is counted ("and 4 more"). On a narrow one a question's
  answers stand one above the other, and on a very narrow one they're cut short and
  lose the names of their keys; `Enter` and `Esc` still answer. Under twenty columns or
  six rows nothing is drawn but "Needs more room".
- A floating pane is no smaller than twelve columns by four rows.
- A curly, dotted, dashed or double underline is a plain one, unless your terminal is
  known to show every colour or `colours = "truecolor"` says it does.
- A key written as a sign (`?`, `<`) works wherever that sign is on your keyboard. One
  written with `Ctrl` or `Alt` and a sign (`ctrl-}`) is found by where the key is on a
  US keyboard.
- Of the links a program marks, a row of a pane keeps 64 and the screen 1024. A link
  beyond them isn't passed to your terminal; its words stay.
- When you copy, or let go of the button after selecting, the pane has 300 ms to say
  what's selected. One too busy to answer by then copies nothing: copy again.
- If a paste's end never arrives from your terminal, what arrived is pasted after five
  seconds, and your keys work again.
- A pane whose connection to the server fails is attached again by itself, a quarter
  of a second later at first and then less often, up to every eight seconds.
- It can't be run inside one of insensical's own panes on the same daemon, where it
  would show itself.

### Other commands

- `isc config` prints a default settings file. `isc completions SHELL` prints
  completions for `bash`, `zsh` or `fish`. Neither touches the
  daemon. `just install-local`, `just install-mac` and the packages install the
  completions for bash, zsh and fish. On a Mac they're also inside the application, in
  `Contents/Resources/completions`. To put them somewhere yourself:

  ```sh
  isc completions fish > ~/.config/fish/completions/isc.fish
  isc completions zsh > ~/.local/share/zsh/site-functions/_isc    # any directory in $fpath
  isc completions bash > ~/.local/share/bash-completion/completions/isc
  ```

  The output is a few lines that make the shell ask `isc` each time you press Tab, so
  the suggestions match the session at that moment:

  - commands and their options, with what each does;
  - a pane by its number wherever a command takes one, with what it runs, its state
    if it has one, and its project. The pane you're typing the command in is left out,
    because it's the one used when you don't name one;
  - a tab by its number, with its name or what its panes run;
  - a project by its name, after `--project`. A project whose name a shell wouldn't
    read as one plain word (a space, `$`, `*`, `;` and so on), whose name is a number,
    or whose name another project has, is offered by its number, with the name shown
    beside it;
  - the states `isc status` takes and the sides `isc focus` takes.

  Asking never starts the daemon. If none is running, or the one that is doesn't
  answer within a third of a second, you still get commands and options, but not
  panes, tabs or projects. It asks the daemon at the default socket or
  `$INSENSICAL_SOCKET`, not one given with `--socket` on the line you're typing.
  Completions saved from a version before this one list commands only, so you have to
  print them again.

  The lines set `ISC_COMPLETE` when they ask. Completions printed by an earlier
  version set `COMPLETE`, and still work: `isc` answers that only when it's asked the
  way a shell asks, so `COMPLETE` set for some other reason, in an agent's environment
  or a build's, doesn't change what `isc` does.

- `isc version` prints this version and the running daemon's, and whether they can
  talk to each other.
- `isc kill-server` stops the daemon and every program it runs, and returns once the
  daemon is gone.
- `isc restart-server [--yes]` moves the daemon to this version with everything still
  running. If no daemon is running, it starts one. If you answer no, everything stays
  as it is and the command exits with status 1. If the daemon is in the middle of
  running `notify-command` or posting to `notify-url`, it lets that finish first,
  which takes ten seconds at the most.

---

## 17. Stopping, restarting and new versions

### What survives what

| What happens | Programs | Screens and scrollback | Layout, directories, names |
|---|---|---|---|
| The window is closed | keep running | kept | kept |
| The window is opened again | — | shown as they are | — |
| The daemon moves to a new version (restart from the window, `isc restart-server`) | keep running | kept | kept |
| The daemon is stopped (`isc kill-server`, a logout that stops your user manager, reboot) | **end** | kept, as text and colour | kept |
| The daemon dies with no warning (crash, power cut, `kill -9`) | **end** | **lost** | kept |

### After the daemon has been stopped

The layout is saved in a file. It holds every project with its tabs, splits and their
shares, floats and their rectangles, minimized panes and where they return to, focus,
each pane's directory, and names. The file is rewritten whenever the layout changes.

When the daemon starts, it rebuilds all of it.

- **How panes were arranged comes back as it was.** A tab that a pane filled is still
  zoomed. A floating pane floats where it did, still pinned on top or shown over every
  tab if it was. A minimized pane is still in the dock. A muted pane or project is still
  muted, and a project that was asleep is still asleep.

- **Each pane gets a fresh shell**, in the pane's directory. If that directory no
  longer exists, the shell starts in the project's root, or failing that, your home
  directory.
- A pane that was running `nvim` or an agent comes back as a shell. **Nothing is run
  again automatically.**
- Every pane of every project starts at once.
- None of a program's environment is kept. Anything you run from the prompt runs in the
  new shell's environment.

**Each pane comes back showing what it showed before**, history included, at the same
size. Below that is a dim line, "── from before the server stopped ──", and then the new
shell's prompt.

- It's what was printed, redrawn. You can scroll, search and copy it, but the programs
  that printed it are gone.
- Its lines keep the width they had. A long line that was wrapped stays in its pieces
  when you make the pane wider, where what's printed since is wrapped again.
- A program that had taken over the screen, such as an editor or `htop`, leaves what
  was under it, the same as when it quits.
- Nothing a program had switched on comes back with it: not mouse reporting, not its
  way of reading keys, not line-drawing characters or margins. The new shell starts in
  a terminal as it is when new.
- It's written when the daemon is told to stop, and when a pane's program ends by
  itself. **It isn't written while things are running**, so a crash or a power cut keeps
  the layout but loses what the panes showed.
- A pane you closed keeps nothing.

**What panes showed is on disk.** It's in `~/.local/state/insensical/screens`, readable
only by you. **Anything a program printed is in there, secrets included.**
With `keep-scrollback = false`, nothing is written, and files already there are removed
as their panes close.

**What ran in a pane is offered, not run.**

- A pane started with a command, as in `isc run -- npm start`, comes back as a shell
  with that command typed at the prompt. Press Enter to run it, or edit or delete it
  like anything else you've typed. It appears a moment after the prompt does.
- A word of it that a shell would read as more than a word is typed between quotes, so
  `--flag=value` comes back as `'--flag=value'`. bash, zsh and fish read it the same.
- A command that holds a line end or another control character isn't offered.
- A command you typed inside a shell isn't remembered. Only the command the pane was
  started with is.
- **An agent's session is offered the same way.** With the Claude Code hooks installed,
  a pane where Claude was last working comes back with `claude --resume <session>` at
  the prompt. It's offered until you run something else in that pane, so it also
  appears for a session that you quit normally. Codex, opencode and pi are offered
  their own resume commands (section 11). Gemini CLI isn't.
- An agent's command comes before the pane's own.

**When a machine shuts down**, programs often die a moment before the daemon is told to
stop. The daemon doesn't record that as panes being closed, so the layout comes back
whole. A pane's removal is written two seconds late, and a termination signal freezes
the file as it is.

**A layout file that can't be read is set aside**, and the daemon starts empty. That's
a file from a version this one can't read, one that's damaged, or one holding a pane,
tab or project number too large ever to have been given. It's renamed
`state.json.unreadable`, beside where it was, so the next save doesn't write over it.
A layout file that holds more text than the server can send to a window (4 MB of names,
commands and directories) is set aside the same way. One such file is kept: a later one takes its place. To go back to it, stop the server
(`isc kill-server`), mend the file, and put it back as `state.json`.

### A new version installed underneath

**Installing replaces files and nothing else.** The running daemon keeps running what
it's running, and the open window keeps working.

The window tells you in the top bar: "insensical was updated. Restart to use the new
version." `isc version` and `isc ls` say the same.

To move to the new version, do any of these:

- press the notice;
- press `Ctrl+Alt+Shift+R` (leader `U`);
- choose "Restart insensical" in the palette;
- close the application and open it again;
- run `isc restart-server` in a terminal.

Each one asks first, and tells you that everything running carries on. Press Enter to
restart, or Escape to leave everything as it is. With no terminal to ask on,
`isc restart-server` does nothing unless you pass `--yes`.

**Restarting ends nothing.** The daemon becomes the installed version in place. It's the
same process, holding the same terminals.

- Programs keep running and never notice.
- Every screen and all the scrollback is still there, including anything a program
  printed during the move.
- The window shows "Restarting" for a moment, then shows everything as it was.
- `isc version` names the same process before and after.

**The first move to a version that can do this still ends everything.** A daemon from
before this was possible can't make way, so it's stopped and the new one is started. The
prompt then lists the panes that are running something, counts the shells waiting at a
prompt, and tells you that projects, tabs and panes come back. After that, updates end
nothing.

Other things to know:

- **Opened after an update with the old daemon still running**, the application opens a
  window that says so. Press Enter to restart the daemon and carry on. Press Escape to
  leave the daemon running and close the window.
- **If the application itself is from before the update**, the installed application
  starts in place of the running one. This happens on a Mac, with a window open, when
  you move the server from a terminal.
- **If the server is already newer than the application**, the window says so: "The
  server is already the new version, and this window is from before it." This happens
  on a Mac when you move the server from a terminal while the old application is still
  running with no window. The server is left alone whatever you press. Where the
  application's own file was replaced by the update, the new application opens by
  itself, or on Enter. Otherwise, close the window and open the insensical you
  installed.
- **If the daemon cannot make way**, because the installed file is gone or is a version
  that can't take over what this one runs, nothing is stopped. The window says "The
  server did not move to the new version", that it's still running everything, and
  why. Press Enter to go back to it. `isc restart-server` prints the reason and exits
  with status 1.
- **If the new daemon fails once it has taken over**, the programs end, as they do when
  any daemon stops, and the layout comes back on the next start. Before the old daemon
  lets go, the new version is asked whether it can read what it's being given, so this
  takes a fault and not only a difference between versions.
- **The notice also appears when you rebuild or reinstall from source.** The
  application and the daemon each remember the file they were started from and notice
  when it's been replaced. That's correct, not a fault.

What isn't carried across a move:

- Desktop notifications already showing stay until they're dismissed. The new daemon
  can't withdraw them.
- A command that was already running is timed from the move, so it may not be reported
  as a long command when it ends.
- Windows, `isc attach`, `isc events` and `isc wait` lose their connection. Windows
  reconnect by themselves. The commands end, and you need to run them again.
- When the new version has a different terminal engine, screens are carried across by
  redrawing them, not exactly. Text, colours, history and cursor are kept. A colour a
  program had redefined, or an escape sequence it was halfway through writing, isn't.
  A program that redraws itself, such as an editor or an agent, fixes that the next
  time it draws.

**Restarting is not the way to get a fresh daemon**, for instance when the daemon was
started with the wrong environment. It's the same process afterwards, with the same
environment. For a fresh one, run `isc kill-server`, which ends every program, and then
run any command or open the window, which starts a new one.

### When the daemon goes away under an open window

- The window says "The server has stopped" and that the layout comes back when it
  starts.
- If the daemon is moving to a new version because something else asked it to, the
  window says "The server is moving to a new version" and finds it again by itself.
- If the daemon cut the window off for falling behind (it left 8 MiB unread, which a
  stopped or frozen window does), the window connects again by itself and says "This
  window fell behind the server and was cut off. It has connected again." A window
  that had stopped reading altogether can't be told why, and says "The server has
  stopped" for the second it takes to find the daemon again.
- If you ended the window's connection from another client, it says "This window was
  disconnected" and waits for Enter.
- The interface in a terminal and a page in a browser say the same things in a line of
  their own.
- It looks for a daemon once a second and carries on by itself when one appears. For
  instance, after you run `isc restart-server` in a terminal, it shows the same
  programs still running.
- Press Enter to start one.
- If the daemon that appears is a different version from the window, the window starts
  the installed application in its place. If it can't, it tells you to close and
  reopen.

### Different versions of `isc`, window and daemon

- They need the same protocol and the same terminal engine to work together. A
  mismatch is refused with a message that names `isc restart-server`. Two releases
  with the same protocol and engine work together, whatever their version numbers.
- With `--host`, the other machine's `isc` has to match this one's in the same way.
  If it doesn't, the message names both machines' versions.
- A few things work regardless: `isc version`, `isc status`, `isc notify`, the agents'
  hooks, `isc kill-server`, `isc restart-server`.
- A daemon too old to answer those, or one that's stuck, gets three seconds and is
  then stopped with a signal. On another machine it gets twenty, and can't be
  signalled from here.

---

## 18. When something goes wrong

### Starting, stopping and versions

| What is seen | Why | What to do |
|---|---|---|
| "The insensical server cannot be reached" | No daemon is running and none could be started: `isc` isn't beside the application or on `PATH`, or the socket's directory isn't private | Put `isc` beside `insensical`; check the directory's permissions |
| "insensical was updated", and that the server still running is from before, at launch | A new version was installed under a running daemon | Press Enter to restart it. Everything keeps running; if the daemon is too old to make way, programs end and the layout comes back. Or press Escape to leave it |
| "The server is already the new version, and this window is from before it", at launch | The server was moved to a new version while an older application was still running | Press Enter to open the new application, or close the window and open insensical again. Nothing that's running ends |
| `isc: protocol N does not match the running daemon's M` | The same, from the command line | Run `isc restart-server` |
| "The server has stopped" in the window | The daemon was stopped or crashed | Press Enter to start it; the layout comes back |
| "This window was disconnected", or "Disconnected" in a terminal or a browser | You ended that connection from another client, with `isc disconnect` or **Disconnect** | Press Enter in the window, any key in a terminal, or load the page again |
| "The server did not move to the new version" in the window, or `isc: the daemon was not moved` | The installed `isc` is missing from where the daemon was started, or can't take over what it runs | Nothing was lost. Press Enter to go back to it. Reinstall; or run `isc kill-server` and start again, which ends what's running |
| `daemon: running, but it does not say what it is`, from `isc version` | The daemon is from before the requests that every version answers, or it's stuck | Run `isc restart-server` |
| `a daemon is already running` | `isc daemon` was run twice | Nothing; or run `isc kill-server` first |
| `path must be shorter than SUN_LEN` | The socket's path is too long | Set a shorter `INSENSICAL_SOCKET` |
| `isc: ssh needs your key or password for build` | `ssh` to that machine needs something typed, and a link can't type it | `isc host answer build` shows what it asks. Give the machine a key (`ssh-copy-id build`), or load yours into the agent (`ssh-add`) |
| `isc: ssh hasn't met build before` | `ssh` would have asked about the host's key | `isc host answer build`, and answer it |
| `isc: build's host key is not the one ssh has for it` | The machine's key changed | Find out why before anything else. `ssh-keygen -R build` forgets the old key |
| `isc: build is off` | It's a machine of your tailnet, and Tailscale says it isn't online | Turn it on, or start Tailscale there |
| `` isc: build has no `isc` where a shell finds it `` | insensical isn't installed on that machine, or is somewhere `ssh` doesn't look | Install it there; `~/.local/bin`, `/usr/local/bin` and Homebrew's `bin` are looked in |
| `isc: build has isc X … and this machine has isc Y … which cannot speak to each other` | The two machines have different versions | Install the same version on both (`isc host put build` puts this one there), then `isc host restart build` |
| `isc: this connection was not given the right to …, which … needs. Its rights are: …` | The command was run with `--rights`, and what it asked needs a right that was left out | Run it again with that right, or without `--rights` |
| `` isc: `isc integrate` asks before it changes an agent's settings, and on another machine there is nobody to ask `` | `isc --host NAME integrate` without `--status` or `--yes` | Look with `--status`, then add `--yes` |
| `isc: theme is a setting of the interface, read from the file on the machine the interface runs on` | `isc --host NAME config` was given a setting that isn't the daemon's | Leave `--host` out |
| `` isc: this daemon was not started as `isc daemon`, so it has no `isc` to do that with `` | The daemon is one a test or another program runs inside itself | Run `isc integrate` or `isc config` on that machine |
| `isc: the link to the other machine was lost`, or "is out of reach" in the interface | The machine stopped answering, or `ssh` ended | Nothing there was lost. Run the command again; the interface tries by itself |
| The update notice appears though nothing was updated | The binaries were rebuilt or reinstalled | Restart when convenient |

### Settings and appearance

| What is seen | Why | What to do |
|---|---|---|
| A red message in the top bar | A setting couldn't be used; the message names the file and line | Fix the line; it applies within a second |
| All settings seem ignored | One unreadable line voids the whole file | As above |
| Text smaller or larger than expected | `font-size` is in points | 12 points is 16 pixels on Linux |
| Text thinner than in another terminal | There's no font weight setting | Nothing; the regular face is always used |
| `theme = "ghostty"` is reported as an error | Ghostty isn't installed | Use a built-in theme or a theme file |
| The window has a generic icon | It was run without installing, or the icon cache is old | Run `just install` or `just install-local`; restart the launcher or panel |

### Panes and shells

| What is seen | Why | What to do |
|---|---|---|
| A different shell, prompt or completion style than in another terminal | Panes run `$SHELL`; the other terminal was set to run something else | Set `shell = "…"` |
| Commands not found in panes that exist in other terminals | The daemon was started from a launcher with a short `PATH` | Set `PATH` in the shell's start-up files, or stop the daemon with `isc kill-server` and start it from a terminal |
| On another machine, the first letters of a line show late and the rest at once | Typing is drawn ahead only once the program has shown two letters you typed | Nothing: it's how a password is kept from being drawn. `type-ahead = "off"` makes every letter wait alike |
| On another machine, a letter you typed shows and then goes | The program answered the key with something else, or with nothing | Nothing: what's left is what the program has |
| A new variable in a shell profile is missing in panes | The daemon keeps the environment it was started with | The same |
| Programs over `ssh` complain about an unknown terminal | The other machine has no `xterm-ghostty` description | `infocmp -x xterm-ghostty \| ssh host tic -x -`, or `term = "xterm-256color"` in the settings |
| A pane shows the generic terminal icon | It has printed nothing and been sent nothing, or its program isn't in the list | Nothing |
| A pasted block waits at the foot of the pane | It has several lines and the program didn't ask for marked pastes, or it holds the sequence that ends a marked paste. The card says which | Read what it shows, then press Enter to paste it, or Escape to drop it |
| `Ctrl`+click on a path shows it in the file manager instead of opening it | It's a directory, a file that can be run, or a kind of file that opening could run | Open it yourself, or set `open-files-with` to your editor |
| A path isn't underlined under the pointer | Paths are looked up only while `Ctrl` is held | Hold `Ctrl` |
| An input method does nothing | It's off by default | Set `input-method = true` |
| On a Mac, Alt+B and Alt+F do nothing in a shell, or type `∫` and `ƒ` | Option types the keyboard layout's character | Set `option-as-alt = true` |
| Search misses text that is on screen | The text wraps across three rows or more. What wraps across two is found | Search for a part that's on one or two rows |
| A pane does not fit its window, and its header says "Sized elsewhere" | Another window, or `isc attach`, was used last and the pane is its size | Click this window, or press the mark |
| `` Ctrl+` `` does not reach a program | That key brings up the scratch pane | Set the key to `none` in `[keys]` |

### States, agents and notifications

| What is seen | Why | What to do |
|---|---|---|
| A pane never shows `done` | The command ran for under 30 seconds and reported nothing | Lower `long-command`, use `isc status` from a hook, or a program that reports progress |
| A failed command shows `done` | The shell doesn't mark its commands | Use fish 4, or the `isc shell-integration` line for bash or zsh |
| A pane stuck in a state | A hook or the program reported it and nothing has reported since | `isc status explain` says who. `isc status clear`, or "Forget what this pane's program said it was doing" in the palette |
| `isc status` changes nothing in a pane | The pane's program says in its terminal that it's working or waiting, and what `isc status` says is set aside | `isc status explain` says so. It counts again when the program says it's done or idle, or ends |
| A pane stays `running` or `waiting` after its program was killed | The program reported in its terminal and never took it back, somewhere insensical can't see it end: through `ssh`, in a pane started with a command, or under a shell inside a shell. See "Programs that report by themselves" | `isc status clear`, or "Forget what this pane's program said it was doing" in the palette |
| A program's reports aren't heard | It runs inside tmux or screen, which don't pass them on | Run it in the pane itself |
| A Gemini CLI pane stays `running` after a failed or cancelled turn | Gemini CLI has no event for either | The next prompt clears it; or run `isc status idle` |
| Hooks stopped reporting | `isc` moved since `integrate` | Run `isc integrate` for that agent again |
| Codex reports nothing | Its hooks aren't trusted | Run `/hooks` in Codex |
| The bell marks a pane but no notification arrives | The bell never notifies | Nothing |
| Nothing reaches your phone | A window, a terminal or a browser that can type is attached, so you count as there; or the command or the address failed; or a `notify-` setting is mistyped | `isc notify --sinks` says how the last one went. Set `notify-command-when` or `notify-url-when` to `always` |
| `isc notify --test` says `https` can't be done on this processor | It lacks what the built-in `https` needs | Use `notify-command` with `curl` |
| A clicked notification selects the pane but the window stays behind | Wayland; see section 12 | Bring the window forward yourself |
| Notifications have no application name or icon | The desktop entry isn't installed | Install |
| The tray's icon is blank | insensical's icons aren't installed | Install |
| There's no icon in the tray | The desktop has no tray for status notifier items, or `tray` is off | On GNOME, add the AppIndicator extension |
| The leader does nothing | The desktop took `Ctrl+Space` | Pick another `leader` |

---

## 19. Reference

### Keys

"—" means there's no
default key: the command is in the palette, and you can bind it in `[keys]`. The
Command column is the name to use in `[keys]`. A capital letter after the leader means
the letter with Shift, written `shift-h` in `[keys]`.

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
| `forget` | Forgets what this pane's program said it was doing, and puts the pane at rest |
| `select-all` | Selects everything in the pane |
| `integrate` | Gets the agents on this system to report what they're doing |
| `none` | Not a command. In `[keys]`, it leaves the key to the program. |

After the leader, these commands repeat without it for one second: `resize-left`,
`resize-right`, `resize-up`, `resize-down`, `move-left`, `move-right`, `move-up`,
`move-down`, `move-tab-left` and `move-tab-right`.

Resize mode is `resize-mode`: leader, then `Shift+R`. While it's on, `Left` or `H`,
`Down` or `J`, `Up` or `K` and `Right` or `L` move the edges of the pane in front, and
Escape or Enter turns it off. See [Changing keys](#changing-keys).

Five commands have no key and are in the window only: `open-project-file` ("Open a
project file") and `save-project-file` ("Save this project as a file"), for which see
[A project in a file](#a-project-in-a-file), `share-project` ("Share this
project") and `share-pane` ("Share this terminal"), for which see "Share one project
or one terminal" under [In a browser](#in-a-browser), and `add-machine` ("Add a
machine"), for which see [Other machines in the window](#other-machines-in-the-window).

### Settings

Settings go in `config.toml`. Every key is optional.

| Key | Meaning | Default |
|---|---|---|
| `theme` | A theme name, or one for each appearance of the desktop: `{ light = "paper", dark = "fjord" }`. A name can be a theme that comes with insensical (see [Themes](#themes)), a file in `themes`, `ghostty`, or `ghostty:Name`. | `fjord` when the desktop is dark, `paper` when it's light |
| `font-family` | The font for terminals. It doesn't have to be a Nerd Font: icons it lacks come from the icon font that's part of insensical. | `JetBrainsMono Nerd Font` |
| `interface-font` | The font for the interface around the terminals: tabs, sidebar, headers, panels | the font the system uses for its own interface |
| `interface-style` | `themed`: the window is in the theme's colours. `neutral`: it's in greys around the terminals, with the theme's accent and state colours. | `themed` |
| `background-opacity` | How much of what's behind the window its background hides, 0.2 to 1. Below 1 the desktop shows through behind the text. | `1` |
| `background-blur` | Whether the desktop is asked to blur what shows through. Not every desktop does. | `false` |
| `motion` | Whether sheets, notices and moved panes fade in, the working mark turns and cursors blink. `system` follows the desktop, `on` and `off` say it outright. | `system` |
| `font-ligatures` | Whether symbols side by side are drawn as the font's ligature for them, such as `->` and `!=` | `true` |
| `font-size` | Terminal text size in points, 6 to 72. A point is 1⅓ pixels on Linux and one pixel on macOS. Interface text doesn't follow it. | `12`; on macOS `13` |
| `density` | `comfortable`: each pane is a rounded card with space around it. `compact`: panes sit edge to edge with a hairline between them. | `comfortable` |
| `pane-gap` | Space between panes in pixels, 0 to 40 | `6`, or `0` when `density` is `compact` |
| `sidebar` | The levels listed under a project's name: `["tabs", "panes"]`, `["tabs"]`, `["panes"]`, or `[]` for names only | `["tabs", "panes"]` |
| `activity-idle` | With the sidebar showing only activity, whether agents at rest are listed | `true` |
| `tabs` | Where the tabs show. `sidebar`: down the side, for every project. `top`: along the top, for the current project. | `sidebar` |
| `pane-header` | `full`: what the pane is and what it's doing, plus buttons to arrange it. `name`: the same without the buttons. `hidden`: no header; a floating pane keeps its name. | `full` |
| `keymap` | `direct`: each command has its own chord. `leader`: each command is a key you press after the leader, which leaves the chords to the programs in the panes. | `direct` |
| `leader` | The key you press before a command's key when `keymap` is `leader` | `ctrl-space` |
| `shell` | What a pane runs when you give it no command, with any arguments: `"fish"`, `"/bin/zsh -l"`. The daemon reads it when a pane starts. | the login shell, `$SHELL` |
| `input-method` | Whether your system's input method may compose text in a pane, as for Bangla, Chinese or Japanese. What it's composing shows at the cursor; the program gets the finished text. | `false` |
| `option-as-alt` | On a Mac, whether Option with a key is Alt to the program in the pane, as a shell's Alt+B and Alt+F are, and no longer types the character the keyboard layout gives it. It does nothing elsewhere. | `false` |
| `type-ahead` | Whether what you type in a pane on another machine is drawn at once, a little fainter, before its program shows it. `auto`: when the round trip there takes 70 milliseconds or more. `on`: over any link to another machine. `off`: never. See [Other machines, over ssh](#other-machines-over-ssh). | `auto` |
| `open-files-with` | What a file you `Ctrl`+click in a terminal opens in: `system`, an editor by name (`code`, `cursor`, `zed`, `subl`, `idea`, `nvim`, `vim`, `hx`, `nano`), or a command with `{file}` in it. See [Web addresses and paths](#web-addresses-and-paths). | `system` |
| `term` | What a pane's program is told the terminal is, in `TERM`. `xterm-ghostty` gives programs every feature. `xterm-256color` is known on every machine, which matters over `ssh`, and has fewer. Read by the daemon when a pane starts. | `xterm-ghostty` |
| `scrollback` | How much history each pane keeps, in megabytes, 1 to 200. Read by the daemon when a pane starts. | `10` |
| `keep-scrollback` | Whether what panes show is saved to disk when the daemon stops, so it's there above a new shell the next time the daemon starts. Read by the daemon. With `false`, nothing is saved, and anything saved earlier is removed as panes close. | `true` |
| `tmux-shim` | Whether a program in a pane finds a `tmux` that insensical answers, so an agent that opens its helpers in tmux panes opens them as panes here (section 11). Read by the daemon when a pane starts. While it's on, `tmux` typed in a pane isn't the real one. | `false` |
| `clipboard` | Whether a program can put text on the clipboard. `focused`: only the program in the pane you're looking at. `never`: no program. No program can ever read what's on the clipboard. Read by the daemon. | `focused` |
| `notifications` | Whether you're told when a pane has something for you. Read by the daemon. | `true` |
| `attention-glow` | Whether a waiting pane takes the waiting colour at its edge and casts a soft light of it into the gaps around it. | `true` |
| `notify-waiting` | Whether you're told when a pane waits for you, or a program asks for you. Read by the daemon. | `true` |
| `notify-failed` | Whether you're told when a command or an agent ends badly. Read by the daemon. | `true` |
| `notify-done` | Whether you're told when a long command or an agent's turn ends well. Read by the daemon. | `true` |
| `notify-command` | A program and its arguments that the daemon runs to tell you, with what happened in its environment. See [When nobody is attached](#when-nobody-is-attached). | none |
| `notify-command-when` | When the command is run. `away`: only with no window, terminal or browser attached. `always`: every time. | `away` |
| `notify-url` | An `http` or `https` address the daemon posts to, to tell you. | none |
| `notify-url-when` | When it's posted to: `away` or `always`, as for the command. | `away` |
| `notify-url-body` | What's posted. `json`: the title, the message and the rest. `text`: the message alone, with the title in a `Title` header. | `json` |
| `notify-url-token` | A token sent as `Authorization: Bearer …` with what's posted. A secret: shown masked, and on no settings sheet. | none |
| `notify-url-user`, `notify-url-password` | A user and password sent as Basic. The token wins if both are set. Secrets, as the token is. | none |
| `notify-url-headers` | A table of headers sent with what's posted, for a token. Shown masked, and on no settings sheet. | none |
| `long-command` | How many seconds a command runs before its end is news: the pane is marked done or failed, and you're told if you weren't looking. 1 to 3600. Read by the daemon. | `30` |
| `sound` | Which notifications make a sound. `needs-you`: a pane that waits or failed. `all`: those, and a softer sound for a finish. `off`: none. | `needs-you` |
| `tray` | On Linux, whether insensical has an icon in your desktop's tray, marked while a pane needs you, with a menu of those panes. Read by the daemon. It does nothing on a Mac. | `true` |
| `menu-bar` | On a Mac, whether insensical has an item in the menu bar that counts and lists the panes that need you. It does nothing elsewhere. | `true` |
| `dock-badge` | On a Mac, whether the number of panes that need you is on insensical's Dock icon. It does nothing elsewhere. | `true` |
| `[tui]` | What `isc tui` does differently from the window, which never reads this table: `theme` (where `terminal` is the terminal's own colours), `activity-idle`, `interface-style`, `motion`, `attention-glow`, `type-ahead`, `clipboard`, `keymap` and `leader`, each in place of the one at the top of the file, and its own `colours` (`auto`, `truecolor`, `256`, `16`) and `mouse` | `colours = "auto"`, `mouse = true` |
| `[keys]` | Keys and the command each one runs, or `none`, in either keymap | empty |
| `[modes]` | Modes of your own, and the resize mode: a table for each, with a `title`, a `timeout` and the keys that work in it. See [Changing keys](#changing-keys). | `resize` |
| `[icons]` | A program's name and the tile it is shown by: a program that has one, a mark, or `{ mark = "docker", colour = "#ff8800" }`. See [Icons for your own programs](#icons-for-your-own-programs). | empty |
| `[commands]` | Commands of your own: a name and a list of steps. See [Changing keys](#changing-keys). | empty |
| `[keys.direct]` | The direct keymap, written out. It lies over the defaults: your lines win, a default you don't mention stays, and `none` takes a default key away. | the keys in the table above |
| `[keys.leader]` | The leader keymap, written out. Each key is the one you press after the leader. It lies over the defaults as `[keys.direct]` does. | the keys in the table above |
| `[hosts]` | Other machines by the name you give each, and where `ssh` goes for it: `build = "me@build.example.com"`. Written as `{ ssh = "…", color = "…", icon = "…", careful = true }`, a machine also has a colour (`#rrggbb` or `blue`, `green`, `orange`, `purple`, `pink`, `teal`, `gold`, `grey`, `rust`, `indigo`), an icon (`laptop`, `desktop`, `server`, `cloud`), and a mark that it's one to be careful on. `isc host add` writes these, and `node = "…"` for a machine it found on your tailnet: the node's id, by which the machine is dialled at the address it has now. See [Other machines, over ssh](#other-machines-over-ssh). | empty |

An unknown key is an error. The file that `isc config` prints sets `theme` to
`{ light = "paper", dark = "fjord" }`, which is the same as leaving it out. It shows
`shell` and `interface-font` as lines to uncomment, and writes out `[keys]`,
`[keys.direct]` and `[keys.leader]` with their defaults.

For the theme file format, see [A theme file](#a-theme-file).

### `isc` commands

`PANE` is a pane's number: `3` or `pane:3`. A closed pane's number isn't given to
another, with two exceptions: when the saved layout is missing or was set aside as
unreadable, numbers start again from 1; and a pane opened within two seconds of another
closing has its number given again if the server stops in those two seconds. `TAB` is
a tab's number: `7` or `tab:7`. A project is its name or its number: `4` or
`project:4`. Where `[PANE]` is optional, the command acts on the pane you run it in.
Outside any pane, it acts on the pane that has the keyboard. Every command accepts
`--socket PATH`. Every command about a daemon also accepts `--host NAME`, which makes
it act on the daemon of a machine you've named: see
[Other machines, over ssh](#other-machines-over-ssh). `--rights look,type` asks the
daemon for those rights and no others, of `look`, `type`, `arrange` and `manage`: see
[Who is connected](#who-is-connected).

| Command | What it does |
|---|---|
| `isc ls [--json]` | Lists projects, tabs and panes, with each pane's placement and state. `--json` prints everything the daemon knows. `isc list` does the same. |
| `isc run [--cwd DIR] [--keep] [WHERE] [-- COMMAND…]` | Runs a command in a new pane, or the shell if you give none, and prints the pane as `pane:N`. `--cwd` sets the working directory; the default is the one you run `isc` in. With `--keep`, the pane stays when the command ends. Press Enter in it, or use `isc close`, to close it. A command and the directory it starts in can be 256 KB together at most. The names, commands and directories of everything you have open can be 4 MB together, and there can be 4,096 panes; past either, the server says so and starts nothing. |
| `WHERE` for `isc run` | Nothing: a new tab in the current project. `--project NAME`: a new tab in that project. `--new-project [--name NAME]`: a new project for the working directory. `--split right\|down [--beside PANE]`: beside a pane, sharing its space. `--float [--rect X,Y,W,H]`: floating above the current tab. |
| `isc split right\|down [PANE] [-- COMMAND…]` | The same, beside a pane, in that pane's directory |
| `isc focus PANE`, `isc focus left\|right\|up\|down` | Gives a pane the keyboard: by number, or the focused pane's neighbour on that side |
| `isc float [PANE] [--rect X,Y,W,H]` | Lifts a pane out of the layout, or moves one that's already floating. The rectangle is in fractions of the tab. Without it, the rectangle is `0.2,0.15,0.6,0.7`. |
| `isc tile [PANE]` | Puts a floating pane back into the layout |
| `isc pin [PANE] [--off]` | Keeps a floating pane above the others. `--off` stops that. |
| `isc follow [PANE] [--off]` | Shows a floating pane over every tab of its project. `--off` shows it over its own tab only. |
| `isc hide [PANE]`, `isc show PANE` | Hides a floating pane while its program keeps running, and brings it back to the front |
| `isc scratch [--project NAME]` | Brings the project's scratch pane to the front. If it's already there with the keyboard, puts it away. Starts it the first time. `--project` names a project other than the one in front. |
| `isc floats show\|hide` | Shows or hides the current tab's floating panes |
| `isc minimize [PANE]`, `isc restore PANE` | Puts a pane away while its program keeps running and keeps its size, and brings it back where it was |
| `isc zoom [PANE] [--off]` | Gives a pane the keyboard and lets it fill its tab. `--off` stops that. |
| `isc swap A B` | Swaps the places of two panes |
| `isc put PANE left\|right\|above\|below [--of OTHER] [--tab TAB]` | Moves a pane to that side of another pane, or of all the panes of a tab |
| `isc resize left\|right\|up\|down [PANE] [--by PERCENT]` | Moves the divider a pane shares with its neighbours in that direction, by 5% of the divided space unless you set `--by`. It fails when the pane has no divider on that axis. |
| `isc rename PANE NAME` | Gives a pane a name, shown instead of its directory and title. An empty name, `""`, removes it. |
| `isc close [PANE]` | Closes a pane and ends its program |
| `isc tab new` | Opens a tab with a shell in the current project |
| `isc tab next`, `isc tab prev` | Goes to the next or previous tab of the current project, wrapping round at the ends |
| `isc tab select TAB` | Goes to a tab |
| `isc tab close [TAB]` | Closes every pane in a tab, or in the current tab if you name none |
| `isc tab move TAB INDEX` | Moves a tab to a position among its project's tabs, counted from 0 |
| `isc tab rename TAB NAME` | Names a tab. An empty name removes it. |
| `isc project new [DIR] [--name NAME] [--make]` | Opens a project for a directory, or for the current one if you give none. It's named after the directory unless you set `--name`. With `--make`, a `DIR` that isn't there is made first, with the ones above it, on the machine the command is for; if one can't be made, nothing is left made and no project is opened. Prints its first pane. |
| `isc project open [FILE] [--again] [--yes]` | Makes the project a file describes: see [A project in a file](#a-project-in-a-file). `FILE` is the file, or a directory holding one called `.insensical.toml`; without it, the current directory. A project of the file's name and root that's already open is gone to and left alone, unless you say `--again`. A file that starts programs is listed and asked about the first time and whenever it has changed; `--yes` answers for you, and without a terminal or `--yes` it isn't opened. Prints the project as `project:N`. |
| `isc project save [FILE] [--force]` | Writes the project in front as such a file: `.insensical.toml` in the project's directory, or `FILE`, or with `-` to standard output. It doesn't write over a file unless you say `--force`. |
| `isc project select NAME` | Goes to a project |
| `isc project rename NAME NEW` | Renames a project |
| `isc project icon NAME [--mark MARK] [--colour #rrggbb] [--svg FILE]` | Chooses what a project is shown by. With none of them, it's the first letter of its name again |
| `isc project move NAME INDEX` | Moves a project to a position among the projects, counted from 0 |
| `isc project close NAME` | Closes a project and everything running in it |
| `isc send PANE TEXT` | Types text into a pane, as if at its keyboard |
| `isc key PANE KEY…` | Presses keys by name, in order: `ctrl-c`, `enter`, `escape`, `shift-tab`, `a` |
| `isc attach [PANE]` | Shows a pane in this terminal and lets you type into it. `Ctrl+]` leaves. |
| `isc`, `isc tui` | Draws the whole interface in this terminal. `isc` alone does so only for a person at a terminal: in a script, a pipe or one of insensical's own panes it lists the commands. `Ctrl+Space` then `D` leaves. See [The whole interface in a terminal](#the-whole-interface-in-a-terminal). |
| `isc capture PANE [--vt]` | Prints a pane's screen and scrollback. With `--vt`, it also prints colours and styles, as escape sequences. |
| `isc status STATE [-m MESSAGE] [--resume COMMAND] [-p PANE]` | Reports what a pane is doing: `idle`, `running`, `waiting`, `done` or `failed`. `-m`, or `--message`, says what the state is about. `--resume` is a command that picks the work up again, offered at the pane's prompt after the daemon has stopped. `-p`, or `--pane`, is required outside a pane. |
| `isc status explain [-p PANE]` | Explains why a pane is in its current state |
| `isc status clear [-p PANE]` | Forgets what was reported for a pane, by its program or by a hook, and puts it at rest |
| `isc notify TITLE [BODY] [-p PANE]` | Asks for your attention on a pane's behalf |
| `isc notify --test` | Runs `notify-command` and posts to `notify-url` now, and says how each went |
| `isc notify --sinks` | Says what those two are set to, how the last sent to each went, and how many weren't sent |
| `isc mute [PANE] [--off]` | Stops a pane from sending notifications. `--off` lets it send them again. Its state still shows. |
| `isc integrate [claude\|codex\|gemini\|opencode\|pi] [--remove] [--yes]` | Adds an agent's hooks, after showing you the change and asking. With no agent named, it offers each agent found on this system. `--remove` takes the hooks out. `--yes` skips the question. |
| `isc integrate [AGENT] --status` | Changes nothing. Prints, as JSON, for each agent: whether it's found, whether it already reports, the file its hooks go in, and what's left for you to do afterwards. |
| `isc wait PANE [--until STATE]… [--timeout SECONDS]` | Blocks until the pane reaches a state: `idle`, `running`, `waiting`, `done`, `failed` or `closed`. Without `--until`, the states are `done`, `failed`, `waiting` or `closed`. Prints the state reached. The exit status is 124 when the time runs out. |
| `isc wait PANE --output TEXT [--timeout SECONDS]` | Blocks until the pane's screen or history shows the text. Exits with 124 if the time runs out. |
| `isc events` | Prints what happens, one JSON object per line, until interrupted |
| `isc api [REQUEST]` | Sends the daemon one request as JSON, from the argument or standard input, and prints the answer as JSON |
| `isc api [--json]`, `isc api NAME [--json]` | Lists every request, or says what one holds, with an example. Sends nothing, and needs no daemon. |
| `isc do NAME [--yes]` | Runs a command by its name: one of your own from `[commands]`, or one of the window's. Stops at the first step that fails. `--yes` goes ahead where a step would end what's running. |
| `isc project sleep NAME` | Ends every program in the project and keeps its tabs, panes and what they showed. Doesn't ask. |
| `isc project wake NAME` | Puts a shell in each pane of a sleeping project, in the pane's directory |
| `isc project mute NAME [--off]` | Stops every pane of the project from sending notifications, including panes opened in it later. `--off` lets them again. |
| `isc usage [--json]` | Prints what the programs in each pane are using: memory, processor time and how many processes, with a total for each project |
| `isc clients [--json]` | Lists who is connected to the daemon: each client's kind, machine, version, rights, process, and the pane it's in or looking at |
| `isc disconnect CLIENT` | Puts another client off the daemon, by the number `isc clients` shows (`7` or `client:7`). Needs the right to `manage`. |
| `isc config` | Prints a settings file with every setting at its default and every key of both keymaps |
| `isc config get NAME` | Prints what a setting is: what the file gives it, or its default. With `--host`, for one of the daemon's settings, from the file on that machine. |
| `isc config set NAME VALUE` | Gives a setting a value in the settings file, changing only that line. With `--host`, one of the daemon's settings, in the file on that machine. With `--stdin` in place of `VALUE`, the value is read from standard input, so it isn't on the command line where other users can see it. |
| `isc config reset NAME` | Takes a setting out of the settings file, which leaves its default. With `--host`, as `set`. |
| `isc completions bash\|zsh\|fish` | Prints what that shell needs to complete `isc` commands, including the panes, tabs and projects that exist |
| `isc shell-integration bash\|zsh\|fish` | Prints what that shell needs to tell its pane where each command starts and ends, and its exit status. fish 4 and later doesn't need it. |
| `isc host add NAME WHERE` | Names another machine. `WHERE` is what you'd give `ssh`. It's written under `[hosts]` in the settings file. `--color C`, `--icon I` and `--careful` set those on it. A machine of your tailnet is kept by its node and dialled at its address; `--no-node` keeps it as typed. |
| `isc host set NAME` | Changes what's set on a machine: `--ssh WHERE`, `--color C` or `--no-color`, `--icon I` or `--no-icon`, `--careful` or `--no-careful`, and `--name NEW` to rename it. Only what you give changes. |
| `isc host ls` | Lists the machines you've named, with where `ssh` goes for each and what's set on it, and for a machine kept by its tailnet node the address it has now. `--json` prints them as JSON, with `node`. |
| `isc host answer NAME` | Runs your `ssh` to the machine at your terminal, free to ask: a host key, a passphrase, a password, a sign-in. You answer `ssh` itself. Then it meets the machine. It succeeds only if the machine was then reached. `NAME` can be where `ssh` goes, for a machine with no name. |
| `isc host rm NAME` | Forgets a machine's name |
| `isc host check NAME` | Meets the machine and prints four lines: whether it was reached and the round trip, whether insensical is there in a version that works with this one, whether its server is running, and what's open there. A line that fails says what to do. It changes nothing there, and fails unless the machine is ready to use. |
| `isc host restart NAME [--yes]` | Restarts the server on a machine when it's running an older version than the `isc` installed there. It meets the machine, says what a restart comes to and asks; `--yes` doesn't ask. When the server there already is the installed version it says so and restarts nothing, without asking. A server that can move keeps what runs in its panes. |
| `isc host put NAME` | Puts this version's `isc` in `~/.local/bin` on a Linux machine that has none or an older one, over `ssh`, with no sudo. It says what it would write and asks; `--yes` doesn't ask. A server running there is left alone unless you say `--restart-server` or answer yes to a second question. |
| `isc proxy` | What `isc --host` runs on the other machine, through `ssh`. It carries connections to that machine's daemon over its standard input and output, and listens nowhere. You don't run it yourself. |
| `isc web [--port N] [--listen ADDR] [--tls-cert FILE --tls-key FILE] [--trust-proxy LOGIN,…] [--name NAME]… [--guest-port N] [--guest-name NAME]…` | Serves insensical to a browser, on this machine only unless given a certificate. `--name` is a name browsers reach it by, besides this machine's addresses and `localhost`. `--guest-port` (7434) and `--guest-name` are the port and the names of the door a public address leads to. See [In a browser](#in-a-browser) |
| `isc web token add --name NAME [--rights …] [--uses N\|any] [--expires 10m\|never] [--stays 1h\|1d\|always] [--only project:NAME\|pane:N] [--far] [--public] [--url BASE] [--json]` | Makes a token and prints it once, with the address to open and a code to scan. The address pairs one browser, for ten minutes, unless told otherwise. `--stays` is how long the device stays paired: it is signed out and the token gone when the time is up. `--only` makes it a guest that sees one project or one terminal and may look, or look and type |
| `isc web token ls [--json]`, `isc web token rm NAME` | Lists the tokens without showing them, with what each sees; ends one, and closes every browser paired with it |
| `isc web token set NAME --far\|--no-far` | Lets a device that is paired already open your other machines' terminals through this computer, or keeps it to this one again, which closes what it has open elsewhere at once |
| `isc web login add LOGIN [--rights …]`, `login ls`, `login rm LOGIN` | The logins `--trust-proxy` may trust, and what each may do |
| `isc web push ls [--json]`, `isc web push test [--at MACHINE] [--json]` | Lists the browsers that are sent notifications, and which hear your other machines; sends each a test and says how its push service answered, with `--at` as from that machine to the browsers that hear of it |
| `isc web open [--print\|--json]` | Opens insensical in your browser on this computer, signed in as the device `This computer`, from an address that works once, for ten minutes, and only on this computer; with `--print` or `--json` prints it and opens nothing |
| `isc web status [--json]`, `isc web stop` | Says whether `isc web` is running and where; stops it |
| `isc web public on`, `off`, `status [--json]` | Has Tailscale Funnel open a public address to the devices made with `token add --public`; removes those devices and takes the address away; says whether one is open, where, and for which devices (without `--json` it ends with 1 when none is) |
| `isc tmux COMMAND...` | Answers a `tmux` command with insensical's panes, as the `tmux` a pane finds with `tmux-shim` on does. See section 11 |
| `isc daemon` | Runs the daemon in the foreground |
| `isc kill-server` | Stops the daemon and every program it runs, and returns once the daemon is gone. The layout comes back the next time it starts. It also works on a daemon of another version. |
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
| Tokens, paired browsers and logins for `isc web`, as hashes | `~/.config/insensical/web/access.json` | the same |
| The key `isc web` signs notifications to browsers with, closed to other users | `~/.config/insensical/web/vapid.key` | the same |
| Where a running `isc web` is | `web.lock`, beside the daemon's socket | the same |
| Which port `isc web public on` pointed the public address at, until `isc web public off` has closed it | `~/.local/state/insensical/web-funnel` | `~/Library/Application Support/insensical/web-funnel` |
| That the settings sheet's switch left `isc web` on, and the names it answers to, where no service keeps it started | `~/.config/insensical/web-on` | the same |
| That the settings sheet's Tailscale switch made the `tailscale serve` mapping, which is the only one it turns off | `~/.config/insensical/web-serve` | the same |
| The window's size, whether its tabs were hidden, and what was folded in the sidebar | `~/.local/state/insensical/window` | `~/Library/Application Support/insensical/window` |
| What was chosen lately in the palette | `~/.local/state/insensical/palette-recent` | `~/Library/Application Support/insensical/palette-recent` |
| The terminal's description, when the daemon had to compile its own | `~/.local/state/insensical/terminfo/` | `~/Library/Application Support/insensical/terminfo/` |
| The host keys Tailscale holds for a machine of your tailnet that runs its own ssh server, which `ssh` is told to accept for it | `~/.local/state/insensical/tailnet-keys` | `~/Library/Application Support/insensical/tailnet-keys` |
| That the application has said, once, that it keeps running with no window | not written | `~/Library/Application Support/insensical/told-still-running` |
| Socket | `$XDG_RUNTIME_DIR/insensical/daemon.sock` | `$TMPDIR/insensical/daemon.sock` |

`INSENSICAL_CONFIG_DIR` replaces the settings directory. `INSENSICAL_STATE_DIR` replaces
the directory for the layout, screens and declined agents. `INSENSICAL_SOCKET` replaces
the socket.

The state directory is closed to other users (`0700`), and so is each file the daemon
writes in it (`0600`): the saved layout names the directories you work in and the
commands your panes were started with. If the directory or those files are open to
others when the daemon starts, it closes them. It writes nothing in a state directory
that belongs to someone else.

insensical writes these files in other programs' directories, and only when you tell it
to. It writes nothing else outside the directories above:

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
| `INSENSICAL_WEB_FILES` | `isc web` | A directory to serve the page from, in place of the files built into `isc`. For working on the page. |
| `INSENSICAL_ISC` | window, `isc` | The `isc` binary to start the daemon from. When set, it's used before the `isc` beside the program and the one on `PATH`. |
| `INSENSICAL_HOST` | `isc` | The name of another machine, as `--host` takes it. Every command about a daemon then acts on that machine's, except the ones that end something (`kill-server`, `close`, `tab close`, `project close`, `project sleep`), which want `--host` on their own line. `--socket` given on the command line comes first. The daemon doesn't pass it on to its panes. |
| `INSENSICAL_SSH` | `isc` | A program to run in place of `ssh` to reach another machine, with the same arguments. |
| `INSENSICAL_TAILSCALE` | `isc`, the application | A program to run in place of `tailscale`, with the same arguments. When set, no other place is looked in. |
| `INSENSICAL_ISC_X86_64`, `INSENSICAL_ISC_AARCH64` | `isc` | A file on this machine to send as `isc` when insensical is put on another machine with that processor. When set, it's sent in place of the release's file. It has to be an `isc` for Linux in the version you have here. |
| `SHELL` | daemon | What a pane runs when there's no command and no `shell` setting. `/bin/sh` if unset. |
| `TERMINFO`, `HOME` | daemon | Searched for Ghostty's terminal description. |
| `TERMINFO_DIRS` | daemon | Kept, after the daemon's own directory, when it points a pane at the description it compiled. |
| `XDG_CONFIG_HOME` | window, daemon, `isc` | Where the settings directory is, instead of `~/.config`. Ghostty's and opencode's directories are also looked for here. |
| `XDG_STATE_HOME` | daemon, window | Where the state directory is, instead of `~/.local/state`. Linux only. |
| `XDG_RUNTIME_DIR` | everything | Where the socket's directory is, on Linux. Where `ssh` runs `isc proxy` without it, `/run/user/` and your user's number is used if it's there, and otherwise `run` under the state directory. |
| `TMPDIR` | everything | Where the socket's directory is, on macOS. |
| `PATH` | daemon, window, `isc` | Every pane inherits the daemon's. The window's is searched for `isc` and for the program that `shell` names. `isc integrate` searches its own for agents. |
| `CLAUDE_CONFIG_DIR`, `CODEX_HOME`, `GEMINI_CLI_HOME`, `PI_CODING_AGENT_DIR` | `isc integrate` | Where each agent keeps its settings, instead of its default directory. |
| `GHOSTTY_SOURCE_DIR` | the build | A local Ghostty checkout to build from, instead of downloading. |
| `PREFIX` | `just install-local` | Where to install, instead of `~/.local`. |

Set in every pane: `TERM`, `COLORTERM=truecolor`, `TERM_PROGRAM=insensical`,
`TERM_PROGRAM_VERSION`, `INSENSICAL_PANE`, `INSENSICAL_SOCKET`.

Set for `notify-command`: `INSENSICAL_EVENT`, `INSENSICAL_TITLE`, `INSENSICAL_BODY`,
`INSENSICAL_KIND`, `INSENSICAL_PANE`, `INSENSICAL_PROJECT`, `INSENSICAL_MACHINE` and
`INSENSICAL_PROGRAM`. See [When nobody is attached](#when-nobody-is-attached).
