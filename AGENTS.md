# AGENTS.md — pod-labwc

Standalone candy repo for the `labwc` candy — a nested wlroots-based Wayland
compositor that hosts the Selkies streaming desktop. The candy lives in
`charly.yml` at the repo root plus its compositor configuration files.

Canonical files:

- `charly.yml` — the `labwc:` candy entity (description, `require`, `env`,
  `env_accept`, `distro`, `service`, `plan`) and its `skill:` entity.
- `labwc-wrapper` — connects to pixelflux's `wayland-1`, exports the XKB
  defaults, then execs labwc.
- `rc.xml` / `autostart` — the compositor config and session wiring.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-selkies:labwc` — the owning skill: the nested compositor
  architecture (pixelflux → labwc → apps), the XKB env contract, and the Chrome
  ownership split. Load before editing, building, deploying, or troubleshooting
  this candy.
- `/charly-selkies:selkies` — the pixelflux streaming engine that provides the
  `wayland-1` this compositor nests into.
- `/charly-selkies:selkies-core` — the shared selkies transport and the
  supervised `[program:chrome]` service that owns Chrome for both flavors.
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; tree-position nesting, services).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs, the
  `wl:` verb, and `charly check run <bed>`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs, service declarations).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The live R10 witness is the `check-selkies-labwc-pod` bed; the candy's own
  `check:` steps assert the `labwc` binary, the `labwc-wrapper` / `rc.xml` /
  `autostart` files, the `foot` / `thunar` / `Xwayland` binaries, the `labwc`
  service RUNNING, the `wayland-0` socket, and a stable RUNNING state ≥20s.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `labwc:` candy entity in `charly.yml` and the wrapper/`rc.xml` /
  `autostart` files together. The `skill:` entity in the same file is the owning
  skill's source — a candy change and its skill change land together.
- The wait for pixelflux's `wayland-1` is the service's supervised `wait_for:`,
  NOT a hand-rolled loop in the wrapper (a supervised timeout fails the service
  instead of looking like a labwc crash).
- Chrome stays supervised by the shared `selkies-core` candy; do not launch it
  from `autostart`.
- The `WAYLAND_DISPLAY` / `XDG_RUNTIME_DIR` env and the XKB `env_accept` entries
  are the input contract; keep them in step with the wrapper's exports.
- The `skill:` entity is the source for `/charly-selkies:labwc`; never edit the
  generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
