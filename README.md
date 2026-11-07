# 红米 K Pad (turner) TWRP 设备树

[English](README.en.md)

为 **红米 K Pad / Xiaomi Pad Mini**（`turner`，联发科天玑 9400+）编写的 TWRP，
Android 16 / HyperOS 3，从零新建——目前没有官方 TWRP 支持这台平板。

底子是 **YARP**：TWRP-Test 的 `platform_manifest_twrp_aosp`（`twrp-16.0` 分支，
TWRP 3.7.1_16，GUI2/LVGL 界面）+ PR #42（OTG 热插拔修复）。改编自 cogan17 的
rothko（小米 14T Pro，同 MT6991 平台）设备树，是最接近的、可用的联发科参考树。

## 设备

| 项目 | 详情 |
| --- | --- |
| 代号 | turner |
| 型号 | 25079RPDCC（国行）、25079RPDCG（国际版） |
| 芯片 | 联发科天玑 9400+ (MT6991) |
| 屏幕 | 8.8 英寸 IPS LCD，1880x3008，165 Hz |
| 电池 | 7500 mAh，67 W |
| 所用官方固件 | OS3.0.310.0.WAOCNXM（安卓 16 国行稳定版） |

## 进度

- [x] 提取官方文件（DTB 533115 字节、vendor_boot 模块、vendor 文件）
- [x] 旧底子（twrp-14.1 minimal manifest）编译成功，但刷到平板上黑屏
- [ ] **YARP 底子首次编译**（本树，还没编过）
- [ ] 启动测试（刷入 vendor_boot 后进 recovery，先确认屏幕 / 触摸 / 解密）
- [ ] 解密（FBE，含小米 Weaver + Gatekeeper 双 protector）
- [ ] ADB / MTP / sideload / fastbootd / OTG
- [ ] 振动

## 如何编译

两个 GitHub Actions 工作流（不需要在本地同步 AOSP 源码）：

1. **提取官方文件**（`.github/workflows/extract-stock.yml`）——在 runner 上下载官方
   Recovery ROM，从 `payload.bin` 中解出 `boot` / `init_boot` / `vendor_boot` /
   `vendor`，把 DTB、vendor_boot 内核模块、一阶段 fstab 和 vendor 文件提交到
   `stock/<version>` 分支。默认值：`OS3.0.310.0.WAOCNXM`，
   md5 `ec27187eb7bd6cab587f70cf1dfec582`。
2. **编译 TWRP**（`.github/workflows/build-twrp.yml`）——同步 TWRP-Test 的
   `platform_manifest_twrp_aosp`（`twrp-16.0`），合并 PR #42，再按
   `patches/` 里的补丁打补丁，编译 `vendorbootimage`（`lunch twrp_turner`），
   把 `vendor_boot.img` 作为 artifact 上传。

## 安全测试

turner 没有独立的 recovery 分区，recovery 在 **vendor_boot** 里。
`fastboot boot` 只能启动带内核的 boot 镜像，启动不了 vendor_boot，
所以测试就是刷入再进：

```sh
fastboot flash vendor_boot vendor_boot.img
fastboot reboot recovery
```

前提是 bootloader 已解锁（小米解锁要等 168 小时，解锁会清空数据）。
只有确认屏幕、触摸、解密都正常后，这个 vendor_boot 才能长期留着。
保留一份官方 `vendor_boot.img`（提取工作流会作为 artifact 上传），
需要恢复时用 `fastboot flash vendor_boot <官方 vendor_boot.img>` 刷回。

## 补丁说明

见 `patches/PATCHES.md`：14.1 时代的 8 个补丁逐个核对过，能用在 16.0 上
的只有 3 个（启动行去版本号、zstd 压 ramdisk、Gatekeeper 解密），
其余 5 个在 16.0 上游已经修了，不要再加回去。

## 致谢

- cogan17 / JonesqPacMan——本设备树所改编的 rothko 设备树的作者
- lingqiqi——YARP（TWRP 3.7.1_16 + GUI2）的作者
- TeamWin Recovery Project
