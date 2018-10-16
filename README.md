# Manual de instalação do Arch Linux

Estaremos abordando a instalação do Arch, o uso e opções.
[TOC]

## Particionamento

### BtrFS
1. Criando a partição
```zsh
mkfs.btrfs -L "Arch Linux" /dev/sda1
```
2. Montando a partição

```zsh
mkdir /mnt/btrfs-root
mount -o defaults,relatime,discard,ssd,nodev,nosuid /dev/sda1 /mnt/btrfs-root
```
3. Criando os SubVolumes

```zsh
mkdir -p /mnt/btrfs/__snapshot
mkdir -p /mnt/btrfs/__current
btrfs subvolume create /mnt/btrfs-root/__current/root
btrfs subvolume create /mnt/btrfs-root/__current/home
```
4. Montando os SubVolumes

```zsh
mkdir -p /mnt/btrfs-current

mount -o defaults,relatime,discard,ssd,nodev,subvol=__current/root /dev/sda1 /mnt/btrfs-current
mkdir -p /mnt/btrfs-current/home

mount -o defaults,relatime,discard,ssd,nodev,nosuid,subvol=__current/home /dev/sda1 /mnt/btrfs-current/home
```

### Ext4 FS

## Instação do Arch
1. Selecione o espelho a ser usado

```zsh
vim /etc/pacman.d/mirrorlist 
```
2. A instalação

```zsh
pacstrap /mnt/btrfs-current base base-devel
```
3. Gerando a ordem de boot
** copy the partition info for / and mount it on /run/btrfs-root (remember to remove subvol parameter! and add nodev,nosuid,noexec parameters) **

```zsh
genfstab -U -p /mnt/btrfs-current >> /mnt/btrfs-current/etc/fstab
vim /mnt/btrfs-current/etc/fstab
```

## Configurações Finais
```zsh
arch-chroot /mnt/btrfs-current /bin/bash

pacman -S btrfs-progs
```
** Descomente ***
```zsh
vim /etc/locale.gen
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
ln -s /usr/share/zoneinfo/Europe/Kiev /etc/localtime
hwclock --systohc --utc
```
** Setando Hostname**
```zsh
echo 'idv-HP-EliteBook-840-G1' > /etc/hostname
nano /etc/nsswitch
```

pacman -S wicd
systemctl enable wicd.service

nano /etc/mkinitcpio.conf
 * Remove fsck and add btrfs to HOOKS
mkinitcpio -p linux

passwd
groupadd idv
useradd -m -g idv -G users,wheel,storage,power,network -s /bin/bash -c "Ihor Dvoretskyi" idv
passwd idv

== Install boot loader ==

pacman -S grub-bios
grub-install --target=i386-pc --recheck /dev/sda
nano /etc/default/grub
 * Edit settings (e.g., disable gfx, quiet, etc.)
grub-mkconfig -o /boot/grub/grub.cfg

== Unmount and reboot ==

exit

umount /mnt/btrfs-current/home
umount /mnt/btrfs-current
umount /mnt/btrfs-root

reboot


loadkeys br-abnt
ping -c3 google.com
cfdisk
mkfs.ext4 /dev/sda1
fdisk -l