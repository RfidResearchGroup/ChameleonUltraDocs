# CLI

The CLI (**C**ommand **L**ine **I**nterface) is the official way to control your Chameleon.

It requires at least **Python 3.9** version.

## Installing

There are multiple ways to install the CLI, depending on your OS.

### Windows

Windows users have the choice of 4 options:

#### ProxSpace

Using ProxSpace to build the CLI is the easiest and most comfortable way to get started.

1. Download ProxSpace from the [official GitHub](https://github.com/Gator96100/ProxSpace/releases/latest)

2. [Download 7zip](https://www.7-zip.org/) to extract the archive

3. Install 7zip by double clicking the Installer and clicking `Install`

4. Right-click on the downloaded archive and select `7zip -> Unpack to "ProxSpace"`

5. Open a terminal in the proxspace folder. If you are on a new Windows install, you should be able to just right-click and select `Open in Terminal`. If that option is not visible and the ProxSpace folder is still in your downloads folder, press `win+r` and type `powershell` followed by enter. In Powershell now type `cd ~/Downloads/ProxSpace`

6. Run the command `.\runme64.bat`. After successful completion, you should be dropped to the `pm3 ~ $` shell.

7. Clone the Repository by typing `git clone https://github.com/RfidResearchGroup/ChameleonUltra.git`

8. Now go into the newly created folder with `cd ChameleonUltra/software/src`

9. Prepare for package installation with `pacman-key --init; pacman-key --populate; pacman -S msys2-keyring --noconfirm; pacman-key --refresh`

10. Proceed by installing Ninja with `pacman -S ninja --noconfirm`

11. Build the required config by running `cmake .`

12. And the binaries with `cmake --build .`

13. Go into the script folder with `cd ~/ChameleonUltra/software/script/`

14. Install python requirements with `pip install -r requirements.txt`

15. Finally run the CLI with `python chameleon_cli_main.py`

To use after installing, just do the following:

1. Run `runme64.bat`

2. Go into the script folder with `cd ~/ChameleonUltra/software/script/`

3. Run the CLI with `python chameleon_cli_main.py`

#### WSL2

Coming Soon

#### WSL1

Coming Soon

#### Build Natively

Building natively is a bit more advanced and not recommended for beginners

1. Download and install [Visual Studio Community](https://visualstudio.microsoft.com/de/downloads/)

2. On the workload selection screen, choose the `Desktop development with C++` workload. Click `Download and Install`

3. Download and install [git](https://git-scm.com/download). When asked, add to your path

4. Download and install [cmake](https://cmake.org/download/). Again, when asked, add to your path

5. Download and install [python](https://www.python.org/downloads/). When asked, add to your path (small checkbox in the bottom left). Python 3.9 or above is required.

6. Choose a suitable location and open a terminal. Clone the repository with `git clone https://github.com/RfidResearchGroup/ChameleonUltra.git`

7. Change into the binaries folder with `cd ChameleonUltra/software/src`

8. Build the required config by running `cmake .`

9. And the binaries with `cmake --build .`

10. Copy the binaries by running `cp -r ../bin/Debug/* ../script/`

11. Go into the script folder with `cd ../script/`

12. Create a python virtual environment with `python -m venv venv`

13. Activate it by running `.\venv\Scripts\Activate.ps1`

14. Install python requirements with `pip install -r requirements.txt`

15. Finally run the CLI with `python chameleon_cli_main.py`

To run again after installing, just do the following:

1. Activate venv by running `.\venv\Scripts\Activate.ps1`

2. Run the CLI with `python chameleon_cli_main.py`

### MacOS

Requires [Homebrew](https://brew.sh/) to be installed.
  - If you don't have Homebrew installed on your macOS, open the Terminal and run: 
  `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

See Linux/Macos instructions below for the rest.

### Linux / MacOS

Install the dependencies
  - Ubuntu / Debian:  
  `sudo apt install git cmake build-essential python3-venv`
  - Arch:  
  `sudo pacman -S  git cmake base-devel python3`
  - MacOS:
  `brew install git cmake python3`

Python 3.9 or above is required.

Run the following script to clone the Repository, compile the tools and install Python dependencies in a virtual environment.

```sh
#!/bin/bash

git clone https://github.com/RfidResearchGroup/ChameleonUltra.git
(
  cd ChameleonUltra/software/src
  mkdir -p out
  (
    cd out
    cmake ..
    cmake --build . --config Release
  )
)
(
  cd ChameleonUltra/software/script
  python3 -m venv venv
  source venv/bin/activate
  pip3 install -r requirements.txt
  deactivate
)
```

To run the client after installing, do the following:

```sh
cd ChameleonUltra/software/script
source venv/bin/activate
python3 chameleon_cli_main.py
deactivate
```

## Usage

When in the CLI, plug in your Chameleon and connect with `hw connect`. If autodetection fails, get the Serial Port used by your Chameleon and run `hw connect -p COM11` (Replace `COM11` with your serial port, on Linux it may be `/dev/ttyACM0`)

### MFKEY32v2 walk-through
Make sure to be in the `software/` directory and run the Python CLI from there.

```sh
# Connect to the CLI
hw connect
# Check which slot can be used
hw slot list
# Change the slot type, here using slot 8 for a MFC 1k emulation
hw slot type -s 8 -t MIFARE_1024
# Init the slot content
hw slot init -s 8 -t MIFARE_1024
# or load an existing dump and set UID and anticollision data,
# cf 'hf mf eload' and 'hf mf econfig'
# Enable the slot
hw slot enable -s 8 --hf
# Change to the new slot
hw slot change -s 8
# Activate the authentication logs
hf mf econfig --enable-log
```
Now disconnect, go to a reader and swipe it a few times

Come back

```sh
# connect to the CLI
hw connect
# See if nonces were collected. We need 2 nonces per key to recover
hf mf elog
# Recover the key(s) based on the collected nonces
hf mf elog --decrypt
# Clean the logged detection nonces
hf mf econfig --disable-log
```
  Output example:
```
 - MF1 detection log count = 6, start download.
 - Download done (144bytes), start parse and decrypt
 - Detection log for uid [DEADBEEF]
  > Block 0 detect log decrypting...
  > Block 1 detect log decrypting...
  > Result ---------------------------
  > Block 0, A key result: ['a0a1a2a3a4a5', 'aabbccddeeff']
  > Block 1, A key result: ['010203040506']

```

---

## Command Reference

This section documents all available CLI commands. Use `-h` or `--help` with any command to see full usage details.

### Command Hierarchy Overview

```
root
├── clear              - Clear screen
├── rem                - Add timestamped comment
├── exit               - Exit CLI
├── dump_help          - Show all commands
├── hw                 - Hardware commands
│   ├── connect        - Connect to device
│   ├── disconnect     - Disconnect device
│   ├── mode           - Get/set device mode
│   ├── chipid         - Get chip ID
│   ├── address        - Get BLE address
│   ├── version        - Get firmware version
│   ├── dfu            - Enter DFU mode
│   ├── factory_reset  - Factory reset
│   ├── battery        - Battery status
│   ├── raw            - Send raw command
│   ├── slot           - Slot management
│   └── settings       - Device settings
├── hf                 - High frequency commands
│   ├── 14a            - ISO14443-A commands
│   ├── mf             - MIFARE Classic commands
│   └── mfu            - MIFARE Ultralight/NTAG commands
└── lf                 - Low frequency commands
    ├── em             - EM4x commands
    ├── hid            - HID Prox commands
    └── viking         - Viking commands
```

---

### Root Commands

These are basic utility commands available at the root level of the CLI.

#### `clear`

Clear the terminal screen. This removes all previous output from the terminal window, giving you a clean workspace.

**When to use:** When your terminal gets cluttered with output and you want a fresh view.

```sh
clear
```

#### `rem`

Add a timestamped remark or comment to the CLI output. This is useful for documenting your session, especially when saving output to a log file for later review. Each remark is prefixed with a timestamp showing when it was added.

**When to use:** During card analysis sessions to annotate your findings, or when logging a session for documentation purposes.

```sh
# Add a simple note
rem Starting card analysis

# Document findings
rem Found key A for sector 0: FFFFFFFFFFFF
rem Card appears to be standard MIFARE Classic 1K

# Mark session milestones
rem === Beginning nested attack ===
```

#### `exit`

Exit the CLI application and return to your system shell. Any unsaved emulator data will be preserved in the device's RAM until power loss or reset.

**Important:** Make sure to run `hw slot store` before exiting if you want to permanently save any emulator changes to flash memory.

```sh
exit
```

#### `dump_help`

Display a comprehensive list of all available commands in the CLI. This is your go-to command when you need to discover available functionality or remember command names.

| Option | Description |
|--------|-------------|
| `-d`, `--show-desc` | Include a brief description for each command, explaining what it does |
| `-g`, `--show-groups` | Organize commands by their category (hw, hf, lf, etc.) |

**When to use:** When you're unsure what commands are available, or need to find a specific command by browsing.

```sh
# List all commands (names only)
dump_help

# List with descriptions - recommended for learning
dump_help -d

# List organized by category
dump_help -g

# Combine both options for complete reference
dump_help -d -g
```

---

### HW (Hardware) Commands

Hardware commands control the Chameleon device itself - connection, configuration, battery status, and device settings. These commands affect the device's operation rather than interacting with external cards.

#### `hw connect`

Establish a connection to a Chameleon device via USB serial port. This is typically the first command you'll run when starting the CLI. The device must be connected via USB for serial communication (Bluetooth connections use a different workflow).

The CLI will auto-detect the Chameleon if no port is specified. On Windows, ports appear as `COM1`, `COM2`, etc. On Linux/Mac, they appear as `/dev/ttyACM0`, `/dev/ttyUSB0`, etc.

| Option | Description |
|--------|-------------|
| `-p`, `--port` | Serial port path. If not specified, the CLI will scan and auto-detect |

**Troubleshooting:**
- If auto-detect fails, check `Device Manager` (Windows) or `ls /dev/tty*` (Linux) to find the port
- Linux users may need to add themselves to the `dialout` group: `sudo usermod -a -G dialout $USER`
- Make sure no other application (like a serial terminal) is using the port

```sh
# Auto-detect and connect (most common usage)
hw connect

# Connect to specific port on Windows
hw connect -p COM3

# Connect to specific port on Linux
hw connect -p /dev/ttyACM0

# Connect to specific port on Mac
hw connect -p /dev/tty.usbmodem14101
```

Output on success:
```
 - Chameleon connected: COM3
 - Firmware version: v2.0.0
```

#### `hw disconnect`

Disconnect from the currently connected Chameleon device. This releases the serial port so other applications can use it. The device continues running normally after disconnection.

**When to use:** When you need to disconnect without closing the CLI, or before physically unplugging the device.

```sh
hw disconnect
```

#### `hw mode`

Get or change the device's operating mode. The Chameleon has two primary modes:

- **Reader mode (`reader`)**: The device acts as an NFC/RFID reader, allowing you to scan, read, and attack cards placed on or near it. Use this mode for card analysis.
- **Tag/Emulator mode (`tag`)**: The device emulates a card using data stored in one of its 8 slots. Use this mode to present the device to readers (e.g., for access control, sniffing reader authentication).

| Option | Description |
|--------|-------------|
| `-m`, `--mode` | Mode to set: `reader` (scan cards) or `tag` (emulate cards) |

**Important:** After sniffing a reader (in tag mode), you must switch back to reader mode to use commands like `hf mf elog --decrypt`.

```sh
# Check current mode
hw mode

# Switch to reader mode (for scanning/reading cards)
hw mode -m reader

# Switch to tag emulator mode (to present to readers)
hw mode -m tag
```

Output:
```
 - Current mode: Reader
```

#### `hw chipid`

Retrieve the unique chip ID of the device's NRF52840 microcontroller. This ID is factory-programmed and unique to each device, useful for identification or support purposes.

```sh
hw chipid
```

Output:
```
 - Chip ID: DEADBEEF12345678
```

#### `hw address`

Get the device's Bluetooth Low Energy (BLE) MAC address. This address is used when connecting to the Chameleon via Bluetooth from mobile apps or other BLE clients.

**When to use:** When setting up Bluetooth connectivity or troubleshooting BLE pairing issues.

```sh
hw address
```

Output:
```
 - BLE Address: AA:BB:CC:DD:EE:FF
```

#### `hw version`

Display the currently installed firmware version. This helps determine if your device is up to date and is useful information when reporting bugs or asking for support.

```sh
hw version
```

Output:
```
 - Firmware version: v2.0.0
 - Git commit: abc1234
```

#### `hw dfu`

Restart the device into DFU (Device Firmware Update) bootloader mode. In this mode, the device can receive firmware updates via USB or BLE. The device will disconnect from the CLI and enumerate as a DFU device.

**Warning:** After entering DFU mode, you'll need to flash new firmware or power-cycle the device to return to normal operation.

**When to use:** When updating firmware using nRF Connect, dfu-util, or other DFU tools.

```sh
hw dfu
```

The device will disconnect and its LED will indicate DFU mode.

#### `hw factory_reset`

Perform a complete factory reset, erasing all slot data, settings, and returning the device to its out-of-box state. This includes:
- All 8 emulator slots cleared
- All saved keys and dumps erased
- Settings reset to defaults
- BLE bonds cleared

| Option | Description |
|--------|-------------|
| `-y`, `--yes` | Skip the confirmation prompt (use with caution) |

**Warning:** This operation is irreversible. All data stored on the device will be permanently lost.

**When to use:** When preparing to sell/give away the device, or when troubleshooting persistent issues.

```sh
# Reset with confirmation prompt (recommended)
hw factory_reset

# Reset immediately without confirmation (scripting)
hw factory_reset -y
```

#### `hw battery`

Display the current battery voltage and estimated charge level. The Chameleon Ultra has a built-in LiPo battery that charges via USB-C.

**Interpreting results:**
- **Voltage:** Typically 3.0V (empty) to 4.2V (full)
- **Level:** Percentage estimate based on voltage

```sh
hw battery
```

Output:
```
 - Voltage: 4.12V
 - Level: 85%
```

#### `hw raw`

Send a raw command directly to the device firmware. This is an advanced debugging command for developers or users with deep knowledge of the Chameleon's protocol. Most users will never need this.

| Option | Description |
|--------|-------------|
| `-c`, `--cmd` | Command code in hexadecimal |
| `-d`, `--data` | Data payload in hexadecimal |
| `-t`, `--timeout` | Response timeout in milliseconds |
| `-b`, `--bitlen` | Bit length for commands that require it |

**Warning:** Sending incorrect raw commands could potentially crash the firmware or cause unexpected behavior.

```sh
# Example: Send command 0x03E8 with data 0x00
hw raw -c 03E8 -d 00 -t 1000
```

---

### HW Slot Commands

The Chameleon Ultra has **8 emulation slots**, numbered 1-8. Each slot can store both a High Frequency (HF) tag and a Low Frequency (LF) tag simultaneously. Think of slots like memory banks - you can configure different cards in each slot and switch between them using the physical buttons or CLI commands.

**Key concepts:**
- Each slot has an HF side and an LF side that operate independently
- Slots must be **initialized** with a tag type before use
- Slots must be **enabled** to be active during emulation
- Only one slot can be active at a time (the "current" slot)
- Use `hw slot store` to permanently save changes to flash memory

#### `hw slot list`

Display the status of all 8 slots, showing which tag types are configured and whether each side is enabled or disabled. This is your main command for understanding what's stored in the device.

| Option | Description |
|--------|-------------|
| `-e`, `--extend` | Show extended information including UIDs, nicknames, and detailed configuration |

**When to use:** To see which slots are available, what's configured, and plan your card organization.

```sh
# Basic slot overview
hw slot list

# Detailed view with UIDs and nicknames
hw slot list -e
```

Output example:
```
 - Slot 1 | HF: MIFARE_1024 [Enabled] | LF: EM410X [Enabled]
 - Slot 2 | HF: NTAG_215 [Enabled] | LF: Unknown [Disabled]
 - Slot 3 | HF: Unknown [Disabled] | LF: Unknown [Disabled]
 - Slot 4 | HF: Unknown [Disabled] | LF: Unknown [Disabled]
 - Slot 5 | HF: Unknown [Disabled] | LF: Unknown [Disabled]
 - Slot 6 | HF: Unknown [Disabled] | LF: Unknown [Disabled]
 - Slot 7 | HF: Unknown [Disabled] | LF: Unknown [Disabled]
 - Slot 8 | HF: MIFARE_1024 [Enabled] | LF: Unknown [Disabled]
```

Extended output includes:
```
 - Slot 1 | Nick: "Office Door"
   HF: MIFARE_1024 [Enabled] UID: DEADBEEF
   LF: EM410X [Enabled] ID: 1A2B3C4D5E
```

#### `hw slot change`

Switch the currently active slot. The active slot is what the Chameleon will emulate when in tag mode. You can also change slots using the physical buttons on the device.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot number (1-8) **required** |

**Tip:** The device LEDs indicate the current slot number in binary.

```sh
# Switch to slot 1
hw slot change -s 1

# Switch to slot 3
hw slot change -s 3

# Switch to slot 8
hw slot change -s 8
```

#### `hw slot type`

Set the tag type for a slot. This defines what kind of card the slot will emulate. You must set the type before loading data or using the slot for emulation.

**Important:** Setting the type initializes the slot's memory structure for that card type. If you change the type, existing data may be lost.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot number (1-8) **required** |
| `-t`, `--type` | Tag type identifier **required** |

**Supported HF (13.56 MHz) types:**

| Type | Description | Memory Size |
|------|-------------|-------------|
| `MIFARE_Mini` | MIFARE Classic Mini / S20 | 320 bytes (5 sectors) |
| `MIFARE_1024` | MIFARE Classic 1K / S50 | 1KB (16 sectors) |
| `MIFARE_2048` | MIFARE Classic 2K | 2KB (32 sectors) |
| `MIFARE_4096` | MIFARE Classic 4K / S70 | 4KB (40 sectors) |
| `NTAG_213` | NTAG213 | 144 bytes user memory |
| `NTAG_215` | NTAG215 (amiibo compatible) | 504 bytes user memory |
| `NTAG_216` | NTAG216 | 888 bytes user memory |
| `MF0ICU1` | MIFARE Ultralight | 64 bytes |
| `MF0ICU2` | MIFARE Ultralight C | 192 bytes (with 3DES) |
| `MF0UL11` | MIFARE Ultralight EV1 | 48 bytes user memory |
| `MF0UL21` | MIFARE Ultralight EV1 | 128 bytes user memory |

**Supported LF (125 kHz) types:**

| Type | Description |
|------|-------------|
| `EM410X` | EM4100/EM4102 - Most common LF card |
| `HIDPROX` | HID Proximity cards (26-bit, 35-bit, etc.) |
| `Viking` | Viking access cards |

```sh
# Set slot 1 to MIFARE Classic 1K (most common card type)
hw slot type -s 1 -t MIFARE_1024

# Set slot 2 to NTAG215 (for amiibo cloning)
hw slot type -s 2 -t NTAG_215

# Set slot 3 to MIFARE Classic 4K
hw slot type -s 3 -t MIFARE_4096

# Set slot 4 to EM410X for LF emulation
hw slot type -s 4 -t EM410X

# Set slot 5 to HID Proximity
hw slot type -s 5 -t HIDPROX
```

#### `hw slot delete`

Erase tag data from a slot, resetting it to an unconfigured state. The slot will show as "Unknown" after deletion. Use this to clear sensitive data or start fresh with a slot.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot number (1-8) **required** |
| `--hf` | Delete the HF (13.56 MHz) tag data |
| `--lf` | Delete the LF (125 kHz) tag data |

**Note:** You must specify `--hf` and/or `--lf` to indicate which side to delete.

```sh
# Delete HF data from slot 1
hw slot delete -s 1 --hf

# Delete LF data from slot 2
hw slot delete -s 2 --lf

# Delete both HF and LF from slot 3
hw slot delete -s 3 --hf --lf
```

#### `hw slot init`

Initialize a slot with default factory data for the specified tag type. This creates a blank card with default UID, keys, and empty data blocks. Use this as a starting point before customizing the emulated card.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot number (1-8) **required** |
| `-t`, `--type` | Tag type to initialize **required** |

**What gets initialized:**
- Default UID (random or factory default)
- Default keys (usually `FFFFFFFFFFFF` for MIFARE Classic)
- Empty data blocks
- Default access bits

```sh
# Initialize slot 1 as a blank MIFARE Classic 1K
hw slot init -s 1 -t MIFARE_1024

# Initialize slot 2 as a blank NTAG215
hw slot init -s 2 -t NTAG_215

# Initialize slot 3 as blank EM410X
hw slot init -s 3 -t EM410X
```

#### `hw slot enable`

Enable a slot for emulation. A disabled slot will not respond when the device is in tag mode, even if it contains valid data. Enable only the slots you need to avoid confusion.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot number (1-8) **required** |
| `--hf` | Enable the HF (13.56 MHz) side |
| `--lf` | Enable the LF (125 kHz) side |

**Note:** You can enable HF, LF, or both independently on the same slot.

```sh
# Enable HF emulation for slot 1
hw slot enable -s 1 --hf

# Enable LF emulation for slot 2
hw slot enable -s 2 --lf

# Enable both HF and LF for slot 3 (dual-frequency emulation)
hw slot enable -s 3 --hf --lf
```

#### `hw slot disable`

Disable a slot, preventing it from responding during emulation. The data remains stored but the slot becomes inactive. Useful for temporarily hiding a card without deleting it.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot number (1-8) **required** |
| `--hf` | Disable the HF (13.56 MHz) side |
| `--lf` | Disable the LF (125 kHz) side |

```sh
# Disable HF on slot 1
hw slot disable -s 1 --hf

# Disable LF on slot 2
hw slot disable -s 2 --lf
```

#### `hw slot nick`

Manage nicknames for slots. Nicknames are human-readable labels that help you identify what card is stored in each slot (e.g., "Office Door", "Gym Locker", "Test Card").

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot number (1-8) |
| `--hf` | Apply to HF side |
| `--lf` | Apply to LF side |
| `-n`, `--name` | Nickname text (max ~32 characters) |
| `-d`, `--delete` | Delete the nickname |

**Tip:** Nicknames appear in `hw slot list -e` and in some mobile apps.

```sh
# Set a nickname for the HF card in slot 1
hw slot nick -s 1 --hf -n "Office Main Door"

# Set a nickname for the LF card in slot 2
hw slot nick -s 2 --lf -n "Parking Garage"

# View current nickname
hw slot nick -s 1 --hf

# Delete a nickname
hw slot nick -s 1 --hf -d
```

#### `hw slot store`

**Permanently save all slot configurations and data to flash memory.** Changes made to slots (type, data, keys, nicknames) are initially held in RAM. If the device loses power before running this command, changes will be lost.

**When to use:** After making any changes you want to keep permanently - loading dumps, changing UIDs, modifying keys, etc.

**Important:** Always run this command after finishing your slot configuration!

```sh
hw slot store
```

Output:
```
 - Slot data saved to flash.
```

#### `hw slot openall`

Initialize all 8 slots with default configurations and enable them. This is a quick way to set up the device for testing or reset all slots without doing a full factory reset.

**Warning:** This will overwrite any existing slot data!

```sh
hw slot openall
```

---

### HW Settings Commands

Device configuration commands that control LED behavior, Bluetooth settings, and button functions. These settings persist across power cycles when saved to flash.

#### `hw settings animation`

Configure the LED animation mode. The Chameleon has RGB LEDs that indicate status, slot number, and activity. You can reduce or disable animations to save power or for stealth.

| Option | Description |
|--------|-------------|
| `-m`, `--mode` | Animation mode: `FULL`, `MINIMAL`, or `NONE` |

**Animation modes:**
- **FULL**: All animations enabled - slot indicators, activity flashes, charging animation, etc.
- **MINIMAL**: Reduced animations - only essential status indicators
- **NONE**: LEDs disabled completely (stealth mode, maximum battery life)

```sh
# Check current animation mode
hw settings animation

# Enable full animations (default)
hw settings animation -m FULL

# Reduce to minimal animations
hw settings animation -m MINIMAL

# Disable all animations (stealth mode)
hw settings animation -m NONE
```

#### `hw settings bleclearbonds`

Clear all stored Bluetooth bonding information. Bonding is the process of pairing with a phone or computer over BLE. If you're having connection issues or want to remove all previously paired devices, use this command.

| Option | Description |
|--------|-------------|
| `-y`, `--yes` | Skip confirmation prompt |

**When to use:** 
- When Bluetooth connections fail consistently
- Before giving the device to someone else
- When changing phones/computers

```sh
# Clear bonds with confirmation
hw settings bleclearbonds

# Clear bonds immediately
hw settings bleclearbonds -y
```

#### `hw settings store`

Save the current settings to flash memory. Like `hw slot store` but for device settings (animation mode, button configuration, BLE settings, etc.).

**Important:** Settings changes are lost on power-off unless you run this command!

```sh
hw settings store
```

#### `hw settings reset`

Reset all settings to factory defaults. This does NOT affect slot data - only device settings like animation mode, button configuration, and BLE settings.

| Option | Description |
|--------|-------------|
| `-y`, `--yes` | Skip confirmation prompt |

```sh
# Reset with confirmation
hw settings reset

# Reset immediately
hw settings reset -y
```

#### `hw settings btnpress`

Configure what happens when you press the physical buttons on the Chameleon. Each button (A and B) can have different actions for short press vs. long press.

| Option | Description |
|--------|-------------|
| `-b`, `--button` | Which button: `a` or `b` |
| `-s`, `--short` | Configure the short press action |
| `-l`, `--long` | Configure the long press action |
| `-f`, `--function` | Function to assign |

**Available functions:**

| Function | Description |
|----------|-------------|
| `NONE` | Button does nothing |
| `FORWARD` | Switch to next slot (1→2→3→...→8→1) |
| `BACKWARD` | Switch to previous slot (8→7→6→...→1→8) |
| `CLONE_IC` | Instantly clone the HF card currently on the reader to the current slot |
| `CLONE_ID` | Instantly clone the LF card currently on the reader to the current slot |

**Default configuration:**
- Button A short: `FORWARD` (next slot)
- Button A long: `CLONE_IC` (clone HF)
- Button B short: `BACKWARD` (previous slot)
- Button B long: `CLONE_ID` (clone LF)

```sh
# Check current button A configuration
hw settings btnpress -b a

# Set button A short press to cycle forward through slots
hw settings btnpress -b a -s -f FORWARD

# Set button A long press to clone HF card
hw settings btnpress -b a -l -f CLONE_IC

# Set button B short press to cycle backward
hw settings btnpress -b b -s -f BACKWARD

# Set button B long press to clone LF card
hw settings btnpress -b b -l -f CLONE_ID

# Disable button A short press
hw settings btnpress -b a -s -f NONE
```

#### `hw settings blekey`

Get or set the Bluetooth pairing key. This 6-digit PIN is required when pairing with the device over Bluetooth. The default key is typically `123456`.

| Option | Description |
|--------|-------------|
| `-k`, `--key` | New 6-digit numeric key |

**Security note:** Change the default key if you use Bluetooth in public environments.

```sh
# View current BLE key
hw settings blekey

# Set new BLE key
hw settings blekey -k 987654
```

#### `hw settings blepair`

Enable or disable the requirement for BLE pairing. When enabled, devices must enter the correct PIN to connect over Bluetooth. When disabled, any BLE client can connect without authentication.

| Option | Description |
|--------|-------------|
| `-e`, `--enable` | Require pairing (more secure) |
| `-d`, `--disable` | Allow connections without pairing |

**Security recommendation:** Keep pairing enabled unless you have a specific reason to disable it.

```sh
# Enable BLE pairing requirement (recommended)
hw settings blepair -e

# Disable BLE pairing (open access)
hw settings blepair -d
```

---

### HF 14A Commands

ISO14443-A is the communication standard used by most high-frequency (13.56 MHz) contactless cards, including MIFARE Classic, MIFARE Ultralight, NTAG, and many others. These commands provide low-level access to ISO14443-A cards.

#### `hf 14a scan`

Perform a quick scan for ISO14443-A tags and display basic identification information. This is usually the first command to run when analyzing an unknown HF card.

**What it shows:**
- **UID**: The card's Unique Identifier (4, 7, or 10 bytes depending on card type)
- **ATQA**: Answer To Request Type A - indicates card type and capabilities
- **SAK**: Select Acknowledge - indicates card type and features

**Common SAK values:**

| SAK | Card Type |
|-----|-----------|
| `08` | MIFARE Classic 1K |
| `18` | MIFARE Classic 4K |
| `09` | MIFARE Mini |
| `00` | MIFARE Ultralight / NTAG |
| `20` | MIFARE Plus / DESFire |

```sh
hf 14a scan
```

Output example:
```
 - UID  : 079C4B61
 - ATQA : 0400 (0x0004)
 - SAK  : 08
```

#### `hf 14a info`

Perform a detailed scan with additional analysis. This command does everything `scan` does, plus:

- **Type guessing**: Identifies the likely card type based on ATQA/SAK
- **PRNG analysis**: For MIFARE Classic, tests if the card has a **weak or hard PRNG** (critical for choosing attack strategy)
- **Technology detection**: Confirms the underlying technology

**Understanding PRNG results:**
- **Weak PRNG**: Card is vulnerable to `nested` attack - faster key recovery
- **Hard PRNG**: Card requires `hardnested` attack - slower but still possible
- **Static**: Card may be vulnerable to `staticnested` attack

```sh
hf 14a info
```

Output example:
```
 - UID  : 079C4B61
 - ATQA : 0400 (0x0004)
 - SAK  : 08
 - Guessed type(s) from SAK: MIFARE Classic 1K | Plus SE 1K | Plug S 2K | Plus X 2K
 - Mifare Classic technology
   # Prng: Weak
```

**Next steps based on results:**
- If `MIFARE Classic` with `Weak PRNG`: Use `hf mf fchk` → `hf mf nested`
- If `MIFARE Classic` with `Hard PRNG`: Use `hf mf fchk` → `hf mf hardnested`
- If `MIFARE Ultralight/NTAG`: Use `hf mfu` commands

#### `hf 14a raw`

Send raw ISO14443-A commands directly to a card. This is an advanced command for protocol-level interaction, debugging, or communicating with non-standard cards.

| Option | Description |
|--------|-------------|
| `-a`, `--activate` | Activate the RF field (power on the card) |
| `-s`, `--select` | Perform SELECT sequence (anticollision + select) |
| `-d`, `--data` | Data to send in hexadecimal |
| `-b`, `--bitlen` | Number of bits to send (for partial-byte commands like REQA) |
| `-t`, `--timeout` | Timeout in milliseconds to wait for response |
| `-r`, `--no-response` | Don't wait for a response (for write commands) |
| `-cc`, `--crc-calc` | Automatically calculate and append CRC-A |
| `-k`, `--keep-field` | Keep the RF field on after command (for command sequences) |
| `-n`, `--no-select` | Don't select card, but keep field on (for multi-command sessions) |

**Common raw commands:**

| Command | Hex | Description |
|---------|-----|-------------|
| REQA | `26` (7 bits) | Request Type A - wake card |
| WUPA | `52` (7 bits) | Wake-Up Type A - wake all cards including HALTed |
| HLTA | `50 00` | Halt - put card to sleep |
| RATS | `E0 80` | Request ATS - get card capabilities |

```sh
# Send REQA command (7-bit, needs bitlen)
hf 14a raw -a -d 26 -b 7

# Select card and send a command with auto-CRC, keeping field on
hf 14a raw -a -s -d 60 00 -cc -k

# Send follow-up command without re-selecting
hf 14a raw -n -d B0 00 -cc -k

# Turn off the field
hf 14a raw
```

---

### HF MF (MIFARE Classic) Commands

MIFARE Classic is one of the most widely deployed contactless smart card technologies, used in access control, public transit, and many other applications. These commands handle reading, writing, and cryptographic attacks against MIFARE Classic cards.

**MIFARE Classic Memory Structure:**

MIFARE Classic cards are divided into **sectors**, each containing **4 blocks** of 16 bytes:

| Card Type | Sectors | Blocks | Size |
|-----------|---------|--------|------|
| MIFARE Mini | 5 | 20 | 320 bytes |
| MIFARE 1K | 16 | 64 | 1 KB |
| MIFARE 2K | 32 | 128 | 2 KB |
| MIFARE 4K | 40 | 256 | 4 KB |

**Block types:**
- **Block 0**: Manufacturer block - contains UID (usually read-only)
- **Data blocks**: Store user data
- **Sector trailer** (last block of each sector): Contains Key A, Access Bits, and Key B

**Authentication:**
Each sector requires authentication with either **Key A** or **Key B** (6 bytes each) before reading or writing.

#### `hf mf fchk`

**Fast Check** - Test a list of keys against all sectors to find which keys work. This is usually the first attack to try, as many cards use default or common keys.

The command tries each provided key against both Key A and Key B positions on every sector, building a map of known keys.

| Option | Description |
|--------|-------------|
| `--mini` | Target is MIFARE Mini (5 sectors) |
| `--1k` | Target is MIFARE 1K (16 sectors) - **default** |
| `--2k` | Target is MIFARE 2K (32 sectors) |
| `--4k` | Target is MIFARE 4K (40 sectors) |
| `keys` | Space-separated list of keys to try (12 hex chars each) |
| `--key FILE` | Import keys from a `.key` file |
| `--dic FILE` | Import keys from a `.dic` dictionary file |
| `--export-key FILE` | Export found keys to a `.key` file |
| `--export-dic FILE` | Export found keys to a `.dic` file |
| `-m`, `--mask` | Sector mask in hex - skip certain sectors |

**Common default keys to try:**
- `FFFFFFFFFFFF` - Factory default (most common)
- `A0A1A2A3A4A5` - MAD (Mifare Application Directory)
- `000000000000` - Blank key
- `B0B1B2B3B4B5` - Common alternate
- `4D3A99C351DD` - NDEF default
- `D3F7D3F7D3F7` - Another common key

```sh
# Try common default keys on a 1K card
hf mf fchk --1k FFFFFFFFFFFF A0A1A2A3A4A5 000000000000 B0B1B2B3B4B5

# Check and save found keys for later use
hf mf fchk --1k FFFFFFFFFFFF A0A1A2A3A4A5 000000000000 --export-key found.key

# Load keys from a dictionary file
hf mf fchk --1k --dic extended_keys.dic

# Check a 4K card
hf mf fchk --4k FFFFFFFFFFFF A0A1A2A3A4A5

# Use a previously found key file
hf mf fchk --1k --key previous_found.key FFFFFFFFFFFF
```

Output example:
```
 - loaded 4 keys
 - progress of checking keys... 4 / 4 (100.0 %)
 - elapsed time: 2.315s

-----+-----+--------------+---+--------------+----
 Sec | Blk | key A        |res| key B        |res
-----+-----+--------------+---+--------------+----
 000 | 003 | FFFFFFFFFFFF | 1 | FFFFFFFFFFFF | 1
 001 | 007 | ------------ | 0 | FFFFFFFFFFFF | 1
 002 | 011 | ------------ | 0 | FFFFFFFFFFFF | 1
 ...
-----+-----+--------------+---+--------------+----
( 0: Failed, 1: Success )
```

**Interpreting results:**
- `1` = Key found and works
- `0` = No key in our list worked for this position
- `------------` = Unknown (need to attack this key)

#### `hf mf nested`

**Nested Authentication Attack** - Recover unknown keys when you already know at least one key. This exploits weaknesses in the CRYPTO1 cipher's PRNG (Pseudo-Random Number Generator).

**Requirements:**
- At least one known key (Key A or Key B from any sector)
- Card must have **weak PRNG** (check with `hf 14a info`)

**How it works:** The attack uses a known key to authenticate, then attempts authentication to another sector. By analyzing the encrypted nonces, the attack can determine the target key.

| Option | Description |
|--------|-------------|
| `--blk` | Block number where you have a known key **required** |
| `-a`, `-A` | The known key is Key A (default) |
| `-b`, `-B` | The known key is Key B |
| `-k`, `--key` | The known key value (12 hex characters) **required** |
| `--tblk` | Target block number to attack **required** |
| `--ta`, `--tA` | Target Key A (default) |
| `--tb`, `--tB` | Target Key B |

**Understanding block numbers:**
- Sector N's trailer block = N × 4 + 3 (for 1K cards)
- Sector 0: blocks 0-3, trailer = block 3
- Sector 1: blocks 4-7, trailer = block 7
- Sector 2: blocks 8-11, trailer = block 11
- etc.

```sh
# You know Key A for sector 0 (block 0), attack Key A of sector 1 (block 4)
hf mf nested --blk 0 -a -k FFFFFFFFFFFF --tblk 4 --ta

# You know Key B for sector 0 (block 3), attack Key A of sector 1
hf mf nested --blk 3 -b -k FFFFFFFFFFFF --tblk 4 --ta

# Attack Key B instead of Key A
hf mf nested --blk 0 -a -k FFFFFFFFFFFF --tblk 4 --tb
```

Output on success:
```
 - Nested recover one key running...
 - NT vulnerable: Nested
 - [8 candidate key(s) found ]
 - Found key: A0A1A2A3A4A5
```

**Tips:**
- If it fails, **try again** - the attack is probabilistic
- Try using a different known key as the source
- Keep the card stable and close to the reader

#### `hf mf darkside`

**Darkside Attack** - Recover a key without knowing ANY keys. This attack exploits vulnerabilities in cards with weak PRNG by analyzing authentication failures.

**Requirements:**
- Card must have **weak PRNG**
- Card must respond to authentication attempts for block 0

**When to use:** When `hf mf fchk` found zero keys and you need to get your first key.

**Limitations:** Some cards block this attack by not responding after failed authentications.

```sh
hf mf darkside
```

Output on success:
```
 - Darkside attack running...
 - Found key: FFFFFFFFFFFF
```

Output on failure:
```
 - Darkside error: Cannot get tag response enc(nak)
 - Key recover fail.
```

**If darkside fails:**
- The card may have protections against this attack
- Try `hf mf senested` if it might be a Chinese clone
- Use reader sniffing with `hf mf elog` as alternative

#### `hf mf hardnested`

**Hardnested Attack** - Recover keys from cards with **hardened PRNG**. This is a more sophisticated attack that works on newer cards where the regular nested attack fails.

**Requirements:**
- At least one known key
- Significant time (1-10 minutes per key typically)
- Card must be stable during the attack

| Option | Description |
|--------|-------------|
| `--blk` | Block number with known key **required** |
| `-a`, `-A` | Known key is Key A (default) |
| `-b`, `-B` | Known key is Key B |
| `-k`, `--key` | Known key value **required** |
| `--tblk` | Target block number **required** |
| `--ta`, `--tA` | Target Key A (default) |
| `--tb`, `--tB` | Target Key B |
| `--slow` | Slower but more thorough nonce acquisition |
| `--keep-nonces` | Save nonce file for debugging/retry |
| `--max-runs` | Maximum nonce collection runs per attempt (default: 200) |
| `--max-attempts` | Maximum attack attempts (default: 3) |

```sh
# Basic hardnested attack
hf mf hardnested --blk 0 -a -k FFFFFFFFFFFF --tblk 4 --ta

# More thorough attack (if basic fails)
hf mf hardnested --blk 0 -a -k FFFFFFFFFFFF --tblk 4 --ta --slow

# Maximum effort attack
hf mf hardnested --blk 0 -a -k FFFFFFFFFFFF --tblk 4 --ta --slow --max-runs 500 --max-attempts 5
```

Output shows progress:
```
 - HardNested attack starting...
 - Collecting nonces: 50/256 unique MSBs found
 - Finished acquisition phase for attempt 1.
 - Running key recovery...
 - Found key: 123456789ABC
```

**If hardnested fails repeatedly:**
- Increase `--max-runs` and `--max-attempts`
- Use `--slow` flag for better nonce quality
- Ensure card is stable and well-positioned
- Try a different known key as source

#### `hf mf senested`

**Static Encrypted Nested Attack** - Specialized attack for certain Chinese MIFARE Classic clones (FM11RF08S) that have a backdoor key.

Some clone cards have factory backdoor keys that allow direct access. This attack uses these backdoors to recover all sector keys.

| Option | Description |
|--------|-------------|
| `-k`, `--key` | Backdoor key to use (default: `A396EFA4E24F`) |
| `--sectors` | Number of sectors to attack (default: 16 for 1K) |
| `--starting-sector` | Start from this sector number (default: 0) |

**Known backdoor keys:**
| Key | Notes |
|-----|-------|
| `A396EFA4E24F` | Most common FM11RF08S backdoor |
| `A31667A8CEC1` | Alternate backdoor |
| `518B3354E760` | Another variant |

```sh
# Try default backdoor key
hf mf senested

# Try alternate backdoor key
hf mf senested -k A31667A8CEC1

# Try third known backdoor
hf mf senested -k 518B3354E760

# Attack only sectors 0-7
hf mf senested --sectors 8
```

```sh
# Use default backdoor key
hf mf senested

# Use alternate backdoor key
hf mf senested -k A31667A8CEC1
```

#### `hf mf rdbl`

**Read Block** - Read a single 16-byte block from a MIFARE Classic card. You must know a valid key for the sector containing the block.

| Option | Description |
|--------|-------------|
| `--blk` | Block number to read (0-63 for 1K) **required** |
| `-a`, `-A` | Authenticate with Key A (default) |
| `-b`, `-B` | Authenticate with Key B |
| `-k`, `--key` | Key to use for authentication (12 hex chars) **required** |

**Understanding block numbers:**
```
Sector 0: Block 0 (UID), Block 1 (data), Block 2 (data), Block 3 (trailer)
Sector 1: Block 4 (data), Block 5 (data), Block 6 (data), Block 7 (trailer)
...and so on
```

**Block 0 special:** Contains the UID and manufacturer data. On standard cards, this is read-only.

**Sector trailer:** Contains Key A (bytes 0-5), Access Bits (bytes 6-9), and Key B (bytes 10-15). Note: Key A is **never readable** - it always appears as zeros even if you authenticate with it!

```sh
# Read block 0 (UID block) using Key A
hf mf rdbl --blk 0 -a -k FFFFFFFFFFFF

# Read block 1 (data block) using Key A
hf mf rdbl --blk 1 -a -k FFFFFFFFFFFF

# Read using Key B instead
hf mf rdbl --blk 4 -b -k A0A1A2A3A4A5

# Read sector trailer (block 3)
hf mf rdbl --blk 3 -a -k FFFFFFFFFFFF
```

Output:
```
 - Data: 079c4b61b1080400034791f5f850d490
```

**Interpreting block 0 (UID block):**
```
079c4b61 b1 08 04 00 034791f5f850d490
│        │  │  │  │  └─ Manufacturer data
│        │  │  │  └──── Unused (varies)
│        │  │  └─────── SAK (08 = MF Classic 1K)
│        │  └────────── BCC (XOR checksum of UID)
│        └───────────── ATQA byte
└────────────────────── 4-byte UID
```

**Interpreting sector trailer (block 3, 7, 11, etc.):**
```
000000000000 ff078069 ffffffffffff
│            │        └─ Key B (readable if access bits allow)
│            └────────── Access bits + GPB
└─────────────────────── Key A (ALWAYS shows as zeros)
```

#### `hf mf wrbl`

**Write Block** - Write 16 bytes of data to a block on a MIFARE Classic card. You must have a valid key with write permissions.

| Option | Description |
|--------|-------------|
| `--blk` | Block number to write **required** |
| `-a`, `-A` | Authenticate with Key A (default) |
| `-b`, `-B` | Authenticate with Key B |
| `-k`, `--key` | Key for authentication **required** |
| `-d`, `--data` | Data to write (32 hex characters = 16 bytes) **required** |

**⚠️ WARNINGS:**

1. **Block 0 (UID block):** Writing here only works on "magic" cards (Gen1a, Gen2). On standard cards, block 0 is read-only. Writing incorrect data can permanently brick the card!

2. **Sector trailers:** Writing incorrect access bits can permanently lock the sector. Always verify access bits format before writing!

3. **Data is overwritten completely** - there's no append or partial write.

**Access bits reference:**
The safe default access bits are `FF078069` which allows Key A to read/write everything.

```sh
# Write data to block 1
hf mf wrbl --blk 1 -a -k FFFFFFFFFFFF -d 00000000000000000000000000000000

# Write sector trailer - BE CAREFUL!
# Format: KeyA (6 bytes) + Access Bits (4 bytes) + KeyB (6 bytes)
hf mf wrbl --blk 3 -a -k FFFFFFFFFFFF -d FFFFFFFFFFFF7F078069FFFFFFFFFFFF

# Change Key A to 123456123456, keeping default access bits and Key B
hf mf wrbl --blk 3 -a -k FFFFFFFFFFFF -d 123456123456FF078069FFFFFFFFFFFF

# Write to block using Key B
hf mf wrbl --blk 5 -b -k A0A1A2A3A4A5 -d 0102030405060708090A0B0C0D0E0F10
```

#### `hf mf view`

**View** - Display card memory contents, either from a dump file or by reading directly from the card using known keys.

| Option | Description |
|--------|-------------|
| `--mini` | Card is MIFARE Mini |
| `--1k` | Card is MIFARE 1K (default) |
| `--2k` | Card is MIFARE 2K |
| `--4k` | Card is MIFARE 4K |
| `-d`, `--dump` | Path to dump file to view |
| `-k`, `--key` | Path to key file - read from card using these keys |

```sh
# View contents of a dump file
hf mf view -d card_dump.bin

# Read card using known keys and display
hf mf view --1k -k found.key

# View a 4K card dump
hf mf view --4k -d dump_4k.bin

# Read and display, saving the dump
hf mf view --1k -k found.key -d backup.bin
```

Output shows a formatted memory dump with sector boundaries, UIDs, and data highlighted.

#### `hf mf value`

**Value Block Operations** - MIFARE Classic supports special "value blocks" that store a signed 32-bit integer with built-in backup and atomic increment/decrement operations. These are commonly used for stored-value applications (transit cards, vending, etc.).

| Option | Description |
|--------|-------------|
| `--blk` | Block number **required** |
| `-a`, `-A` | Use Key A (default) |
| `-b`, `-B` | Use Key B |
| `-k`, `--key` | Authentication key **required** |
| `--get` | Read and display the current value |
| `--set` | Initialize the block as a value block with specified value |
| `--inc` | Increment the value |
| `--dec` | Decrement the value |
| `-v`, `--value` | Value for set/inc/dec operations |

**Value block format:** A value block stores the value 3 times (with inversions) for redundancy, plus an address byte. This provides error detection and recovery.

```sh
# Read current value from block 5
hf mf value --blk 5 -a -k FFFFFFFFFFFF --get

# Initialize block 5 as a value block with value 100
hf mf value --blk 5 -a -k FFFFFFFFFFFF --set -v 100

# Increment value by 10 (100 → 110)
hf mf value --blk 5 -a -k FFFFFFFFFFFF --inc -v 10

# Decrement value by 5 (110 → 105)
hf mf value --blk 5 -a -k FFFFFFFFFFFF --dec -v 5
```

#### `hf mf elog`

**Emulator Log** - Retrieve and decrypt authentication logs captured while the Chameleon was emulating a MIFARE Classic card. This enables the **MFKEY32 attack** - recovering reader keys by sniffing.

**How the MFKEY32 attack works:**
1. Set up the Chameleon to emulate a card with detection logging enabled
2. Present the Chameleon to the target reader
3. The reader attempts authentication, sending encrypted nonces
4. The Chameleon logs these nonces
5. `hf mf elog --decrypt` analyzes the nonces to recover the reader's keys

| Option | Description |
|--------|-------------|
| (none) | Show the count of captured detection logs |
| `--decrypt` | Download logs and attempt to decrypt/recover keys |

```sh
# Check how many authentication attempts were logged
hf mf elog

# Download and decrypt to recover keys
hf mf elog --decrypt
```

Output example:
```
 - MF1 detection log count = 6, start download......
 - Download done (6 records), start parse and decrypt
 - Detection log for uid [079C4B61]
  > Decrypting block 3 key A detect log...
  > 6 records => 15/15 combinations. 1 key(s) found
  > Result ---------------------------
  > Block 3, A key result: {'123456123456'}
```

**Important:** You need at least **2 authentication attempts per key** for recovery to work. Present the Chameleon to the reader multiple times!

#### `hf mf eload`

**Emulator Load** - Load a card dump file into the emulator memory. This prepares the Chameleon to emulate the card stored in the dump.

| Option | Description |
|--------|-------------|
| `-f`, `--file` | Path to dump file **required** |
| `-t`, `--type` | File format: `bin` (binary) or `hex` (text hex) |
| `-s`, `--slot` | Target slot number (default: current slot) |

**Supported formats:**
- **Binary (.bin, .dump, .mfd):** Raw 1KB/4KB binary dumps
- **Hex (.hex, .eml):** Text files with hex data

```sh
# Load a binary dump file
hf mf eload -f card_dump.bin

# Load to a specific slot
hf mf eload -f card_dump.bin -s 2

# Load a hex format dump
hf mf eload -f card_dump.hex -t hex

# Load a .mfd file (Proxmark format)
hf mf eload -f card.mfd
```

After loading, use `hf mf econfig` to set the UID if needed, and enable the slot with `hw slot enable`.

#### `hf mf esave`

**Emulator Save** - Save the current emulator memory contents to a file. Use this to backup the emulated card or transfer it to another device.

| Option | Description |
|--------|-------------|
| `-f`, `--file` | Output file path **required** |
| `-t`, `--type` | File format: `bin` or `hex` |
| `-s`, `--slot` | Source slot number (default: current slot) |

```sh
# Save to binary file
hf mf esave -f backup.bin

# Save as hex format
hf mf esave -f backup.hex -t hex

# Save from a specific slot
hf mf esave -f slot3_backup.bin -s 3
```

#### `hf mf eview`

**Emulator View** - Display the current contents of the emulator memory in a formatted view. Useful for verifying what's loaded before emulating.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot to view (default: current slot) |

```sh
# View current slot's emulator memory
hf mf eview

# View specific slot
hf mf eview -s 2
```

#### `hf mf econfig`

**Emulator Configuration** - Configure MIFARE Classic emulator settings including UID, ATQA, SAK, keys, and special modes like Gen1a emulation.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot to configure |
| `--uid` | Set UID (4, 7, or 10 bytes in hex) |
| `--atqa` | Set ATQA (2 bytes hex) |
| `--sak` | Set SAK (1 byte hex) |
| `--enable-log` | Enable authentication logging for MFKEY32 |
| `--disable-log` | Disable authentication logging |
| `--gen1a` | Enable Gen1a magic card emulation |
| `--gen2` | Enable Gen2 (CUID) magic card emulation |
| `--no-gen` | Disable magic card modes |
| `--key` | Set key for all sectors (same key for A and B) |
| `--write-mode` | Write behavior: `normal`, `denied`, `deceive`, `shadow` |

**Write modes explained:**
- **normal**: Accept writes normally
- **denied**: Reject all write attempts
- **deceive**: Pretend to accept writes but don't actually save them
- **shadow**: Accept writes to RAM only (lost on power cycle)

**Gen1a mode:** Allows block 0 to be written, enabling UID changes. Required for cloning to magic cards.

```sh
# Set UID to match a specific card
hf mf econfig --uid DEADBEEF

# Set 7-byte UID
hf mf econfig --uid 04112233445566

# Enable detection logging (for MFKEY32 attack)
hf mf econfig --enable-log

# Disable detection logging
hf mf econfig --disable-log

# Configure as Gen1a magic card
hf mf econfig --gen1a

# Set all keys to a specific value
hf mf econfig --key FFFFFFFFFFFF

# Set write mode to shadow (RAM only)
hf mf econfig --write-mode shadow

# Full configuration for MFKEY32 sniffing
hf mf econfig --uid 079C4B61 --enable-log
```

---

### HF MFU (MIFARE Ultralight / NTAG) Commands

Commands for MIFARE Ultralight, NTAG, and related NFC tags. These tags are simpler than MIFARE Classic - they don't use CRYPTO1 encryption and have a different memory structure.

**Memory Structure:**

Unlike MIFARE Classic's 16-byte blocks, Ultralight/NTAG uses **4-byte pages**:

| Tag Type | Pages | User Memory | Special Features |
|----------|-------|-------------|------------------|
| MIFARE Ultralight | 16 | 48 bytes | Basic, no security |
| Ultralight C | 48 | 144 bytes | 3DES authentication |
| Ultralight EV1 (MF0UL11) | 20 | 48 bytes | Password protection |
| Ultralight EV1 (MF0UL21) | 41 | 128 bytes | Password protection |
| NTAG213 | 45 | 144 bytes | Password, signature |
| NTAG215 | 135 | 504 bytes | Password, signature (amiibo) |
| NTAG216 | 231 | 888 bytes | Password, signature |

**Page layout (common):**
- Pages 0-1: UID
- Page 2: Internal/lock bytes
- Page 3: Capability container (OTP)
- Pages 4+: User data
- Last pages: Configuration, password, PACK

#### `hf mfu rdpg`

**Read Page** - Read a single 4-byte page from an Ultralight/NTAG card.

| Option | Description |
|--------|-------------|
| `-p`, `--page` | Page number to read **required** |
| `-k`, `--key` | Password for protected cards (8 hex chars = 4 bytes) |
| `-l`, `--swap-endian` | Swap byte order (for some specific applications) |

```sh
# Read page 0 (first part of UID)
hf mfu rdpg -p 0

# Read page 4 (start of user data)
hf mfu rdpg -p 4

# Read from password-protected card
hf mfu rdpg -p 4 -k FFFFFFFF
```

Output:
```
 - Page 4: 01020304
```

#### `hf mfu wrpg`

**Write Page** - Write 4 bytes to a page on an Ultralight/NTAG card.

| Option | Description |
|--------|-------------|
| `-p`, `--page` | Page number to write **required** |
| `-d`, `--data` | Data to write (8 hex chars = 4 bytes) **required** |
| `-k`, `--key` | Password for protected cards |
| `-l`, `--swap-endian` | Swap byte order |

**⚠️ WARNINGS:**
- Pages 0-2 contain UID and lock bytes - writing here can brick the card on non-magic tags
- Lock bytes are OTP (One-Time Programmable) - once bits are set to 1, they cannot be changed back to 0
- Configuration pages have specific formats - incorrect values can lock the card

```sh
# Write to page 4 (user data area)
hf mfu wrpg -p 4 -d 01020304

# Write with password authentication
hf mfu wrpg -p 10 -d DEADBEEF -k FFFFFFFF
```

#### `hf mfu dump`

**Dump** - Read all pages from an Ultralight/NTAG card and optionally save to file.

| Option | Description |
|--------|-------------|
| `-p`, `--start-page` | Starting page number (default: 0) |
| `-n`, `--pages` | Number of pages to read |
| `-k`, `--key` | Password for protected cards |
| `-f`, `--file` | Output file path |
| `--bin` | Save as binary file (default is hex/text) |
| `-l`, `--swap-endian` | Swap byte order |

```sh
# Dump entire card to screen
hf mfu dump

# Dump to binary file
hf mfu dump -f ntag_dump.bin --bin

# Dump password-protected card
hf mfu dump -k FFFFFFFF

# Dump only pages 4-20
hf mfu dump -p 4 -n 16

# Dump and save to file
hf mfu dump -f mytag.bin --bin
```

#### `hf mfu version`

**Version** - Request version information from NTAG/Ultralight EV1 cards. This command returns detailed information about the tag type, manufacturer, and storage size.

**Note:** Only works on NTAG and Ultralight EV1. Original Ultralight doesn't support this command.

```sh
hf mfu version
```

Output example:
```
 - Vendor ID: 04 (NXP)
 - Type: 04 (NTAG)
 - Subtype: 02
 - Product: 0F (NTAG215)
 - Storage: 11 (504 bytes)
 - Protocol: 03
```

This is useful for identifying exactly which tag type you're working with.

#### `hf mfu signature`

**Signature** - Read the factory ECC signature from NTAG/Ultralight EV1 tags. This signature is written by NXP during manufacturing and can be used to verify tag authenticity.

**Note:** Only genuine NXP tags have valid signatures. Clones will either fail this command or return invalid signatures.

```sh
hf mfu signature
```

Output:
```
 - Signature: A1B2C3D4E5F6... (32 bytes)
```

#### `hf mfu rcnt`

**Read Counter** - Read one of the three counter values from NTAG/Ultralight EV1 tags. Counters can only be incremented and are used for anti-replay mechanisms.

| Option | Description |
|--------|-------------|
| `-c`, `--counter` | Counter number (0, 1, or 2) **required** |
| `-k`, `--key` | Password if counter reading is protected |
| `-l`, `--swap-endian` | Swap byte order |

```sh
# Read counter 0
hf mfu rcnt -c 0

# Read all three counters
hf mfu rcnt -c 0
hf mfu rcnt -c 1
hf mfu rcnt -c 2
```

#### `hf mfu ercnt`

**Emulator Read Counter** - Read a counter value from the Chameleon's Ultralight/NTAG emulator.

| Option | Description |
|--------|-------------|
| `-c`, `--counter` | Counter number (0, 1, or 2) |

```sh
hf mfu ercnt -c 0
```

#### `hf mfu ewcnt`

**Emulator Write Counter** - Set a counter value in the Chameleon's Ultralight/NTAG emulator.

| Option | Description |
|--------|-------------|
| `-c`, `--counter` | Counter number |
| `-v`, `--value` | Counter value to set |
| `-t`, `--tearing` | Tearing event flag |

```sh
# Set counter 0 to 100
hf mfu ewcnt -c 0 -v 100
```

#### `hf mfu eview`

**Emulator View** - Display the current contents of the Ultralight/NTAG emulator memory.

```sh
hf mfu eview
```

#### `hf mfu eload`

**Emulator Load** - Load a dump file into the Ultralight/NTAG emulator.

| Option | Description |
|--------|-------------|
| `-f`, `--file` | Dump file path **required** |
| `-t`, `--type` | File type: `bin` or `hex` |

```sh
# Load a binary dump
hf mfu eload -f ntag_dump.bin

# Load hex format
hf mfu eload -f ntag_dump.hex -t hex
```

#### `hf mfu esave`

**Emulator Save** - Save the Ultralight/NTAG emulator memory to a file.

| Option | Description |
|--------|-------------|
| `-f`, `--file` | Output file path **required** |
| `-t`, `--type` | File type: `bin` or `hex` |

```sh
hf mfu esave -f backup.bin
```

#### `hf mfu econfig`

**Emulator Configuration** - Configure the Ultralight/NTAG emulator settings.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot number |
| `--uid` | Set UID (7 bytes for NTAG/UL) |
| `--atqa` | Set ATQA |
| `--sak` | Set SAK |
| `--max-page` | Maximum page number (defines memory size) |
| `--pwd` | Set password (4 bytes) |
| `--pack` | Set PACK - Password Acknowledge (2 bytes) |
| `--version` | Set version data (for version command response) |
| `--signature` | Set signature data |
| `--reset-auth-cnt` | Reset authentication failure counter |

**Password/PACK:** The password is a 4-byte value. PACK is a 2-byte acknowledgment returned after successful password authentication. Default PACK is often `8080`.

```sh
# Set UID
hf mfu econfig --uid 04112233445566

# Set password and PACK
hf mfu econfig --pwd FFFFFFFF --pack 8080

# Set maximum page (for memory size)
hf mfu econfig --max-page 134    # NTAG215

# Reset auth failure counter
hf mfu econfig --reset-auth-cnt
```

#### `hf mfu edetect`

**Emulator Detection Log** - View logs of reader access attempts while emulating an Ultralight/NTAG card. Useful for understanding what a reader is trying to do.

| Option | Description |
|--------|-------------|
| `-c`, `--count` | Only show the count of logged events |
| `--clear` | Clear the detection log |
| `-f`, `--file` | Export logs to file |

```sh
# Check detection count
hf mfu edetect -c

# View all detections
hf mfu edetect

# Export to file
hf mfu edetect -f detections.txt

# Clear logs
hf mfu edetect --clear
```

---

### LF EM 410x Commands

EM4100/EM4102 are among the most common low frequency (125 kHz) RFID cards. They are simple read-only tags that transmit a fixed 40-bit ID (5 bytes = 10 hex characters) when powered by a reader's field. They have **no encryption or authentication** - the ID is transmitted in plain.

**Common uses:** Basic access control, time & attendance, animal identification.

**EM410x ID Format:** 10 hex characters representing 40 bits. The first 2 characters (8 bits) are typically a customer/site code, and the remaining 8 characters (32 bits) are the card number.

#### `lf em 410x read`

**Read** - Scan and read an EM410x card's ID. Place the card on the Chameleon's LF antenna (the smaller coil, or the back of the device).

**Tips for successful reading:**
- Position the card flat against the Chameleon
- Some cards work better at specific orientations
- Thick cards may need to be positioned precisely
- If reading fails, try moving the card slowly across the antenna

```sh
lf em 410x read
```

Output:
```
 - EM410x ID: 1A2B3C4D5E
```

**Decoding the ID:**
- ID `1A2B3C4D5E` = Site code `1A` (26), Card number `2B3C4D5E` (725561694)
- Different systems interpret the bytes differently - some use all 5 bytes as a single number

#### `lf em 410x write`

**Write** - Write an EM410x ID to a writable T55xx card. T55xx cards are special cards that can emulate EM410x and other LF formats.

| Option | Description |
|--------|-------------|
| `-i`, `--id` | EM410x ID to write (10 hex characters) **required** |

**Requirements:** You need a T55xx (T5577) writable card. These are sometimes called "clone cards" or "writable LF cards."

**⚠️ Warning:** Some cheap T55xx cards may require multiple write attempts or have reliability issues.

```sh
# Write ID to T55xx card
lf em 410x write -i 1A2B3C4D5E

# Clone the ID you just read
lf em 410x read          # Returns: 1A2B3C4D5E
lf em 410x write -i 1A2B3C4D5E
```

After writing, the T55xx card will respond as an EM410x with the programmed ID.

#### `lf em 410x econfig`

**Emulator Configuration** - Configure the Chameleon to emulate an EM410x card with a specific ID.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot number (1-8) |
| `-i`, `--id` | EM410x ID to emulate (10 hex characters) |

**Note:** Make sure to also run `hw slot type -s X -t EM410X` and `hw slot enable -s X --lf` if setting up a new slot.

```sh
# Set emulated ID for current slot
lf em 410x econfig -i 1A2B3C4D5E

# Set ID for specific slot
lf em 410x econfig -s 2 -i DEADBEEF00

# Full setup for a new slot
hw slot type -s 3 -t EM410X
lf em 410x econfig -s 3 -i 1A2B3C4D5E
hw slot enable -s 3 --lf
hw slot store
```

---

### LF HID Prox Commands

HID Proximity cards are widely used in corporate access control systems. Unlike simple EM410x, HID Prox cards encode facility codes, card numbers, and use different bit formats.

**Common HID formats:**

| Format | Bits | Structure |
|--------|------|-----------|
| H10301 (26-bit) | 26 | Facility code (8-bit) + Card number (16-bit) |
| H10302 (37-bit) | 37 | Facility code (16-bit) + Card number (19-bit) |
| Corporate 1000 (35-bit) | 35 | Company code (12-bit) + Card number (20-bit) |

**Note:** HID formats include parity bits for error checking. The actual card data is fewer bits than the format name suggests.

#### `lf hid prox read`

**Read** - Scan and read an HID Proximity card. The command attempts to decode the card data and display the format, facility code, and card number.

| Option | Description |
|--------|-------------|
| `-l`, `--length` | Expected bit length (helps with decoding) |

```sh
# Basic read
lf hid prox read

# Specify expected format (26-bit)
lf hid prox read -l 26
```

Output:
```
 - HID Prox Card
 - Bit Length: 26
 - Facility Code: 123
 - Card Number: 45678
 - Raw: 2004EC371E
```

#### `lf hid prox write`

**Write** - Write HID Prox card data to a T55xx card.

| Option | Description |
|--------|-------------|
| `-f`, `--facility` | Facility code |
| `-c`, `--card` | Card number **required** |
| `-l`, `--length` | Bit format (26, 35, 37, etc.) |
| `--raw` | Raw hex data (alternative to facility/card) |
| `--oem` | OEM code (for some formats) |

```sh
# Write 26-bit HID card
lf hid prox write -f 123 -c 45678 -l 26

# Write using raw data
lf hid prox write --raw 2004EC371E
```

#### `lf hid prox econfig`

**Emulator Configuration** - Configure the Chameleon to emulate an HID Prox card.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot number |
| `-f`, `--facility` | Facility code |
| `-c`, `--card` | Card number |
| `-l`, `--length` | Bit format |
| `--raw` | Raw hex data |
| `--oem` | OEM code |

```sh
# Set up 26-bit HID emulation
lf hid prox econfig -f 123 -c 45678 -l 26

# For specific slot
hw slot type -s 4 -t HIDPROX
lf hid prox econfig -s 4 -f 123 -c 45678 -l 26
hw slot enable -s 4 --lf
hw slot store
```

---

### LF Viking Commands

Viking access cards are another LF format used in some access control systems. They use a simpler encoding than HID.

#### `lf viking read`

**Read** - Scan and read a Viking card's ID.

```sh
lf viking read
```

Output:
```
 - Viking ID: 12345678
```

#### `lf viking write`

**Write** - Write a Viking ID to a T55xx card.

| Option | Description |
|--------|-------------|
| `-i`, `--id` | Viking ID (8 hex characters) **required** |

```sh
lf viking write -i 12345678
```

#### `lf viking econfig`

**Emulator Configuration** - Configure the Chameleon to emulate a Viking card.

| Option | Description |
|--------|-------------|
| `-s`, `--slot` | Slot number |
| `-i`, `--id` | Viking ID (8 hex characters) |

```sh
# Set emulated Viking ID
lf viking econfig -i 12345678

# Full slot setup
hw slot type -s 5 -t Viking
lf viking econfig -s 5 -i 12345678
hw slot enable -s 5 --lf
hw slot store
```

---

## Common Workflows

This section provides complete step-by-step guides for common tasks.

### MIFARE Classic Full Attack and Dump Workflow

Complete workflow to analyze, crack, and dump a MIFARE Classic card:

```sh
# ============================================
# STEP 1: Connect and Identify the Card
# ============================================

# Connect to your Chameleon
hw connect

# Quick scan to verify card is readable
hf 14a scan
# Output: UID, ATQA, SAK

# Detailed scan with PRNG analysis - IMPORTANT!
hf 14a info
# Look for: "Prng: Weak" or "Prng: Hard"
# This determines which attack to use later

# ============================================
# STEP 2: Try Default Keys (Dictionary Attack)
# ============================================

# Test common keys against all sectors
hf mf fchk --1k FFFFFFFFFFFF A0A1A2A3A4A5 000000000000 B0B1B2B3B4B5 4D3A99C351DD 1A982C7E459A D3F7D3F7D3F7 --export-key found.key

# Review results - look for which keys succeeded (1) vs failed (0)
# If ALL keys found: Skip to Step 5
# If SOME keys found: Continue to Step 3
# If NO keys found: Go to Step 4

# ============================================
# STEP 3: Recover Missing Keys (Nested Attack)
# ============================================

# For cards with WEAK PRNG:
# Use a known key to recover unknown keys
# Example: You know Key B for sector 0, recover Key A for sectors 0-6

# Sector 0 Key A (target block 0)
hf mf nested --blk 3 -b -k FFFFFFFFFFFF --tblk 0 --ta

# Sector 1 Key A (target block 4)
hf mf nested --blk 3 -b -k FFFFFFFFFFFF --tblk 4 --ta

# Sector 2 Key A (target block 8)
hf mf nested --blk 3 -b -k FFFFFFFFFFFF --tblk 8 --ta

# ... continue for each missing sector

# If nested fails repeatedly, try:
# - Running multiple times (it's probabilistic)
# - Using a different known key as source
# - Using hardnested instead (see Step 3b)

# ============================================
# STEP 3b: Hardnested Attack (for Hard PRNG)
# ============================================

# For cards with HARD PRNG, use hardnested instead:
hf mf hardnested --blk 0 -a -k FFFFFFFFFFFF --tblk 4 --ta

# If it fails, try more aggressive settings:
hf mf hardnested --blk 0 -a -k FFFFFFFFFFFF --tblk 4 --ta --slow --max-runs 500 --max-attempts 5

# ============================================
# STEP 4: No Keys Known - Start from Scratch
# ============================================

# Option A: Try darkside attack (weak PRNG only)
hf mf darkside
# If successful, use the found key for nested attacks

# Option B: Try static nested backdoor (Chinese clones)
hf mf senested
# Try alternate backdoor keys if default fails:
hf mf senested -k A31667A8CEC1
hf mf senested -k 518B3354E760

# Option C: Use reader sniffing (see MFKEY32 workflow below)

# ============================================
# STEP 5: Dump Card Data
# ============================================

# Once you have all keys, dump the card:
hf mf view --1k -k found.key -d card_dump.bin

# Or read individual blocks:
hf mf rdbl --blk 0 -a -k FFFFFFFFFFFF
hf mf rdbl --blk 1 -a -k FFFFFFFFFFFF
# ... etc

# ============================================
# STEP 6: Clone to Chameleon
# ============================================

# Load dump into emulator
hw slot type -s 1 -t MIFARE_1024
hf mf eload -f card_dump.bin -s 1

# Set the UID to match original
hf mf econfig -s 1 --uid 079C4B61

# Enable the slot
hw slot enable -s 1 --hf

# Save to flash
hw slot store

# Test in tag mode
hw mode -m tag
```

---

### Reader Sniffing (MFKEY32) Workflow

Recover keys by capturing authentication from a real reader. This is useful when:
- The card is unknown and no keys work
- You need to find the key a specific reader uses
- Your cloned card doesn't work (wrong keys)

```sh
# ============================================
# STEP 1: Set Up the Emulator
# ============================================

# Connect
hw connect

# Configure slot 1 for MIFARE Classic
hw slot type -s 1 -t MIFARE_1024
hw slot init -s 1 -t MIFARE_1024

# Set UID to match the original card (IMPORTANT!)
# The reader may check the UID before attempting auth
hf mf econfig -s 1 --uid 079C4B61

# Enable authentication logging
hf mf econfig -s 1 --enable-log

# Enable the slot
hw slot enable -s 1 --hf
hw slot change -s 1

# Save configuration
hw slot store

# Switch to tag emulator mode
hw mode -m tag

# ============================================
# STEP 2: Capture Authentication Attempts
# ============================================

# Physically present the Chameleon to the target reader
# Tap it like you would with a normal card
# Do this AT LEAST 2-3 times per sector you want to crack
# More taps = better chance of recovery

# The LEDs may blink during authentication attempts

# ============================================
# STEP 3: Recover Keys
# ============================================

# Return to CLI and switch back to reader mode
hw mode -m reader

# Check how many nonces were captured
hf mf elog
# Output: "MF1 detection log count = X"
# You need at least 2 records per key for recovery

# Decrypt and recover keys
hf mf elog --decrypt

# Output example:
#  - Detection log for uid [079C4B61]
#   > Block 3, A key result: {'123456123456'}
#   > Block 7, A key result: {'AABBCCDDEEFF'}

# ============================================
# STEP 4: Use Recovered Keys
# ============================================

# Disable logging (no longer needed)
hf mf econfig --disable-log

# Now use the recovered keys to read the original card:
hf mf rdbl --blk 0 -a -k 123456123456
hf mf rdbl --blk 1 -a -k 123456123456

# Or add them to your key file and continue cracking:
hf mf fchk --1k 123456123456 AABBCCDDEEFF FFFFFFFFFFFF --export-key updated.key
```

---

### Complete Card Cloning Workflow

Clone an entire MIFARE Classic card including proper keys:

```sh
# ============================================
# STEP 1: Read Original Card
# ============================================

hw connect
hf 14a scan                    # Get UID
hf 14a info                    # Check PRNG type

# Find all keys (using methods from above)
hf mf fchk --1k FFFFFFFFFFFF A0A1A2A3A4A5 --export-key keys.key

# Dump the card
hf mf view --1k -k keys.key -d original.bin

# ============================================
# STEP 2: Clone to Chameleon Emulator
# ============================================

# Set up slot
hw slot type -s 1 -t MIFARE_1024

# Load the dump
hf mf eload -f original.bin -s 1

# Match the UID
hf mf econfig -s 1 --uid 079C4B61

# Enable and save
hw slot enable -s 1 --hf
hw slot store

# ============================================
# STEP 3: Clone to a Magic Card (Gen1a/Gen2)
# ============================================

# Place a blank magic card on the reader

# Write Block 0 (UID) - Only works on magic cards!
hf mf wrbl --blk 0 -a -k FFFFFFFFFFFF -d 079C4B61B1080400034791F5F850D490

# Write data blocks
hf mf wrbl --blk 1 -a -k FFFFFFFFFFFF -d <data_from_dump>
hf mf wrbl --blk 2 -a -k FFFFFFFFFFFF -d <data_from_dump>

# Write sector trailer with correct keys
# Format: KeyA + AccessBits + KeyB
hf mf wrbl --blk 3 -a -k FFFFFFFFFFFF -d 123456123456FF078069FFFFFFFFFFFF

# Repeat for all sectors...
```

---

### Cloning EM410x LF Card

```sh
# ============================================
# STEP 1: Read Original
# ============================================

hw connect
lf em 410x read
# Output: EM410x ID: 1A2B3C4D5E

# ============================================
# STEP 2: Clone to Chameleon
# ============================================

# Configure slot for EM410X
hw slot type -s 2 -t EM410X

# Set the ID
lf em 410x econfig -s 2 -i 1A2B3C4D5E

# Enable and save
hw slot enable -s 2 --lf
hw slot store

# ============================================
# STEP 3: Clone to T55xx Card (Optional)
# ============================================

# Place a blank T55xx card on the reader
lf em 410x write -i 1A2B3C4D5E

# Verify
lf em 410x read
```

---

### Dual-Frequency Badge Cloning

Some access systems use both HF and LF on the same badge. The Chameleon can emulate both simultaneously:

```sh
# Read both frequencies from original
hf 14a scan           # Get HF info
lf em 410x read       # Get LF info (or lf hid prox read)

# Configure single slot for both
hw slot type -s 1 -t MIFARE_1024    # HF type
hw slot type -s 1 -t EM410X          # LF type

# Set up HF (MIFARE)
hf mf eload -f card.bin -s 1
hf mf econfig -s 1 --uid DEADBEEF

# Set up LF (EM410X)
lf em 410x econfig -s 1 -i 1A2B3C4D5E

# Enable both
hw slot enable -s 1 --hf --lf

# Save
hw slot store
```

---

## Troubleshooting

### Connection Issues

**Problem:** `hw connect` fails or times out

**Solutions:**
```sh
# Windows - Find available ports
# Open PowerShell and run:
[System.IO.Ports.SerialPort]::GetPortNames()

# Connect to specific port
hw connect -p COM3
hw connect -p COM4
```

```sh
# Linux - Find available ports
ls /dev/ttyACM* /dev/ttyUSB*

# Connect to specific port
hw connect -p /dev/ttyACM0

# If permission denied, add user to dialout group:
sudo usermod -a -G dialout $USER
# Then log out and back in
```

**Other connection issues:**
- Ensure no other program is using the serial port (close other terminals, serial monitors)
- Try a different USB cable (some cables are charge-only)
- Try a different USB port (USB 2.0 ports sometimes work better)
- On Windows, check Device Manager for driver issues

---

### Card Reading Issues

**Problem:** Card not detected / No response

**Solutions:**
- Position card flat against the Chameleon's antenna
- Move the card slowly around to find the "sweet spot"
- Some thick cards need very precise positioning
- For LF cards, use the back/bottom of the device
- For HF cards, use the top/front of the device
- Check that you're using the correct frequency commands (hf vs lf)

**Problem:** `hf 14a scan` says "No tag found"

**Solutions:**
- Verify the card is HF (13.56 MHz), not LF (125 kHz)
- Try holding the card in different positions
- Some cards may be damaged or shielded

---

### Attack Issues

**Problem:** Nested attack finds candidates but no valid key

**Solutions:**
- Run the attack multiple times (it's probabilistic)
- Keep the card very stable during the attack
- Try using a different known key as the source
- Verify the card actually has weak PRNG

**Problem:** Hardnested attack fails with "Reached max runs"

**Solutions:**
- Increase max runs: `--max-runs 500`
- Increase attempts: `--max-attempts 5`
- Use slow mode: `--slow`
- Keep the card extremely stable
- Try a different source key
- The card may have additional protections

**Problem:** Darkside attack fails

**Solutions:**
- This attack only works on cards with weak PRNG
- Some cards block after failed auth attempts - wait a moment
- Try the reader sniffing method instead (MFKEY32)

**Problem:** No keys found from dictionary

**Solutions:**
- Use a larger dictionary file with more keys
- Try the darkside attack to get one key
- Use reader sniffing (MFKEY32) if you have access to a working reader
- Check if it's a Chinese clone with `hf mf senested`

---

### Emulation Issues

**Problem:** Cloned card doesn't work at the reader

**Solutions:**
- Verify the UID matches exactly: `hf mf econfig --uid ORIGINAL_UID`
- Verify all sector keys match the original card
- Some readers check additional data (ATQA, SAK) - match those too
- Some readers verify data block contents, not just authentication
- Use reader sniffing to find out what key the reader expects

**Problem:** Clone works once then stops

**Solutions:**
- The reader might update a counter or timestamp
- Use shadow write mode to prevent permanent changes: `hf mf econfig --write-mode shadow`
- Check if the card uses value blocks that increment

**Problem:** Changes aren't saved after power cycle

**Solutions:**
- Run `hw slot store` after making changes!
- This saves both slot data and configuration to flash

---

### CLI Issues

**Problem:** "No such file or directory" errors

**Solutions:**
- Use full/absolute paths when specifying files
- Check that the file exists in the specified location
- On Windows, use backslashes or forward slashes consistently

**Problem:** Script execution disabled (Windows PowerShell)

**Solutions:**
```powershell
# Allow scripts for current session only (safest)
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process

# Then activate venv
.\venv\Scripts\Activate.ps1
```

---

*More examples and troubleshooting coming soon. Feel free to contribute!*

