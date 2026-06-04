# Local ZMK Build With Docker

This guide shows how to build this ZMK firmware locally using the official ZMK
Docker image. This is close to how GitHub Actions builds the firmware, but runs
on your machine.

## Requirements

- Docker Desktop installed and running.
- This repository checked out at:

```sh
/Users/dev.briginas/dev/zmk-sweep
```

## Start The Build Container

Create a persistent workspace for downloaded ZMK/Zephyr sources and build
outputs:

```sh
mkdir -p ~/dev/zmk-workspace
```

Start an interactive container:

```sh
docker run --rm -it \
  -v /Users/dev.briginas/dev/zmk-sweep:/workspaces/zmk-config \
  -v ~/dev/zmk-workspace:/workspaces/zmk-workspace \
  -w /workspaces/zmk-workspace \
  zmkfirmware/zmk-build-arm:stable \
  bash
```

All commands below are run inside the container.

## Initialize The West Workspace

Run this once, or again after deleting `~/dev/zmk-workspace`:

```sh
west init -l /workspaces/zmk-config/config .
west update --fetch-opt=--filter=tree:0
west zephyr-export
```

## Build Firmware

### Left Half

The left half includes ZMK Studio over USB UART:

```sh
west build -p always -s zmk/app -d build/sweep_l \
  -b "nice_nano//zmk" \
  -S studio-rpc-usb-uart \
  -- \
  -DSHIELD=cradio_left \
  -DZMK_CONFIG=/workspaces/zmk-config/config \
  -DZMK_EXTRA_MODULES=/workspaces/zmk-config \
  -DCONFIG_ZMK_STUDIO=y
```

Output:

```sh
~/dev/zmk-workspace/build/sweep_l/zephyr/zmk.uf2
```

### Right Half

```sh
west build -p always -s zmk/app -d build/sweep_r \
  -b "nice_nano//zmk" \
  -- \
  -DSHIELD=cradio_right \
  -DZMK_CONFIG=/workspaces/zmk-config/config \
  -DZMK_EXTRA_MODULES=/workspaces/zmk-config
```

Output:

```sh
~/dev/zmk-workspace/build/sweep_r/zephyr/zmk.uf2
```

### Settings Reset

Use this firmware to clear Bluetooth settings/pairing state:

```sh
west build -p always -s zmk/app -d build/reset \
  -b "nice_nano//zmk" \
  -- \
  -DSHIELD=settings_reset \
  -DZMK_CONFIG=/workspaces/zmk-config/config \
  -DZMK_EXTRA_MODULES=/workspaces/zmk-config
```

Output:

```sh
~/dev/zmk-workspace/build/reset/zephyr/zmk.uf2
```

## Rebuild Faster

After a successful first build, rebuild a target by build directory:

```sh
west build -d build/sweep_l
west build -d build/sweep_r
west build -d build/reset
```

Use `-p always` again when switching board/shield combinations or after major
ZMK changes.

## Build The Named Variants

The `build.yaml` file also builds named variants with different keyboard names.
To build the black variant left half locally:

```sh
west build -p always -s zmk/app -d build/sweep_l_b \
  -b "nice_nano//zmk" \
  -S studio-rpc-usb-uart \
  -- \
  -DSHIELD=cradio_left \
  -DZMK_CONFIG=/workspaces/zmk-config/config \
  -DZMK_EXTRA_MODULES=/workspaces/zmk-config \
  -DCONFIG_ZMK_STUDIO=y \
  -DCONFIG_ZMK_KEYBOARD_NAME=\"sweep_split_b\"
```

For the black heavy variant, replace the build directory and keyboard name:

```sh
-d build/sweep_l_bh
-DCONFIG_ZMK_KEYBOARD_NAME=\"sweep_split_bh\"
```

The right-half variants are the same pattern, without `-S studio-rpc-usb-uart`
and without `-DCONFIG_ZMK_STUDIO=y`.

## Flashing

Put the nice!nano into bootloader mode, then copy the matching `zmk.uf2` file to
the mounted bootloader drive.

Recommended order after pairing issues or a major migration:

1. Flash `settings_reset` to both halves.
2. Flash the right half.
3. Flash the left half.

## References

- ZMK container setup: https://zmk.dev/docs/development/local-toolchain/setup/container
- ZMK build and flash docs: https://zmk.dev/docs/development/local-toolchain/build-flash
- ZMK Studio local build docs: https://zmk.dev/docs/features/studio
