# Arch Linux Installation

Check for wireless devices on laptop 
ip addr

Turn on wifi module
ip link set wlp2s0 up

Cli interface to connect to wifi
wifi-menu

netctl list
netctl start profile
ping 1.1.1.1

cfdisk /dev/sda

PARTITIONING IS IMPORTANT

USE GPT for UEFI
USE BIOS for MBR

My lap has 232.9G of storage and 4G of RAM

/mnt/boot 512M
swap 8G
/mnt {rest of storage}


/dev/sda1 EFI 
/dev/sda2 SWAP
/dev/sda3 ROOT

mkfs.fat /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
free -m
mkfs.ext4 /dev/sda3


mount /dev/sda3 /mnt
cd /mnt
mkdir boot
mount /dev/sda1 /mnt/boot
cd /mnt

INSTALLATION

pacstrap /mnt base base-devel
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
pacman -S dialog wpa_supplicant
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime

hwclock --systohc

nano /etc/locale.gen

Uncomment en_US.UTF-8 UTF-8
locale-gen

nano /etc/hostname
arch

nano /etc/hosts
127.0.0.1	localhost
::1		localhost
127.0.1.1	arch.localdomain arch

mkinitcpio -p linux

passwd

pacman -S grub efibootmgr

grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot --recheck
grub-mkconfig -o /boot/grub/grub.cfg


xorg i3-wm dmenu sddm xorg-xinit xterm

lxqt




## To use AUR and Enable Teamviewer on Systemd

	git clone https://aur.archlinux.org/anydesk-bin.git
	cd anydesk-bin

	sudo rm /var/lib/pacman/db.lck

	makepkg -Acs
	pacman -U anydesk-bin.pkg.tar.xz

## SystemD
	sudo systemctl enable teamviewerd.service
	sudo systemctl start teamviewerd.service

Then start teamviewer


For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.
