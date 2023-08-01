# First installation

A lusk2 encrypted btrfs over SSD with `systemd-boot`.

References:

1. (Arch Wiki) [Installation Guide](https://wiki.archlinux.org/index.php?title=Installation_guide)
2. (Blog) [Archlinux on encrypted btrfs with systemd-boot and KDE](https://rich.grundy.io/blog/archlinux-on-encrypted-btrfs-with-systemd-boot-and-kde/)
3. (Github) [RichGuk's Archlinux install](https://github.com/RichGuk/dotfiles/blob/master/INSTALL.md)
4. (Arch Wiki) [SSD](https://wiki.archlinux.org/title/Solid_state_drive#Security)

## Arch Live Environment

### Connect to the internet

References:

1. (Arch Wiki) [iwctl](https://wiki.archlinux.org/index.php?title=Iwd&oldid=732938#iwctl)

Enter the program's shell

```
iwctl
```

List all wireless devices
```
device list
```

Initiate scan for networks, doesn't output anything
```
station <device> scan
```

List all available networks
```
station <device> get-networks
```

Connect to a network
```
station <device> connect <SSID>
```

Enter passphrase
```
<passphrase>
```

Verify internet connection
```
ping archlinux.org
```

Ensure the system clock is accurate
```
timedatectl set-ntp true
```

### Partition the disks

References:

1. (Btrfs Wiki) [Subvolume Layout](https://btrfs.wiki.kernel.org/index.php/SysadminGuide#Layout)
2. (Btrfs Wiki) [Manage Spanshots](https://btrfs.wiki.kernel.org/index.php/SysadminGuide#Managing_Snapshots)
3. (Arch Wiki) [Suggested filesystem layout](https://wiki.archlinux.org/title/Snapper#Suggested_filesystem_layout)
4. (Librredit) [Suggested layout](https://libreddit.nl/r/btrfs/comments/nd6bnd/what_are_the_advantages_of_including_boot_in/)
5. (Librredit) [Suggested layout 2](https://libreddit.nl/r/btrfs/comments/sgw45u/what_btrfs_subvolumes_to_exclude_from_snapshots/)
6. (Arch Wiki) [Dm-crypt with btrfs subvolumes](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#Btrfs_subvolumes_with_swap)
7. (Arch Wiki) [Cryptsetup](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Cryptsetup_usage)
8. (Btrfs Readthedocs) [btrfs-man](https://btrfs.readthedocs.io/en/latest/btrfs-man5.html)

Recognize block devices designated to disks
```
fdisk -l
```

Target device is called `/dev/nvme0n1`

Modify partition tables
```
gdksk /dev/nvme0m1
```

Create a new empty GUID partition table (GTP) and confirm operation

Add a new partition (number 1) of `550M` with hexcode `ef00`

Add a new partition (number 2) of the rest of the disk with default hexcode `8300`

Write table to disk and exit

Encrypt /dev/nvme0n1p2 (second partition)
```
cryptsetup --type luks2 luksFormat /dev/nvme0n1p2
```

Confirm the operation and set a passphrase
```
cryptsetup luksOpen /dev/nvme0n1p2 cryptroot
```

Format the partitions
```
mkfs.fat -F32 -n ESP /dev/nvme0n1p1
mkfs.btrfs -L Arch /dev/mapper/cryptroot
```

Mount the btrfs into the live system
```
mount -o compress=zstd,noatime /dev/mapper/cryptroot /mnt
```

Create btrfs subvolumes for FHS
```
btrfs subvol create /mnt/@
btrfs subvol create /mnt/@home
btrfs subvol create /mnt/@swap
```

Create btrfs subvolumes for snapshots
```
mkdir /mnt/snapshots
btrfs subvol create /mnt/snapshots/@
btrfs subvol create /mnt/snapshots/@home
```

Mount `@` inside `/mnt`
```
umount /mnt
mount -o compress=zstd,noatime,subvol=@ /dev/mapper/cryptroot /mnt
```

Mount `@home` subvolume inside `/mnt/home`
```
mkdir -p /mnt/home
mount -o compress=zstd,noatime,subvol=@home /dev/mapper/cryptroot /mnt/home
```

Mount `/dev/nvme0n1p1` partition inside `/mnt/boot`
```
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```

Create btrfs subvolumes for var
```
mkdir -p /mnt/var
btrfs subvol create /mnt/var/cache
btrfs subvol create /mnt/var/log
```

Disable Copy-on-Write (CoW) for databases
```
mkdir -p /mnt/var/lib/{mysql,postgres,machines}
chattr +C /mnt/var/lib/{mysql,postgres,machines}
```

### Installation

Set closest mirrors
```
reflector -c <country> > /etc/pacman.d/mirrorlist
```

Install packages
```
pacstrap /mnt linux linux-firmware base base-devel intel-ucode btrfs-progs dosfstools sudo screen tmux rsync openssh git nano iwd
```

Change the boot partition to use UUID
```
/mnt/etc/fstab
————
-- /dev/nvme0n1p1
++ UUID=559E-32F1
```

Chroot into installed system
```
arch-chroot /mnt
```

Continue the regular steps from the installation guide
```
nvim /etc/locale.gen # Uncomment en_GB.UTF-8
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
echo keymap=en > /etc/vconsole.conf
ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
hwclock --systohc

echo new-hostname > /etc/hostname

passwd # Set root password
```

### Initramfs and bootloader

Add modules to initramfs hooks
```
/etc/mkinitcpio.conf
————
-- HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)
++ HOOKS=(base systemd autodetect keyboard sd-vconsole modconf block sd-encrypt filesystems fsck)

-- BINARIES=()
++ BINARIES=(btrfs)
```

Generate initramfs
```
mkinitcpio -P
```

Install the systemd-boot loader:
```
bootctl --path=/boot install
```
Enable auto update of bootctl
```
systemctl enable systemd-boot-update.service
```

Get the UUID of the luks partition
```
UUID_OF_LUKS=$(blkid -s UUID | grep /dev/nvme0n1p2 | awk '{ print $2 }')
```

Create bootloader entry, very important to include `rd.luks.options=discard` for **TRIM** support. 
 
```
/boot/loader/entries/arch.conf
————
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options rd.luks.name=UUID_OF_LUKS=cryptroot root=/dev/mapper/cryptroot rootflags=subvol=@ rd.luks.options=discard rw
```

Set default bootloader
```
/boot/loader/loader.conf
————
default arch
timeout 0
```

Exit the chroot and reboot into system
```
exit
umount -R /mnt
reboot
```