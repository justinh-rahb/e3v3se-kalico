# Kalico for Ender 3 V3 SE with Display Support

A fork of [Kalico](https://github.com/KalicoCrew/kalico) with integrated support for the stock Creality Ender 3 V3 SE display. This enables full use of the original touchscreen without any hardware modifications.

## Features

This fork combines the advanced features of Kalico (a community-maintained Klipper fork) with native display support for the Ender 3 V3 SE:

**Display Features:**
- File selection and print control from the stock display
- Print tuning (Z-offset, flow rate, speed)
- Pause/Resume/Stop controls
- Axis movement and homing
- Temperature management (nozzle and bed)
- Preheat profiles (PLA, TPU, custom)
- Motor disable
- Manual probe/leveling
- Custom macro buttons

**Multi-language Support:**
- Chinese, English, German, Russian, French, Turkish, Spanish, Italian, Portuguese, Japanese

## Requirements

- Ender 3 V3 SE with display firmware version 1.0.6
- Raspberry Pi or similar SBC running Klipper host software
- Moonraker and a web interface (Mainsail, Fluidd, etc.)

> **Note:** Creality firmware updates may alter display asset memory locations, which could affect compatibility with newer display firmware versions.

## Installation

### Building the Firmware

1. Clone this repository:
   ```bash
   git clone https://github.com/justinh-rahb/kalico-e3v3se.git ~/klipper
   ```

2. Clean and configure the build:
   ```bash
   cd ~/klipper
   make clean
   make menuconfig
   ```

3. Select the following options in menuconfig:
   - Micro-controller Architecture: **STMicroelectronics STM32**
   - Processor model: **STM32F103**
   - Bootloader offset: **28KiB bootloader**
   - Communication interface: **Serial (on USART1 PA10/PA9)**
   - Enable extra low-level configuration options: **[*] Enable**
     - Serial Bridge: **[*] Enable Serial Bridge**
       - Bridge USART: **USART2**

   > **Note:** If your board uses the GD32F303 chip, it is usually compatible with the STM32F103 setting.

4. Build the firmware:
   ```bash
   make
   ```

5. Copy `out/klipper.bin` to an SD card, renaming it to something unique (e.g., `klipper_kalico_v1.bin`). The printer won't flash if the filename matches the previous flash.

6. Insert the SD card into your printer and power cycle to flash. Wait approximately 10 seconds for the flash to complete.

### Installing Klipper Host Software

#### Option 1: Fresh Installation with KIAUH

If you haven't installed Klipper before, use [KIAUH](https://github.com/dw-0/kiauh) to install Klipper, Moonraker, and your preferred web interface.

To point KIAUH to this repository:

1. Configure KIAUH:
   ```bash
   cd ~/kiauh
   cp default.kiauh.cfg kiauh.cfg
   nano kiauh.cfg
   ```

2. Add the repository URL to your config:
   ```
   https://github.com/justinh-rahb/kalico-e3v3se, main
   ```

3. Use KIAUH's settings menu to switch Klipper source repository

#### Option 2: Switching from Existing Klipper

If you already have Klipper installed and want to switch to this fork:

1. Stop Klipper:
   ```bash
   sudo systemctl stop klipper
   ```

2. Backup your existing installation and switch:
   ```bash
   mv ~/klipper ~/klipper.bak
   git clone https://github.com/justinh-rahb/kalico-e3v3se.git ~/klipper
   ```

   Alternatively, use a symlink to keep both versions:
   ```bash
   mv ~/klipper ~/klipper.bak
   git clone https://github.com/justinh-rahb/kalico-e3v3se.git ~/klipper-e3v3se
   ln -s ~/klipper-e3v3se ~/klipper
   ```

3. Update the Python environment if needed:
   ```bash
   ~/klippy-env/bin/pip install -r ~/klipper/scripts/klippy-requirements.txt
   ```

4. Restart Klipper:
   ```bash
   sudo systemctl start klipper
   ```

## Configuration

Add the following section to your `printer.cfg`:

```ini
[e3v3se_display]
language: english
#logging: False
```

### Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `language` | `english` | Display language (chinese, english, german, russian, french, turkish, spanish, italian, portuguese, japanese) |
| `logging` | `False` | Enable debug logging for the display module |
| `update_interval` | `1.0` | Seconds between status refreshes. Raise it (e.g. `60`) to cut the periodic traffic on the display serial bridge. Knob, button and macro input stay event-driven and are unaffected. |

### Custom Macro Buttons

You can add custom buttons to the display's Misc menu by creating sub-sections:

```ini
[e3v3se_display macro1]
label: Calibrate
icon: 1
gcode: G28

[e3v3se_display macro2]
label: Level Bed
icon: 2
gcode: BED_MESH_CALIBRATE

[e3v3se_display macro3]
label: PID Tune
gcode: PID_CALIBRATE HEATER=extruder TARGET=200
```

#### Macro Options

| Option | Required | Description |
|--------|----------|-------------|
| `label` | No | Display text for the button (defaults to section name) |
| `icon` | No | Icon identifier for the button |
| `gcode` | Yes | G-code command(s) to execute |

## Supported Operations

| Feature | Status |
|---------|--------|
| File selection & printing | Supported |
| Print tuning | Supported |
| Pause/Resume/Stop | Supported |
| Axis movement & homing | Supported |
| Z-offset calibration | Supported |
| Temperature control | Supported |
| Preheat profiles | Supported |
| Motor disable | Supported |
| Manual probe | Supported |
| Custom macros | Supported |
| Speed adjustment | Not implemented |
| Acceleration adjustment | Not implemented |
| Steps/mm calibration | Not implemented |

## Upstream Projects

This project builds upon the work of several upstream projects:

- **[Kalico](https://github.com/KalicoCrew/kalico)** - Community-maintained Klipper fork with advanced features
- **[jpcurti/ender3-v3-se-klipper-with-display](https://github.com/jpcurti/ender3-v3-se-klipper-with-display)** - Original E3V3SE display implementation
- **[Klipper](https://github.com/Klipper3d/klipper)** - The original 3D printer firmware

## Kalico Features

This fork includes all Kalico features from the main branch. See the [Kalico documentation](https://docs.kalico.gg) for details on:

- MPC temperature control
- Velocity PID and PID profiles
- Dockable probe support
- Z calibration
- Independent X/Y acceleration and velocity
- Gcode shell commands
- And many more advanced features

## Verification

After installation, verify everything is working:

1. Check `klippy.log` for any errors related to `e3v3se_display`:
   ```bash
   grep e3v3se ~/printer_data/logs/klippy.log
   ```

2. The display should show the Klipper interface (not the Creality screensaver)

3. Test basic operations: temperature display, axis movement, file browsing

## Troubleshooting

**Display not responding:**
- Verify the firmware was flashed correctly (try a different filename)
- Check that `[e3v3se_display]` is in your printer.cfg
- Enable `logging: True` to debug serial communication
- Check `klippy.log` for errors related to `e3v3se_display`

**Wrong language displayed:**
- Ensure the `language` option matches one of the supported languages exactly (lowercase)

**Custom macros not appearing:**
- Verify the `gcode` option is set in each macro section
- Check Klipper logs for any configuration errors

**Firmware won't flash:**
- Rename the `.bin` file to something different from the previous flash
- Ensure the file is in the root of the SD card
- Try a different SD card (some cards have compatibility issues)

## License

This project is licensed under the GPL-3.0 License. See the [LICENSE](COPYING) file for details.
