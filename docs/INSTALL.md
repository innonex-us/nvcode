# Install nvcode

Every release in this repository attaches the binaries, installers, checksums,
and install scripts. Release page:
<https://github.com/innonex-us/nvcode/releases>

The examples use `X.Y.Z` for the version you are installing. `nvcode --version`
needs no API key; a key is only required when you call a model.

## Shell installer

`install.sh` and `install.ps1` download the archive for this machine from the
latest release, check it against `SHA256SUMS`, and copy `nvcode` onto `PATH`.

```sh
curl --proto '=https' --tlsv1.2 -fsSL https://github.com/innonex-us/nvcode/releases/latest/download/install.sh | sh
```

Windows PowerShell:

```powershell
irm https://github.com/innonex-us/nvcode/releases/latest/download/install.ps1 | iex
```

| Variable | Effect |
| --- | --- |
| `PREFIX` | Install prefix. `/usr/local` when that `bin` directory is writable, otherwise `~/.local` on Unix, `%LOCALAPPDATA%\nvcode` on Windows |
| `NVCODE_MUSL=1` | Use the x86_64 musl archive (Alpine and other musl systems) |
| `NVCODE_REPO` | `owner/name` to take releases from |

The glibc Linux archives are built on Ubuntu 24.04. On an older glibc, set
`NVCODE_MUSL=1`. There is no aarch64 musl archive.

## Verify a download by hand

```sh
curl -fsSLO https://github.com/innonex-us/nvcode/releases/download/vX.Y.Z/SHA256SUMS
curl -fsSLO https://github.com/innonex-us/nvcode/releases/download/vX.Y.Z/nvcode-aarch64-apple-darwin.tar.gz
shasum -a 256 -c SHA256SUMS --ignore-missing
tar -xzf nvcode-aarch64-apple-darwin.tar.gz
install -m 755 nvcode /usr/local/bin/nvcode   # or ~/.local/bin
```

`SHA256SUMS` is plain `sha256sum` text with one line per asset.

## Homebrew

Each release includes a rendered formula with per-architecture URLs and
checksums filled in at publish time.

```sh
curl -fsSL -o nvcode.rb \
  https://github.com/innonex-us/nvcode/releases/download/vX.Y.Z/nvcode.rb
brew install ./nvcode.rb
```

That covers macOS (Apple Silicon and Intel) and Linux (aarch64 and x86_64).
Homebrew 4 or newer is required for the formula's `on_macos` / `on_linux`
blocks.

## macOS package

| Asset | Machine |
| --- | --- |
| `nvcode-X.Y.Z-aarch64-apple-darwin.pkg` | Apple silicon |
| `nvcode-X.Y.Z-x86_64-apple-darwin.pkg` | Intel |
| `nvcode-X.Y.Z-universal-apple-darwin.pkg` | either |

```sh
curl -fsSL -o nvcode.pkg \
  https://github.com/innonex-us/nvcode/releases/download/vX.Y.Z/nvcode-X.Y.Z-universal-apple-darwin.pkg
sudo installer -pkg nvcode.pkg -target /
nvcode --version
```

The package installs `/usr/local/bin/nvcode`. Packages are unsigned unless the
maintainers have configured Apple signing secrets, in which case the same asset
is signed and notarized. Gatekeeper warns on an unsigned package; installing
with `sudo installer` as above avoids the quarantine prompt.

## Windows

`nvcode-X.Y.Z-x86_64-pc-windows-msvc.msi` installs:

- `C:\Program Files\nvcode\nvcode.exe`
- `C:\Program Files\nvcode\LICENSE`
- `C:\Program Files\nvcode\nvcode.ico`
- the install directory on the system `PATH` (optional feature, on by default)

```powershell
msiexec /i nvcode-X.Y.Z-x86_64-pc-windows-msvc.msi
```

Apps & Features registers the entry, which is also the uninstaller and removes
the `PATH` change. A newer MSI upgrades an older install. Open a new terminal
so the `PATH` change is visible. The MSI is unsigned unless a code-signing
certificate is configured.

## Debian and Ubuntu

