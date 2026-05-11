# Guacho Agent Instructions

## 🛠️ Core Workflow & Build
*   **Primary Focus**: Updating `recipes/*.yaml` and `files/` system configurations for BlueBuild (Fedora OSTree images).
*   **Build Trigger**: Changes to `recipes/*.yaml` trigger GitHub Actions.
*   **Local Dev**: Use `bash bluebuild.sh --recipe <recipe_name> --verbose` (local setup required).
*   **Image Verification**: `cosign verify --key cosign.pub ghcr.io/esteganobvio/guacho`
*   **Manual Rebase**: Follow this strict 4-step sequence:
    1.  `rpm-ostree rebase ostree-unverified-registry:ghcr.io/esteganobvio/guacho-sway:latest`
    2.  `systemctl reboot`
    3.  `rpm-ostree rebase ostree-image-signed:docker://ghcr.io/esteganobvio/guacho-sway:latest`
    4.  `systemctl reboot`

## 📁 Structure & Conventions
*   `recipes/*.yaml`: Environment definitions (Keep < 50 lines; use YAML anchors).
*   `modules/`: Build modules (scripts).
*   `files/`: System configurations (e.g., `files/system/usr/...`).
*   **Naming**: Use kebab-case for files/modules. GPU variants use `-nvidia` suffix (e.g., `swayfx-nvidia.yaml`).
*   **Shell Scripts**: Use `#!/usr/bin/env bash` with `set -euo pipefail`. Always quote variables.
*   **Scripts**: Nushell (`.nu`) is the preferred scripting language for modules.
*   **YAML**: Use kebab-case for module names; 2-space indentation.
*   **Commits**: Use conventional commits (`feat:`, `fix:`, `chore:`, etc.).

## ⚠️ Critical Constraints
*   **No Rebooting Packages**: Never add packages in recipes that require a reboot.
*   **Error Handling**: Always check for file existence before processing in scripts.
*   **Secrets**: Never commit secrets or credentials.
