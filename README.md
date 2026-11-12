# Isla

Preview builds of the Isla runtime for macOS. Nothing here is released yet: builds are for testing and may be removed.

The current build is the pre-release [`v0.0.0-try`](https://github.com/isla-rs/isla-release/releases/tag/v0.0.0-try), with one tarball per CPU:

| Mac | File |
|---|---|
| Apple Silicon (M1 and later) | `isla-try-macos-aarch64.tar.gz` |
| Intel | `isla-try-macos-x86_64.tar.gz` |

Each holds `isla` (the runtime), `isla-web` (the helper for Chromium web views) and `system/` (Hub and the devtools). Needs macOS 11 or later. The binaries are signed ad hoc, not by a Developer ID, so download them with `curl` as below: a file downloaded by a browser is quarantined, and macOS then refuses to open it.

## Try it without installing

Everything goes into `~/isla-try`, with a data root of its own, so an existing Isla install is untouched.

```sh
arch=$( [ "$(uname -m)" = arm64 ] && echo aarch64 || echo x86_64 )
mkdir -p ~/isla-try && cd ~/isla-try
B=https://github.com/isla-rs/isla-release/releases/download/v0.0.0-try
curl -fsSLO "$B/isla-try-macos-$arch.tar.gz" && curl -fsSLO "$B/SHA256SUMS"
grep "$arch" SHA256SUMS | shasum -a 256 -c
tar -xzf "isla-try-macos-$arch.tar.gz"
./isla --data-root ~/isla-try/root run dev.isla.hub
```

Hub opens in a window. Remove it all with `rm -rf ~/isla-try`.

## Check it

In `~/isla-try`:

| Check | Command | Expected |
|---|---|---|
| The download is intact | `grep "$arch" SHA256SUMS \| shasum -a 256 -c` | `…tar.gz: OK` |
| Not quarantined | `xattr -lr . \| grep quarantine` | nothing |
| The right CPU | `lipo -archs isla` | `arm64` or `x86_64`, as `uname -m` |
| Signed ad hoc | `codesign -dv isla 2>&1 \| grep Signature` | `Signature=adhoc` |
| It runs | `./isla --version` | `isla 0.1.0 (ABI level 0, protocol 5547d1a726cf31ff)` |
| macOS lets it open | `./isla --data-root ~/isla-try/root run dev.isla.hub` | Hub's window, no "can't be opened" alert |
| The keychain is usable | `security add-generic-password -s isla-check -a k -w v && security find-generic-password -s isla-check -w && security delete-generic-password -s isla-check` | `v`, then the item is deleted |

Apps keep their secrets (tokens, keys) in the login keychain. If the last check fails, for example on a Mac whose policy locks the keychain, Isla runs, but apps can't remember their secrets.

## Install it

This puts the runtime where `make install` does and replaces what is there. Skip it to only try the build.

```sh
cd ~/isla-try
mkdir -p ~/.local/lib/isla ~/.local/bin
cp isla ~/.local/lib/isla/isla.new && mv -f ~/.local/lib/isla/isla.new ~/.local/lib/isla/isla
cp isla-web ~/.local/lib/isla/isla-web.new && mv -f ~/.local/lib/isla/isla-web.new ~/.local/lib/isla/isla-web
rm -rf ~/.local/lib/isla/system && cp -R system ~/.local/lib/isla/system
ln -sf ~/.local/lib/isla/isla ~/.local/bin/isla
~/.local/lib/isla/isla setup
```

`isla setup` makes `~/Applications/Isla/Isla.app`, which opens Hub from the Dock, Spotlight and Launchpad. Apps that are running keep their old files until they quit. Put `~/.local/bin` on your `PATH` to type `isla`.

The keychain treats an ad-hoc build as a new program, so the first time each app reads a secret after this install, macOS asks to allow it ("Always Allow").

Uninstall, keeping installed apps and their data:

```sh
~/.local/lib/isla/isla setup --remove
rm -f ~/.local/bin/isla
rm -rf ~/.local/lib/isla
```

## Source

The runtime's source isn't public. This repository holds only its builds.
