# CLAUDE.md - AI Assistant Operating Manual for pkg-cli

## Project Overview

**Project**: pkg-cli - Standalone Package Manager
**Location**: `~/.pkg-cli/`
**Version**: 0.1.0
**Status**: Phase 2 Complete
**Language**: Bash

---

## Critical Context for AI Assistants

### What is pkg-cli?

`pkg-cli` is a **standalone bootstrap tool** for managing packages in the pkgs ecosystem. It:
- Lives OUTSIDE the managed package ecosystem (at `~/.pkg-cli/`)
- Cannot manage itself (avoids circular dependency)
- Uses GNU Stow for symlink-based installations
- Manages packages located in `~/.pkgs/`

### Key Principle

**STANDALONE = NOT SELF-MANAGED**

This tool must NEVER be in `~/.pkgs/`. It's the bootstrap mechanism that manages everything else.

---

## Architecture

### File Structure

```
~/.pkg-cli/
├── bin/
│   └── pkg-cli          # Main executable (1167 lines of bash)
├── lib/                 # Future: helper libraries
├── README.md            # User documentation
├── CLAUDE.md            # This file
└── .git/                # Git repository
```

### Core Components

1. **Logging Functions** (lines 50-100)
   - `log_info`, `log_success`, `log_warn`, `log_error`
   - Color-coded output

2. **Package Config** (lines 100-200)
   - `load_package_config()` - Loads package.conf
   - `reset_package_config()` - Resets variables

3. **Core Commands** (lines 548-736)
   - `cmd_install` - Install packages with dependency resolution
   - `cmd_uninstall` - Uninstall packages
   - `cmd_reinstall` - Reinstall packages
   - `cmd_list` - List available packages
   - `cmd_info` - Show package details

4. **GitHub Stubs** (lines 864-900)
   - `cmd_clone` - Stub for cloning from GitHub
   - `cmd_update` - Stub for updating from GitHub
   - `cmd_remote` - Stub for listing remote packages

5. **Dependency Resolver** (lines 906-956)
   - `pkg_resolve_deps` - Parse depends=() from package.conf
   - `pkg_install_with_deps` - Recursive dependency installation
   - `is_package_installed` - Check if package is installed

6. **Doctor Command** (lines 962-1078)
   - System health checks (7 checks)
   - Validates environment and configuration

7. **Main Function** (lines 1084-1163)
   - Argument parsing
   - Command routing

---

## Development Guidelines

### When Making Changes

1. **Test After Edits**
   ```bash
   # Syntax check
   bash -n ~/.pkg-cli/bin/pkg-cli

   # Test help
   ~/.pkg-cli/bin/pkg-cli --help

   # Test doctor
   ~/.pkg-cli/bin/pkg-cli doctor
   ```

2. **Preserve Existing Functionality**
   - All original commands must continue to work
   - Don't break backward compatibility in v0.1.x

3. **Follow Bash Best Practices**
   - Use `set -euo pipefail`
   - Quote variables: `"${var}"`
   - Use `local` for function variables
   - Check return codes

4. **Conventional Commits**
   ```
   feat(cli): add new command
   fix(install): resolve dependency bug
   docs(readme): update usage examples
   refactor(doctor): improve health checks
   ```

### Adding New Commands

1. Create `cmd_<name>()` function
2. Add to help text in `cmd_help()`
3. Add to command parsing (line 1119)
4. Add to case statement (line 1150)
5. Test thoroughly

**Example:**
```bash
cmd_search() {
    local term="$1"
    log_info "Searching for: ${term}"
    # Implementation
}

# Add to line 1119:
i|install|...|search)

# Add to line 1150:
search) cmd_search "${packages[@]}" ;;
```

### Adding New Features

**Before coding:**
1. Check if it aligns with Phase 2 scope
2. Consider if it should be in pkg-cli or a package
3. Review MIGRATION-PLAN.md for guidance

**Current Phase 2 Scope:**
- ✅ Basic commands (install, uninstall, reinstall, list, info)
- ✅ Dependency resolver
- ✅ Doctor command
- ✅ GitHub stubs (not full implementation)

**Deferred to Post-Phase 4:**
- ⏸️ Full GitHub integration
- ⏸️ Search command
- ⏸️ Upgrade command

---

## Common Tasks

### Updating Dependencies

Dependency logic is in `pkg_install_with_deps()`:

