# Radxa Zero Imager

A modified **Raspberry Pi Imager** for writing [radxa-zero-archlinux-builder](https://github.com/kxgx/radxa-zero-archlinux-builder) images to SD cards, with a single encrypted `config.txt` for headless first-boot setup.

> **Based on / 基于**: [Raspberry Pi Imager (`rpi-imager`)](https://github.com/raspberrypi/rpi-imager) by **Raspberry Pi Ltd**, licensed under **Apache 2.0**.  This is a fork that keeps the upstream history for attribution.
>
> **Adapted for / 针对**: the **Radxa Zero** (Amlogic S905Y2) running **Arch Linux ARM** images built by `radxa-zero-archlinux-builder` — not for Raspberry Pi boards.

## What's changed from upstream (魔改点)

| Area | Upstream (Raspberry Pi Imager) | This fork (Radxa Zero Imager) |
|---|---|---|
| **Target** | Raspberry Pi boards + the Pi OS catalog | **Radxa Zero only** (our builder's images) |
| **First-boot config** | Pi `firstrun.sh` / cloud-init / rpi-preseed | a single **`config.txt`** (hostname / timezone / ssh / wifi / user) |
| **Secrets** | written into the script | **hashed**: account password → `crypt(3)`, WiFi → PBKDF2-derived hex PSK (**no plaintext on the card**) |
| **OS list** | online Raspberry Pi OS manifest | **Radxa Zero manifest** (hosted + **embedded offline fallback** + local images) |
| **Name** | Raspberry Pi Imager | **Radxa Zero Imager** (`radxa-zero-imager.exe`) |

All modified files carry a `[MODIFIED for radxa-zero-archlinux-builder]` marker (an Apache 2.0 requirement).

## How it works

1. Pick a **Radxa Zero Arch Linux** image (hosted manifest / embedded / a local `.img` you built).
2. Pick the SD card.
3. Open **OS Customisation** and set hostname, username/password, WiFi, SSH, timezone.
4. Write.  The imager drops a `config.txt` on the SD card's boot partition (secrets **hashed/derived**).
5. Boot the Radxa Zero — the image's `boot-config-apply` first-boot hook reads `config.txt`, applies everything, then deletes it.

### config.txt format

```ini
hostname=radxa-zero
timezone=Asia/Shanghai
ssh=1
wifi_ssid=MyNetwork
wifi_country=CN
wifi_psk=<PBKDF2-derived 64-hex PSK>
username=xg
password_hash=$y$...   # crypt(3) hash, not the plaintext password
```

## Building

See [BUILDING.md](./BUILDING.md).  On Windows it needs Qt 6.9+ (MinGW), CMake and Ninja:

```sh
cmake -S src -B build -G Ninja -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_PREFIX_PATH=/path/to/Qt/6.x.y/mingw_64 \
  -DCMAKE_C_COMPILER=/path/to/mingw/bin/gcc.exe \
  -DCMAKE_CXX_COMPILER=/path/to/mingw/bin/g++.exe
cmake --build build
# → build/deploy/radxa-zero-imager.exe
```

## Related

- **Image builder**: [radxa-zero-archlinux-builder](https://github.com/kxgx/radxa-zero-archlinux-builder) — builds the Arch Linux ARM images this imager writes.

## License

**Apache 2.0** — same as upstream [Raspberry Pi Imager](https://github.com/raspberrypi/rpi-imager).
Copyright (C) 2020 Raspberry Pi Ltd (upstream) and contributors to this fork.
See [license.txt](./license.txt) for the full terms and third-party component licenses.
