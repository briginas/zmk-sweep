# k4i's split kbd

ZMK firmware for a **Sweep-style cradio** split: **nice!nano v2** + `cradio_left` / `cradio_right`, built from this repo. The keyboard name in firmware is `sweep split kbd`.

**Build outputs:** `cradio_left`, `cradio_right`, and `settings_reset` (Bluetooth pairing reset / clear). The left half build includes ZMK Studio (RPC over USB-UART).

## Quick start

### Flash new firmware

1. [Fork this repo](https://github.com/briginas/zmk-sweep/fork)
2. (Optional) Edit `config/cradio.keymap` and `config/cradio.conf`.
3. Wait for GitHub Actions to finish, then download the artifact zip and extract the UF2 (or other) files for your halves.
4. Connect the half you are flashing with a USB cable.
5. Enter the bootloader on that half:
   1. A **bootloader** key on **Symbols** or **Nav** (see below), if you can still type
   2. Double-click the reset button
   3. Quickly short the reset button twice (if the button is unreliable)
   4. Quickly short **GND** and **RST** on the controller twice (e.g. second and third pins on the corner row—check your board’s pinout)
6. Copy the correct file to the bootloader drive:
   - Names containing **`left`** → left half
   - Names containing **`right`** → right half
   - **`settings_reset`** → run on a half to clear Bluetooth state when pairing is stuck (not required for normal keymap or name changes)

### Firmware behavior (config)

- Bluetooth: up to five paired connections (`CONFIG_BT_MAX_CONN` / paired count in `cradio.conf`).
- Battery reporting for both halves (split central can show the peripheral’s level).
- Idle timeout 30 s; **sleep** after one hour of inactivity.
- **Soft off** is enabled in the build; wakeup is wired via GPIO in the keymap (no `&soft_off` binding in the current layer map—see `config/cradio.keymap` if you add one).
- **Pointing** is enabled in Kconfig (`CONFIG_ZMK_POINTING=y`); the current keymap does not define a mouse layer.

## Keymap (overview)

Home row mods use **balanced** hold-tap (`hml` / `hmr`) on the letters **A S D F** and **J K L ;** (Shift / Ctrl / Gui / Alt as labeled in the map).

**Combos:** **Esc** and **Caps Lock**—see `combos` in `config/cradio.keymap` for key positions.

| Layer       | How you get there                                      | Role                                                                                                                                               |
| ----------- | ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Base**    | Default                                                | QWERTY; **left thumb** = hold **Symbols**; **right thumb** = hold **Nav**; thumbs share **Space** and **Enter** between them.                      |
| **Symbols** | Hold **left thumb** from Base                          | Punctuation and symbols; **Tab**; **Bootloader** on the left block; thumb row includes **Backspace** and **Adjust** (right thumb → **Adjust**).    |
| **Nav**     | Hold **right thumb** from Base                         | Numbers; arrows and navigation (with mods on the home row); **Bootloader**; **Adjust** via **left thumb** (`mo 3`).                                |
| **Adjust**  | From **Symbols** (right thumb) or **Nav** (left thumb) | **F1–F12**; **BT_SEL 0–3** and **BT_CLR**; **OUT_BLE** / **OUT_USB** (pick output, not a single toggle); **studio_unlock**; volume and media keys. |

There is **no** separate “Windows” layer, **no** mouse layer in this keymap, and **no** `&default_report` binding in the layer map (battery is still reported via ZMK’s battery settings).

## References

- [Sweep (hardware)](https://github.com/davidphilipbarr/Sweep)
- [Keymap I initially copied](https://www.youtube.com/watch?v=VShLPvF693k)
