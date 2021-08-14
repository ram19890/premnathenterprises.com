# Gentoo Linux Complete Installation on Intel CPU V0.42(Documented on: 210814)

> When installing gentoo make sure you keep the https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation URL page open on another device, as you might see an outdated installation method if you care to follow this tutorial by now! This installation is done with minimum effort for installing the base system, without tangling with exterme detailing.

### **Basics (Before you continue, check below)**
* Check the intergrity of the iso file with sha512sum checksum if necessory
* Download the gentoo minimum iso from the download page
* Kindly have the fastest INTEL CPU & Internet connection at home(For moderate compilation speeds, as gentoo is pure source based distribution)
* SSD or NVME please
* RAM above 8GB
* Some patience ;-)

### **Partition**

    Let's use 128GB SSD for the installation, with 3 partitions sda1(F32)(boot),sda2(swap) and sda3(ext4)(root). Partition them to your liking but the above stays manditory.

* We should be using the uefi based GPT for boot partition
* But disable SECURE BOOT on BIOS as it m   ay interfere with the installation



### **Check internet connection**
	ping google.com
* If this doesn't work, refer Handbook!

### **Partition the SSD**
Check existing status of the drive

	fdisk -l
	cfdisk /dev/sda

`USE GPT for UEFI`

My lap has 232.9G of storage and 8G of RAM

### **Creating 3 patitons sda1, sda2 and sda3**

`CURRENT LAYOUT WITH SIZES`

	/mnt/boot 512M - /dev/sda1 EFI
	swap 8G - /dev/sda2 SWAP
	/mnt {rest of storage} -  /dev/sda3 ROOT
* Using sda1 for boot (/dev/sda1 EFI)
	mkfs.vfat -F 32 /dev/sda1
* Using sda2 for Swap (/dev/sda2 SWAP)
	mkswap /dev/sda2
	swapon /dev/sda2
	free -m
* Using sda3 for the root partition /mnt {rest of storage}
	mkfs.ext4 /dev/sda3
### **After the partiton we need to mount all of them**

	mount /dev/sda3 /mnt/gentoo
	cd /mnt/gentoo
	wget "stage-3_tarball.iso" from gentoo download page
	tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
* Setup CFLAGS and CXXFLAGS
	nano -w /mnt/gentoo/etc/portage/make.conf
* This is only for the Haswell architecture only, kindly refer to Handbook for your specific flag setup!

	 # These settings were set by the catalyst build script that automatically
	 # built this stage.
	 # Please consult /usr/share/portage/config/make.conf.example for a more
	 # detailed example.
	 COMMON_FLAGS="-march=native -O2 -pipe"
	 CFLAGS="${COMMON_FLAGS}"
	 CXXFLAGS="${COMMON_FLAGS}"
	 FCFLAGS="${COMMON_FLAGS}"
	 FFLAGS="${COMMON_FLAGS}"
	 MAKEOPTS="-j4"
	 # NOTE: This stage was built with the bindist Use flag enabled
	 PORTDIR="/var/db/repos/gentoo"
	 DISTDIR="/var/cache/distfiles"
	 PKGDIR="/var/cache/binpkgs"
	 #USE="-gtk -gnome qt4 qt5 kde dvd alsa cdr systemd udev activities declarative dri kde kwallet > phonon plasma policykit qml semantic-desktop widgets"
	 # This sets the language of build output to English.
	 # Please keep this setting intact when reporting bugs.
	 LC_MESSAGES=C
	 GRUB_PLATFORMS="efi-64"
	 USE="euse bluetooth"
	 #ACCEPT_KEYWORDS="~amd64"
### **Setup fastest mirror**
	mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
	nano /mnt/gentoo/etc/portage/repos.conf/gentoo.conf


	 [DEFAULT]
	 main-repo = gentoo

	 [gentoo]
	location = /var/db/repos/gentoo
	sync-type = rsync
	sync-uri = rsync://rsync.gentoo.org/gentoo-portage
	auto-sync = yes
	sync-rsync-verify-jobs = 1
	sync-rsync-verify-metamanifest = yes
	sync-rsync-verify-max-age = 24
	sync-openpgp-key-path = /usr/share/openpgp-keys/gentoo-release.asc
	sync-openpgp-key-refresh-retry-count = 40
	sync-openpgp-key-refresh-retry-overall-timeout = 1200
	sync-openpgp-key-refresh-retry-delay-exp-base = 2
	sync-openpgp-key-refresh-retry-delay-max = 60
	sync-openpgp-key-refresh-retry-delay-mult = 4
### **MOUNTING**
	mount --types proc /proc /mnt/gentoo/proc
	mount --rbind /sys /mnt/gentoo/sys
	mount --make-rslave /mnt/gentoo/sys
	mount --rbind /dev /mnt/gentoo/dev
	mount --make-rslave /mnt/gentoo/dev

### **Entering the new environment (Chrooting)**
	chroot /mnt/gentoo /bin/bash
	source /etc/profile
	export PS1="(chroot) ${PS1}"

### **Mounting the boot partition**
	mount /dev/sda1 /boot

### **Configuring Portage**
	emerge-webrsync

### **Choosing the right profile**
	eselect profile list
	eselect profile set 9
