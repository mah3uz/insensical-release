# Changelog

What changed in each release of insensical, newest first. Each release's section is also
its release notes.

## Unreleased

## 0.2.0 - 2026-10-06 03:16 +06:00

### Upgrading from 0.1.0

This version's window and server speak a newer protocol than 0.1.0's. After installing
it, restart the server from the window, or with `isc restart-server`. What runs in your
panes carries on.

Homebrew now installs insensical from `mah3uz/tap`. If you ran
`brew tap mah3uz/insensical …` for 0.1.0, that tap stays at 0.1.0: run
`brew untap mah3uz/insensical`, then `brew install --cask mah3uz/tap/insensical`.

### Knowing what needs you

- **A waiting pane glows.** A pane that waits for you takes the waiting colour at its
  edge and in its header, and casts a soft light of it into the gaps around it. The
  light beats three times as the pane begins to wait, then stands still.
  `attention-glow = false` turns it off.
- **An icon in the tray on Linux.** With no window open, insensical's icon in your
  desktop's tray is marked while a pane needs you. Its menu lists those panes; pick one
  and a window opens on it. Close the window from there too, or quit: quitting ends
  every program, so a window asks first and says what would end. **Show in the
  tray**, in the settings, turns it off. It needs a tray that shows
  status notifier items, which GNOME has only with the AppIndicator extension.
- **Notifications on Linux carry insensical's icon**, where they showed a plain
  terminal's.
- **A password prompt is noticed.** A command that stops to ask for a password, as
  `sudo` does in the middle of an install, marks its pane as waiting and tells you, with
  the prompt as what it says.

- **You're told where you are.** While insensical is in front, a pane that waits, fails
  or finishes is said in a notice at the foot of the sidebar, and not on your desktop.
  Click the notice to go to the pane. A finish goes after six seconds; a pane that
  waits or failed keeps its notice until you've seen it. When insensical isn't in
  front, you get the desktop notification as before.
- **Sound.** A pane that waits or failed makes a short sound; `sound = "all"` adds a
  softer one for a finish, and `sound = "off"` turns it off.
- **Notifications can be turned off**, all of them with `notifications = false`, or one
  kind with `notify-waiting`, `notify-failed` and `notify-done`.
- **Notifications on a Mac arrive with the window closed**, for as long as the
  application is running. Click one and a window opens on the pane it's about. Before,
  only an open window posted them.
- **A sidebar that says more in fewer rows.** A pane that needs you shows why on its
  row. A tab that holds one pane no longer takes a row of its own. Projects are
  headings. A pane has one name everywhere it's listed: yours, else its program's
  title, else its program and directory, such as `fish · api`.
- The count of panes that need you is two counts: those that wait or failed, and those
  that only finished.

### Many projects, many panes

- **A shell is called by its directory.** The sidebar, the dock and the palette no
  longer put `fish ·` or `zsh ·` before every shell's directory: the icon says it's a
  shell.

- **Faster where there was room to be.** A program that floods a pane is read by a
  thread of its own for as long as the flood lasts, so it is held back only by the
  kernel: long lines and raw output go through at some 450 MB/s here, up from 260. A
  build's lines were already at what the kernel allows.
- **Mute a whole project.** "Silence this project's notifications" in the palette, or
  `isc project mute NAME`. Panes opened in it later are quiet too, and their states
  still show.
- **You say what a long command is.** `long-command` sets how many seconds a command
  runs before its end is told of. It was fixed at 30.
- **See what a project costs.** A project's heading in the sidebar shows the memory
  its programs hold and the processor they're using; the overview shows it for each
  pane, and `isc usage` prints it.
- **Put a project to sleep.** It ends the project's programs and gives back the memory
  they held, and keeps its tabs, panes, directories and what each pane showed. Wake it
  and each pane has a shell where it was, with what ran there ready at the prompt.
- **Far less memory with many panes.** A pane's history is packed away when the pane
  goes quiet and unpacked when you read it. A pane with a full history takes about
  1.6 MB in place of 11.5 MB; seventy panes took 74 MB in place of 365 MB.
- **Faster with several panes.** A pane that hasn't changed is no longer drawn again
  when another prints, or while an agent's mark turns. With six panes and one printing,
  a frame takes a third of the time it did.
- Dragging a window's edge no longer resizes every program at every frame: a pane's
  size changes at most once in 25 milliseconds.
- The server raises its limit on open files when it starts, so a Mac is no longer
  held to some eighty panes.
- **`scrollback`** sets how much history a pane keeps, from 1 to 200 megabytes.

### Settings and commands
- **The settings file is there from the first start.** The window makes
  `config.toml` if you have none, with every setting explained and none of them set.
