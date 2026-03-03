# Agentic Development Guidelines for Guacho

This repository hosts BlueBuild-based atomic Fedora OSTree images. Follow these guidelines when making changes.

## Build & Test

### Build Commands

```bash
# Full build via GitHub Actions (push to recipe files triggers build)
git commit -m "update: swayfx recipe"
git push

# Manual trigger via GitHub UI
# Go to Actions → bluebuild → Run workflow

# Local BlueBuild setup (for development)
# https://blue-build.org/how-to/setup/
# 1. Clone bluebuild repository
# 2. Copy your recipe and module files to /var/home/<user>/bluebuild/
# 3. Run: bash bluebuild.sh --recipe swayfx

# Verify image signature
cosign verify --key cosign.pub ghcr.io/esteganobvio/guacho
```

### Testing

```bash
# BlueBuild will test on:
# - Daily scheduled builds (06:00 UTC)
# - Push events to recipe files
# - Pull request validation

# For local testing, use BlueBuild interactive mode:
# bash bluebuild.sh --recipe swayfx --verbose
```

## Code Style & Formatting

### YAML Recipes (recipes/*.yaml)

- Use YAML anchors for shared configurations
- Keep recipes under 50 lines
- Use kebab-case for module names
- Indent consistently at 2 spaces
- Comment changes briefly

```yaml
---
modules:
  - type: dnf
    install:
      packages:
        - niri
  - from-file: common/dms.yaml
```

### Shell Scripts (files/scripts/, .bluebuild-scripts_f82e9742/)

- Use `#!/usr/bin/env bash` shebang
- Enable strict mode: `set -euo pipefail`
- Quote all variables
- Use `local` for function parameters
- Return appropriate exit codes (0 for success, 1 for failure)

```bash
#!/usr/bin/env bash

set -euo pipefail

main() {
    local output="success"
    if [[ ! -f "$CONFIG_FILE" ]]; then
        output="not_found"
    fi
    echo "$output"
}

main
```

### Module Scripts (NuShell/SH)

- Preferred format: `.nu` files (Nushell)
- Accept parameters as positional arguments
- Use `get_script_path` helper for script discovery
- Return 0 on success, 1 on failure

## Error Handling

### BlueBuild Scripts

- Always check for file existence before processing
- Use `set -euo pipefail` for strict error handling
- Print informative status messages at module start/end
- Exit with code 1 on failure

### Nushell/Shell Modules

- Validate inputs at module start
- Use `try-catch` for command failures
- Print clear error messages

## Import Organization

### Shell Scripts

```bash
#!/usr/bin/env bash

set -euo pipefail

# Source exported variables and functions
source /tmp/scripts/exports.sh

# Use functions from here

# Local functions
print_banner() {
    # implementation
}

main "$@"
```

## Naming Conventions

### Recipes
- Recipe name describes environment: `swayfx.yaml`, `niri.yaml`, `cosmic.yaml`
- Add `-nvidia` suffix for GPU variants
- Keep names lowercase with hyphens

### Modules
- Lowercase with hyphens: `systemd-user-sessions.nu`
- Group related functionality in common/

### Files
- Use kebab-case
- Group related scripts in files/ subdirectories

## Commit Guidelines

- Follow conventional commits: `feat:`, `fix:`, `chore:`, `docs:`, `test:`
- Reference issues when applicable
- Keep commit messages concise (< 72 chars)

## Testing Checklist

Before pushing changes:

- [ ] YAML syntax is valid
- [ ] No new packages that require reboot
- [ ] Module scripts return appropriate exit codes
- [ ] Error messages are user-friendly
- [ ] No breaking changes to existing functionality

## Secrets

- Never commit secrets or credentials
- Use GitHub Actions secrets for `SIGNING_SECRET` and other credentials
- Never log secrets

## Repository Structure

```
.
├── recipes/           # Environment definitions
│   └── common/        # Shared module configurations
├── modules/           # Build modules (scripts)
├── files/             # System configuration
│   ├── scripts/       # Setup and cleanup scripts
│   ├── systemd/       # Systemd units
│   └── system/        # Other system files
├── .bluebuild-scripts_f82e9742/  # BlueBuild execution scripts
├── .github/workflows/ # CI/CD pipelines
└── cosign.pub         # Signature verification key
```

## Integration Points

### BlueBuild Action
- Uses `blue-build/github-action@v1.11`
- Supports matrix builds for multiple recipes
- Automatically runs on daily schedule, PRs, and pushes
- Requires `SIGNING_SECRET` for cosign verification

### OSTree Rebase
```bash
# First rebase to unsigned (get keys/policies)
rpm-ostree rebase ostree-unverified-registry:ghcr.io/<user>/guacho-sway:latest
systemctl reboot
# Then rebase to signed
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/<user>/guacho-sway:latest
systemctl reboot
```