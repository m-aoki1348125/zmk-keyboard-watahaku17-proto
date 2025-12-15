# watahaku-17 Project Overview

## Purpose
This is a ZMK firmware configuration for the **watahaku-17** split keyboard with integrated trackball support. The keyboard is a custom split mechanical keyboard that uses the ZMK (Zephyr Mechanical Keyboard) firmware framework running on the Zephyr RTOS.

## Technology Stack

### Core Framework
- **ZMK Firmware v0.2**: Open-source keyboard firmware based on Zephyr RTOS
- **Zephyr RTOS**: Real-time operating system for embedded devices
- **West**: Zephyr's meta-tool for managing multi-repository projects

### Hardware Platform
- **Board**: `bmp_boost` (BMP Boost development board)
- **MCU**: Nordic nRF52 series (based on GPIO and BLE configurations)
- **Connectivity**: Bluetooth Low Energy (BLE) for split keyboard communication

### Custom Components (from west.yml dependencies)
1. **zmk-component-bmp-boost**: BMP boost board support
2. **zmk-feature-status-led**: Status LED functionality
3. **zmk-driver-paw3222**: Trackball sensor driver (PAW3222)
4. **zmk-feature-cdc-acm-bootloader-trigger**: USB bootloader trigger support
5. **zmk-feature-non-lipo-battery-management**: Battery management for non-LiPo batteries
6. **zmk-scroll-snap**: Scroll snapping feature for trackball

### Build System
- **CMake**: Build configuration system
- **DeviceTree**: Hardware description language for defining keyboard matrix, pins, and peripherals
- **GitHub Actions**: Automated CI/CD for building firmware variants

## Key Features

### Split Keyboard
- Left and right halves communicate via BLE
- Each side can be configured as either central or peripheral
- Configurable split roles via `CONFIG_ZMK_SPLIT_ROLE_CENTRAL`

### Trackball Integration
- PAW3222 optical sensor support
- Split trackball modes:
  - `split-trackball`: Enables trackball on peripheral side
  - `split-trackball-listner`: Enables trackball listener on central side
- Configurable input transformations (X/Y axis inversion) per side

### Power Management
- Progressive power-saving modes based on idle time:
  - Active mode: Full speed
  - Sleep1 (5s idle): 2× connection interval
  - Sleep2 (15s idle): 4× connection interval
  - Sleep3 (30s idle): 8× connection interval
- Automatic return to active mode on keyboard/mouse input or USB power
- Custom implementation in `src/board.c`

### Battery Management
- Non-LiPo battery support (custom ADC-based monitoring)
- Disabled default ZMK battery monitoring

### Keymap
- 7 layers total:
  - Layer 0: Base (QWERTY layout)
  - Layer 1: Mouse buttons
  - Layer 2: Scroll mode
  - Layer 3: Numbers and navigation
  - Layer 4: Symbols
  - Layer 5: Function keys
  - Layer 6: Settings (Bluetooth management)
- Multiple combos for common shortcuts (Tab, Ctrl+F, Ctrl+Z/Y, etc.)
- Auto mouse layer activation with idle timeout

### Build Variants
The project builds 7 different firmware configurations:
1. Left central (standard)
2. Right peripheral (standard)
3. Left peripheral
4. Right central
5. Left peripheral with split trackball
6. Right central with trackball listener
7. Settings reset firmware
