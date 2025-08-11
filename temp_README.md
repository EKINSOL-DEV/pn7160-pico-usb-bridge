# NFC Controller Setup & Initialization

## Pin Configuration
| Pin Function | GPIO Pin | Value |
|--------------|----------|-------|
| I2C_SDA_PIN  | 16       | -     |
| I2C_SCL_PIN  | 17       | -     |
| VEN_PIN      | 14       | -     |
| IRQ_PIN      | 15       | -     |
| FWDL_PIN     | 11       | -     |

## I2C Configuration
| Parameter   | Value   |
|-------------|---------|
| I2C_ADDRESS | 0x28    |
| I2C_FREQ    | 100,000 Hz |
| BOOT_DELAY  | 120 ms  |

## ⚠️ CRITICAL: Initialization Sequence

**DO NOT INTERACT WITH THE I2C BUS BEFORE COMPLETING THESE STEPS**

1. **Set VEN to LOW** (power off)
2. **Wait 10 ms**
3. **Set FWDL to LOW** (normal operation mode)
4. **Set VEN to HIGH** (power on)  
5. **Wait at least 120 ms** (BOOT_DELAY)
6. **Now you can safely use the I2C bus**

### Why This Order Matters
- VEN controls power to the NFC controller
- FWDL must be set before powering on to avoid firmware download mode
- The 120ms delay ensures the controller is fully booted before I2C communication

## Arduino Bridge API

The `nfc_bridge_new.ino` provides a serial interface for NFC operations. Depends on ElectronicCats-PN7150.

### Test Commands - MacOS
1. Install homebrew
2. Run `brew install minicom`
3. Find the USB device (must be flashed first)
4. Run this command (replace XXXX with the number after `usbmodem`)
```
minicom -D /dev/tty.usbmodemXXXX -b 115200
```

### Commands

#### PING
Test connectivity
```
PING
→ OK:PONG
```

#### POLL[:MAX]
Detect and read NFC tags (default: 1 tag)
```
POLL
→ OK:1:2:0:0477174263680:"Hello World"
→ POLLEND

POLL:3
→ OK:1:2:0:0477174263680:"First tag"
→ OK:2:2:0:1234567890AB:"Second tag"  
→ POLLEND
```

**Response Format:** `OK:TAG#:PROTOCOL:TECH:ID:"MESSAGE"`
- TAG# = Sequential number (1, 2, 3...)
- PROTOCOL = Protocol number (2=T2T for NTAG215)
- TECH = Technology (0=PASSIVE_NFCA)
- ID = Tag UID in compact hex
- MESSAGE = NDEF text content ("" if empty)

#### WRITE:message
Write text to NFC tag (T2T tags only)
```
WRITE:Hello World
→ OK:WRITE_SUCCESS
```

### Error Codes
```
ERROR:TIMEOUT           - No tags detected within 10 seconds
ERROR:UNSUPPORTED_TAG   - Tag type not supported for writing
ERROR:TEXT_TOO_LONG     - Message exceeds tag capacity (~40 chars)
ERROR:WRITE_FAILED      - Hardware write operation failed
```

### Usage
1. Upload `nfc_bridge_new.ino` to Pico 2W
2. Connect at 115200 baud
3. Send commands ending with `\n`