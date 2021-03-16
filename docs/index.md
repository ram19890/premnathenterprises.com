# Arch Linux Complete Installation on Laptop V0.1


##Check for wireless devices on laptop 
This shows the list of active network connections
	
	ip addr

Turn on wifi module
	
	ip link set wlp2s0 up

Use CLI interface to connect to wifi

	wifi-menu
	
	netctl list
	
	netctl start profile
	
	ping 1.1.1.1

`PARTITIONING IS VERY IMPORTANT, kindly BACKUP you drives before further processing`

##Partition the HardDisk/SSD
Check existing status of the drive
	
	fdisk -l

	cfdisk /dev/sda

`USE GPT for UEFI` or `USE BIOS for MBR`

My lap has 232.9G of storage and 4G of RAM

###Creating 3 patitons sda1, sda2 and sda3

`CURRENT LAYOUT WITH SIZES`

* /mnt/boot 512M - /dev/sda1 EFI 
* swap 8G - /dev/sda2 SWAP
* /mnt {rest of storage} -  /dev/sda3 ROOT

Using sda1 for boot (/dev/sda1 EFI)
	
	mkfs.fat /dev/sda1
Using sda2 for Swap (/dev/sda2 SWAP)
	
	mkswap /dev/sda2
	swapon /dev/sda2
	free -m
Uing sda3 for the root partition /mnt {rest of storage}
	
	mkfs.ext4 /dev/sda3
###After the partiton we need to mount all of them

	mount /dev/sda3 /mnt
	cd /mnt
	mkdir boot
	mount /dev/sda1 /mnt/boot
	cd /mnt
##INSTALLATION
Installs the base packages for the installation

	pacstrap /mnt base base-devel
FSTAB is responsible for booting partition from the etc directory
	
	genfstab -U /mnt >> /mnt/etc/fstab
We now need to boot into Arch Vertual Env
	
	arch-chroot /mnt
WPA-Supplicant is required to store Wirless Hash Passwords

	pacman -S dialog wpa_supplicant
Set TimeZone	
	
	ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
	hwclock --systohc
Set Keyboard
	
	nano /etc/locale.gen

	`Uncomment`en_US.UTF-8 UTF-8
	locale-gen
Change hostname

	nano /etc/hostname
	arch
Edit hosts file(To identify your network and set loopback)

	nano /etc/hosts
	
	127.0.0.1	localhost
	::1		localhost
	127.0.1.1	arch.localdomain arch
Initialize RamDisk
	
	mkinitcpio -p linux
Change current password for root
	
	passwd
To setup EFI Boot `Make sure to DISABLE Secure Boot while Enabling EFI on LAPTOP BIOS`
	
	pacman -S grub efibootmgr

	grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot --recheck
	grub-mkconfig -o /boot/grub/grub.cfg

To interact with the OS using Mouse and KeyBoard, we need a Window Manager(LXQT)
	
	pacman -S xorg i3-wm dmenu sddm xorg-xinit xterm lxqt




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

##Upcoming

###`Plan to create an automation script for fast Arch Install (Time duration often depends on net and storage speed!)`
