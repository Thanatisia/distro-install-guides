# Debian via Debootstrap Base Installation Guide

## Information
### System Information
+ Architecture: AMD64

## Setup
### Dependencies
+ debootstrap

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
1. Install Debootstrap
	- Using Package Manager
		+ Using apt
			```console
			sudo apt install debootstrap
			```
		+ Using pacman
			```console
			sudo pacman install debootstrap
			```
	- Using Manual Installation/Compilation (Build from Source)
		- Create working directory
			+ i'll name it 'work'
			```console
			mkdir -p work

			# Change directory into working directory
			cd work
			```
		- Download debootstrap binary from Debian archive
			+ Debian Archive: http://ftp.debian.org/debian/pool/main/d/debootstrap/
			```console
			wget "http://ftp.debian.org/debian/pool/main/d/debootstrap/debootrstrap_[version-number]_all.deb"
			```
		- Extract Debootstrap binary from Debian archive
			```console
			ar -x debootstrap_[version-number]_all.deb
			```
		- Change to root directory
			```console
			cd /
			```
		- Read the extracted data gzip tar file and untar it
			```console
			zcat [path-to-working-directory]/data.tar.gz | tar xv
			```
		- To uninstall/remove
			+ Delete the '/usr/sbin/debootstrap' folder 
			```console
			rm -r /usr/sbin/debootstrap
			```
