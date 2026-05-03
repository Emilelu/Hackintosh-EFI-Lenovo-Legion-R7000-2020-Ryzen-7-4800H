# Hackintosh-EFI-Lenovo-Legion-R7000-2020-Ryzen-7-4800H

## 配置

| 组件 | 型号 / 名称 | 备注 |
|:---|:---|:---|
| 主板 | LENOVO LNVNB161216 | AMD 平台笔记本，联想拯救者 R7000 2020 |
| BIOS | EUCN41WW (2023-09-20) | x64-based，UEFI，Secure Boot 已禁用（BIOS 内） |
| CPU | AMD Ryzen 7 4800H (Renoir) | 8 核心 16 线程，支持 AVX2 |
| 独立显卡 | NVIDIA GeForce GTX 1650 | macOS 无解，已通过 SSDT 屏蔽 |
| 核显 | AMD Radeon(TM) Graphics (Renoir) | 已通过 UMAF 修改显存至 4G（推荐≥1G） |
| 内建显示器 | LQ156T1JW05 | 2560x1440 @ 165Hz，连接至核显（自行更换） |
| 有线网卡 | Realtek PCIe GbE Family Controller | 设备 ID: 10EC-8168 |
| 无线网卡 | Intel® Wi-Fi 6 AX200 160MHz | 设备 ID: 8086-2723 |
| 蓝牙 | Intel® Wireless Bluetooth® | 设备 ID: 8087-0029，已集成修复版 BlueToolFixup 解决 AirPods 无声问题 |
| 声卡 | Realtek® Audio | 设备 ID: 10EC-0257，layout-id = 101 |
| NVMe SSD（出厂） | SKHynix_HFS512GD9TNI-L2A0B | PC611 控制器，macOS 不兼容，已通过 SSDT 屏蔽 |
| NVMe SSD（加装） | WD Blue SN570 500GB | SanDisk / WD Blue SN570 控制器，作为系统盘 |
| 触摸板 | SYNA2BA6 | I2C HID 设备 |
| 键盘 | Standard PS/2 Keyboard | FUJ7401 |

---

## 一、前言

本项目为 Lenovo Legion R7000 2020 款（AMD Ryzen 7 4800H）量身定制的 OpenCore EFI，基于 SimpleKaruzi 生成并经过深度调校，力求 **开箱即用**。

各大版本安装成果截图详见 /Screenshots 文件夹。

### 版本拆分说明

由于不同 macOS 版本对 USB 控制器、显卡补丁及驱动要求存在显著差异，原通用配置已拆分为 **三个专用配置文件**。请根据你当前使用的系统版本选择对应的文件，将其重命名为 `config.plist` 后使用：

| 配置文件 | 适用系统 | 关键差异 |
|:---|:---|:---|
| `config_Catalina.plist` | macOS Catalina (10.15.x) | 需禁用 XHCI0 控制器；开启 `XhciPortLimit`；SecureBootModel = `Default` |
| `config_BigSur.plist` | macOS Big Sur (11.x) | 需开启 `XhciPortLimit`；使用 `UTBDefault.kext` |
| `config.plist`（默认） | macOS Monterey (12.x) ～ Tahoe (26.x) | Monterey 使用 `UTBDefault.kext`；Ventura ~ Sequoia 使用 `UTBMap.kext`；Tahoe 使用 `UTBMapTahoe.kext` |

> **关于 SecureBootModel**：仅在 Catalina 配置中设为 `Default`（否则无法正常加载驱动），Big Sur 及以上版本均为 `Disabled`。

**理论支持范围**：macOS Catalina (10.15.x) ~ macOS Tahoe (26.x)。低于 Catalina 的版本（如 High Sierra、Mojave）因显卡无驱动，未做测试与适配。

### 通用启动参数（boot-args）

```
-v debug=0x100 keepsyms=1 ipc_control_port_options=0 -lilubetaall -amfipassbeta -vi2c-force-polling alcid=101 -btlfxa2dpcheck
```

各参数作用简述：

| 参数 | 作用 |
|:---|:---|
| `-v` | 显示详细启动日志 |
| `debug=0x100 keepsyms=1` | 内核调试参数，便于排查问题 |
| `ipc_control_port_options=0` | 对某些 Electron 及应用进行优化 |
| `-lilubetaall` | 允许 Lilu 及其插件在不支持的操作系统版本上加载 |
| `-amfipassbeta` | 允许 AMFIPass 在不支持的操作系统版本上加载（仅 Sequoia+ 需要） |
| `-vi2c-force-polling` | 强制 I2C 触摸板轮询模式，提升兼容性 |
| `alcid=101` | 注入声卡 layout-id |
| `-btlfxa2dpcheck` | 修复 Intel 蓝牙 + AirPods 无声问题（仅 Monterey+ 生效） |

