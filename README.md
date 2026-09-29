# pod-labwc

The `labwc` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It ships a nested labwc Wayland compositor that hosts the
Selkies streaming desktop.

## What it provides

labwc installs a wlroots-based Wayland compositor plus the `foot` terminal,
`thunar` file manager, and Xwayland for X11 compatibility. The `labwc-wrapper`
connects to pixelflux's `wayland-1` socket and creates its own `wayland-0` client
socket so desktop apps render into the captured desktop; `rc.xml` maximizes every
window and `autostart` wires the session.

| Property | Value |
|---|---|
| Service | `labwc` (`%(ENV_HOME)s/.local/bin/labwc-wrapper`, priority 12, `scope: system`) |
| Requires | `pod-dbus` |
| Packages | `labwc`, `foot`, `xorg-xwayland` (Arch) / `xorg-x11-server-Xwayland` (Fedora), `thunar` |
| Env | `WAYLAND_DISPLAY=wayland-0`, `XDG_RUNTIME_DIR=/tmp` |
| `env_accept` | `XKB_DEFAULT_LAYOUT`, `XKB_DEFAULT_VARIANT`, `XKB_DEFAULT_MODEL`, `XKB_DEFAULT_OPTIONS` |
| `wait_for` | `${XDG_RUNTIME_DIR}/wayland-1`, timeout 30s |

The compositor and selkies input handler both read `XKB_DEFAULT_LAYOUT` from the
environment, so the scancode map matches the compositor's layout. Chrome is NOT
launched here — the supervised `[program:chrome]` service in the shared
`selkies-core` candy owns it for both selkies flavors.

## How to use it

Compose the candy into a selkies desktop box. The `labwc` service waits for
pixelflux's `wayland-1` (a supervised `wait_for:`) before starting.

```bash
charly box validate
```

The candy's own `check:` steps assert the `labwc` binary, the `labwc-wrapper`,
`rc.xml`, and `autostart` files, the `foot` / `thunar` / `Xwayland` binaries, the
`labwc` service RUNNING, the `wayland-0` socket, and a stable RUNNING state ≥20s.

## Layout

- `charly.yml` — the `labwc:` candy entity plus its `skill:` entity.
- `labwc-wrapper` — connects to pixelflux's `wayland-1`, exports the XKB defaults,
  then execs labwc.
- `rc.xml` — server-side decorations, maximize-all window rule, keybindings.
- `autostart` — session wiring (Chrome is supervised elsewhere).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-selkies:labwc` — the nested Wayland compositor for the
  selkies desktop.
- `/charly-selkies:selkies` — the pixelflux streaming engine (provides the
  `wayland-1` labwc connects to).
- `/charly-selkies:selkies-desktop-layer` — the desktop metalayer composing
  labwc + chrome + waybar + selkies.
- `/charly-selkies:waybar-labwc` — the status bar configured for labwc.
- `/charly-check:wl` — Wayland automation (screenshots, input, window management).
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
