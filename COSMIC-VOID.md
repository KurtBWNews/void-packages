# COSMIC desktop on Void Linux (xbps-src templates)

Working COSMIC (Wayland) session on Void Linux, built from source with xbps-src.
Posted as-is, no promises. Tested on one machine only.

- Tested: 2026-10-07, x86_64 glibc
- Based on void-packages commit c8c61057d26 (mesa 26.2.4)
- Starting point: the COSMIC templates from MtFBella109/void-packages,
  rebased onto current upstream. Credit to that author.

## Build environment and effort

- Hardware: Intel Core i5-4210U laptop, 16 GB RAM, Void Linux x86_64 glibc.
  The build completed successfully on this machine.
- Time: roughly 12-15 hours in total, including working through dependency
  problems, rebasing, and a few dead ends that meant starting over. That is
  total elapsed effort, not a clean compile time; I have not timed a clean
  run of the final branch.
- AI disclosure: I used an AI assistant (Claude) to help sort out dependency
  problems and to explain things I did not understand. I built and ran the
  result myself.

## What's in this branch (cosmic-fresh)

1. COSMIC desktop packages (cosmic-*, xdg-desktop-portal-cosmic) and two meta
   packages: cosmic-desktop-minimal (the one I built and tested) and
   cosmic-desktop-full (included, not built or tested by me)
2. pop-* packages needed by COSMIC: pop-fonts, pop-icons, pop-launcher,
   pop-sounds-theme
3. Template fixes found while building:
   - cosmic-osd: use clang19-devel
   - cosmic-settings-daemon: add openssl-devel to makedepends
   - cosmic-wallpapers: add git to hostmakedepends

## Building

    git clone -b cosmic-fresh https://github.com/KurtBWNews/void-packages.git
    cd void-packages
    ./xbps-src binary-bootstrap
    ./xbps-src pkg cosmic-desktop-minimal
    sudo xbps-install --repository=hostdir/binpkgs cosmic-desktop-minimal

(Rust builds are slow and memory hungry. Expect hours on a laptop.)

Start from a current void-packages checkout. On an older base, libsoup3 failed
to link with `libnghttp2.a ... recompile with -fPIC`; rebasing onto current
upstream master made it go away.

## Running it

Services enabled: dbus, elogind, seatd, polkitd, NetworkManager,
bluetoothd, power-profiles-daemon, greetd.

Also install `bluez` and `power-profiles-daemon`; without them COSMIC
Settings shows "backend not found" messages (see Known issues).

greetd with tuigreet (`sudo xbps-install greetd tuigreet`), configured in
/etc/greetd/config.toml:

    [terminal]
    vt = 7

    [default_session]
    command = "tuigreet --time --remember --remember-session --cmd cosmic-session"
    user = "_greeter"

The cosmic-session package installs /usr/share/wayland-sessions/cosmic.desktop,
which runs /usr/bin/start-cosmic.

## Optional extras

- iPhone file transfer with KDE Connect: `sudo xbps-install kdeconnect` works
  under COSMIC without Plasma. It adds nothing to COSMIC Settings; use
  `kdeconnect-app` and `kdeconnect-settings` from the launcher. Received files
  land in `~/Downloads`.
- Launch order matters when sending from an iOS device: open the KDE Connect
  app on the iOS device first, then launch KDE Connect on the Void machine.
  When everything is configured, the iOS app finds the COSMIC/Void box on its
  own. You may need to tap "Refresh devices" in the iOS app. Both devices must
  be on the same Wi-Fi network. This is the only issue I have noticed so far.

## Known issues

- **Display scaling in COSMIC Settings only works when increasing.** Going
  from 100% down to 90% does nothing. Workaround: pick a lower value first
  (e.g. 75%), then scale up to 90%. Seen on one machine; not checked whether
  it is specific to Void or to COSMIC itself.
- **No time zone selector in Settings > Time & Language.** There is no
  `org.freedesktop.timedate1` service on the system bus without systemd,
  which is the likely cause (not confirmed against COSMIC's source). The
  clock and zone are fine. To change zones:
  `sudo ln -sf /usr/share/zoneinfo/<Region/City> /etc/localtime`
- **"Backend not found" / "Is BlueZ installed?" in Settings** until you
  install and enable `power-profiles-daemon` (Power & battery) and `bluez`
  (Bluetooth). With both services enabled, power profiles and Bluetooth
  (pairing, renaming the controller) worked.
- **Locally built packages aren't rebuilt by system updates.** After
  `xbps-install -Su`, run `ldd` on the COSMIC binaries and rebuild anything
  that reports `not found`. A full update with point-release library bumps
  caused no breakage here.
- **greetd launch path:** logging in through tuigreet with the config above
  starts COSMIC. It can go through `--cmd cosmic-session` or through the
  installed session file (`/usr/bin/start-cosmic`), depending on whether a
  session is picked. I have not tested the two paths separately.
- Build warnings (unused variables in cosmic-panel, dead code in cosmic-osd,
  a future-incompat note for proc-macro-error2) are harmless.

## Licenses

COSMIC is mostly GPL-3.0. Void templates are BSD-2-Clause per void-packages.