所有参数均已做兼容处理，低版本系统会自动忽略不支持的部分，无需手动删除。

---

### 已集成的核心驱动

| Kext | 用途 | 适用版本 |
|:---|:---|:---|
| `Lilu.kext` | 基础补丁引擎，所有插件依赖 | 全部 |
| `VirtualSMC.kext` | 模拟 SMC 芯片 | 全部 |
| `SMCBatteryManager.kext` | 电池电量显示 | 全部 |
| `SMCLightSensor.kext` | 环境光传感器 | 全部 |
| `NootedRed.kext` | AMD Renoir 核显驱动 | 全部 |
| `AppleALC.kext` | 声卡驱动 (alcid=101) | 全部 |
| `NVMeFix.kext` | NVMe SSD 电源管理优化 | 全部 |
| `RealtekRTL8111.kext` | 有线网卡驱动 | 全部 |
| `AirportItlwm` 系列 | Intel Wi-Fi 驱动 | 按系统版本自动匹配对应 kext |
| `IntelBluetoothFirmware.kext` | 蓝牙固件 | 全部 |
| `IntelBTPatcher.kext` | 蓝牙补丁 | 全部 |
| `IntelBluetoothInjector.kext` | 蓝牙注入 (Big Sur 及以下) | Catalina / Big Sur |
| `BlueToolFixup.kext` | 蓝牙修复 (含 AirPods 无声修复) | Monterey+ |
| `GenericUSBXHCI.kext` | AMD XHCI 控制器驱动 | Big Sur+ |
| `USBToolBox.kext` | USB 映射工具 | Big Sur+ |
| `UTBDefault.kext` | USB 默认映射 | Big Sur / Monterey |
| `UTBMap.kext` | USB 定制映射 | Ventura ~ Sequoia |
| `UTBMapTahoe.kext` | USB 定制映射 (Tahoe) | Tahoe |
| `AMFIPass.kext` | 允许 OCLP-MOD 打补丁 | Sequoia+ |
| `IOSkywalkFamily.kext` | 网络框架补丁 | Sequoia+ |
| `IO80211FamilyLegacy.kext` | 传统 Wi-Fi 框架 | Sequoia+ |
| `RestrictEvents.kext` | 修复 CPU 名称显示等 | 全部 |
| `YogaSMC.kext` | 联想笔记本 Fn 键优化 | 全部 |
| `BrightnessKeys.kext` | 亮度快捷键支持 | 全部 |
| `ForgedInvariant.kext` | TSC 同步修复 | 全部 |
| `AppleMCEReporterDisabler.kext` | 禁用 MCE 报告（避免 AMD Panic） | 全部 |
| `CtlnaAHCIPort.kext` | SATA 控制器兼容补丁 | Big Sur+ |
| `SATA-unsupported.kext` | SATA 控制器兼容补丁 | Catalina |
| `HoRNDIS.kext` | Android USB 网络共享 | 全部 |

### BIOS 设置要求

使用本 EFI 前，请确保 BIOS 已做如下设置：

- **启用** Switchable Graphics（双显卡切换）
- **UMAF GFX**：显存设置为 1G（推荐 2G 及以上）
- **建议关闭** AMD SVM、TPM 及 Secure Boot

### ⚠️ NVMe 兼容性提醒

本机出厂硬盘为海力士 PC611（macOS 不兼容），EFI 已通过 `SSDT-Disable_NVMe_GPP1.aml` 将其屏蔽，另加装 WD Blue SN570 作为系统盘。使用时请注意：

- **若你的出厂硬盘同为海力士（或同系列不兼容型号）**：无需任何改动，直接使用本 EFI 即可。
- **若你的出厂硬盘为其他兼容 macOS 的型号（如 WD、Samsung 等）**：请禁用或删除 `EFI/OC/ACPI/SSDT-Disable_NVMe_GPP1.aml`，否则该硬盘位将无法识别。

---

## 二、系统兼容性详解

### 1. macOS Catalina (10.15.x) 特殊注意事项

Catalina 无法使用为 Big Sur 11+ 修改的 `GenericUSBXHCI.kext`，因此多 XHCI 控制器无法同时驱动，必须采取以下措施：

- **禁用 XHC0 控制器**：启用 `SSDT-Disable_XHC0.aml`，关闭第二个 XHCI 控制器
- **开启 `XhciPortLimit`**：`config_Catalina.plist` 中已设为 `true`
- **SecureBootModel = `Default`**：已配置（否则无法正常加载某些驱动）

> ⚠️ **禁用 XHC0 的代价**：左侧 USB-A 口、后侧 Type-C 口及内置摄像头将不可用。这是 Catalina 获得稳定蓝牙与正常启动的必要代价，无法绕过。

