## 引言

之前我装双系统的时候我就发现了一个问题，从Archlinux切回Windows的时候会出现时钟错乱的问题。不过我一直没管（）

后面有空的时候研究了下，发现是两个系统对**RTC（硬件时钟）**的定义不一致导致的这个问题

## 解决方案

演示环境：

- `Windows 11 25H2`
- `ArchLinux`

这里的原理就是将Windows的时间改为**UTC（世界标准时间）**

1. 在Windows中，按下 `Win + X`并选择 `终端管理员`
2. 运行命令 `Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1`
3. 修改后手动同步一次系统时间或者重启系统

以上的操作将会修改Windows的注册表，使Windows采用和Linux相同的策略

## 引用说明

<https://github.com/SHORiN-KiWATA/Shorin-ArchLinux-Guide>

<https://www.cnblogs.com/born2run/articles/16634450.html>
