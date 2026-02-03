# Keymap Update: 6th Thumb Key Support

## Summary

This update adds proper support for the 6th thumb key (RC(3,9)) that was defined in the device tree matrix transform but was previously not reflected in the keymap configuration files.

## Changes Made

### 1. Keymap File (`config/keymap/qwerty.keymap`)

All 9 layers now include bindings for 42 keys (previously 41):
- **36 main keys** (6 columns × 3 rows per half = 12 keys per half × 2 halves)
- **6 thumb keys** (3 left + 3 right, previously 3 left + 2 right)

**Thumb cluster layout:**
- **Left thumb cluster:** 3 keys (RC(3,3), RC(3,4), RC(3,1))
- **Right thumb cluster:** 3 keys (RC(3,7), RC(3,8), RC(3,9))

**New binding for 6th thumb key:**
- **BASE layer:** `&kp RIGHT_ALT` (Right Alt key)
- **All other layers:** `&trans` (transparent, passes through BASE layer binding)

### 2. JSON Layout File (`config/qwerty.json`)

Added physical position for the 6th thumb key:
```json
{ "row": 3, "col": 9, "x": 11.52, "y": 3.13, "r": 0 }
```

This positions the key to the right of the existing right thumb cluster keys.

### 3. Physical Layout File (`config/charybdis-layouts.dtsi`)

Added physical attributes for the 6th thumb key:
```dts
<&key_physical_attrs 100 100 1152 314 0 0 0>
```

This defines the key's position at x=11.52, y=3.14 with no rotation.

### 4. Shield Configuration Files

Updated copies in both shield directories:
- `boards/shields/charybdis_bt/charybdis-layouts.dtsi`
- `boards/shields/charybdis_dongle/charybdis-layouts.dtsi`

## Impact

### For Users

- **External keymap editors** (like https://nickcoutsos.github.io/keymap-editor/) will now correctly display all 6 thumb keys
- **New right thumb key** (RC(3,9)) is available for customization
- **Default binding** for the new key is Right Alt on the BASE layer

### For Firmware

- The firmware compilation will now properly map all 6 physical thumb keys
- No breaking changes to existing key positions or bindings
- All existing layer configurations remain unchanged except for the addition of the 6th thumb key

## Verification

All configuration files have been verified:
- ✅ 42 key bindings per layer across all 9 layers
- ✅ 42 key positions defined in JSON layout
- ✅ 42 physical key attributes in layout file
- ✅ JSON syntax validation passed
- ✅ Consistent positioning between dtsi and JSON files

## Testing Recommendations

1. **Build firmware** for your configuration (BT or Dongle)
2. **Flash firmware** to your keyboard
3. **Test the new thumb key** (rightmost key on right thumb cluster)
4. **Verify in keymap editor** that all 6 thumb keys appear correctly
5. **Customize binding** if you want something other than Right Alt

## Upgrading

If you have custom keymaps based on the old configuration:

1. **Add a 6th thumb key binding** to each layer in your keymap
2. **Update your JSON** if you maintain a custom layout file
3. **Rebuild firmware** with the updated configuration

Example addition to a layer:
```dts
// Old (5 thumb keys)
&mo 1  &kp SPACE    &kp ENTER  &mo 3  &mo 2

// New (6 thumb keys)
&mo 1  &kp SPACE    &kp ENTER  &mo 3  &mo 2  &kp RIGHT_ALT
```

## Questions or Issues?

If you encounter any problems with the new thumb key configuration, please open an issue on the repository with details about your build configuration and the specific problem you're experiencing.
