# TWRP device tree for Redmi K Pad (turner)

[中文](README.md)

TWRP for the **Redmi K Pad / Xiaomi Pad Mini** (`turner`, MediaTek Dimensity 9400+),
Android 16 / HyperOS 3. Built from scratch — no official TWRP exists for this tablet.

Based on **YARP**: TWRP-Test's `platform_manifest_twrp_aosp` (`twrp-16.0` branch,
TWRP 3.7.1_16, GUI2/LVGL interface) + PR #42 (OTG hotplug fix). Adapted from the
rothko (Xiaomi 14T Pro, same MT6991 platform) tree by cogan17, which is the
closest working MediaTek reference.

## Device

| Item | Detail |
| --- | --- |
| Codename | turner |
| Models | 25079RPDCC (China), 25079RPDCG (Global) |
| Chipset | MediaTek Dimensity 9400+ (MT6991) |
| Display | 8.8" IPS LCD, 1880x3008, 165 Hz |
| Battery | 7500 mAh, 67 W |
| Stock firmware used | OS3.0.310.0.WAOCNXM (Android 16, China stable) |

## Status

- [x] Stock blobs extracted (DTB 533115 bytes, vendor_boot modules, vendor blobs)
- [x] Old base (twrp-14.1 minimal manifest) built, but black-screened on the tablet
- [ ] **First YARP-base build** (this tree, not built yet)
- [ ] Boot test (flash vendor_boot, enter recovery; check display / touch / decryption)
- [ ] Decryption (FBE, incl. Xiaomi Weaver + Gatekeeper dual protectors)
- [ ] ADB / MTP / sideload / fastbootd / OTG
- [ ] Vibrator

## How it is built

Two GitHub Actions workflows (no local AOSP sync needed):

1. **Extract stock blobs** (`.github/workflows/extract-stock.yml`) — downloads the
   official recovery ROM on the runner, dumps `boot`/`init_boot`/`vendor_boot`/
   `vendor` from `payload.bin`, and commits the DTB, vendor_boot kernel modules,
   first-stage fstab and vendor blobs to a `stock/<version>` branch.
   Defaults: `OS3.0.310.0.WAOCNXM`, md5 `ec27187eb7bd6cab587f70cf1dfec582`.
2. **Build TWRP** (`.github/workflows/build-twrp.yml`) — syncs TWRP-Test's
   `platform_manifest_twrp_aosp` (`twrp-16.0`), merges PR #42, applies the
   patches in `patches/`, builds `vendorbootimage` (`lunch twrp_turner`),
   uploads `vendor_boot.img` as an artifact.

## Test safely

turner has no dedicated recovery partition; recovery lives in **vendor_boot**.
`fastboot boot` can only boot a boot image (with a kernel), not a vendor_boot
image, so testing means flashing it and rebooting to recovery:

```sh
fastboot flash vendor_boot vendor_boot.img
fastboot reboot recovery
```

The bootloader must be unlocked first (Xiaomi unlock takes a 168-hour wait and
wipes all data). Only keep the flashed vendor_boot once display, touch and
decryption are confirmed working. Keep a copy of the stock `vendor_boot.img`
(the extract workflow uploads it as an artifact) to restore with
`fastboot flash vendor_boot <stock vendor_boot.img>`.

## Patches

See `patches/PATCHES.md`: the 8 patches from the 14.1 era were checked one by
one against 16.0 — only 3 still apply (drop git revision from the startup
line, zstd ramdisk compression, Gatekeeper decryption); the other 5 are
already fixed upstream in 16.0 and must not be re-added.

## Credits

- cogan17 / JonesqPacMan — the rothko tree this one is adapted from
- lingqiqi — YARP (TWRP 3.7.1_16 + GUI2)
- TeamWin Recovery Project
