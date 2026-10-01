# Guacho

Guacho is a set of customizable Fedora OSTree images built using [BlueBuild](https://blue-build.org/).

## Available Images

The following images are currently actively built via GitHub Actions:

- `niri`
- `sway`

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

> **Note**
> Images are published as chunked OCI images, so updates between builds only download the chunks that actually changed. The move from the last pre-chunking build to the first chunked build requires a one-time full rebase (~3.7 GB); every update after that is incremental.

## Verification

These images are signed with [Sigstore](https://www.sigstore.dev/)'s [cosign](https://github.com/sigstore/cosign). You can verify the signature by downloading the `cosign.pub` file from this repository and running:

```bash
cosign verify --key cosign.pub ghcr.io/esteganobvio/guacho
```
