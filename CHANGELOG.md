# Changelog

What changed in each release of insensical, newest first. Each release's section is also
its release notes.

## Unreleased

## 0.8.0 - 2026-10-28 01:55 +06:00

### New

- **At login.** Settings > Interface can open insensical when you log in, or start it in the tray (the menu bar on a Mac).

### Improvements

- A copy built from source puts itself on your Linux machines, even over a release of the same version.
- insensical is smaller: 42 MB installed where it was 50, and `isc` 25 where it was 28.
- Tidier settings in the window and on the page: one style of switch, a grey Disconnect, "connected 1h ago" on each connection and "Alert me only when I'm away"; on the page, each machine's state with the hub tagged, and Settings named for its machine.
- The settings dialogs and Add Remote Host show `Esc` and `Enter` on their answers, as the window's other dialogs do.

### Bug fixes

- Each uploaded path is followed by a space, so the next upload or word doesn't run into it.
- `isc project new` refuses a directory that isn't there unless you add `--make`, where it used to open in your home directory; `.` and a bare name are from where you are.
- On a phone, the page's This browser sheet no longer scrolls sideways.
- On a Mac, the settings file says `⌘,` opens the settings.

## 0.7.0 - 2026-10-21 02:30 +06:00

### New

- **`isc hub start` and `stop`.** `isc hub start` turns the hub on and keeps it running, and `isc hub stop` turns it off. They replace `isc hub autostart`. On a Mac, launchd keeps it running and the Mac stays awake while it's on power.
- **An iPhone's home-screen app pairs itself.** In Safari, press Add where the page offers it, and the app opens already paired.
- **The Hub page has Restart Hub**, and a joined machine has the hub's switch too.

### Improvements

- The page connects to every machine you have, so Home shows all their projects without waiting.
- The page lists connections by machine, and who's connected by session.
- A machine that runs another version of insensical sends a notice, and the page says what to run to fix it.
- Projects start folded in the sidebar, new ones too, and the page remembers what you folded on a joined machine.
- `isc hub`, `isc hub status` and `isc version` say what the hub is doing, and a copy built from source shows its commit.
- A joined machine's hub follows an update by itself.
- Agents on other machines can't use a computer until you turn on "Can agents on other machines use this computer?" under Agents in the settings.
- `isc agent tell` works with Claude Code, Codex, opencode, pi and Hermes.
- Small icons (the tray, notifications and the browser tab) show the terminal chip alone, and notifications show insensical's icon on Linux.

### Bug fixes

- A restart no longer leaves your hub down, and `isc hub start` with new options keeps it running.
- The window stays responsive while the server is replaced after an update.
- A machine that leaves its hub (`isc hub leave`, `isc hub rekey`) is gone from it.
- A message waiting for an agent's turn to end is no longer lost.
- Keeping agents out now holds across machines with `isc --host` and `isc tui --host`.
- `isc hub tailscale on` no longer takes over another program's Tailscale mapping.

## 0.6.0 - 2026-10-20 00:59 +06:00

### New

- **insensical updates itself.** `isc upgrade` installs a new version over the one you're running. 0.5.0 can't, so to get this release, update the way you installed 0.5.0.
- **Update checks.** insensical looks once a day for a new release and the window says when one is out, with What's New, Update and Skip. `check-for-updates = false` stops the daily look.
- **Update remote hosts from here.** Hosts that run an older version say so as they connect, and Update Them brings each to this computer's version. It works on a Mac and with package or Homebrew copies.

### Improvements

- `install.sh` checks the release's signature when `minisign` is installed.
- Tab completion offers `--host`, `--socket` and `--rights` only where a command takes them, and completes devices, machines, commands, panes, tabs and projects for the hub commands.
- The settings sheet has a close button.

### Bug fixes

- A hub with machines joined to it can no longer join another hub, which left them dialling a hub that no longer listened.

## 0.5.0 - 2026-10-19 21:16 +06:00

### New

- **Run a hub and join other computers to it.** `isc hub invite NAME` makes a code, and a paired phone or browser reaches every machine joined to the hub from one page, with what each device may do.
- **Pair devices by a code.** Open the code's link, paste it or scan it. Approve a new phone from one that's paired, and pair a device again with `isc hub device pair-again`.
- **Allow and Deny.** insensical asks before a program in a pane changes who can reach this computer.
- **Agents start and talk to other agents.** `isc agent start codex "Review the diff"` opens a helper, which is listed under the agent that started it. Limits keep it in check (`agents-at-once`, `agents-deep`).
- **Coding agents can use insensical.** Ask in your own words, and the agent opens tabs and panes and reads and types into them.
- **`isc upload PANE FILE` gives an agent your files**, and the page's paperclip adds photos and files from a phone.
- **Keep agents out of a pane** from the palette or `isc`.
- **A Home tab on the page** shows every project as a card.

