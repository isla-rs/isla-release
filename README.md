# isla releases

## Mockups try build

A throwaway try build of Mockups (mockup-studio), Apple Silicon only, for testing a curl install without quarantine: [mockup-studio-v0.0.0-try](https://github.com/isla-rs/isla-release/releases/tag/mockup-studio-v0.0.0-try). It carries its own isla runtime (a portable bundle), signed with a local "Isla Dev" identity, not notarized.

### Install

Paste into Terminal on an Apple Silicon Mac:

```sh
cd "$(mktemp -d)"
base=https://github.com/isla-rs/isla-release/releases/download/mockup-studio-v0.0.0-try
curl -fL -O "$base/Mockups-try-macos-aarch64.tar.gz" -O "$base/SHA256SUMS"
shasum -a 256 -c SHA256SUMS
tar -xzf Mockups-try-macos-aarch64.tar.gz -C ~
xattr -lr ~/Mockups.app | grep quarantine || echo "no quarantine"
open ~/Mockups.app
```

The checksum line prints `OK` and the next one `no quarantine`; then the app opens in its window. At the first start its setup installs Node and the agent CLI.

### Data

Its own data root: `~/Library/Application Support/isla/bundles/dev.isla.mockup-studio/`, not shared with an installed isla.

### Remove

```sh
rm -rf ~/Mockups.app "$HOME/Library/Application Support/isla/bundles/dev.isla.mockup-studio"
```
