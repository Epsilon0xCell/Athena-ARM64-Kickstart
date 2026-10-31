# GitHub Actions Workflows

This repository includes automated workflows to build and validate the Athena OS ARM64 ISO.

This repo contains exactly two workflow files. Earlier versions of this
page described two further workflows that are not in this repo, and
described the build as a native macOS Podman build. Both of those
descriptions were wrong and are corrected below. No workflow was run
live in the pass that made this correction; the statements below are
from reading the two workflow files.

## Workflows

### 1. Build and Release ARM64 ISO (`build-arm64-iso.yml`)

**Purpose**: Builds the complete Athena OS ARM64 ISO and creates GitHub releases.

**Triggers**:
- Manual trigger via GitHub Actions UI, with `release_tag` and `fedora_version` inputs (defaults: `v1.0.0-arm64` and `42`)
- Git tags matching `v*-arm64` or `v*-aarch64`

**How it actually builds**:
- Runs on `ubuntu-latest`, not a macOS runner
- Installs `qemu-user-static` and uses Docker to run the Fedora build container as linux/arm64, so the ARM64 side comes from the container platform and QEMU, not from runner hardware
- Automatic checksum generation (SHA256 and SHA512)
- GitHub release creation with detailed release notes
- Build artifacts with 30-day retention

### 2. Simple Validation (`simple-validation.yml`)

**Purpose**: Lightweight validation without external dependencies.

**Triggers**:
- Manual trigger
- Push to the main or master branch touching `athena-iso.ks`
- Pull requests touching `athena-iso.ks`

**Checks**:
- Pure shell and grep checks (no Python dependencies, and no kickstart parser, so these checks prove the ARM64 pieces are present in the file, not that the file parses)
- Required sections (`%packages`, `%post`, `%end`) present
- ARM64 GRUB packages (`grub2-efi-aa64`) present
- EFI partition (`/boot/efi`) present
- x86-specific package strings (`syslinux`, `virtualbox-guest-additions`, `xorg-x11-drv-vmware`) absent
- Package analysis and counting
- Repository configuration validation

## Usage

### Building a Release

1. **Tag-based release** (recommended):
   ```bash
   git tag v1.0.0-arm64
   git push origin v1.0.0-arm64
   ```

2. **Manual trigger**:
   - Go to Actions → "Build and Release Athena OS ARM64 ISO"
   - Click "Run workflow"
   - Enter release tag and Fedora version
   - Click "Run workflow"

### Validation Only

The validation workflow runs automatically on pushes touching the
kickstart file, or can be triggered manually for quick checks.

## Build Requirements

The workflows are designed to be self-contained and don't require special repository configuration beyond:

- GitHub Actions enabled
- `GITHUB_TOKEN` (automatically provided)

## Build Output

### Successful Build Produces:
- `athenaos-live-aarch64.iso` - Main ISO file
- `athenaos-live-aarch64.iso.sha256` - SHA256 checksum
- `athenaos-live-aarch64.iso.sha512` - SHA512 checksum

### Release Notes Include:
- Build information and compatibility matrix
- Installation instructions
- File checksums
- ARM64-specific changes and requirements

No build times are stated here: the earlier native-runner timing claims
were tied to the wrong runner description and were not verified.

## Troubleshooting

### Common Issues:

1. **Build fails with package not found**:
   - Check if ARM64 packages are available in Fedora repositories
   - Verify kickstart syntax with validation workflow

2. **ISO too large**:
   - Adjust package list in `athena-iso.ks`
   - Consider removing unnecessary packages

3. **GRUB/boot issues**:
   - Ensure EFI partition is properly configured
   - Verify ARM64 GRUB packages are included

### Debug Build Issues:

1. Check workflow logs in GitHub Actions
2. Run validation workflow first to catch configuration errors
3. Use manual trigger with debug mode if needed

## ARM64 Compatibility

The built ISO supports:
- Apple Silicon Macs (M1/M2/M3/M4)
- ARM64 servers and workstations
- ARM64 development boards with UEFI
- Not x86/x64 systems (use standard Athena OS)

## Contributing

To modify the build process:

1. Test changes with validation workflow first
2. Update kickstart file (`athena-iso.ks`) for package changes
3. Modify workflows for build process changes
4. Document significant changes in `ARM64_CHANGES.md`
