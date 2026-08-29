# 可选优化与增强

> 返回 [主 README](../README.md)

## HiDPI

- `one-key-hidpi-mod`：仅兼容 Ventura 及以下。
- `BetterDisplay`：适配 Sonoma 及以上（激活工具：<https://macked.app/tools/betterdisplay.php>），运行时可能轻微卡顿。

## SwitchResX 注册信息

- Name：`Peter Gunn`
- Code：`0079FCB69A1708F8`

## 内建麦克风（实验性）

本 EFI 默认不含麦克风驱动。若需要内建数字麦克风，可尝试 [AMDMicrophone-Continuity](https://github.com/hrx114514x/AMDMicrophone-Continuity)（Renoir ACP PDM 麦克风驱动，含 DMA 连续性修复）。注意：

- 必须安装到 `/Library/Extensions`，不支持经 OpenCore 注入（因此本 EFI 不内置）；
- 需允许加载不受信任的 kext（`csr-active-config=01000000`，降低系统安全性）；
- 已在 Ryzen 5 5500U + macOS 26.5.1 验证；本机麦克风是否走 ACP/PDM 通道需自行确认。
