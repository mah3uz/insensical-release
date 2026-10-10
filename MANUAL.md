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
  It counts the panes that need you, and one key takes you to the most urgent.
- Tells you when a pane starts waiting, finishes or fails, unless you're looking at it:
  with a notice inside the window while insensical is in front, and a desktop
  notification when it isn't.
- Shows what each project's programs hold in memory, and puts a project to sleep when
  you say so: its programs end, and its panes and what they showed are kept.
- Has every setting on one sheet, to be switched or picked without opening a file.
- Does everything the window does from the command line too.
- Draws the whole interface in a terminal, with `isc` alone, for a machine with no desktop
  or one you reach over `ssh`.

### How far it has been checked

Read this before you rely on it.

- **Linux.** Everything described here is built and covered by tests, including tests
  that drive the real application off-screen. The application is in daily use in a real
  window on Arch Linux under Hyprland, at a fractional scale of 1.33. The most recent
  additions have been checked off-screen only, not in a real window: moving the daemon
  to a new version in place, panes showing what they showed after a stop, the scratch
  pane, finding a file and its coloured icons, the list of what follows the leader, the
  offer to set agents up, dragging tabs, notices inside the window, the ranked palette,
  what a project's programs hold, and putting a project to sleep. So have these: two
  windows on one pane, a tab by its number, dragging rows in the sidebar, search and
  links across a wrapped row, going into a directory in the file finder, and icons for
  your own programs.
  Used in a real window, under Hyprland with DankMaterialShell: the tray's icon and its
  menu, with closing the window and quitting from it; a clicked notification bringing
  the window forward, and opening one when none is open; what was folded in the sidebar
  kept for the next window; dragging a pane to a side of another; a picture shown by a
  real program; a project's icon and its choice; the Codex integration; and completions
  in zsh. The two sounds have been listened to. Where it matters most, this manual
  says what has not been tried.
- **macOS.** It builds on a Mac and its tests pass there, including the window's tests,
  drawn off-screen (macOS 26.6.2, Apple silicon). It has been installed from the
  source, updated in place, quit and brought back, and its author works in it there. A
  development build has been driven in a real window and looked at, feature by feature.
  `docs/macos.md` lists what was checked, and for each item who saw it. Every
  statement about macOS in this manual marked "never run" is unverified.

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
- `just` lists the tasks: `build`, `release`, `run`, `isc …`, `test`, `lint`, `check`,
  `bench`, `banner`, `icons`, `install`, `install-local`, `install-mac`.
- `cargo test --workspace`, or `just test`, runs the tests. The window tests draw
  off-screen and write screenshots to `target/screenshots/`. No test puts anything on
  your desktop or reads your own settings or layout. The window tests can hang at the
  very end of a test, about one run in twelve. This is a fault in off-screen testing;
  the real application doesn't have it. `just test` runs under a time limit. If it
  hangs, run it again.

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

**Other Linux**, on x86-64 with glibc 2.36 or later. Unpack the tarball and put
`insensical` and `isc` together in a directory on your `PATH`. The tarball also holds
the desktop entry, the icon, the user unit and completions for bash, zsh and fish. Put
them where your system looks for them.

