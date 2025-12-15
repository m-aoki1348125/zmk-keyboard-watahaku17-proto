# Build Commands and Development Workflow

## Build System Overview

This project uses **West** (Zephyr's meta-tool) for local builds and **GitHub Actions** for automated CI/CD builds.

## Local Build Setup

### First-Time Setup

```bash
# 1. Initialize West workspace (from project root)
west init -l config/

# 2. Update dependencies (fetches ZMK and all components from west.yml)
west update

# 3. Export Zephyr environment
west zephyr-export
```

### Build Commands

#### Build for Specific Configuration

```bash
# Build left side as central (USB connected side)
west build -b bmp_boost -s app -- -DSHIELD=watahaku_17_left -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y

# Build right side as peripheral
west build -b bmp_boost -s app -- -DSHIELD=watahaku_17_right

# Build left side as peripheral
west build -b bmp_boost -s app -- -DSHIELD=watahaku_17_left

# Build right side as central
west build -b bmp_boost -s app -- -DSHIELD=watahaku_17_right -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y

# Build with trackball support (peripheral side)
west build -b bmp_boost -s app -- -DSHIELD=watahaku_17_left -DSNIPPET="studio-rpc-usb-uart split-trackball"

# Build with trackball listener (central side)
west build -b bmp_boost -s app -- -DSHIELD=watahaku_17_right -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y -DSNIPPET="studio-rpc-usb-uart split-trackball-listner"

# Clean build
west build -t clean
```

#### Build Parameters Explained

- **`-b bmp_boost`**: Target board (BMP Boost development board)
- **`-s app`**: Source directory (ZMK application directory)
- **`-DSHIELD=...`**: Shield definition (watahaku_17_left or watahaku_17_right)
- **`-DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y`**: Configure as central (USB host) side
- **`-DSNIPPET="..."`**: Apply DeviceTree snippets (space-separated list)
  - `studio-rpc-usb-uart`: Enable ZMK Studio RPC over USB serial
  - `split-trackball`: Enable trackball input splitting (peripheral)
  - `split-trackball-listner`: Enable trackball listener (central)

## GitHub Actions Build

### Automated Builds

Builds are triggered automatically on:
- Push to any branch
- Pull requests
- Manual workflow dispatch

### Build Matrix (from build.yaml)

The CI builds 7 firmware variants:

1. **watahaku_17_left_central**: Left as central (standard)
2. **watahaku_17_right_peripheral**: Right as peripheral (standard)
3. **watahaku_17_left_peripheral**: Left as peripheral
4. **watahaku_17_right_central**: Right as central
5. **watahaku_17_double_ball_left_peripheral**: Left with split trackball
6. **watahaku_17_double_ball_right_central**: Right with trackball listener
7. **settings_reset**: Settings reset firmware

### Artifacts

Firmware files (`.uf2`) are available as GitHub Actions artifacts after each build.

## Common Development Commands

### Git Commands
```bash
# Check status
git status

# Stage changes
git add <file>

# Commit changes
git commit -m "message"

# Push to remote
git push

# View commit history
git log --oneline -10
```

### File Operations
```bash
# List files/directories
ls -la

# Change directory
cd <path>

# Find files
find <path> -name "pattern"

# Search in files
grep -r "pattern" <path>

# View file content
cat <file>
head -n 20 <file>
tail -n 20 <file>
```

### DeviceTree and Keymap Utilities

```bash
# Check keymap syntax (if zmk-cli is available)
keymap check config/keymap.keymap

# Generate keymap visualization (using keymap-drawer, if configured)
# This is typically done via GitHub Actions (see .github/workflows/draw-keymaps.yml)
```

## Flashing Firmware

### UF2 Bootloader Method
1. Put keyboard into bootloader mode:
   - Double-tap reset button, or
   - Use bootloader trigger combo (if configured), or
   - Use USB CDC ACM trigger command
2. A USB drive appears (typically named `BMP-BOOST`)
3. Copy the `.uf2` file to the drive
4. The keyboard automatically resets with new firmware

### Flash Locations
- **Local builds**: `build/zephyr/zmk.uf2`
- **GitHub Actions**: Download from workflow artifacts

## Debugging

### View Build Logs
```bash
# Local build with verbose output
west build -b bmp_boost -s app -- -DSHIELD=watahaku_17_left -v

# GitHub Actions: View workflow logs in the Actions tab
```

### Common Build Issues

1. **West workspace not initialized**: Run `west init -l config/`
2. **Dependencies not fetched**: Run `west update`
3. **Environment not exported**: Run `west zephyr-export`
4. **Outdated cache**: Run `west build -t clean` and rebuild

## Project-Specific Notes

### Important Paths
- **West manifest**: `config/west.yml`
- **Active keymap**: `config/keymap.keymap` (NOT boards/shields/watahaku_17/watahaku_17.keymap)
- **Build configuration**: `build.yaml`
- **Hardware definitions**: `boards/shields/watahaku_17/*.overlay` and `*.dtsi`

### System Environment
- **Platform**: Linux (WSL2 on Windows)
- **OS**: Linux 6.6.87.1-microsoft-standard-WSL2
- **Shell**: Standard Linux commands available
