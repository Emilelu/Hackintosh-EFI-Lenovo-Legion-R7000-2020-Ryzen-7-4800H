# Hackintosh-EFI-Lenovo-Legion-R7000-2020-Ryzen-7-4800H

Lenovo 拯救者 R7000 2020（AMD Ryzen 7 4800H）专用 OpenCore EFI（OpenCore 1.0.7），基于 SimpleKaruzi 的 EFI 深度调校，力求开箱即用。

- **理论支持范围**：macOS Mojave (10.14) ～ Tahoe (26)，低于 Mojave 的版本未做测试。
- 各版本安装成果截图见 [Screenshots](./Screenshots)。
- 本 EFI 引用的全部外部链接汇总于 [Links.txt](./Links.txt)。

## 文档导航

| 文档 | 内容 |
|:---|:---|
| [docs/Compatibility.md](./docs/Compatibility.md) | 各 macOS 版本差异速查、Wi-Fi 驱动矩阵、Tahoe 音频 |
| [docs/KnownIssues.md](./docs/KnownIssues.md) | 已知问题与修复（Kernel Panic、AirPods、卡顿等） |
| [docs/Kexts.md](./docs/Kexts.md) | Kext 一览、维护须知（⚠️ 含不可更新清单）、NVMeFix 说明 |
| [docs/Optional.md](./docs/Optional.md) | 可选优化（HiDPI、内建麦克风等） |

## 硬件配置

| 组件 | 型号 / 名称 | 备注 |
|:---|:---|:---|
| 主板 | LENOVO LNVNB161216 | AMD 平台笔记本 |
| BIOS | EUCN41WW (2023-09-20) | UEFI，Secure Boot 已在 BIOS 内禁用 |
| CPU | AMD Ryzen 7 4800H (Renoir) | 8 核心 16 线程 |
| 独立显卡 | NVIDIA GeForce GTX 1650 | macOS 无驱动，已通过 `SSDT-Disable_GPU_GPP0.aml` 屏蔽 |
| 核显 | AMD Radeon(TM) Graphics (Renoir) | 已通过 UMAF 解锁并将显存设为 4G（NootedRed 最低 512M，建议 ≥1G） |
| 内建显示器 | LQ156T1JW05 | 2560x1440 @ 165Hz，连接核显（自行更换），EDID 已注入 config |
| 有线网卡 | Realtek RTL8168/8111 | 设备 ID `10EC-8168` |
| 无线网卡 | Intel Wi-Fi 6 AX200 | 设备 ID `8086-2723` |
| 蓝牙 | Intel Wireless Bluetooth | 设备 ID `8087-0029`，AirPods 无声问题已修复（见 [KnownIssues.md](./docs/KnownIssues.md#2-intel-蓝牙连接-airpods-无声已修复)） |
| 声卡 | Realtek ALC257 | 设备 ID `10EC-0257`，layout-id = 101 |
| NVMe SSD（出厂） | SK hynix HFS512GD9TNI-L2A0B (PC611) | macOS 不兼容，已通过 SSDT 屏蔽（见下文） |
| NVMe SSD（加装） | WD Blue SN570 500GB | 系统盘 |
| 触摸板 | SYNA2BA6 | I2C HID，VoodooI2C 轮询模式驱动 |
| 键盘 | PS/2 (FUJ7401) | VoodooPS2Controller 驱动 |

## BIOS 设置

- **启用** Switchable Graphics（双显卡切换）
- UMA Frame Buffer Size ≥ **1G**
- **关闭** Secure Boot；建议同时关闭 AMD SVM 与 TPM

## 配置文件选择

不同 macOS 版本对 USB 控制器、显卡补丁及驱动要求差异较大，故拆分为三份配置。**使用前将所选文件重命名为 `config.plist`**：

| 配置文件 | 适用系统 | 关键差异 |
|:---|:---|:---|
| `config_Mojave2Catalina.plist` | Mojave (10.14) ～ Catalina (10.15) | 启用 `SSDT-Disable_XHC0.aml` 禁用第二 XHCI 控制器；开启 `XhciPortLimit`；SecureBootModel = `Default` |
| `config_BigSur.plist` | Big Sur (11.x) | 开启 `XhciPortLimit`；USB 使用 `UTBDefault.kext` |
| `config.plist`（默认） | Monterey (12.x) ～ Tahoe (26.x) | Monterey 使用 `UTBDefault.kext`；Ventura ～ Sequoia 使用 `UTBMap.kext`；Tahoe 使用 `UTBMapTahoe.kext`（经 MinKernel/MaxKernel 自动切换） |

> 出厂海力士硬盘的屏蔽 SSDT（`SSDT-Disable_NVMe_GPP1.aml`）**仅在默认配置中启用**；Mojave2Catalina 与 BigSur 配置中该条目默认关闭，如有需要请手动开启。

### boot-args

```
-v debug=0x100 keepsyms=1 ipc_control_port_options=0 -lilubetaall -amfipassbeta -vi2c-force-polling alcid=101 -btlfxa2dpcheck
```

| 参数 | 作用 |
|:---|:---|
| `-v debug=0x100 keepsyms=1` | 详细启动日志与内核调试信息，便于排错 |
| `ipc_control_port_options=0` | 改善 Electron 应用兼容性 |
| `-lilubetaall` | 允许 Lilu 及其插件在所有系统版本加载 |
| `-amfipassbeta` | 允许 AMFIPass 加载（配合 OCLP-MOD） |
| `-vi2c-force-polling` | I2C 触摸板强制轮询 |
| `alcid=101` | 声卡 layout-id 注入 |
| `-btlfxa2dpcheck` | 修复 Intel 蓝牙 + AirPods 无声（Monterey+ 生效） |

三份配置的 boot-args 相同；低版本系统会忽略不适用参数，无需手动删改。

## NVMe 屏蔽说明

`SSDT-Disable_NVMe_GPP1.aml` 的作用是**屏蔽出厂硬盘位**（GPP1）。本机出厂海力士 PC611 正插在该硬盘位且不兼容 macOS，故默认配置已启用该 SSDT（Mojave2Catalina / BigSur 配置中默认关闭）。

- 海力士原装盘（或同系列不兼容型号）仍插在该硬盘位：保持 SSDT 启用即可。
- 该硬盘位更换为 macOS 兼容硬盘（WD、Samsung 等）：禁用或删除该 SSDT，否则此硬盘位无法识别。
- **替代方案**：也可尝试启用内置的 `PC711Probe.kext` / `PC711ProbeForce.kext` 免除屏蔽，详见 [Kexts.md](./docs/Kexts.md#pc711probe--pc711probeforce备用默认禁用)。

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
