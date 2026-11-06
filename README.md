# TWRP device tree for Redmi K Pad (turner)

TWRP for the **Redmi K Pad / Xiaomi Pad Mini** (`turner`, MediaTek Dimensity 9400+),
Android 16 / HyperOS 3. Built from scratch — no official TWRP exists for this tablet.

Adapted from the rothko (Xiaomi 14T Pro, same MT6991 platform) tree by cogan17,
which is the closest working MediaTek reference.

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

- [ ] First build (extract stock blobs → build vendor_boot.img)
- [ ] Boot test (`fastboot boot`, do NOT flash first)
- [ ] Display / touch
- [ ] Decryption (FBE)
- [ ] ADB / MTP / sideload / fastbootd
- [ ] Vibrator

## How it is built

Two GitHub Actions workflows (no local AOSP sync needed):

1. **Extract stock blobs** (`.github/workflows/extract-stock.yml`) — downloads the
   official recovery ROM on the runner, dumps `boot`/`init_boot`/`vendor_boot`/
   `vendor` from `payload.bin`, and commits the DTB, vendor_boot kernel modules,
   first-stage fstab and vendor blobs to a `stock/<version>` branch.
   Defaults: `OS3.0.310.0.WAOCNXM`, md5 `ec27187eb7bd6cab587f70cf1dfec582`.
2. **Build TWRP** (`.github/workflows/build-twrp.yml`) — syncs the
   `minimal-manifest-twrp/platform_manifest_twrp_aosp` (twrp-14.1) manifest,
   builds `vendorbootimage` (`lunch twrp_turner-ap2a-eng`), uploads
   `vendor_boot.img` as an artifact.

After the extract run, merge `stock/<version>` into the main branch, fix
`BOARD_DTB_SIZE` in `BoardConfig.mk` to the extracted DTB size (the extract
run prints it), and update `recovery/root/system/etc/recovery.fstab` from the
stock first-stage fstab if the partition layout differs.

## Test safely

```sh
fastboot boot vendor_boot.img
```

Only flash (`fastboot flash vendor_boot`) after the booted image shows
working display, touch and decryption. Keep a copy of the stock
`vendor_boot.img` (the extract workflow uploads it as an artifact) to
restore with `fastboot flash vendor_boot <stock vendor_boot.img>`.

## Credits

- cogan17 / JonesqPacMan — the rothko tree this one is adapted from
- TeamWin Recovery Project
