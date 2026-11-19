# YARP Recovery for Redmi K Pad (turner)

[简体中文](README.md)

**YARP Recovery** (TWRP 16.0 / GUI2) for the **Redmi K Pad / Xiaomi Pad Mini**
(`turner`, MediaTek Dimensity 9400+). There is no official TWRP for this tablet;
this is the first fully working third-party recovery for it.

**v1.0 is out**: display, touch, **data decryption (pattern / PIN / password)**,
OTG, MTP, ADB, haptics, backup & restore — all working, and the system boots
normally after a decrypt session. Get the image from
[Releases](../../releases); see [CHANGELOG.md](CHANGELOG.md) for details.

## How it works

Directly compiled images all black-screened on this device, so the release uses
a **hybrid** build:

- Stock HyperOS OS4 `vendor_boot` as the base (F0 and DTB stay stock;
  the DTB is byte-identical)
- The YARP (TWRP-Test `twrp-16.0` / GUI2) userspace in F1, with strict
  library isolation
- The crypto and touch HALs are started at runtime from the **installed
  system**: KeyMint / Gatekeeper run inside the system's own mitee TEE, so
  credential verification follows exactly the stock chain — the key to full
  FBE decryption on this MediaTek platform
- Full analysis, build materials and the per-version log live in the dev repo:
  [YARP-KPAD-dev](https://github.com/GQiangZI/YARP-KPAD-dev)

## Device

| Item | Detail |
| --- | --- |
| Codename | turner |
| Models | 25079RPDCC (China), 25079RPDCG (Global) |
| SoC | MediaTek Dimensity 9400+ (MT6991, ro.hardware = mt8799) |
| Display | 8.8", 1880 × 3008, 165 Hz |
| Battery | 7500 mAh, 67 W |
| Verified firmware | HyperOS OS4.0.3.0 (stable) |

## Feature status (v1.0)

- [x] GUI2 display and touch (Novatek NT36536E + Xiaomi touchfeature HAL)
- [x] **Data decryption**: metadata + FBE (DE/CE), pattern / PIN / password,
      verified by mitee Gatekeeper
- [x] ADB / MTP / sideload / USB OTG storage
- [x] Haptic feedback
- [x] Backup / restore, partition management, Install Image for Vendor Boot / DTBO
- [x] Boot-chain guard: flashable zips cannot write lk / preloader,
      protecting the bootloader unlock
- [ ] WiFi (not implemented; not needed for core recovery work)

## Download & install (prebuilt image)

Requires an unlocked bootloader. turner has no separate recovery partition —
recovery lives in **vendor_boot**, so **`fastboot boot` cannot be used**;
flash it:

```sh
fastboot flash vendor_boot YARP-turner-K_PAD-vendor_boot-46428cf2.img
fastboot reboot recovery
```

- Keep a stock `vendor_boot.img` handy; flash it back the same way to revert
- Do not flash lk / preloader from a full stock package (it can re-lock the
  bootloader). The recovery guards its own zip installs, but full-package
  flashing tools are outside that guard
- SHA256: `e9ede605818609f9cbb7725feef7bbfc0490b15200f2d88d85ee87051af3714a`

## Building (device tree)

This repo is the turner TWRP device tree, with two GitHub Actions workflows
(no local source sync needed):

1. **Extract stock files** (`.github/workflows/extract-stock.yml`) — downloads
   the stock recovery ROM and pulls boot / init_boot / vendor_boot / vendor
   out of `payload.bin` into a `stock/<version>` branch.
2. **Build TWRP** (`.github/workflows/build-twrp.yml`) — syncs TWRP-Test's
   `platform_manifest_twrp_aosp` (`twrp-16.0`), merges PR #42, applies the
   patches in `patches/`, and builds `vendorbootimage` (`lunch twrp_turner`).

Note: tree-built images black-screen on this device (see MEDIATEK-NOTES);
the release image is assembled with the hybrid method above, and its
materials live in the dev repo.

## Patches

See `patches/PATCHES.md`.

## MediaTek porting notes

Real-world porting issues (why direct builds don't boot, ro.hardware being
mt8799, symbol-pairing audits during repacks) are collected in
[MEDIATEK-NOTES.md](MEDIATEK-NOTES.en.md); the full crypto-stack audit is in
the dev repo at `hybrid/decrypt-stack/FINDINGS.md`.

## Credits

- cogan17 / JonesqPacMan — authors of the rothko device tree this one adapts
- lingqiqi, YuKongA and the TWRP-Test org — YARP (TWRP 3.7.1_16 + GUI2)
- kirklin — Skyward Recovery; 小智乃敲可爱 — the Skyward turner port whose
  image validated the hardware path this project builds on
- kodeaqua — taiko TWRP, the first public mitee decryption precedent
- TeamWin Recovery Project
