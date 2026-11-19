# 更新日志 — YARP Recovery for Redmi K Pad（turner）

本项目为红米 K Pad（`turner`，天玑 9400+ / MT6991，ro.hardware = mt8799）适配的
YARP（TWRP 16.0 / GUI2）第三方 Recovery。实现方式为「混合拼接」：以 HyperOS
OS4 原厂 vendor_boot 为底（F0/DTB 保持原厂），F1 放入 YARP 用户态并做严格库
隔离，解密与触控所需的 HAL 在运行时从已安装系统拉起，借用系统自带的
mitee TEE 链（KeyMint / Gatekeeper）完成凭据验证。

状态标记：✅ 已实机验证 / 🧪 已加入待验证 / ❌ 不支持（附原因）。

---

## v1.0（2026-11-19）— 首个正式版

**镜像**：`YARP-turner-K_PAD-vendor_boot-46428cf2.img`（64 MiB，vendor_boot v4）
**SHA256**：`e9ede605818609f9cbb7725feef7bbfc0490b15200f2d88d85ee87051af3714a`
**验证基线**：HyperOS OS4.0.3.0 正式版、槽位 A、图案锁

### 功能总览（全部实机验证）

- ✅ **数据显示与界面**：YARP/TWRP GUI2 原生界面完整显示（minui DRM legacy_modeset 补丁）
- ✅ **触摸**：Novatek NT36536E 驱动 + 小米 touchfeature HAL + THP 守护链，流畅无延迟
- ✅ **数据解密（完整打通）**：metadata 解密 + FBE DE/CE 解密；支持**图案 / PIN / 密码**；
  凭据经系统 mitee Gatekeeper 验证、KeyMint 授权令牌下发，与原厂完全同一条 TEE 链
- ✅ **USB OTG / U 盘**：自动挂载，可读写、导出日志
- ✅ **MTP 文件传输**：ConfigFS functionfs MTP（mtp,adb 复合模式）
- ✅ **ADB**：recovery 内 adb 可用；内置终端为 root 权限
- ✅ **振动反馈**：AIDL vibratorfeature HAL + AW8693x 执行器驱动
- ✅ **镜像刷入**：Install Image 新增 **Vendor Boot** 与 **DTBO** 条目（twrp.flags），dtbo 进入分区表
- ✅ **启动链防护**：卡刷包禁止写 lk / preloader 等启动链分区（by-name 路径重定向），保护 BL 解锁
- ✅ **开机速度**：并行/事件驱动启动，各服务就绪即退，无串行干等
- ✅ **系统兼容**：解密流程不破坏 /data 状态，退出 Recovery 后系统正常启动（见下方修复记录）
- ⏳ WiFi：未做（不影响 Recovery 核心功能）

### 自 v3.31 以来的关键变更（v3.32 → v1.0）

- **v3.36 元数据解密打通**：libturner_crypto 拦截 `AServiceManager_checkService`，
  对 NDK 查找失效的服务回退 legacy binder 查询；配合 additional.fstab 注入
  metadata 密文参数，metadata 解密成功、/data 挂载
- **v3.39 DE 密钥打通**：补齐镜像中缺失的 `vold_prepare_subdirs`，首次到达图案输入页
- **v3.40 凭据验证打通**：keystore2 的 init 服务名改为 vold 同步数据库所要求的
  字面名 `keystore2`；数据库同步成功，图案经 Gatekeeper 验证通过并取得 HAT
- **v1.0 授权环节补齐**：libturner_crypto 的 legacy 回退覆盖
  `android.security.authorization`，HAT 得以写入 keystore2，CE 解密全程贯通。
  （中途的独立 authz 库方案因拦截 waitForService 导致 recovery 段错误循环，已废弃；
  最终方案只保留 v3.36 起验证过的 checkService 单点拦截面）
- **v1.0 helper 重写**：`vold_prepare_subdirs` 改用 libselinux 官方 API
  （`selinux_android_file_context_handle` + `selinux_android_context_with_level`）
  解析完整上下文集合（含 vendor）并计算 MLS 等级。旧实现漏加载 vendor 上下文、
  把 /data/user_de/0 的标签改错，一度导致系统启动 init_user0 失败被 Rescue Party
  弹回 Recovery——已定位（pstore 日志实锤 vold 的 SELinux 拒绝记录）并修复，
  且新实现不再错误重标已有目录
- **v1.0 MTP**：F0 hw/init.rc 增加 ConfigFS MTP 触发组（联发科控制器路径），
  recovery 内 MTP 可用
