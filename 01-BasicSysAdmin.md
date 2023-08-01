# Basic system administration

## Periodic TRIM

References:

1. (Arch Wiki) [Periodic TRIM](https://wiki.archlinux.org/title/Solid_state_drive#Periodic_TRIM)

Run weekly (default) by activating its timer
```
systemctl enable fstrim.timer
```

## Internet

References:

1. (Arch Forum) [iwd/iwctl doesn't work anymore](https://bbs.archlinux.org/viewtopic.php?pid=1973099#p1973099)

Enable `iwd.service` with systemd

On boot `iwctl` will connect, but ping will fail.
```
ping archlinux.org
————
ping: archlinux.org: Temporary failure in name resolution
```

Ping to DNS will also fail
```
ping 8.8.8.8
————
ping: connect: Network is Unreachable
```

This happens due to a lack of [DHCP](https://wiki.archlinux.org/title/Network_configuration#DHCP) on the system

### Method #1

Activate iwd built-in DHCP with a config file
```
/etc/iwd/main.conf
————
[General]
EnableNetworkConfiguration=true

[Network]
NameResolvingService=systemd
```

Since it relies on systemd
```
systemctl enable systemd-resolved.service --now
```

### Method #2

References:
1. (Arch Wiki) [Dhdcpcd](https://wiki.archlinux.org/title/Dhcpcd)

Enable `dhdcpcd.service` with systemd

## Users

Create an user
```
useradd -m -G wheel rich
passwd rich
```

Enable wheel group sudo access (use `visudo`)
```
/etc/sudoers
————
-- # %wheel ALL=(ALL) ALL
++ %wheel ALL=(ALL) ALL
```

Example command as `root` user:
```
# command --flag
```

Example command as non-root user:
```
$ command --flag
```

## Firewall

References:

1. (Arch Wiki) [Uncomplicated Firewall](https://wiki.archlinux.org/title/Uncomplicated_Firewall)

Install Uncomplicated Firewall
```
# pacman -S ufw
```

Enable systemd service
```
# systemctl enable ufw.service --now
```

Set basic rules
```
# ufw default deny
# ufw allow from 192.168.0.0/24
# ufw limit ssh
```

Start firewall
```
# ufw enable
```

# Graphical Interface

## Video

### Desktop Environment and Display Manager

References:

1. (Arch Wiki) [xfce](https://wiki.archlinux.org/title/Xfce)

Install required packages
```
# pacman -S lxdm xfce4 xfce4-goodies
```

Edit config file for autologin
```
/etc/lxdm/lxdm.conf
————
-- # autologin=dgod
++ autologin=<username>
```

Execute graphical environment
```
# systemctl enable lxdm.service --now
```

### Hardware Video Acceleration

References:

1. (Arch Wiki) [Hardwarw video acceleration for Intel](https://wiki.archlinux.org/title/Hardware_video_acceleration#Intel)

Install VAAPI driver
```
# pacman -S intel-media-driver
```

## Audio

References:

1. (Arch Wiki) [pulseaudio over pipewire](https://wiki.archlinux.org/title/PipeWire#PulseAudio_clients)

Install required package
```
# pacman -S pipewire-pulse
```

Enable the user service
```
$ systemctl --user start pipewire-pulse
```

Install useful package
```
# pacman -S playerctl
```

## Graphical iwctl

References:

1. (Arch Wiki) [iwgtk](https://wiki.archlinux.org/title/Iwd#iwgtk)

Install the package
```
$ paru -S iwgtk
```

Prefer to logout and login to let xfce handle the process

## Graphical bluetooth

References:

1. (Arch Wiki) [bluetooth](https://wiki.archlinux.org/title/Bluetooth)
2. (Arch Wiki) [blueman](https://wiki.archlinux.org/title/Blueman)

Install the packages
```
# pacman -S bluez bluez-utils blueman pavucontrol
```

Enable the service
```
# systemctl enable bluetooth.service --now
```

Prefer to logout and login to let xfce handle the process

The output will not automatically change to the bluetooth device, open `pavucontrol` to change it manually

## Graphical Volume Manager

References:
1. (Arch Wiki) [Thunar Volume Manager](https://wiki.archlinux.org/title/Thunar#Thunar_Volume_Manager)
2. (xfce Forum) [ntfs3 support on thunar](https://forum.xfce.org/viewtopic.php?id=15643#p66393)

Install the package
```
# pacman -S gvfs gvfs-mtp
```

Restart the computer

Now all disks and network locations are available

Add support for `ntfs3` with udev rules
```
/etc/udev/rules.d/99-ntfs3.rules
————
SUBSYSTEM=="block", ENV{ID_FS_TYPE}=="ntfs", ENV{ID_FS_TYPE}="ntfs3"
```

# Package managment

## Arch User Repository

Install AUR helper
```
$ mkdir -p .local/aur && cd .local/aur
$ git clone https://aur.archlinux.org/paru-bin.git $ && cd paru-bin
$ makepkg -srci
```

## Repositories

Add `herecura`
```
/etc/pacman.conf
-----
++ [herecura]
++ # packages built against stable
++ Server = https://repo.herecura.be/herecura/x86_64
```

# Pending

## Migrate to unified kernel images

References:

1. (Arch manpages) [Unified Kernel Images](https://man.archlinux.org/man/systemd-boot.7.en)

Mount `/dev/nvme0n1p2` to `/efi`

Edit mkinitcpio to enable unified kernel images

Save u.k.i to `/efi/EFI/arch`

Check how this affects snapshots and subvolumes
