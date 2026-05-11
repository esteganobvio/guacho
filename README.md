# Guacho

Guacho is a set of customizable Fedora OSTree images built using [BlueBuild](https://blue-build.org/).

## Available Images

The following images are currently actively built via GitHub Actions:

- `niri`
- `niri-nvidia`

Other planned images include `hyprland`, `swayfx`, `labwc`, and `cosmic`.

## Installation

> **Warning**  
> [This is an experimental feature](https://www.fedoraproject.org/wiki/Changes/OstreeNativeContainerStable), try at your own discretion.

To rebase an existing atomic Fedora installation to the latest Guacho build:

1. Rebase to the unsigned image to install signing keys and policies:
   ```bash
   rpm-ostree rebase ostree-unverified-registry:ghcr.io/esteganobvio/guacho-niri:latest
   ```
2. Reboot:
   ```bash
   systemctl reboot
   ```
3. Rebase to the signed image:
   ```bash
   rpm-ostree rebase ostree-image-signed:docker://ghcr.io/esteganobvio/guacho-niri:latest
   ```
4. Reboot again to complete the installation:
   ```bash
   systemctl reboot
   ```

The `latest` tag always points to the most recent build for the version specified in the recipes.

## Verification

These images are signed with [Sigstore](https://www.sigstore.dev/)'s [cosign](https://github.com/sigstore/cosign). You can verify the signature by downloading the `cosign.pub` file from this repository and running:

```bash
cosign verify --key cosign.pub ghcr.io/esteganobvio/guacho
```
