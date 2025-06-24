---
title: 'Ubuntu 实时内核安装 NVIDIA 驱动'
published: 2025-06-24T08:04:46.618Z
description: ''
updated: ''
tags:
  - Note
  - Linux
  - NVIDIA
draft: false
pin: 0
toc: true
lang: ''
abbrlink: ''
---

测试版本：ubuntu 22.04 + 5.15.0-1080-realtime + 535.230.02

步骤：
1. 下载 `NVIDIA .run` 驱动
2. 禁用 `nouveau` 驱动
3. 停用图形界面
4. 重启
5. 使用参数 `IGNORE_PREEMPT_RT_PRESENCE=1` 安装
6. 启用图形界面
7. 重启

```bash
curl -o nvidia-535.run -L https://cn.download.nvidia.com/XFree86/Linux-x86_64/535.230.02/NVIDIA-Linux-x86_64-535.230.02.run

echo "blacklist nouveau
options nouveau modeset=0" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf

sudo update-initramfs -u
sudo systemctl isolate multi-user.target

sudo IGNORE_PREEMPT_RT_PRESENCE=1 bash nvidia-535.run
sudo systemctl isolate graphical.target
```
