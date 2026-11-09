# RadioBee P1 — Firmware v0.2.42: How to Flash

This guide installs RadioBee firmware **v0.2.42** on a RadioBee P1 (ESP32-WROOM-32) over USB.
Download page: <https://github.com/hebersoto/radiobee-releases/releases/latest>

## 1. Pick the right file

| File | Flash address | When to use it |
|---|---|---|
| `RadioBee_P1_v0.2.42_update_0x10000.bin` | **`0x10000`** | **Normal update.** The RadioBee already runs RadioBee firmware. Keeps your saved settings (FX.25 TX, TX level). |
| `RadioBee_P1_v0.2.42_full_0x0.bin` | **`0x0`** | Blank or unknown ESP32, or recovery when the update file does not boot. **Erases saved settings** (back to defaults). |

Use the **update** file unless you were told otherwise. The device name (`RadioBee P1-XXXX`) never changes.

> **Flash address matters.** The address is in each file name. Writing a file at the wrong address leaves the board
> unable to start (fix: flash the full file at `0x0`).

### Check the download (optional)

| File | SHA-256 |
|---|---|
| `RadioBee_P1_v0.2.42_update_0x10000.bin` | `0c6bb231bd8cca37b52cae298f49cb238ae7bdb62b1625ddc8da120245417851` |
| `RadioBee_P1_v0.2.42_full_0x0.bin` | `742b88fc909b9ccda52d17eedd7f96c44f74e4919135e1db1421b4d68675821e` |

- Windows: `certutil -hashfile RadioBee_P1_v0.2.42_update_0x10000.bin SHA256`
- Mac / Linux: `shasum -a 256 RadioBee_P1_v0.2.42_update_0x10000.bin`

## 2. What you need

- A **USB data cable**. Some cables only charge and will not work.
- The **USB driver** for the board's USB chip:
  - Most RadioBee P1 boards: **Silicon Labs CP210x** "USB to UART Bridge VCP" driver
    (<https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers>). Windows 10/11 usually installs it
    automatically; Mac and Linux have it built in.
  - Some ESP32 DevKit boards use a **CH340** chip instead: install the WCH CH340 driver.
- **Close every program that uses the RadioBee COM port** (PinPoint, PuTTY, Arduino Serial Monitor).
- Then use **one** of the three methods below.

### Find the port

- **Windows:** Device Manager → *Ports (COM & LPT)* → "Silicon Labs CP210x USB to UART Bridge (**COM5**)".
- **Mac:** `ls /dev/cu.*` → `/dev/cu.usbserial-0001` or `/dev/cu.SLAB_USBtoUART`.
- **Linux:** `/dev/ttyUSB0` (your user needs to be in the `dialout` group).

## 3. Method A — Web flasher (easiest, nothing to install)

Works in **Google Chrome** or **Microsoft Edge** on a computer (not Firefox or Safari, not phones).

1. Open <https://espressif.github.io/esptool-js/>.
2. Plug in the RadioBee. Set **Baudrate** to `460800`, click **Connect** and pick the port.
3. Under **Program**:
   - **Flash Address:** `0x10000`
   - **File:** `RadioBee_P1_v0.2.42_update_0x10000.bin`

   (Full image instead: address `0x0` and the `full_0x0` file.)
4. Click **Program** and wait until it reaches 100 %.
   **Do not click "Erase Flash"**: that wipes your saved settings.
5. Click **Disconnect**, then unplug and replug the RadioBee (or press its **EN/RST** button).

## 4. Method B — esptool (command line: Windows, Mac, Linux)

Install esptool, either:

- with Python: `pip install esptool`, or
- as a standalone program without Python: download it from <https://github.com/espressif/esptool/releases>.

Run the command in the folder that holds the `.bin` file. Replace `COM5` with your port
(Mac: `/dev/cu.usbserial-0001`, Linux: `/dev/ttyUSB0`).

**Update (keeps settings):**

