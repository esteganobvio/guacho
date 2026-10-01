# Guacho Agent Instructions

Fedora OSTree images built with [BlueBuild](https://blue-build.org/). The repo is YAML recipes + shipped config files; there is no application code to test.

## Build commands
- Validate a recipe first: `bluebuild validate recipes/<recipe>.yaml`
- Local build: `bluebuild build recipes/<recipe>.yaml --verbose` (requires docker)
- Rebase this machine onto the local build: `bluebuild switch`
- CI: pushing non-`.md` files triggers `.github/workflows/build.yml`. **The `recipe:` matrix in that file is the only list of built images — commented entries are disabled. Any new recipe must be added to the matrix or it will never build.**
- `image-version` in a recipe becomes the image tags (`:latest`, `:<version>`, `:<date>`, `:<date>-<version>`).

## Structure
- `recipes/<name>.yaml` — one image per file. The `name:` field (e.g. `guacho-sway`) is the ghcr image name: `ghcr.io/esteganobvio/guacho-sway`.
- `recipes/common/<compositor>.yaml` — shared module bundles (dnf packages, os-release) included by every variant of that compositor via `from-file`.
- `files/system/**` — copied into `/` of every image by `common/common.yaml`, so anything under `files/system/etc/...` is global config (greetd lives at `files/system/etc/greetd/config.toml`).
- Base images: `ghcr.io/blue-build/base-images/fedora-base` (and `-nvidia` for GPU variants).
- `modules/` is currently empty; recipes use BlueBuild built-in modules only.
- Conventions: kebab-case filenames, `-nvidia` suffix for GPU variants, keep recipes under ~50 lines, conventional commits.

## Gotchas (verified)
- `skip-unavailable: true` in dnf modules **silently skips packages that are not installed** — a typo'd or unavailable package name will not fail the build. Check the built image to confirm packages actually landed.
- greetd runs `noctalia-greeter-session` with session auto-detection; the picker only lists `.desktop` entries in `/usr/share/wayland-sessions/`. E.g. `sway.desktop` ships only in the `sway-config-upstream` subpackage (plain `sway` pulls `sway-config`, which has no desktop entry), so `common/sway.yaml` must keep `sway-config-upstream`.
- The `greetd` user is created by the Fedora `greetd` package (sysusers); keep `user = "greetd"` in the greetd config.
- Verify a published image: `cosign verify --key cosign.pub ghcr.io/esteganobvio/guacho-sway:latest`. `cosign.key` is gitignored — never commit signing secrets.
- Never add packages that require a reboot to recipes.

## Manual rebase (existing hardware)
See README for the full sequence: rebase to the **unsigned** image (`ostree-unverified-registry:ghcr.io/esteganobvio/<image>:latest`), reboot, rebase to the **signed** image (`ostree-image-signed:docker://ghcr.io/esteganobvio/<image>:latest`), reboot.
