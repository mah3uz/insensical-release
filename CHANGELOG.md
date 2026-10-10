# Changelog

What changed in each release of insensical, newest first. Each release's section is also
its release notes.

## Unreleased

## 0.3.0 - 2026-10-10 23:20 +06:00

### Upgrading from 0.2.0

This version's window and server speak a newer protocol than 0.2.0's. After installing
it, restart the server from the window, or with `isc restart-server`. What runs in your
panes carries on.

**On a Mac, restart the server from the window, or quit insensical before you restart
it from a terminal.** 0.2.0 stays in the menu bar when its window is closed. If you
then run `isc restart-server` in another terminal and open a window again, that
window is still 0.2.0's and asks to restart the server. Don't: it would stop the
server and everything in your panes, and hang for some seconds each time, without
ever opening. Quit insensical from the menu bar's item, or with `⌥⌘Q`, and open it
again. From this version on, such a window opens the new application and leaves the
server alone.

If you use opencode or pi, run `isc integrate` again. Their scripts are new: the ones
you have lose a message that begins with a hyphen, as a list does, and offer a session
named like an option as part of the command that resumes it.

Some things you're used to behave differently:

- **A path a program printed is underlined only while you hold `Ctrl`** (`⌘` on a Mac).
  That's when it's looked for on the disk. A web address is still underlined under
  the pointer.
- **`Ctrl`+click on a path opens fewer files.** With `open-files-with` left at
  `system`, a file opens only if opening just shows it: text, source, a picture or a
  PDF, which can't be run, begins as its name says, and can't be replaced by someone
  else. Everything else is shown in your file manager: a directory, a `.csv` or
  `.tsv`, a Markdown file that begins with HTML, a file under a shared directory such
  as `/tmp`. The note beside the pointer says which the click will do. With an editor
  chosen in `open-files-with`, any file still opens in it.
- **A pane made narrower rewraps its lines even when a program has switched line
  wrapping off.** A full-screen program's screen is still cut at the new edge, and the
  program draws it again.
- **In a pane started with a command**, such as `isc run -- claude` or
  `isc run -- bash`, what an agent reported stands until its next report or until the
  program ends. A command that ends inside it no longer clears it.
- **A command offered at a prompt after a restart is quoted another way.**
  `--flag=value` comes back as `'--flag=value'`, which bash, zsh and fish read alike.
- **`isc capture --vt` prints text, colours and styles only.** It printed the pane's
  palette, title, links and modes too, which changed the terminal you read it in.
- **`isc`, with nothing after it, opens the interface in a terminal** when a person
  runs it at one. In a script, in a pipe and inside one of insensical's own panes it
  lists the commands.
- **What the server refuses is said.** Ask for a new tab in a project that's asleep,
  say, and a notice headed "It could not be done" gives the server's own words. The
  window said nothing.

### Added