- **v1.0 分区表补全**：recovery.fstab 增加 dtbo（slotselect）；twrp.flags 增加
  Vendor Boot / DTBO 的 flashimg 条目，Install Image 可直接刷写
- **v1.0 keystore2 监管重构**：改由 init 持有 keystore2 PID 并在 vold 播种
  tmpfs 数据库后重启它，同步/重启语义更稳；服务组对齐其数据库访问需求
- F0 与原厂对比仅两处差异：servicemanager 的 4 字节 VINTF NOP 补丁，
  以及 hw/init.rc 中 recovery 服务的库路径/LD_PRELOAD 与 MTP 触发组；
  DTB 与原厂逐字节一致（533,115 字节）；AVB 哈希描述符可复算通过

### 已知事项

- 载荷（官方 YARP 构建）不含上游 PR #42（GUI2 存储热插拔刷新，2026-11-06 合并）；
  基础 OTG 挂载不受影响，插入 U 盘后如列表未刷新，返回重进存储页即可
- 仅验证 HyperOS OS4.0.3.0；其他固件版本未逐一验证（HAL 运行时借用已装系统，
  理论上可随系统版本自适应，但不作保证）
- 本机无独立 recovery 分区，Recovery 位于 vendor_boot；请勿用 `fastboot boot` 测试

---

## 版本变更