```bash
pkg_install_with_deps() {
    local package="$1"
    local -a deps

    # Get dependencies from package.conf
    mapfile -t deps < <(pkg_resolve_deps "$package")

    # Install dependencies first (recursive)
    for dep in "${deps[@]}"; do
        [[ -z "$dep" ]] && continue
        if ! is_package_installed "$dep"; then
            log_info "Installing dependency: ${dep}"
            pkg_install_with_deps "$dep"
        fi
    done

    # Install the package itself
    install_package "$package"
}
```

To modify dependency resolution:
1. Edit `pkg_resolve_deps()` - parsing logic
2. Edit `pkg_install_with_deps()` - installation logic
3. Edit `is_package_installed()` - detection logic

### Updating Doctor Checks

Doctor command is in `cmd_doctor()`. Current checks:

1. **stow installation** - Verify GNU stow
2. **PKGS_SOURCE exists** - Check ~/.pkgs directory
3. **package.conf validity** - Syntax check all configs
4. **Broken symlinks** - Find broken links in $HOME
5. **Required commands** - Check git, curl, zsh, bash
6. **lib package structure** - Verify Phase 1 completion
7. **PATH configuration** - Check if pkg-cli in PATH

To add a check:
```bash
# Check 8: New check
print_colored "${BLUE}[8/8]${NC} Checking something..."
if some_condition; then
    log_success "Check passed"
else
    log_error "Check failed"
    ((issues++))
fi
echo
```

Update summary count from `[7/7]` to `[8/8]`.

### Implementing GitHub Features

**Current Status:** Stubs only (show TODO messages)

**Post-Phase 4 Implementation:**

1. **Clone Command**
   ```bash
   cmd_clone() {
       local package="$1"

       # Convert @core/lib → pkg-core-lib
       local repo_name="${package//@/pkg-}"
       repo_name="${repo_name//\//-}"

       # Clone from GitHub
       gh repo clone "andronics/${repo_name}" "${PKGS_SOURCE}/${package}"
   }
   ```

2. **Update Command**
   ```bash
   cmd_update() {
       local package="$1"
       local pkg_dir="${PKGS_SOURCE}/${package}"

       [[ ! -d "${pkg_dir}/.git" ]] && {
           log_error "Not a git repository: ${package}"
           return 1
       }

       git -C "${pkg_dir}" pull
   }
   ```

3. **Remote List**
   ```bash
   cmd_remote() {
       gh repo list andronics --limit 100 | grep "^pkg-"
   }
   ```

**Don't implement these yet** - wait for Phase 4 when repos exist.

---

## Testing

### Manual Testing Checklist

After making changes, test these scenarios:

```bash
# 1. Help and version
pkg-cli --help
pkg-cli help

# 2. List packages
pkg-cli list

# 3. Package info
pkg-cli info lib

# 4. Install (dry-run)
pkg-cli -n install lib

# 5. Doctor command
pkg-cli doctor

# 6. Verbose mode
pkg-cli -v list

# 7. GitHub stubs
pkg-cli clone @core/lib    # Should show TODO
pkg-cli update lib          # Should show TODO
pkg-cli remote             # Should show TODO

# 8. Error handling
pkg-cli install nonexistent  # Should error gracefully
pkg-cli                      # Should show help + error
```

### Syntax Validation

```bash
# Check bash syntax
bash -n ~/.pkg-cli/bin/pkg-cli

# Check for common issues
shellcheck ~/.pkg-cli/bin/pkg-cli  # If installed
```

---

## Common Pitfalls

### For AI Assistants

1. **Don't move pkg-cli to ~/.pkgs/**
   - It must stay standalone at ~/.pkg-cli/

2. **Don't implement full GitHub features yet**
   - Stubs only until Phase 4 (when repos exist)

3. **Don't break dependency resolution**
   - Critical feature for package ecosystem

4. **Don't remove existing commands**
   - install, uninstall, reinstall, list, info must work

5. **Don't forget to update help text**
   - When adding commands, update `cmd_help()`

6. **Don't use functions before they're defined**
   - Bash reads top-to-bottom

7. **Don't assume pkg-cli is in PATH**
   - Use full path or test: `command -v pkg-cli`

### For Humans

1. Always test after making changes
2. Check MIGRATION-PLAN.md before adding features
3. Use conventional commits
4. Document new features in README.md
5. Keep CLAUDE.md updated