- **The whole interface in a terminal.** `isc`, with nothing after it, runs insensical
  in the terminal you run it in, as a terminal program: your projects down the side,
  tabs along the top, each pane in a box, a line at the foot that says what your keys
  are doing, and the palette, the overview, the settings sheet, the sheet of keys and
  every question. It's for a machine with no desktop and for one you reach with
  `ssh host -t isc`. It's one more client of the same server, so what you do in it
  shows in a window at once. Commands follow `Ctrl+Space`, whose list of keys stays for
  as long as you read it, and `Ctrl+Space` then `D` leaves with everything still
  running. It uses your theme and your keys, and a `[tui]` table in the settings says
  what a terminal should do differently: `theme = "terminal"` there gives it your
  terminal's own colours. Copying goes to your own terminal's clipboard, where the
  terminal takes one from a program (Terminal on a Mac doesn't), and on a
  machine with no desktop notifications reach your terminal too. It needs a Nerd Font
  for its icons. Pictures aren't shown in it, and paths aren't opened from it.
- **`isc` by itself, for a machine with no desktop.** Each release has
  `isc-x86_64-linux`: one file that needs nothing installed and runs on any Linux
  distribution. It's the daemon, the command line and the interface in a terminal.
- **Renaming and closing a project, and sizing the panes for this window, are
  commands.** "Rename this project", "Close this project and everything in it" and
  "Size the panes for this window, over any other that shows them" are in the palette
  and on the sheet of keys, in the window and in a terminal. They're `rename-project`,
  `close-project` and `lead` in `[keys]`, and have no default key. Before, the window
  had them only as buttons on a project's row and on a pane's "Sized elsewhere" mark.
- **Pictures in the terminal.** A program can show a picture in a pane by the Kitty
  graphics protocol, as `kitty +kitten icat`, `chafa`, `timg` and `viu` do. It scrolls
  with the text, is drawn at the size the program asked, and is still there when you
  come back to the tab. Sixel isn't supported, and pictures don't survive the daemon
  stopping or moving to a new version.
- **Drag a pane anywhere.** A pane dragged by its header could only swap places with
  another. Now, let go near a side of another pane and it takes that half, so two panes
  side by side can become one above the other. Let go at the tab's edge and it becomes
  a column or row beside everything; let go on a tab and it moves to that tab. The panes
  glide into the arrangement you'd get while you hold it, the carried pane says what
  letting go would do, and Escape puts it back. `isc put` does the same from the command
  line.
- **An icon for each project.** A project has a tile beside its name in the sidebar, the
  top bar and the overview: the first letter of its name on a colour of its own, until
  you choose. Pressing the tile on a project's sidebar row picks a colour and a mark, or
  takes a square SVG of yours, unless it shows a picture from another file.
  `isc project icon` does the same.
- **Two windows on one pane.** A pane is the size that fits the window you used last,
  or `isc attach` if that was last. The other window shows it at that size with a
  "Sized elsewhere" mark in its header, and you can move over what doesn't fit with
  Shift or Alt and the wheel. Before, each window resized the pane for the other.
- **Dragging in the sidebar.** A project's row dragged onto another puts the projects in
  order, and a tab's row dragged onto another tab of its project puts the tabs in order,
  as dragging a tab along the top already did. `isc project move NAME INDEX` does the
  first from the command line.
- **A tab by its number.** `Alt+1` to `Alt+9` go to the project's first to ninth tab
  (`⌘1` to `⌘9` on a Mac, `1` to `9` after the leader). They're the commands `tab-1` to
  `tab-9`. A program in a pane no longer gets those chords: set one to `none` in `[keys]`
  to give it back.
- **Icons for your own programs.** An `[icons]` table in the settings file gives a
  program a tile: `gitu = "git"`, or `deploy = { mark = "docker", colour = "#ff8800" }`.
  It applies as you save.
- **Tab goes into a directory in the file finder.** With a directory marked, Tab lists
  only what's in it. Shift+Tab, or Backspace with nothing typed, comes back out.

### Changed

- On the settings sheet, the seconds a command runs before it counts as long step by
  five, since by ones there are thousands of presses between its ends.
- A wrong value for a theme, a key or an icon in the settings file is said in plain
  words.
- A tab's deck of program tiles is spread wider, and each card has a pale edge and a
  shadow, so the cards behind the front one can be told apart on a dark sidebar.
- A window uses less memory for a pane with a long history: its own copy of the history
  is packed away once the pane has been quiet for two seconds, as the daemon's already
  was. One pane with its 10 MB of history took 17 MB of the window's memory and now
  takes 9 MB.

### Fixed

#### The terminal's screen and scrollback

- A pane could go blank, or drop out of the window, after you made it narrower, and
  stay that way. It happened when a full-screen program in it, or one that had run in
  it earlier, had drawn a wide character (an emoji, or Chinese, Japanese or Korean
  text) where the pane's new edge fell. The pane now shows as it should.
- A very large pane (300 columns by 160 rows, say) with a full-screen program that
  scrolls in it, a pager for one, could go blank while the window asked for it over
  and over. The window is now sent the pane drawn afresh. Until the program clears its
  screen or ends, a window that opens on the pane gets it without its links and without
  the shell's screen and history behind the program.
- A pane with a history longer than 8 MB whose program kept printing could never be
  shown over a slow connection: the window attached, was cut loose, and attached
  again, without end.
- A pane the window couldn't open stayed blank for good. It now says "Not shown yet"
  and is tried again by itself.
- After a restart, a pane's new shell could inherit what the old program had switched
  on: mouse reporting typed at the prompt, another way of reading keys, line-drawing
  characters, margins. What's kept of a screen is now only what it showed.
- Search missed a word that the terminal's width had broken across two rows. It's now
  found, and selected across the break.
- The two half-round Powerline characters are drawn as shapes like the triangles are,
  so a rounded badge a program prints fills its cell at any size.

#### Links and files

- `Ctrl`+click on a path a program printed could start something: a directory that is a
  Mac application, a `.jar`, a web page, a `.DESKTOP` file, a web page named `notes.md`
  on a desktop that chooses what opens a file by what's in it. A click now opens a
  file only where opening just shows it, and shows the rest in your file manager. On
  Linux your desktop is asked what kind of file it is as you click, and a file changed
  or swapped between the note and the click is shown, not opened.
- `Ctrl`+click follows only what the note said. If a program changed where its words
  lead as you clicked, or while the note was away, nothing opens and the note says
  where they lead now. A program isn't told that the pointer is on its link while you
  hold `Ctrl`.
- The note that says where a web address leads could be made to show another site: it
  was cut at its end, and kept invisible and direction-changing characters. It now names
  the machine first, and an address with such characters isn't a link. Nor is one with
  nothing where the machine's name belongs, such as `https:///name@host/`, which a
  browser reads the next name out of. A name with letters outside ASCII is shown as
  the network is asked for it (`xn--…`).
- A `file://` link that names another machine opened the file of the same path on yours.
  It's now ignored.
- An address or a path that the terminal's width broke onto the next row was followed
  only to the end of its row, which opened half of it. It's now read whole from either
  row, and both parts are underlined.
- Moving the pointer over a path a program printed looked it up on the disk each time
  the pane was drawn, which could freeze the window on a dead network mount. A path is
  now looked up once, and not on a file system known to be a network or automounted
  one, or kept by a program through FUSE.
- Opening the file finder in a repository could run a program the repository names:
  one in its own Git settings, or the one that fetches a part of it kept elsewhere.
  Neither runs now. Git is given three seconds to say what's ignored; after that the
  files are listed without it.
- A file chosen in the finder is typed so that fish reads it as bash and zsh do, and a
  name that begins with a dash is typed as `./-name`. A name that holds a line end,
  another control character, or an invisible or direction-changing one isn't listed.

#### Pasting and the clipboard

- The question about a held paste didn't show what would be pasted, counted lines ending
  in a carriage return as one, and gave the wrong reason for text holding the
  end-of-paste mark. It now shows the text line by line, with hidden characters made
  visible, and says why it waits.
- A path typed by the file finder, and text an input method gives in one piece with a
  line end in it, skipped the checks a paste gets. They're now checked the same way.
- A program could fill your clipboard from a window that was on another workspace or
  had closed, since the window said it had left the front only when it was next drawn.
  It now says so as it leaves, and a closed window is gone from the server at once:
  its pane could stay that window's size, and count as looked at, until the server
  next had something to send.
- The window takes text for the clipboard only from the pane it said you're looking
  at, and no more than a megabyte, whatever the server sends. Control characters are
  left out of it, so a terminal you paste into has nothing in it to obey, and at most
  three texts are put there at once and one a second after that.

#### Notifications and status

- A command could print the marks that say a command ended and so clear an agent's
  `waiting`, or a wait for a password, and take down its notification. Marks count now
  only when the pane's own shell had the terminal, and progress reports and
  notifications from a command still running end neither.
- A "paused" progress report, which a `cat` of a file can print, left its pane
  `waiting` for good. It now ends with the command, or when you look at the pane.
- A file shown with `cat` that held a shell-integration mark no longer stops a pane
  from telling you when long commands end, in a shell that marks none of its own.
- A report dated in the future silenced every later report from its pane, so an
  agent's request for permission never showed. A date is now held to the daemon's
  clock, and putting the clock back no longer drops reports either.
- `isc restart-server` forgot what each pane's state rested on: afterwards a program's
  output could clear an agent's `waiting`, and a bell's mark stayed however often you
  looked.
- A pane opened after a restart could be given the number of a pane closed before it,
  and a hook or program left running from the closed pane then reported on the new one.
  The highest number given is now kept with the saved layout, and numbering goes on
  from it.
- A notification on the desktop could carry a link or a picture from a program's
  text, in its title or under it, where the desktop reads markup. The text is now
  shown as text, also when the notification service hadn't answered as insensical
  started, or was started or replaced since.
- A server could have the application start a sound for every notification, and put
  any number of them on your desktop. The application now puts at most five there at
  once and two a second after that, and starts no sound within 300 ms of the last.
  What's held back is told when there's room, the last from each pane.
- A clicked notification, or a pane picked in the tray, left a window on another
  workspace where it was: the window acted on it only when it was next drawn, and a
  window out of sight isn't drawn. It now asks to come forward at once.
- Clicking a notification with no window open selected the pane and showed nothing. A
  window now opens on the pane, as from the tray.
- Picking from the tray just after a window opened could open a second window.
- **Quit insensical…** chosen from the tray with no window open, where no window then
  came, put its question to the next window you opened, however much later. The
  question is dropped after ten seconds.
- Some invisible and direction-changing characters, and others drawn as nothing or as
  a blank, got through in titles, notes, names and notifications, where they made two
  names look the same. The daemon, `isc` and links now go by one list of them. A
  program's own name, and a project named after its directory, are made plain the
  same way: they were taken as they came, control characters included.

#### The server and saved state

- A window from before an update, meeting a server that had already moved to the new
  version, offered to restart the server. Doing so stopped the server and everything
  in its panes, started the same server again, and held the window for five seconds,
  as often as it was pressed. It happened on a Mac, where the application stays in the
  menu bar with no window while the server is moved from a terminal. Such a window now
  says that it's the one from before, leaves the server alone, and opens the installed
  application in its place.
- A server you reach on another machine could stop your window, write over its memory
  or fill it: with a screen that claimed more room than it had, a layout nested
  without end, a terminal of a size none has, a floating pane wider than its tab, or
  output without end. What a server sends is now checked before it's used.
- The window and `isc` connect only to a socket in a directory that's yours and closed
  to others, answered by a program of yours, and say what's wrong when it isn't. A
  variable such as `XDG_CONFIG_HOME` or `XDG_RUNTIME_DIR` set to nothing no longer has
  the settings or the socket looked for in the directory you happen to be in.
- The saved layout and its directory could be read by other users of the machine. They,
  the window's own state files and everything else the daemon keeps are now yours
  alone to read, and a directory left open by an earlier version is closed when the
  daemon starts. Changing a setting keeps `config.toml` as private as you made it.
- A saved layout that can't be read is set aside as `state.json.unreadable`, where it
  was written over at the next save. One that holds a number too large ever to have
  been given counts as unreadable: it could make two panes share a number.
- A tab can no longer be split so deep that the layout couldn't be read back after a
  restart, and a pane started with a command or a directory of megabytes, which left
  the layout too large to send for good, is refused: 256 KB of them together is the
  most.
- A command offered at a prompt after a restart was quoted for bash and zsh, and fish
  could read a word holding `\'` as further commands, run by your Enter. Each word is
  now typed in the one form all three read alike. A saved offer that holds a line end
  isn't typed at all.
- A program that changed its title, rang its bell or reported progress without pause
  could slow every window and fill the daemon's memory, most of all with a window or
  `isc events` that had stopped reading. The daemon now takes such changes at a pace
  of its own and always ends with the last, and keeps one layout waiting for a client
  rather than every one.
- A client that stopped reading could fill the server's memory: with what it was sent
  unasked, with the answers to requests it went on making, or with the histories of
  many panes it attached to. Its connection is now closed once 8 MB of the first or
  of the second waits, and the histories that wait for one client are 256 MiB and one
  pane's at most.
- A program that asked the terminal questions and never read the answers could hold
  back what you typed, Ctrl-C included, and fill the daemon's memory. What you type now
  goes first, and unread answers are limited.
- The daemon, and everything running in it, ended when it ran out of file descriptors
  as a client connected. The connection now waits.
- A pane sized far beyond any screen, by a client, could end the daemon.
- A pane left open after its program ended stayed as it was when its project was put to
  sleep, and had no shell when the project woke. It now sleeps and wakes like any pane.

#### `isc` and agents

- A directory whose name holds escape sequences could act on your terminal when `isc ls`,
  `isc usage`, `isc restart-server` or a shell's completions printed it: write to the
  clipboard, or rewrite the lines above a question. Such characters are now printed the
  way `\u{1b}` is here, and `isc ls --json`, `isc api` and `isc events` write them as
  JSON's escape for them, which reads back as the same text.
- Completing `--project` in bash put a project's name on the command line as shell
  words, so a project called `$(id)` was run. A name that isn't one plain word is now
  offered by its number.
- A number given for a project could pick a project whose name was that number instead
  of the project with that number. A number is now always the project's own.
- With `COMPLETE` set in the environment for some other reason, every `isc` command,
  hooks included, answered as a shell completion and did nothing. Completions now ask
  through `ISC_COMPLETE`; the ones you have installed go on working.
- `isc ls` said nothing of a project being asleep. Its line now ends in `asleep`.
- `isc wait --timeout inf` crashed. It's now refused, with why.
- An agent's last message that began with a hyphen, as a list does, was read as an
  option, and the pane never said the agent had finished, failed or asked. This affected
  opencode and pi, and `isc status -m` and `isc notify` called by hand.
- A session named like an option, such as `--dangerously-skip-permissions`, became
  part of the resume command offered at a prompt. Such a name is no longer offered.
- A hook told more than a megabyte, such as everything a tool printed, reported nothing.
- `isc integrate` wrote the path of `isc` into an agent's hooks unquoted, so a path with a
  space left the agent silent and one with `$(...)` in it was run at every event. It
  also took any hook ending in `hook claude` for its own, and rewrote or removed it.
- `isc integrate` could leave an agent's settings half-written, saved its copy of them
  readable by others when the settings weren't, would write through a link left at
  the copy's name, and wrote over a change the agent made to its settings while you
  were being asked. The settings are now replaced in one step with their permissions,
  and read again after your yes: if they changed, it stops.
- `isc integrate` took `CLAUDE_CONFIG_DIR=conf`, or another agent's variable holding
  part of a path, as a place in the directory you were in. Only a whole path counts
  now. A settings file of more than 16 MB isn't read, an opencode or pi script it
  can't read is left alone where it was written over, and a settings file or a script
  that is a pipe no longer keeps it waiting for ever.
- `just install-local` wrote a broken desktop entry when `PREFIX` held a space.

#### The window and the settings

- A new window opened with every project and tab unfolded in the sidebar, whatever you'd
  folded away. What you fold is now kept, and the next window opens as you left it.
- A directory too long for its place lost its end, the part that says where you are. In
  a pane's header, in the question before a tab or project closes, and where the
  settings name their file, it now loses its start instead.
- With `HOME` set to a path ending in `/`, the window wrote no directory from `~`, and
  with `HOME` set to nothing it wrote every directory from it.
- On a Mac, a pane kept its shell's name while `sudo` waited in it, since `sudo` runs as
  root. It's now named for what runs in it.
- Three notices at once took half the sidebar in a small window. They now take a third
  of the window's height at most, with a line that counts the rest.
- In a short window, the palette ran off the bottom, with its foot and its count of what
  is below out of sight. It now lists as many rows as fit.
- The sign beside a question sat halfway down a long explanation. It now sits at the
  top, beside the question.
- The question for a new project said "Tab: into the marked one" with nothing listed
  to mark.
- The file finder said a deep directory from its start and cut its end. It now says the
  last names that fit: `In …/payments-service/migrations/2026`.
- In a small window, the sheet of keys squeezed its heading into a column one word
  wide beside the field. The field now goes under the heading there, and the line at
  the sheet's foot wraps instead of being cut.
- The list of what follows the leader cut long command names short. They now wrap, as
  they do in the sheet of keys.
- The sheet of keys and a long section of the settings cut their last row at the edge
  with nothing to say there was more. They now fade there until you reach the end.
- In the settings, a long explanation was cut short with "…" where the window was
  narrow. It now runs onto a second line.
- The settings showed the terminal types as "Xterm ghostty" and "Xterm 256color". They're
  now written as they go in the file: `xterm-ghostty`, `xterm-256color`.
- The settings listed themes as they're written in the file, `ember`. They're now
  listed by name, `Ember`.
- The settings offered every font on the system for the terminal, proportional ones
  too. The list now has only fonts whose letters are all one width.
- A theme name such as `../../x` no longer reads a file outside the themes directory.
- `long-command = nan` in the settings closed panes as their commands ended.

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
