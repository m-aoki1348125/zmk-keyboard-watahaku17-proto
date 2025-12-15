# Code Style and Conventions

## General Philosophy
This project follows ZMK firmware conventions and Zephyr RTOS coding standards.

## C Code Style (src/board.c)

### File Headers
```c
// SPDX-License-Identifier: GPL-2.0-or-later
// copyright (C) 2025 sekigon-gonnoc
```
- SPDX license identifier on first line
- Copyright notice on second line

### Includes
```c
#include <zephyr/sys/util_macro.h>
#include <zephyr/kernel.h>
#include <zephyr/bluetooth/bluetooth.h>
#include <zephyr/logging/log.h>
#include <zmk/events/*.h>
```
- System headers first (Zephyr)
- ZMK headers second
- Grouped logically

### Logging
```c
LOG_MODULE_REGISTER(split_power_mgmt, CONFIG_ZMK_LOG_LEVEL);

LOG_INF("Returned to active mode");
LOG_DBG("USB power detected, staying in active mode");
```
- Use Zephyr logging macros (`LOG_INF`, `LOG_DBG`, `LOG_ERR`, `LOG_WRN`)
- Register log module at file scope
- Use appropriate log levels

### Naming Conventions
- **Functions**: `snake_case` (e.g., `power_mode_transition`)
- **Variables**: `snake_case` (e.g., `current_mode`, `last_activity_time`)
- **Enums**: `UPPER_CASE` for values (e.g., `POWER_MODE_ACTIVE`)
- **Defines/Macros**: `UPPER_CASE` (e.g., `SLEEP1_TIMEOUT_MS`)
- **Static variables**: Prefix with `static` keyword

### Formatting
- **Indentation**: 4 spaces (no tabs in C code)
- **Braces**: K&R style (opening brace on same line for functions, control structures)
- **Line length**: Keep reasonable (no strict limit, but avoid overly long lines)

### Comments
- Use `//` for single-line comments
- Document complex logic and important state transitions
- No excessive commenting for self-explanatory code

## DeviceTree Style (*.dts, *.dtsi, *.overlay)

### Indentation
- **4 spaces** per level (no tabs)

### Node Structure
```dts
node_label: node_name {
    compatible = "vendor,device";
    property = <value>;
    
    child_node {
        property = <value>;
    };
};
```

### GPIO Definitions
```dts
col-gpios
    = <&gpio0 22 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>,
    <&gpio1 15   (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>,
    <&gpio0 23   (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>
    ;
```
- Align continuation lines
- One GPIO per line for readability
- Semicolon on separate line for long lists

### Property Formatting
```dts
// Simple properties
compatible = "zmk,kscan-gpio-matrix";
status = "okay";

// Multi-line arrays
map = <
    RC(0,1) RC(0,2) RC(0,3)
    RC(1,1) RC(1,2) RC(1,3)
>;
```

### Comments
```dts
// Single-line comments
/* Multi-line comments when needed */

// Commented-out code blocks (size M layout example)
// size_m_transform: keymap_transform_1 {
//     ...
// };
```

## Keymap Style (*.keymap)

### File Structure
```keymap
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>

/ {
    combos { ... };
    
    input_processors { ... };
    
    keymap {
        layer_name {
            display-name = "Display Name";
            bindings = <...>;
        };
    };
};
```

### Layer Naming
- **Node name**: `snake_case` (e.g., `main_layer0`, `num_layer`)
- **Display name**: Human-readable (e.g., "Base", "Number", "Settings")

### Key Bindings Formatting
```keymap
bindings = <
    &kp Q         &kp W             &kp E   &kp R   &kp T
    &kp A         &kp S             &kp D   &kp F   &kp G
>;
```
- Align columns for visual representation of physical layout
- Group by rows (blank lines between rows optional but common)
- Use spacing to align similar columns

### Behavior Naming
- Use ZMK standard behaviors: `&kp`, `&mt`, `&lt`, `&mkp`, `&bt`, etc.
- Clear parameter names: `&kp TAB` not `&kp 43`

## Configuration Files (*.conf)

### Format
```conf
# Comments start with #
CONFIG_SETTING=y
CONFIG_ANOTHER_SETTING=n
CONFIG_VALUE_SETTING=123
```

### Conventions
- One setting per line
- Use comments to group related settings
- Boolean values: `y` (yes) or `n` (no)

## Build Configuration (build.yaml)

### Format
```yaml
include:
  - board: bmp_boost
    shield: watahaku_17_left
    snippet: studio-rpc-usb-uart
    cmake-args: -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y
    artifact-name: watahaku_17_left_central
```

### Conventions
- **Indentation**: 2 spaces
- **Alignment**: Align similar properties vertically
- Clear artifact naming: `<shield>_<role>` or `<shield>_<variant>_<role>`

## Documentation

### Markdown Files (*.md)
- Use proper heading hierarchy (# ## ###)
- Code blocks with language specifiers: ```bash, ```c, ```dts
- Clear section organization
- Examples for complex concepts

### Comments in CLAUDE.md
- Clear section headers
- Practical examples
- Architecture explanations
- Development notes

## Project-Specific Patterns

### Power Management (src/board.c)
- Use Zephyr work queues: `k_work_*` APIs
- Time tracking with `k_uptime_get()`
- BLE connection parameter updates via `bt_conn_le_param_update()`

### DeviceTree Patterns
- Common definitions in `.dtsi` files
- Side-specific overrides in `.overlay` files
- Reference nodes with `&label` syntax

### Keymap Patterns
- Combos for frequently used shortcuts
- Layer toggle/momentary: `&lt`, `&mo`
- Mod-tap for dual-purpose keys: `&mt`
- Mouse key prefix: `&mkp`
- Bluetooth controls: `&bt BT_SEL 0`, `&bt BT_CLR`

## No Strict Rules For

- **Line length**: No hard limit, use judgment
- **Documentation**: Document when helpful, not for obvious code
- **Type hints**: Not applicable (C and DeviceTree)
- **Docstrings**: Not a standard practice in Zephyr/ZMK
