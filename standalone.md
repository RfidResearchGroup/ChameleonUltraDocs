# Standalone Modes

Standalone modes let the ChameleonUltra run workflows without being connected
to a host. Configure the device once over USB or BLE, then use the physical
buttons to operate it untethered. Connect again later to retrieve results.

> **ChameleonUltra only.** ChameleonLite supports the framework (slot cycle,
> configuration, status) but not modes that require the HF reader hardware
> (AuthTrace).

## How it works

The device has two operational states:

- **Disarmed** , normal operation. Button A and B behave as configured in
  your button settings (cycle slots, clone UID, etc.). Standalone mode is
  inactive.
- **Armed** , the selected standalone mode is active. Button chords (both
  buttons pressed simultaneously) control the mode.

Arm and disarm by holding **both buttons at the same time for 1–5 seconds**,
then releasing. The LEDs sweep across all eight slots on arm and reverse on
disarm.

## Button gestures

Standalone uses **simultaneous-press chords** so they never conflict with
your existing single-button settings.

|Gesture            |How                           |Action               |
|-------------------|------------------------------|---------------------|
|**Short chord**    |Hold both < 1 second, release |Mode’s primary action|
|**Long chord**     |Hold both 1–5 seconds, release|Arm / disarm toggle  |
|**Very long chord**|Hold both ≥ 5 seconds, release|Clear captured data  |

Single-button presses (just A or just B) always behave normally, regardless
of whether standalone is armed.

## Available modes

### AuthTrace

The device acts as an HF reader and performs an authentication attempt
against a MIFARE Classic card, capturing the complete wire exchange ,
REQA, ATQA, anticollision, SELECT, SAK, auth command, nonce, and response.

Up to 8 sessions are stored in RAM. Each session is tagged with a status
(success, no tag, auth fail) and the full frame trace.

**Use case:** collect two or more auth traces from the same card, then feed
them to `mfkey32v2` offline to recover the sector key , without needing the
card to be tethered to a host.

**Primary action (short chord):** perform one auth session. Present the card
to the antenna first, then press both buttons briefly. The device polls for
up to the configured timeout, authenticates, and stores the trace. A brief
green flash means a trace was captured; a red flash means no card was found
in time.

**Very long chord:** discard all stored sessions.

-----

### Slot Cycle

The device rotates through enabled emulation slots at a configurable
interval. No tag interaction, no data written anywhere , the device just
steps through slots automatically.

**Use case:** extended emulation sessions where you want to cycle through
several card identities without manual intervention.

**Primary action (short chord):** manually advance to the next slot
immediately, and resume automatic cycling from there.

**Very long chord:** pause or resume automatic cycling.

## Setup (CLI)

All configuration is done over USB or BLE using the Python CLI before you
go untethered.

### Pick a mode

```
chameleon --> standalone set-mode authtrace
ok: state=DISARMED mode=AUTHTRACE flags=0x00
```

```
chameleon --> standalone set-mode slot_cycle
ok: state=DISARMED mode=SLOT_CYCLE flags=0x00
```

The selected mode and its configuration are saved to flash and persist
across reboots.

### Configure AuthTrace

```
chameleon --> standalone config authtrace
no persisted config for AUTHTRACE
```

```
chameleon --> standalone config --block 4 --key-type A --key FFFFFFFFFFFF --timeout 3000 authtrace
ok: type=A block=4 timeout=3000ms key=ffffffffffff
```

|Option          |Description                               |Default       |
|----------------|------------------------------------------|--------------|
|`--block N`     |Target block to authenticate against      |`4`           |
|`--key-type A|B`|MIFARE key type                           |`A`           |
|`--key`         |6-byte sector key as 12 hex chars         |`FFFFFFFFFFFF`|
|`--timeout MS`  |How long to poll for a card (100–30000 ms)|`3000`        |

The key does not need to be correct. Even a failed auth captures the nonces
needed for offline key recovery.

