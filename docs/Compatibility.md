# 版本兼容性与 Wi-Fi 驱动

> 返回 [主 README](../README.md)

## 各版本差异速查

| macOS | USB 方案 | 蓝牙 | Wi-Fi | 其他 |
|:---|:---|:---|:---|:---|
| Mojave (10.14) ～ Catalina (10.15) | 禁用 XHC0 + `XhciPortLimit` | IntelBluetoothInjector | AirportItlwmMojave / Catalina | SecureBootModel = `Default` |
| Big Sur (11.x) | UTBDefault + `XhciPortLimit` | IntelBluetoothInjector | AirportItlwmBigSur | — |
| Monterey (12.x) | UTBDefault | BlueToolFixup | AirportItlwmMonterey | — |
| Ventura (13.x) ～ Sequoia (15.x) | UTBMap | BlueToolFixup | AirportItlwmVentura / Sonoma 分版 / AboveSequoia | Sequoia+ 需 OCLP-MOD 补丁 |
| Tahoe (26.x) | UTBMapTahoe | BlueToolFixup | AirportItlwmAboveSequoia | 需 OCLP-MOD 补丁 |

> **Catalina 及以下的代价**：无法使用为 Big Sur+ 修改的 `GenericUSBXHCI.kext`，双 XHCI 控制器互相冲突，必须通过 `SSDT-Disable_XHC0.aml` 禁用其中一个。左侧 USB-A、后侧 Type-C 及内置摄像头将不可用，此为获得稳定蓝牙与正常启动的必要代价。

> **Tahoe (26) 的 USB 映射**：Tahoe 内核（Darwin 25）的 USB 结构有变化，故使用单独定制的 `UTBMapTahoe.kext`（MinKernel 25.0.0）。

## Wi-Fi 驱动矩阵

AirportItlwm 通过 MinKernel / MaxKernel 自动匹配版本，**Sonoma 14.0–14.3 与 14.4+ 的驱动互不通用，切勿混用**：

| macOS | Kext | 内核版本 |
|:---|:---|:---|
| Mojave (10.14) | AirportItlwmMojave | 18.x |
| Catalina (10.15) | AirportItlwmCatalina | 19.x |
| Big Sur (11.x) | AirportItlwmBigSur | 20.x |
| Monterey (12.x) | AirportItlwmMonterey | 21.x |
| Ventura (13.x) | AirportItlwmVentura | 22.x |
| Sonoma 14.0–14.3 | AirportItlwmSonomaUpTo14dot3 | 23.0.0–23.3.99 |
| Sonoma 14.4+ | AirportItlwmSonomaAbove14dot4 | 23.4.0–23.99.99 |
| Sequoia (15.x) ～ Tahoe (26.x) | AirportItlwmAboveSequoia | 24.0.0+ |

Sequoia+ 额外加载 `IOSkywalkFamily`、`IO80211FamilyLegacy`（传统 Wi-Fi 框架，均 MinKernel 24.0.0），并需配合 OCLP-MOD 补丁。

> **关于 AirportItlwmAboveSequoia**：itlwm 官方 release 目前最新仅提供到 Ventura 版构建，Sequoia+ 只能沿用该构建。本 EFI 中的 `AirportItlwmAboveSequoia.kext` 即为作者将 Ventura 版构建重命名所得，二进制与 `AirportItlwmVentura.kext` 相同。

## Tahoe 音频

AppleALC 已集成，但 Tahoe 底层变更后需配合 OCLP-MOD 补丁，音频设备才能正常唤醒。
