# DYA Studio Migration Status

**Branch:** JIS-SP-DYA
**Base:** JIS-SP (commit c699be0)
**Status:** ⚠️ Work in Progress (Build Blocked)
**Last Updated:** 2026-02-14

## 🎯 Goal

Migrate from ZMK v0.2 to cormoran's v0.3-branch+dya to enable DYA Studio features while maintaining existing functionality (JIS layout, PAW3222 trackball, split keyboard).

## ✅ Completed Tasks

### Phase 1: Branch Setup
- ✅ Created `JIS-SP-DYA` branch from `JIS-SP` (c699be0)
- ✅ Documented baseline commit for rollback

### Phase 2: Dependencies
- ✅ Updated `config/west.yml`:
  - Changed ZMK: `zmkfirmware/zmk@v0.2` → `cormoran/zmk@v0.3-branch+dya`
  - Added cormoran remote
  - Added 4 DYA Studio modules:
    - `zmk-module-ble-management` (main)
    - `zmk-module-battery-history` (main)
    - `zmk-module-settings-rpc` (main)
    - `zmk-module-runtime-input-processor` (main)
- ✅ Executed `west update` successfully
- ✅ All dependencies downloaded (ZMK, Zephyr, DYA modules, existing modules)

### Phase 3: Configuration Files
- ✅ Updated `boards/shields/watahaku_17/watahaku_17_left.conf`:
  - Added `CONFIG_ZMK_BLE_MANAGEMENT=y`
  - Added `CONFIG_ZMK_BLE_MANAGEMENT_STUDIO_RPC=y`
  - Added `CONFIG_ZMK_BATTERY_REPORTING=y`
  - Added `CONFIG_SETTINGS=y`
  - Added `CONFIG_ZMK_BATTERY_HISTORY=y`
  - Added `CONFIG_ZMK_BATTERY_HISTORY_STUDIO_RPC=y`
  - Added `CONFIG_ZMK_SETTINGS_RPC=y`
  - Added `CONFIG_ZMK_SETTINGS_RPC_STUDIO=y`
  - Added `CONFIG_ZMK_RUNTIME_INPUT_PROCESSOR=y`
  - Added `CONFIG_ZMK_RUNTIME_INPUT_PROCESSOR_STUDIO_RPC=y`
  - Disabled `CONFIG_ZMK_CDC_ACM_BOOTLOADER_TRIGGER` (v0.3 compatibility issue)
- ✅ Updated `boards/shields/watahaku_17/watahaku_17_right.conf` (same changes)

### Phase 4: Device Tree Fixes
- ✅ Fixed `watahaku_17_left.overlay`:
  - Added `#include <behaviors.dtsi>` (required for `&none` behavior)
  - Added `auto_mouse_layer` node definition (temp-layer input processor)
- ✅ Fixed `watahaku_17_right.overlay` (same changes)

### Phase 5: Build System
- ✅ Updated `.github/workflows/build.yml`:
  - Changed workflow: `zmkfirmware/zmk@v0.2` → `cormoran/zmk@v0.3-branch+dya`
- ✅ Created `config/zephyr/module.yml`:
  - Set `board_root: ..` (points to v0 directory)
  - Set `snippet_root: ..` (points to v0 directory)
  - Enables shield discovery from `boards/shields/watahaku_17/`

## ⚠️ Current Blocker

### Kconfig Validation Error

**Error:**
```
warning: ZMK_KSCAN_DIRECT_POLLING (defined at .../clueboard_california/Kconfig.defconfig:11)
         defined without a type
error: Aborting due to Kconfig warnings
```

**Root Cause:**
- ZMK v0.3's Kconfig validation treats all warnings as errors
- `ZMK_KSCAN_DIRECT_POLLING` warning originates from `clueboard_california` shield (not our code)
- This is a known issue in ZMK v0.3 development

**Impact:**
- Build cannot proceed past CMake configuration stage
- No firmware artifacts generated
- Cannot test DYA Studio features

## 🔧 Attempted Solutions

1. ❌ Suppressed deprecation warnings with `CONFIG_WARN_DEPRECATED=n`
   - Result: Reduced warnings but `ZMK_KSCAN_DIRECT_POLLING` warning persists
2. ❌ Disabled USB CDC ACM bootloader trigger
   - Result: Resolved USB dependency conflict but Kconfig validation still fails
3. ❌ Set `-DKCONFIG_WARN_UNDEF=n`
   - Result: Flag not recognized by Kconfig validation

## 📋 Next Steps

