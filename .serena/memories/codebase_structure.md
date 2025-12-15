# Codebase Structure

## Directory Layout

```
watahaku-17/v0/
├── .github/workflows/     # CI/CD automation
│   ├── build.yml         # Main build workflow (uses ZMK's build-user-config)
│   └── draw-keymaps.yml  # Keymap visualization workflow
├── boards/shields/        # Hardware shield definitions
│   └── watahaku_17/      # Shield for watahaku-17 keyboard
│       ├── *.overlay     # DeviceTree overlays (left/right specific)
│       ├── *.dtsi        # DeviceTree includes (common definitions)
│       ├── *.conf        # Build configuration files
│       ├── *.keymap      # Default keymap (not used, see config/)
│       ├── Kconfig.*     # Kconfig build system files
│       └── *.zmk.yml     # ZMK metadata
├── config/               # User configuration (West project root)
│   ├── keymap.keymap     # Active keymap definition (THIS IS THE REAL KEYMAP)
│   ├── info.json         # Physical layout metadata for ZMK Studio
│   └── west.yml          # West manifest (dependencies)
├── snippets/             # Reusable DeviceTree snippets
│   ├── split-trackball/       # Trackball on peripheral
│   └── split-trackball-listner/ # Trackball listener on central
├── src/                  # Custom C source code
│   └── board.c          # Split power management implementation
├── zephyr/              # Zephyr module configuration
│   └── module.yml       # Defines board_root and snippet_root
├── keymap-drawer/       # Keymap visualization files
│   ├── keymap.svg       # Generated keymap image
│   ├── keymap.yaml      # Keymap description for drawer
│   └── config.yaml      # Drawer configuration
├── doc/                 # Documentation
├── tmp/                 # Temporary files
├── build.yaml           # Build matrix configuration (7 variants)
├── CMakeLists.txt       # CMake build configuration
├── CLAUDE.md            # AI assistant instructions
└── README.md            # Project readme
```

## Key Files and Their Purposes

### Hardware Configuration
- **`boards/shields/watahaku_17/watahaku_17.dtsi`**: Common DeviceTree definitions
  - GPIO matrix configuration (4 rows × 7 columns per side)
  - Keymap transformations for size S layout
  - KSCAN (keyboard scan) configuration
  - Status LED configuration
  - Trackball listener setup
  - Non-LiPo battery configuration

- **`boards/shields/watahaku_17/watahaku_17_left.overlay`**: Left side specific configuration
  - GPIO pin mappings
  - Trackball sensor configuration (if enabled)
  - Input transformations (X/Y inversion for left side)

- **`boards/shields/watahaku_17/watahaku_17_right.overlay`**: Right side specific configuration
  - GPIO pin mappings
  - Trackball sensor configuration (if enabled)
  - Default input orientation

- **`boards/shields/watahaku_17/watahaku_17_layouts.dtsi`**: Physical layout definitions
  - Key positions for ZMK Studio

### User Configuration
- **`config/keymap.keymap`**: **THE ACTIVE KEYMAP** (edit this, not the one in boards/shields/)
  - 7 layer definitions
  - Combo definitions
  - Auto mouse layer processor configuration

- **`config/west.yml`**: West workspace manifest
  - Defines ZMK version (v0.2)
  - Lists all dependencies and their versions
  - Sets `config/` as the West project root

- **`config/info.json`**: Physical layout metadata for ZMK Studio

### Custom Code
- **`src/board.c`**: Split power management
  - Progressive sleep modes based on idle time
  - Connection interval adjustment for power saving
  - USB power detection
  - Activity monitoring (keyboard + mouse input)

### Build System
- **`build.yaml`**: Build matrix for GitHub Actions
  - Defines 7 build variants with different board/shield/snippet/cmake-args combinations

- **`CMakeLists.txt`**: CMake configuration
  - Includes `src/board.c` when building watahaku_17 shields

- **`zephyr/module.yml`**: Zephyr module definition
  - Sets `board_root` and `snippet_root` to project directory
  - Allows custom boards and snippets

### DeviceTree Snippets
- **`snippets/split-trackball/`**: Snippet for enabling trackball input splitting
  - Creates virtual split input device
  - Transmits trackball data from peripheral to central

- **`snippets/split-trackball-listner/`**: Snippet for trackball listener on central side

## File Type Overview

### DeviceTree Files (.dts, .dtsi, .overlay)
- **Purpose**: Hardware description and configuration
- **.dtsi**: Include files with common/shared definitions
- **.overlay**: Override/extend base definitions for specific board variants

### Kconfig Files (.conf, Kconfig.*)
- **Purpose**: Build-time feature configuration
- **.conf**: Configuration values for specific shields
- **Kconfig.***: Kconfig menu definitions

### Keymap Files (.keymap)
- **Purpose**: Key behavior and layer definitions
- Uses ZMK's keymap syntax with DeviceTree-like structure

## Important Notes

1. **Keymap Location**: The active keymap is in `config/keymap.keymap`, NOT in `boards/shields/watahaku_17/watahaku_17.keymap`

2. **DeviceTree Hierarchy**:
   - Common definitions: `*.dtsi` files
   - Side-specific overrides: `*_left.overlay` or `*_right.overlay`
   - Both overlays must be updated if changes affect both sides

3. **West Workspace**: `config/` is the West project root as defined in `west.yml`

4. **Custom Code**: Only `src/board.c` contains custom C code (power management)
