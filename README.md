# Relay — desktop builds

Public home of the Relay desktop installers. The application source is private;
this repository carries release artifacts only.

Relay is a local-first API client: HTTP/2, GraphQL, WebSocket, SSE, gRPC and MQTT
in one workspace, collections stored on disk as human-readable `.request.yaml`
files, with optional team sync against a self-hosted Relay cloud.

- **Latest build** — https://github.com/tamiz-uddin/relay/releases/latest
- **Download page** — https://relay.tamiz.pro/downloads
- **Cloud / web app** — https://relay.tamiz.pro

## Downloads

| Platform | File | Install |
|---|---|---|
| Debian / Ubuntu | `Relay_<version>_amd64.deb` | `sudo apt install ./Relay_<version>_amd64.deb` |
| Fedora / RHEL | `Relay-<version>-1.x86_64.rpm` | `sudo dnf install ./Relay-<version>-1.x86_64.rpm` |
| Any Linux | `Relay_<version>_amd64.AppImage` | `chmod +x` then run it |
| Windows x64 | `Relay_<version>_x64-setup.exe` | run the installer |
| macOS | `Relay_<version>_*.dmg` | open the image, drag Relay to *Applications* |

## Installation

### One command (any Linux, no root)

```bash
curl -fsSL https://relay.tamiz.pro/install.sh | bash
```

The script resolves the newest release, downloads the AppImage for your
architecture, puts it in `~/.local/bin/relay.AppImage`, adds an icon and a menu
entry, and prints how to start it. Run it again to upgrade.

| Flag | Effect |
|---|---|
| `--system` | install into `/usr/local/bin` + `/usr/share` (needs sudo) |
| `--dir <path>` | install the AppImage somewhere else |
| `--package` | use the native `.deb`/`.rpm` through sudo instead |
| `--dry-run` | print every action, change nothing |
| `--uninstall` | remove the binary, icon and menu entry |

Manual installs are documented below.

### Debian / Ubuntu

```bash
wget https://github.com/tamiz-uddin/relay/releases/latest/download/Relay_0.1.0_amd64.deb
sudo apt install ./Relay_0.1.0_amd64.deb
relay-desktop        # or launch "Relay" from the application grid
```

The package declares `libwebkit2gtk-4.1-0` and `libgtk-3-0`, so `apt` pulls the
runtime it needs (Ubuntu 22.04+, Debian 12+).

### Fedora / RHEL

```bash
sudo dnf install https://github.com/tamiz-uddin/relay/releases/latest/download/Relay-0.1.0-1.x86_64.rpm
```

### Any Linux — portable AppImage

```bash
wget https://github.com/tamiz-uddin/relay/releases/latest/download/Relay_0.1.0_amd64.AppImage
chmod +x Relay_0.1.0_amd64.AppImage
./Relay_0.1.0_amd64.AppImage
```

If it stops with a FUSE error, install `libfuse2` (Debian/Ubuntu) or `fuse2`
(Fedora), or run it unpacked:

```bash
./Relay_0.1.0_amd64.AppImage --appimage-extract-and-run
```

#### Install the AppImage permanently

An AppImage is a single file, so "installing" means putting it on `PATH` and
adding a menu entry. Run this once:

```bash
mkdir -p ~/.local/bin ~/.local/share/applications \
         ~/.local/share/icons/hicolor/256x256/apps

install -m 755 Relay_0.1.0_amd64.AppImage ~/.local/bin/relay.AppImage

# icon straight out of the image
cd "$(mktemp -d)" && ~/.local/bin/relay.AppImage --appimage-extract '*.png' >/dev/null
install -m 644 squashfs-root/Relay.png \
  ~/.local/share/icons/hicolor/256x256/apps/relay.png

cat > ~/.local/share/applications/relay.desktop <<EOF
[Desktop Entry]
Type=Application
Name=Relay
Comment=Local-first API client
Exec=$HOME/.local/bin/relay.AppImage %U
Icon=relay
Terminal=false
Categories=Development;
StartupWMClass=Relay
EOF

update-desktop-database ~/.local/share/applications
```

After that, start it from the application grid (search *Relay*) or type
`relay.AppImage` in a terminal. To upgrade, download the new release and repeat
the `install -m 755` line; the menu entry keeps working.

Uninstall:

```bash
rm ~/.local/bin/relay.AppImage \
   ~/.local/share/applications/relay.desktop \
   ~/.local/share/icons/hicolor/256x256/apps/relay.png
```

`libfuse2` is only needed to *run* it — the extraction step above does not need
FUSE.

### Windows x64

1. Run `Relay_<version>_x64-setup.exe`.
2. SmartScreen shows *"Windows protected your PC"* because the installer is not
   code-signed: **More info → Run anyway**.
3. It installs per user (`%LOCALAPPDATA%\Relay`), so no administrator prompt.

### macOS

1. Open the `.dmg` and drag **Relay** into *Applications*.
2. The build is not notarised, so Gatekeeper blocks the first launch:
   right-click the app → **Open** → **Open**, or clear the quarantine flag:

```bash
xattr -dr com.apple.quarantine /Applications/Relay.app
```

Apple Silicon (`aarch64`) and Intel (`x64`) images are separate — take the one
that matches your Mac.

## First launch and local data

| Platform | Data directory |
|---|---|
| Linux | `~/.local/share/com.relay.app` |
| macOS | `~/Library/Application Support/com.relay.app` |
| Windows | `%APPDATA%\com.relay.app` |

0.1.0 changed the bundle identifier from `com.apix.app` to `com.relay.app`. On
first launch Relay copies the previous data directory across and renames the
database and its WAL sidecars, so existing collections, environments and
settings carry over. The older build keeps working, so both can stay installed
until the new one is verified.

## Uninstall

| Platform | Command |
|---|---|
| Debian / Ubuntu | `sudo apt remove relay` |
| Fedora / RHEL | `sudo dnf remove Relay` |
| AppImage | delete the `.AppImage` file |
| Windows | Settings → Apps → Relay → Uninstall |
| macOS | drag Relay from Applications to the Trash |

Delete the data directory from the table above only if you also want to remove
your collections.

## Build notes

- These artifacts are unsigned: no Authenticode certificate on Windows, no
  notarisation on macOS. There is no checksum file or signature to verify yet.
- macOS `.dmg` files can only be produced on a macOS host, so they land a little
  after the Linux and Windows artifacts of the same tag.
- Installer names encode the version: `Relay_<version>_amd64.deb`,
  `Relay-<version>-1.x86_64.rpm`, `Relay_<version>_amd64.AppImage`,
  `Relay_<version>_x64-setup.exe`.