**A Linux machine with no desktop**, on x86-64. You don't need the window there, only
`isc`: it's the daemon, the command line and [the whole interface in a
terminal](#the-whole-interface-in-a-terminal). `isc-x86_64-linux` is that one file. It
needs nothing installed and runs on any distribution, old or new:

```sh
cd "$(mktemp -d)"
curl -fLO https://github.com/mah3uz/insensical-release/releases/latest/download/isc-x86_64-linux -O https://github.com/mah3uz/insensical-release/releases/latest/download/isc-x86_64-linux.sha256
sha256sum -c isc-x86_64-linux.sha256 && install -m755 isc-x86_64-linux ~/.local/bin/isc
isc
```

- The second line fetches the file and its checksum, and the third installs the file
  only if the two agree. Both come from the same release, so this catches a download
  that was cut short or damaged. It doesn't tell you the release itself is the right
  one.
- That address always gives the latest release. To update, run the same lines.
  Replacing the file doesn't stop a daemon that's running; `isc restart-server --yes`
  moves it to the new version with everything still running.
- It has no completions with it. `isc completions bash`, `zsh` or `fish` prints them.
- It isn't on the website's download page, only on the release's own page.

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

The icon is `logo.svg` in the repository. A launcher shows it from the desktop entry.
A taskbar or switcher shows it for a window only if the desktop can match the window to
the entry, so the window names itself `insensical`. This matching has not been checked
in a real window.

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
- `isc daemon` runs it in the foreground. A second daemon on the same socket refuses to
  start and says "a daemon is already running". While it runs, a daemon holds a lock on
  a file beside its socket, `daemon.sock.lock`.
- It owns the projects, tabs, panes and their layout. Anything a window does to the
  layout, a command can do too, and each sees the other's changes immediately.
- It does no work while idle: no timers and no wakeups while nothing is happening.
- Only the same user can connect to it. There's no network listener and there never
  will be. To reach a pane on another machine, use `ssh` (see
  [Attaching from a terminal](#attaching-from-a-terminal)). The window itself has no
  remote hosts.
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

### Appearance

- The UI around the terminals uses the system's own proportional font at 13 pixels,
  whatever `font-size` is. On Linux that's the family fontconfig gives for `system-ui`,
  looked up once when the application starts. Use `interface-font` to name another. If
  you name the terminal's font, the UI uses the terminal's typeface.
- Icons in the UI are drawings, not letters of a font, so they look the same whatever
  font you use. Icons a program prints in a terminal come from the terminal's font if
  it's a Nerd Font, and otherwise from the icon font that comes with insensical (Nerd
  Fonts' symbols), so `font-family` doesn't have to be a Nerd Font.
- **A program is shown as a tile**: its own mark (Claude's, Neovim's, Docker's, Git's
  and the rest) on a small glossy card in the program's colours, like an application's
  icon. A program insensical has no mark for gets a terminal's. The colours belong to the program and not
  to the theme, so Neovim is green and Claude Code is coral in every theme, and you can
  find a pane by colour before you read its name. The tile is in the sidebar, the pane
  header, the tab strip, the dock, the overview and the palette.
- **The foot of the sidebar** has the window's tools: Settings, Show every key, and the
  overview of every tab, and at its other end a button for a new project. With the
  sidebar hidden, the three tools are at the right end of the top bar.
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
  - If the path isn't a directory, the window says so, and Enter does nothing.
- `Tab` in the name field moves to the directory field; `Shift+Tab` moves back. Clicking
  either field moves to it. Enter, or "Create", makes the project and opens a shell in
  it. Escape, "Cancel", or a click outside makes nothing.

**From the sidebar's field.** Press `Ctrl+Shift+L` to put the keyboard in the field at
the top of the sidebar. This shows the sidebar if it was hidden. With `tabs = "top"`
there's no such field, and the key opens the palette instead, which finds the same
things.

| Typed, then Enter | Result |
|---|---|
| A directory that exists, absolute or beginning `~` | Opens a project there, named after the directory |
| The name of an existing project | Goes to that project |
| Anything else | Creates a project with that name and no root |

As you type, the same field filters the tree.

From the command line, use `isc project new [DIR] [--name NAME]`, or
`isc run --new-project`.

**Where new panes open.** A new tab opens in the project's root. In a project with no
root, it opens in the directory of that project's front pane, and in the home directory
only when there's no such pane. A split opens in the directory of the pane it splits. A
new floating pane opens in the focused pane's directory.

**A project's name does not follow its panes.** A project named `insensical` is still
called `insensical` even if all its shells have changed directory to
`~/Projects/quote`. The tab and pane rows show where they really are.

**Renaming.** Press the pencil that appears on the project's sidebar row, or use
`isc project rename OLD NEW`. You can't give a project an empty name.

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
  fitted inside it. A copy is kept in `icons` beside your settings
  (`~/.config/insensical/icons/`, under the file's name and a number made of what it
  holds), so the file you chose can be moved or deleted, and two projects can each have
  a `logo.svg` of their own. Only SVG is taken. The copy is deleted when no project is
  shown by it any more: you chose another icon, reset it, or closed the project.
- A drawing that shows a picture from another file (an `<image>` in it) isn't taken,
  and you're told why. Draw it with shapes, or put the picture's own shapes in it.
- A drawing has to be a file of text. One packed with gzip (an `.svgz` given the name
  `.svg`) isn't taken: unpack it first.
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
- **A pane that needs you says why on its row.** If the pane is `waiting`, `failed` or
  `done` and gave a note, the note shows on a second line, in the state's colour. The
  row is taller for as long as that lasts, so the rows below it move down.
- Click a row to go to it. Click a minimized pane's row to bring the pane back.
- Each project stands a little apart from the one above it.
- The rename and close buttons take up room on the row even when they're invisible, so
  a long name is cut a little sooner than the row's width suggests.

**Naming.** A pencil appears at the end of the row under the pointer. Press it, type a
name, and press Enter. Escape cancels. For a tab or a pane, Enter on an empty field
removes the name. The daemon keeps names: they survive a
restart and appear in the tab strip, the pane's header, notifications and `isc ls`. From
the command line, use `isc tab rename TAB NAME` or `isc rename PANE NAME`. From the
palette, choose "Name this pane" or "Name this tab" (`name-pane`, `name-tab`; neither
has a default key): the sidebar's field takes the name, or with the sidebar hidden or
the tabs along the top, a field over the window does.

**Folding.** The mark at the start of a project or tab row hides what's under it.
Folding is per window and isn't remembered. While the filter field has text, its matches
are shown even inside something folded.

**Filtering.** Type in the field to narrow the tree to the projects, tabs and panes
whose name, title, program or directory contains what you typed. Case doesn't matter.

**What is listed.** The `sidebar` setting chooses which levels appear under a project's
name: `["tabs", "panes"]` (the default), `["tabs"]`, `["panes"]`, or `[]` for project
names alone.

**From the keyboard.** Press `Ctrl+Shift+L` (leader `e`) to put the keyboard in the
field. Down and Up then move a highlight through the rows in the order they're listed
or, after you've typed, through the rows that match. Left folds the highlighted project
or tab, and Right unfolds it. Enter goes to the row and gives the keyboard back to the
pane. With no row highlighted, Enter opens the project you typed. Typing clears the
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
- A pane whose terminal has more than 1,024,000 cells in all, 2000 columns by 600 rows
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

  The list is fixed. You can't add to it in the settings.

### The header

The header shows, in order: the icon; the title, or the name you gave the pane; the
directory, unless the title already contains it; a mute mark; the state as one word,
when the pane isn't idle; a percentage, when the program reports progress; what the
state is about; and "Enter closes" on a kept pane whose program has ended.

Buttons show on the focused pane and on the pane under the pointer: split right, split
down, float (or, on a float, put into the layout, keep on top and show over every tab),
minimize, fill the tab, close.

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

When you switch to a tab, its panes can show empty for an instant, until each one
receives its screen. If you scroll back into history that hasn't arrived, nothing shows
that it's missing.

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
  default is 12, or 13 on macOS. On Linux a point is 1⅓ pixels, as in Ghostty, kitty
  and Alacritty. So 12 points is 16 pixels, and the size you set in those terminals
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
  accent colour and "Copied" shows at the bottom of the pane. If nothing is selected,
  nothing is copied and no message shows. Wrapped lines are joined when you copy them.
- "Select everything in the pane" is a command, `select-all`. It has no default key.
- Typing returns the view to the bottom and clears the selection.
- A selection belongs to the window you made it in. It stays on its text as output
  arrives.

### Pasting

- Press `Ctrl+Shift+V` to paste the clipboard. Press `Shift+Insert` or the middle button
  to paste what you last selected.
- **A paste that would do more than it looks like is held**, and you're asked first.
  That's any of these:
  - More than one line, into a program that hasn't asked for pastes to be marked
    (bracketed paste). Such a program runs each line as it arrives.
  - Text that contains the sequence that ends a marked paste.
- **Who copied it makes no difference.** What a program put on the clipboard (see
  [The clipboard and programs](#the-clipboard-and-programs)) is pasted the way your
  own copy is. A command an agent copied for you lands at your prompt, where you read
  it before you press Enter.
- **The question shows what you'd be pasting.** A card at the bottom of the pane says
  how many lines there are and which of the two reasons applies, and shows the text
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

- `Ctrl`+click a web address to open it (`⌘`+click on macOS).
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
  else, your own group aside. A path through a shared directory such as `/tmp` is
  shown, not opened.
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
- It's made plain before it's put there. Control characters other than a tab and a
  line end are left out, and so are the marks that begin and end a paste: terminals
  differ in what they paste for those.
- At most three texts are put there at once, and one a second after that. The rest
  are dropped.
- `clipboard = "never"` turns it off completely.
- **No program can read the clipboard.** A request to read it is ignored, and there's no
  setting to allow it. Pasting is always something you do.

### Pictures

A program can show a picture in a pane: a plot, a thumbnail, a file manager's preview.
insensical draws pictures sent by the **Kitty graphics protocol**, which is what
`kitty +kitten icat`, `chafa`, `timg`, `viu`, matplotlib's kitty backend and many others
use. Programs find out by asking the terminal, and it answers that it can.

- A picture is part of the screen. It scrolls with the text around it, goes into the
  history, and is cut off at the pane's edge.
- It's drawn at the size the program asked for, counted in cells, so it takes the same
  cells whatever your text size.
- A program can put it under the text or over it, show a part of it, show it several
  times, and take it away again.
- **It's still there when you come back.** A pane's pictures are shown again when you
  return to its tab, open another window, or use `isc attach`.

What isn't done:

- **Sixel** pictures aren't shown.
- A picture has to be sent in the escape sequences themselves. One a program names by a
  file or by shared memory isn't read. Programs that prefer those fall back to sending
  it: `icat` does.
- Pictures placed with Unicode placeholder characters aren't drawn.
- Each pane keeps 64 MB of pictures. Past that, the oldest that aren't on screen are
  forgotten.
- A program can show one picture in many places. Of more than 256 places in view, the
  256 nearest the front are drawn.
- **Pictures don't survive the daemon stopping or moving to a new version.** The text
  comes back; the pictures don't until the program draws them again.
- When you come back to a pane, a picture whose top edge has scrolled out of view isn't
  shown again, and neither is anything past 32 MB of them. Scroll or redraw brings a
  program's own pictures back.
- A picture taller than its pane lies over the lines printed after it.
- `isc capture` and copying give the text only.

### What the terminal does not do

- **Input methods (IME) are off unless you turn them on.** With `input-method = true`,
  your system's input method (fcitx, ibus, the Mac's) composes text in a pane: what
  it's composing shows at the cursor, underlined, and the program is given only the
  finished text. Keys the input method doesn't take are typed as before. It's off by
  default because it has been tried only without a real input method: turn it on,
  and turn it off again if typing misbehaves. Whether accents typed as two keys (dead
  keys) work with it on depends on the toolkit and hasn't been tried.
- Sixel images aren't shown. Kitty graphics are: see [Pictures](#pictures).
- Shell-integration marks (OSC 133) are read to learn when a command ends and how it
  ended, and where each prompt is.
- When you make a pane narrower, its lines are rewrapped, even when a program has
  switched line wrapping off. A full-screen program's screen isn't rewrapped: it's cut
  off at the new edge, and the program draws it again.
- In a very large pane, of more than about 46,000 cells (300 columns by 160 rows, say),
  a full-screen program that scrolls from its first line can leave the pane's screen
  in a state that can't be copied to a window as it is. You then see the pane drawn
  afresh: what's on the screen, with its colours and the cursor, but without links, and
  without the shell's screen and history behind the program. You get those back the
  next time the pane is shown after the program has cleared its screen or ended.

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
- **In the sidebar, a project or a tab shows a state only while it's folded.** Unfolded,
  the rows beneath it show their own states and its row shows none. Folded, it shows the
  most urgent state of what it holds, and a project shows how many of its panes need you.
  If the settings list no rows beneath it, it always shows a state.
- **A waiting pane glows.** Its edge and its header take the waiting colour and a soft
  light of that colour fills the gaps around it, so you catch it from the corner of
  your eye among many panes. The light beats three times as the pane begins to wait and
  then stands still; with `motion = "off"` it's still from the start. It never falls on
  another pane's text or on the sidebar. With `pane-gap = 0` there's no gap to fill, and in a
  see-through window a tiled pane has nothing to hide the light behind, so in both
  cases you get the coloured edge and header alone. `attention-glow = false` turns off
  the light, the coloured edge and the coloured header.
- A pane's header shows its state as a word, along with what the pane says about it. On
  a tab, in the sidebar and on a dock chip, the state is a mark only. Hover over the mark
  to see the same words, for example "waiting: Needs permission: Bash".
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
  others.
- The palette opens with every pane that needs you at the top, most urgent first.
- The overview shows every tab of every project with each pane's state. For a pane that
  wants you, it also shows what the pane is asking.

### Where a state comes from

Strongest first:

1. **A report** from an agent's own hooks or a script:
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
     `waiting`, until the command ends or you look at the pane.
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
   status.

States aren't only for agents. Any command that runs for 30 seconds gives its pane a
state. So does any program that reports progress, sends a notification or rings the bell,
and any script that calls `isc status`.

`isc status explain [--pane N]` tells you why a pane is in its current state.

### Commands that fail, and short commands

- **A long command is one that ran for 30 seconds.** `long-command` in the settings
  changes that, from 1 second to 3600. The daemon reads it as each command ends.
- **Without shell integration, a failed command shows as `done`, not `failed`.** The end
  of a command is read from the terminal's foreground process, which gives no exit code.
- **With a shell that marks its commands, a failed command is `failed`**, with a note
  such as "failed with 2 after 1m 35s".
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

- **With no setup**, from what agents already write to their terminal: progress,
  notifications and the bell, as described in section 10.
- **Exactly**, from the agent's own hooks, which call `isc status`. insensical can add
  those hooks for Claude Code, Codex, Gemini CLI, opencode and pi.

**Only the Claude Code integration has been used.** The other four are written from
each agent's documentation and source code as of October 2026, and tested with the
events those describe. They haven't been tried against the agents themselves.

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

---

## 12. Notifications

**When you're told**: a pane becomes `waiting`, `done` or `failed`; a program sends a
notification sequence; or `isc notify TITLE [BODY]` is called.

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
  removed once you've seen the pane.
- A pane can send at most three in quick succession, and earns another every second
  and a half. Any beyond that are dropped.
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

- **The icon** is insensical's own, with an amber mark at its corner while any pane
  needs you: one that waits, failed, or finished and hasn't been seen. Point at it for
  how many.
- **Its menu** lists those panes, the most urgent first, each with a sign for its
  state, its project and name, and what it says. Pick one to go to that pane. Nine are
  named and the rest counted. Below them: how many panes are running, and Open
  insensical, which reads Show insensical while a window is open. Clicking the icon
  itself does what that line does.
- **Close the window** is there while a window is open. It closes every window and
  ends nothing: your programs run on, and the icon stays.
- **Quit insensical…** stops the server, which ends every program in every pane. It
  asks first, in a window, opening one if there is none: the question names the panes
  that would end and says what's kept. Nothing stops until you answer **End
  everything**; `Esc` leaves it all running. Your projects and panes are kept, and each
  comes back with a new shell when you open insensical again. If the window it opens
  for the question hasn't come within ten seconds, the question is dropped: choose
  **Quit insensical…** again.
- **With no window open, one opens**, and with one open it's asked to come forward,
  both as for a clicked notification: on Wayland it may not be able to.
- **insensical running in a terminal isn't a window.** **Close the window** leaves it
  open. With only that open, the menu says Open insensical, and picking a pane opens a
  window on it. If it's the interface you used last, **Quit insensical…** asks its
  question there.
- **Not every desktop has a tray.** It needs one that shows status notifier items: KDE
  Plasma, Waybar and most panels do, and GNOME does only with the AppIndicator
  extension. Nothing depends on the icon: notifications arrive without it.
- The icon is found in the installed icon theme. If insensical's icons aren't
  installed, the tray shows a blank or a placeholder.
- `tray = false` takes the icon away, and **Show in the tray** under **Attention** in
  the settings is the same switch. The icon follows it within a couple of seconds.

**On macOS** the application posts them, not the daemon, and only from an app bundle.
They arrive with the window closed too, for as long as the application is running. The
system asks once whether to allow them. Click one to bring the window to the front on
the pane it's about; with no window open, one opens there. None arrive once you quit
the application.

**On macOS, the menu bar and the Dock say what needs you**, so you can tell with the
window closed or behind other work:

- **In the menu bar**, insensical's mark, with the number of panes that need you
  beside it: those that wait, failed, or finished and haven't been seen. It's the
  same number the window shows. With none, the mark is alone; with no server to hear
  from, it's dimmed. It never changes colour and never moves.
- **Its menu** lists those panes, the most urgent first, each with its state's mark,
  the same marks the window draws, its project and name, and what it says. Pick one
  and insensical comes forward on that pane, opening a window if there is none. Nine
  are named and the rest counted; the count opens the overview. Below them: how many
  panes are running, Open insensical, Settings…, Restart… when an update is installed,
  and Quit.
- **On the Dock icon**, the same number as a badge, and the same panes in the menu you
  get by right-clicking the icon. The system can hide a menu bar item when the bar is
  crowded or behind a notch; the Dock always has it. If the number doesn't appear on
  the icon, turn on **Badge application icon** for insensical in System Settings,
  Notifications: macOS shows a badge only where that's allowed.
- **The settings say what macOS allows.** At the top of **Attention**, a line says
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

The menu also has About insensical, which shows its version, `Settings…` (`⌘,`), which
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
  leaves it out, the sheet says `not in your [keys.direct]` beside it, in the colour of
  something waiting for you.
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
- Your keys are added to the defaults, and yours win where the two are the same. The
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

- **If a table is present, it replaces all of that keymap's defaults.** A command it
  doesn't list has no key in that keymap. This is different from `[keys]`, which adds.
- Each table is read only in its own keymap. So when `keymap = "leader"`, chords written
  under `[keys.direct]` aren't bound, and reach the programs.
- `[keys]` is still read in both keymaps, on top of the table.
- Copy, paste, paste-selection, find and the key that locks aren't part of either table.
  They're the same in both keymaps, and you change them in `[keys]`.
- You can't change the leader pressed twice, or Escape after the leader.
- `isc config` prints both tables in full, with every command that has no key shown as a
  line to uncomment. A file made from that output lists the commands of the version that
  printed it. A command added by a later version has no key until you add a line for it; the sheet of
  keys marks each one your table leaves out.
- A command or key that can't be read is reported in the top bar, which names the table.

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

- A pane shows its icon, its name, and a border in the colour of its state. Under the
  name is the memory its programs hold, once that's a megabyte or more.
- A pane that's waiting, done or failed also shows what it says about that, so you can
  read an agent's question without going there.
- A project's name is followed by the number of its panes that need you.
- The overview opens with the tab in front marked "here". Left and Right go through
  every tab in order, across projects. Up and Down go to the nearest tab in the row
  above or below. All four stop at the ends.
- Press Tab to show only the tabs that have a pane that needs you, and Tab again to
  show every tab. A project with no such tab is left out, and if there are none at all
  the overview says "Nothing needs you."
- Press Enter, or click a tab, to go there. Escape, or a click anywhere else, closes the
  overview and changes nothing.
- It doesn't resize anything and doesn't attach to any pane. It's drawn from what the
  daemon already reports about each pane.
- It doesn't show what's on the panes' screens.

### Finding a file

Press `Ctrl+Shift+I` or leader `/`, or choose "Find a file under the pane's directory
and type its path" in the palette.

A small list opens at the terminal's cursor. It shows the files and directories under
the focused pane's directory, each with an icon for its kind. Type a few letters of the
one you want, then press Enter to type its path into the pane.

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
- At most 30 000 files and directories are read. In a directory with more, such as a
  home directory, only the nearest 30 000 can be found.
- The directory is where the pane's shell is, not where a program running in the shell
  has gone.
- The list opens under the cursor's row, or above it when there's no room underneath.
- **What git ignores isn't listed.** Inside a git repository the list leaves out
  everything git is told to ignore, by any `.gitignore`, your own excludes or the
  repository's. Git itself is asked, once, as the list opens. Outside a repository, or
  without git, nothing is left out but the directories named above.
  - Git is told to run none of the programs a repository's settings can name, to fetch
    nothing, and to write nothing.
  - Git has three seconds to answer. After that the files are listed without it.

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
- The program's name is the one in the pane's header and in `isc ls`.

### The settings sheet

Press `Ctrl+,` (`⌘,` on macOS), or leader `,`, or choose "Settings" in the palette. A
sheet opens over the window with every setting on it. Nothing on it is typed as a
value: a setting is switched, picked or stepped. A number steps by one, the
background's opacity by 0.05, and `long-command`, which goes up to 3600 seconds, in
fives.

- **Sections are down the left**, in a column of their own with the field for finding a
  setting above them: Appearance, Attention, Terminal, Keys, Agents, and Changed. Each
  has a tile in its own colour. A number beside a section says how many of its settings
  your file changes. The sheet opens on the section you last looked at, and its heading
  is that section's.
- **A change takes effect at once** and is written to `config.toml`, to that
  setting's one line. The window behind the sheet stays in view, so you see a theme,
  a font or a density change as you make it.
- **A setting your file changes** has a bar at the left edge of its row and a `Reset`
  button. Reset takes the line out of the file. **Changed** lists all of them.
- **Type to find a setting** in any section, by its name or what it does. Each row
  found says which section it's from.
- A file edited by hand while the sheet is open shows on the sheet as it's saved.
- Mistakes in the file are listed at the top of the sheet. The settings that could be
  read still work.
- The file's path is at the foot. Click it to open the file.
- **The list for the terminal's font has only fonts of one width**, since a terminal is
  set in cells. Any other font can still be named in the file.
- A section with more settings than fit fades at its foot. Scroll for the rest.

| Control | For | Change it |
|---|---|---|
| A switch | on or off | Click it, or Enter |
| Words side by side | one of a few choices | Click one, or Enter for the next |
| Ticks side by side | any of a few, such as what the sidebar lists | Click one to tick or untick it, or Enter to go through every combination |
| `−` number `+` | text size, space between panes | Click `−` or `+`, or `Ctrl+Right` and `Ctrl+Left` |
| A value with `›` | a theme, a font, a shell | Click, or Enter, for a list in place of the rows. Type to narrow it. Enter chooses, Esc goes back. |
| Key caps | the leader | Click, or Enter, then press the key. It needs Ctrl, Alt or Cmd held. Esc cancels. |

- A theme is shown on the whole window as you move through the list, before you choose
  it. Esc puts back the one you had.
- The font lists hold every font the system has. The shell list holds the shells in
  `/etc/shells`.
- **Agents** isn't settings of insensical's. It lists each agent it knows, and whether
  the agent reports what it's doing: it does, it doesn't yet, its hooks have gone
  stale (they name an `isc` that's no longer there), or the agent isn't on this
  system. `Set up…` asks before adding hooks, as in [section 11](#11-agents).
- **Keys** has the keymap and the leader. The keys themselves are tables in the file:
  the sheet has buttons for the sheet of every key and for opening the file.

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
- A value that can be read but not used is reported, and only that setting falls back.
  That covers a font size of 500, a theme that doesn't exist, a leader that isn't a key,
  and a shell that isn't a program.
- The top bar shows only the first problem. All of them are printed to the window's
  standard error.

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
- The accent is the palette's blue. The state colours are its blue, yellow, green and
  red.
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
`isc status explain`, and so does `isc restart-server` when no daemon is running. These
never start it: `daemon`, `kill-server`, `version`, `status` with a state, `notify`,
`integrate`, `shell-integration`, `config`, `completions`, and the agents' hooks.

### Listing

`isc ls` prints projects, tabs and panes. `isc list` is the same command. It marks the
current project, tab and focused pane with `*`. A project's line shows its number, name
and root, and `asleep` while it sleeps. A tab's line shows its number, its name if it
has one, and `zoomed` while a pane fills it. A pane's line shows its number, size in
columns and rows, placement (`tiled`, `floating` or `minimized`), state, program, and
name or title. If an update has been installed under the running daemon, `isc ls` says
so on standard error.

```text
* project:5  api  /home/you/code/api
    tab:4  zoomed
      * pane:3  120x40  tiled  running  claude  Refactor the auth middleware
        pane:6  120x40  tiled  idle  nvim  src/session.rs
  * tab:8  servers
      * pane:10  80x24  floating  idle  fish  
        pane:7  96x30  minimized  waiting  cargo  
  project:9  web  /home/you/code/web  asleep
      tab:12
        pane:11  80x24  tiled  idle  fish  
```

`isc ls --json` prints everything the daemon knows. That's the stable format for
scripts. No other command has `--json`, and no command speaks the daemon's protocol
directly.

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

### Typing into a pane and reading it

- `isc send PANE TEXT` types text.
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
succeeds when the pane's screen shows `TEXT`:

```sh
pane=$(isc run -- npm run dev)
isc wait "${pane#pane:}" --output "ready in" --timeout 60 && xdg-open http://localhost:5173
```

- It looks at the screen as it is, so it succeeds at once if the text is already there,
  and doesn't see text that has scrolled off.
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
- `isc api '"?"'` fails and lists the name of every request. What each takes is as the
  commands in this manual describe; `isc ls --json` shows the shapes of panes, tabs and
  rectangles.
- The requests can change between versions. The commands are the steadier way to
  script; `isc api` is for what they don't cover.

### Events

`isc events` prints what happens, one JSON object per line. Each has an `event` and
usually a `pane`.

| `event` | Other fields |
|---|---|
| `opened`, `status` | `status`, `note`, `progress`, `program` |
| `closed` | |
| `focus` | `pane` is null when no pane has the keyboard |
| `bell` | |
| `notification` | `title`, `body` |
| `activate` | You picked the pane from a notification |

- If you stop reading, as when the output is piped to a pager, the daemon keeps a
  limited amount for you, 8 MB. Past that it closes the connection and `isc events`
  ends; run it again. A window that was stopped for long enough to leave that much
  unread loses its connection the same way, and finds the daemon again by itself
  when it runs on.
- A program that sends the daemon requests and never reads the answers loses its
  connection too, once more than 8 MB of answers wait behind the first.
- A program that rings its bell, or changes its title or progress, many times in a
  moment is reported at most twenty times a second, always with what it said last.

### Attaching from a terminal

`isc attach [PANE]` shows a pane in the terminal you run it in, and sends what you type
to it. Press `Ctrl+]` to leave.

- The terminal gets the pane's history and then the program's own output. Nothing is
  redrawn or re-encoded on the way, so the terminal's own scrollback, selection and
  search work on it.
- That means every escape sequence the program prints reaches your terminal as it
  would if you had run the program there: titles, links, clipboard requests and all.
  `isc attach` is a terminal for the pane, and it filters nothing. Use `isc capture`
  to read what a pane you don't trust has printed.
- Whatever was on the terminal's screen before moves up into its history. When you
  leave, the terminal is restored: mouse reporting off, cursor shown, and so on.
  Leaving doesn't affect the pane or its program.
- **The pane takes that terminal's size**, for everyone looking at it, and follows it
  when the terminal is resized, until you go back to a window that shows the pane. See
  [Two windows](#two-windows).
- You reach a pane on another machine the same way:

  ```sh
  ssh build-host isc attach 3
  ```

### The whole interface in a terminal

`isc`, with nothing after it, runs insensical in the terminal you run it in. It's for
a machine that has no desktop, and for one you reach over `ssh`:

```sh
ssh build-host -t isc
```

- `isc tui` is the same thing said in full.
- `isc --help` lists the commands. So does `isc` alone where there's no terminal to
  draw in (a script, a pipe), and inside one of insensical's own panes, where the
  interface would show itself.

It's another client of the same daemon. What you do in it shows in a window at once, and
the other way round. Leaving it ends nothing.

It's a terminal program and looks like one.

- **The top row** has the tabs of the project in front, each with its number. The one
  in front is a pill. At the right end are the counts of what finished and what needs
  you, and the leader's key.
- **The list at the left** is your projects, one to a row, each with the state of
  what runs in it or how many panes it has. Under 76 columns it's left out.
- **Each pane is in a box.** Its name is in the top, and its state at the end of the
  top. The focused pane's box is in the theme's accent colour, and a waiting one in its
  waiting colour: your terminal's blue and yellow under the `terminal` theme.
  Panes aren't listed anywhere: they're on the screen.
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
[the leader keymap](#the-leader-keymap): `Ctrl+Space` then `T` opens a tab, `V` and `S`
split, `H` `J` `K` `L` move between panes. While the leader waits, a panel at the foot
lists every key that can follow. It stays until you press one, however long you take;
`Esc` closes it. The line at the foot of the screen names the keys for the palette and
for the list of keys, so you don't have to remember the leader.

- **`Ctrl+Space` then `D` leaves.** The terminal is given back as it was.
- `Ctrl+Space` then `Space` is the palette, and `Ctrl+Space` then `Shift+/` lists every
  key, under the same headings as the window's sheet.
- [The palette](#the-palette) lists what the window's lists, under the same headings,
  and typing finds things the same way. A few things differ here. It lists no themes.
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
  apart. The line at the foot names the keys, as you've bound them. `Ctrl+Alt+L` works
  too, where your desktop doesn't keep it for locking the screen.
- While they're locked, the leader goes to the program together with the key you press
  after it, not before.
- `Ctrl+Z` goes to the program in the pane. To stop the interface itself, send it
  `SIGTSTP` (`kill -TSTP`): it gives the terminal back while it's stopped, and draws
  everything again when you bring it back with `fg`.
- **Closing a tab, closing a project and putting a project to sleep ask first**, with
  the question the window asks, and only when something would end.
- **A project is renamed and closed from the palette**: "Rename this project" and
  "Close this project and everything in it". They're the commands `rename-project` and
  `close-project`, which have no default key.
- **A project that's asleep** says so where its panes would be, and `Enter` wakes it.
- **If the server becomes another version** while you're in it (after an update and a
  restart), it says so: leave, and run `isc` again.
- **If you choose to stop everything from the tray** while this is the interface you
  used last, it asks you here what the window would ask, and stops the server if you
  say so.

**The mouse** goes to the pane you press, selects text, scrolls, and drags the edge
where two panes meet. Press a tab, a project or something in the dock to go there.

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
`interface-style`, `motion`, `attention-glow`, `keymap` and `leader`, and holds the two
settings that only a terminal has:

```toml
[tui]
theme = "terminal"     # or any of those six, to differ here
colours = "auto"       # "truecolor", "256" or "16"
mouse = true
```

- The window never reads `[tui]`, whatever is in it.
- `colours` and `mouse` are read from `[tui]` only. At the top of the file they're a
  key the window doesn't know, and this interface tells you where they go.
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
`interface-style`, `motion`, `attention-glow`, `keymap`, `notifications` and the three
`notify-` settings, `long-command`, `sound`, `tray`, `scrollback`, `term`, `clipboard`
and `keep-scrollback`, and under "This terminal", `colours` and `mouse`. It shows a
value from the `[tui]` table where that table has one, and changes it there. `theme`,
`leader`, `shell` and your own keys are set in the file. The window's density, fonts
and sidebar settings have no meaning here and aren't listed.

**What it doesn't do.**

- Pictures aren't shown.
- A pane has no buttons to press. Closing, zooming, floating and putting away are
  commands. Double-click a pane's name to fill the tab with it.
- Nothing is dragged into another place with the mouse: not a pane within its tab,
  nor a pane to another tab, nor a project into another order. The move commands do
  the first, and `isc put` and `isc project move` the other two.
- A project's icon isn't chosen here, and the question about agents isn't asked: the
  `integrate` command opens `isc integrate` in a floating pane.
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

  Asking never starts the daemon. If none is running, you still get commands and
  options, but not panes, tabs or projects. It asks the daemon at the default socket or
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
  as it is and the command exits with status 1.

---

## 17. Stopping, restarting and new versions

### What survives what

| What happens | Programs | Screens and scrollback | Layout, directories, names |
|---|---|---|---|
| The window is closed | keep running | kept | kept |
| The window is opened again | — | shown as they are | — |
| The daemon moves to a new version (restart from the window, `isc restart-server`) | keep running | kept | kept |
| The daemon is stopped (`isc kill-server`, logout, reboot) | **end** | kept, as text and colour | kept |
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
One such file is kept: a later one takes its place. To go back to it, stop the server
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
- The window shows "Restarting…" for a moment, then shows everything as it was.
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
  starts in place of the running one. This was seen once, on a Mac, with a window open
  as the server was moved from a terminal.
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
- It looks for a daemon once a second and carries on by itself when one appears. For
  instance, after you run `isc restart-server` in a terminal, it shows the same
  programs still running.
- Press Enter to start one.
- If the daemon that appears is a different version from the window, the window starts
  the installed application in its place. If it can't, it tells you to close and
  reopen.

### Different versions of `isc`, window and daemon

- They need the same protocol and the same terminal engine to work together. A
  mismatch is refused with a message that names `isc restart-server`.
- A few things work regardless: `isc version`, `isc status`, `isc notify`, the agents'
  hooks, `isc kill-server`, `isc restart-server`.
- A daemon too old to answer those, or one that's stuck, gets three seconds and is
  then stopped with a signal.

---

## 18. When something goes wrong

### Starting, stopping and versions

| What is seen | Why | What to do |
|---|---|---|
| "The insensical server cannot be reached" | No daemon is running and none could be started: `isc` isn't beside the application or on `PATH`, or the socket's directory isn't private | Put `isc` beside `insensical`; check the directory's permissions |
| "insensical was updated, and the server still running is from before", at launch | A new version was installed under a running daemon | Press Enter to restart it. Everything keeps running; if the daemon is too old to make way, programs end and the layout comes back. Or press Escape to leave it |
| "The server is already the new version, and this window is from before it", at launch | The server was moved to a new version while an older application was still running | Press Enter to open the new application, or close the window and open insensical again. Nothing that's running ends |
| `isc: protocol N does not match the running daemon's M` | The same, from the command line | Run `isc restart-server` |
| "The server has stopped" in the window | The daemon was stopped or crashed | Press Enter to start it; the layout comes back |
| "The server did not move to the new version" in the window, or `isc: the daemon was not moved` | The installed `isc` is missing from where the daemon was started, or can't take over what it runs | Nothing was lost. Press Enter to go back to it. Reinstall; or run `isc kill-server` and start again, which ends what's running |
| `daemon: running, but it does not say what it is`, from `isc version` | The daemon is from before the requests that every version answers, or it's stuck | Run `isc restart-server` |
| `a daemon is already running` | `isc daemon` was run twice | Nothing; or run `isc kill-server` first |
| `path must be shorter than SUN_LEN` | The socket's path is too long | Set a shorter `INSENSICAL_SOCKET` |
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
| A pane stuck in a state | A hook reported it and nothing has reported since | `isc status explain`, `isc status idle` |
| A Gemini CLI pane stays `running` after a failed or cancelled turn | Gemini CLI has no event for either | The next prompt clears it; or run `isc status idle` |
| Hooks stopped reporting | `isc` moved since `integrate` | Run `isc integrate` for that agent again |
| Codex reports nothing | Its hooks aren't trusted | Run `/hooks` in Codex |
| The bell marks a pane but no notification arrives | The bell never notifies | Nothing |
| A clicked notification selects the pane but the window stays behind | Wayland; see section 12 | Bring the window forward yourself |
| Notifications have no application name or icon | The desktop entry isn't installed | Install |
| The tray's icon is blank | insensical's icons aren't installed | Install |
| There's no icon in the tray | The desktop has no tray for status notifier items, or `tray` is off | On GNOME, add the AppIndicator extension |
| The leader does nothing | The desktop took `Ctrl+Space` | Pick another `leader` |

---

## 19. Reference

### Keys

In the macOS column, the keys for tabs, splits, panes, zoom, float, the sheet of keys,
a new project, the path picker, the lock, the settings, the overview and quit have been
pressed on a Mac. The rest are written and have never been run. "—" means there's no
default key: the command is in the palette, and you can bind it in `[keys]`. The
Command column is the name to use in `[keys]`. A capital letter after the leader means
the letter with Shift, written `shift-h` in `[keys]`.

| Action | Command | Linux | macOS | After the leader |
|---|---|---|---|---|
| New tab | `new-tab` | `Ctrl+Shift+T` | `⌘T` | `t` |
| New project | `new-project` | `Ctrl+Shift+N` | `⌘⇧N` | `o` |
| Find a project, tab or pane in the sidebar | `filter` | `Ctrl+Shift+L` | `⌘K` | `e` |
| Close pane | `close-pane` | `Ctrl+Shift+W` | `⌘W` | `x` |
| Close tab | `close-tab` | `Ctrl+Shift+Q` | `⌘⇧W` | `q` |
| Split right | `split-right` | `Ctrl+Shift+O` | `⌘D` | `v` |
| Split down | `split-down` | `Ctrl+Shift+E` | `⌘⇧D` | `s` |
| Focus the pane left, down, up, right | `focus-left`, `focus-down`, `focus-up`, `focus-right` | `Ctrl+Alt+arrow` | `⌘⌥arrow` | `h` `j` `k` `l`, or the arrows |
| Move the pane's edge left, down, up, right | `resize-left`, `resize-down`, `resize-up`, `resize-right` | `Ctrl+Alt+Shift+arrow` | `⌘⌃arrow` | `H` `J` `K` `L` |
| Move the pane left, down, up, right | `move-left`, `move-down`, `move-up`, `move-right` | — | — | `Ctrl+H` `Ctrl+J` `Ctrl+K` `Ctrl+L` |
| Next / previous pane | `next-pane`, `previous-pane` | `Ctrl+Shift+]` / `Ctrl+Shift+[` | `⌘]` / `⌘[` | `]` / `[` |
| Next / previous tab | `next-tab`, `previous-tab` | `Ctrl+Tab` / `Ctrl+Shift+Tab`; `Ctrl+PageDown` / `Ctrl+PageUp` | `Ctrl+Tab` / `Ctrl+Shift+Tab`; `⌘⇧]` / `⌘⇧[` | `n` / `p` |
| Go to the project's first to ninth tab | `tab-1` to `tab-9` | `Alt+1` to `Alt+9` | `⌘1` to `⌘9` | `1` to `9` |
| Next / previous project | `next-project`, `previous-project` | `Ctrl+Alt+PageDown` / `Ctrl+Alt+PageUp` | `⌘⌥]` / `⌘⌥[` | `N` / `P` |
| Move the tab left / right | `move-tab-left`, `move-tab-right` | `Ctrl+Shift+PageUp` / `Ctrl+Shift+PageDown` | the same | `<` / `>` |
| Fill the tab with the pane, or stop | `zoom` | `Ctrl+Shift+Enter` | `⌘⇧Enter` | `z` |
| Show or hide the tabs, wherever the settings put them | `sidebar` | `Ctrl+Shift+B` | `⌘B` | `b` |
| Float the pane, or put it back | `float` | `Ctrl+Shift+G` | `⌘⇧G` | `f` |
| New floating pane | `new-float` | `Ctrl+Shift+D` | `⌘⇧F` | `F` |
| Keep a float on top, or stop | `pin` | `Ctrl+Shift+K` | `⌘⇧K` | `i` |
| Show a float over every tab, or over its own | `everywhere` | — | — | `I` |
| Hide or show all floating panes | `floats` | `Ctrl+Shift+H` | `⌘⇧H` | `g` |
| Bring the scratch pane, or put it away | `scratch` | `` Ctrl+` `` | `` ⌃` `` | `` ` `` |
| Minimize the pane | `minimize` | `Ctrl+Shift+M` | `⌘M` | `m` |
| Restore the pane minimized last | `restore` | `Ctrl+Shift+R` | `⌘⇧M` | `r` |
| Go to the pane that most needs you | `next-attention` | `Ctrl+Shift+A` | `⌘⇧A` | `a` |
| Command palette | `palette` | `Ctrl+Shift+P` | `⌘⇧P` | `Space` |
| Put this project to sleep, or wake it | `sleep-project` | — | — | — |
| Rename this project | `rename-project` | — | — | — |
| Close this project and everything in it | `close-project` | — | — | — |
| Size the panes for this window, when another window or a terminal sized them last | `lead` | — | — | — |
| Name this pane | `name-pane` | — | — | — |
| Name this tab | `name-tab` | — | — | — |
| Show every key | `keys` | `Ctrl+Shift+/`; `Ctrl+F1` | `⌘?` | `?` |
| Settings | `settings` | `Ctrl+,` | `⌘,` | `,` |
| See every tab of every project | `overview` | `Ctrl+Shift+Space` | `⌘⇧O` | `w` |
| Find a file and type its path | `pick-path` | `Ctrl+Shift+I` | `⌘⇧I` | `/` |
| Restart insensical, to move to a new version | `restart` | `Ctrl+Alt+Shift+R` | `⌘⌃⇧R` | `U` |
| Quit the window; what is running carries on. On macOS with insensical in the menu bar, put its windows away and leave it there | `quit` | — | `⌘Q` | — |
| Give the leader's key to the program | | | | the leader |
| Take the leader back | | | | `Esc` |
| Copy the selection | `copy` | `Ctrl+Shift+C` | `⌘C` | same chord |
| Paste | `paste` | `Ctrl+Shift+V` | `⌘V` | same chord |
| Paste what was last selected | `paste-selection` | `Shift+Insert`, middle button | — | same chord |
| Find text in the pane | `search` | `Ctrl+Shift+F` | `⌘F` | same chord |
| Scroll to the command before | `previous-prompt` | `Ctrl+Shift+Up` | `⌘↑` | same chord |
| Scroll to the command after | `next-prompt` | `Ctrl+Shift+Down` | `⌘↓` | same chord |
| Give every key to the program, or take them back | `lock` | `Ctrl+Alt+Shift+L` | `⌘⌃⇧L` | same chord |
| Open a web address | | `Ctrl`+click | `⌘`+click | |

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
| `select-all` | Selects everything in the pane |
| `integrate` | Gets the agents on this system to report what they're doing |
| `none` | Not a command. In `[keys]`, it leaves the key to the program. |

After the leader, these commands repeat without it for one second: `resize-left`,
`resize-right`, `resize-up`, `resize-down`, `move-left`, `move-right`, `move-up`,
`move-down`, `move-tab-left` and `move-tab-right`.

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
| `tabs` | Where the tabs show. `sidebar`: down the side, for every project. `top`: along the top, for the current project. | `sidebar` |
| `pane-header` | `full`: what the pane is and what it's doing, plus buttons to arrange it. `name`: the same without the buttons. `hidden`: no header; a floating pane keeps its name. | `full` |
| `keymap` | `direct`: each command has its own chord. `leader`: each command is a key you press after the leader, which leaves the chords to the programs in the panes. | `direct` |
| `leader` | The key you press before a command's key when `keymap` is `leader` | `ctrl-space` |
| `shell` | What a pane runs when you give it no command, with any arguments: `"fish"`, `"/bin/zsh -l"`. The daemon reads it when a pane starts. | the login shell, `$SHELL` |
| `input-method` | Whether your system's input method may compose text in a pane, as for Bangla, Chinese or Japanese. What it's composing shows at the cursor; the program gets the finished text. | `false` |
| `option-as-alt` | On a Mac, whether Option with a key is Alt to the program in the pane, as a shell's Alt+B and Alt+F are, and no longer types the character the keyboard layout gives it. It does nothing elsewhere. | `false` |
| `open-files-with` | What a file you `Ctrl`+click in a terminal opens in: `system`, an editor by name (`code`, `cursor`, `zed`, `subl`, `idea`, `nvim`, `vim`, `hx`, `nano`), or a command with `{file}` in it. See [Web addresses and paths](#web-addresses-and-paths). | `system` |
| `term` | What a pane's program is told the terminal is, in `TERM`. `xterm-ghostty` gives programs every feature. `xterm-256color` is known on every machine, which matters over `ssh`, and has fewer. Read by the daemon when a pane starts. | `xterm-ghostty` |
| `scrollback` | How much history each pane keeps, in megabytes, 1 to 200. Read by the daemon when a pane starts. | `10` |
| `keep-scrollback` | Whether what panes show is saved to disk when the daemon stops, so it's there above a new shell the next time the daemon starts. Read by the daemon. With `false`, nothing is saved, and anything saved earlier is removed as panes close. | `true` |
| `clipboard` | Whether a program can put text on the clipboard. `focused`: only the program in the pane you're looking at. `never`: no program. No program can ever read what's on the clipboard. Read by the daemon. | `focused` |
| `notifications` | Whether you're told when a pane has something for you. Read by the daemon. | `true` |
| `attention-glow` | Whether a waiting pane takes the waiting colour at its edge and casts a soft light of it into the gaps around it. | `true` |
| `notify-waiting` | Whether you're told when a pane waits for you, or a program asks for you. Read by the daemon. | `true` |
| `notify-failed` | Whether you're told when a command or an agent ends badly. Read by the daemon. | `true` |
| `notify-done` | Whether you're told when a long command or an agent's turn ends well. Read by the daemon. | `true` |
| `long-command` | How many seconds a command runs before its end is news: the pane is marked done or failed, and you're told if you weren't looking. 1 to 3600. Read by the daemon. | `30` |
| `sound` | Which notifications make a sound. `needs-you`: a pane that waits or failed. `all`: those, and a softer sound for a finish. `off`: none. | `needs-you` |
| `tray` | On Linux, whether insensical has an icon in your desktop's tray, marked while a pane needs you, with a menu of those panes. Read by the daemon. It does nothing on a Mac. | `true` |
| `menu-bar` | On a Mac, whether insensical has an item in the menu bar that counts and lists the panes that need you. It does nothing elsewhere. | `true` |
| `dock-badge` | On a Mac, whether the number of panes that need you is on insensical's Dock icon. It does nothing elsewhere. | `true` |
| `[tui]` | What `isc tui` does differently from the window, which never reads this table: `theme` (where `terminal` is the terminal's own colours), `interface-style`, `motion`, `attention-glow`, `keymap` and `leader`, each in place of the one at the top of the file, and its own `colours` (`auto`, `truecolor`, `256`, `16`) and `mouse` | `colours = "auto"`, `mouse = true` |
| `[keys]` | Keys and the command each one runs, or `none`, in either keymap | empty |
| `[keys.direct]` | The whole direct keymap, written out. When present, it replaces the defaults. | the keys in the table above |
| `[keys.leader]` | The whole leader keymap, written out. Each key is the one you press after the leader. When present, it replaces the defaults. | the keys in the table above |

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
`--socket PATH`.

| Command | What it does |
|---|---|
| `isc ls [--json]` | Lists projects, tabs and panes, with each pane's placement and state. `--json` prints everything the daemon knows. `isc list` does the same. |
| `isc run [--cwd DIR] [--keep] [WHERE] [-- COMMAND…]` | Runs a command in a new pane, or the shell if you give none, and prints the pane as `pane:N`. `--cwd` sets the working directory; the default is the one you run `isc` in. With `--keep`, the pane stays when the command ends. Press Enter in it, or use `isc close`, to close it. A command and the directory it starts in can be 256 KB together at most. |
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
| `isc project new [DIR] [--name NAME]` | Opens a project for a directory, or for the current one if you give none. It's named after the directory unless you set `--name`. Prints its first pane. |
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
| `isc notify TITLE [BODY] [-p PANE]` | Asks for your attention on a pane's behalf |
| `isc mute [PANE] [--off]` | Stops a pane from sending notifications. `--off` lets it send them again. Its state still shows. |
| `isc integrate [claude\|codex\|gemini\|opencode\|pi] [--remove] [--yes]` | Adds an agent's hooks, after showing you the change and asking. With no agent named, it offers each agent found on this system. `--remove` takes the hooks out. `--yes` skips the question. |
| `isc integrate [AGENT] --status` | Changes nothing. Prints, as JSON, for each agent: whether it's found, whether it already reports, the file its hooks go in, and what's left for you to do afterwards. |
| `isc wait PANE [--until STATE]… [--timeout SECONDS]` | Blocks until the pane reaches a state: `idle`, `running`, `waiting`, `done`, `failed` or `closed`. Without `--until`, the states are `done`, `failed`, `waiting` or `closed`. Prints the state reached. The exit status is 124 when the time runs out. |
| `isc wait PANE --output TEXT [--timeout SECONDS]` | Blocks until the pane's screen shows the text. Exits with 124 if the time runs out. |
| `isc events` | Prints what happens, one JSON object per line, until interrupted |
| `isc api [REQUEST]` | Sends the daemon one request as JSON, from the argument or standard input, and prints the answer as JSON |
| `isc project sleep NAME` | Ends every program in the project and keeps its tabs, panes and what they showed. Doesn't ask. |
| `isc project wake NAME` | Puts a shell in each pane of a sleeping project, in the pane's directory |
| `isc project mute NAME [--off]` | Stops every pane of the project from sending notifications, including panes opened in it later. `--off` lets them again. |
| `isc usage [--json]` | Prints what the programs in each pane are using: memory, processor time and how many processes, with a total for each project |
| `isc config` | Prints a settings file with every setting at its default and every key of both keymaps |
| `isc config get NAME` | Prints what a setting is: what the file gives it, or its default |
| `isc config set NAME VALUE` | Gives a setting a value in the settings file, changing only that line |
| `isc config reset NAME` | Takes a setting out of the settings file, which leaves its default |
| `isc completions bash\|zsh\|fish` | Prints what that shell needs to complete `isc` commands, including the panes, tabs and projects that exist |
| `isc shell-integration bash\|zsh\|fish` | Prints what that shell needs to tell its pane where each command starts and ends, and its exit status. fish 4 and later doesn't need it. |
| `isc daemon` | Runs the daemon in the foreground |
| `isc kill-server` | Stops the daemon and every program it runs, and returns once the daemon is gone. The layout comes back the next time it starts. It also works on a daemon of another version. |
| `isc restart-server [--yes]` | Moves the running daemon to this version while the programs in its panes keep running. It asks first; `--yes` skips the question. |
| `isc version` | Prints this version and the running daemon's, with the protocol and terminal engine of each, the daemon's process number, and whether they can talk to each other |

### Files and directories

On a Mac, the settings, the saved layout, the window's size, the terminal's description
and the socket have been found where the macOS column says. The other rows are
written and have never been run.

| What | Linux | macOS |
|---|---|---|
| Settings | `~/.config/insensical/config.toml` (`$XDG_CONFIG_HOME`) | the same |
| Your own themes | `~/.config/insensical/themes/<name>.toml` | the same |
| Your own drawings for projects' icons | `~/.config/insensical/icons/<name>.svg` | the same |
| Saved layout | `~/.local/state/insensical/state.json` (`$XDG_STATE_HOME`) | `~/Library/Application Support/insensical/state.json` |
| A saved layout that could not be read, set aside | `~/.local/state/insensical/state.json.unreadable` | `~/Library/Application Support/insensical/state.json.unreadable` |
| What each pane showed when the daemon stopped | `~/.local/state/insensical/screens/<pane number>` | `~/Library/Application Support/insensical/screens/<pane number>` |
| Agents declined | `~/.local/state/insensical/agents-declined` | `~/Library/Application Support/insensical/agents-declined` |
| The window's size, whether its tabs were hidden, and what was folded in the sidebar | `~/.local/state/insensical/window` | `~/Library/Application Support/insensical/window` |
| What was chosen lately in the palette | `~/.local/state/insensical/palette-recent` | `~/Library/Application Support/insensical/palette-recent` |
| The terminal's description, when the daemon had to compile its own | `~/.local/state/insensical/terminfo/` | `~/Library/Application Support/insensical/terminfo/` |
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
| `INSENSICAL_NOTIFICATIONS` | daemon | `clients` makes the daemon leave desktop notifications to its windows, and put no icon in the tray. |
| `INSENSICAL_ISC` | window, `isc` | The `isc` binary to start the daemon from. When set, it's used before the `isc` beside the program and the one on `PATH`. |
| `SHELL` | daemon | What a pane runs when there's no command and no `shell` setting. `/bin/sh` if unset. |
| `TERMINFO`, `HOME` | daemon | Searched for Ghostty's terminal description. |
| `TERMINFO_DIRS` | daemon | Kept, after the daemon's own directory, when it points a pane at the description it compiled. |
| `XDG_CONFIG_HOME` | window, daemon, `isc` | Where the settings directory is, instead of `~/.config`. Ghostty's and opencode's directories are also looked for here. |
| `XDG_STATE_HOME` | daemon, window | Where the state directory is, instead of `~/.local/state`. Linux only. |
| `XDG_RUNTIME_DIR` | everything | Where the socket's directory is, on Linux. |
| `TMPDIR` | everything | Where the socket's directory is, on macOS. |
| `PATH` | daemon, window, `isc` | Every pane inherits the daemon's. The window's is searched for `isc` and for the program that `shell` names. `isc integrate` searches its own for agents. |
| `CLAUDE_CONFIG_DIR`, `CODEX_HOME`, `GEMINI_CLI_HOME`, `PI_CODING_AGENT_DIR` | `isc integrate` | Where each agent keeps its settings, instead of its default directory. |
| `GHOSTTY_SOURCE_DIR` | the build | A local Ghostty checkout to build from, instead of downloading. |
| `PREFIX` | `just install-local` | Where to install, instead of `~/.local`. |

Set in every pane: `TERM`, `COLORTERM=truecolor`, `TERM_PROGRAM=insensical`,
`TERM_PROGRAM_VERSION`, `INSENSICAL_PANE`, `INSENSICAL_SOCKET`.