### Check status

```
chameleon --> standalone status
state: DISARMED  mode: AUTHTRACE  flags: (none)
```

## Capturing sessions (device buttons)

1. Ensure the mode is configured (`standalone config authtrace` to verify).
1. **Hold both buttons ~2 seconds** → LEDs sweep → device is armed.
1. For AuthTrace: place a MIFARE Classic card on the antenna, then **briefly
   press both buttons** (<1 second). The LED flashes briefly , green on
   success, red if no card was found.
1. Repeat step 3 for additional sessions (you need at least 2 for mfkey32v2).
1. **Hold both buttons ~2 seconds** → reverse sweep → device is disarmed and
   returns to normal tag emulation.

> **Note:** captured sessions are stored in RAM only and are lost on power
> off. Pull the results before unplugging.

## Retrieving results

Reconnect over USB or BLE, then:

```
chameleon --> standalone get-result --dump
2 authtrace sessions

=== session #0  status=ok ===
  -->  [  7 bits]  26
  <--  [ 16 bits]  0400
  ...
  <--  [ 32 bits]  619a379d       (NT)
  -->  [ 64 bits]  3e36b683b71d7daa  (NR || AR)
  <--  [ 32 bits]  04b42698       (AT)

=== session #1  status=ok ===
  ...
```

### Output options

|Option         |Output                                 |
|---------------|---------------------------------------|
|*(none)*       |One-line summary per session           |
|`--dump`       |Frame-by-frame trace in Proxmark3 style|
|`--json`       |Structured JSON for scripting          |
|`--raw -f FILE`|Raw binary blob saved to file          |

### Discard results

```
chameleon --> standalone clear-result
cleared
```

Or use the very long chord (≥5 seconds) on the device.

## Key recovery with mfkey32v2

Two sessions on the same card and block are enough to recover the key
offline. Extract from the dump:

- **UID** , first 4 bytes of the anticollision response
- **NT** , 4-byte tag response after the auth command
- **NR** , first 4 bytes of the 8-byte reader response
- **AR** , last 4 bytes of the 8-byte reader response

Then run:

```
mfkey32v2 <UID> <NT0> <NR0> <AR0> <NT1> <NR1> <AR1>
```

Example from real captures:

```
mfkey32v2 b2cfb11a 619a379d 3e36b683 b71d7daa 0f97c148 e76a1d21 3e34bfd6
```

If mfkey32v2 finds a key, it is the valid key for the configured block and
key type.

`mfkey32v2` is included with the [Proxmark3 client](https://github.com/RfidResearchGroup/proxmark3).

## Trigger from CLI

You can fire the mode’s primary action directly from the CLI without using
the device buttons , useful for scripting or when the device is mounted
somewhere awkward:

```
chameleon --> standalone trigger
triggered
```

The device must be armed first (`standalone set-mode` selects the mode;
arming still requires the button chord or a prior `standalone trigger` which
auto-arms).

## Troubleshooting

**LEDs show the wrong color on arm**
CU’s RGB LEDs can render certain colors differently across hardware
revisions. This is cosmetic and does not affect functionality.

**Short chord advances the slot instead of capturing**
The chord needs both buttons to register simultaneously. If there is too
much delay between pressing A and B, the device sees two independent
single-button events and runs the normal button function instead. Use a
single squeezing motion rather than two separate presses.

**`get-result` returns no data after captures**
Sessions are stored in RAM and are lost on reboot. If the device was
power-cycled between capture and retrieval, the sessions are gone. Collect
results before disconnecting power.

**Session status is `no_tag`**
The device polled for a card during the timeout window and did not find one.
Check that the card is within ~10mm of the HF antenna (top of the device)
when triggering.

**Session status is `auth_fail`**
The card responded but the authentication did not complete, usually because
the configured key is wrong for the target sector. The nonces are still
captured and are still useful for mfkey32v2 , a correct key is not required
for key recovery.