2. Bootstrap the necessary files required directly from the Debian archives mirrors
	- Run Debootstrap and Install the root filesystem
		- Notes
			+ Please refer to [System Architectures](#system-architectures) for a full list of system architecture and its keywords
			+ Please refer to [Release Version Code-name](#release-version-code-name) for a full list (as of writing) of Debian versions and its code names, you may find more in the Debian documentations.
		- Synopsis/Syntax
			```console
			/usr/sbin/debootstrap {other-options} --arch [your-system-architecture] [release-version-codename] [root-system-mountpoint] [debian-archive-mirror-url]
			```
		- Parameters
			- Positionals
				+ release-version-codename  : Your target debian version release codename; i.e. Debian 11 = Bullseye, Debian 12 = Bookworm; can also use Debian-based Distribution release names such as Kali, Ubuntu etc
				+ root-system-mountpoint    : The directory path which you mounted the root filesystem; i.e. /mnt
				+ debian-archive-mirror-url : The Debian archive mirror that you wish to retrieve the packages list from
			- Optionals
				+ --arch [your-system-architecture] : Specify the target system architecture; amd64, arm64, i386
				+ --include [packages-to-bootstrap] : Specify all packages to bootstrap and install into the filesystem from the start; similar to pacstrap (for ArchLinux); please seperate all package names with a ',' delimiter
		- Usage
			- amd64 with bullseye (Debian 11)
				```console
				/usr/sbin/debootstrap --include linux-image-amd64,vim,sudo,locales,build-essential --arch amd64 bullseye /mnt https://deb.debian.org/debian
				```
			- i386 (x64|64-bit) with bullseye (Debian 11)
				```console
				/usr/sbin/debootstrap --include linux-image,vim,sudo,locales,build-essential --arch i386 bullseye /mnt https://deb.debian.org/debian
				```
	- Verification
		+ Ensure that the standard output shows "Base system installed successsfully." message
3. Prepare chroot environment
	- Copying the host's DNS (Domain Name Service) resolver file to the mount point
		+ In order to use an internet connection in the chroot environment
		```console
		cp /etc/resolv.conf [mount-point]/etc/resolv.conf
		```
	- (OPTIONAL) Copy/Prepare the apt sources list
		- Using an apt-based system
			+ apt sources list can be found in '/etc/apt'
			```console
			cp /etc/apt/sources.list /mnt/etc/apt
			```
		- Using other systems
			- Generate your own apt sources list
				- Synopsis/Syntax:
					```console
					[archive-type] [repository-url] [distribution/release] [components]
					```
				- Parameters
					+ archive-type : The type of archive
					+ repository-url : The URL of the repository mirror that you want to download the packages from
					+ distribution/release : The release code name/alias or the release class of the debian version; i.e. bullseye; stable; testing; unstable
					- component : Parameters and keywords to indicate components like permissions or package types
						- Components
							+ main : Consists of DFSG-compliant packages
							+ contrib : Contains DFSG-compliant software packages; but have dependencies not in main
							+ non-free : Contains software that does not comply with the DFSG
				- Usage
					- deb : Archives containing binary packages
						```console
						deb [mirror] {release} [component-1] [component-2] [component-3]
						```
					- deb-src : Archives containing Source packages
						```console
						deb-src [mirror] {release} [component-1] [component-2] [component-3]
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
		- Using genfstab
			- Exit from chroot for a moment
				```console
				exit
				```
			- Generate filesystems table of mount point and write into target filesystem's '/etc/fstab' file
				```console
				genfstab -U [root-mount-point] | sudo tee -a [mount-point]/etc/fstab
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
4. Configure Base System
	- Optional Steps
		- If the target architecture is different from the host system
			+ You will first need to copy 'qemu-user-static' to the new host
				```console
				cp /usr/bin/qemu-[your-architecture]-static [root-mount-point]/usr/bin
				LANG=C.UTF-8 chroot [root-mount-point] qemu-[your-architecture] [shell
				```
			+ You need to finish the multi-stage bootstrap
				```console
				/debootstrap/debootstrap --second-stage
				```
		- Set the terminal definition to be compatible with the Debian base system
			```console
			export TERM=xterm-color
			```
		- Depending on the value of the TERM environment variable
			+ you may have to install the 'ncurses-term' package to get support for it
		- Create Device Files
			+ Folder '/dev/' only contains very basic device files
			+ Additional device files may be needed
			- Options
				+ Install the 'makedev' package and create a default set of static device files
					```console
					# Install package
					apt install makedev

					# Mount Process virtual filesystem
					mount none /proc -t proc

					# Change directory to '/dev'
					cd /dev

					# Make generic set of device files
					MAKEDEV generic
					```
				+ Manually create only specific device files using MAKEDEV
				+ Bind mount /dev from your host system on top of /dev in the target system
	- (Optional) Edit apt sources list
		> Please refer to the [Debian Sources List in Resources](#resources) to find a full list of mirrors and apt sources to add
		- Edit '/etc/apt/sources.list'
			```console
			$EDITOR /etc/apt/sources.list
			```
			- Recommended to add
				- Main release repository
					+ deb https://deb.debian.org/debian [release] main
					+ deb-src https://deb.debian.org/debian [release] main
				- Security repository
					+ deb https://deb.debian.org/debian-security/ [release]-security main
					+ deb-src https://deb.debian.org/debian-security/ [release]-security main
				- Updates repository
					+ deb https://deb.debian.org/debian [release]-updates main
					+ deb-src https://deb.debian.org/debian [release]-updates main
		- Update repository
			```console
			sudo apt update && sudo apt upgrade
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
			- Using dpkg-reconfigure
				+ Edit '/etc/adjtime' file
					```console
					$EDITOR /etc/adjtime
					```
				+ Reconfigure timezone data your time zone
					```console
					dpkg-reconfigure tzdata
					```
	- Setting Locales and Keyboard
		- Locales
			- Add locale
				- Methods
					1. Editing /etc/locale.gen and add locale
						- Information
							+ Edit '/etc/locale.gen' and uncomment your locale (i.e. en_[country-code].UTF-8 UTF-8) and other needed locales
						- Manual
							- Edit '/etc/locale.gen'
								```console
								$EDITOR /etc/locale.gen
								```
							+ Uncomment locales and save files
						- Automatic via sed
							- Information
								+ the regex '/[keyword]/s/^#//g' will uncomment the keyword from the file
								+ the regex '/[keyword]/s/^/#/g' will comment the keyword from the file
							- Documentation
								- Synopsis/Syntax
									```console
									sed {options} 'regex-pattern' [file-name]
									```
								- Parameters
									- Positionals
										+ regex-pattern : The regex pattern to substitute and edit within the file
										+ file-name : Specify the filename to edit
									- Options
										+ -i : Include regex
							```console
							sed -i '/[locale]/s/^#//g' /etc/locale.gen
							```
					2. Using dpkg-reconfigure
						+ Install 'locales' support package
							```console
							sudo apt install locales
							```
						+ Configure locales
							```console
							dpkg-reconfigure locales
							```
			- Generate locales
				```console
				locale-gen
				```
			- Set LANG variable according to your selected locale
				```console
				echo "LANG=[locale]" | tee -a /etc/locale.conf
				```
		- Keyboard (Optional)
			- Editing '/etc/vconsole.conf'
				- Make changes persistent in vconsole.conf
					```console
					echo "KEYMAP=[your-keymap]" | tee -a /etc/vconsole.conf
					```
			- Using dpkg-reconfigure
				- Install 'console-setup' support package
					```console
					apt install console-setup
					```
				- Configure keyboard configuration
					```console
					dpkg-reconfigure keyboard-configuration
					```
	- Configure Networking
		- Edit '/etc/network/interfaces'
			+ This is your network interfaces configuration file
		- Edit '/etc/resolv.conf'
			+ This is your DNS resolver nameserver configuration file
		- Edit '/etc/hostname'
			+ This is your hostname file; where you define the name mapped to this system on the network
			```console
			echo "<your-hostname>" | tee -a /etc/hostname
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
		- Install pre-built linux kernel package
			- Search for a linux kernel
				```console
				apt search linux-image
				```
			- Install packages
				```console
				apt install linux-image-[architecture]
				```
	- (Optional) Initialize RAM Filesystem 
		- Using update-initramfs
			- Documentations
				- Synopsis/Syntax
					```console
					update-initramfs {options}
					```
				- Parameters
					- Options
						+ -u : To update the initramfs (Initial RAM Filesystem) of the newest kernel
			```console
			sudo update-initramfs -u
			```
	- Setting root password
		```console
		passwd
		```
	- Setup Bootloader
		- Grub
			- Install essential packages
				```console
				apt install grub
				```
			- If Partition Table is
				- MBR (MSDOS)
					+ Install grub to boot partition and partition table
						```console
						grub-install --target=[architecture] {--debug} [disk-label]
						```
			- Update grub
				- Using update-grub
					```console
					update-grub
					```
				- Make grub configuration file
					```console
					grub-mkconfig -o /boot/grub/grub.cfg
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
		+ search in 'apt search linux-image'
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
#### Debian
+ 10 : Buster
+ 11 : bullseye
+ 12 : Bookworm
+ stable
+ unstable
+ testing
#### Ubuntu
> Version/Release Number : release code = official Release Name
+ 13.04 : raring = Raring Ringtail
+ 21.04 : hirsute = Hirsute Hippo

### Important Packages
+ build-essential : For coreutils, base development packages; equivalent to 'base-devel'
+ linux-image-amd64 : For x64|64-bit Linux kernel image
+ sudo
+ ntp : for Network Time Protocol
+ dhcpcd5 : For DHCP Daemon; Internet connectivity

### Package Source Mirrors
+ https://ftp.us.debian.org/debian
+ https://deb.debian.org/debian
+ https://security.debian.org

## Resources
+ Debian Debootstrap Archive: http://ftp.debian.org/debian/pool/main/d/debootstrap/
+ Debian Archive Mirrors: http://www.debian.org/mirror/list
+ Debian Sources List: https://wiki.debian.org/SourcesList

## References
+ [Debian Docs - AMD64 Stable Installation via Debootstrap](https://www.debian.org/releases/stable/amd64/apds03.en.html#idm4353)
+ [GitHub Gist - varqox/install_debian_with_debootstrap_howto.md](https://gist.github.com/varqox/42e213b6b2dde2b636ef)
+ [My linux general starting steps](https://github.com/Thanatisia/SharedSpace/blob/main/Docs/Linux/Guides/Setup/General%20Flow.md)

## Remarks