| Asset | Architecture |
| --- | --- |
| `nvcode-X.Y.Z-x86_64-unknown-linux-gnu.deb` | amd64 |
| `nvcode-X.Y.Z-aarch64-unknown-linux-gnu.deb` | arm64 |

```sh
curl -fsSL -o nvcode.deb \
  https://github.com/innonex-us/nvcode/releases/download/vX.Y.Z/nvcode-X.Y.Z-x86_64-unknown-linux-gnu.deb
sudo apt install ./nvcode.deb
```

## Fedora, RHEL, and derivatives

| Asset | Architecture |
| --- | --- |
| `nvcode-X.Y.Z-x86_64-unknown-linux-gnu.rpm` | x86_64 |
| `nvcode-X.Y.Z-aarch64-unknown-linux-gnu.rpm` | aarch64 |

```sh
sudo dnf install ./nvcode-X.Y.Z-x86_64-unknown-linux-gnu.rpm
```

The `.deb` and `.rpm` both install `/usr/bin/nvcode`, a
`/usr/share/applications/nvcode.desktop` entry (`Terminal=true`), and hicolor
icons at 16, 32, 48, 64, 128, 256, 512, and 1024 px.

## AppImage

`nvcode-X.Y.Z-x86_64.AppImage` is an x86_64 Linux bundle containing the binary,
`nvcode.desktop`, and the 256 px icon.

```sh
curl -fsSL -o nvcode.AppImage \
  https://github.com/innonex-us/nvcode/releases/download/vX.Y.Z/nvcode-X.Y.Z-x86_64.AppImage
chmod +x nvcode.AppImage
./nvcode.AppImage --version
```

## Update

```sh
nvcode update --check   # report the latest version
nvcode update           # verify, replace, restart
```

`/update` does the same from inside a session. The updater takes the archive
for this OS and architecture, checks it against `SHA256SUMS` when the release
has that file, replaces the running binary, prints the release notes, and
starts the new binary. If the install directory is not writable it prints
`sudo nvcode update` (Unix) or tells you to rerun as Administrator (Windows)
instead of installing partially. Sessions also check releases in the background
on startup, cache the result for six hours, and skip the check when stdin is
not a terminal or `NVCODE_NO_UPDATE_CHECK` is set.

## Uninstall

| Installed with | Remove |
| --- | --- |
| `install.sh` / `install.ps1` | delete `$PREFIX/bin/nvcode` (`.exe` on Windows) |
| Homebrew | `brew uninstall nvcode` |
| macOS `.pkg` | `sudo rm /usr/local/bin/nvcode` |
| Windows `.msi` | Apps & Features → nvcode → Uninstall |
| `.deb` | `sudo apt remove nvcode` |
| `.rpm` | `sudo dnf remove nvcode` |
| AppImage | delete the `.AppImage` file |

Configuration and state live in `~/.config/nvcode` (`config.toml`, `mcp.json`,
`history.json`, `sessions/`, `plugins/`) and `~/.cache/nvcode`
(`model-probe.json`, `tools/`, Composer and pnpm stores). Per-project state
lives in `.nvcode/` next to the project. Delete what you do not want to keep.

## Troubleshooting

- **`command not found` after installing.** Check `echo $PATH`; the default
  `~/.local/bin` needs to be on it, and Windows needs a new terminal after an
  MSI install.
- **`GLIBC_x.y not found`.** Use the musl archive: `NVCODE_MUSL=1` with
  `install.sh`, or download `nvcode-x86_64-unknown-linux-musl.tar.gz`.
- **Checksum mismatch.** Re-download. If it persists, open an issue here with
  the asset name and the two hashes; do not install the file.
- **`nvcode: command not found` inside `run_shell` tasks.** `run_shell` always
  starts `sh -lc`, so Windows needs a `sh` on `PATH` (Git Bash is enough). File
  tools do not need it.
- **Keys.** `nvcode login` rewrites `~/.config/nvcode/config.toml` and drops
  comments; keep any hand-written notes elsewhere. A project
  `.nvcode/config.toml` cannot set the API key.
