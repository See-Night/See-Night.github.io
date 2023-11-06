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

```css
.a {
    width: 10px;
}
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