- `isc config get`, `set` and `reset` complete the names of the settings.
- **`isc wait --output TEXT`** waits until a pane's screen shows the text, for scripts
  that start something and then use it.
- **`isc api`** sends the daemon any request as JSON and prints the answer, so whatever
  the window can do can be scripted.
- **Input methods, if you turn them on.** `input-method = true` lets your system's
  input method compose text in a pane, for Bangla, Chinese, Japanese and the rest. It's
  off by default, as it hasn't been tried with a real one yet.
- **Files open in your editor, at the line.** `Ctrl`+click `src/main.rs:12:5` and it
  opens there in the editor `open-files-with` names; a terminal editor opens in a pane
  beside the one you clicked in.
- **Links a program marks are followed** (OSC 8): `Ctrl`+click the words.
- **The terminal's description comes with insensical.** Programs get `xterm-ghostty`
  and every feature a pane has, whether or not Ghostty is installed. `term =
  "xterm-256color"` goes back to the description every machine knows.
- **A settings sheet.** `Ctrl+,` (`⌘,` on a Mac) opens every setting on one sheet, to be
  switched, picked or stepped, with nothing to type. A change takes effect at once and
  is written to your `config.toml`. It marks what you've changed and resets it, finds
  a setting as you type, and shows each agent and whether its hooks still work.
- **`isc config get`, `set` and `reset`** read and change one setting, and leave the
  rest of your settings file as you wrote it.
- **A better command palette.** It lists things under headings, with icons, keys as key
  caps and each setting's value. What you type is ranked, so the best fit is first,
  and what you chose recently is offered first. Settings are in it too.
- "Name this pane" and "Name this tab" are commands, in the palette and for a key.
- **The sheet of keys says why a command has no key** when your own `[keys.direct]` or
  `[keys.leader]` table left it out. Its long names wrap, and what follows the leader
  is listed under the same headings.

### The window

- **A tab shows a deck of its programs.** In the sidebar and along the top, a tab's
  icon is the tile of its front pane with the tiles of its other panes stacked behind
  it, in place of an outline and a number.
- **Settings, keys and the overview are a click away.** They're at the foot of the
  sidebar, with a button for a new project, and at the end of the top bar when the
  sidebar is hidden.
- **The settings sheet is laid out as your desktop's is.** Sections stand in a column
  of their own, each with a tile in its own colour, and a section's settings are one
  roomy group.
- **Icons are drawings.** Every icon of the interface is drawn, so it sits in the
  middle of its button or tile whatever font you use. Panels are headed by glossy
  tiles like the programs', files in the path picker are coloured by kind, and panels
  that lie over the window are lit from above.
- **Taller rows, lifted capsules.** Sidebar rows, tabs and pane headers have a little
  more height, and the row and tab in front are lit from above with a hairline and a
  soft shadow.

- **The overview can show only what needs you.** Press Tab in it.
- **A neutral interface.** `interface-style = "neutral"` draws the window around the
  terminals in greys, whatever the theme.
- **A see-through window.** `background-opacity` below 1 lets your desktop show behind
  the text, and `background-blur` asks the desktop to blur it. Both are in the settings
  sheet.
- **Programs are shown as tiles.** What runs in a pane is its own mark on a small
  glossy card in its maker's colour: some two hundred programs, from Claude, Codex
  and Gemini to Vite, Rails, Laravel, Django, Docker and psql. A dev server is called
  by its own name, `vite` or `rails`, and not `node` or `ruby`. The tile is the same in every theme, in the sidebar, the pane
  header, the tabs, the dock, the overview and the palette.
- **Any font works.** The icons come with insensical now, so `font-family` doesn't have
  to be a Nerd Font.
- **A little motion.** Sheets, questions and notices fade in, and a pane you float, put
  back or zoom clears from a light wash so you see where it went. `motion = "off"`
  stops it, along with the turning mark and blinking cursors; by default it follows
  your desktop.
- **From command to command.** `Ctrl+Shift+Up` and `Ctrl+Shift+Down` scroll to the
  prompt before and after, when your shell marks its prompts.
- **Ligatures.** Symbols side by side are drawn as your font's ligature for them, such
  as `->` and `!=`. `font-ligatures = false` turns it off.
- **Five new themes, each for a dark desktop and a light one**: `tidepool`, `heather`,
  `lichen`, `lantern` and `quartz`. Set one by name and it follows the desktop as it
  turns. A theme file of your own can hold both sides too, under `[dark.terminal]` and
  `[light.terminal]`.
- **A calmer window.** Work in progress is shown in grey with no coloured line, so a
  pane that waits or failed stands out. Keys look the same wherever they're shown.
  Faint text is easier to read. Sizes and corners follow one scale; rows and pane
  headers are two pixels taller.
