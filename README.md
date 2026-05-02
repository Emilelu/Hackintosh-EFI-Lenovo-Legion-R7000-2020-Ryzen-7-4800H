# Hackintosh-EFI-Lenovo-Legion-R7000-2020-Ryzen-7-4800H

## 配置

| 组件 | 型号 / 名称 | 备注 |
|:---|:---|:---|
| 主板 | LENOVO LNVNB161216 | AMD 平台笔记本，联想拯救者 R7000 2020 |
| BIOS | EUCN41WW (2023-09-20) | x64-based，UEFI，Secure Boot 已禁用 |
| CPU | AMD Ryzen 7 4800H (Renoir) | 8 核心 16 线程，支持 AVX2 |
| 独立显卡 | NVIDIA GeForce GTX 1650 | 设备 ID: 10DE-1F99，macOS 下无驱动，屏蔽处理 |
| 核显 | AMD Radeon(TM) Graphics (Renoir) | 设备 ID: 1002-1636，使用 UMAF 修改显存至 2G |
| 内建显示器 | LQ156T1JW05 | 2560x1440 / 165Hz，连接至核显，使用 SwitchResX 修改刷新率、BetterDisplay 开启 HiDPI |
| 有线网卡 | Realtek PCIe GbE Family Controller | 设备 ID: 10EC-8168 |
| 无线网卡 | Intel(R) Wi-Fi 6 AX200 160MHz | 设备 ID: 8086-2723 |
| 蓝牙 | Intel(R) Wireless Bluetooth(R) | USB 接口，设备 ID: 8087-0029，使用修复版 BlueToolFixup.kext 解决 AirPods 无声问题 |
| 声卡 | Realtek(R) Audio | 设备 ID: 10EC-0257，layout-id 需设为 101 |
| NVMe SSD 1 | SKHynix_HFS512GD9TNI-L2A0B | PC611 NVMe 控制器，512GB，不兼容 macOS，屏蔽处理 |
| NVMe SSD 2 | WD Blue SN570 500GB SSD (DRAM-less) | SanDisk Ultra 3D / WD Blue SN570 控制器 |
| 触摸板 | SYNA2BA6 | I2C HID 设备 |
| 键盘 | Standard PS/2 Keyboard | FUJ7401 |

---

## 一、前言

理论上支持 macOS Catalina (10.15) ~ macOS Tahoe (26.x)。

本 EFI 基于 SimpleKaruzi 生成。  
生成时，有关 USB 的 Kext 仅勾选 `GenericUSBXHCI.kext` 及 `USBToolBox.kext`，UTBMap.kext 需通过 USBToolBox (Windows.exe 工具) 需按 `C` 更改设置以生成（若生成 USBMap.kext native class 则 identifier 为 SMBIOS ProductionName）。  
生成后，需修改 `config.plist` 的 NVRAM 中的 `boot-args` 为：

```
-v debug=0x100 keepsyms=1 ipc_control_port_options=0 -lilubetaall -amfipassbeta -vi2c-force-polling alcid=101
```

### 1. 额外驱动引入

- 所有 Kext 应确保为最新 Dev 版。
- `USBMap.kext`：USB 端口映射。Tahoe 以下版本可直接使用生成的 kext，Tahoe 则需通过 USBMap 脚本更新。
- `HoRNDIS.kext`：驱动 Android USB 网络共享。

### 2. 音频驱动关键配置

加载 AppleALC 驱动后，**必须在 `boot-args` 中添加 `alcid=101`**，否则声卡无法正常运作。

### 3. BIOS 相关设置

- 启用 Switchable Graphics（双显卡切换）
- UMAF GFX 2G+（核显显存配置）
- 关闭 AMD SVM、TPM 及 Secure Boot

## 二、常见问题解决方案

### 1. 应用卡死问题

**故障成因**：NootedRed 硬件加速适配不完善。

**解决方案**：  
- 方案一：终端执行  
  ```bash
  open -a 拖入 App --args --disable-gpu
  ```
- 方案二：借助 GLFriend 工具修复权限隔离  
  ```bash
  sudo spctl --master-disable
  sudo xattr -r -d com.apple.quarantine 拖入 glfriend
  ```

### 2. SwitchResX & BetterDisplay 配置无法保存

完成 RDR 恢复安装后，通过磁盘工具扩展分区容量；多组分辨率配置重复保存，可适当添加低刷新率方案以提升保存成功率。

### 3. HiDPI 适配方案

- `one-key-hidpi-mod`：仅兼容 macOS 13 Ventura 及以下版本。
- `BetterDisplay`：适配 macOS 14 Sonoma 及以上版本，运行存在轻微卡顿。

### 4. 无线网卡驱动注意事项

- **Sequoia 15+**：沿用 Ventura 版本 Kext，额外增补三款驱动（IO80211 / Skywalk...），并搭配 OCLP-MOD 完成补丁修复。
- **Sonoma 14.0~14.3 & 14.4+**：两大版本驱动互不通用。
- **其它 macOS 版本**：直接部署对应系统版本的 `AirportItlwm.kext` 即可。

### 5. Tahoe 音频驱动注意事项

集成 AppleALC 驱动，搭配 OCLP-MOD 补丁完成底层兼容修复。

## 三、系统专属兼容异常（原因未知）

- **macOS Tahoe 26**：正常关机或重启后首次引导大概率触发 Kernel Panic，需再强制重启一次方可继续引导
- **全版本通病**：系统加载有时会卡死或 Kernel Panic，有时需强制重启、有时按任意键即可继续引导。（怀疑是 AMD USB XHCI 不稳定导致）

## 四、工具注册与资源链接

### 1. SwitchResX v4.x.x 注册信息

- Name：`Peter Gunn`
- Code：`0079FCB69A1708F8`

### 2. BetterDisplay

激活工具地址：<https://macked.app/tools/betterdisplay.php>