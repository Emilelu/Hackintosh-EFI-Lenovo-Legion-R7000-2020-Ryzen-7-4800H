# Hackintosh-EFI-Lenovo-Legion-R7000-2020-Ryzen-7-4800H
Powered by SimpleKaruzi

## 一、必做配置（生成EFI后必须完成）
1.  引入必备Kext
    - USBMap.kext：通过 USBToolBox 生成，生成后必须使用 USBMap 脚本更新，确保USB端口正常识别与使用
    - HoRNDIS.kext：引入以支持USB网络共享功能
2.  声卡驱动关键操作（Tahoe系统）：引入AppleALC后，**必须将 layout-id=101 添加到 boot-args 中**，否则声卡无法正常工作

## 二、常见问题解决方案
### 程序卡死的解决方法（原因是NootedRed硬件加速不完美）
open -a Google\ Chrome --args --disable-gpu

### SwitchResX & BetterDisplay Crack 版失效？无法保存？
RDR恢复安装完成后，磁盘工具扩分区，添加的分辨率多保存几次（可以加几个刷新率低的）

### HiDPI
- one-key-hidpi-mod（仅在macOS 13 Ventura及以下可用？）
- BetterDisplay（有点卡，macOS 14 Sonoma以上可用）

### WIFI无法驱动？
- Tahoe：需要额外引入三个 kext，并OCLP-MOD打补丁
- Sonoma 14.4~Sequoia 15.X：使用的kext和其他版本不一样，Sequoia还需要OCLP-MOD打补丁
- 其他版本的macOS：直接使用对应版本的kext即可

### Tahoe声卡驱动？
引入AppleALC，**生成EFI后必须将 layout-id=101 添加到 boot-args**，并OCLP-MOD打补丁

## 三、系统专属Bug（原因未知）
- macOS Tahoe 26.4：每次开机都会panic，重启一次才能进系统；系统加载时偶尔突然卡死，需再次重启
- macOS 15 Sequoia：开机需按一下键盘按键才能继续启动

## 四、工具注册信息
### SwitchResX v4.x.x Serial Number
Name : Peter Gunn
Code  : 0079FCB69A1708F8