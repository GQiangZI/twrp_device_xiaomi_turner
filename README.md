# TWRP device tree for Redmi K Pad (turner) / 红米 K Pad (turner) TWRP 设备树

TWRP for the **Redmi K Pad / Xiaomi Pad Mini** (`turner`, MediaTek Dimensity 9400+),
Android 16 / HyperOS 3. Built from scratch — no official TWRP exists for this tablet.

为 **红米 K Pad / Xiaomi Pad Mini**（`turner`，联发科天玑 9400+）编写的 TWRP，
Android 16 / HyperOS 3，从零新建——目前没有官方 TWRP 支持这台平板。

Adapted from the rothko (Xiaomi 14T Pro, same MT6991 platform) tree by cogan17,
which is the closest working MediaTek reference.

改编自 cogan17 的 rothko（小米 14T Pro，同 MT6991 平台）设备树，是最接近的、
可用的联发科参考树。

## Device / 设备

| Item / 项目 | Detail / 详情 |
| --- | --- |
| Codename / 代号 | turner |
| Models / 型号 | 25079RPDCC (China / 国行), 25079RPDCG (Global / 国际版) |
| Chipset / 芯片 | MediaTek Dimensity 9400+ (MT6991) / 联发科天玑 9400+ |
| Display / 屏幕 | 8.8" IPS LCD, 1880x3008, 165 Hz |
| Battery / 电池 | 7500 mAh, 67 W |
| Stock firmware used / 所用官方固件 | OS3.0.310.0.WAOCNXM (Android 16, China stable / 安卓 16 国行稳定版) |

## Status / 进度

- [ ] First build (extract stock blobs → build vendor_boot.img)
  首次编译（提取官方文件 → 编译 vendor_boot.img）
- [ ] Boot test (`fastboot boot`, do NOT flash first)
  启动测试（`fastboot boot`，先不要刷入）
- [ ] Display / touch — 屏幕 / 触摸
- [ ] Decryption (FBE) — 解密（FBE）
- [ ] ADB / MTP / sideload / fastbootd
- [ ] Vibrator — 振动

## How it is built / 如何编译

Two GitHub Actions workflows (no local AOSP sync needed):

两个 GitHub Actions 工作流（不需要在本地同步 AOSP 源码）：

1. **Extract stock blobs** (`.github/workflows/extract-stock.yml`) — downloads the
   official recovery ROM on the runner, dumps `boot`/`init_boot`/`vendor_boot`/
   `vendor` from `payload.bin`, and commits the DTB, vendor_boot kernel modules,
   first-stage fstab and vendor blobs to a `stock/<version>` branch.
   Defaults: `OS3.0.310.0.WAOCNXM`, md5 `ec27187eb7bd6cab587f70cf1dfec582`.

   **提取官方文件**（`.github/workflows/extract-stock.yml`）——在 runner 上下载官方
   Recovery ROM，从 `payload.bin` 中解出 `boot` / `init_boot` / `vendor_boot` /
   `vendor`，把 DTB、vendor_boot 内核模块、一阶段 fstab 和 vendor 文件提交到
   `stock/<version>` 分支。默认值：`OS3.0.310.0.WAOCNXM`，
   md5 `ec27187eb7bd6cab587f70cf1dfec582`。
2. **Build TWRP** (`.github/workflows/build-twrp.yml`) — syncs the
   `minimal-manifest-twrp/platform_manifest_twrp_aosp` (twrp-14.1) manifest,
   builds `vendorbootimage` (`lunch twrp_turner-ap2a-eng`), uploads
   `vendor_boot.img` as an artifact.

   **编译 TWRP**（`.github/workflows/build-twrp.yml`）——同步
   `minimal-manifest-twrp/platform_manifest_twrp_aosp`（twrp-14.1）代码清单，
   编译 `vendorbootimage`（`lunch twrp_turner-ap2a-eng`），把 `vendor_boot.img`
   作为 artifact 上传。

After the extract run, merge `stock/<version>` into the main branch, fix
`BOARD_DTB_SIZE` in `BoardConfig.mk` to the extracted DTB size (the extract
run prints it), and update `recovery/root/system/etc/recovery.fstab` from the
stock first-stage fstab if the partition layout differs.

提取完成后，把 `stock/<version>` 合并进 main 分支，按提取结果里打印的 DTB 大小
修正 `BoardConfig.mk` 中的 `BOARD_DTB_SIZE`；如果分区布局有变化，再按官方一阶段
fstab 更新 `recovery/root/system/etc/recovery.fstab`。

## Test safely / 安全测试

```sh
fastboot boot vendor_boot.img
```

Only flash (`fastboot flash vendor_boot`) after the booted image shows
working display, touch and decryption. Keep a copy of the stock
`vendor_boot.img` (the extract workflow uploads it as an artifact) to
restore with `fastboot flash vendor_boot <stock vendor_boot.img>`.

只有确认启动的镜像屏幕、触摸、解密都正常后，再刷入（`fastboot flash vendor_boot`）。
保留一份官方 `vendor_boot.img`（提取工作流会作为 artifact 上传），需要恢复时用
`fastboot flash vendor_boot <官方 vendor_boot.img>` 刷回。

## Credits / 致谢

- cogan17 / JonesqPacMan — the rothko tree this one is adapted from
  本设备树所改编的 rothko 设备树的作者
- TeamWin Recovery Project
