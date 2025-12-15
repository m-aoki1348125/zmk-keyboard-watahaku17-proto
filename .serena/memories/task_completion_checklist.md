# Task Completion Checklist

When completing a task related to this ZMK firmware project, consider the following:

## Code Changes

### 1. Verify Syntax
- **DeviceTree files**: Check for syntax errors (unmatched braces, missing semicolons)
- **Keymap files**: Ensure all key positions are valid and behaviors are correct
- **C code**: Verify compilation (if building locally)

### 2. Test Build Locally (if applicable)
```bash
# Clean previous build
west build -t clean

# Build for the modified configuration
west build -b bmp_boost -s app -- -DSHIELD=watahaku_17_left -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y

# Check build output for errors/warnings
```

### 3. Update Both Sides (if needed)
- If changes affect hardware configuration, update both `*_left.overlay` and `*_right.overlay`
- If changes affect keymap, ensure layers are consistent

### 4. Verify Build Matrix (build.yaml)
- If adding new build configurations, add to `build.yaml`
- Ensure artifact names are descriptive

## Version Control

### 1. Check Git Status
```bash
git status
```

### 2. Review Changes
```bash
git diff
```

### 3. Stage Relevant Files
```bash
git add <files>
```

### 4. Commit with Descriptive Message
```bash
git commit -m "Brief description of changes"
```

### 5. Push to Remote (if ready)
```bash
git push
```

## GitHub Actions

### 1. Verify Workflow Triggers
- Check if `.github/workflows/build.yml` will be triggered by your changes
- Push/PR will automatically trigger builds

### 2. Monitor Build Status
- Check GitHub Actions tab for build results
- Download artifacts if needed for testing

## Documentation

### 1. Update CLAUDE.md (if architecture changes)
- Modify project instructions if significant changes were made
- Update build commands if new configurations added

### 2. Update README.md (if user-facing changes)
- Update keymap image if keymap changed (via keymap-drawer workflow)
- Document new features or configurations

### 3. Update Memory Files (if needed)
- If significant project structure changes, update Serena memory files

## Testing (for firmware changes)

### 1. Flash Firmware
- Flash to actual hardware if available
- Test both left and right sides if split-specific changes

### 2. Functional Testing
- **Keymap changes**: Test all modified keys and combos
- **Power management changes**: Verify sleep modes and wake behavior
- **Trackball changes**: Test movement and button functionality
- **BLE changes**: Test split connection stability

### 3. Battery Life (for power changes)
- Monitor battery consumption if power management was modified

## Common Gotchas

### 1. Keymap Location
- ✅ Modify: `config/keymap.keymap`
- ❌ Don't modify: `boards/shields/watahaku_17/watahaku_17.keymap`

### 2. West Workspace
- Ensure West workspace is up to date: `west update`
- Re-export if Zephyr version changed: `west zephyr-export`

### 3. DeviceTree Syntax
- Semicolons required after properties
- Matching braces for nodes
- Proper includes at top of file

### 4. Split Configuration
- Central side needs `CONFIG_ZMK_SPLIT_ROLE_CENTRAL=y`
- Peripheral side is default (no flag needed)
- Only one side should be central in a working configuration

## Not Required (No linting/formatting/testing commands)

This project does not have:
- Automated linting (no `west build -t lint` or similar)
- Code formatting tools (manual formatting following style guide)
- Unit tests (firmware testing is done on hardware)
- Pre-commit hooks

## Final Check

Before considering a task complete:

- [ ] Code compiles without errors (if built locally)
- [ ] Changes are consistent with project style
- [ ] Both sides updated if split-specific changes
- [ ] Git changes reviewed and committed (if appropriate)
- [ ] GitHub Actions build passes (if pushed)
- [ ] Documentation updated (if significant changes)
- [ ] Hardware tested (if critical firmware changes)
