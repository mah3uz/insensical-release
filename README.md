<h1 align="center">
  <img src="banner.svg" alt="insensical: a terminal multiplexer with a native window. Run many agents; know which one needs you." width="100%">
</h1>

A terminal multiplexer with a native window, for running many terminals and coding
agents and knowing which of them needs you. Linux and macOS.
<https://insensical.com>

This repository holds the releases. [`MANUAL.md`](MANUAL.md) is the manual, and
[`CHANGELOG.md`](CHANGELOG.md) says what each release changed.

## Installing

Every file named here is on the [latest release](../../releases/latest).

**Arch Linux**, from the AUR:

```sh
yay -S insensical-bin
```

**Debian 12 and later, Ubuntu 24.04 and later**, on x86-64:

```sh
sudo apt install ./insensical_*_amd64.deb
```

**Other Linux**, on x86-64 with glibc 2.36 or later: unpack
`insensical-*-x86_64-unknown-linux-gnu.tar.gz` and put `insensical` and `isc` in the same
directory on your `PATH`. The desktop entry, the icon, a user unit and completions for bash, zsh and fish are beside them.

**macOS 13 and later, on Apple silicon**, with Homebrew:

```sh
brew tap mah3uz/insensical https://github.com/mah3uz/insensical-release
brew install --cask insensical
```

or open `insensical-*-aarch64-apple-darwin.dmg` and drag the application to
Applications. It is not signed with a Developer ID, so macOS refuses to open a copy
that was downloaded until you run:

```sh
xattr -dr com.apple.quarantine /Applications/insensical.app
```

Homebrew does that for you. There is no build for a Mac with an Intel processor.

It needs a Nerd Font for its icons. The default font is JetBrainsMono Nerd Font.

## Checking a download

Each file has a `.sha256` beside it: `sha256sum -c insensical-*.sha256`.
