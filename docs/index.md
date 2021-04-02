# Arch Linux Complete Installation on Laptop V0.5(210402)

## To check whether the existing boot is UEFI or BIOS

	ls /sys/firmare/efi/efivars

If it shows without an error it's UEFI
If directory does not exist, then it's BIOS	

## Check for wireless devices on laptop 
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

## Partition the HardDisk/SSD
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
## INSTALLATION
Installs the base packages for the installation

	pacstrap /mnt base base-devel linux linux-firmware
FSTAB is responsible for booting partition from the etc directory
	
	genfstab -U /mnt >> /mnt/etc/fstab
We now need to boot into Arch Vertual Env
	
	arch-chroot /mnt
WPA-Supplicant is required to store Wirless Hash Passwords

	pacman -S dialog wpa_supplicant
Set TimeZone	

	ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
	hwclock --systohc
Set Timezone(Alternative)

	timedatectl set-timezone "Asia/Kolkata"
	timedatectl status
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
	::1			localhost
	127.0.1.1	arch.localdomain arch
Initialize RamDisk
	
	mkinitcpio -P linux
Change current password for root
	
	passwd
To setup EFI Boot `Make sure to DISABLE Secure Boot while Enabling EFI on LAPTOP BIOS`
	
	pacman -S grub efibootmgr vi vim ntfs-3g xorg sddm xorg-xinit xterm

	grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot --recheck
	grub-mkconfig -o /boot/grub/grub.cfg

To interact with the OS using Mouse and KeyBoard, we need a Window Manager(LXQT)
	
	pacman -S i3-wm dmenu lxqt

For KDE use the line below

	pacman -S plasma plasma-wayland-session		

# REBOOT to complete the installation, make sure the grub shows up!

## POST INSTALLATION

	sudo systemctl enable NetworkManager
	sudo systemctl start NetworkManager

Add a new user
	useradd -m ram
Add password for new user
	passwd ram	
Add new user to the wheel and network groups
	usermod --append --groups wheel ram
	usermod --append --groups network ram
Uncomment the wheel, for root level access for the new user
	visudo
	%wheel ALL=(ALL)ALL

## To enable WOL, make sure you enable it on BIOS
Kindly check, if your ethernet device supports WOL 
	ethtool interface | grep Wake-on
Make sure to find the proper ethernet MAC address to make this work!
	vim /usr/lib/systemd/network/99-default.link
	
	[Match]
	MACAddress=aa:bb:cc:dd:ee:ff

	[Link]
	NamePolicy=kernel database onboard slot path
	MACAddressPolicy=persistent
	WakeOnLan=magic

## SAMBA(Still buggy)
To create a samba server all you need is to create a samba user and point to the directory to share!
Instead of /opt use your own user directory
	
	sudo pacman -S samba
Create a new samba user	& password for the same.
	smbpasswd -a ram
	smbpasswd ram
To list all samba users	
	sudo pdbedit -L -v

	sudo vim /etc/samba/smb.conf

	[global]
	logging = systemd
	#workgroup = WORKGROUP
	client min protocol = NT1
	#server min protocol = SMB2_02
	#server max protocol = SMB3
	map to guest = bad password
	server min protocol = NT1
	[share]
	path = /opt
	valid users = ram
	guest ok = yes
	browseable = yes
	public = yes
	writable = yes

To check for errors, use this command to determine the cause of the issue!
	 testparm
Start the below services to start Samba
	sudo systemctl enable smbd 
	sudo systemctl enable nmb
	sudo systemctl start smbd 
	sudo systemctl start nmb
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

## Upcoming

### `Planned to create an automation script for fast Arch Install (Time duration often depends on net and storage speed!)`