### Improvements

- `isc web` is now `isc hub`, and `isc web token` is now `isc hub device`. Cookies, `isc web login`, `--trust-proxy` and `/enrol` are gone, and the hub's files moved to `~/.local/share/insensical/hub`.
- A computer joined to your hub notifies your phone too.
- `isc hub setup` walks you through reaching this computer from a browser or a phone, and `isc hub tailscale on` does what the Tailscale switch does.
- `isc capture` prints only the end of a pane (`--trim` is gone), and `isc wait` and `isc ask` wait for the right thing.
- `isc run` and `isc split` take `--name`, `--background` and `--wait`, and `isc ls` can filter by program, state, opened and here, with a shorter line for each pane.
- Cursor CLI, Copilot CLI and Hermes report what they're doing, and about six hundred programs have their own icon.
- The window is grey by default, with a new app icon, calmer settings, and menus and fields that look and work alike. Settings has no Changed page.
- The window speaks to screen readers.
- Windows with many agents stay light, and an idle hub and joined machines cost nothing.
- Devices have an icon and a colour.
- `isc tui`'s settings have three pages: Settings, Remote Hosts and Hub.

### Bug fixes

- Removing a device takes hold at once.
- `isc hub status` no longer says now and then that the hub isn't running while it is.
- A paired phone or browser isn't asked to pair again when the hub just can't be reached.
- A pane you open while the page is reconnecting has its keys at once.

## 0.4.0 - 2026-10-13 05:17 +06:00

### New

- **insensical in a browser.** `isc web` serves a page you can open on a phone or at a desk. It shows what needs you, opens panes at your phone's width, and a home-screen copy pairs by itself.
- **Share a project or a terminal with a guest**, on your network or not, from the Share dialog. They see what you shared and nothing else.
- **Other machines in the window.** Reach another machine's daemon through your own `ssh` or its tailnet address, and add a machine, even put insensical on it, from the window.
- **Know what needs you.** Each row says in words what its pane is doing and how long it has needed you. Long shell commands count as activity, and many panes stopping at once make one notification.
- **Notify when nobody is attached.** Set `notify-command` to a program the daemon runs.
- **Open a project from a file.** Put a `.insensical.toml` in a directory and open it in one go.
- **Commands and modes of your own**, with a resize mode, under `[commands]` and in the keys.
- **Select and copy with the keyboard.** `Ctrl+Shift+X` or the leader and `v`.
- **The overview shows every pane's screen**, small and live.

### Improvements

- The sidebar can show only activity, and the count of panes that need you has moved from the bell to the activity button.
- Pictures show in more places and stay, and the terminal engine is a newer Ghostty.
- `isc api` lists what you can ask, `isc clients` and `isc disconnect` manage connections, and `--rights` asks for fewer rights.
- A server with very many panes starts much faster on Linux.
- The settings sheet is a panel of cards with a page for each thing you set.
- `isc ls` rows say what a waiting pane waits for, and a new project can be made in a directory that isn't there yet.
- A new icon, `isc` by itself for arm64 Linux, and an install line that takes a version.
- Your `[keys.direct]` and `[keys.leader]` tables now lie over the defaults. After the leader, `v` selects and `Shift+S` splits right, and the sidebar's field only filters.
- On a Mac, keys are shown in signs alone.

### Bug fixes

- A picture too large for its pane is shown whole.
- Ctrl-C works in a pane whatever started the server, and the daemon no longer dies with the session that started it.
- One mistyped setting no longer undoes the others, and two programs changing the settings file at once no longer lose a change.
- `isc attach` ends when the program in the pane does.
- An agent that stops on its usage limit says "Usage limit reached".
- A pane's number is never given to another pane, even after a crash.
- Tab in a shell no longer hangs when the daemon doesn't answer.

## 0.3.0 - 2026-10-10 23:20 +06:00

### New

- **The whole interface in a terminal.** `isc` with nothing after it runs insensical inside your terminal, over `ssh` too.
- **Install `isc` by itself** with one line on any Linux, for a machine with no window.
- **Pictures in the terminal.** Programs can show pictures by the Kitty graphics protocol.
- **Drag a pane anywhere**, and drag projects in the sidebar to reorder them.
- **An icon for each project**, and icons for your own programs in an `[icons]` table.
- **`Alt+1` to `Alt+9`** go to a project's tabs.

