# Void Linux installation guide

## Table of Contents
- [Information](#information)
    - [Introduction](#introduction)
    - [Project](#project)
- [Setup/Preparation](#setup/preparation)
    - [Dependencies](#dependencies)
    - [Pre-Requisites](#pre-requisites)
        - Preparation Steps
        - Host System Preparation
        - Setup Keyboard Layout
        - Verify Boot Mode (Motherboard Firmware)
        - Update System Clock
        - Prepare Disk/Filesystem
        - Disk/Filesystem
            - Partitions
            - Mounting Disks
        - Basically the same as the other distro installation guides
- [Base Filesystem Installation](#base-filesystem-installation)
    - [Methods](#methods)
        + XBPS
        + ROOTFS
        + TUI Installer
    - [Bootstrap the Base Filesystem](#bootstrap-the-base-filesystem)
- [Configuration](#configuration)
- [Post-Installation](#post-installation)
- [Documentation](#documentation)
- [Wiki](#wiki)
- [Resources](#resources)
- [References](#references)
- [Remarks](#remarks)

## Information
### Introduction
```
Void Linux (aka Void) is an independent base distribution - like Debian, ArchLinux, NixOS etc - that focuses on choices

Void is a Stable release distribution, which means its packages only update every now and then
```

### Project
- Configuration Folder: 
- Configuration File:
- init system options
    - runit
    - openrc
    - systemd
- System Architecture Options
    +  x86_64       : Using glibc (Default)
    +  x86_64-musl  : Using musl
    +  i686 : PC Arcitecture computers
    +  aarch64

## Setup/Preparation
### Dependencies
- Host
    - If you are using an existing distribution/system (non-NixOS)
        - Install Packages
            + xbps
            - Optionals 
                - arch-install-scripts
                    - If you are using package manager
                        - apt
                            ```console
                            sudo apt install arch-install-scripts
                            ```
                        - pacman
                            ```console
                            sudo pacman -S arch-install-scripts
                            ```
                    - Build from Source
                        - Dependencies
                            + git
                            + base-devel
                            + m4
                            + make
                            + coreutils
                            + util-linux
                            + asciidoc
                        - Pre-Requisites
                        + Clone the repository
                            ```console
                            git clone https://github.com/archlinux/arch-install-scripts
                            ```
                        + Change directory into repository folder
                            ```console
                            cd arch-install-scripts
                            ```
                        + Compile/Build packages
                            ```console
                            make
                            ```
                        + Install compiled binaries into system
                            ```console
                            sudo make install
                            ```
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

## Base Filesystem Installation
### Methods
> Please stick to just one method in this section
+ XBPS Method    : Installs the base/root filesystem using the XBPS Package Manager to install the base system; Requires 'xbps' to be installed
+ ROOTFS Method  : Installs the base/root filesystem by unpacking a ROOTFS tarball
+ void-installer : Installs the base/root filesystem by executing the 'void-installer' TUI Installer

### Bootstrap the base filesystem
- XBPS Method
    - Pre-Requisites
        - Ensure that you have the following packages
            + xbps
        - Set your environment variables
            - Repository mirror URL: 
                + glibc installation : `REPO=https://repo-default.voidlinux.org/current`
            + Target system Architecture: `ARCH={x86_64|x86_64-musl|i686|aarch64}`
        - Create a XBPS RSA keys directory in your mount point's root var folder
            ```console
            mkdir -p [mount-point]/var/db/xbps/keys
            ```
        - Copy the XBPS keys from the installation medium to the created mount point's RSA keys directory
            ```console
            cp /var/db/xbps/keys/* /mnt/var/db/xbps/keys/
            ```
    - Bootstrap the root/base filesystem into the mount point
        - By installing the `base-system` metapackage
            ```console
            XBPS_ARCH=$ARCH xbps-install -S -r [mount-point] -R "$REPO" base-system
            ```

- ROOTFS Method
    - Pre-Requisites
        - ROOTFS Tarball
            - Download a ROOTFS tarball matching your architecture from [here](https://voidlinux.org/download/#download-installable-base-live-images-and-rootfs-tarballs)
                + Tarball file name convention: 'void-<...>-ROOTFS.tar.xz'
                ```console
                wget [rootfs-tarball-url]
                ```
            - Unpack the tarball
                ```console
                tar -xvf void-<...>-ROOTFS.tar.xz -C [mount-point]
                ```

### Configuration
> The remainder of this guide is common to both the XBPS and ROOTFS installation methods
- Prepare chroot environment
	- Copying the host's DNS (Domain Name Service) resolver file to the mount point
		+ In order to use an internet connection in the chroot environment
		```console
		cp /etc/resolv.conf [mount-point]/etc/resolv.conf
		```
    - Configure the filesystems table (fstab) file
        - If you are using an existing linux system that is not Void Linux
            - Generate a fstab file automatically
                - Using `genfstab`
                    - Pre-Requisites
                        - Install dependencies
                            + arch-install-scripts (According to the pacman/apt package names)
                    - Information
                        + genfstab is the filesystems table generator CLI utility, this generates a filesystems table based on the specified mount point you provided.
                        + Unfortunately, Void Linux repositories does not have the 'arch-install-scripts' package and/or genfstab utility, hence you will need to Build From Source from (https://github.com/archlinx/arch-install-scripts)
                    - Generate Filesystems Table
                        ```console
                        genfstab -U [mount-point] | tee -a [mount-point]/etc/fstab
                        ```
        - Manually
            1. (OPTIONAL) Copy and use the mount points from '/proc/mounts' in your fstab
                ```console
                cp [mount-point]/proc/mounts [mount-point]/etc/fstab
                ```
                - Edit your filesystems table file
                    + Remove lines that refer to proc, sys, devtmpfs and pts
                        ```console
                        $EDITOR /etc/fstab
                        ```
            2. Retrieve Block UUID of your devices
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
                        - Add the UUID of the filesystem device/labels into the filesystems table (/etc/fstab)
- Entering the Chroot
    - Pre-Requisites
        - Install packages required to enter the chroot
            - Either
                + xchroot (from xtools)
                + arch-chroot (from arch-install-scripts) : A wrapper for using chroot with the various pre-requisites; Useful and used with ArchLinux base installation from the command line via pacstrap (bootstrapping with pacman)
    - Chroot into mount point
        - Using xchroot
            + Required if using the Void linux live ISO
            ```console
            xchroot [mount-point] [shell]
            ```
		- Automatically via arch-chroot
			- the ArchLinux install scripts package has various very useful QoL utilities that wraps around steps
			- One of the utility is 'arch-chroot'
				+ Basically, the steps and mountings specificed in 'manually via chroot' are done in various similar chroot wrapper scripts such as in 'arch-chroot'
			- Pre-Requisite
				- Install 'arch-install-scripts'
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
    - ROOTFS-only
        - Update the package manager
            - The ROOTFS images generally contain out-of-date software 
                + due to being a snapshot of the time when they were built, 
                + and do not come with a complete `base-system`
            ```console
            xbps-install -Su xbps
            ```
        - Update package list
            ```console
            xbps-install -u
            ```
        - Install `base-system`
            ```console
            xbps-install base-system
            ```
        - Remove the void base-system bootstrapper metapackage
            ```console
            xbps-remove base-voidstrap
            ```
- Configure Base System
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
                - Notes
                    + This is to change configuration relating to the TTY font
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
    - Edit OpenRC configuration file
        ```console
        $EDITOR /etc/rc.conf
        ```
    - Edit locales
        - If installing a glibc distribution
            - Edit '/etc/default/libc-locales' and 
                + Uncomment desired 'locales (i.e. en_<country-code> UTF-8 UTF-8)'
                ```console
                $EDITOR /etc/default/libc-locales
                ```
            - Generate locale files
                ```console
                xbps-reconfigure -f glibc-locales
                ```
    - Make Initialization RAM filesystem (initramfs)
        - Using dracut
            - Check for linux modules in '/lib/modules'
                ```console
                ls -lha /lib/modules
                ```
            - Generate initramfs image
                + This will output the initramfs image file to '/boot/initramfs-<version>.img'
                ```console
                dracut --force --hostonly --kver <kernel-version>
                ```
        - Using mkinitcpio
            ```console
            mkinitcpio -P linux|linux-lts
            ```
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
    - Set a root password
        ```console
        passwd
        ```
    - Installing Bootloader
        - GRUB
            - Using grub-install
                - Setup
                    - Pre-Requisite
                        - Install Dependencies
                            + grub
                - Documentation
                    - Synopsis/Syntax
                        ```console
                        grub-install {options} [disk-label]
                        ```
                    - Parameters
                        - Positionals
                            + disk-label : This is the target disk/device's filesystem label (i.e. /dev/sdX etc)
                                - Label filename formats
                                    + SCSI/SATA devices : /dev/sdX
                                    + eMMC storage      : /dev/mmcblkX
                                    + NVME SSD          : /dev/nvmeNpX
                        - Optionals
                            - With Arguments
                                - (U)EFI Options
                                    - --bootloader-id=[Bootloader-OS-ID-name] : Specify the ID/name of the bootloader as UEFI doesnt have a standard label
                                        - Recommended Values
                                            + the name of the distribution (i.e. "Void" for VoidLinux, "Arch" for "ArchLinux)
                                    - --efi-directory=[EFI boot directory] : Explicitly specify the EFI boot directory
                                        - Recommended Values
                                            + /boot/efi
                                - General
                                    - --target=[system-architecture] : Specify the target system architecture
                                        - Architecture Types
                                            + i386-pc : For general motherboard
                                            + x86_64-efi : For EFI 64-bit architectures
                            - Flags
                                - General
                                    + --debug     : Enable verbose message output
                                - Troubleshooting
                                    + --no-nvram  : If EFI variables are not available
                                    + --removable : If installing onto a removable disk (such as a USB)
                - Filesystem Types 
                    - BIOS/MSDOS
                        - Install bootloader to filesystem drive (i.e. /dev/sda, /dev/sdb etc)
                            ```console
                            grub-install {--target=[system-architecture]} --debug /dev/{sdX|mmcblkX|nvmeNpX}
                            ```
                    - UEFI/GPT
                        - Pre-Requisites
                            - Install Dependencies
                                + grub-x86_64-efi : For 64-bit UEFI devices
                                + grub-i386-efi   : For i386 UEFI devices
                                + grub-arm64-efi  : For ARM64 UEFI devices
                        - Install bootloader to filesystem drive (i.e. /dev/sda, /dev/sdb etc)
                            ```console
                            grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id="[id-name]" --debug /dev/{sdX|mmcblkX|nvmeNpX}
                            ```
            - Generate GRUB configuration file
                ```console
                grub-mkconfig -o /boot/grub/grub.cfg
                ```
            - Update Grub after (re)generating configuration files
                ```console
                update-grub
                ```
    - Edit sudoers file
        - Information
            + Sudoers file is in '/etc/sudoers'
        - Instructions
            + Uncomment '%wheel ALL=(ALL:ALL) ALL' to allow members of group wheel to execute any command as sudo
        - Manually
            - Use 'visudo' to enter the sudo file safely
                ```console
                EDITOR=[your-editor] sudo visudo
                ```
        - Automatically
            ```console
		    sed -i 's/^#\s*\(%wheel\s\+ALL=(ALL:ALL)\s\+ALL\)/\1/' /etc/sudoers
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
				- Create a new user account
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
    - Finalization
        - (Optional) Use 'xbps-reconfigure' to ensure all installed packages are configured properly
            - Notes
                - This will 
                    + make 'dracut' generate an initramfs and 
                    + make GRUB generate a working configuration
                - Thus, if you have done the above 2, this is not necessary
            ```console
            xbps-reconfigure -fa
            ```

### Post-Installation
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
- Troubleshooting
    - To update the initramfs
        - Using update-initramfs
            ```console
            sudo update-initramfs -u
            ```
        - Using dracut
            ```console
            dracut /boot/initramfs-linux.img
            ```
        - Using xbps-reconfigure
            - Search 'ls /lib/modules' for the linux version of choice
            - Reconfigure linux version
                ```console
                xbps-reconfigure -f [linux<output-from-module>]
                ```

## Documentation
### Commands
- xbps-docs         : XBPS Documentations on command line
- xbps-query        : Queries and searches for packages
- xbps-reconfigure  : Reconfigures a package
- xbps-install      : Installs/Updates packages
    - Synopsis/Syntax
        ```console
        xbps-install {options} [package-name]
        ```
    - Parameters
        - Positionals
            + package-name : The target package
        - Optionals
            - With Arguments
                + -R [repository-mirror-URL] : Specify the repository mirror URL
            - Flags
                + -h : Displays help menu
                + -S : Install//Synchronize the package to the system
                + -u : Update the package
    - Usage
        - Update a package 
            ```console 
            xbps-install -u [package-name]
            ```
        - Update xbps 
            ```console 
            xbps-install -u xbps
            ```
        - Install (Synchronize) package 
            ```console
            xbps-install -S [package-name]
            ```
- void-installer    : To start the base system installation through a TUI application

## Wiki

### Environment Variables
+ XBPS Architecture: `XBPS_ARCH=$ARCH`

### Snippets and Examples
- Bootstrap the root/base filesystem into the mount point
    - Preparing 
        - Environment Variables
            + Target system architecture: `ARCH=x86_64`
            + Target repository mirror  : `REPO=https://default.voidlinux.org/current`
    - By installing the `base-system` metapackage
        ```console
        XBPS_ARCH=$ARCH xbps-install -S -r /mnt -R "$REPO" base-system
        ```

## Resources
+ [Void Linux Wiki - Install alongside Arch Linux](https://wiki.voidlinux.org/voidlinux_en_all_2021-04/A/Install_alongside_Arch_Linux)
+ [Void Linux Docs - Installation via chroot](https://docs.voidlinux.org/installation/guides/chroot.html)
+ [Void Linux Tarballs Download](https://voidlinux.org/download/#download-installable-base-live-images-and-rootfs-tarballs)

## References

## Remarks
