---
title: 在小新 13 Pro 上安装 Linux Desktop 环境
categories:
    - Linux
date: '2020-06-10T14:37:13+08:00'
---
最近在尝试有啥舒服的开发环境，将 Windows 更新到了 2004 之后启用 WSL 2 还是不能满足我的大部分需求，而 MacOS 对于某些开发工具支持有问题（说的就是你 Forge Gradle）。最后不如直接尝试一下开源 ~~拖拉机~~, 直接在实体机上安装 Linux 作为日用系统。

<!--more-->

![System details](https://i.loli.net/2020/06/10/XwPjD2lumSvqbLo.png)

~~AMD YES!~~

## 安装 ArchLinux

这一步非常简单，安装过多次 Arch 的话应该已经非常熟练了，这里采用的 UEFI 启动

```bash
# Root 目录在 /mnt, efi 文件夹挂载到 /mnt/boot
pacstrap /mnt base linux linux-firmware
genfstab -U /mnt >> /mnt/etc/fstab
```

然后 `arch-chroot /mnt` 到我们的新系统里并且执行

```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
passwd # set root password
```

重启就可以引导到硬盘上的 Arch 中了

### 安装 KDE

这里采用的 KDE 作为桌面环境 ~~我个人挺喜欢 KDE 的, GNOME 快滚~~，根据 ArchLinux Wiki 上的[安装手册](https://wiki.archlinux.org/index.php/KDE)，只需要

```
pacman -S plasma kde-applications
systemctl enable sddm
```

然后重启就可以看到 SDDM 的登录界面了

### 安装其他字体

为了在 Linux 下更好的显示大部分的字体, 我安装了 [ttf-ms-win10 - aur](https://aur.archlinux.org/packages/ttf-ms-win10/)  这个包, 需要提取出 Windows 的字体然后安装

```bash
7z e Win10_2004_Chinese\(Simplified\)_x64.iso sources/install.wim
7z e install.wim 1/Windows/{Fonts/"*".{ttf,ttc},System32/Licenses/neutral/"*"/"*"/license.rtf} -ofonts/
cd fonts
wget 'https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=ttf-ms-win10' -O PKGBUILD
makepkg --skipchecksums
sudo pacman -U *.tar.xz
```

## 适配 HiDPI 屏幕

### 设定 KDE 和 SDDM 缩放

在 KDE 中进入 System Settings -> Display and Monitor 中可以设定缩放，我设定的值为 150% 缩放。

![Set glolbal scale](https://i.loli.net/2020/06/10/1zfCoXTa8yJdUYu.png)

然后去 Startup and Shutdown 的 Login Screen (SDDM) 下面的 Advanced 选项卡中，点击 Sync 来同步缩放设定到 SDDM 中

![Sync DPI with SDDM](https://i.loli.net/2020/06/10/ag5pcoxHsPhObut.png)

### 调整 GRUB 字体大小

```bash
# pacman -S ttf-iosevka
grub-mkfont -s 30 -o /boot/grub/fonts/unicode.pf2 /usr/share/fonts/ttf-iosevka/iosevka-regular.ttf
```

可以将 `/usr/share/fonts/ttf-iosevka/iosevka-regular.ttf` 更换为你看着舒服的字体, 重启之后 GRUB 的字体就不是那么的难受了

## 配置 IR 传感器进行人脸识别解锁

这台机器上是有 IR 传感器的，在 Windows 下可以通过 Windows Hello 进行人脸识别快速解锁。

已经有现成的开源项目实现了 Linux 上的  Windows Hello style 人脸解锁: [Howdy](https://github.com/boltgolt/howdy)

一开始我发现 `/dev/video2` 是一个可以用的摄像头，但是因为没有发出红外光而读取不到任何信息，在 [Howdy #269](https://github.com/boltgolt/howdy/issues/269) 上发现了如何去启用摄像头的 IR-LED

在终端中执行（执行后重启也有效果，只需要执行一次）

```bash
cd /tmp
git clone https://github.com/PetePriority/chicony-ir-toggle.git
gcc main.c -o chicony-ir-toggle
./chicony-ir-toggle -d /dev/video2 on
```

就可以在使用摄像头的时候启用 IR-RED ~~不知道联想为什么要这么做~~

记得调整 Howdy 的 `dark_threshold` 参数为一个合适的值，我个人使用的是 90，也可以调整的更大一些

等到 `howday test` 测试能识别到人脸并通过 `howdy add` 添加人脸之后, 我们需要配置 Linux 的 PAM 登录来达到人脸识别登录的效果

## 启用蓝牙

默认安装完是不会启用蓝牙的，需要手动启用 systemd 的服务才能使用蓝牙

```bash
systemctl enable --now bluetooth
```

启用之后 KDE 的托盘部分就会显示蓝牙图标了

为了使用蓝牙耳机，还需要安装 PulseAudio 的蓝牙模块

```
pacman -S pulseaudio-bluetooth
pluseaudio -k
pactl load-module module-bluetooth-discover
```

## 配置中文输入法

作为一个中文为母语的人，当然要先配置中文输入法啦，在 KDE 上使用中文输入法也非常简单

```bash
pacman -S fcitx fcitx-rime

cat > ~/.pam_environment <<EOF
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
EOF
```

然后就可以重启后在 KDE 任务栏发现 fcitx 的图标了，区域切换到中文就可以添加 RIME 输入法了。RIME 在默认的主题中输入中文会有些问题，可以自己去找一个喜欢的主题（我现在在使用 [fcitx-skin-material](https://github.com/hrko99/fcitx-skin-material)）

![Fcitx Input](https://i.loli.net/2020/06/10/qayKLB2RAdeIf3m.png)