### 2. macOS Big Sur (11.x) 特殊注意事项

- **必须开启 `XhciPortLimit`**：`config_BigSur.plist` 中已设为 `true`
- **必须使用 `UTBDefault.kext`**：因 `UTBMap.kext` 的 `MinKernel` 设为 22.0.0 (Ventura)，Big Sur (20.x) 不会加载

### 3. macOS Monterey (12.x) 特殊注意事项

- **使用 `UTBDefault.kext`**：与 Big Sur 相同
- **`XhciPortLimit` 已关闭**：`false`
- **蓝牙驱动切换**：`IntelBluetoothInjector.kext` 的 `MaxKernel` 为 20.99.99，Monterey (21.x) 不会加载；`BlueToolFixup.kext` 的 `MinKernel` 为 21.0.0，接管蓝牙

### 4. macOS Ventura (13.x) ～ macOS Tahoe (26.x)

- **USB 映射切换**：
  - Ventura (22.x) ~ Sequoia (24.x)：`UTBMap.kext`（MinKernel 22.0.0）
  - Tahoe (25.x)：`UTBMapTahoe.kext`（MinKernel 25.0.0，因 Tahoe 的 USB 结构有变化）
- **AMFIPass.kext**：MinKernel 为 24.0.0，仅在 Sequoia+ 加载，用于配合 OCLP-MOD 打补丁
- **IOSkywalkFamily / IO80211FamilyLegacy**：同样仅在 Sequoia+ 加载，用于 Wi-Fi 补丁

---

## 三、已知问题修复记录

### 1. macOS Tahoe 26.4+ 开机 Kernel Panic（已修复）