* We would like to install kde with systemd support
### **Updating the @world set**
	emerge --ask --verbose --update --deep --newuse @world
* This will take time....

* Edit make.conf for further changes and run emerge!
	nano -w /etc/portage/make.conf
	emerge -uavDN @world

	emerge -uUDav --exclude python:3.9 @world
* If facing any issues with python, exclude the python version

* List all USE flags for further addition and subtraction of flags!
	less /var/db/repos/gentoo/profiles/use.desc

* NOTICE: Do not accept EULA!
### **TimeZone**
	ls /usr/share/zoneinfo
	ln -sf ../usr/share/zoneinfo/Asia/Kolkata /etc/localtime

`etc-update`
`emerge --sync`
`etc-update --automode -3`

### **Locale generation**
	nano -w /etc/locale.gen
* Uncomment en_US.UTF-8 UTF-8
	locale-gen

`. /etc/profile`

### **Locale selection**
	eselect locale list
	eselect locale set 5
* Set to "en_US.utf8"

* Reload the environment:
	env-update && source /etc/profile && export PS1="(chroot) ${PS1}"

### **INSTALLATION**

* Kernel Source:
	emerge --ask sys-kernel/gentoo-sources
* Building and Installing an initramfs:
    emerge --ask sys-kernel/genkernel
* Some drivers require additional firmware:
	emerge --ask sys-kernel/linux-firmware
`echo "sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE" | tee -a /etc/portage/package.license`
	nano -w /etc/fstab
`genkernel all`
### **Partition labels & UUIDs**
	blkid
	nano -w /etc/fstab
	emerge --ask vim
	emerge --ask wpa_supplicant
	wpa_cli
	systemctl status sshd
	systemctl enable sshd
	systemctl start sshd
	fdisk -l
	vim /etc/fstab

## **Host and domain information**
	nano -w /etc/conf.d/hostname
	hostname="tux"
	nano -w /etc/conf.d/net
	dns_domain_lo="homenetwork"

* All networking information is gathered in /etc/conf.d/net
	emerge --ask --noreplace net-misc/netifrc
	nano -w /etc/conf.d/net

* For Manual ip assign
	config_eth0="192.168.0.2 netmask 255.255.255.0 brd 192.168.0.255"
	routes_eth0="default via 192.168.0.1"

* For DHCP
	config_eth0="dhcp"

### **Host File(example):**
	nano -w /etc/hosts


	# This defines the current system and must be set
	127.0.0.1     tux.homenetwork tux localhost
	# Optional definition of extra systems on the network
	192.168.0.5   jenny.homenetwork jenny
	192.168.0.6   benny.homenetwork benny

### **Change current password for root**
	passwd
### **Install the system logger, DHCP, wireless networking tools & File indexing**
	emerge --ask app-admin/sysklogd
	emerge --ask sys-apps/mlocate
	emerge --ask net-misc/dhcpcd
	emerge --ask net-wireless/iw net-wireless/wpa_supplicant

### **GRUB2 config**
	echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
	emerge --ask --verbose sys-boot/grub:2
	grub-install --target=x86_64-efi --efi-directory=/boot

### **Generate the final GRUB2 configuration**
	grub-mkconfig -o /boot/grub/grub.cfg

## **POST INSTALLATION**

	ip addr
	wpa_cli -i wlp0s20u9
	dhcpcd
	ping ddg.gg
	vim /etc/resolv.conf
	vim /etc/systemd/network/50-dhcp.network
	vim /etc/resolv.conf
	systemctl enable systemd-resolved.service
	systemctl start systemd-resolved.service
	nmtui
	vim /etc/resolv.conf
	systemctl start dhcpcd
	systemctl enable dhcpcd
	vim /etc/dispatch-conf.conf
	emerge --ask app-misc/colordiff
	vim  /etc/dispatch-conf.conf
	eselect editor list
	emerge --ask dispatch-conf
	emerge --ask app-portage/dispatch
	emerge --ask rcs
	vim /etc/dispatch-conf.conf
	emerge --verbose --info | egrep '^USE'
	portageq envvar EDITOR
	emerge --ask git
	emerge --ask dev-vcs/git
	emerge --ask kde-plasma/plasma-meta
	startx
	systemctl enable sddm
	systemctl start sddm
	emerge --ask sudo
	sudo
	visudo
	useradd -m -G users,wheel,audio -s /bin/bash ram
	passwd ram

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

## **To enable WOL, make sure you enable it on BIOS**
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

## **SAMBA(Still buggy)**
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

* To check for errors, use this command to determine the cause of the issue!
	testparm
* Start the below services to start Samba

	sudo systemctl enable smbd
	sudo systemctl enable nmb
	sudo systemctl start smbd 
	sudo systemctl start nmb
## **To use AUR and Enable Teamviewer on Systemd**

	git clone https://aur.archlinux.org/anydesk-bin.git
	cd anydesk-bin

	sudo rm /var/lib/pacman/db.lck

	makepkg -Acs
	pacman -U anydesk-bin.pkg.tar.xz

## **SystemD**
	sudo systemctl enable teamviewerd.service
	sudo systemctl start teamviewerd.service

Then start teamviewer
