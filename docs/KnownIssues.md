# 已知问题与修复

> 返回 [主 README](../README.md)

### 1. macOS Tahoe 26.4+ 启动 Kernel Panic（已修复）

正常关机或重启后首次引导大概率触发 Kernel Panic，需强制重启一次。根因：Darwin 25.4.0 中内核线程 TSC 检查指令的偏移量由 0x488 移至 0x490，旧版 AMD 内核补丁的匹配掩码无法命中，补丁未生效。更新 AMD 内核补丁后彻底解决。
> 详情：[AMD_Vanilla PR #215](https://github.com/AMD-OSX/AMD_Vanilla/pull/215)

### 2. Intel 蓝牙连接 AirPods 无声（已修复）

macOS 的 `bluetoothd` 在建立 A2DP 通道时会读取蓝牙硬件时钟（HCI Read Clock），Intel 网卡无法支持或响应缓慢，超时后系统直接切断音频通道，表现为"已连接但无声音"。本 EFI 已集成修复版 `BlueToolFixup.kext` (v2.7.3) 并添加 `-btlfxa2dpcheck` 启动参数，理论支持 macOS 12 ～ 26 全版本。
> 已在 Tahoe 26.4.1 + AX200 + AirPods Pro 2 上验证。详情：[远景论坛帖子](https://bbs.pcbeta.com/viewthread-2068319-1-1.html)、[BlueToolFixup-FixA2DP](https://github.com/hexxyan/BlueToolFixup-FixA2DP)
>
> ⚠️ 本 EFI 内置的 BlueToolFixup 为**修改版**，请勿替换为官方版本，详见 [Kexts.md](./Kexts.md#不能跟随官方更新的-kext)。

### 3. 引导加载偶发卡住（未根治）

全版本通病：系统加载过程中偶尔卡住。可先按键盘任意键恢复引导，无效则强制重启。普遍怀疑与 AMD 双 XHCI 控制器的不稳定有关；本 EFI 已做 USB 定制并按版本区分控制器策略，发生率已降至最低。

### 4. Chromium / Electron 应用卡死（NootedRed 已知限制）

NootedRed 官方不支持 Chromium 系硬件加速，浏览器及 Electron 应用可能卡死或出现图形异常。绕过方法：

- 在 `chrome://flags` 中禁用 **GPU 光栅化（GPU rasterization）**，或以 `open -a 应用名称 --args --disable-gpu` 启动。
- **Sonoma+ 壁纸 / 视频解码挂起、反复 gpuRestart**：CoreMedia 的 Metal 传输会话与 VideoToolbox 硬解冲突，可通过禁用该偏好绕过（[NootedRed #235](https://github.com/ChefKissInc/NootedRed/issues/235)）：
  ```bash
  sudo defaults write /Library/Preferences/com.apple.coremedia allowMetalTransferSession -bool NO
  sudo chmod 644 /Library/Preferences/com.apple.coremedia.plist
  ```
  在恢复模式下操作时，将路径替换为 `/Volumes/<系统卷名>/Library/...`，改完重启生效。

### 5. macOS Tahoe 系统卡顿（Spotlight 索引）

Tahoe 上 Finder、右键菜单、打开应用卡顿的常见主因是 Spotlight 索引进程（`mds`）。缓解方法：

- 临时：`sudo mdutil -a -i off`，并在「聚焦」设置中将 `/Applications` 加入隐私排除。
- 永久（需在 Recovery 中关闭 SIP）：
  ```bash
  sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist
  sudo rm -rf /.Spotlight-V100
  ```
- 代价：失去系统级搜索，可改用 Raycast 等第三方工具。
> 详情：[远景论坛帖子](https://bbs.pcbeta.com/viewthread-2073020-1-1.html)

### 6. macOS 26 动画掉帧

可尝试 [CompositorPacer](https://github.com/githubzijian/CompositorPacer)：通过一个不可见的 Metal 节拍窗口保持 WindowServer 合成路径活跃，改善部分 macOS 26 环境下的动画掉帧感。
