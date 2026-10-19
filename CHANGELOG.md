# Changelog

What changed in each release of insensical, newest first. Each release's section is also
its release notes.

## Unreleased

## 0.5.0 - 2026-10-19 21:16 +06:00

### Upgrading from 0.4.0

What you'll meet differently, and what to do:

- **A device paired with the hub reaches every machine joined to it**, with what it may do
  on the hub and no code from that machine. Removing a device, or changing what it may do,
  holds on every joined machine at once.
- **A joined machine no longer pairs your own devices.** There, `isc hub device add` pairs
  only a guest kept to one project or terminal, and that guest's notifications are the only
  ones it keeps. Its Hub page has no Add a Device, and the window's sidebar says the Hub is
  joined where it said Running.
- **Joined machines are on your page by themselves.** A page paired with the hub lists
  every machine joined to it, connected or not, ready to open. There is no Add a Joined
  Machine and no machine code, and nothing to forget.
- **`isc web` is now `isc hub`**, with the same commands under it. The systemd unit is
  `insensical-hub.service`, and the Server page of Settings is Hub.
- **The hub's files moved** to `~/.local/share/insensical/hub` (on a Mac, `~/Library/Application
  Support/insensical/hub`), out of the settings folder. Pair your devices and join your
  machines again after this version; `config.toml` stays where it was.
- **Cookies, `isc web login`, `--trust-proxy` and `/enrol` are gone**, and `--local-visits`
  is refused. A phone or browser paired before pairs again with a code (see "Devices pair
  by a code").
- **`isc web token` is now `isc hub device`** (`add`, `ls`, `rm`, `set`). JSON output says
  device where it said token: `isc hub status --json` gives `devices`, and `isc hub push ls
  --json` and `push test --json` give `device`.
- **`isc hub device add --json` and `isc hub open --json` give the code** in `code`, as a
  link, beside the address. The QR code in Add a Device holds that link.
- **Interface colours changed.** The window is grey by default, and `isc tui` keeps the
  theme's colours. See "The window".
- **The agent limits are `agents-at-once` and `agents-deep`**, under Agents in the settings.
- **Settings has no Changed page.** A setting you've changed is on its own page, where the
  arrow beside it resets it.
- **`isc ls` gives each pane a shorter line**: number, state, program, directory, name or
  title and what its state is about, with `floating` or `minimized` only where it is. The
  size is left out; `isc ls --json` still has it.
- **`isc capture` prints only the end of a pane**, and `--trim` is gone. `isc wait --output`
  takes a regular expression, and a new project's name follows `--new-project` itself. See
  "Agents".

### The hub and your devices

- **Run a hub and join other computers to it.** `isc hub invite NAME` on the hub prints a
  line, and `isc hub join` on the other computer takes it. A joined computer listens on
  nothing, your devices reach it through the hub, and the hub can't read what passes.
- **Join a Hub is always on the Hub page** of the window and `isc tui`, on any computer
  that joined no hub. `isc hub join` and Join a Hub start `isc hub` if it isn't running.
  A joined computer whose `isc hub` has stopped says "Not connected: isc hub isn't
  running" and has a `Start` button.
- **Manage joined computers from the window, `isc tui` or the command line.** On a hub,
  **Machines** lists each computer with when it joined, removes one after asking, and
  **Invite a Machine** gives the line to paste. `isc hub machine ls` and `rm`, `ban`,
  `unban`, `leave` and `rekey` do the same, and `isc hub device rm --with-approved`
  removes the devices one added.
- **A joined machine offers only what a joined machine does.** Its Hub page says "Joined to
  the hub at" and the hub's name, with Leave and its devices, and no switch, link,
  Tailscale or public address. `isc hub open`, `invite`, `machine`, `ban`, `unban`,
  `tailscale on` and `public on` are refused there, with a line saying to run them on the
  hub.
- **Joined machines are listed apart from remote hosts.** Each says how long it has been
  connected ("Connected for 2h") and whether it is now. `isc hub machine ls --json` gives
  `connected`, and `isc hub status --json` on a joined computer gives `joined`.
- **Devices pair by a code and are known by their key.** Open the code's link, paste it,
  scan it or type its twelve groups of five on the pairing screen. The browser keeps a key
  of its own and stays paired until you forget it or remove its device.
- **Typing a code on a phone is forgiving.** The field groups letters as you type and
  counts them ("48 of 60"), and a refusal keeps what you typed and says what's wrong.
  **Scan the Code** comes first, and **Pair** fills in once the code is whole.
- **Approve a phone from one that's paired.** **Approve a Device**, in the page's
  settings, lets in a new device that shows its own code, with no more than you may do and
  at most three an hour. Both screens show the same three groups of letters, and you turn
  on **The same as on the device** before **Approve** works.
- **A device that may manage a computer can add another device**, giving it no more than it
  has, at most three an hour. The computer says who added it, and a device whose end you
  change later is held to the new end at once.
- **A home-screen app on an iPhone stays paired** and opens at the page itself. One that
  isn't paired yet says it pairs apart from Safari, with no error.
- **Pair a device again without removing it.** `isc hub device pair-again NAME`, or **Pair
  Again** on a device in the window, `isc tui` (`p`) and the page, makes a new one-use
  code. The device keeps its name, icon, colour and rights, and the old browser works
  until the new one pairs.
- **Devices have an icon and a colour.** A device shows as a phone, tablet, laptop, desktop
  or browser on a tile of one of ten colours. Both are guessed until you choose, in Add a
  Device or with `--icon` and `--color` (`--no-icon` and `--no-color` take them away).
- **Add a Device is clearer.** It shows the code typed as well as to scan, says when it's
  used, closed or run out, and says in red when the code would lead a phone to an address
  it can't reach. Each device lists the key of each browser and who added it, and the code
  is drawn larger in a terminal with room.
- **Reach every machine joined to your hub from one page.** Each machine is a heading in the
  list, with "not joined" beside one that isn't connected. A page at a Tailscale name whose
  link drops says Tailscale on the device may be off.
- **A computer joined to your hub notifies your phone.** Turn notifications on in a device
  paired with the hub, and a pane that waits, fails or finishes on any joined computer
  reaches it, with that computer's name first. Press one to open that pane. **Tell this
  device** covers every joined machine, and a guest hears of none.
- **A joined computer can't crowd out the hub.** Its devices get a share of the hub's list
  of its own, its notifications are posted in turn, and a computer whose share is full is
  named by `isc hub status`, `isc hub machine ls` and the Hub page.
- **Removing a device takes hold at once.** A removed device can do nothing more from its
  page. A change asked from a page is made only if it arrives while the page is shown, and
  one held back and delivered later is refused. `isc hub ban ID` also bans a device paired
  only with the hub's own computer.
- **A device keeps a limited number of sessions.** One known by its key holds 16 sessions
  and 32 connections at once; one given one project, or made for the public address, holds
  2 and 8.
- **If the hub can't read `members.json`**, it goes on with what it read last and changes
  nothing until it can, rather than lifting every ban. `isc hub open` no longer puts the
  address on the command line of the program that opens your browser.
- **`isc hub setup` walks you through reaching this computer** from a browser or a phone,
  in a terminal. It asks whether this computer is the hub or joins one, how you'll reach a
  hub, whether to start it after a restart and whether to add a device, then ends on the
  address and the plain commands that do the same. A name typed with `http://` is refused.
