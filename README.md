# Charybdis Wireless Mini(Dongle+ZMK) - Assembly & Build Guide

[![.github/workflows/build.yml](https://github.com/280Zo/charybdis-wireless-mini-zmk-firmware/actions/workflows/build.yml/badge.svg)](https://github.com/280Zo/charybdis-wireless-mini-zmk-firmware/actions/workflows/build.yml)

This comprehensive guide covers the assembly of the **Charybdis Wireless Mini keyboard** with trackball support and the complete firmware building process for both **Bluetooth/USB** and **Dongle** configurations.
![main_photo](docs/photos/130825.jpg)
[Additional project photos](docs/photos/130825.jpg)
---

## Hardware Requirements

### Essential Components

| Component | Notes |
|-----------|-------|
| **nice!nano v2** | Microcontroller (2 units for split keyboard) |
| **MX Switch Sockets** | For mechanical switches |
| **Diodes** | 1N4148 signal diodes (1 per key) |
| **Stabilizers** | For longer keys (spacebar, etc.) |
| **PMW3610 Sensor** | Trackball sensor (right half) |
| **Mechanical Switches** | MX-compatible switches (42 keys: 36 main + 6 thumb) |
| **Keycaps** | Any standard MX keycaps |
| **USB-C Cable** | For flashing and programming |
| **Battery** | See [Battery Recommendations](#battery-recommendations) |

For full bill of materials and assembly instructions, refer to the [Charybdis Wireless Mini Build Guide](https://github.com/280Zo/charybdis-wireless-mini-3x6-build-guide).
Display can be ordered [here(non-touch version)](https://www.waveshare.com/1.69inch-touch-lcd-module.htm).

### Case Files

3D-printable case files are available in the `case/` directory:

- **Left half**: `left_case.stl`, `left_plate.stl`
- **Right half**: `right_case.stl`, `right_plate.stl`
- **Trackball mount**: Files in `case/trackball/`
- **Dongle case**: Files in `case/dongle/`

---

## Battery Recommendations

### Right Half Battery (Critical for Trackball)

⚠️ **IMPORTANT:** The PMW3610 trackball sensor consumes significant power. I **recommend using a 1000mAh** for the right half instead of a standard 130mAh battery.

#### Battery Comparison

| Capacity     | Runtime | Notes |
|--------------|---------|-------|
| **130mAh**   | ~2-3 days | Insufficient for trackball usage|
| **1000mAh**  | ~7-10 days | **Recommended** for trackball configurations |

#### Battery Selection Tips

1. **Form Factor**: Ensure the battery fits within the allocated space in your case
2. **Connector**: Use JST-PH 2.0 connectors for consistency
3. **Quality**: Choose reputable brands (e.g., Adafruit, Sparkfun)
4. **Voltage**: Should be 3.7V nominal (standard LiPo/LiPo)

#### Left Half Battery

The left half can use a standard **500mAh battery** since it doesn't power the trackball sensor.

### Installation Notes

- Route battery cables through designated paths to avoid pinching
- Use kapton tape to insulate battery connectors
- Secure batteries with double-sided tape or velcro
- Leave enough space for safe removal and replacement
- Never force batteries into tight spaces; modify the case if needed

---

## Building Firmware

### Option 1: Local Build (Recommended for Quick Testing)

The fastest way to build firmware locally is using the containerized Docker build system.

#### Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- This repository cloned on your machine

#### Build Commands

```bash
# Navigate to the local build directory
cd local-build

# Build all firmwares
docker-compose run --rm builder

# Build specific shield (e.g., charybdis_bt for Bluetooth only)
docker-compose run --rm builder -s charybdis_bt

# Build specific keymap (e.g., qwerty only)
docker-compose run --rm builder -k qwerty

# Enable USB logging for debugging
docker-compose run --rm builder -l true
```

#### Output

The built firmware files will be in `build_output/` organized by shield and keymap:

```
build_output/
├── charybdis_bt_qwerty/
│   ├── charybdis_left.uf2
│   └── charybdis_right.uf2
├── charybdis_dongle_qwerty/
│   ├── charybdis_left.uf2
│   ├── charybdis_right.uf2
│   └── charybdis_dongle.uf2
└── ...
```

See the [local build README](local-build/README.md) for more details, including USB logging and troubleshooting.

### Option 2: GitHub Actions (CI/CD Pipeline)

For automatic builds on every push:

1. **Fork or clone** this repository
2. **Push your changes** to GitHub
3. **GitHub Actions** automatically builds your firmware
4. **Download artifacts** from the Actions tab

#### Advantages

- No local toolchain setup required
- Builds run on GitHub servers
- Automatic artifact publishing
- Perfect for collaborative development

#### Disadvantages

- Slower feedback loop (~5-10 minutes per build)
- Requires GitHub account
- Limited to GitHub's CI/CD schedule

---

## Dongle + Trackball Configuration

### Overview

The dongle configuration allows you to use a **wireless dongle with USB connection**, supporting the **PMW3610 trackball sensor** for an all-in-one wireless input device.

### Components Required

- **Charybdis Dongle** (receiver with nice!nano v2)
- **Left and right keyboard halves**
- **PMW3610 trackball sensor** (mounted on right half)
- **USB-C cable** (for connection to computer)
- **Appropriate batteries** (see [Battery Recommendations](#battery-recommendations))

### Configuration Files

Dongle-specific configurations are in `boards/shields/charybdis_dongle/`:

- `charybdis_dongle.conf` - Dongle-specific settings
- `charybdis_dongle.overlay` - Pin definitions
- Keyboard halves use same files as Bluetooth version

### Trackball + Dongle Features

- **Full trackball support** with PMW3610 sensor
- **USB connection** to computer
- **Shared configuration** between Bluetooth and Dongle modes

### Building Dongle Firmware with Trackball

```bash
cd local-build

# Build dongle firmware with trackball support
docker-compose run --rm builder -s charybdis_dongle -k qwerty
```

### Trackball Behavior Configuration

All trackball parameters are configured in `config/charybdis_pointer.dtsi`:

```dts
// Current configuration in charybdis_pointer.dtsi
&trackball_listener {
    /* base processor chain on all layers */
    input-processors = <&zip_xy_scaler 5 5>;

    /* layer-7 scroll override */
    scroller: scroller {
        layers           = <7>;
        input-processors = <
            &zip_xy_scaler 1 15                              // Scales the scroll speed
            &zip_xy_to_scroll_mapper                         // Converts XY movement to scroll events
            &zip_scroll_transform (INPUT_TRANSFORM_Y_INVERT) // Inverts the scroll direction
        >;
    };

    /* layer-6 slow-pointer override */
    slow_pointer: slow_pointer {
        layers           = <6>;
        input-processors = <&zip_xy_scaler 2 6>;
    };
};
```

For detailed trackball customization, see [Modifying Trackball Behavior](#modifying-trackball-behavior) below.

### Dongle Display Support

The dongle configuration includes support for a **1.69" SPI display** to show status information. The `dongle_screen` shield is included in the dongle build (see `build.yaml`).

**Hardware:**
- 1.69" SPI display module ([non-touch version](https://www.waveshare.com/1.69inch-touch-lcd-module.htm))
- Wiring guide available in `docs/nice_nano_wire_guide.md`

**Configuration:**
- Display shield: `dongle_screen` (included in dongle build)
- Based on [zmk-dongle-screen](https://github.com/janpfischer/zmk-dongle-screen) repository

The display shows keyboard status, battery levels, active layer information, and connection status.

---

## Keyboard Layouts

### Primary Layout

This firmware uses **QWERTY** as the primary keyboard layout:

| Layout | File | Status | Notes |
|--------|------|--------|-------|
| **QWERTY** | `config/keymap/qwerty.keymap` | ✅ Primary | Standard English layout with 42 keys (36 main + 6 thumb) |

### Multi-Language Support

The QWERTY layout includes support for Polish and Russian characters through custom character definitions integrated into the keymap layers:

#### Polish Character Support

Polish characters are available through the BASE layer using custom key definitions:

- **Configuration**: `config/keys_pl.h` - Polish character definitions (already included in keymap)
- **Characters**: Ą, Ć, Ę, Ł, Ń, Ó, Ś, Ź, Ż and their uppercase variants
- **Usage**: Use RAlt (Right Alt) modifier with appropriate keys to access Polish characters
- **No OS switching required**: Works with Polish keyboard layout at OS level

#### Russian Character Support

Russian (Cyrillic) characters are available through dedicated keyboard layers:

- **Configuration**: `config/keys_ru.h` - Russian character definitions (already included in keymap)
- **Characters**: Full Cyrillic alphabet and common Russian symbols
- **Usage**: Switch your OS keyboard layout to Russian, then use the designated layer for Russian input
- **Layer integration**: Russian characters are accessible through the keymap layers

### Adding Custom Layouts

To add a new keyboard layout:

1. Create a new `.keymap` file in `config/keymap/`
2. Define your key mappings following ZMK syntax (42 keys per layer: 36 main + 6 thumb)
3. Reference any custom key definitions (e.g., `keys_pl.h`, `keys_ru.h`)
4. Update `build.yaml` to include your new layout in the build matrix
5. Build the firmware - GitHub Actions or local build will compile your layout

Example:

```bash
# Copy and modify the existing QWERTY keymap
cp config/keymap/qwerty.keymap config/keymap/my_layout.keymap

# Edit my_layout.keymap with your custom mappings
# Remember: Each layer needs 42 key bindings (36 main + 6 thumb)

# Add to build.yaml matrix
# - name: my_layout-bt
#   board: nice_nano_v2
#   keymap: my_layout
#   shields: [charybdis_left, charybdis_right]

# Build locally
cd local-build
docker-compose run --rm builder -k my_layout
```

---

## Flashing Firmware

### Prerequisites

- Downloaded firmware (`.uf2` files)
- USB-C cable
- One device at a time (you'll repeat for each half/dongle)

### Flashing Steps

1. **Unzip the firmware package**

   ```bash
   unzip charybdis-qwerty-dongle.zip
   ```

2. **Enter bootloader mode**

   - Plug the device into your computer via USB
   - **Double-press the reset button** on the nice!nano v2
   - The device should mount as `NICENANO` storage device

3. **Copy the firmware file**

   ```bash
   # On Windows
   copy charybdis_left.uf2 F:\  # Replace F:\ with your NICENANO drive

   # On macOS/Linux
   cp charybdis_left.uf2 /Volumes/NICENANO/
   ```

4. **Wait for programming**

   - The file will be copied
   - The device will automatically unmount
   - LED will flash indicating successful programming
   - The device will restart

5. **Repeat for other halves**

   - Repeat steps 2-4 for `charybdis_right.uf2`
   - If using dongle, also flash `charybdis_dongle.uf2`

### First-Time Setup

> ⚠️ **Important:** If flashing for the first time or switching between Bluetooth and Dongle configurations:

1. **Flash the reset firmware first** to all devices:

   ```bash
   # Available in firmwares/ directory
   # Flash settings_reset.uf2 to each device
   ```

2. Then proceed with normal firmware flashing

This clears any previous pairing information and ensures a clean state.

### Troubleshooting Flashing

- **Device not mounting**: Try a different USB port or cable
- **Still in bootloader**: Hold reset button while plugging in for 3 seconds
- **File copy fails**: Ensure the drive is properly mounted
- **Device won't restart**: Check the firmware file is not corrupted

---

## Modifying Trackball Behavior

The trackball uses ZMK's modular input processor system for customization.

### Configuration File

All trackball settings are in `config/charybdis_pointer.dtsi`.

### Current Configuration

The trackball is configured with three processor chains:

1. **Base (all layers)**: `&zip_xy_scaler 5 5` - Normal pointer movement
2. **SCROLL layer (7)**: Converts trackball movement to scroll events with inverted Y-axis
3. **SLOW layer (6)**: `&zip_xy_scaler 2 6` - Reduced pointer speed for precision work

### Common Adjustments

#### Adjusting Base Pointer Speed

```dts
// In charybdis_pointer.dtsi
&trackball_listener {
    // Change 5 5 to higher values for faster movement (e.g., 8 8)
    // Change to lower values for slower movement (e.g., 3 3)
    input-processors = <&zip_xy_scaler 5 5>;
```

#### Adjusting Scroll Speed

```dts
// In charybdis_pointer.dtsi - scroller configuration
scroller: scroller {
    layers           = <7>;
    input-processors = <
        &zip_xy_scaler 1 15    // Change 15 to higher for faster scroll
        &zip_xy_to_scroll_mapper
        &zip_scroll_transform (INPUT_TRANSFORM_Y_INVERT)
    >;
};
```

#### Adjusting Precision Mode Speed

```dts
// In charybdis_pointer.dtsi - slow_pointer configuration
slow_pointer: slow_pointer {
    layers           = <6>;
    input-processors = <&zip_xy_scaler 2 6>;  // Adjust 2 6 for different slow speeds
};
```

#### Changing Scroll Direction

```dts
// Remove Y_INVERT to change scroll direction
&zip_scroll_transform (INPUT_TRANSFORM_Y_INVERT)

// Or add X_INVERT for horizontal scroll inversion
&zip_scroll_transform (INPUT_TRANSFORM_X_INVERT)
```

### Building After Changes

After modifying `charybdis_pointer.dtsi`:

```bash
cd local-build
docker-compose run --rm builder
```

The changes will be compiled into the new firmware.

---

## Customizing Keymaps

### Using ZMK Studio (Bluetooth Only)

[ZMK Studio](https://zmk.studio/) allows real-time keymap editing on supported devices.

1. Build Bluetooth firmware
2. Pair keyboard to computer
3. Open [zmk.studio](https://zmk.studio/) in your browser
4. Connect to keyboard
5. Modify keymaps in real-time
6. Changes persist on the device

Dongle support is coming soon.

### Using Keymap Editor GUI

[nickcoutsos' keymap editor](https://nickcoutsos.github.io/keymap-editor/) provides a visual interface:

1. Fork/clone this repository
2. Open the keymap editor
3. Give it access to your GitHub repo
4. Select your branch
5. Modify keys visually
6. Save changes (auto-commits and triggers GitHub Actions)
7. Download built firmware

### Direct Keymap Editing

Edit `config/keymap/qwerty.keymap` (or your layout file) directly:

```dts
// Add layers, modify behaviors, change keycodes
// See ZMK documentation: https://zmk.dev/docs/features/keymaps
```

---

## Keyboard Layers
### Overview & Usage

![keymap base](keymap-drawer/base/qwerty.svg)

To see all the layers check out the [full render](keymap-drawer/qwerty.svg).

#### Layer Summary

This keyboard uses **9 layers** with **42 keys** per layer (36 main keys + 6 thumb keys):

| # | Layer       | Display Name | Purpose                                      |
|---|-------------|--------------|----------------------------------------------|
| 0 | **BASE**    | Rattus       | Standard typing with Polish character support |
| 1 | **NUM**     | Numbers      | Numbers + function keys                      |
| 2 | **NAV**     | Media        | Media controls and navigation                |
| 3 | **MOUSE**   | bong         | Mouse movement, scrolling, and navigation    |
| 4 | **SYM**     | Sym          | Symbols and punctuation                      |
| 5 | **EXTRAS**  | Xtra         | Shortcuts, Bluetooth controls                |
| 6 | **SLOW**    | Slow         | Low-speed pointer mode                       |
| 7 | **SCROLL**  | Scroll       | Vertical/horizontal scrolling and navigation |
| 8 | **FUNC**    | Functional   | Function keys (F1-F12)                       |

#### 6-Key Thumb Clusters

This keyboard features **6 thumb keys total** (3 per side):

**Left Thumb Cluster (3 keys):**
- K36: Left GUI (Win/Cmd)
- K37: Layer 1 (NUM) hold
- K38: Space

**Right Thumb Cluster (3 keys):**
- K39: Enter
- K40: Layer 3 (MOUSE) hold  
- K41: Right Alt

The 6th thumb key (K41, Right Alt) provides easy access to Polish characters and other AltGr-based symbols when using appropriate OS keyboard layouts.

🔗 Combos
| Trigger Keys              | Result                                 |
| ------------------------- | -------------------------------------- |
| `K17 + K18`               | **Caps Word** (one-shot words in CAPS) |
| `K25 + K26`               | **Left Click**                         |
| `K26 + K27`               | **Middle Click**                       |
| `K27 + K28`               | **Right Click**                        |
| `K13 + K22`               | Toggle **MOUSE** layer                 |
| `K38 + K39` (thumb keys)  | Layer-swap **BASE ⇄ EXTRAS**           |


⚙️ Other Highlights
- **42-key layout:** 36 main keys + 6 thumb keys (3 per side) for ergonomic typing
- **Layer-tap on X key:** Hold X (K26) to access SCROLL layer (7) for trackball scrolling
- **Thumb-scroll mode:** Access SCROLL layer (7) to convert trackball motion into scroll events
- **Precision cursor mode (SLOW layer):** Accessible via layer 6 for detailed cursor work with reduced pointer speed
- **K37 - Layer 1 (NUM) Access:**
    - Hold: Activates NUM layer for numbers and symbols
- **K38 - Space:**
    - Tap: Space
- **K40 - Layer 3 (MOUSE) Access:**
    - Hold: Activates MOUSE layer for navigation and mouse controls
- **Bluetooth profile quick-swap:** Jump to the EXTRAS layer (5) and tap the dedicated BT-select keys to pair or switch among up to four saved hosts (plus BT CLR to forget all)
- **PMW3610 low power trackball sensor driver:** Provided by [badjeff](https://github.com/badjeff/zmk-pmw3610-driver)
    - Patched to remove build warnings and prevent cursor jump on wake
- **Modifier keys readily accessible:** Left Shift, Left Ctrl on home row; Left GUI/Win and Right Alt on thumb keys
- **ZMK Studio:** Supported on BT builds for real-time keymap adjustments. Dongle support coming soon.

---

## Troubleshooting

### Connection Issues

**Problem:** Keyboard halves aren't connecting (Bluetooth)

**Solutions:**
1. Press reset button on both halves simultaneously
2. Follow [ZMK connection troubleshooting](https://zmk.dev/docs/troubleshooting/connection-issues#acquiring-a-reset-uf2)
3. Flash reset firmware to both halves
4. Re-pair from Bluetooth settings

### Trackball Issues

**Problem:** Trackball not moving or erratic movement

**Solutions:**
1. Verify PMW3610 sensor is properly soldered
2. Check pin connections with multimeter
3. Adjust pointer speed in `charybdis_pointer.dtsi`
4. Verify I2C/SPI lines for interference
5. Test with a different sensor if available

**Problem:** Trackball drains battery quickly

**Solutions:**
1. Use 1000mAh or larger battery (see [Battery Recommendations](#battery-recommendations))
2. Lower pointer speed to reduce sensor polling
3. Enable low-power mode if available in firmware
4. Check for sensor firmware updates

### Firmware Issues

**Problem:** Firmware won't flash

**Solutions:**
1. Try different USB port or cable
2. Verify device is in bootloader mode (NICENANO should appear)
3. Check firmware file integrity
4. Update bootloader if needed

**Problem:** Keys not responding

**Solutions:**
1. Test switches individually
2. Verify diodes are installed with correct polarity
3. Check for cold solder joints with a magnifying glass
4. Test with continuity meter
5. Rebuild firmware and re-flash

### Layout Issues

**Problem:** Polish/Russian characters not working

**Solutions:**
1. Ensure you included the correct character definitions file
2. Check keymap is properly configured
3. Verify your system keyboard layout supports these languages
4. Test individual key codes

---

## Credits & References

This project builds upon the excellent work of the ZMK community and specialized contributors:

### Original Projects & Authors

#### Charybdis Keyboard Hardware
- **[280Zo/charybdis-wireless-mini-3x6-build-guide](https://github.com/280Zo/charybdis-wireless-mini-3x6-build-guide)** - Wireless Charybdis keyboard design and assembly guide

#### ZMK Firmware
- **[ZMK Firmware Project](https://zmk.dev/)** - Modern open-source keyboard firmware for wireless input devices
- **[janpfischer/zmk-dongle-screen](https://github.com/janpfischer/zmk-dongle-screen)** - ZMK dongle with display support and configuration examples
- **[nickcoutsos/keymap-editor](https://github.com/nickcoutsos/keymap-editor)** - Visual keymap editor for ZMK
- **[caksoylar/keymap-drawer](https://github.com/caksoylar/keymap-drawer)** - Keymap visualization tool
- **[badjeff/zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver)** - PMW3610 trackball sensor driver for ZMK
- **[urob/zmk-config](https://github.com/urob/zmk-config)** - Timeless home-row mods implementation
- **[eigatech](https://github.com/eigatech)** - Community contributions

---

## Additional Resources

- **[ZMK Documentation](https://zmk.dev/docs/)** - Official ZMK guide and API reference
- **[PMW3610 Sensor Specs](https://www.pixart.com.tw/)** - Sensor specifications

---

## Support & Issues

- **Found a bug?** Open an issue on the [main repository](https://github.com/280Zo/charybdis-wireless-mini-zmk-firmware)
- **Need help?** Check the troubleshooting section or ask in the ZMK community
- **Have improvements?** Submit a pull request!

---