### Option 1: Wait for Upstream Fix (Recommended)
- Monitor [cormoran/zmk v0.3-branch+dya](https://github.com/cormoran/zmk/tree/v0.3-branch+dya) for updates
- Check if newer commits fix Kconfig validation issues
- Estimated timeline: Unknown (depends on upstream)

### Option 2: Patch ZMK Locally
- Modify `/mnt/c/work/keyboard/watahaku-17/v0/zmk/app/boards/shields/clueboard_california/Kconfig.defconfig`
- Add type to `ZMK_KSCAN_DIRECT_POLLING` definition
- **Risk:** Patch may conflict with future updates

### Option 3: Disable Strict Kconfig Validation
- Modify `/mnt/c/work/keyboard/watahaku-17/v0/zephyr/cmake/modules/kconfig.cmake:355`
- Remove or comment out "abort on warnings" behavior
- **Risk:** May hide legitimate configuration errors

### Option 4: Use Different ZMK v0.3 Branch
- Try `zmkfirmware/zmk@main` (official development branch)
- May not have DYA Studio patches but could have Kconfig fixes
- **Trade-off:** Lose DYA Studio features

## 🧪 Testing Plan (Once Build Succeeds)

### Priority 1: Existing Features (Critical)
1. **JIS Layout** (`zmk-layout-shift` v1):
   - Test `@`, `[`, `]`, `:`, and other JIS-specific symbols
   - Verify layout-shift persistent state
2. **PAW3222 Trackball** (`zmk-driver-paw3222` main):
   - Cursor movement (half speed with threshold filtering)
   - Left/right click
   - Scroll mode on layers 2, 3, 4
   - Auto-mouse layer activation
3. **Split Keyboard**:
   - Left-right pairing
   - Bluetooth connection stability
   - Battery level reporting from both sides
4. **Other Features**:
   - Status LED behavior
   - All 7 keymap layers

### Priority 2: DYA Studio Features (New)
1. **Web UI Connection**:
   - USB serial connection to https://studio.dya.cormoran.works/
   - Bluetooth connection (main DYA advantage)
2. **BLE Management**:
   - Profile switching
   - Device name changes
   - Connection management
3. **Battery History**:
   - Graph display
   - Historical data accuracy
4. **Settings RPC**:
   - Sleep timeout adjustment
   - Idle timeout adjustment
   - Settings persistence
5. **Runtime Input Processor**:
   - Trackball sensitivity adjustment via Web UI
   - Immediate effect (no firmware rebuild)
   - Settings persistence

## 📁 Modified Files

```
config/west.yml                                  # DYA dependencies added
config/zephyr/module.yml                         # New: board_root/snippet_root config
.github/workflows/build.yml                      # Updated to v0.3-branch+dya
boards/shields/watahaku_17/watahaku_17_left.conf # DYA configs added, CDC disabled
boards/shields/watahaku_17/watahaku_17_right.conf # DYA configs added, CDC disabled
boards/shields/watahaku_17/watahaku_17_left.overlay # behaviors.dtsi, auto_mouse_layer added
boards/shields/watahaku_17/watahaku_17_right.overlay # behaviors.dtsi, auto_mouse_layer added
```

## 🔄 Rollback Instructions

If you need to return to the stable JIS-SP branch:

```bash
git checkout JIS-SP
west update  # Restore v0.2 dependencies
rm -rf build/ .west/
```

The `JIS-SP-DYA` branch is preserved for future work once compatibility issues are resolved.

## 📚 References

- [DYA Studio Web UI](https://studio.dya.cormoran.works/)
- [cormoran/zmk v0.3-branch+dya](https://github.com/cormoran/zmk/tree/v0.3-branch+dya)
- [DYA Dash Keyboard (Reference Implementation)](https://github.com/cormoran/dya-dash-keyboard)
- [DYA Studio導入記事 (moNa2編)](https://note.com/heace/n/nf06b797ffa79)
- [kot149/zmk-layout-shift](https://github.com/kot149/zmk-layout-shift)
- [sekigon-gonnoc GitHub](https://github.com/sekigon-gonnoc)

## 💡 Community Support

If you encounter this Kconfig issue:
1. Check [ZMK Discord](https://zmk.dev/community/discord) for similar reports
2. Search [ZMK GitHub issues](https://github.com/zmkfirmware/zmk/issues) for `ZMK_KSCAN_DIRECT_POLLING`
3. Consider reporting to [cormoran/zmk issues](https://github.com/cormoran/zmk/issues) if not already documented

## ✨ Expected Benefits (Once Working)

- ✅ **Bluetooth Keymap Editing**: No USB cable required for configuration changes
- ✅ **Better UI**: Improved ZMK Studio interface with DYA enhancements
- ✅ **Battery Insights**: Visualize battery consumption patterns over time
- ✅ **BLE Management**: Easier multi-device switching and pairing
- ✅ **Dynamic Trackball Tuning**: Adjust sensitivity without rebuilding firmware
- ✅ **Maintained Compatibility**: All JIS-SP features (JIS layout, trackball, split) preserved
