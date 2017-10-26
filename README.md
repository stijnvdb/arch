# arch
Arch Linux

# Basic installation
Copy/paste the following commands to perform the basic installation

## Disk partitioning

1. Unencrypted

```
cfdisk /dev/sda #create partitions / , /boot, /home, /var and optional swap
mkfs.ext4 /dev/sdaW # /
mkfs.ext2 /dev/sdaX # /boot
mkfs.ext4 /dev/sdaY # /home
mkfs.ext4 /dev/sdaZ # /var
# mkswap /dev/sda?
mount /dev/sdaW /mnt
mkdir /mnt/boot /mnt/home /mnt/var
mount /dev/sdaX /mnt/boot
mount /dev/sdaY /mnt/home
mount /dev/sdaZ /mnt/var
#swapon /dev/sda?
```

2.  Encrypted

```
cfdisk /dev/sda # create 2 partitions, 1 200 MB for /boot, the rest for the encrypted volume
cryptsetup -y -v luksFormat /dev/sda1
cryptsetup open /dev/sda1 cryptolvm
pvcreate /dev/mapper/cryptolvm
vgcreate arch /dev/mapper/cryptolvm
lvcreate -L 20G arch -n root
lvcreate -L 10G arch -n var
lvcreate -L ??G arch -n home
# lvcreate -L 4G arch -n swap
mkfs.ext4 /dev/mapper/arch-root
mkfs.ext4 /dev/mapper/arch-home
mkfs.ext4 /dev/mapper/arch-var
# mkswap /dev/mapper/arch-swap
mkfs.ext2 /dev/sda2
mount /dev/mapper/arch-root /mnt
mkdir /mnt/boot /mnt/home /mnt/var
mount /dev/mapper/arch-home /mnt/home
mount /dev/mapper/arch-var /mnt/var
mount /dev/sda2 /mnt/boot
# swapon /dev/mapper/arch-swap
```

## Installation

```
pacstrap -i /mnt base base-devel
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt /bin/bash
pacman -Sy git vim
vim /etc/locale.gen
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
rm /etc/localtime
ln -sf /usr/share/zoneinfo/Europe/Brussels /etc/localtime
hwclock --systohc --utc
echo arch.stijn > /etc/hostname
vim /etc/hosts
pacman -S networkmanager
systemctl enable NetworkManager
sed -i -e "s/ˆHOOKS=\".*\"/HOOKS=\"base udev autodetect modconf keyboard keymap block encrypt filesystems fsck lvm2\"/g" /etc/mkinitcpio.conf
mkinitcpio -p linux
pacman -S device-mapper grub linux lvm2
grub-install --recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
export sda1_uuid=`blkid | head -n1 | cut -f2 -d"\"" # Get the UUID of /dev/sda1
sed -i "s/\/vmlinuz-linux root=/\/vmlinuz-linux cryptdevice=UUID=${sda1_uuid}:cryptolvm root=/g" /boot/grub/grub.cfg
exit
reboot
```