- **`isc hub device add` opens a form in a terminal** when you give no `--name`, and ends
  on the code to scan. With `--name`, `--json` or no terminal it works as before.
- **`isc hub autostart on` makes the hub start by itself, on a server too.** It sets up a
  service for your user, moves the running hub into it and enables lingering, saying each
  step. `off` removes only a service it wrote, and `status` says whether it's on. On a Mac,
  opening insensical starts the hub again.
- **`isc hub tailscale on` does what the Tailscale switch does** from the command line. It
  prints the address and whether the hub comes back after a restart. `off` and `status`
  go with it, and `isc hub status` says when Tailscale serves a different port than the
  hub's, which used to show up as a 502.
- **`isc hub status` no longer says now and then that the hub isn't running** while it is,
  and `tailscale on` no longer starts a second one.
- **The hub moves to a newly installed version** whenever your daemon has. Restart the
  server after installing and the next browser gets the new page.
- **A hub and the computers joined to it stay idle** when nobody is using them, and a phone
  that opens the page gets its first screen at once.
- **What you type to another machine goes only while it answers**, within a second and a
  half. A slow line no longer lets keys land seconds after the screen they answered.
- **The hub raises its limit on open files** as the daemon does, so many devices don't run
  it out on a Mac.

### Allow and Deny

- **Insensical asks you before a program in a pane changes who can reach this computer.**
  `isc hub device add`, `isc hub device set --far`, `isc hub open` and the device step of
  `isc hub setup` ask first, in the window or `isc tui`. Nothing changes unless you press
  **Allow**; Enter denies, and so does leaving it for two minutes.
- **It asks even when the program leaves out `INSENSICAL_PANE`** or names another socket,
  and on a Mac too. If it can't reach insensical to ask, the command says so and is
  refused.
- **With no window open, you're told on the desktop.** A notification and a mark on the
  tray icon or menu bar appear, and a click opens a window with the question.

### Agents

- **`isc upload PANE FILE` gives an agent your files.** Each is kept in the pane's
  directory, in `.insensical-uploads` (which git ignores), and its path is typed at the
  cursor. A file is 50 MB at most.
- **Agents start and talk to other agents.** `isc agent start codex "Review the diff"`
  opens Codex in a new tab with that task, and `isc agent wait` returns `done` and its
  reply, or `waiting` and its question. `isc agent tell`, `reply` and `answer` carry the
  next prompt, the last reply and a permission answer, and `--quiet` asks a small question
  with no pane. It works between Claude Code, Codex and Gemini CLI; `isc guide agents`
  teaches it.
- **Agent limits keep it in check.** One agent starts 8 at most, going 2 levels deep, both
  set under Agents in the settings. A message is 64 KB at most, and `isc agent wait`
  waits 30 minutes unless you give `--timeout`.
- **`isc agent start` checks first.** It refuses an agent whose hooks aren't set up and
  asks you to set them up. It starts agents found in `~/.local/bin` or `/opt/homebrew/bin`
  even when the daemon began from the Dock, and leaves your keyboard where it is; `--front`
  brings the agent forward.
- **A started agent that hasn't heard from its hooks says `no word yet`.** After 30 seconds
  its pane waits for you, saying what's likely (untrusted hooks in Codex, an untrusted
  folder in Claude Code), and `isc agent wait` fails. After you set up Codex, **Open Codex
  to trust them** opens a tab where `/hooks` trusts them.
- **Started agents are listed under the agent that started them** in the sidebar, joined by
  a line. In the activity list a started agent's row says `waiting · from Claude`, and its
  starter's says `working · 3 started`. A permission question says what the tool would act
  on, as `Needs permission: Bash · git push origin main`.
- **A message goes in only while the agent still runs in its pane.** After it has quit, a
  message is refused rather than typed at the shell.
- **Coding agents can use insensical.** Ask in your own words, and the agent opens tabs and
  panes, runs things beside it, reads what a pane shows and types into one you named.
  `isc integrate` and the top bar's "Set up" give each agent a skill and a note beside its
  hooks, `isc integrate --status` says what's current, outdated or missing, and `isc guide`
  prints what the agent is taught. Agents set up before are offered the new parts.
- **`isc tui` offers to set up the agents it finds**, as the window does, with "Set up 2
  agents" at the end of the sidebar. The question lists every file it changes before
  `Enter` sets them up, and `N` says never.
- **A pane can keep agents out.** Choose Keep agents out of this pane in the palette, or
  run `isc protect PANE`. A program in another pane can then neither read it, wait on it
  nor type into it, and is told so with exit status 4. Its header and sidebar row show a
  crossed-out robot, and only you let agents in again.
- **Keeping agents out is hard to get round.** A terminal a program opens from a pane is in
  that pane, `setsid` doesn't take it out, and a program can't answer its own pane's
  prompt or notify in the name of a pane that keeps agents out. The manual says what still
  gets past it.
- **`isc capture` prints just the end of a pane**, at most 4 KB, and says how much was left
  out. `--last` prints the latest command's output and how it ended, `--screen` what's in
  view, `--tail N`, `--grep PATTERN` and `--context N` cut it further, `--since TOKEN`
  prints only what's new, and `--all` prints everything. `isc history PANE` lists the
  commands run in a pane, with when, how long and how each ended.