---

## Variable Reference

### Global Configuration

```bash
PKG_CLI_VERSION="0.1.0"              # Version number
PKG_CLI_DIR="<path-to-bin>"          # Where pkg-cli is installed
CONFIG_FILE="package.conf"           # Config filename
AUTO_IGNORE="package.conf|\.git.*"   # Always ignored patterns
PKGS_SOURCE="${HOME}/.pkgs"          # Package source directory
PKGS_TARGET="${HOME}"                # Installation target
```

### Global Flags

```bash
DRY_RUN=false          # Preview mode
VERBOSE=false          # Detailed output
SKIP_HOOKS=false       # Skip pre/post hooks
VALIDATE_INSTALL=true  # Run validation after install
```

### Package Config Variables

Loaded from `package.conf`:

```bash
name=""                # Package name
description=""         # Package description
version=""             # Package version
method="stow"          # Installation method
depends=()             # Package dependencies
requires_commands=()   # Required system commands
supported_os=()        # Supported operating systems
ignore=()              # Custom ignore patterns
fold=false             # Stow folder folding
```

---

## Debugging

### Enable Verbose Mode

```bash
pkg-cli -v install lib
```

### Trace Execution

```bash
bash -x ~/.pkg-cli/bin/pkg-cli install lib 2>&1 | less
```

### Check What Stow Would Do

```bash
# Dry-run
stow -nvt ~ -d ~/.pkgs lib

# Actual install
stow -vt ~ -d ~/.pkgs lib
```

### Find Where a Symlink Points

```bash
ls -la ~/.local/bin/some-command
readlink -f ~/.local/bin/some-command
```

---

## Git Workflow

### Committing Changes

```bash
cd ~/.pkg-cli

# Check status
git status

# Add changes
git add bin/pkg-cli README.md

# Commit with conventional commit
git commit -m "feat(cli): add new feature"

# Push to GitHub (if remote exists)
git push origin main
```

### Syncing with Remote

```bash
cd ~/.pkg-cli

# Pull latest
git pull

# Test after pulling
pkg-cli doctor
```

---

## Integration with pkgs Ecosystem

### Relationship to Other Components

```
~/.pkg-cli/          → Standalone bootstrap tool (this project)
    └── bin/pkg-cli  → Manages packages in ~/.pkgs/

~/.pkgs/             → Package source directory
    ├── lib/         → @core/lib (Phase 1 restructured)
    ├── audio/       → @system/audio
    ├── player/      → @apps/player
    └── ...          → (40 packages total)

~/ (target)          → Installation target
    ├── .config/     → Symlinked config files
    ├── .local/      → Symlinked local files
    └── ...
```

### Package Naming Convention

```bash
# Old style (pre-Phase 3)
lib, audio, player

# New style (Phase 3+)
@core/lib, @system/audio, @apps/player

# GitHub repos (Phase 4+)
pkg-core-lib, pkg-system-audio, pkg-apps-player
```

pkg-cli should support both naming styles.

---

## Emergency Procedures

### If Something Breaks

1. **Restore from backup**
   ```bash
   cp ~/.pkg-cli.backup-XXXXXX/bin/pkg-cli ~/.pkg-cli/bin/pkg-cli
   ```

2. **Check syntax**
   ```bash
   bash -n ~/.pkg-cli/bin/pkg-cli
   ```

3. **Review git log**
   ```bash
   cd ~/.pkg-cli
   git log --oneline -10
   git diff HEAD~1
   ```

4. **Revert last commit**
   ```bash
   git revert HEAD
   ```

### If Doctor Fails

Doctor command failing is serious - it means something is wrong with the environment.

Check each section:
1. Install stow if missing
2. Create ~/.pkgs if missing
3. Fix broken package.conf files
4. Clean up broken symlinks
5. Install missing commands
6. Add pkg-cli to PATH

---

## Version History

- **0.1.0** (2025-11-13)
  - Initial release from pkgs-cli refactor
  - Added dependency resolver
  - Added doctor command
  - Added GitHub integration stubs
  - Phase 2 complete

---

## Contact & Support

- **Maintainer**: andronics
- **GitHub**: https://github.com/andronics/pkg-cli
- **pkgs Ecosystem**: https://github.com/andronics/.pkgs

---

*This document is living documentation. Update it as pkg-cli evolves.*