> 问题详情与修复方案：[AMD_Vanilla Pull #215](https://github.com/AMD-OSX/AMD_Vanilla/pull/215)

正常关机或重启后首次引导大概率触发 Kernel Panic，必须强制重启一次后才能正常进入系统。经过排查，确认为 AMD 内核补丁在 Tahoe 26.4 及更新版本上的兼容性问题。更新 AMD 内核补丁（特别是 `non-monotonic time panic` 相关补丁，增加了 `laobamac` 为 26.4+ 提供的新版本）后，该问题彻底解决。

### 2. Intel 蓝牙连接 AirPods 无声（已修复）

> 问题详情与修复方案：[【全球首发】彻底解决 Airpods 蓝牙连接上但无声音的 bug](https://bbs.pcbeta.com/viewthread-2068319-1-1.html)

此问题的根源在于 macOS 的 `bluetoothd` 进程在建立 A2DP 音频通道时，会尝试读取蓝牙硬件时钟 (`HCI_Read_Clock`)。Intel 网卡无法支持或响应缓慢，导致系统判定超时并直接切断音频通道，表现为“已连接但无声音”。**本 EFI 已集成修复版 `BlueToolFixup.kext` (v2.7.3) 并添加启动参数 `-btlfxa2dpcheck`**，连接 AirPods 即可正常播放。

> 此修复已在 macOS Tahoe 26.4.1 + Intel AX200 + AirPods Pro 2 上验证通过，理论支持 macOS 12 至 26 全版本。

### 3. 系统加载偶发卡死（已知问题，原因未明）

**本 EFI 已进行 USB 端口定制并针对不同系统版本使用不同的控制器策略**，理论上已将问题发生率降至最低。

全版本通病，具体表现为系统加载过程中有时会卡住，若偶尔遭遇此问题，请尝试按键盘任意键恢复引导，无效则需强制重启。

普遍怀疑与 AMD USB XHCI 控制器的不稳定有关，尤其是在双 XHCI 控制器的机型上（联想拯救者 R7000 2020 即为双控制器设计）。

---

## 四、技术细节（供参考）

### 1. 无线网卡驱动方案

Intel AX200 在不同 macOS 版本上需要使用匹配的 `AirportItlwm.kext`，本 EFI 已通过 `MinKernel` / `MaxKernel` 实现自动化版本匹配：

- **macOS Catalina (10.15.x)**：使用 `AirportItlwmCatalina.kext`（内核 19.x）
- **macOS Big Sur (11.x)**：使用 `AirportItlwmBigSur.kext`（内核 20.x）
- **macOS Monterey (12.x)**：使用 `AirportItlwmMonterey.kext`（内核 21.x）
- **macOS Ventura (13.x)**：使用 `AirportItlwmVentura.kext`（内核 22.x）
- **macOS Sonoma 14.0~14.3**：使用 `AirportItlwmSonomaUpTo14dot3.kext`（内核 23.0.0 ~ 23.3.99）
- **macOS Sonoma 14.4+**：使用 `AirportItlwmSonomaAbove14dot4.kext`（内核 23.4.0 ~ 23.99.99）
- **macOS Sequoia 15+**：沿用 Ventura 版 Kext（`AirportItlwmAboveSequoia.kext`），额外增补 `IO80211FamilyLegacy.kext`、`IOSkywalkFamily.kext`，并搭配 OCLP-MOD 补丁完成 Wi-Fi 修复

> **注意**：Sonoma 14.0~14.3 与 14.4+ 的驱动互不通用，切勿混用。

### 2. Tahoe 音频驱动方案

本 EFI 已集成 AppleALC 驱动，但由于系统底层变更，需搭配 OCLP-MOD 补丁完成底层兼容修复，否则音频设备无法被正确唤醒。

### 3. 某些应用卡死（Chromium / Electron 系应用）

**原因**：NootedRed 硬件加速适配不完善，Chromium 系应用调用的 OpenGL 双源混合等图形接口与驱动存在兼容性问题，导致卡死。

**临时绕过方案**：
- 终端启动应用并禁用 GPU 加速：
  ```bash
  open -a 应用名称 --args --disable-gpu
  ```
- 或使用 GLFriend 工具修复应用权限隔离：
  ```bash
  sudo spctl --master-disable
  sudo xattr -r -d com.apple.quarantine /路径/glfriend
  ```

### 4. HiDPI 工具参考

- `one-key-hidpi-mod`：仅兼容 macOS 13 Ventura 及以下版本。
- `BetterDisplay`：适配 macOS 14 Sonoma 及以上版本（激活工具：<https://macked.app/tools/betterdisplay.php>），运行时可能存在轻微卡顿。

### 5. SwitchResX 注册信息

- Name：`Peter Gunn`
- Code：`0079FCB69A1708F8`

### 6. USB 映射方案总览

由于不同 macOS 版本对 USB 控制器的处理方式不同，本 EFI 使用了三套 USB 映射方案，通过 `MinKernel` / `MaxKernel` 自动切换：

| Kext | 适用系统 | MinKernel | MaxKernel | 说明 |
|:---|:---|:---|:---|:---|
| `UTBDefault.kext` | Big Sur ~ Monterey | 20.0.0 | 21.99.99 | 通用默认映射，包含常用端口 |
| `UTBMap.kext` | Ventura ~ Sequoia | 22.0.0 | 24.99.99 | 针对高版本系统定制，包含蓝牙内建 |
| `UTBMapTahoe.kext` | Tahoe | 25.0.0 | — | Tahoe USB 结构变更后重新定制 |

> **关于 `GenericUSBXHCI.kext`**：该驱动是为 Big Sur 11+ 修改的 AMD XHCI 控制器驱动，Catalina 无法使用。Catalina 改用 `SSDT-Disable_XHC0.aml` 禁用第二个 XHCI 控制器，配合 `XhciPortLimit` 来实现 USB 功能。

### 7. XHCI 双控制器说明

联想拯救者 R7000 2020 采用双 XHCI 控制器设计，在 macOS 下存在天然兼容问题：

- **正常情况（Big Sur 及以上）**：`GenericUSBXHCI.kext` 可同时驱动两个控制器，所有 USB 端口均可用，USB 定制只需保证端口数量不超限即可。
- **Catalina 的特殊情况**：由于无法使用修改版 `GenericUSBXHCI.kext`，两个 XHCI 控制器会互相冲突，导致蓝牙等 USB 设备无法正常工作。**必须通过 `SSDT-Disable_XHC0.aml` 禁用其中一个控制器**，代价是部分物理 USB 端口失效（左侧 USB-A、后侧 Type-C、内置摄像头）。

---

## 后记

从 Chameleon 到 Clover，从 Clover 到 OpenCore。
从 Yosemite 到 Tahoe。

十年前初次接触黑苹果时，笔者尚在小学阶段，折腾的第一台机器是联想 IdeaPad Y460N。该机型不支持 UEFI 引导，只能依靠 Chameleon，或通过模拟 EFI 的方式使用 Clover。彼时若能成功装上一个可用的 Hackintosh，便足以欣喜许久。

日后设备几经更迭，从 Intel 平台到 AMD 平台，始终以笔记本为主，每一台都曾用于黑苹果的尝试与研究。

如今 Apple Silicon 已迭代数代，黑苹果的时代行将落幕。这块拼了十年的拼图，终于在 macOS Tahoe 26 上拼完了最后一块。

**谨以此帖，纪念十年青春，画上完美的终止符。**

感谢一路上所有开源开发者的付出，没有前辈们的无私奉献，黑苹果不会有今天的高度。

黑果将死，但折腾永存。

愿这份 EFI 能帮助到仍在坚守黑苹果的各位。

---

*本 EFI 已涵盖目前所有已知问题的修复。因系统版本差异较多，请务必选择对应的配置文件。如遇到新问题，欢迎提交 Issue & Pull Request。*
