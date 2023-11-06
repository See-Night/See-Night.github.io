---
title: 记一次 Debian 12 Gnome 桌面美化过程
date: 2023-11-05 22:48:02
tags:
- Linux
- Debian
- 美化
categories:
- 记录
---

> 写在前面：
>
> Canonical 越来越不当人，因为那个弱智 Snap 所以放弃 Ubuntu 转 Debian 了。
>
> 以前主力机都使用 Ubuntu 的，没用过正统的 Debian，导致在使用初期遇到了巨大的瓶颈。
>
> 本篇文章目的是将 Debian 12 的折腾过程记录下来，为以后再遇到类似问题做经验储备。

<!-- more -->

## Grub 优化

作为 Linux 单系统用户，Grub 的引导界面不能说毫无用处吧，只能说屁用没有，还浪费时间。

可以通过修改 `/etc/default/grub` 跳过 Grub 引导界面。

```shell
GRUB_TIMEOUT=0
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash loglevel=0"
```

修改 `/etc/default/grub` 后运行以下命令更新 Grub：

```shell
sudo update-grub
```

## Plymouth

在跳过 Grub 后，会发现在 splash 界面有个巨丑的背景。这方面真的要表扬 Ubuntu，Ubuntu 的 splash 界面采用了 Plymouth 的 `Bgrt` 主题，这种加载动画更加现代化。

![ubuntu plymouth](/images/Debian-12-Gnome-桌面美化/ubuntu-plymouth.png)

我们可以通过手动安装 Plymouth 主题来达到同样的效果。

```shell
sudo apt install plymouth-themes
```

然后你就会在 `/usr/share/plymouth/themes` 下发现 `bgrt` 主题的文件夹。

修改 `/usr/share/plymouth/plymouthd.defaults`：

```
[Daemon]
Theme=bgrt
ShowDelay=0
DeviceTimeout=8
```

修改 `/etc/plymouth/plymouthd.conf`：

```
[Daemon]
Theme=bgrt
ShowDelay=0
```

更新 `initramfs`：

```shell
sudo update-initramfs -u
```

重启计算机，你就能看到以下效果：

![debian plymouth](/images/Debian-12-Gnome-桌面美化/debian-plymouth.png)

> 截图为虚拟机截图，实机上会有主板的 Logo。

## Nvidia 驱动

> FUCK YOU NVIDIA !!

Debian nonfree 提供的 `Nvidia-driver` 是 `525.125.06` 版本的，在使用过程中，笔者发现这个版本在 wayland 环境下的表现简直就是一坨，Steam 的用户界面打开都在一直闪，推荐尽可能安装最新版的驱动。

在 [Nvidia](https://www.nvidia.com/Download/index.aspx?lang=en-us) 官网下载最新版本的 Linux 图形驱动：

![nvidia](/images/Debian-12-Gnome-桌面美化/nvidia.png)

下载安装脚本，赋予权限：

```bash
chmod +x NVIDIA-Linux-x86_64-535.129.03.run
```

由于 Nvidia 驱动安装时需要彻底关闭所有的图形界面，所以推荐使用 SSH 或者切换到其他 tty 进行安装。

```bash
sudo ./NVIDIA-Linux-x86_64-535.129.03.run
```

第一次安装时，会提示现在系统中正在使用 Nouveau 驱动，需要先禁用这个驱动才能安装。然后请求是否要创建配置去禁用 Nouveau，选择 `Yes` 即可。脚本禁用 Nouveau 后，会提示你需要重启系统后重新运行安装脚本。但是在这之前，需要先更新一下 `initramfs` 才能使上面的配置生效：

```bash
sudo update-initramfs -u
reboot
```

重启系统后再次运行安装脚本，一路 `Yes` 即可。安装好后，可以进入系统中运行 `nvidia-smi` 去查看驱动版本：

```bash
$ nvidia-smi
Mon Nov  6 20:36:49 2023       
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.129.03             Driver Version: 535.129.03   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA GeForce GTX 1050        Off | 00000000:01:00.0 Off |                  N/A |
| N/A   30C    P8              N/A / ERR! |    141MiB /  2048MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+
                                                                                         
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A      1716      G   /usr/bin/gnome-shell                          1MiB |
|    0   N/A  N/A      4889      G   ...local/share/Steam/ubuntu12_32/steam        2MiB |
|    0   N/A  N/A      5455      G   ...re/Steam/ubuntu12_64/steamwebhelper       22MiB |
|    0   N/A  N/A      5492      G   ...eam/logs/cef_log.txt --shared-files      113MiB |
+---------------------------------------------------------------------------------------+
```

## Wayland

在安装好 Nvidia 显卡驱动后，会发现原本应该是 Wayland 的桌面环境被替换为了 X11，而且无法通过修改 `/etc/gdm3/daemon.conf` 中的 `WaylandEnable=true` 来开启 Wayland 登陆选项。

需要修改 `/usr/lib/udev/rules.d/61-gdm.rules`，注释掉 `#RUN+="/usr/lib/gdm-runtime-config set daemon PreferredDisplayServer xorg"` 和 `#RUN+="/usr/lib/gdm-runtime-config set daemon WaylandEnable false"` 两行：

```shell
LABEL="gdm_prefer_xorg"
#RUN+="/usr/lib/gdm-runtime-config set daemon PreferredDisplayServer xorg"
GOTO="gdm_end"

LABEL="gdm_disable_wayland"
#RUN+="/usr/lib/gdm-runtime-config set daemon WaylandEnable false"
GOTO="gdm_end"
```

然后重启计算机。

> 该方法来源于 [Ask Ubuntu](https://askubuntu.com/questions/1403854/cant-use-wayland-with-nvidia-510-drivers-on-ubuntu-22-04-lts)

> FUCK YOU NVIDIA !!!
