# pkg-cli

> Standalone Package Manager for pkgs Ecosystem

**Version:** 0.1.0
**Status:** Phase 2 - Bootstrap Tool
**Location:** `~/.pkg-cli/`

---

## Overview

`pkg-cli` is a standalone bootstrap tool for managing packages in the pkgs ecosystem. It uses GNU Stow for symlink-based installations and supports automatic dependency resolution, hooks, and system health checks.

**Key Feature:** This tool is NOT managed by itself (it's standalone) - it exists outside the package ecosystem to avoid circular dependencies.

## Installation

### Quick Setup

```bash
# Clone this repository
git clone https://github.com/andronics/pkg-cli.git ~/.pkg-cli

# Add to PATH (add to ~/.zshrc or ~/.bashrc)
export PATH="$HOME/.pkg-cli/bin:$PATH"

# Verify installation
pkg-cli --help

# Run health check
pkg-cli doctor
```

### Requirements

- **GNU Stow**: Package management via symlinks
  ```bash
  # Arch Linux
  sudo pacman -S stow

  # Ubuntu/Debian
  sudo apt install stow

  # macOS
  brew install stow
  ```

- **Git**: For package updates (optional)
- **GitHub CLI (`gh`)**: For GitHub integration (post-Phase 4)

## Usage

### Basic Commands

```bash
# List available packages
pkg-cli list

# Show package information
pkg-cli info @core/lib

# Install package (with auto-dependency resolution)
pkg-cli install @core/lib

# Install multiple packages
pkg-cli install @system/audio @apps/player

# Uninstall package
pkg-cli uninstall @apps/player

# Reinstall package
pkg-cli reinstall @core/lib

# Run system health checks
pkg-cli doctor
```

### Options

```bash
-n, --dry-run       # Preview changes without executing
-v, --verbose       # Show detailed output
-s, --skip-hooks    # Skip pre/post install hooks
--no-validate       # Skip post-install validation
--source <path>     # Use different package source (default: ~/.pkgs)
--target <path>     # Use different install target (default: $HOME)
```

### Examples

```bash
# Dry-run install to see what would happen
pkg-cli -n install @core/lib

# Verbose installation
pkg-cli -v install @system/audio

# Install all packages (use with caution!)
pkg-cli install '*'

# Use custom source directory
pkg-cli --source /path/to/packages list

# Environment variable override
PKGS_SOURCE=/tmp/test pkg-cli list
```

## Features

### 1. Automatic Dependency Resolution

Packages can declare dependencies in `package.conf`:

```bash
# package.conf
depends=("@core/lib" "@system/audio")
```

When you install a package, `pkg-cli` automatically:
1. Reads the `depends=()` array
2. Installs missing dependencies first (recursively)
3. Then installs the requested package

### 2. System Health Checks

The `doctor` command performs comprehensive system checks:

```bash
pkg-cli doctor
```

Checks:
- ✓ GNU stow installation
- ✓ Package source directory existence
- ✓ package.conf file validity
- ✓ Broken symlinks detection
- ✓ Required commands availability
- ✓ lib package structure (Phase 1 completion)
- ✓ PATH configuration

### 3. Pre/Post Hooks

Packages can define hooks in `package.conf`:

```bash
preinstall() {
  echo "Running before installation"
}

postinstall() {
  echo "Running after installation"
}

preuninstall() {
  echo "Running before uninstallation"
}

postuninstall() {
  echo "Running after uninstallation"
}
```

Skip hooks with `--skip-hooks` flag.

### 4. Custom Ignore Patterns

Per-package ignore patterns using Perl regex:

```bash
# package.conf
ignore=("README.md" "docs" ".*\.backup" "screenshots")
```

`package.conf` and `.git*` are always automatically ignored.

### 5. GitHub Integration (Stubs)

**Status:** Implemented as stubs in Phase 2, full implementation post-Phase 4

**Note:** Production implementation will use standard `git` commands, NOT `gh` CLI, for better portability.

```bash
# These commands show TODO messages currently
pkg-cli clone @core/lib      # Will use: git clone https://github.com/andronics/pkg-core-lib
pkg-cli update @core/lib      # Will use: git pull
pkg-cli remote list           # Will use: GitHub API or curl

# Configuration via environment variables
PKGS_GITHUB_USER="myuser" pkg-cli clone @core/lib  # Override for forks
PKGS_GITHUB_URL="https://mirror.com" pkg-cli clone @core/lib  # Override for mirrors
```

## Package Structure

Packages follow a standardized structure:

```
package-name/
├── package.conf              # Configuration file (optional)
├── .config/                  # Config files to install
│   └── app/
│       └── config.conf
├── .local/                   # Local files to install
│   ├── bin/
│   │   └── app
│   └── share/
│       └── app/
└── README.md                 # Documentation (auto-ignored)
```

### package.conf Format

```bash
# Metadata
name="package-name"
description="Package description"
version="1.0.0"

# Installation method
method="stow"                 # or "script"

# Dependencies
depends=("@core/lib")
requires_commands=("git" "curl")
supported_os=("linux" "darwin")

# Ignore patterns (Perl regex)
ignore=("README.md" "docs" ".*\.md")

# Hooks (optional)
preinstall() { ... }
postinstall() { ... }
preuninstall() { ... }
postuninstall() { ... }

# Custom install (if method="script")
custom_install() { ... }
custom_uninstall() { ... }

# Validation
validate() { ... }
```

## Directory Structure

```
~/.pkg-cli/
├── bin/
│   └── pkg-cli              # Main executable
├── lib/                     # Future: helper libraries
├── README.md                # This file
└── CLAUDE.md                # AI assistant documentation
```

## Configuration

### Environment Variables

- **`PKGS_SOURCE`**: Package source directory (default: `~/.pkgs`)
- **`PKGS_TARGET`**: Installation target directory (default: `$HOME`)
- **`PKGS_GITHUB_USER`**: GitHub username/organization (default: `andronics`)
  - Override for forks: `PKGS_GITHUB_USER="myusername" pkg-cli clone @core/lib`
- **`PKGS_GITHUB_URL`**: GitHub base URL (default: `https://github.com`)
  - Override for mirrors: `PKGS_GITHUB_URL="https://github.enterprise.com" pkg-cli clone @core/lib`

### PATH Setup

Add to `~/.zshrc` or `~/.bashrc`:

```bash
export PATH="$HOME/.pkg-cli/bin:$PATH"

# Optional: Override GitHub defaults for forks
# export PKGS_GITHUB_USER="myusername"
```

## Development

### Version History

- **0.1.0** (2025-11-13): Initial release
  - Refactored from pkgs-cli
  - Added dependency resolver
  - Added doctor command
  - Added GitHub integration stubs
  - Moved to standalone location

### Roadmap

**Phase 2** (Current):
- ✅ Standalone structure
- ✅ Basic commands (install, uninstall, reinstall, list, info)
- ✅ Dependency resolver
- ✅ Doctor command
- ✅ GitHub stubs

**Post-Phase 4:**
- ⏸️ Full GitHub integration (clone, update, remote)
- ⏸️ Search command
- ⏸️ Upgrade command
- ⏸️ Automated tests

## Troubleshooting

### Common Issues

**"stow not found"**
```bash
# Install GNU stow
sudo pacman -S stow  # Arch
sudo apt install stow  # Ubuntu/Debian
brew install stow  # macOS
```

**"Failed to change to ~/.pkgs"**
```bash
# Source directory doesn't exist
mkdir -p ~/.pkgs
# OR specify different location
pkg-cli --source /path/to/packages list
```

**Broken symlinks after uninstall**
```bash
# Run doctor to identify
pkg-cli doctor

# Remove manually or re-stow
stow -D package-name
```

## Contributing

This is a personal dotfiles management system, but suggestions and improvements are welcome!

## License

MIT

## Links

- **GitHub**: https://github.com/andronics/pkg-cli
- **pkgs Ecosystem**: https://github.com/andronics/.pkgs
- **Documentation**: See CLAUDE.md for AI assistant guidelines

---

**Made with ❤️ for managing dotfiles sanely**