- A floating pane stands clear of what's under it. A window with nothing open lists
  the ways on, each with its key. The overview sits in the middle of the window.
- The window opens at the size it last had, and remembers that you hid the tabs.
- The file finder leaves out what git ignores.
- **The note of what a click opens keeps out of the way.** It's smaller, sits in the
  pane's bottom right corner where it was across the bottom left, over what you were
  typing, and goes after four seconds if the pointer rests.

### On a Mac

- **The menu bar and the Dock say what needs you.** insensical's mark sits in
  the menu bar with the number of panes that wait, failed or finished unseen; its menu
  lists them, each with its state's mark, and goes to the one you pick, opening a window
  if there is none. The Dock icon carries the same number and, on a right click, the
  same panes. `menu-bar` and `dock-badge` turn each off, in a **Menu bar and Dock**
  section of the settings.
- **The settings say what macOS allows.** The Attention section opens with whether
  macOS lets insensical notify you and badge its Dock icon, marked when it doesn't,
  with a button to the place in System Settings that changes it.
- **It says once that it's still running.** The first time you close insensical's last
  window, a notification tells you it's still there and how to quit it. The
  application's menu has About insensical.
- **`⌘Q` leaves insensical in the menu bar.** It closes the windows and the application
  stays, counting what needs you and telling you. Quit insensical, in its menu or the
  menu bar item's, or `⌥⌘Q`, quits it. With `menu-bar = false`, `⌘Q` quits as before.
- **Quitting asks first while something runs.** Quit with a pane running or
  waiting and insensical says what you'd stop being told about, and offers to close the
  window in its place. The application's menu gains Settings…, Services, Hide, Hide
  Others and Show All.
- **Programs are called what you called them.** A Mac names a process by its file, not
  by the name you started it with, so Claude Code's pane read `2.1.289` and Python's
  `python3.12`, and both were shown as a plain terminal. A pane now says `claude` and
  `python3`, with their tiles. The server names programs, so restart it once after
  installing.
- **Option as Alt.** `option-as-alt = true` makes Option with a key Alt to the
  program in the pane, so a shell's Alt+B and Alt+F answer. It's off by default: Option
  then types your keyboard layout's character, as before.
- `isc key` with Alt, as in `isc key 3 alt-b`, sends Alt and the key. It sent
  nothing.
- **Keys are shown with `⌘` and `⌥`**, in the sheet of keys, the hints and the
  palette, where they were spelt `Cmd` and `Alt`, and `⌘` comes first.
- **Terminal text is 13 points by default**, where it was 12, which is small
  there. If you've set `font-size`, nothing changes.
- **Completions.** Installing, with Homebrew or with
  `just install-mac`, now sets up completions for zsh, bash and fish. Before, nothing
  did, and `isc` completed only file names.
- **Fast output is more than twice as fast.** A build's log or a long listing reaches
  the window at about 160 MB/s where it reached it at 67. A Mac hands a program's output
  over a kilobyte at a time, and insensical passed it on that way.
- **An icon and a disk image made for a Mac.** The application's icon is drawn to the
  size and shape a Mac's icons have, so it sits level with its neighbours in the Dock.
  The disk image opens on a picture that shows where to drag the application, and
  carries the application's icon as its own.

### Fixed

- The overview set every project on a row of its own, so several projects of one tab
  each were a column down the middle of the window. Projects now stand side by side.
- The sheet of keys left empty room at its right in a wide window. Its columns now
  fill it from edge to edge.
- A path or an address that ended its row was marked as one, and opened on a click,
  from anywhere in the empty room to its right. It's now marked only under the pointer.

- The search field could lie over the match it had found. It moves to the foot of the
  pane when it would.
- A pane without the keyboard kept room in its header for buttons that weren't
  showing, which cut what the pane had to say down to "..". Its buttons now lie over
  the header's end under the pointer, and a pane with something to say gives it the
  room its directory had.
- With the tabs along the top, a tab that had a state mark was drawn too narrow for its
  title, and the project's name was cut at 200 pixels with room to spare.
- A project's name in the sidebar was cut to a letter or two to make room for what its
  programs hold and for buttons that weren't showing. The name now comes first.
- Dragging a pane's border could type text such as `28;5R28;10R` at the prompt of a
  shell beside it. It happened in a pane where an editor or an agent had been killed
  after asking to be told of every resize: the shell left behind was sent reports it
  never asked for, and fish typed them, and the answers to its own questions, at its
  prompt. A resize is now reported only to the program that asked.
- In the direct keymap, the word `leader` in `[keys]` meant `ctrl-space` whatever
  `leader` was set to. It's now the key you set.

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
