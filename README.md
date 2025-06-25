Arch Linux and Arch-based distributions currently lack native packaging for Raku/zef modules, forcing users to mix package managers. This creates conflicts with the single package manager philosophy adopted by many distributions. Arch Linux has already solved this for Python by packaging all modules with the `python-` prefix as native Arch packages, avoiding foreign package managers and providing benefits for dependency management, updates, and file tracking (see [PEP 668](https://packaging.python.org/en/latest/specifications/externally-managed-environments/)).

In [Ditana](https://github.com/acrion/ditana) specifically, Sparrow6 and other Raku modules are only available during installation in chroot, but not in the installed system. Alpine Linux has demonstrated a similar approach for Raku by packaging zef modules as native APK packages (see https://pkgs.alpinelinux.org/packages?name=raku-%2A, listing 94 packages of the 991 currently available). This project aims to bring an automated solution to the Arch ecosystem for Raku modules.

## Objective

Create a fully automated system to package all Raku/zef modules as native Arch packages, enabling system-wide availability without mixing package managers.

## Implementation Details

### Package Discovery
- Query all packages by filtering `zef list` output to show unique names:
  ```bash
  zef list | sed 's/:.*$//' | sort -u
  ```
  Currently yields 991 unique packages (implementation will be in Raku)
- Metadata extraction: `zef info -v <package>` provides version, description and direct dependencies

### Build Process
1. **Continuous Monitoring via systemd service**
   - Service runs as root on build server startup
   - Poll for updates using filtered `zef list`, then `zef upgrade` for each package with three possible outcomes:
     - `Can't upgrade identities that aren't installed` → install with `zef install --install-to=site` → generate PKGBUILD and upload
     - `All requested distributions are already at their latest versions` → skip
     - Otherwise (provided exit code 0) → generate PKGBUILD and upload

2. **Package Generation**
   - Dynamic PKGBUILD generation (not version-controlled)
   - Install via `zef install --install-to=site` to ensure consistent paths and metadata
   - Collect installed files for packaging. Note:
     - to be clarified how to find the relevant files
     - installation to a temporary directory instead of `site` may help
     - `zef` doesn’t seem to provide a command analogous to `pacman -Ql`
   - Map Raku package names to Arch package names, following best practices and using prefix `raku-`
   - Run on dedicated build server

## Benefits
- Complete ecosystem coverage (vs. curated subset)
- Zero manual intervention after initial setup
- Raku packages available during first boot for system services
- Maintains single package manager philosophy
- Transparent versioning and updates
