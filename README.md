# 红米 K Pad（turner）YARP Recovery

[English](README.en.md)

为 **红米 K Pad / Xiaomi Pad Mini**（`turner`，联发科天玑 9400+）适配的
**YARP Recovery**（TWRP 16.0 / GUI2 界面）——目前没有官方 TWRP 支持这台平板，
本项目是首个功能完整、可日常使用的第三方 Recovery。

**v1.0 已发布**：显示、触控、**数据解密（图案 / PIN / 密码）**、OTG、MTP、
ADB、振动、备份恢复全部打通，解密后退出可正常进系统。成品镜像在
[Releases](../../releases) 下载，更新日志见 [CHANGELOG.md](CHANGELOG.md)。

## 实现方式

直出编译的镜像在本机全部黑屏，最终采用**混合拼接**路线：

- 以 HyperOS OS4 原厂 `vendor_boot` 为底（F0 与 DTB 保持原厂，DTB 逐字节一致）
- F1 放入 YARP（TWRP-Test `twrp-16.0` / GUI2）用户态，做严格库隔离
- 解密与触控的 HAL 在运行时从**已安装系统**拉起：KeyMint / Gatekeeper 跑在
  系统自带的 mitee TEE 里，凭据验证与原厂走同一条链——这是联发科平台
  FBE 解密能完整打通的关键
- 开发全过程的分析、拼接材料与逐版记录在开发仓库
  [YARP-KPAD-dev](https://github.com/GQiangZI/YARP-KPAD-dev)

## 设备

| 项目 | 详情 |
| --- | --- |
| 代号 | turner |
| 型号 | 25079RPDCC（国行）、25079RPDCG（国际版） |
| 芯片 | 联发科天玑 9400+（MT6991，ro.hardware = mt8799） |
| 屏幕 | 8.8 英寸，1880 × 3008，165 Hz |
| 电池 | 7500 mAh，67 W |
| 验证固件 | HyperOS OS4.0.3.0 正式版 |

## 功能状态（v1.0）

- [x] GUI2 界面显示、触控（Novatek NT36536E + 小米 touchfeature HAL）
- [x] **数据解密**：metadata + FBE（DE/CE），图案 / PIN / 密码，mitee Gatekeeper 验证
- [x] ADB / MTP / sideload / OTG U 盘
- [x] 振动反馈
- [x] 备份 / 恢复、分区管理、Install Image 刷 Vendor Boot / DTBO
- [x] 启动链防护：卡刷包禁止写 lk / preloader，保护 BL 解锁
- [ ] WiFi（未做，不影响核心功能）

## 下载与刷入（成品镜像）

前提：bootloader 已解锁。turner 没有独立 recovery 分区，Recovery 在
**vendor_boot** 里，**不能用 `fastboot boot` 测试**，只能刷入：

```sh
fastboot flash vendor_boot YARP-turner-K_PAD-vendor_boot-46428cf2.img
fastboot reboot recovery
```

- 刷前请先备一份原厂 `vendor_boot.img`，需要恢复时刷回即可
- 不要刷入原厂完整包里的 lk / preloader（可能导致重新上锁），本 Recovery
  已内置启动链防护，但线刷全量包不在防护范围内
- 校验：SHA256 `e9ede605818609f9cbb7725feef7bbfc0490b15200f2d88d85ee87051af3714a`

## 如何编译（设备树）

本仓库是 turner 的 TWRP 设备树，两个 GitHub Actions 工作流（不需要本地同步源码）：

1. **提取官方文件**（`.github/workflows/extract-stock.yml`）——下载官方 Recovery
   ROM，从 `payload.bin` 解出 boot / init_boot / vendor_boot / vendor，提交到
   `stock/<version>` 分支。
2. **编译 TWRP**（`.github/workflows/build-twrp.yml`）——同步 TWRP-Test 的
   `platform_manifest_twrp_aosp`（`twrp-16.0`），合并 PR #42，按 `patches/`
   打补丁，编译 `vendorbootimage`（`lunch twrp_turner`）。

注意：直出镜像在本机黑屏（原因见 MEDIATEK-NOTES），发布镜像是按上文
「混合拼接」路线组装的，拼接材料与脚本在开发仓库。

## 补丁说明

见 `patches/PATCHES.md`。

## 联发科适配问题记录

适配过程中实际遇到的问题（直出镜像为什么起不来、ro.hardware 是 mt8799、
repack 合并时的符号配对审计等）集中记录在 [MEDIATEK-NOTES.md](MEDIATEK-NOTES.md)，
联发科解密栈的完整审计在开发仓库的 `hybrid/decrypt-stack/FINDINGS.md`。

## 致谢

- cogan17 / JonesqPacMan——本设备树所改编的 rothko 设备树作者
- lingqiqi、YuKongA 等——YARP（TWRP 3.7.1_16 + GUI2）与 TWRP-Test 组织
- kirklin——Skyward Recovery 作者；小智乃敲可爱——Skyward 的 turner 适配，
  其镜像是本项目硬件通路的验证起点
- kodeaqua——taiko TWRP，首个公开的 mitee 解密先例
- TeamWin Recovery Project
