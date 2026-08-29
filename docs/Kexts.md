# Kext 一览与维护须知

> 返回 [主 README](../README.md)

## 已集成 Kext

| 类别 | Kext | 版本 | 用途 / 适用版本 |
|:---|:---|:---|:---|
| 基础 | `Lilu` | 1.7.3 | 补丁引擎，所有插件的前置依赖 |
| | `VirtualSMC` | 1.3.8 | SMC 模拟（含 `SMCBatteryManager` 电池、`SMCLightSensor` 光线传感器） |
| | `ForgedInvariant` | 1.5.0 | TSC 不变量修复 |
| | `AppleMCEReporterDisabler` | 1.2 | 禁用 MCE 报告，避免 AMD 平台 Kernel Panic |
| | `RestrictEvents` | 1.1.7 | 修复 CPU 名称显示等 |
| 核显 | `NootedRed` | 0.8.10 | Renoir 核显驱动（Mojave+） |
| | `BrightnessKeys` | 1.0.4 | 亮度快捷键 |
| 音频 | `AppleALC` | 1.9.8 | 声卡驱动（`alcid=101`） |
| 有线网 | `RealtekRTL8111` | 3.0.4 | Realtek 网卡驱动 |
| 无线网 | `AirportItlwm`（8 个版本） | 2.3.0 | 按 macOS 版本自动匹配，见 [Compatibility.md](./Compatibility.md#wi-fi-驱动矩阵) |
| | `IOSkywalkFamily` / `IO80211FamilyLegacy` | 1.0 / 1200.12.2b1 | 传统 Wi-Fi 框架，仅 Sequoia+，配合 OCLP-MOD |
| 蓝牙 | `IntelBluetoothFirmware` / `IntelBTPatcher` | 2.5.0 | Intel 蓝牙固件与补丁 |
| | `IntelBluetoothInjector` | 2.5.0 | ≤ Big Sur（MaxKernel 20.99.99） |
| | `BlueToolFixup`（修复版） | 2.7.3 | Monterey+，含 AirPods 无声修复 ⚠️ 勿更新，见下文 |
| USB | `GenericUSBXHCI` | 1.3.0b2 | AMD XHCI 控制器驱动，Big Sur+ ⚠️ 修改版勿更新 |
| | `USBToolBox` | 1.2.0 | USB 映射框架，Big Sur+ |
| | `UTBDefault` / `UTBMap` / `UTBMapTahoe` | 1.0 / 1.1 / 1.1 | 本机定制映射，对应 Big Sur～Monterey / Ventura～Sequoia / Tahoe |
| 存储 | `NVMeFix` | 1.1.4 | NVMe 电源管理与兼容性修复（见下文说明） |
| | `CtlnaAHCIPort` / `SATA-unsupported` | 341.0.2 / 0.9.2 | SATA 控制器支持（Big Sur+ / Catalina 及以下） |
| 输入 | `VoodooPS2Controller` | 2.3.8 | PS/2 键盘 |
| | `VoodooI2C` + `VoodooI2CHID` | 2.9.1 | I2C 触摸板 |
| 其他 | `YogaSMC` | 1.5.3 | 联想 Fn 功能键 |
| | `AMFIPass` | 1.4.1 | 仅 Sequoia+，允许 OCLP-MOD 打补丁 |
| | `HoRNDIS` | 9.2 | Android USB 网络共享 |
| 备用 | `PC711Probe` | 1.8.1 | SK hynix NVMe panic 修复，**默认禁用**，见下文 |
| | `PC711ProbeForce` | 1.8.1 | 同上 Force 版，供标准版不匹配的型号使用，**默认禁用**，与标准版不可同时启用 |

## 不能跟随官方更新的 Kext

以下 kext 为修改版或本机定制产物，**更新后会失去修复 / 映射，直接导致功能异常**：

| Kext | 原因 |
|:---|:---|
| `BlueToolFixup.kext` 2.7.3 | 来自 [BlueToolFixup-FixA2DP](https://github.com/hexxyan/BlueToolFixup-FixA2DP) 的修复版，含 A2DP 时钟读取补丁；官方版无此修复，替换后 AirPods 将再度无声（配套 `-btlfxa2dpcheck` 也会失效） |
| `GenericUSBXHCI.kext` 1.3.0b2 | 为 Big Sur+ 修改的 AMD XHCI fork；官方 RehabMan 版无 AMD 适配 |
| `UTBDefault` / `UTBMap` / `UTBMapTahoe` | 由 USBToolBox 生成的**本机定制映射**，不是上游发布物 |
| `AirportItlwmAboveSequoia.kext` | 作者将 itlwm 官方 **Ventura 版构建重命名**（官方 release 无 Sequoia+ 版本）；更新时应下载最新 Ventura 版并保持该重命名 |
| `IOSkywalkFamily` / `IO80211FamilyLegacy` | 取自 OCLP 的框架补丁，需与 OCLP-MOD 版本配套 |
| `CtlnaAHCIPort` / `SATA-unsupported` / `AppleMCEReporterDisabler` | 无官方 release 渠道的社区维护版，维持现状 |

## 可更新 Kext 的现状（2026-08 核对）

其余 kext 均可跟随官方更新。核对结果：

- **已是最新**：VoodooI2C 2.9.1、YogaSMC 1.5.3、USBToolBox 1.2.0、HoRNDIS 9.2（官方早已停更）、RealtekRTL8111 3.0.x、AirportItlwm 系列（= itlwm v2.3.0 stable，最新 release；其中 `AirportItlwmAboveSequoia` 为作者将 Ventura 版构建重命名用于 Sequoia+）。
- **本地版本高于最新正式 release**（CI 构建版，无需降级）：Lilu 1.7.3、VirtualSMC 1.3.8、AppleALC 1.9.8、NVMeFix 1.1.4、RestrictEvents 1.1.7、BrightnessKeys 1.0.4、VoodooPS2Controller 2.3.8、IntelBluetoothFirmware 系列 2.5.0。
- **上游走 CI 构建、无正式 release**（当前为可得最新）：NootedRed 0.8.10、ForgedInvariant 1.5.0、AMFIPass 1.4.1。
- **OpenCore 本体**：已更新至 **1.0.7**（2026-03-20），替换了 `BOOTx64.efi`、`OpenRuntime.efi`、`OpenCanopy.efi`、`ResetNvramEntry.efi`；`HfsPlus.efi` 已是 OcBinaryData 最新版。三份 config 均通过 ocvalidate (1.0.7) 校验，并顺带移除了无效键 `HideVerbose`。

更新方法：从各官方仓库 release 下载，整体替换 `EFI/OC/Kexts/<名称>.kext` 目录即可，config 无需改动（BundlePath 不变）。

## PC711Probe / PC711ProbeForce（备用，默认禁用）

[PC711Probe](https://github.com/hrx114514x/PC711Probe)（v1.8.1）为 Lilu 插件，修复旧版 macOS（11–15）上部分 NVMe 的 Identify 超时 panic（macOS 26 已原生支持 PC711）。官方实机验证过 SK hynix PC711 / BC711 / BC511 与三星 PM991；**据网友反馈，其他型号的 NVMe 也能正常工作**——标准版可匹配的型号直接用 `PC711Probe.kext`，未在验证列表的型号改用 `PC711ProbeForce.kext` 即可。已在 AMD 平台 + macOS 15.6.1 完成全新安装与 96 GiB 读写压测。

- 本机系统盘 SN570 工作正常、无此问题，两者均默认 `Enabled=False`。
- 若需要用（如出厂盘位为 PC711/BC711，或想免屏蔽尝试其他不兼容硬盘）：在 config 的 Kernel → Add 中启用二者之一（**标准版与 Force 版绝不可同时启用**），并按需处理 `SSDT-Disable_NVMe_GPP1.aml`（见主 README）。是否兼容请自行测试，做好备份。

## NVMeFix 是否需要？

**建议保留**。系统盘 WD SN570 为非 Apple NVMe，NVMeFix 提供其电源管理（APST）修复，可降低睡眠唤醒时 NVMe 掉盘 / panic 的概率；与 PC711Probe、SSDT 屏蔽方案互不冲突，开销可忽略。