- **`isc wait` and `isc ask` wait for the right thing.** `isc wait --prompt` waits for the
  command at a shell to end and exits with its status, `--quiet SECONDS` for a pane to go
  silent, and `--address` for an address to show. `isc ask PANE TEXT` types a line, waits
  for the answer and prints only it. A pane that's printing isn't typed into unless you say
  `--anyway`.
- **`isc run` and `isc split` do more.** They take `--name`, `--background`, `--wait` (print
  the end of the output and exit with the command's status), `--keep` and `--json`. A pane
  opened from another remembers it, `isc ls` shows `opened by pane:3`, and `isc close
  --opened` closes what the pane you run it in opened, leaving a pane you've used.
- **`isc ls` can filter.** `--program NAME`, `--state STATE`, `--opened` and `--here` list
  only matching panes. Wherever a command takes a pane or a tab, its name works as well as
  its number; a name that fits several lists them and exits with status 3.
- **More agents are known.** Cursor CLI, Copilot CLI and Hermes now report what they're
  doing, and `isc integrate` finds them. Cursor CLI can't say when it's waiting, and Hermes
  needs `hermes plugins enable insensical` afterwards.
- **About six hundred programs have their own icon**, up from about two hundred: agents,
  databases, clouds, build tools, languages and everyday apps. The manual's "Tile" table
  names the main ones.
- **Windows with many agents stay light.** The window repaints only an agent's turning mark
  while it works, and the activity list no longer slows with hundreds of panes.

### The window and `isc tui`

- **The app icon is a raised terminal chip on a glossy blue tile**, and on a Mac the menu bar
  shows the chip in its own colours.
- **The window is grey by default**, near-black on a dark desktop and near-white on a light
  one. `interface-style = "terminal"` (the switch Follow terminal theme) gives you the
  theme's colours again. The window asks your desktop to blur what's behind it; turn that
  off with `background-blur = false`.
- **Settings are calmer.** Names are brighter, cards stand further apart, and a setting has
  a few quiet words under its name only where the name doesn't say it all. Each remote host
  is one card that folds to its first row, and healthy hosts start folded where you have
  several.
- **The settings' icons are drawn for their tiles**, and machines have icons of the same
  make: server, laptop, desktop, mini PC, NAS, board, phone, container or cloud. A project
  can take one of thirteen marks (`isc project icon --mark`), and `isc host set --icon`
  takes the new machine icons.
- **Dialogs over the settings sheet are calmer and keep the keys.** `Tab` goes round the
  dialog's own controls, `Space` and `Enter` press only its buttons, and closing it gives
  the keys back. Cancel is grey, and only the button Return presses is filled.
- **`isc tui`'s settings are calmer.** A switch is one row with its knob at the end it's
  turned to, a two-value setting marks the one in force with a filled dot, and what a
  setting does is said under its row. Wide enough, the headings are listed down the left.
- **`isc tui`'s settings have three pages**: **Settings**, **Remote Hosts** and **Hub**.
  `1`, `2` and `3` go to a page. Hub does what the window's does, and Remote Hosts adds,
  renames and forgets a machine and answers what `ssh` asks of it.
- **The window speaks to screen readers.** Buttons, tabs, the sidebar, switches, choices,
  menus, sheets and questions have names and states, and a notice is read out as it
  arrives. `Tab` and `Shift+Tab` stay inside a sheet, and the settings' sections turn with
  `Ctrl+Tab` and `Ctrl+Shift+Tab`. `motion = "system"` follows the desktop's
  reduced-motion setting.
- **Nothing steps to its next value on a press.** A choice of two shows both words and you
  click one, a choice of more opens a menu under it, and on and off is a switch. A number
  has − and +. In `isc tui` the arrows, Home and End move in a list, Enter chooses, and
  Esc closes it.
- **Menus look and work alike.** The `+` menu and dropdown settings open the same solid
  menu, walked with the arrows, Home and End, Enter and Escape, or a letter. A tall menu
  scrolls and keeps your row in view. A machine's colour and icon, a new device's rights
  and a share's end are chosen from a dropdown.
- **The top bar and the sidebar are tidier.** With tabs along the top, the bar says where
  you are and is 4 pixels shorter, giving the panes more room. Drag the sidebar's right
  edge to make it 190 to 520 pixels wide, and the next window opens at that width.
- **Every field is a proper field**, in the window and in `isc tui`. The caret moves by
  character and word, Shift selects, and typing over a selection replaces it. There is
  undo and redo, and in the window copy, cut and paste, and double and triple press select
  a word and a line. A secret shows dots.
- **The palette's switches say which way they go**, as in Silence this pane's notifications
  or Let this pane's notifications through. Either way's words find it.
- **The window has a title, `insensical`.** A common Hyprland rule for empty popups no
  longer catches it and shows your wallpaper sharp instead of blurred.

### The browser page

- **Home**, the first tab on a phone, shows every project as a card in a section for each
  machine. A card says what runs there, and a tap opens the pane that needs you, else the
  one at work, else the one you used last. With nothing waiting, **All quiet** is a line at
  the top. Home no longer slides sideways on a phone.
- **Add photos and files to a pane from your phone or browser.** The paperclip in a pane's
  bar, or its header on a wide screen, opens your photos and files. Each is kept on the
  pane's machine and its path is typed at the cursor, so an agent there can read it.
- **This browser**, the page's menu, is laid out as a phone's own settings are. It has
  one-line rows in groups, **Icon** and **Theme** on pages of their own, and each device
  on one line in **Settings**, with **Pair Again** and **Remove** on its page. Dialogs with
  a field open at the top of the screen so the keyboard doesn't cover them.
- **In the page's list**, a tab's panes hang from it by a thin line and sit further in, so
  you can tell which pane is whose. A tab that is one pane is listed level with the others.
- **A notice on a phone** such as "arch-3s06 added" is drawn above the Everything and
  Activity switch, not over it.
- **A pane you open while the page is reconnecting** has its bar of keys at once, instead of
  hiding it until you open the pane again.
- **A paired phone or browser isn't asked to pair again** when the hub just can't be reached.
  The page says "Can't reach" and tries again by itself.

## 0.4.0 - 2026-10-13 05:17 +06:00

This release takes insensical beyond one desk. `isc web` puts your panes in a browser or
on your phone, where you can read them, answer what waits and be told when something
needs you, and you can share one project or one terminal with someone else. One window
shows the machines you've named beside this computer, reached through your own `ssh`,
and finds the ones on your tailnet. Panes say what they're doing in words, the sidebar
can show only what's active, and notifications no longer flood. You can also write
commands and key modes of your own, keep a project's layout in a file, and see pictures
in more places.

### Upgrading from 0.3.0

This version has a newer protocol (27, which `isc version` prints) and a newer terminal
engine than 0.3.0's. The window, `isc` and the server all have to be this version, on
one machine or two, so update both. After installing it, restart the server from the
window, or with `isc restart-server`. What runs in your panes carries on. Because the
engine changed, each screen is carried across by redrawing it: you keep the text,
colours, history and cursor. A full-screen program draws itself again the next time it
draws.

What you'll meet differently:

- **After the leader, `v` selects with the keyboard and `Shift+S` splits right.** `v` used
  to split right, and `s` still splits down. `"leader v" = "split-right"` under `[keys]`
  puts it back.
- **Your keys tables lie over the defaults.** `[keys.direct]` and `[keys.leader]` no longer
  replace them, so a key added in a new version works with an older file. Taking a line
  out brings its default back; to leave a key to the programs in a pane, give it `"none"`.
- **The sidebar's field only filters.** Enter there no longer makes a project. Use
  `Ctrl+Shift+N`, the `+` button or the palette.
- **The count of panes that need you has moved from the bell to the activity button**, and
  counts only panes that wait or failed. A finish is counted beside it, with a tick.
- **You're told of a finish 15 seconds after it, and of many panes at once in one
  notification.** See "Knowing what needs you".
- **`--host` and `--rights` are `isc`'s own wherever they stand.** `isc send 3 --host` typed
  `--host` into the pane; now it's the option. Put `--` before text that is one of `isc`'s
  options: `isc send 3 -- --host`. In `isc notify`, `--test` and `--sinks` are options now
  and were a title. Other text that begins with a hyphen is still text.
- **A row of `isc ls` has more after the pane's title**: what a waiting pane waits for and
  for how long (`wants permission, 2h`), the name a program gives itself, in brackets, and
  how many things it reports. Everything up to the title is as it was. A script that reads
  the title as the rest of the line gets those too; `isc ls --json` is for scripts.
- **A state is put in words a little differently.** "working" stands where "running" did
  wherever a state is put in words; `isc status running`, a row of `isc ls` and the events
  are as they were. The words after a state are set off with a dot, where a colon was. A
  command that failed has the note "exited with 2 after 1m 35s", where it said "failed
  with". `isc status explain` is worded differently; it's for reading, not for a script.
- **No words the interface shows carry three dots.** Buttons, palette titles, choices and
  prompts say what they say and stop: "Restart Server", "Add Remote Host". The button that
  was "How" with three dots is "How to Install".
- **In the interface in a terminal, a new project's directory is written from `/` or `~`.**
  `./api` is a name now, where it was a directory beside where the interface was started.
- **`isc version` prints the engine as `ghostty-3425025e`.** Its name no longer has the
  release in it: two releases work together when their protocol and engine are the same,
  and one that changes neither is installed without restarting the server.
- **The tarball, the `.deb` and the AUR package carry a statically linked `isc`**, in place
  of one linked to the system's C library.
- **On a Mac, keys are shown in signs alone.** See "On a Mac".
- **A saved layout of more than 4 MB of text, or 4,096 panes, is set aside** as
  `state.json.unreadable`, and the server starts empty. See "Fixed".

### From a phone or a browser

- **insensical in a browser.** `isc web` serves a page you can open on a phone or in any
  browser: your panes with their states, to read, scroll back through and type into. It
  runs only when you start it and listens on this machine only, so you reach it through
  `ssh -L`, `tailscale serve` or your own proxy; the daemon still listens on no network.
  Nothing typed blind reaches an agent: the page says **Link lost · trying again** two and
  a half seconds after it last heard from your machine, and what you type arrives within
  that or never. `Ctrl`-click (`⌘`-click on a Mac) opens a web address in a new tab; on a
  phone a tap shows a sheet with the whole address, **Open** and **Copy**. After you update
  insensical, `isc web` starts itself again as the installed version on the same port, and
  a page left open says a newer one is ready. See "In a browser" in the manual.
- **A phone is told, and answers.** The page opens on what needs you: a card for each pane
  that waits, failed or finished, with the program's own question, how long it has waited
  and the live foot of its screen, and under it the keys agents ask for and a line you can
  dictate. Add the page to your phone's home screen and it can notify you when a pane needs
  you, through the phone's own push service and nothing of ours; a press lands on that
  pane's card. The title, **Needs you** and the home-screen badge count the panes that wait
  or failed. An answer pressed while the link is lost isn't sent and isn't kept: the card
  says **Not sent**.
- **A browser is paired with a token, and a device can end by itself.** `isc web token add`
  prints an address and a code to scan. The address pairs one browser, for ten minutes, and
  that browser stays paired; `--uses N|any` and `--expires 10m|1h|7d|never` say otherwise.
  `--rights look` makes a device that only looks. `--stays 1h` (or minutes, days, `always`)
  says how long the device stays paired: then its browsers are signed out and the token
  leaves the list. `isc web token ls` says how many more browsers an address pairs and
  "ends in 40m", and `isc web token rm` ends a device and closes what it has open. A device
  made from the page never outlasts the browser that made it.
- **A phone's home-screen copy pairs by itself, and the page scans a code.** A code that
  pairs a device also pairs its home-screen app, once, within fifteen minutes and from the
  same network address. **Add to Home Screen**, in the page's menu on an iPhone, makes a
  one-use address for it, good for ten minutes, and a camera button on the pairing page
  reads a code. **Header looks blurred** moves the header below the edge iOS 26 blurs. A
  browser pairs its home-screen copy at most three times in ten minutes, and the server
  posts notifications only to public addresses of a push service on port 443.
- **`isc web open` opens insensical in your browser on this computer, signed in.** It makes
  an address that works once, for 10 minutes, and only on this computer. The browser is
  paired as the device `This computer`, with every right. `--print` prints the address;
  `--json` prints it with when it ends and opens nothing.
- **The page has a Settings screen for the computer it's paired with.** A browser whose
  token may manage finds it in the menu under its letter: this computer, your devices (add
  one with a code to scan, remove one), who is sent notifications, the alerts when you're
  away, each with a test, and who is connected, with Disconnect. An alert's address is
  shown shortened and read-only, with **Change**.
- **A pane opens at your phone's width.** Its columns and rows are what the screen has room
  for, and follow a pinch of the letters and the phone's keyboard. A button in the pane's
  header switches whether the pane has your phone's size; leaving the pane gives the size
  back. A bar has the keys a phone's keyboard lacks. **Text size** in the page's menu draws
  terminals from 9 to 28 pixels and is remembered.
- **At a desk, the page is the sidebar and the tab as it's laid out.** It draws panes in
  your `font-family` and `font-size`, with the signs of a Nerd Font. Drag the border
  between two panes to resize them, and a pane by its header onto another to swap the two
  or put it at one side; a token that may not arrange has neither. Each pane's header has a
  menu with Find and Copy everything.

### Sharing with someone else

- **Share one project or one terminal with a guest.** An eye on a project's row and in a
  terminal's header, and "Share this project" and "Share this terminal" in the palette,
  open one dialog: what they see, whether they look or also type (look unless you change
  it; typing asks you to agree to a warning first), and when it ends (a day unless you
  change it). It makes a link and a QR code that work once. What's shared carries a small
  sign, the same button lists who it's shared with, and `End` signs a guest out. From a
  terminal it's `isc web token add --name sam --only project:api --stays 1d`, or `--only
  pane:12`, with `--rights look,type` to let them type; on the page it's **What it sees**
  under **Add a Device**. A guest gets no notifications and no Settings. The share is to
  the project's number, so renaming it doesn't move it. A guest who can type in a shell has
  your account from that shell: the command says so. `isc web token ls`, `isc clients` and
  the Server page say what each guest sees.
- **A guest's page shows what you shared and no sign of anything else.** A bar says "You're
  a guest on desk. You can look at api." and when it ends, and under it is only that
  project or terminal: no "Needs you", no "Everything", no Settings, no bell, no "Fit to
  this screen". When you close what you shared, remove the device or its time is up, their
  page says so and stops.
- **Share with someone who isn't on your network.** In the Share dialog, "Where they are"
  has "Anywhere on the internet": the guest can only look, the share ends in a week at
  most, and the dialog says what it opens before you press `Open a Public Link`. While one
  is open an amber sign in the window says so, and the Server page has a `Public link` row
  with `Close`; ending the last public guest closes it too. From a terminal, add `--public`
  to the guest's `isc web token add`, then `isc web public on`. The address is your own
  Tailscale Funnel's: nothing of insensical's listens on the internet, and your own devices
  don't work there. `isc web public off` removes those devices and always closes the
  address, and fails if Tailscale won't close it. `isc web public status` says how it
  stands by what Tailscale says; `--json` gives it as data, as does `door` in `isc web
  status --json`. `isc web` takes `--guest-name` and `--guest-port` for it.

### Other machines and your tailnet

- **Another machine's daemon, from this one, through your own `ssh`.** Name the machine
  once, `isc host add build me@build.example.com`, and put `--host build` before any
  command: `isc --host build ls`, or `isc --host build` alone for the whole interface in
  your terminal. Your ssh configuration, keys, agent and jump hosts apply, and the daemon
  there opens nothing to the network. It needs the same version of insensical on both.
  When the machine stops answering, the interface says so on a card over dimmed panes and
  tries again by itself, up to every thirty seconds, or at once when you press a key. A
  command that ends something acts on another machine only when you say `--host`: with
  only `INSENSICAL_HOST` set, `isc kill-server`, `close`, `tab close`, `project close` and
  `project sleep` are refused.
- **Other machines in the window.** One window shows this computer's projects and those of
  every machine you've named, each under its name in the sidebar. A pane on another machine
  carries the machine's chip in its colour. Going to the pane that needs you, the filter,
  the overview, the palette and the counts span machines, and a window opened from the tray
  comes to where you last were. Only you change the machine in front: another machine can
  say a pane needs you, and can't raise the window or take the keyboard. The window never
  stops to wait for a machine, and if this computer's server stops, the others carry on. In
  the interface in a terminal, "Go to a machine" in the palette lists them.
- **Add a machine, and put insensical on it, from the window.** "Add" at the foot of the
  sidebar, "Add a machine" in the palette and `Add Remote Host` in the settings open one
  dialog. Under its Address field are two tabs, `Tailscale` and `SSH config`, each listing
  its machines; a tailnet machine shows its system and whether it's on. The machine you
  choose is met in four lines. If insensical isn't there, one button puts it into
  `~/.local/bin`, after saying what it will write; a server running there is restarted only
  on a second yes. A machine with another version says so and offers this one. `isc host
  check` is that meeting with how long a round trip takes, `isc host put` puts this `isc`
  on a Linux machine, and `isc host restart NAME` restarts the server there; use it instead
  of `ssh NAME isc restart-server`, which often can't find the server. `isc host set` and
  `isc host add` give a machine a colour, an icon and a mark that it's one to be careful
  on. Where `ssh` goes is written in letters, digits and `. - _ + @ : % [ ]`. Renaming a
  machine keeps its link, and a settings file saved with a mistake leaves your machines as
  they were.
- **A machine of your tailnet is reached at its address, whatever its name becomes.** `isc
  host add`, and a choice from the `Tailscale` tab, keep the machine's node. Each time it's
  dialled, Tailscale is asked where that node is and `ssh` goes to its address, with the
  host key filed under the name you typed. A machine Tailscale says is off is said to be
  off at once. `--no-node` keeps a machine as typed, machines you already have are dialled
  as before, and `isc host set NAME --ssh WHERE` looks for a node. `INSENSICAL_TAILSCALE`
  names the `tailscale` to run, as `INSENSICAL_SSH` does for `ssh`.
- **Where `ssh` has a question, you're told which, and can answer it.** A host `ssh` hasn't
  met, a key that changed, a key or password wanted, a sign-in Tailscale wants, a user its
  rules refuse, a machine that's off: each has its own words and one thing to do. `Answer`,
  in the dialog, the sidebar and Remote Hosts, opens a tab on this computer where `ssh`
  asks you itself, and the machine is met again when that ends well. `isc host answer NAME`
  does it at your terminal, and `a` in "Go to a machine" in the interface in a terminal. A
  key that changed is never answered for you.
- **What belongs to the other machine is asked of it.** The file finder lists that
  machine's files, a new project's directory is a directory there and `~` is your home
  there. `isc --host build integrate` sets up the agents there, and `isc --host build
  config` changes the settings there; the settings sheet does both for the computer you
  choose. A path you click in a pane there opens in an editor in a pane there, or is
  copied. A pane's history comes a part at a time: the screen and the newest half megabyte
  at once, and four megabytes more each time you scroll to the top. The daemon holds 256
  MiB of history waiting to be fetched at most.
- **A machine marked careful says so, and nothing typed blind is delivered.** A careful
  machine has a filled chip on its panes, and a paste of more than one line there asks
  first; the interface in a terminal says `careful` beside its name and asks too. The mark
  comes off only when the settings file is read and says so. When a machine's link is lost
  its panes keep their last screen, dimmed, and say so, four seconds after the machine last
  sent anything. A key pressed there is refused, and what you typed just before arrives
  within two and a half seconds or never, so nothing turns up at an agent when the network
  returns.
- **Typing on a far machine is drawn at once.** When the round trip takes 70 milliseconds
  or more, the letters you type show as you type them, a little fainter, until the program
  shows them. Nothing is drawn until the program has shown two letters you typed, so a
  password is never drawn. `type-ahead = "off"` turns it off. The page `isc web` serves
  does the same.
- **A paired device can be let through to your other machines.** `isc web token add --far`,
  or `isc web token set NAME --far`, lets its browser be carried from this computer to the
  machines of `isc host ls`, over your own `ssh`, with the rights it has here. It's all of
  them or none, only for a device that may type, and never for a guest. On the page each
  machine is a heading over its projects, and its terminals join the list, **Needs you**
  and the counts. A machine that can't be reached says so within ten seconds and comes back
  by itself, and one machine can't pass for another. The `ssh` it runs forwards nothing and
  ends with `isc web`.
- **Your phone is told about your other machines too.** A device with `--far` that has
  notifications on hears when a pane waits, fails or finishes on any of them, with the
  machine's name first. Each machine decides for itself what's worth telling, and you're
  never told that a machine went out of reach or came back. It keeps one `ssh` open to each
  machine meanwhile: about 15 KB a minute each when nothing happens. `isc web push ls`
  marks the devices that hear them, and `isc web push test --at NAME` sends a test as from
  one.

### Knowing what needs you

- **A program can say what it's doing in its own terminal, and its pane shows it.**
  insensical reads the Program Status Protocol (OSC 7501): `idle`, `working`, `blocked`,
  `done` or `error`, with a line of words. It needs no hook and nothing installed, and it's
  heard over `ssh` and from inside a container. Claude Code 2.1.295 speaks it unasked. A
  program is shown by the name it gives itself, so Claude Code, which runs as `node`, has
  its own tile. An agent that has insensical's hooks as well isn't heard twice, and `isc
  status explain` says which was believed. A program that reports several things at once
  has its pane show the most urgent, with a count such as "3 tasks · 1 waiting". A program
  that asks whether it's heard is told which states and kinds are read (revision 0.4).
- **A row says what its pane is doing, in words, and for how long it has needed you.**
  Under a pane's name: "wants permission", "asks a question", "needs you to sign in",
  "waiting", "failed" or "done", then the program's own words, on two lines at most and
  whole under the pointer. Beside its mark: `now`, `2m`, `1h`, `3d`, bold once a pane has
  waited ten minutes. The header, the overview, the palette, notifications, the tray's
  menu, `isc ls` and the interface in a terminal say the same. How long a pane has waited
  is kept when the server moves to a new version and not when it stops. `isc ls --json` and
  `isc api '"State"'` have `kind`, `app`, `tasks` and `since` on each pane and `at` on the
  state; `isc events` has `kind` and `app` on its `opened` and `status` lines, and a new
  `kind` event when what a pane waits for changes.
- **The sidebar can show only activity.** A button at its head (`Ctrl+Shift+Y`, or
  `activity` in the palette) leaves just the agents and the panes with something to say,
  each with its state in words and how long, what needs you first. The list is kept quiet:
  a row doesn't flicker as an agent pauses, nothing moves under your pointer, and what
  finished goes a few seconds after you've looked at it. `Ctrl+Shift+J` and `Ctrl+Alt+/`
  walk it (`next-activity`, `previous-activity`), and `activity-idle = false` leaves out
  agents at rest. The interface in a terminal has it by the same rules (`Ctrl+Space` then
  `Y`; `}`, `{` and `A` walk it; `[tui]` can set `activity-idle` apart), and so has the
  browser's page, with a switch your browser remembers.
- **The same number of panes need you wherever it's shown.** A pane needs you when it waits
  or failed. The button, a machine's heading, a project's row, the tray, the menu bar and
  the interface in a terminal no longer count panes that finished: a finish is told of
  beside the count, with a tick. The key that goes to the next pane goes to the ones that
  need you first, then to the ones that finished.
- **A long command at a shell is activity too.** With shell integration, a command that has
  run for 10 seconds is listed while it runs, with what you typed and for how long, and
  ends done or failed like any pane. `isc ls --json` has `running_since` and `running`.
- **You're told of a finish once it has stood, and never of a pause.** An agent that waits
  on background work ends its turn and wakes by itself, and each time used to be a
  notification. Now a finish is told of after the pane has stayed done for 15 seconds, and
  a pane that has paused is waited on for longer than its pauses, up to ten minutes: you
  get one notification at the end, with its last words. The row still shows `done` at once,
  a pane that waits or failed is told of as quickly as before, and a pane's first pause
  longer than 15 seconds is still told of.
- **Many panes that stop at once are one notification.** Ten agents that reach a usage
  limit in the same seconds are one, `10 panes failed`, with their words under it. A pane
  that fails the same way again says nothing more until it says something else, ten minutes
  pass or you've looked at it. One pane alone is told of a second after it changes.
- **The daemon can tell you when nobody is attached.** Set `notify-command` to a program,
  or `notify-url` to a web address, and the daemon runs the one and posts to the other:
  enough for ntfy by itself, and for anything else through a script. They're sent only
  while you're away unless you say `always`, under the same mutes and limits. A topic that
  needs signing in takes `notify-url-token`, or `notify-url-user` with
  `notify-url-password`; they're shown masked and never sent over plain `http://` to
  another machine. `isc config set --stdin NAME` reads a value from standard input. `isc
  notify --test` tries them and `isc notify --sinks` says how the last one went. `https` is
  built into `isc`: it needs a processor from about 2014 or later, or on ARM one with the
  AES and SHA-2 extensions, which a Raspberry Pi 4 lacks.
- **A pane stuck in `running` or `waiting` can be cleared in one step.** What a program
  said it was doing ends with the program. Where insensical can't see a program go, such as
  on the far side of `ssh`, the pane goes on saying it: `isc status clear`, or "Forget what
  this pane's program said it was doing" in the palette, puts the pane at rest. insensical
  never ends a state for having been silent.

### Projects, panes and the overview

- **A project can be written in a file and opened in one go.** Put a `.insensical.toml` in
  a project's directory and `isc project open` makes its tabs, splits, floating and
  minimized panes, each running its program in its directory. Two panes side by side is one
  line, `columns = ["nvim", "cargo watch -x test"]`, and a list inside a list stacks.
  Commit the file and everyone who clones the repository opens the same project. `isc
  project save` writes the project in front as such a file. A file that starts programs is
  listed and asked about the first time, and whenever it has changed; a mistake is reported
  with its line and nothing is made. The window opens them too, with "Just the directory"
  for a plain project, and the palette has "Open a project file" and "Save this project as
  a file".
- **An agent's team opens as panes here.** Some agents start helpers in tmux panes, and
  only inside tmux. With `tmux-shim` on (`isc config set tmux-shim true`, or "Answer as
  tmux" in the settings), a program in a pane finds a `tmux` that insensical answers: its
  helpers open beside it in its own tab, as panes you can move, name and be told about.
  It's off unless you turn it on, and while it's on `tmux` in a pane isn't the real one.
  The manual's section 11 has the commands and the limits.
- **The overview shows what's on every pane.** Each pane has its screen, small and live, so
  you can tell an editor from a build from a waiting prompt. A pane that needs you has its
  question over the picture, and the strip along the foot shows the last lines of one pane
  at full size. One pane has a ring, which is where Enter goes; Space moves it. The screens
  are live only while the overview is open, redraw four times a second at most, and never
  resize a pane.
- **A name is changed where it's written.** The pencil on a project, tab or pane, and the
  palette's naming commands, turn the name itself into a field. Enter or a click elsewhere
  keeps it, Escape leaves it.
- **A new project can be made in a directory that isn't there yet**, on this computer or
  another. The card says which path it will make, and one more Enter makes it. `isc project
  new DIR --make` does the same.
- **Switching tabs no longer flashes empty panes.** The tab you left stays on screen until
  the new tab's panes have their screens, for 150 milliseconds at most.
- **A server with very many panes starts much faster on Linux.** With a thousand panes,
  starting the server again took 20 seconds or more and each new pane 18 ms. They now take
  about a second and 3 ms.

### The terminal and pictures

- **Select and copy with the keyboard.** `Ctrl+Shift+X` (`⇧⌘X` on a Mac), the leader and
  then `v`, or "Select and copy with the keyboard" in the palette, puts a cursor of your
  own in the pane. Move it with `h` `j` `k` `l`, `w` `b`, `0` `$`, `g` `G` and the Page
  keys, through the screen and the history; `v` selects, `V` whole lines, `/` searches, and
  `y` or Enter copies and leaves. `p` selects the path or address on the line, `o` the
  whole output of a command, `c` the command line. `Space` puts a letter on every path,
  address and commit in view, and typing it copies that one. Nothing you press reaches the
  program, and a strip in the pane says what the keys do. It works in `isc` at a terminal
  and in a pane on another machine, and not in a browser. A copy's note says how much:
  "Copied 14 lines".
- **Pictures show in more places, and stay.** Pictures a program places with Unicode
  placeholder characters are drawn, so Yazi and other file managers show their previews.
  The interface in a terminal shows what a program draws with the Kitty graphics protocol,
  where your terminal shows such pictures itself, cut at the pane's edge and taken away
  while a list, a question or a floating pane lies over them. The pictures in view come
  back after the server stops and starts, and stay when it moves to a new version with the
  same terminal engine.
- **The terminal engine is a newer Ghostty**, of 4 October 2026.
- **`isc wait --output` finds text that was printed before you asked** and has scrolled off
  the screen since. Text the pane's width broke across two lines still isn't found.
- A full reset of the terminal, as `reset` does, removes the progress a program reported.

### Commands, modes and `isc`

- **Commands of your own.** Under `[commands]` in the settings, give a name a list of
  steps: commands that exist, a program to run in a new tab, split or float, and text to
  type into the pane in front. Bind it to a key, find it in the palette, or run it with
  `isc do NAME`. A step that fails stops the rest and says which it was. A command that's
  written wrongly is reported and left out.
- **Modes, and a resize mode.** A mode is a set of plain keys that run commands until you
  press Escape. Resize mode comes ready: leader, `Shift+R`, then the arrows or `H` `J` `K`
  `L` as often as you like. A key that isn't the mode's turns it off and is typed. Write
  your own under `[modes]`; each is turned on by a command called `NAME-mode`.
- **`isc api` says what you can ask.** With no request it lists every request; `isc api
  NAME` says what one holds, with an example and what answers it; `--json` gives either as
  data. It needs no daemon.
- **`isc clients` lists who is connected, and `isc disconnect` puts one off.** Each client
  has its kind, machine, version, rights, process and pane, and what it only says of itself
  is marked as claimed. `isc disconnect N` needs the right to manage; the client can
  connect again, says you disconnected it and waits for you, and a paired browser stays
  paired. After a client changes something, `isc events` prints a `did` line that names the
  request and the client.
- **`--rights` asks the daemon for fewer rights**: `isc --rights look ls` can look and
  nothing else. The rights are look, type, arrange and manage. Only a client that can type
  counts as you: a pane open in one that only looks still notifies you. Without `manage`,
  `notify-url` is shown as its scheme and host and `notify-command` as its program's name.

### The interface in a terminal

- **More of the window's commands are in its palette, in the window's order.** "Choose this
  project's icon" (`project-icon`) lists the colours and marks a terminal can show, and
  keeps a drawing of yours unless you pick a mark or the folder. "Move this pane to another
  tab" (`move-to-tab`) lists the tabs. "Move this project up the list" and "down"
  (`move-project-up`, `move-project-down`) order the projects, as dragging a row does. The
  themes are listed, `terminal` among them.
- **It says when the server has gone away**, on a card over the list and the panes, and
  carries on by itself when a server runs on that socket again.

### The settings sheet

- **The settings sheet is a panel of cards, with a page for each thing you set.** It floats
  over the window with the pages at one side, a search field, and arrows back and forward.
  A setting your file changes has a dot after its name and an arrow that resets it. The
  pages are Server, Interface, Themes, Terminal, Shortcuts, Remote Hosts, Notifications,
  Agents and Changed. **Server** has this computer's server with `Restart Server`;
  **Access from your other devices**, with a switch that starts `isc web`, `Open in
  Browser`, a switch for Tailscale, the devices paired with `Add a Device` and a code to
  scan, how long each stays paired, and a test notification; and **Connected**, with
  `Disconnect` on each connection. **Remote Hosts** lists each machine with its state in
  words, changes where `ssh` goes, renames one in its row, and offers to update insensical
  there. **Notifications** holds what tells you, and how you're told when away, with a test
  whose result is a sentence. The computer the pages are about is chosen at the top right
  (`Alt+M` steps it). The Tailscale switch turns off only a mapping it made, and says so
  when your tailnet hasn't turned Serve on. `isc web stop` also stops the window starting
  it again.

### Installing

- **insensical has a new icon.** A rounded teal pane with two cursors for eyes, on a
  dark tile. It's the icon in your launcher, the tray and the Dock, and it's easy to tell at
  any size. In the tray an amber dot joins it on the pane's corner while a pane needs you.
- **`isc` by itself for arm64 Linux.** Each release has `isc-aarch64-linux` beside
  `isc-x86_64-linux`, for a Graviton or Ampere server or a Raspberry Pi with a 64-bit
  system, and the install line picks the one for its machine. The window is for x86-64
  only on Linux.
- **The install line takes a version and can be run by a script.**
  `curl -fsSL https://insensical.com/install.sh | ISC_VERSION=0.3.1 sh` installs that
  release. With `ISC_QUIET=1`, or where there's no terminal, it asks nothing, restarts
  nothing and never waits for a password. With `ISC_DIR` set, it makes the directory and
  needs no `sudo`.

### On a Mac

- **Keys are shown in signs alone**, in the order its menus use: `⇧⌘N` where it read
  `⌘+Shift+N`, with `⌃`, `⇧`, `⌥`, and `↩`, `⌫`, `⇥` and arrows. That's in tooltips, the
  palette, the sheet of keys, the settings sheet, and `isc` at a terminal on a Mac.
- **Tailscale is found from a window opened from the Dock**, which has only the system's
  `PATH`: where Homebrew and the Tailscale application keep it is looked in too.
- **`isc` run through a link finds the copy of itself for Linux that the application
  carries**, to put on a machine that has none.
- A shell script is shown by its own name, as on Linux, where it was shown as `sh`. A
  script run in a pane's shell's place, as an agent's wrapper is by `exec`, was taken for a
  shell at its prompt, and what the agent reported there was dropped.

### Fixed

- **A picture too large for its pane is shown whole.** A tall or large picture drawn by the
  Kitty graphics protocol used to have its top scrolled away or its side cut. It's now
  drawn smaller, in its own shape; a wide picture made smaller has empty rows under it.
  `isc tui` does the same, in whole cells.
- **Ctrl-C works in a pane whatever started the server.** A server started by something
  that ignores the interrupt or hang-up signal passed that on to every program in its panes.
- **A daemon no longer dies with the session that first started it**, such as a terminal
  tab or an `ssh` session. On Linux with systemd it runs in a scope of its own.
- **One mistyped setting no longer undoes the daemon's others.** Only that setting falls
  back, a file that's broken TOML leaves the last good settings in force, and you're told
  what was wrong, in a notification and in `isc notify --sinks`.
- **`isc attach` ends when the program in the pane does**, and gives the terminal back.
  `Ctrl+]` leaves also when a program has put your terminal into a keyboard mode that sends
  it as a longer sequence. The terminal's title says "Ctrl+] to leave", and `Ctrl+C` three
  times in two seconds shows a line that says so.
- A pane that waits for your permission goes on saying so when another tool the agent was
  running finishes, and says running again once you've refused and it starts its next tool.
- An agent that stops on its usage limit says "Usage limit reached", not `rate_limit`.
- A file in a folder that your group can write to is shown in your file manager instead of
  opened, even when the group is your own. `chmod g-w` on the folder opens them again.
- A program that asks to set the primary selection (OSC 52 with `p` or `s`) no longer
  replaces what's on your clipboard.
- Panes, commands and names can't add up to more than the server can send to a window: 4
  MB of text and 4,096 panes. Past that a new pane or a rename is refused, and says why.
- A pane's number is never given to another pane, however the server ended. After a crash
  the next number can be up to 256 higher than the last.
- A client the server cut off for falling behind connects again and says that's what
  happened, and one that sees the server move to a new version says so while it waits.
- A quiet pane you look at, or read with `isc capture`, no longer costs the server its
  whole history unpacked in memory: 12 MB for a full history, where 2 MB is what it takes.
- In a pane of more than about 46,000 cells, a full-screen program that scrolled from its
  first line could leave the pane without the shell's screen and history behind it.
- An icon a program prints right before a letter or another icon no longer lies under it.
- In the overview, Up and Down go to the nearest card in the row above or below, as it's
  drawn, and Enter on a sleeping project wakes it and goes to it.
- A file you put in `icons` yourself is never deleted.
- A theme marked in the settings' list stays on screen while you look at it, also when the
  settings file is read again just then.
- Two programs changing the settings file at the same moment no longer lose a change.
- Tab in a shell no longer hangs when the daemon doesn't answer: after a third of a second,
  panes, tabs and projects aren't offered and the commands still are.
- When a daemon's answer can't be read, `isc` says so and why. It used to call any such
  daemon too old to answer.
- The pencil on a pane's row no longer hides the last letters of what the pane's program
  says it's doing.

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
- **`isc` by itself: insensical with no window**, on a desktop or a machine that has
  none. Each release has
  `isc-x86_64-linux`: one file that needs nothing installed and runs on any Linux
  distribution. It's the daemon, the command line and the interface in a terminal.
- **One line installs `isc` by itself**, insensical with no window, on any Linux on
  x86-64: `curl -fsSL https://insensical.com/install.sh | sh`. It checks the download
  against its checksum, installs to `/usr/local/bin` (asking for your password if it
  needs it, or to `ISC_DIR` if you set it), and offers to restart a server that's
  already running, which ends nothing in your panes. Run it again to update. It came
  just after the release, with the website.
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
  off by default.
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
  from the window or with `isc`.
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
