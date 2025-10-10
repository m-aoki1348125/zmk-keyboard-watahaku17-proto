# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a ZMK firmware configuration for the "torabo-tsuki" split keyboard with integrated trackball support. The project uses the Zephyr RTOS and is built on top of ZMK (Zephyr Mechanical Keyboard) framework.

## Build System

### GitHub Actions Build
Builds are automated via GitHub Actions (`.github/workflows/build.yml`), which uses ZMK's official build workflow. The `build.yaml` configuration defines multiple build variants:

- **Central/Peripheral configurations**: Each side can be built as either central or peripheral
- **Trackball variants**:
  - `split-trackball`: Enables split trackball on peripheral side
  - `split-trackball-listner`: Enables trackball listener on central side
- **Board**: Uses `bmp_boost` board
- **Snippet**: `studio-rpc-usb-uart` for USB serial communication

### Local Build
This project uses West (Zephyr's meta-tool) for building. Based on `config/west.yml`:

```bash
# Initialize west workspace (first time only)
west init -l config/
west update

# Build for specific configuration (example)
west build -b bmp_boost -s app -- -DSHIELD=torabo_tsuki_lp_left -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y
```

## Architecture

### Directory Structure

- **`boards/shields/torabo_tsuki_lp/`**: Shield definition for the keyboard hardware
  - `*.overlay`: DeviceTree overlays for left/right sides
  - `*.dtsi`: Common DeviceTree definitions and layout configurations
  - `Kconfig.*`: Build configuration
  - `*.keymap`: Default keymap (not used; actual keymap is in `config/`)

- **`config/`**: User configuration (this is the West project root)
  - `keymap.keymap`: Active keymap with 4 layers
  - `info.json`: Physical layout metadata for ZMK Studio
  - `west.yml`: Manifest defining ZMK and dependencies

- **`src/board.c`**: Custom C code for split power management

- **`snippets/`**: Reusable DeviceTree snippets
  - `split-trackball/`: Enables trackball input splitting between halves

- **`zephyr/module.yml`**: Defines this directory as a Zephyr module with custom board/snippet roots

### Key Dependencies (from west.yml)

- **ZMK**: v0.2 from zmkfirmware
- **zmk-component-bmp-boost**: BMP boost board support
- **zmk-feature-status-led**: Status LED support
- **zmk-driver-paw3222**: Trackball sensor driver (PAW3222)
- **zmk-feature-cdc-acm-bootloader-trigger**: USB bootloader trigger
- **zmk-feature-non-lipo-battery-management**: Battery management for non-LiPo batteries

### Hardware Configuration

The shield supports multiple physical layouts (S, M, L sizes) with different key counts:
- **Size S**: 4 rows with thumb clusters (40 keys)
- **Size L**: 5 rows with thumb clusters (70 keys)

Key hardware features configured in `torabo_tsuki_lp.dtsi`:
- **Split keyboard**: Uses ZMK's split support with BLE
- **GPIO matrix**: 5 rows × 7 columns per side
- **Trackball**: PAW3222 sensor on SPI0
- **Status LED**: GPIO-controlled LED for status indication
- **Battery monitoring**: Custom non-LiPo battery management

### Power Management (src/board.c)

The custom power management code implements progressive power-saving modes for the split connection:

1. **Active mode**: Full speed (USB powered or recent activity)
2. **Sleep1**: After 5s idle - 2× connection interval
3. **Sleep2**: After 15s idle - 4× connection interval
4. **Sleep3**: After 30s idle - 8× connection interval

The system automatically returns to active mode on:
- Keyboard input (position state changes)
- Mouse input (trackball movement)
- USB power connection

### Keymap Structure

The keymap (`config/keymap.keymap`) has 4 layers:
- **Layer 0**: Base QWERTY layout with mouse clicks
- **Layer 1**: Mouse button layer
- **Layer 2**: Numbers and symbols (accessed via hold on layer 0)
- **Layer 3**: Function keys, navigation, and Bluetooth controls

Bluetooth clear combo: Pressing positions 28+29 on layer 2.

### Trackball Configuration

The trackball uses different input transformations for left/right sides:
- **Left side**: X and Y inverted (`INPUT_TRANSFORM_X_INVERT | INPUT_TRANSFORM_Y_INVERT`)
- **Right side**: Default orientation (defined in respective `.overlay` files)

For split trackball configurations, the `split-trackball` snippet creates a virtual split input device that transmits trackball data from peripheral to central.

## Development Notes

### DeviceTree Editing
When modifying hardware configuration:
- Common definitions go in `*.dtsi` files
- Side-specific overrides go in `*_left.overlay` or `*_right.overlay`
- Remember to update both left and right overlays if changes affect both sides

### Keymap Modifications
Edit `config/keymap.keymap` - this is the active keymap, not the one in `boards/shields/`.

### Adding New Build Configurations
Add new entries to `build.yaml` with appropriate board, shield, cmake-args, and snippet combinations.

### Code Style
C code follows ZMK conventions:
- SPDX license headers
- Zephyr logging (`LOG_MODULE_REGISTER`, `LOG_INF`, etc.)
- Zephyr kernel APIs (`k_work_*`, `k_uptime_get`, etc.)
