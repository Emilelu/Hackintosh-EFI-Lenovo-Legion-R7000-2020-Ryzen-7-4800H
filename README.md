# Hackintosh-EFI-Lenovo-Legion-R7000-2020-Ryzen-7-4800H

## 配置

| 组件 | 型号 / 名称 | 备注 |
|:---|:---|:---|
| 主板 | LENOVO LNVNB161216 | AMD 平台笔记本，联想拯救者 R7000 2020 |
| BIOS | EUCN41WW (2023-09-20) | x64-based，UEFI，Secure Boot 已禁用 |
| CPU | AMD Ryzen 7 4800H (Renoir) | 8 核心 16 线程，支持 AVX2 |
| 独立显卡 | NVIDIA GeForce GTX 1650 | 设备 ID: 10DE-1F99，macOS 下无驱动，已屏蔽 |
| 核显 | AMD Radeon(TM) Graphics (Renoir) | 设备 ID: 1002-1636，已通过 UMAF 修改显存至 4G |
| 内建显示器 | LQ156T1JW05 | 2560x1440 / 165Hz，连接至核显（非原屏，自行更换） |
| 有线网卡 | Realtek PCIe GbE Family Controller | 设备 ID: 10EC-8168 |
| 无线网卡 | Intel(R) Wi-Fi 6 AX200 160MHz | 设备 ID: 8086-2723 |
| 蓝牙 | Intel(R) Wireless Bluetooth(R) | 设备 ID: 8087-0029，已集成修复版 BlueToolFixup 解决 AirPods 无声问题 |
| 声卡 | Realtek(R) Audio | 设备 ID: 10EC-0257，layout-id 已配置为 101 |
| NVMe SSD（出厂） | SKHynix_HFS512GD9TNI-L2A0B | PC611 控制器，macOS 不兼容，已通过 SSDT 屏蔽 |
| NVMe SSD（加装） | WD Blue SN570 500GB SSD | SanDisk / WD Blue SN570 控制器，用作系统盘 |
| 触摸板 | SYNA2BA6 | I2C HID 设备 |
| 键盘 | Standard PS/2 Keyboard | FUJ7401 |

---

## 一、前言

本 EFI 基于 SimpleKaruzi 生成，已针对 **Lenovo Legion R7000 2020 (Ryzen 7 4800H)** 完成全部配置与优化，**开箱即用**。

- **已注入的 `boot-args`**：`-v debug=0x100 keepsyms=1 ipc_control_port_options=0 -lilubetaall -amfipassbeta -vi2c-force-polling alcid=101 -btlfxa2dpcheck`
- **理论支持范围**：macOS Catalina (10.15) ~ macOS Tahoe (26.x)
- **USB 定制**：已集成定制好的 USB 端口映射，用户无需额外操作
- **Kext 版本**：所有驱动均已更新至最新 Dev 版

### 已集成的额外驱动

| Kext | 用途 |
|:---|:---|
| `UTBMap.kext` | USB 端口映射 |
| `HoRNDIS.kext` | Android USB 网络共享 |
| 修复版 `BlueToolFixup.kext` | 解决 Intel 蓝牙连接 AirPods 无声问题 |

### BIOS 要求

使用本 EFI 前，请确保 BIOS 已做如下设置：
- **启用** Switchable Graphics（双显卡切换）
- **UMAF GFX 4G**（核显显存配置）
- **推荐关闭** AMD SVM、TPM 及 Secure Boot

### ⚠️ 使用前请注意 NVMe 兼容性

本机出厂硬盘为海力士 PC611（macOS 不兼容），已通过 `SSDT-Disable_NVMe_GPP1.aml` 将其屏蔽，另加装 WD Blue SN570 作为系统盘。使用时请注意：

- **若您的出厂 NVMe 同样为海力士（或同款不兼容型号）**：无需任何改动，直接使用本 EFI 即可
- **若您的出厂 NVMe 为其他兼容 macOS 的型号（如 WD、三星等）**：请禁用或删除 `EFI/OC/ACPI/SSDT-Disable_NVMe_GPP1.aml`，否则该硬盘位将无法识别

---

## 二、系统兼容性说明

### 1. macOS Tahoe 26.4+ 开机 Panic（已修复）

> 问题详情与修复方案：[AMD_Vanilla Pull #215](https://github.com/AMD-OSX/AMD_Vanilla/pull/215)

此问题表现为正常关机或重启后首次引导大概率触发 Kernel Panic，需强制重启后才能进入系统。**本 EFI 已应用最新的 AMD 内核补丁，此问题不再出现。**

### 2. Intel 蓝牙连接 AirPods 无声（已修复）

> 问题详情与修复方案：[【全球首发】彻底解决 Airpods 蓝牙连接上但无声音的 bug](https://bbs.pcbeta.com/viewthread-2068319-1-1.html)

此问题根源在于 macOS 的 `bluetoothd` 进程建立 A2DP 音频通道时，会读取蓝牙硬件时钟 (`HCI_Read_Clock`)，Intel 网卡不支持或响应缓慢导致音频通道被切断。**本 EFI 已集成修复版 `BlueToolFixup.kext` 并添加启动参数 `-btlfxa2dpcheck`，连接 AirPods 即可正常播放。**

> 此修复已在 macOS Tahoe 26.4.1 + Intel AX200 + AirPods Pro 2 上验证通过，理论支持其他 macOS 版本。

### 3. 系统加载偶发卡死（已知问题，原因未明）

**本 EFI 已定制 USB 端口映射，理论上问题发生率已降至最低**，但系统加载过程中有时仍会卡住，按任意键可继续引导，若无效需强制重启。此问题为全版本通病，普遍怀疑与 AMD USB XHCI 控制器不稳定有关。

---

## 三、技术细节（供参考）

### 1. 无线网卡驱动方案

- **Sequoia 15+**：沿用 Ventura 版 Kext，额外增补 IO80211 / Skywalk 系列驱动，搭配 OCLP-MOD 补丁
- **Sonoma 14.0~14.3 & 14.4+**：两大版本驱动互不通用，已分别适配
- **其它版本**：使用对应系统版本的 `AirportItlwm.kext`

### 2. Tahoe 音频驱动方案

已集成 AppleALC 驱动，搭配 OCLP-MOD 补丁完成底层兼容修复。

### 3. 某些应用卡死（Chromium / Electron 系应用）

**原因**：NootedRed 硬件加速适配不完善。

**临时绕过方案**：
- 终端执行：`open -a 应用名称 --args --disable-gpu`
- 或使用 GLFriend 工具修复：
  ```bash
  sudo spctl --master-disable
  sudo xattr -r -d com.apple.quarantine /路径/glfriend
  ```

### 4. HiDPI 工具参考

- `one-key-hidpi-mod`：仅兼容 macOS 13 Ventura 及以下
- `BetterDisplay`：适配 macOS 14 Sonoma 及以上（激活工具：<https://macked.app/tools/betterdisplay.php>）

### 5. SwitchResX 注册信息

- Name：`Peter Gunn`
- Code：`0079FCB69A1708F8`

---

*本 EFI 已集成了目前所有已知问题的修复。如遇到新问题，欢迎提交 Issue & Pull Request。*