### Improvements

- A path a program printed is underlined only while you hold `Ctrl`, and `Ctrl`+click opens fewer files.
- Two windows on one pane work: it's sized to the window you used last.
- Tab goes into a directory in the file finder.
- `isc capture --vt` prints text, colours and styles only.
- What the server refuses is said, and a window uses less memory for a pane with a long history.
- Wrong settings values are explained in plain words.

### Bug fixes

- A pane no longer goes blank after you make it narrower, and one the window couldn't open says "Not shown yet".
- Search finds a word that the terminal's width broke across two rows.
- Clicking a notification with no window open now shows the pane, and the tray no longer opens a second window.
- A program can no longer fill your clipboard from a window on another workspace, or put a fake link in the note that says where a click leads.
- `Ctrl`+click on a path can no longer start something you didn't mean, and opening the file finder no longer runs a program a repository names.
- The saved layout and the daemon's socket can no longer be read by other users of the machine.
- A client that stopped reading, or a pane sized far beyond any screen, can no longer fill the server's memory or end the daemon.
- `isc integrate` no longer leaves an agent's settings half-written or mishandles paths with spaces.
- `isc wait --timeout inf` no longer crashes.
- A new window no longer opens with everything unfolded in the sidebar.
- Long directories, the palette, the sheet of keys and the settings no longer cut off their last rows in a small window.
- Notices, themes and fonts are listed with readable names in the settings.

## 0.2.0 - 2026-10-06 03:16 +06:00

### New

- **Knowing what needs you.** A waiting pane glows, a password prompt is noticed, and you're told where you are while insensical is in front. A short sound plays for a pane that waits or failed.
- **A tray icon on Linux**, and notifications that carry insensical's icon. On a Mac the menu bar and the Dock say what needs you, and notifications arrive with the window closed.
- **Mute a whole project**, or turn notifications off with `notifications = false`.
- **Put a project to sleep** to give back its memory. A project's heading shows what it costs.
- **A settings sheet**, opened with `Ctrl+,` (`⌘,` on a Mac), and `isc config get`, `set` and `reset`.
- **Ctrl+click opens files in your editor at the line**, and follows links a program marks.
- **A tab shows a deck of its programs**, and programs are shown as tiles.
- **Five new themes** for a dark desktop and a light one, a see-through window with `background-opacity`, and ligatures.
- **`isc wait --output TEXT`** and **`isc api`** for scripts.
- **A Mac build**: an icon, a disk image, Option as Alt with `option-as-alt = true`, and quitting asks first while something runs.

### Improvements

- Far less memory with many panes, and output from a program that floods a pane is more than twice as fast on a Mac.
- `scrollback` sets how much history a pane keeps, and `long-command` says what counts as a long command.
- A better command palette, with headings, icons and keys, and a sheet of keys that says why a command has no key.
- Any font works, since the icons now come with insensical.
- A calmer window, with taller rows and a little motion.
- `Ctrl+Shift+Up` and `Down` scroll from command to command.
- The window opens at the size it last had, and the file finder leaves out what git ignores.
- The settings file is there from the first start, and input methods work if you turn them on.

### Bug fixes

- The overview no longer puts every project on a row of its own.
- The search field no longer lies over the match it found.
- Dragging a pane's border no longer types text such as `28;5R28;10R` at the prompt.
- A path or an address that ended its row is no longer opened on a click by mistake.
- A project's name in the sidebar is no longer cut to a letter or two.

## 0.1.0 - 2026-10-04 04:35 +06:00

### New

- **Programs that stay.** They outlive the window, come back after a restart of the machine, and survive an update.
- **Projects in a sidebar**, as a tree of tabs and panes, with splits, floating panes, a dock for minimized panes and an overview of every tab.
- **A command palette**, a file finder, and search in a pane's text.
- **Themes and keys your way**, each command on a chord of its own or after a leader key as in tmux.
- **A state on every pane**: working, waiting for you, done or failed, with a count of panes that need you and a key to go to the most urgent.
- **Hooks for Claude Code, Codex, Gemini CLI, opencode and pi**, and desktop notifications when a pane waits, finishes or fails.
- **The `isc` command line** to drive everything the window does.
- **Installing** on Arch Linux (`insensical-bin` in the AUR), Debian 12 and later and Ubuntu 24.04 and later (the `.deb`), other Linux on x86-64 (the tarball), and macOS 13 and later on Apple silicon (the disk image, or Homebrew).
