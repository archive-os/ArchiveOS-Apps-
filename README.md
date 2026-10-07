# ArchiveOS Applications

This bundle contains eight DEB packages for ArchiveOS / KDE Plasma:

| Package | Version | Purpose |
| --- | --- | --- |
| aos-welcome | 1.0.8 | Welcome, system settings, maintenance and application shortcuts |
| aos-app-remover | 2.2.0 | Select and remove user-installed applications |
| aos-deb-installer | 2.2.0 | Install local DEB packages |
| aos-update-manager | 2.2.0 | Package updates and scheduled update notifications |
| aos-windows-app-support | 2.2.0 | Wine, UMU / GE-Proton and Windows application controls |
| aosver | 2.2.0 | ArchiveOS version information |
| aos-gaming-center | 1.0.8 | Game library, launchers and compatibility settings |
| aos-touch-keyboard | 1.1.2 | Manually opened on-screen keyboard |

Applications follow the ArchiveOS light/dark appearance. Their interface uses
Turkish for Turkish locales and English for other locales. After an upgrade,
close and reopen running ArchiveOS applications.

## Installation

Open a terminal in this folder and run:

```bash
sudo ./install.sh
```

The installer verifies embedded DEB checksums, installs all eight packages and
dependencies without allowing package removal, and enables the update checker
for all desktop users. On a running installed system it starts the checker in
active user sessions. Without an active session, it starts at the next login.
A startup failure is reported even if package installation has completed.

For ISO integration, copy the bundle into the target image and run `./install.sh`
as root **inside the target chroot**. Running it on the build host installs the
applications on that host. In a chroot or live session the checker is not started
immediately. The installer does not download games or Windows runtimes.

Manual package installation, without the installer's service setup:

```bash
sudo apt install ./*.deb
```

Check package integrity independently with `sha256sum --check SHA256SUMS`.
`previous-packages/` contains older builds; use the current DEBs in this folder.
`services/` contains copies of service/startup definitions installed by the DEB.

## Welcome and update notifications

Open `aos-welcome` to access ArchiveOS applications, appearance, package sources,
updates and maintenance. The navigation sidebar is 340 pixels wide. Welcome has a per-user automatic-start checkbox;
live sessions do not automatically launch it. Toolkit cards use each app's logo.

In Update Manager, choose weekly or monthly checks. The checker displays
notifications and opens Update Manager when requested; it does not install
updates automatically. From an installed desktop session, use these commands
**without sudo** if manual service setup or diagnosis is needed:

```bash
systemctl --user unmask aos-update-checker.service
systemctl --user daemon-reload
systemctl --user enable --now aos-update-checker.service
systemctl --user status aos-update-checker.service --no-pager
journalctl --user -u aos-update-checker.service -n 100 --no-pager
```

Use `systemctl --user stop aos-update-checker.service` to stop the current checker,
or `systemctl --user restart aos-update-checker.service` to restart it.

## Gaming Center

Launch `aos-gaming-center` or open it through Welcome / ArchiveOS Apps.

### Library and launching

- Steam/Heroic installation records and available Games menu entries are scanned
  on launch and once per minute while idle; **Scan games** refreshes manually.
- **Game Tools** separates stores, compatibility tools and emulators from games.
  Recognized Minecraft launchers, including TLauncher and Prism, count as games.
- **Add game** offers an application checklist, AppImage, .sh, Windows EXE and
  Linux executable files, including extensionless executables. The editor can
  grant owner execute permission to the selected Linux file.
- Unchecking an application removes its library record and prevents automatic
  re-addition. It does not uninstall it. Checking it again restores the entry.
- Steam imports include external libraries and Flatpak installations. Heroic
  imports include Epic/Legendary, GOG, Amazon and generated/sideload shortcuts.
  Imports link existing games without downloading, moving or deleting game data.
- Set arguments, working directory, cover, GPU, Windows runtime, Proton version,
  Wine prefix and optional UMU game ID per game. Reimports preserve customization.
- AppImages may be copied into the managed library while leaving the original
  unchanged. For multi-file games, retain the original working directory;
  copying a launcher alone does not copy its assets or DLLs.

Steam/Heroic games use matching .desktop launchers where available, with store
URI fallbacks. Native executables run directly with separate arguments; .sh
files use `/bin/bash`. Removing an entry does not delete its game. Closing
Gaming Center does not kill launched games.

### Artwork and console view

**Assign logo** stores a private copy of a custom image. **Automatic logo** clears
the custom choice and restores discovery. Local Steam/Heroic artwork is used;
manually added games can also search Steam, itch.io and Game Jolt by display name.
**Find logo online** retries a lookup. Custom images take priority. Ambiguous or
failed matches retain the local icon; assigning your own image resolves mistakes.
Only the game display name is sent for these searches. Network failures do not
block launching games. Preview images form the blurred console background.

