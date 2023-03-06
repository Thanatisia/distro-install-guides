# Fedora via Bootstrap Base Installation Guide

## Information
### System Information
+ Architecture: AMD64

## Setup
### Dependencies
+ dnf

### Pre-Requisites

#### Preparation Steps
> The following are general steps as referenced in my [Pre-Installation General Flow Guide](https://github.com/Thanatisia/SharedSpace/blob/main/Docs/Linux/Guides/Setup/General%20Flow.md)
0. [Host System Preparation](#host-system-preparation)
1. [Set Keyboard Layout](#set-keyboard-layout)
2. [Verify Boot Mode](#verify-boot-mode)
3. [Update System Clock](#update-system-clock)
4. [Prepare Disks](#prepare-disks)
5. [Mount Disks](#mount-disks)

#### Host System Preparation
- Network
	- Verify network is active
		```console
		# Get network
		ip a s
		ip link
		```
	- If there is no internet 
		- temporary fix 1: using dhcpcd
			```console
			sudo dhcpcd
			```
	- Test connection
		```console
		ping 8.8.8.8
		```

#### Set Keyboard Layout
- Get Keyboard Layouts
	```console
	ls /usr/share/kbd/keymaps/**/*.map.gz
	```

- (OPTIONAL) If you are changing keyboard layouts
	```console
	loadkeys [keyboard layout code]
	```

#### Verify Boot Mode (Motherboard Firmware)
- Check if firmware is UEFI
	- List efivars directory
		- If the directory displays
			+ without error: Booted in UEFI mode
			+ with error: Booted in BIOS or CSM mode
		```console
		ls /sys/firmware/efi/efivars
		```

#### Update System Clock
- Sync Network Time Protocol (NTP)
	```console
	timedatectl set-ntp true
	```

- Check system clock
	```console
	timedatectl status
	```

#### Prepare Disks
> This step, as I call it, is the Disk/Partition Management and Configuration step
- NOTES
	+ Please refer to [Partition Configuration Schema/Design](#partition-configuration-schema-design) for the Partition Management schema and reference it when creating the Disks
	+ Partitioning and formatting of Disk labels are permanent, please be extra careful here

- Verify disk/device/file block
	+ Note the device names (i.e. /dev/sdX)
	```console
	# List all disks
	lsblk

	# Using fdisk
	sudo fdisk -l
	```

- Prepare Disks
	- NOTES
		- Utilities and Tools
			> there are Multiple Tools for Partitioning
			- Using parted
				- Synopsis/Syntax
					- Create Device Label (aka Reformat)
						+ parted [device_name] mklabel [partition_table]
					- Creating new partition
						- If Partition Table is MSDOS (MBR)
							+ parted [device_name] mkpart [partition_type] [filesystem_Type] [start_size] [end_size]
						- If Partition Table is GPT (UEFI)
							+ parted [device_name] mkpart [partition_label] [filesystem_Type] [start_size] [end_size]
					- Formatting file system/partition
						+ mkfs.{filesystem-type} {options} [device_name](partition_number)
					- Formatting Swap
						+ mkswap [device_name](partition_number)
					- Set settings to partition
						+ parted [device_name] set [partition_number] {settings}
						- settings:
							+ boot {on|off} : Set Boot to Boot Partition of MSDOS (MBR) Partition Table
							+ esp {on|off} : Set Boot to Boot Partition of GPT Partition Table
			- Using fdisk
			- Using cgdisk
		- Information
			- Disk Label
				+ Create a [gpt] Disklabel type if the partition is GPT (UEFI) or
				+ Create a [msdos] Disklabel type if the partition is MBR (BIOS)
			- Partitions:
				- In every instance in a GPT/(U)EFI Partition Table
					- A special bootable EFI system partition is required
						+ Similar to MSDOS (MBR)'s Boot Partition
					+ Requires FAT32 while MSDOS's Boot Partition Requires EXT4
				+ EFI system partition on a MBR partition table is identified by the Partition Type ID [EF]
	- Create/Format Device Label
		- If Partition Table label is MSDOS
			+ Can be either BIOS or UEFI
			```console
			parted /dev/sdX mklabel msdos
			```
		- If Partition Table label is GPT
			+ Typically requires boot firmware to be UEFI
			```console
			parted /dev/sdX mklabel gpt
			```
	- Create Partitions
		- If Partition Table is MSDOS/MBR (running on BIOS Firmware)
			- Required Information
				+ The Master Boot Record (MBR) only supports up to a maximum of 2TB storage due to legacy reasons
			- Synopsis/Syntax
				+ Device Name : The Disk/Device/Image file path/name of your target device; i.e. /dev/sdX
				- Partition Type : The type of partition you want to create; i.e. primary, extended or logical
					- Primary : The primary parititon blocks
					- Extended 
						+ Logical
				- Partition Filesystem : The filesystem you want the partition to use
				+ Partition Start Size : The position of the memory that you want the partition to start from
					- Can be represented in percentages
						+ 0%  : Start from the beginning of the disk block
						+ 25%  : 1/4 of the device/disk/file's total size
						+ 50%  : 1/2 of the device/disk/file's total size
						+ 70%  : 3/4 of the device/disk/file's total size
				+ Partition End Size : The ending position of the memory that you want the partition to fill to
						+ 25%  : 1/4 of the device/disk/file's total size
						+ 50%  : 1/2 of the device/disk/file's total size
						+ 70%  : 3/4 of the device/disk/file's total size
						+ 100% : Allocate to the end of the device/disk/file's total size
			- Structure
				+ Syntax: parted [device_name] mkpart [partition_type] [partition_fileType] [start_size] [end_size]
			- Examples
				- Boot Partition
					```console
					parted /dev/sdX mkpart primary ext4 0% 1024MiB
					```
				- Swap Partition
					```console
					parted /dev/sdX mkpart primary swap 1024MiB x1MiB
					```
				- Root Partition
					```console
					parted /dev/sdX mkpart primary ext4 x1MiB x2MiB
					```
				- Home Partition
					```console
					parted /dev/sdX mkpart primary ext4 x2MiB 100%
					```
		- If Partition Table is GPT (running on UEFI Firmware)
			- Required Information
				+ All disk/devices/image files above 4TB needs to run on GPT because MBR only supports up to a maximum of 2TB due to legacy reasons
				+ The GPT Partition Table boot partition needs to be in FAT32 because of UEFI Bootloader Firmware requirements
			- Synopsis/Syntax
				+ Device Name : The Disk/Device/Image file path/name of your target device; i.e. /dev/sdX
				- Partition Label : A label (or a "Name") assigned to the partition
					+ Labels allows you to find the partition at '/dev/disk/by-partlabel/[your-partition-label' instead of the device name (i.e. /dev/sdX)
				- Partition Filesystem : The filesystem you want the partition to use
				+ Partition Start Size : The position of the memory that you want the partition to start from
					- Can be represented in percentages
						+ 0%  : Start from the beginning of the disk block
						+ 25%  : 1/4 of the device/disk/file's total size
						+ 50%  : 1/2 of the device/disk/file's total size
						+ 70%  : 3/4 of the device/disk/file's total size
				+ Partition End Size : The ending position of the memory that you want the partition to fill to
						+ 25%  : 1/4 of the device/disk/file's total size
						+ 50%  : 1/2 of the device/disk/file's total size
						+ 70%  : 3/4 of the device/disk/file's total size
						+ 100% : Allocate to the end of the device/disk/file's total size
			- Structure
				+ Syntax: parted [device_name] mkpart [partition_label] [partition_fileType] [start_size] [end_size]
			- Examples
				- Boot Partition
					```console
					parted /dev/sdX mkpart "Boot" fat32 0% 1024MiB
					```
				- Swap Partition
					```console
					parted /dev/sdX mkpart "Swap" swap 1024MiB x1MiB
					```
				- Root Partition
					```console
					parted /dev/sdX mkpart "Root" ext4 x1MiB x2MiB
					```
				- Home Partition
					```console
					parted /dev/sdX mkpart "Home" ext4 x2MiB 100%
					```
	- Format Partitions
		- Synopsis/Syntax
			- File Types
				- Fat{16|32}
					+ mkfs.fat -f {16|32} /dev/sdX(n)
				- EXT4
					+ mkfs.ext4 /dev/sdX(n) 
				- Swap
					+ mkswap /dev/sdX(n)
		- Examples
			- FAT32
				```console
				mkfs.fat -F32 /dev/sdX[partition-number]
				```
			- EXT4
				```console
				mkfs.ext4 /dev/sdX[partition-number]
				```
			- Swap
				```console
				mkswap /dev/sdX[partition-number]
				```
	- Set Bootable
		- If Partition Table is MSDOS/MBR
			- Syntax/Synopsis
				```console
				parted [device_name] set [partition-number] boot on
				```
			- Examples
				```console
				parted /dev/sdX set 1 boot on
				```
		- If Partition Table is GPT
			- Syntax/Synopsis
				```console
				parted [device_name] set [partition-number] esp on
				```
			- Examples
				```console
				parted /dev/sdX set 1 esp on
				```
	- (OPTIONAL) Setup Swap Partition
		- Enable swap partition
			- Synopsis/Syntax
				```console
				swapon [device-name][swap-partition-number]
				```
			- Examples
				```console
				swapon /dev/sdX(n)
				```
		- After Chroot
			- Append Swap partition to filesystems table (/etc/fstab) in mount
				```console
				echo "# -- Swap Partition 1" | tee -a /etc/fstab
				echo "UUID=device_UUID none swap defaults 0 0" | tee -a /etc/fstab
				```

#### Mount Disks
- Synopsis/Syntax
	```console
	mount [device-name][partition_number] [mount-path]
	```

- Mount root volume to a mount point
	```console
	mount /dev/sdX2 /mnt
	```

- (OPTIONAL) Make your other directories to mount inside the root volume
	- Notes
		+ This is depending on your partition layout design, refer to it for more information to proceed
	- Boot directory
		- If Partition Table is MBR (MSDOS)
			```console
			mkdir -p /mnt/boot
			```
		- If Partition Table is GPT
			- Typical Boot Partition Mount Points
				+ /boot : The usual boot partition mount point and is the preferred method when directly booting on EFISTUB kernel from UEFI or booting it via a boot manager like systemd-boot
					```console
					mkdir -p /mnt/boot/
					```
				- /efi : Is a replacement for the previously popular (and is possibly still used by other Linux distros) ESP mountpoint [/boot/efi].
					+ Use a bootloader which is capable of accessing the kernel(s) and initramfs image(s) that are stored elsewhere - typically /boot
					```console
					mkdir -p /mnt/efi
					```
	- Home directory
		```console
		mkdir -p /mnt/home
		```

- Mount remaining directories after mounting root
	- Important Information
		- Ensure that you mount your root partition first before any other partition or directories
			+ Otherwise those partitions will be overwritten and mounted by the root partition
	- Examples
		- Home and Boot directories
			```console
			mount /dev/sdX1 /mnt/boot
			mount /dev/sdX3 /mnt/home
			```

## Documentation
### Root Filesystem Installation
1. Install dnf
	- Using Package Manager
		+ Using apt
			```console
			sudo apt install dnf
			```
		+ Using pacman
			```console
			sudo pacman install dnf
			```
	- Get group
		```console
		sudo dnf group list
		```
2. (OPTIONAL) Preparation and Pre-Requisites 
	- If you are using a Fedora/RPM-based distribution (or the LiveISO)
		- Disable SELinux
			+ SELinux can be very limiting
			+ In my experience, SELinux enabled prevented the chroot from changing the root password
			```console
			sudo enforce 0
			```
	- If you are using a non-fedora/RPM distribution
		- Notes
			- Please refer to [My dnf setup guide](https://github.com/Thanatisia/SharedSpace/tree/main/Docs/Linux/distros/Fedora) for steps on setting up dnf and
				+ setting up Fedora core repositories
		- Enable dnf/yum repositories
			- Enable the new repository
				+ using `dnf config-manager`
				```console
				sudo dnf config-manager --add-repo /etc/yum.repos.d/[repository-name].repo
				```
3. Bootstrap the necessary files required directly from the Debian archives mirrors
	- Run dnf and Install the root filesystem
		- Notes
			+ Please refer to [System Architectures](#system-architectures) for a full list of system architecture and its keywords
			+ Please refer to [Release Version Code-name](#release-version-code-name) for a full list (as of writing) of versions and its code names, you may find more in the Fedora documentations.
			+ Please refer to [Package Groups](#package-groups) for a full list (as of writing) of Groups
		- Synopsis/Syntax
			```console
			sudo dnf {other-options} --releasever=[release-version-number] --installroot=[root-system-mountpoint] group install [group]
			```
		- Parameters
			- Positionals
				+ group                  : Manage groups; Fedora dnf uses 'groups' for packages where each group has packages
				+ install                : Install the packages in the specified group
			- Optionals
				+ --arch [your-system-architecture]      : Specify the target system architecture; amd64, arm64, i386
				+ --installroot=[root-system-mountpoint] : Specify the root filesystem directory to install; i.e. /mnt
				+ --releasever=[release-version-number]  : Specify the target release version number
		- Usage
			- Bootstrap to mounted root filesystem directory
				```console
				sudo dnf {other-options} --releasever=[release-version-number] --installroot=[root-system-mountpoint] group install [group]
				```
		- Examples
			- Fedora 37
				```console
				sudo dnf --releasever=37 --installroot=/mnt/fedora group install core
				```
4. Prepare chroot environment
	- (Optional) Copying the host's DNS (Domain Name Service) resolver file to the mount point
		+ In order to use an internet connection in the chroot environment
		- Remove existing DNS Resolver config file
			```console
			sudo rm [mount-point]/etc/resolv.conf
			```
		- Add host DNS Resolver config file
			```console
			cp /etc/resolv.conf [mount-point]/etc/resolv.conf
			```
	- Update mounted root filesystem table
		+ Write the current filesystems table into the 'etc/fstab' file
		- Manually
			- Retrieve Block UUID of your devices
				- Using 'blkid'
					- Information
						+ Leave the device-pathname empty to retrieve all UUIDs
					- Synopsis/Syntax
						```console
						blkid [device-pathname]{partition-number}
						```
					- Usage
						- Retrieve all UUIDs
							```console
							blkid
							```
						- Get specific UUID
							```console
							blkid /dev/sdX(n)
							```
		- (Recommended) Using genfstab
			- Pre-Requisite
				- Install 'arch-install-scripts'
					- Using apt 
						```console
						sudo apt install arch-install-scripts
						```
					- Using pacman
						```console
						sudo pacman -S arch-install-scripts
						```
					- Using dnf
						```console
						sudo dnf install arch-install-scripts
						```
			- Generate filesystems table of mount point and write into target filesystem's '/etc/fstab' file
				```console
				genfstab -U [root-mount-point] | sudo tee -a [mount-point]/etc/fstab
				```
	- (Optional) Setup first boot
		- Using systemd-firstboot
			- Information
				+ This will write the files '/etc/localtime' and '/etc/hostname'
				+ --setup-machine-id : Initializes the system's machine ID to a random ID; only works in combination with --root
			```console
			systemd-firstboot --root=[root-mount-point] --timezone=[Region]/[City] --hostname=[your-hostname] --setup-machine-id
			```
	- Mounting
		- Automatically via arch-chroot
			- the ArchLinux install scripts package has various very useful QoL utilities that wraps around steps
			- One of the utility is 'arch-chroot'
				+ Basically, the steps and mountings specificed in 'manually via chroot' are done in various similar chroot wrapper scripts such as in 'arch-chroot'
			- Pre-Requisite
				- Install 'arch-install-scripts'
					- Using apt 
						```console
						sudo apt install arch-install-scripts
						```
					- Using pacman
						```console
						sudo pacman -S arch-install-scripts
						```
					- Using dnf
						```console
						sudo dnf install arch-install-scripts
						```
			- Synopsis/Syntax
				```console
				sudo arch-chroot [mount-point]
				```
			- Usage
				```console
				sudo arch-chroot /mnt
				```
		- Manually via chroot
			- Temporary API Virtual Filesystems
				- Information
					+ proc  : The processes virtual filesystem
					+ sysfs : system virtual filesystem
					+ dev   : Devices virtual filesystem
					+ run   : Temporary runner virtual filesystem
				```console
				# Mount temporary API filesystems
				mount -t proc /proc [mount-point]/proc
				mount -t sysfs /sys [mount-point]/sys
				mount --rbind /dev [mount-point]/dev
				mount --rbind /run [mount-point]/run # Optional
				```
			- (OPTIONAL) If you are using an UEFI system
				+ Requires access to the EFI variable in the UEFI system firmware folder
				```console
				# If you are using UEFI system
				# require access to EFI variables
				mount --rbind /sys/firmware/efi/efivars [mount-point]/sys/firmware/efi/efivars
				```
			- Begin chroot into the system
				- Synopsis/Syntax
					```console
					# Chroot into mount directory
					sudo chroot [mount_path] [shell]
					```
				- Common Errors
					- Error [chroot: cannot run command '/usr/bin/bash': Exec format error] : Architectures of the host environment and chroot environment do not match
					- Error [chroot: '/usr/bin/bash': permission denied]					: Remount it with the execute permission - [mount -o remount,exec <new_root>]
				- Usage
					```console
					sudo chroot /mnt /bin/bash
					```
5. Configure Base System
	- Update Package Manager
		- Using dnf
			```console
			sudo dnf upgrade
			```
	- Setting Timezone
		- Search for your Timezone
			- To get your Region
				```console
				ls /usr/share/zoneinfo
				```
			- To get your City
				```console
				ls /usr/share/zoneinfo/[Region]
				```
		- Set your timezone
			- Using symlink
				- Symlink the timezone to /etc/localtime
					```console
					ln -sf /usr/share/zoneinfo/[Region]/[City] /etc/localtime
					```
				- Generate /etc/adjtime via hwclock
					```console
					hwclock --systohc
					```
			- Using systemd-firstboot
				- Information
					+ This will write the file '/etc/localtime'
				```console
				systemd-firstboot --timezone=[Region]/[City]
				```
	- Setting Locales and Keyboard
		- Install Essential Packages
			+ glibc-langpack-en : This is the EN language pack; contains various english locale languages such as 'en_US.UTF-8, en_SG.UTF-8' etc
			```console
			sudo dnf install glibc-langpack-en
			```
		- Locales
			- Notes
				- Unlike Debian, Gentoo and ArchLinux; Fedora does not have a locale.gen/locale-gen file
					+ Thus, the following steps using locale-gen will be removed as it is inapplicable
			- Add/Set LANG variable
				- Manually
					```console
					echo "LANG=[locale]" | tee -a /etc/locale.conf
					```
				- Using systemd-firstboot
					- Inofmration
						+ This will write the '/etc/locale.conf' file
						+ locale: i.e. 'en_US.UTF-8'
					```console
					systemd-firstboot --locale=[your-locale]
					```
			- Verify languages
				```console
				localectl list-locales
				```
		- Keyboard (Optional)
			- Editing '/etc/vconsole.conf'
				- Make changes persistent in vconsole.conf
					```console
					echo "KEYMAP=[your-keymap]" | tee -a /etc/vconsole.conf
					```
	- Configure Networking
		- Edit '/etc/network/interfaces'
			+ This is your network interfaces configuration file
		- Edit '/etc/resolv.conf'
			+ This is your DNS resolver nameserver configuration file
		- Edit '/etc/hostname'
			+ This is your hostname file; where you define the name mapped to this system on the network
			- Manually
				```console
				echo "<your-hostname>" | tee -a /etc/hostname
				```
			- Using systemd-firstboot
				```console
				systemd-firstboot --hostname=[your-hostname]
				```
		- Edit '/etc/hosts'
			+ This is your local 'hosts' file, this is where you map hostnames to IP Addresses so that the Operating System is able locally associate a network IP address to a host name
			- Default Values
				```
				127.0.0.1 localhost # This is your loopback interface; aka your 'localhost' address
				::1       localhost # This is your loopback interface; aka your 'localhost' address
				127.0.1.1 <Hostname>.localdomain <HostName> # This is your loopback interface; aka your 'localhost' address; mapped explicitly to your hostname
				```
			- Add matching entries to host file
				```console
				echo "127.0.0.1 localhost" | tee -a /etc/hosts
				echo "::1       localhost" | tee -a /etc/hosts
				echo "127.0.1.1 <your-hostname>.localdomain <your-hostname>" | tee -a /etc/hosts
				```
	- Installing Kernel
		- (Recommended) Install pre-built linux kernel package
			- Information
				+ The latest linux kernel package is known as 'kernel'
			- Install dependencies
				+ kernel
			```console
			sudo dnf install kernel
			```
		- Manual Compilation
			+ Similar to Gentoo
			- Steps
				1. Clone/Download the latest linux kernel 
				2. Make kernel configuration
					```console
					make menuconfig
					```
				3. Make kernel modules
					```console
					make
					```
				4. Install kernel modules
					```console
					make modules_install
					```
				5. Install kernel
					```console
					make install
					```
		- Install from seperate repository source
			- Information
				+ In Fedora, the linux kernel needs to be installed by repositories
				- Kernel Series
					+ kernel-vanilla-mainline           : the latest kernel in pre-release (aka “rc kernel”) stage.
					+ kernel-vanilla-mainline-wo-mergew : same to the previous, but exclude kernels from merge window.
					+ kernel-vanilla-stable             : latest stable kernel according to kernel.org.
					+ kernel-vanilla-stable-rc          : same to the previous, but also include the RC kernels.
					+ kernel-vanilla-fedora             : The latest point release of Fedora’s default Kernel series.
			- Setup 
				- Add Repository source to Kernel series
					- kernel-vanilla
						```console
						curl -s https://repos.fedorapeople.org/repos/thl/kernel-vanilla.repo | sudo tee /etc/yum.repos.d/kernel-vanilla.repo
						```
				- Install Kernel
					```console
					sudo dnf --enablerepo=[kernel-series] update
					```
			- Uninstall kernel from the above repository
				```console
				sudo dnf remove $(rpm -qa 'kernel*' | grep '.vanilla')
				```
	- Setting root password
		```console
		passwd
		```
	- Setup Bootloader
		- Grub
			- Information
				+ grub in dnf is aliased as 'grub2'
			- Install essential packages
				- MBR/MSDOS (BIOS)
					```console
					dnf install grub2
					```
				- GPT (UEFI)
					- Packages
						+ grub2-efi-x64 : GRUB(2) (U)EFI package for x64 architecture CPU
						+ grub2-efi-x64-modules : Modules for the grub2 EFI package
						+ shim : Shim is a simple software package that is designed to work as a first-stage bootloader on UEFI systems
					```console
					dnf install grub2-efi-x64 grub2-efi-x64-modules shim
					```
			- If Partition Table is
				- MBR (MSDOS)
					+ Install grub to boot partition and partition table
						```console
						grub2-install --target=[architecture] {--debug} [disk-label]
						```
			- Update grub
				- MBR/MSDOS (BIOS)
					- Make grub configuration file
						```console
						grub2-mkconfig -o /boot/grub2/grub.cfg
						```
				- GPT (UEFI)
					- Make grub-configuration file
						```console
						grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
						```
		- (Optional) Initialize RAM Filesystem (initramfs)
			- Notes
				+ Compared to Debian, ArchLinux and Gentoo; Fedora uses dracut to generate an initramfs
			- Using dracut
				- Documentations
					- Synopsis/Syntax
						```console
						dracut {options}
						```
					- Parameters
						- Options
							+ --regenerate-all : Generate an initramfs
				```console
				sudo dracut --regenerate-all --force
				```
	- User Management
		- Create user
			- Synopsis/Syntax
				```console
				useradd {options} -g [primary-group] -G [secondary-groups,] -d [home-directory] -s [shell] [username]
				```
			- Parameters
				- Positionals
					+ username: Your new user's username
				- Options/Flags
					+ -d : Specify a explicit/custom home directory for user
					+ -g : Your primary group to assign to the user
					+ -G : Your secondary groups to assign to the user; please seperate all groups with ',' delimiter
					+ -m : Specify to create a custom home directory for user 
					+ -s : Specify the shell to use; default: /bin/sh
			- Usage
				- General
					```console
					useradd -m -g [primary-group] -G [secondary-groups,] -d [home-directory] -s [shell] [username]
					```
		- Change user password
			```console
			passwd [username]
			```
		- Change to new user
			```console
			su - [username]
			```
		- Verify user
			+ User should display 'root' if valid
			```console
			sudo whoami
			```

### Post-Installation
- Install essential packages
	- Build/Development
		+ build-essential : Similar to 'base-devel' in ArchLinux
	- Microcode
		+ intel-microcode : For Intel-based processors
		+ amd64-microcode : For AMD-based 64-bit processors
	- Kernel
		+ linux-image-amd64 : For Linux Kernel AMD64 images 
		+ linux-image-generic : Generic Linux latest release kernel
		+ search in 'dnf search kernel'
	- Networking
		+ dhcpcd5 : The Dynamic Host Control Protocol (DHCP) Daemon; Equivalent to archlinux's 'dhcpcd' package
		+ network-manager : Network Manager package
		+ ntp : Network Time Protocol support package/utility
	- Fonts
	- Icons
- Create Swap files
	- Generate swapfile
		- Using fallocate
			```console
			sudo fallocate -l [size-of-swapfile] /swapfile
			```
	- Format swapfile
		```console
		sudo mkswap /swapfile
		```
	- Change permission to executable
		```console
		sudo chmod 0600 /swapfile
		```
	- Enable swapfile
		```console
		sudo swapon /swapfile
		```
	- Add swapfile to filesystems table (/etc/fstab)
		```console
		echo -e "# /swapfile" | sudo tee -a /etc/fstab
		echo -e "/swapfile none swap defaults 0 0" | sudo tee -a /etc/fstab
		```
- Setup Networking
	- Install packages
		+ dhcpcd5
		+ network-manager
	- Enable service 'NetworkManager'
		- Using systemd
			```console
			sudo systemctl enable NetworkManager
			```
		- Using service
			```console
			sudo service networking enable
			```
	- Start service 'NetworkManager'
		- Using systemd
			```console
			sudo systemctl start NetworkManager
			```
		- Using service
			```console
			sudo service networking start
			```
	- Managing DHCP
		```console
		sudo dhcpcd
		```
	- Manage WiFi
		- using nmcli
			+ The Network Manager Command Line Interface utility
			```console
			nmcli {options} <arguments>
			```
		- using nmtui
			+ The Network Manager Terminal User Interface utility
			```console
			nmtui
			```

## Wiki
### System Architectures
+ amd64 : For x86_64 architecture CPU
+ arm64 : For ARM 64-bit architecture CPU
+ armel
+ armhf
+ i386  : For general x86_64 architecture; use this if you dont know what is your CPU type
+ mips64el
+ mipsel
+ ppc64el
+ s390x

### Release Version Code-name
#### Fedora Workspace
+ 36
#### Package Groups
+ core : Fedora core/workspace

### Important Packages
+ build-essential : For coreutils, base development packages; equivalent to 'base-devel'
+ linux-image-amd64 : For x64|64-bit Linux kernel image
+ sudo
+ ntp : for Network Time Protocol
+ dhcpcd5 : For DHCP Daemon; Internet connectivity

## Resources
+ dnf ReadTheDocs: https://dnf.readthedocs.io/en/latest/command_ref.html
+ ArchLinux Manual - dnf: https://man.archlinux.org/man/dnf.8.en

## References
+ [YouTube - Animortis Productions - Fedora 36 Bootstrap Installation | Fedora 36 Workstation Command Line Instillation](https://www.youtube.com/watch?v=hjR37L2xC6g)
+ [My linux general starting steps](https://github.com/Thanatisia/SharedSpace/blob/main/Docs/Linux/Guides/Setup/General%20Flow.md)
+ [RHEL Documentation - Chapter 10: Managing custom software repositories](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/managing_software_with_the_dnf_tool/assembly_managing-custom-software-repositories_managing-software-with-the-dnf-tool)

## Remarks