```sh
esptool --chip esp32 -p COM5 -b 460800 write-flash 0x10000 RadioBee_P1_v0.2.42_update_0x10000.bin
```

**Full image (erases settings):**

```sh
esptool --chip esp32 -p COM5 -b 460800 write-flash 0x0 RadioBee_P1_v0.2.42_full_0x0.bin
```

With esptool version 4 or older, write `write_flash` instead of `write-flash`
(also `esptool.py` instead of `esptool` on some installs).
esptool resets the board into flash mode by itself and restarts it when it is done.

## 5. Method C — Espressif Flash Download Tool (Windows, graphical)

1. Download **Flash Download Tools** from <https://www.espressif.com/en/support/download/other-tools> and unzip it.
2. Start it and choose **ChipType: ESP32**, **WorkMode: Develop**, **LoadMode: UART**.
3. On the first line, tick the box, pick `RadioBee_P1_v0.2.42_update_0x10000.bin` and type `0x10000` after `@`.
   (Full image: the `full_0x0` file `@ 0x0`.) Leave the other lines unticked.
4. **SPI SPEED** 40 MHz, **SPI MODE** DIO. Choose your **COM** port and **BAUD** 460800.
5. Click **START** and wait for **FINISH**. **Do not press ERASE.**
6. Unplug and replug the RadioBee.

## 6. Check that it worked

1. Open a serial terminal on the USB COM port: **115200 baud, 8N1, no flow control**
   (PuTTY → *Serial*, or the Arduino IDE Serial Monitor). Default PuTTY settings are fine.
2. Unplug and replug the board (or press EN/RST). The boot text should show:

   ```text
   Firmware 0.2.42-...
   BT/BLE Name: RadioBee P1-XXXX
   ```

3. Press **`s`** (no Enter needed) for the status page.

## 7. After updating from an older version (0.2.33 or earlier)

- **BLE phone apps** (e.g. aprs.fi): select the RadioBee again once. Its BLE address changed in this version.
- **Windows:** classic Bluetooth now appears as its own device and can create an SPP COM port for PinPoint or PuTTY.
  If Windows had paired RadioBee before, remove the old RadioBee entries in *Settings → Bluetooth & devices*
  and add it again.
- Classic Bluetooth apps (e.g. APRSdroid): no change needed.
- **PuTTY** (Bluetooth COM port or USB COM port): since 0.2.42 all RadioBee text lines up with PuTTY's default
  settings; "Implicit CR in every LF" is no longer needed (it was for 0.2.40 and older).

## 8. Troubleshooting

| Problem | Fix |
|---|---|
| "Failed to connect" / "Wrong boot mode detected" | Hold the **BOOT** button on the RadioBee, start flashing, release BOOT when writing starts. |
| No COM port appears | Install the CP210x (or CH340) driver; try another USB cable (data, not charge-only) or another USB port. |
| "Port is busy" / "Access denied" | Close PinPoint, PuTTY, Arduino IDE or anything else using that port. |
| Flashing stops partway | Lower the baud rate to `115200` and try again. |
| Board does not start after the update file | Flash the **full** file at `0x0` (settings go back to defaults). |
| Web flasher shows no Connect dialog | Use Chrome or Edge on a computer; Firefox and Safari do not support Web Serial. |

## 9. License and source code

RadioBee firmware is free software under the **GNU General Public License v2.0 or later**. It contains code derived
from Dire Wolf. See [LICENSE](../LICENSE).

The complete source code for this version is `RadioBee_P1_v0.2.42_source.ino`
(SHA-256 `903ac9422d0967fc7dedad493fb4ff621652c8b0d8f1e645990d7ba728bef8b5`). To get it, open an
[issue](https://github.com/hebersoto/radiobee-releases/issues); this offer is valid for at least three years.
It builds with Arduino core **esp32:esp32 3.3.11**, board **ESP32 Dev Module**, default partition scheme.