The first launch is windowed. Enable **Game console view** for full-screen cards.
**Always keep console mode on** persists that choice across launches; untick it
to return to window mode. Arrow keys navigate, Enter launches, F11 toggles the
temporary view, and Esc leaves it when the persistent checkbox is off. Console
settings and Game Tools share the same library and controls as the window view.

### Stores, Windows compatibility and performance

Steam, Heroic and ProtonUp-Qt can be installed as user Flatpaks from Flathub;
existing native launchers also work. UMU is installed per user from its official
release with SHA256 verification. GE-Proton setup through UMU may also download
the Steam Runtime. ProtonUp-Qt manages additional compatibility builds. Wine /
Winetricks installation uses an authenticated helper and includes wine32 on amd64.
Windows App Support provides persistent runtime selection and a Gaming Center
shortcut. Downloads and system changes occur through the chosen app controls.

Per-game optimization uses GameMode when available. **Game optimization →
GameMode → Install / update** configures performance governor, nice -5 and
best-effort I/O priority, backs up existing `/etc/gamemode.ini`, and adds the user
to the gamemode group. Log out/in to activate group membership. User or game-local
GameMode configuration may override system settings.

Standalone games may use a systemd user scope with CPUWeight=1000 and
IOWeight=500. Steam/Heroic supervision requests GameMode for detected game
processes rather than keeping the store client optimized. Registration errors
appear in the log and do not block launching. If necessary, enable GameMode in
Heroic or use Steam launch options `gamemoderun %command%`.

Optimization does not stop background services. GPU offload is selectable for
standalone games; drivers, hardware, runtime and anti-cheat determine actual
compatibility and performance. Universal Windows compatibility or performance
gains are not guaranteed.

Library, managed launch files, runtimes, prefixes and logs are stored under
`${XDG_DATA_HOME:-~/.local/share}/aos-gaming-center/`.

## On-screen keyboard

Open Welcome → On-screen Keyboard or ArchiveOS Apps → Touch Keyboard. Select a
layout, then press **Show keyboard**; **Close keyboard** closes it. Layout search
is available. Desktop auto-start, automatic text-focus opening and tablet
recognition are disabled. The physical keyboard layout is unchanged.

- **X11:** Onboard uses XTest, a tray icon and bottom docking. Installed XKB layouts
  and variants are converted to Unicode/dead-key layouts with Shift/AltGr levels.
  The build system had 590 variants, plus the original Turkish/English Q layouts.
- **Wayland:** KWin launches Maliit over its private input-method connection.
  Installed Maliit languages are listed (42 on the build system). Unsupported
  languages produce an error. This catalog differs from XKB; a layout does not
  provide a complete IME, prediction or handwriting engine.
- **SDDM:** Use the login screen's own Virtual Keyboard button. The package adds
  `InputMethod=qtvirtualkeyboard` and requires Qt's virtual-keyboard module and
  input-context plugin. The existing login theme is preserved.

The working Wayland method is included in 1.1.2: the Plasma environment script
sets `KWIN_IM_SHOW_ALWAYS=1`, and the Show button uses KWin `forceActivate` and
checks actual panel visibility. Log out/in once after installation to load this
permission into KWin. It does not automatically start the keyboard; the app
keeps the next login disabled after manual activation.

For upgrades from the previous automatic keyboard, run once in each existing
user's desktop session:

```bash
aos-touch-keyboard --disable-auto
```

The bundle installer runs this for its invoking active user outside chroot.
Preferences/backups are under `~/.config/archiveos`. The old `--session` entry
point is a no-op and its autostart entry is hidden. SDDM changes take effect at
the next login/reboot; do not restart it during active work.

## Verification and remaining checks

Gaming Center checks covered library persistence, import/launch plans, runtime
and GameMode guards, artwork, discovery/exclusions, console controls, TR/EN
rendering and a harmless native executable fixture. Actual store installation,
Windows game compatibility and CPU/GPU improvements still require system tests.

Keyboard checks: 11 automated tests passed; isolated Onboard/XWayland, a separate
KWin compositor's manual show/hide flow, and a Qt virtual-keyboard panel were
verified. On 8 October 2026 the user reported that the keyboard worked. The
installed application and delivered DEB contain the same working method.
Native X11 login, real SDDM login and complete text delivery across applications
remain separate checks. Bottom docking does not guarantee that every free or
full-screen window avoids overlap.

## Upstream references

- [GameMode](https://github.com/FeralInteractive/gamemode)
- [UMU](https://github.com/Open-Wine-Components/umu-launcher)
- [ProtonUp-Qt](https://github.com/DavidoTek/ProtonUp-Qt)
- [KWin manual keyboard activation](https://github.com/KDE/kwin/blob/Plasma/6.4/src/virtualkeyboard_dbus.cpp)
- [Qt Virtual Keyboard](https://github.com/qt/qtvirtualkeyboard)