- v1（2026-11-14）：首版拼接，卡开机画面（F0 无罪，后定位为 GUI2 DRM 门槛）
- v2.2：二进制补丁 libtwrpminui.so 强制 legacy_modeset —— 首次点亮 TWRP 界面（里程碑：显示）
- v2.4：触控模块加载器重写（依赖路径修复），触控驱动层打通（面板注册 1880×3008）
- v2.7：开机画面机型名改为 REDMI K Pad
- v3.5：xiaomi_touch 驱动模块打包进 recovery（不再依赖系统分区提取）
- v3.6：servicemanager VINTF 校验补丁（1 处 NOP），health 服务首次注册成功
- v3.9：HAL 注册全通（拆分 binder 对 + Accessor/trace 桩 + SM 补丁 + 干净 VINTF 清单）
- v3.10：HAL 改常驻服务，存活问题解决；setup 串行等待并行化（提速第一步）
- v3.12：触摸节点权限补齐 —— **触摸可用**（里程碑）
- 2026-11-16：U 盘/OTG 验证可用（里程碑）
- v3.13：KeyMint/Gatekeeper 改监督进程常驻；定位解密缺失层为 keystore2
- v3.14–v3.15：加入 keystore2；挂载 i18n APEX 补齐 ICU 依赖
- v3.16：liblogcapture v5 补 BBinder 桩，keystore2 首次连通 KeyMint（版本 300）
- v3.17：挂载 tracefs（排除 atrace 因素）
- v3.18：keystore2 换官方 YARP 构建自带二进制+完整库闭包（仍崩溃，排除二进制因素）
- v3.19：keystore2 改 init 服务（仍崩溃，排除启动方式与 memlock 因素）
- v3.20：加入崩溃回溯库 libfaultlog v1（信号结构与 bionic 不匹配，未生效）
- v3.21：libfaultlog v2（signal() 注册 + 装上自证标记）
- v3.22：ro.debuggable=1（adb root 尝试，用户批准）
- v3.23：镜像刷入新增 Vendor Boot / Init Boot；链分区刷包防护（lk/preloader 拦截）
- v3.24（stage40，sha `641c2ada…`，2026-11-17 07:13 CST 打好，64 MiB）：v3.23 全量 + 并行/事件驱动 setup（各 supervisor 背靠背启动、单一轮询环验活、keystore2 由 keymint 注册轮询触发，目标 5–6 秒）+ 振动 HAL supervisor 与 vintf 片段（8.0）+ persist 守卫（新增 haptics 目录；TEE 启动前复检挂载，防 TWRP 早期扫描卸载的竞态重挂）+ twrp.flags 定死 /usb-otg（显示 USB-OTG）。v3.23 待验项（adb root、FAULTLOG 回溯）由本版一并收。未进本版：版本号 99 大法（等 keystore2 活过来后才用）、WiFi、PR #42。**刷机验证（2026-11-17，平板 ✓）**：触摸 ✅；并行点火生效（readiness 0 ticks、内核 5.99 秒全员就绪）**但 setup 收尾拖到 9.57 秒**——keystore2 双重点火（等待器+内联各一次）+ sleep 3 验活吃了约 3.5 秒，体感比 v3.12 更慢（半死的 keystore2 让解密尝试干等，不如没有时失败得干脆）；**FAULTLOG 定案：keystore2 崩在 libhidlbase 的 getAllHalInstanceNames（找 hwservicemanager 枚举 HIDL，本机根本没有）**；振动 HAL 注册成功（SM 在清单中找到 vibratorfeature）但无振动——执行器节点 /sys/class/leds/vibrator 不存在（AW8693x 驱动模块没人加载）；adb root 未被拒绝（debuggable 生效，adbd 接受重启，是 USB 重枚举老毛病，kill-server+重插恢复）；U 盘仍是 /auto4-sdd-1（twrp.flags 改名未生效）；**中文文件名显示正常**；Install Image 列表有 Init Boot。
- v3.25（stage41，sha `951f61d0…`，2026-11-17）：liblogcapture v6 加 getAllHalInstanceNames 桩（返回空列表——本机无 HIDL HAL，是真话）；keystore2 点火去重 + 验活 sleep 3→1（砍收尾拖尾）；打包 OS4 vendor_dlkm 的 aw8693x_vibrator.ko，setup 在点火振动前 insmod + 振动节点权限放宽到 activate/duration/state。校验：F0 与 v3.24 零差异；F1 差异恰为 {turner-setup.sh, turner-vibrator-sup.sh, liblogcapture.so} + 新增 aw8693x_vibrator.ko；AVB 通过。**刷机验证（2026-11-17，平板 ✓）**：**振动 ✅**（用户实测）；**keystore2 段错误消失**（FAULTLOG 零 fatal，v6 桩生效，共享密钥协商成功）**但仍未存活**：错误链定案——它向 servicemanager 查「VINTF 声明实例」恒为空（清单里明明有 IKeyMintDevice/default，SM 自己日志实锤），于是判定「没有正牌 KeyMint」，转走老 HIDL keymaster 兼容服务（android.security.compat，本机不存在）→ NAME_NOT_FOUND → panic，监督进程每 ~8 秒拉起重来（34 次）；metadata 仍解不开，解密页仍是密码页。
- v3.26（stage42，sha `68aa294a…`，2026-11-17）：liblogcapture v7 加声明实例桩——`AServiceManager_forEachDeclaredInstance`/`isDeclared`/`getDeclaredInstances` 对 keymint/secureclock/sharedsecret/RPC 四个 mitee 接口按事实回答 `default`（其余接口转发真实现），把 keystore2 扳回 AIDL 正道；recovery.fstab 删光全部 `voldmanaged=` 行让 /usb-otg 独占认领 U 盘；新增 `/vendor_boot_img` 别名条目试探恢复 Install Image 里的 Vendor Boot。校验：F0 与 v3.25 零差异；F1 差异恰为 {liblogcapture.so, recovery.fstab}；AVB 通过。**刷机验证（2026-11-17，平板 ✓）**：声明桩生效——keystore2 改走正道（错误链变 `globals.rs:245 Trying to connect to genuine KeyMint service`），**但 getService 查 IKeyMintDevice/default 恒 NAME_NOT_FOUND**：keymint 进程本体活着（开机 pid 561 未换、单次点火），振动正常（同一套查服务机制能找到 vibratorfeature）→ 注册表没坏，卡点收窄到「载荷版 keystore2 查 mitee 服务」这一条路；**OTG 回退 ❌**：删 voldmanaged 后 U 盘干脆不挂载（twrp.flags 条目在本载荷上认领不了盘）；**别名 ❌** 未出现在镜像刷入列表。
- v3.27（stage43，sha `c985dba0…`，2026-11-17）：**keystore2 换回已安装系统的原版**（v3.14/v3.16 配方：/mnt/system/system/bin/keystore2 + twrpbinder→系统库→APEX→twrp16）——它是唯一真连上过 mitee KeyMint 的 keystore2（v3.16「current version (Some(300))」），它的两个历史死因（BBinder 符号、getAllHalInstanceNames 段错误）都已被 liblogcapture v5/v6 桩覆盖，v7 桩保留；recovery.fstab **原样恢复 v3.25**（撤销 v3.26 的 OTG/别名两个失败实验，U 盘回到可用的 /auto 挂载）。校验：F0 与 v3.26 零差异；F1 差异恰为 {turner-ks2-sup.sh, recovery.fstab}，且 fstab 与 v3.25 逐字节相同；AVB 通过。**刷机验证（2026-11-17，平板 ✓）：keystore2 链路全通**——开机 7.6 秒连上 KeyMint（current version (Some(300))）、注册 IKeystoreService、共享密钥协商成功、单次点火零崩溃；U 盘恢复挂载（/auto4-sdd-1）；**但 metadata 解密仍失败、仍是密码页**，且解密尝试全程 KeyMint 零流量。
- **解密真根因（2026-11-17 午间定位）**：读 TWRP-Test/android_system_vold 源码（MetadataCrypt.cpp）确认——原厂 vendor fstab 的 /data 行**没有 `metadata_encryption=` 声明**，vold 的 v2 解析路径拿空字符串当加密算法名，查表失败后**在调 keystore2 之前就直接退出**。这解释了 v3.13 以来每一版的「零流量」：keystore2 链路其实早已修好，vold 根本没走到它。metadata 钥匙文件本身完整（Terminal 实查：version=1、encrypted_key、keymaster_key_blob、secdiscardable 全在）。**手动验证**：在 v3.27 活会话里用 sed 给 /etc/additional.fstab 的 /data 行尾加 `,metadata_encryption=aes-256-xts` 成功落盘（TWRP 每次解密都重读这份拷贝）；但 GUI 提交密码不重跑 metadata（开机一次性），`ctl.restart recovery` 又只换来黑屏（已重启恢复）——最终验证留给 v3.28 的开机自动补丁。
- v3.28（stage44，sha `adc1ed2d…`，2026-11-17）：turner-setup.sh 新增 `patch_additional_fstab`——开机时（挂载后 + step:done 前各一次）给 /etc/additional.fstab 的 /data 行补 `metadata_encryption=aes-256-xts`，日志打 `additional fstab: metadata_encryption=aes-256-xts added to /data`。其余与 v3.27 完全一致。校验：F0 零差异；F1 差异恰为 {turner-setup.sh}；AVB 通过。**刷机验证（2026-11-17，平板 ✓）**：补丁按时落地（日志两行齐全），解密尝试用上了新 fstab，**但仍失败、仍无 KeyMint 流量**——解析关过了，倒在下一关。
- **第二关定位（2026-11-17 午间，活体探针实证）**：从官方 YARP 镜像提取其自带 `keystore_cli_v2`（与 vold 同辈的客户端，报错直接上屏）在 v3.28 活会话里跑 `list`，秒回 **`Unable to connect to Keystore.`**——它的源码路径是 `AServiceManager_checkService(IKeystoreService/default)` 返回空。keystore2 明明注册成功（SM 日志全程只有一句「Could not find android.system.keystore2.IKeystoreService/default in the VINTF manifest」）——**F1 缺 keystore2 的 VINTF 声明片段**，未声明的服务对客户查询不可见；而已声明的 keymint、vibratorfeature 都能正常查到，与此完全吻合。
- v3.29（stage45，sha `972a77b5…`，2026-11-17）：F1 新增 `system/etc/vintf/manifest/android.system.keystore2-service.xml`，逐字节取自官方镜像。**刷机验证（2026-11-17，平板 ✓）：失败且自伤**——dmesg 实锤：`Unrecognized manifest.version 9.0 (libvintf@8.0)` → 整个 recovery manifest 变 NULL。**教训：官方片段是 9.0，我们的 libvintf 只认 8.0；搬文件必须对齐版本号**（taiko 的版本毒药在我们这儿以另一种形式重演）。
- v3.30（stage46，sha `aa0606d8…`，2026-11-17）：同一片段，仅把 `version="9.0" type="framework"` 改为 `version="8.0" type="device"`，与 F1 其余七个片段完全同格式，声明内容不变。其余与 v3.29 完全一致。校验：F0 零差异；F1 仅该文件变化；AVB 通过。待验证：manifest 正常解析→IKeystoreService 可见→metadata 解开→图案页。
- **探针结论（2026-11-17 13:19，NDK 探针 libprobe.so 活体实测）**：在 payload 库环境的进程里，`AServiceManager_isDeclared` 对 keystore2/keymint/vibrator 三个服务全返回 1（声明可见），但 `AServiceManager_checkService` **对三个服务全返回 null**——包括明明能用的 vibratorfeature 和 keystore2 自己能连上的 keymint。即：**payload 世代的 NDK 查找路径在本机整体失效，与 keystore2 无关**；此前 keystore_cli_v2 的 "Unable to connect" 不能证明 vold 那一跳坏了。已装系统的 keystore2（SDK37 客户端库）查找 keymint 正常。差异焦点收窄为：客户端库世代 × 原厂（SDK35 世代）servicemanager 的互操作。
