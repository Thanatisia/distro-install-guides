# Linux Distro CLI Base Installation Guide

## Table of Contents
- [Information](#information)
- [Preparation](#preparation)
    - [Pre-Requisites](#pre-requisites)
    - [Setup](#setup)
        - [Obtain Media](#obtain-media)
        - [Configure Network](#configure-network)
        - [Verify Boot Mode](#verify-boot-mode-uefi-bios)
        - [Disk Management](#disk-management)
- [Steps](#steps)
    1. [Date and Time](#date-and-time)
    2. [Stage Tarball](#stage-tarball)
    3. [Configure Compile Options](#configure-compile-options)
    4. [Generate fstab (File System Table)](#generate-fstab-file-system-table)
    5. [Install the Distro Base System](#install-the-distro-base-system)
    6. [Configuring Portage](#configuring-portage)
    7. [Configuring Portage make.conf variables](#configuring-portage-make-conf-variables)
    8. [Configure Timezone](#configure-timezone)
    9. [Configure Locales](#configure-locales)
    10. [Configuring Linux Kernel](#configuring-linux-kernel)
    11. [Configuring the System](#configuring-the-system)
        1. [Filesystem Information](#filesystem-information)
        2. [Networking Information](#networking-information)
        3. [System Information](#system-information)
        4. [Init and Boot Configuration](#init-and-boot-configuration)
    13. [Installing System Tools](#installing-system-tools)
    14. [Configuring the Bootloader](#configuring-the-bootloader)
    15. [Finishing Touches](#finishing-touches)
    16. [Post-Installation](#post-installation)
- [Resources](#resources)
- [References](#references)
- [Remarks](#remarks)

## Information

### Guides Information

+ Last Update: 2022-06-24 1533H

## Preparation

### Pre-Requisites

- Your Target Architecture
    + amd64
    + arm64
- Device Filesystem Table
    + Partition Schema

### Setup

#### Obtain Media

- Downloading Minimal Installation CD 
    - from mirror
        1. Go to 'releases/' directory
        2. Select your target architecture
        3. Select the autobuilds/ directory
            + https://bouncer.gentoo.org/fetch/root/all/releases/[architecture]/autobuilds/[build-folder]/image.iso
        4. If Architecture is
            - amd64 and x86 architectures (32-bit and 64-bit) 
                - Select either
                    + current-install-amd64-minimal/
                    + current-install-x86-minimal/
            - Others
                + current-iso/

- (OPTIONAL) Verify the downloaded files
    - Windows-based
        - Tools
            + GPG4Win
        - Import public keys of the Gentoo Release Engineering team
            - Refer to [References](#references) for the list of keys
        
    - Linux-based
        + Package: app-crypt/gnupg
        - Download the right set of keys from keyserver
            ```console
            gpg --keyserver hkps://keys.gentoo.org --recv-keys 0xBB572E0E2D182910
            ```
        - Use WKD to download the keys
            ```console
            gpg --auto-key-locate=clear,nodefault,wkd --locate-key releng@gentoo.org
            ```
        - Verify cryptographic signature in .asc file
            ```console
            gpg --verify [image-file].iso.asc
            ```
            
#### Configure Network

- Check Network Config
    ```console
    ip a s
    ```
    
- Check device name
    ```console
    ip link
    ```
- Active/Deactive WiFi Interface Device
    ```console
    ip link set dev [wifi interface] {up|down}
    ```

- If you are using the Installation ISO
    - (OPTIONAL) Configure proxies
        + Setup an HTTP proxy (for HTTP and HTTPS traffic)
            ```console
            export http_proxy="http://proxy.gentoo.org:8080"
            ```
        + Setup an FTP proxy
            ```console
            export ftp_proxy="ftp://proxy.gentoo.org:8080"
            ```
        + Setup an RSYNC proxy
            ```console
            export RSYNC_PROXY="proxy.gentoo.org:8080"
            ```
        + If proxy requires a username and password
            ```console
            http_proxy=http://username:password@proxy.gentoo.org:8080
            ```

    - (OPTIONAL) Automatic Network Configuration
        + (Default) Using net-setup script
            ```console
            net-setup eth0
            ```
            
    - (OPTIONAL) Using PPP (aka PPPoE - Power over Ethernet)
        ```console
        pppoe-setup # Configure Connection
        pppoe-start # Start Connection
        ```
        
    - (OPTIONAL) Enable DHCP Server
        - Pre-Requisites
            + dhcpcd
            - If Distro is Debian
                + dhcpcd
                    ```console
                    sudo apt(-get) install dhcpcd
                    ```
            - If Distro is Arch
                + dhcpcd
                    ```console
                    sudo pacman -S dhcpcd
                    ```
            - If Distro is Gentoo
                + net-misc/dhcpcd
                    ```console
                    emerge --ask net-misc/dhcpcd
                    ```
        - Using dhcpcd
            + Syntax : dhcpcd {options} eth0
            + Options
                + -HD : Require that the hostname and domainname provided by the DHCP server is used by the system
            ```console 
            dhcpcd -HD eth0
            ```
    
    - (OPTIONAL) Prepare Wireless Access (WiFi)
        - NOTE:
            + If the wireless network is setup with WPA or WPA2
                + wpa_supplicant needs to be used
        - Check device name
            ```console
            ip link
            ```
        - Ensure interface is active
            ```console
            ip link set dev [wifi interface] up
            ```
        - Check Wifi info
            ```console
            iw dev [wifi interface] info
            ```
        - Check current connection
            ```console
            iw dev [wifi-interface] link 
            ```
        - Connect to an open network
            ```console
            iw dev [wifi-interface] connect -w [wifi-ssid (name)]
            ```
        - Connect with a hex WEP key, prefix the key with 'd:'
            ```console
            iw dev [wifi-interface] connect -w [wifi-ssid (name)] key 0:d:[wifi-hex-wep key]
            ```
        - Connect with an ASCII WEP Key:
            ```console
            iw dev [wifi-interface] connect -w [wifi-ssid (name)] key 0:[your-password]
            ```
    
- Ping and check connectivity
    ```console
    ping -c 5 8.8.8.8
    ```

#### Verify Boot Mode (UEFI/BIOS)

- List efivars directory
    + If it displays without error - booted in UEFI
    + If it displays with error - booted in BIOS or CSM mode
    ```console
    ls /sys/firmware/efi/efivars
    ```
        
#### Disk Management

- Verify disk block
	```
	Note the device name
	- /dev/sdX
	i.e.
		/dev/sdb
		/dev/sdc
	```
	lsblk / sudo fdisk -l

- Prepare Disks
	- NOTES:
		- Multiple Tools for Partitioning
            - parted
                - Syntax: 
                    - Create Device Label (aka Reformat)
                        + parted [device_name] mklabel [partition_table]

                    - Creating new partition
                        - If Partition Table is MSDOS (MBR/BIOS)
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
                            + boot {on|off} : Set Boot to Boot Partition of MSDOS (MBR/BIOS) Partition Table
                            + esp {on|off} : Set Boot to Boot Partition of GPT Partition Table
            + fdisk
            + cgdisk

		- Disk Label:
			+ Create a [gpt] Disklabel type if the partition is GPT (UEFI) or 
			+ Create a [msdos] Disklabel type if the partition is MBR (BIOS)

		- Partitions:
			- In every instance in a GPT/(U)EFI Partition Table
				- A special bootable EFI system partition is required
					- Similar to MSDOS (BIOS/MBR)'s Boot Partition
				- Requires FAT32 while MSDOS's Boot Partition Requires EXT4
			* EFI system partition on a MBR partition table is identified by the Partition Type ID [EF]

	- Create Device Label
		+ If Partition Table is [msdos (MBR/BIOS)]
			```
			label type = msdos
			```
			parted /dev/sdX mklabel msdos

		+ If Partition Table is [gpt (UEFI)]
			```
			label type = gpt
			```
			parted /dev/sdX mklabel gpt

	- Create Partitions
		- If Partition Table is MSDOS (MBR/BIOS)
			```
			Syntax: parted [device_name] mkpart [partition_type] [partition_fileType] [start_size] [end_size]
	
			Example:
                ===================================================================================================================================================
                | disk_Label | partition_Label | mount_path | partition_Type | partition_fileType | partition_start_Size | partition_end_Size | partition_Options |
                ===================================================================================================================================================
				/dev/sdb1 Boot /mnt/boot primary ext4 0% 	    1024MiB	 Bootable
				/dev/sdb2 Root /mnt      primary ext4 1024MiB 	32768MiB
				/dev/sdb3 Home /mnt/home primary ext4 32768MiB	100%
			```
            + parted /dev/sdb mkpart primary ext4 0% 1024MiB
            + parted /dev/sdb mkpart primary ext4 1024MiB 32768MiB			
			+ parted /dev/sdb mkpart primary ext4 32768MiB 100%

		- If Partition Table is GPT (UEFI)
			```
			Syntax: parted [device_name] mkpart [partition_label] [partition_fileType] [start_size] [end_size]
	
			Example:
                ==================================================================================================================================
                | disk_Label | partition_Label | mount_path | partition_fileType | partition_start_Size | partition_end_Size | partition_Options |
                ==================================================================================================================================
				/dev/sdb1 Boot /mnt/boot fat32 0% 	    1024MiB	 Bootable
				/dev/sdb2 Root /mnt      ext4 1024MiB 	32768MiB
				/dev/sdb3 Home /mnt/home ext4 32768MiB	100%
			```
            + parted /dev/sdb mkpart "Boot" ext4 0% 1024MiB
			+ parted /dev/sdb mkpart "Root" ext4 1024MiB 32768MiB		
			+ parted /dev/sdb mkpart "Home" ext4 32768MiB 100%
		
	- Format Partitions
		```
		Syntax: 
			# File Type
			- Fat{16|32}
				mkfs.fat -f {16|32} /dev/sdX(n)
			- EXT4
                # sys-fs/e2fsprogs
				mkfs.ext4 /dev/sdX(n)
            - btrfs
                # sys-fs/btrfs-progs
                mkfs.btrfs /dev/sdX(n)
            - f2fs 
                # sys-fs/f2fs-tools
                mkfs.f2fs /dev/sdX(n)
            - jfs
                # sys-fs/jfsutils
                mkfs.jfs /dev/sdX(n)
            - reiserfs 
                # sys-fs/reiserfsprogs
                mkfs.reiserfs /dev/sdX(n)
            - xfs 
                # sys-fs/xfsprogs
                mkfs.xfs /dev/sdX(n)
            - vfat 
                # sys-fs/dosfstools
                mkfs.vfat /dev/sdX(n)
            - NTFS 
                # sys-fs/ntfs3g
                mkfs.ntfs /dev/sdX(n)
			- Swap
				mkswap /dev/sdX(n)
		```
		- If Partition Table is GPT (UEFI)
			+ mkfs.fat -F32 /dev/sdb1 (Boot Partition for GPT UEFI must be Fat32)
            + mkfs.ext4 /dev/sdb2
            + mkfs.ext4 /dev/sdb3
		- If Partition Table is MSDOS (MBR/BIOS)
			+ mkfs.ext4 /dev/sdb1
            + mkfs.ext4 /dev/sdb2
            + mkfs.ext4 /dev/sdb3

	- Set Bootable
		- If Partition Table is MSDOS (MBR/BIOS)
			```
			Syntax: parted $device_Name set <partition> boot on
			```
			+ parted /dev/sdb set 1 boot on
		- If Partition Table is GPT (UEFI)
			```
			Syntax: parted $device_Name set <partition> esp on
			```
			+ parted /dev/sdb set 1 esp on		

	- (OPTIONAL) Setup Swap Partition
		+ swapon /dev/sdX(n)

		- After chroot
			+ echo "# -- Swap Partition 1" | tee -a /etc/fstab
			+ echo "UUID=device_UUID none swap defaults 0 0" | tee -a /etc/fstab


- Mount Disks
    ```
    :: Mounting Scheme

    let:
        - Disk/Device Label: /dev/sdX
        - Disk Bootloader: MBR (MSDOS)
        - Partition Scheme:
            =============================================================
            | ROW_ID | partition_name | mount_path | partition_filetype |
            =============================================================
            + 1 | Boot | /mnt/gentoo/boot   | ext4
            + 2 | Root | /mnt/gentoo        | ext4
            + 3 | Home | /mnt/gentoo/home   | ext4
    ```
    - Mount the root volume to /mnt. For example, if the root volume is /dev/sdX2: 
        + mount /dev/sdX2 /mnt/gentoo

    - Make other directories (i.e. home, boot)
        - Home directory
            + mkdir -p /mnt/gentoo/home
        - Boot directory
            ```
            If Partition Table is GPT (UEFI):
                Typical Boot Partition Mount Points
                    /boot : The usual boot partition mount point and is the preferred method when directly booting on EFISTUB kernel from UEFI or booting it via a boot manager like systemd-boot
                    /efi : Is a replacement for the previously popular (and is possibly still used by other Linux distros) ESP mountpoint [/boot/efi]
                        - Use a bootloader which is capable of accessing the kernel(s) and initramfs image(s) that are stored elsewhere - typically /boot
            ```
            + mkdir -p /mnt/gentoo/boot

    - Mount remaining directories
        + mount /dev/sdX3 /mnt/gentoo/home
        + mount /dev/sdX1 /mnt/gentoo/boot
    
    - (OPTINAL) If /tmp needs to reside on a seperate partition
        + Change its permission after mounting
            ```console
            chmod 1777 /mnt/gentoo/tmp
            ```
               
## Steps

### Date and Time 

- Verify current date
    ```console
    date
    ```
- Manual Set
    ```console
    : "
    Syntax:
        MM : Month
        DD : Day
        hh : Hour
        mm : Minutes
        YYYY : Year
    "
    date 'MMDDhhmmYYYY'
    ```
- Automatic set
    + NOTE: This may be a privacy concern as you are communicating with an external Time server
    - If using Gentoo Live Environment
        + includes the 'ntpd' command in 'net-misc/ntp' package (for Network-Time-Protocol)
            ```console
            ntpd -q -g
            ```

### Stage Tarball

- Choosing a stage tarball
    - Library Compatibility
        + Multilib (32-bit and 64-bit) [Recommended]
        + No-multilib (pure 64-bit)
    - Init Systems
        + OpenRC
            ```
            Dependency-based init system
            ```
            + Gentoo's native and original init system
            + If you want to use something other than systemd
            + Due to historically, Gentoo is focused on OpenRC, has more documentations targeting OpenRC over systemd
        + Systemd [Recommended]
            + Is a modern SysV-style init and rc replacement for Linux systems
            + Ubiquitously used in other Linux Distros
            + Full supported by Gentoo

- Preparation
    + Assume Specifications to be according to Mount Point in [Disk Management](#disk-management)
    - Change directory to Gentoo mount point
        ```console
        cd /mnt/gentoo
        ```
        
- Download Tarball        
    - Using Graphical Browsers
        - Get the PASTED_STAGE_URL (Stage tarball URL)
            - Official Mirrors to get the PASTED_STAGE_URL
                + Go to [Download Section](https://www.gentoo.org/downloads/#other-arches)
            - Other Mirrors
                + Go to [Mirrors](https://www.gentoo.org/downloads/mirrors/)
        - Download using wget
            + Syntax : wget <PASTED_STAGE_URL>
                ```console
                wget <PASTED_STAGE_URL>
                ```
    - Using Command Line Browsers
        - Command Line Browsers
            + links
                + Gentoo : 'www-clients/links'
                ```console
                : "
                Options:
                    -http-proxy proxy.server.com:8080 
                        To use an HTTP Proxy with links
                "
                links {options} https://www.gentoo.org/download/mirrors
                ```
            + lynx
                + Gentoo : 'www-clients/lynx'
                - (OPTIONAL) If a proxy needs to be defined
                    + export http_proxy="http://proxy.server.com:port"
                    + export ftp_proxy="http://proxy.server.com:port"
                ```console
                lynx https://www.gentoo.org/download/mirrors
                ```
        - Select and Download Stage Tarball
            + Select a mirror
            + Move to 'releases/amd64/autobuilds/' directory
                - Structure : ${MIRROR_URL}/releases/{your-architecture}/autobuilds/
            + Select a stage file
                - Example: stag-3-[your-architecture]....tar.xz
            + Press 'd' to download (or the keybind for the application you're using)

- (OPTIONAL) Verifying and Validating Stage Tarball
    + Similar to the Minimal Installation CDs
    - Files required
        + .CONTENTS file : Contains a list of all files inside the stage tarball
        + .DIGESTS file : Contains checksums of the stage file in different algorithms
        + .DIGESTS.asc file : Contains checksums of the stage file in different algorithms, but is also cryptographically signed to ensure it is provided by the Gentoo project
    + Compare Checksums
        + Use 'openssl' to compare the output with the checksums provided by the .DIGESTS. or .DISGESTS.asc file
            ```console
            openssl dgst -r -sha512 stage3-[architecture]-[release]-[init_system].tar.(bz2|xz)
            ```
        + Use sha512 to compare
            ```console
            sha512sum stage3-[architecture]-[release]-[init_system].tar.(bz2|xz)
            ```
    + Validate whirlpool checksum
        ```console
        openssl dgst -r -whirlpool stage3-[architecture]-[release]-[init_system].tar.(bz2|xz)
        ```
    + Validate the cryptographic signature of the .DIGESTS.asc file using gpg to make sure the checksums are not tampered with
        ```console
        gpg --verify stage3-[architecture]-[release]-[init_system].tar.(bz2|xz){.DIGESTS.asc,}
        ```
        
- Extract Tarball
    + x : Extract
    + p : Preserve Permissions
    + v : Verbose
    + f : To denote that we want to extract a file
    + --xattr-include : Include preservation of the extended attributes in all namespaces stored in the archive
    + --numeric-owner : To ensure that the user and group IDs of the files being extracted from the tarball will remain the same as Gentoo's release engineering team intended
        ```console
        sudo tar -xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
        ```

### Configure Compile Options

- NOTES
    - /etc/portage/make.conf : Gentoo's portage make configuration file, runtime behaviours change dependending on the values (i.e. optimization variables as well as package merging variables) saved in the file.
        + to find more info: man 5 make.conf

    - Important variables
        + COMMON_FLAGS : Some common compiler flags to set for all languages
            - Flags
                + -march= OR -mtune= : Specifies the name of the target architecture
                + -O : (Capital 'O', not zero) Specifies the gcc optimization class flag
                    - Values
                        + -Os : Size-optimized
                        + -O0 : Zero, no optimization
                        + -O{1-3} : Optimization levels 1 to 3, -O2 is the recommended default, -O3 is known to cause problems when used system-wide
                + -pipe : Use pipes rather than temporary files for communication between the various stages of compilation. Uses more memory
        + CFLAGS : Defines the optimization flags for GCC C compiler
        + CXXFLAGS : Defines the optimization flags for GCC C++ compiler
        + USE : This defines rules or flags to pass for the source code compilation
            + (-flag) : To remove flag from compilation
            + (+flag) : To add flag from compilation
        + MAKEOPTS : Defines how many parallel compilations should occur when installing a package
            - Basically how many threads/cores should be used by the CPU for processing the install
            - Rule of Thumb : [(The number of threads (cores) the CPU has OR the total system RAM) / 2 GiB]
                - Have at least 2 GiB of RAM for every job specified
                    + -j6 = 12GiB
            - Syntax: MAKEOPTS="-j(n)"
                ```console
                # /etc/portage/make.conf
                MAKEOPTS="-j12"
                ```

- Edit /etc/portage/make.conf
    ```console
    $EDITOR $GENTOO_ROOT_MOUNT_PATH/etc/portage/make.conf
    ```
    
### Generate fstab (File System Table)
- /etc/fstab File Contents
    - Fields:
        + device_identifier : The block device or remote filesystem to be mounted.
            - Identifiers can be of various types such as 
                + Device Files
                + Filesystem Labels
                + Filesystem UUIDs (obtained from blkid)
                + Partition Labels
                + Partition UUIDs
        + Mount point/mount directory : The mmount poit where the partition should be mounted
        + Filesystem type : The filesystem used by the partition
            - Types
                + ext4
                + fat32
                + btrfs
                + zfs
                + reiserfs
                + ntfs
                + nfs
        + Mount Options : The mount options used by 'mount' when it wants to mount the partition on boot time.
            + As every filesystem has its own mount options, read 'man mount' for a full listing
            + Please seperate all mount options with comma delimiter ','
            - Mount Options
                + defaults : Set as default options
                + r,w : Read + Write
                + w : Write permission
                + r : Read-Only permission
                + noauto : Do not automatically mount the partition; Need to manually mount the partition everytime they want to use it
        + Dump Check : Used by dump to determine if the partition needs to be dumped or not
            + Can be left as '0' (zero)
        + Filesystem check order : Used by 'fsck' to determine the order in which filesystems should be checked if the system wasnt shut down properly
            - Order:
                + 0 : Filesystem Check is not necessary
                + 1 : Priority '1', Root must be 1
                + 2 : Priority '2'
            - Examples:
                + Root filesystem should have '1'
                + Other filesystems should have 2 (or 0 if a filesystem check is not necessary)
    - Structure:
        ```
        device  mount-point filesystem  mount-options dump-check filesystem-check-order
        ```

- Get UUID of filesystem using 'blkid'
    - Syntax : blkid [disk/device label]
    - Parameters:
        + Disk/Device Label : The device/partition name (i.e. /dev/sdX)
    ```console
    blkid
    ```

- Manual
    - Edit /etc/fstab
        ```console
        $EDITOR /etc/fstab
        ```
    - Append Filesystem Label and Filesystem UUID to /etc/fstab
        - Using device label/name
            ```console
            # Boot Partition (/boot)
            /dev/sdX1   /boot   ext4    defaults    0 2
            
            # Root Partition (/)
            /dev/sdX2   /       ext4    defaults    0 1
            
            # Home partition
            /dev/sdX3   /home       ext4    defaults    0 1
            ```
        - Using partition UUID
            ```console
            # Boot Partition (/boot)
            UUID='partition UUID from blkid'   /boot   ext4    defaults    0 2
            
            # Root Partition (/)
            UUID='partition UUID from blkid'   /       ext4    defaults    0 1
            
            # Home partition
            UUID='partition UUID from blkid'   /home       ext4    defaults    0 1
            ```

- Automatic (using genfstab)
	- Pre-Requisite: 
		- If genfstab is not installed / Using another distro:
			```
			Package Name : arch-install-scripts
			```
			- If Distro is Debian:
				+ sudo apt(-get) install arch-install-scripts
			- If Distro is Arch:
				+ sudo pacman -S arch-install-scripts
            - If Distro is Gentoo:
                + sudo emerge --ask sys-fs/genfstab : For genfstab alone
                + sudo emerge --ask dev-util/arch-install-scripts
                
	- Generate filesystem table
		```
		Generate an fstab file (use -U or -L to define by UUID or labels, respectively)
		Syntax: genfstab {options} [root-mount-point] >> {mount-point}/etc/fstab
		Options:
			-U : To get the UUID of the partitions
			-L : To get the label of the partitions
		```
		+ genfstab -U /mnt/gentoo >> /mnt/gentoo/etc/fstab

- Verify fstab file
    ```
    File: {mount-point}/etc/fstab
    ```
    + cat /mnt/gentoo/etc/fstab

### Install the Distro Base System

#### Mirror and Repository Check
```
- If using an existing Linux install/build that is non-Gentoo
    + You can skip this step
```
- (OPTIONAL) Selecting Mirrors
    - NOTES
        + Please seperate all mirror links with space delimiter (link-1' 'link-2)
        - Default
            - Default distfiles mirrors (Approved by mirror admin team)
                + GENTOO_MIRRORS="http://distfiles.gentoo.org"
    - To manually add a Mirror
        + Add a Mirror into the 'GENTOO_MIRRORS=' variable in /etc/portage/make.conf
            ```console
            echo "# Gentoo Portage Mirrors" | sudo tee -a $GENTOO_ROOT_MOUNT_PATH/etc/portage/make.conf
            echo "GENTOO_MIRRORS=\"https://gentoo_mirror_link\"" | sudo tee -a $GENTOO_ROOT_MOUNT_PATH/etc/portage/make.conf
            ```
    - Automatic
        + Refer to [Gentoo Wiki Mirrors](https://wiki.gentoo.org/wiki/GENTOO_MIRRORS) for Mirrors to add into make.conf
        + Install Package: 'app-portage/mirrorselect'
            ```console
            emerge --ask app-portage/mirrorselect
            ```
        + Choose a mirror
            ```console
            mirrorselect -i -o >> $GENTOO_ROOT_MOUNT_PATH/etc/portage/make.conf
            ```   
            
- Configure Gentoo ebuild repository
    - Information
        + Directory: /etc/portage/repos.conf
        + File: /etc/portage/repos.conf/gentoo.conf
        + Template configuration file : /usr/share/portage/config/repos.conf
    - Create the directory if it doesnt exist
        ```console
        mkdir -p $GENTOO_ROOT_MOUNT_PATH/etc/portage/repos.conf
        ```
    - Copy the Gentoo repository configuration file provided by Portage to repos.conf directory
        ```console
        cp $GENTOO_ROOT_MOUNT_PATH/usr/share/portage/config/repos.conf $GENTOO_ROOT_MOUNT_PATH/etc/portage/repos.conf/gentoo.conf
        ```
    - Verify gentoo.conf
        ```console
        ls -l $GENTOO_ROOT_MOUNT_PATH/etc/portage/repos.conf/gentoo.conf # Check that it exists
        cat $GENTOO_ROOT_MOUNT_PATH/etc/portage/repos.conf/gentoo.conf # Read the file
        ```

#### Prepare for Chrooting

- Copy DNS information to ensure that networking still works even after entering the new environment
    ```console
    # De-reference to ensure that if the file is a symbolic link, the link's target file is copied instead of the symbolic link
    cp --dereference /etc/resolv.conf $GENTOO_ROOT_MOUNT_PATH/etc
    ```

- Mount necessary filesystems
    + /proc
        + Is a pseudo-filesystem (it looks like regular files, but is generated on-the-fly). The linux kernel exposes information to the environment
        ```console
        mount --types proc /proc $GENTOO_ROOT_MOUNT_PATH/proc
        ```
    + /sys
        + Is a pseudo-filesystem, Similar to /proc, was once meant to replace, and is more structured than /proc
        ```console
        mount --rbind /sys $GENTOO_ROOT_MOUNT_PATH/sys
        mount --make-rslave $GENTOO_ROOT_MOUNT_PATH/sys
        ```
    + /dev
        + Is a regular filesystem, partially managed by the Linux device manager (usually udev), which contains all device files
        ```console
        mount --rbind /dev $GENTOO_ROOT_MOUNT_PATH/dev
        mount --make-rslave $GENTOO_ROOT_MOUNT_PATH/dev
        ```
    + /run
        + Is a temporary file system used for files generated at runtime, such as PID files or locks
        ```console
        mount --bind /run $GENTOO_ROOT_MOUNT_PATH/run
        mount --make-rslave $GENTOO_ROOT_MOUNT_PATH/run
        ```

- (OPTIONAL) If using an existing Linux distro OR non-Gentoo installation media
    ```console
    # Make /dev/shm a proper tmpfs mount up front
    test -L /dev/shm && \
        rm /dev/shm && \
        mkdir /dev/shm
        
    mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm
    
    # Ensure that mode 1777 is set
    chmod 1777 /dev/shm /run/shm
    ```

#### Entering into the new Environment

- Chroot
    + Syntax: chroot [mount_directory_root] [shell]
    ```console
    # Chroot into mount point
    chroot $GENTOO_ROOT_MOUNT_PATH /bin/bash
    
    # Source /etc/profile
    source /etc/profile
    
    # Temporarily modify primary prompt (PS1) 
    export PS1="(chroot) ${PS1}"
    ```

- Check Partition and Mounts
    lsblk

- (OPTIONAL) Mounting the Boot Partition
    - If Boot partition is not mounted (Assume to be partition 1)
        ```console
        mount /dev/sdX1 /boot
        ```
    - Update your /etc/fstab after update
        + Refer to [Generate fstab (File System Table)](#generate-fstab-file-system-table)
 
### Configuring Portage

- Install a Gentoo ebuild repository snapshot from the web
    + The snapshot contains a collection of files that informs Portage about available software titles for installation
        - Examples
            + Which profiles the system administrator can select, package or 
            + profile-specific news items etc.
    - Use 'emerge-webrsync'
        + Fetch the latest snapshot from one of gentoo's mirrors and install it onto the system
        + Recommended for those who are behind restrictive firewalls - uses HTTP/FTP protocols for downloading the snapshots
            and saves network bandwidth
        + If you have no network or bandwidth restrictions
            - cn skip to the next session
        ```console
        emerge-webrsync
        ```
        
- (OPTIONAL) Update the Gentoo ebuild repository
    + If you need the last package updates up to 1hr
    + Update the Gentoo ebuild repository (that was fetched through emerge-webrsync) to the latest state
        ```console
        emerge --sync {--quiet}
        ```

- (OPTIONAL) Reading news items
    + After the Gentoo ebuild repository is synchronized, Portage may output informational news items
    + Syntax: eselect news [actions]
    - Actions
        + list: List an overview of the available news items
            ```console
            eselect news list
            ```
        + read: Read the news items
            ```console
            eselect news read
            ```
        + purge: Remove news items once they have been read and will not be re-read anymore
            ```console
            eselect news purge
            ```
    - Manual: man news.eselect

- Choose a profile
    - Notes
        + A profile is a building block for any Gentoo system.
        + It specifies the default value for USE, CFLAGS and other important variables according to the selected profile
        + profile comes in the form of the architecture, if its a desktop, gnome-specific, kde-specific etc.
        + It also locks the system to a certain range of package versions
        + Recommended to stay with the default
        + selected item is highlighted by an asterisk '*'
    - Selecting a profile
        - Syntax: eselect profile [actions]
        - Actions
            + list: List all available profile symlink targets
                ```console
                eselect profile list
                ```
            + set [n]: To select and set a different profile for the system
                ```console
                eselect profile set n
                ```
        - No-multilib (pure 64-bit) 
          - Use a 'no-multilib' profile (*/no-multilib)

- Update the @world set
    - Update the system's @world set so that a base can be established
        - This is necessary so that the system can apply any updates or USE flag changes
            + that have appeared since the stage-3 was built and from any pfoile selection
        ```console
        emerge --ask --verbose --update --deep --newuse @world
        ```

### Configuring Portage make.conf variables

- Configure the USE variable
    + in /etc/portage/make.conf
    + the USE variable configures compiling with or without optional support for certain items for programs.
    - Syntax: 'USE=""'
        + (-flag) : To remove flag from compilation
        + (+flag) : To add flag from compilation
    - Examples:
        + USE="gnome gtk -kde -qt5" : Compile programs for GNOME with GTK+ support, but remove KDE and QT5 support
        + USE="-gtk -gnome qt5 kde dvd alsa cdr" : Enabling flags for a KDE/Plasma based system with DVD, ALSA and CD recording support
        + USE="-X acl alsa" : Disable support for X graphical environments with Access Control List (Security - ACL) support and ALSA (Audio) support
    - Notes:
        + The default USE settings are placed in the make.defaults files of the Gentoo profile used by the system
        - A full description on the available USE flags can be found in
            + /var/db/repos/gentoo/profiles/use.desc
    - Edit 'USE=' in /etc/portage/make.conf
        ```console
        $EDITOR /etc/portage/make.conf
        
        > # USE="[flags]"
        ```
        
- To check the currently active settings in the make.conf
    + Syntax: emerge --info | grep ^[variable]
        ```console
        # Check currently active USE settings
        emerge --info | grep ^USE
        ```

- (OPTIONAL) Configuring the ACCEPT_LICENSE variable
    + Portage uses the ACCEPT_LICENSE variable to determine which pages to allow without prompting the user for the licenses previously accepted.
    + Exceptions can be made per-package in /etc/portage/package.license
    - Syntax: 'ACCEPT_LICENSE="-* @LICENSE_NAME"
    - Licenses
        + @GPL-COMPATIBLE : GPL compatible licenses approved by the Free Software Foundation
        + @FSF-APPROVED : Free software licenses approved by the FSF (Includes @GPL-COMPATIBLE)
        + @OSI-APPROVED : Licenses approved by the Open Source Initiative
        + @MISC-FREE : Miscelleneous licenses that are probably free software (i.e. follows the Free Software Definition) but are not approved by either the FSF or OSI
        + @FREE-SOFTWARE : Combines @FSF-APPROVED, @OSI-APPROVED and @MISC-FREE
        + @FSF-APPROVED-OTHER : FSF-approved licenses for "Free Documentation" and "Works of practical use besides software and documentation" (including fonts)
        + @MISC-FREE-DOCS : Miscellenous licenses for Free Documents and other works (including fonts) that follows the 'free' definition but are NOT listened in @FSF-APPROVED-OTHER
        + @FREE-DOCUMENTS : Combines @FSF-APPROVED-OTHER and @MISC-FREE-DOCS
        + @FREE : Metaset of all licenses with the freedom to use, share, modify and share modifications. Combines @FREE-SOFTWARE and @FREE-DOCUMENTS
        + @BINARY-REDISTRIBUTABLE : Licenses that at least permit free redistribution of the software in binary form. Includes @FREE
        + @EULA : License agreements that try to take away your rights. These are more restrictive than "all-rights-reserved" or require explicit approval
    - Examples: 
        + Sample License Acceptance
            ```console
            # For linux-firmware
            echo "sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE" | tee -a /etc/portage/package.license
            ```

### Configure Timezone 

- Files
    + Timezone Data : /etc/timezone
    + Zone Info : /etc/localtime

- To get available timezone regions
    ```console
    ls /usr/share/zoneinfo
    ```

- To get available your City Timezone
    ```console
    ls /usr/share/zoneinfo/[Region]
    ```

- Setup Timezone 
    ```
    Assume Timezone Information:
        Region : Asia
        City : Singapore
    ```
    - OpenRC
        - Write to /etc/timezone
            + Syntax : echo "[Region]/[City]" > /etc/timezone
                ```console
                echo "Asia/Singapore" > /etc/timezone
                ```
        - Reconfigure the 'sys-libs/timezone-data' package
            + Automatically update the /etc/localtime file
                ```console
                emerge --config sys-libs/timezone-data
                ```
    - Systemd
        - Create Symlink for Region
            + Syntax : ln -s /usr/share/zoneinfo/[Region]/[City] /etc/localtime
                ```console
                ln -sf /usr/share/zoneinfo/Asia/Singapore /etc/localtime
                ```
                
### Configure Locales

- Files:
    + /etc/locale.gen : Contains your locales (i.e. en_US.UTF-8, en_SG.UTF-8 etc)

- Modify locale.gen
    ```
    Structure:
        <your_locale_code> <your_text_format (i.e. UTF-8)> 
    ```
    - Manual
        + Edit locale.gen
            ```console
            $EDITOR /etc/locale.gen
            ```
        + Uncomment or type in to your locale
            - Examples:
                + "en_US.UTF-8 UTF-8"
                + "en_SG.UTF-8 UTF-8"
    - Automatic
        - To uncomment from locale.gen
            + Syntax : sed -i '/{keyword}/s/^#//g' <file>
        - To comment in locale.gen
            + Syntax : sed -i '/{keyword}/s/^/#/g' <file>
        ```console
        # Uncomment your locale in /etc/locale.gen
        sed -i '/en_US.UTF-8/s/^#//g' /etc/locale.gen
        sed -i '/en_SG.UTF-8/s/^#//g' /etc/locale.gen
        ```
        
- Generate Locale
    ```console
    locale-gen
    ```

- Verify that selected locales are now available
    ```console
    locale -a
    ```
    
- Select Locales
    - Manual
        - Files
            + OpenRC : /etc/env.d/02locale
            + Systemd : /etc/locale.conf 
        - Set system locale definitions in the files
            - Examples
                ```console
                LANG="en_SG.UTF-8"
                LC_COLLATE="C.UTF-8"
                ```
            - Automatic
                ```console
                echo "LANG=\"en_SG.UTF-8\"" | tee -a {/etc/locale.conf | /etc/env.d/02locale}
                echo "LC_COLLATE=\"C.UTF-8\"" | tee -a {/etc/locale.conf | /etc/env.d/02locale}
                ```
    - Automatic (using eselect)
        - Syntax : eselect locale [action]
        - Actions
            + list : List all available locale targets
            + set [n] : Select and set the specified locale
        ```console
        eselect locale list     # List all available locale targets
        eselect locale set n    # Set and select correct locale
        ```
        
- Reload Environment
    ```console
    env-update && \
        source /etc/profile && \ 
        export PS1="(chroot) ${PS1}"
    ```
    
### Configuring Linux Kernel

- Install linux firmware
    - Notes
        + Some devices requires additional firmware to be installed on the system before they will operate correctly
        + Examples:
            + Network Interface
        + Most firmware for modern hardware are available in the 'sys-kernel/linux-firmware' package
    + Package : 'sys-kernel/linux-firmware'
    ```console
    emerge --ask sys-kernel/linux-firmware
    ```

- (OPTIONAL) Install CPU Microcodes
    - Intel CPU Microcode
        + Package : 'sys-firmware/intel-microcode'
        ```console
        emerge --ask sys-firmware/intel-microsode
        ```

- Preparing for Linux Kernel Configuration
    - (OPTIONAL) To enable installation of the latest kernel version (Nightly/Testing build)
        + Add 'echo "sys-kernel/gentoo-sources ~amd64"' to /etc/portage/package.accept_keywords/kernel
            ```console
            echo "sys-kernel/gentoo-sources ~amd64" >> /etc/portage/package.accept_keywords/kernel
            ```

    - Install sys-kernel/gentoo-sources
        + This will intall the Linux Kernel Sources in '/usr/src/' using the specific kernel version in the path
        + It will not create a symlink by itself without 'USE=symlink' being enabled on the chosen kernel sources package
        ```console
        emerge --ask sys-kernel/gentoo-sources
        ```
    
    - Select a kernel to use
        ```
        Create a symbolic link called /usr/src/linux that will link to the target linux kernel
        ```
        - Manual
            - Syntax : ln -s [src-actual-file] [dest-target-link-file]
            ```console
            ln -s [custom-linux-kernel] /usr/src/linux 
            
            # /usr/src/linux -> [custom-linux-kernel]
            ```
        - Automatic (using eselect)
            - Notes
                + 'eselect kernel set n' will create a symbolic link called linux that will link to the target linux kernel selected by the command
            - Syntax : eselect kernel [actions]
            - Actions:
                + list : List all available kernel symlink targets
                + set [n] : Select and set the specified kernel
            ```console
            eselect kernel list     # List all available kernel symlink targets
            eselect kernel set n    # Set kernel (option) n
            ```

- Linux Kernel Configuration and Compilation
    - Manual Configuration
        - Install 'sys-apps/pciutils'
            + It is vital to know the system when a kernel is configured manually
            + 'sys-apps/pciutils' contains the 'lspci' command
            ```console
            emerge --ask sys-apps/picutils
            ```
        - (OPTIONAL) If you are using the installation CD
            + Run 'lsmod' to see what kernel modules the installation CD uses
            + Might provide a nice hint on what to enable
            ```console
            lsmod
            ```
        - Change Directory to Kernel source directory (/usr/src/linux)
            ```console
            cd /usr/src/linux
            ```
        - Fire up menu-driven configuration screen
            - NOTES
                - Please refer to the 
                    + [Kernel Configuration Guide](https://wiki.gentoo.org/wiki/Kernel/Gentoo_Kernel_Configuration_Guide)
                    + [Kernel Installation - Gentoo Base Install (AMD64)](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Kernel#manual-configuration)
                + For what to enable
            ```console
            make menuconfig
            ``` 
        - Linux Kernel Compilation
            - NOTES
                + Recommend to set 'MAKEOPTS="-jX"' WHERE X = number of cores you have on the device
                    - To allow multithreading and speeding up compilation
            + compile the kernel modules and
            + install the modules
            ```console
            make && make modules_install
            ```

        - Install and Copy the Kernel image to /boot
            + This will copy the kernel image to /boot together with the System.map file and the Kernel configuration file
            ```console
            make install
            ```
        
        - (OPTIONAL) If using genkernel only for generating an initramfs
            - NOTES
               + If using genkernel, it should beused for  both building the kernel and the initrfs 
            ```console
            genkernel --kerel-config=/path/to/kernel.config
            ```

        - (OPTIONAL) Building an initramfs
            + initramfs : Initial RAM-based File System
            - Typical reasons to have
                - Important file system locations (i.e. /usr/ or /var/) are on seperate partitions
                    + With an initramfs, these partitions can be mounted using the tools available inside the initramfs
            - To Install an initramfs
                ```
                The initramfs will be stored in /boot/
                ```
                + Install 'sys-kernel/dracut'
                    ```console
                    emerge --ask sys-kernel/dracut
                    ```
                + Generate an initramfs
                    ```console
                    dracut -kver=[linux-kernel-version]
                    ```
                + To list the files starting with initramfs
                    ```console
                    ls /boot/initramfs*
                    ```

    - Automatic Configuration (using genkernel)
        - Install 'sys-kernel/genkernel'
            ```console
            emerge --ask sys-kernel/genkernel
            ```
        - (OPTIONAL) If you have not yet edited your /etc/fstab
            - Edit the /etc/fstab file so that the line containing /boot/ as second field
                + Has the first field pointing to the right device
            - Refer to [Generate fstab (File System Table)](#generate-fstab-file-system-table)
  
        - Compile kernel sources
            + Once completed, a kernel, full set of modules and init ram disk (initramfs) will be created.
            + Syntax : genkernel all 
            ```console
            genkernel all
            ```
        
        - (OPTIONAL) Note down your kernel version number, kernel names and initrd
            + This will be used when the bootloader configuration file is edited
        
        - Verify vmlinu* and initramfs* files are created
            ```console
            ls /boot/vmlinu* /boot/initramfs*
            ```

    - (OPTIONAL) Alternative: Using distribution kernels
        - Notes
            + Distribution Kernels are ebuilds that cover the complete process of
                + Unpacking the kernel
                + Configuring the kernel
                + Compiling the kernel
                + Installing the Kernel
            + The kernels are upgraded to new versions due to @world upgrade without a need for manual action
            + Distribution kernel default to a configuration supporting the majority of hardware
                + Can be customized via '/etc/portage/savedconfig'
        - Installing correct installkernel
            - If using systemd-boot (formerly gummiboot)
                - Install 'installkernel-gentoo'
                    ```console
                    emerge --ask sys-kernel/installkernel-systemd-boot
                    ```
            - If using a traditional /boot layout (i.e. GRUB, LILO, etc), the gentoo variant should be installed by default
                - Install 'installkrnel-gentoo'
                    ```console
                    emerge --ask sys-kernel/installkernel-gentoo
                    ```
        - Installing a distribution kernel
            - To build a kernel with Gentoo patches from source
                ```console
                emerge --ask sys-kernel/gentoo-kernel
                ```
            - To avoid compiling the kernel sources locally and use precompiled kernel images
                ```console
                emerge --ask sys-kernel/gentoo-kernel-bin
                ```
        - Upgrading and cleaning up
            + Once the kernel is installed, the package manager will automatically upgrade it to newer versions. The previous versions will be kept until the package manager is requested to clean up stale packages.
            + periodically run to save space
                ```console
                emerge --depclean
                ```
        - Post-install/upgrade tasks
            - Manually rebuilding the initramfs
                ```console
                emerge --ask @module-rebuild
                ```
         
### Configuring the System

#### Filesystem Information

- If you still have not added your partition to your /etc/fstab 
    + Please refer to [Generate fstab (File System Table)](#generate-fstab-file-system-table)

#### Networking Information

- Set Hostname
    - Notes
        + The hostname can be changed
        + The hostname is used within the domain
            ```
            domain (Home Network) -> hostname (Device)
            ```
    - OpenRC
        - Variable : 'hostname'
        - Edit the file '/etc/conf.d/hostname' with your hostname
            ```console
            echo "hostname=\"Your-Hostname\"" | sudo tee -a /etc/conf.d/hostname
            ```
    - Systemd
        - Manual
            - Edit the file '/etc/hostname' with your hostname and domain name (OPTIONAL)
                ```console
                $EDITOR /etc/hostname
                
                # /etc/hostname
                # [1] Your-Hostname
                # [2] Your-Hostname.domain-name
                ```
        - Automatic
            - If you have a custom domain name
                ```console
                echo "Your-Hostname.domain-name" | sudo tee -a /etc/hostname
                ```
            - If you just want to change your hostname
                ```console
                echo "Your-Hostname" | sudo tee -a /etc/hostname
                ```
        - Temporarily change your hostname 
            - Using hostnamectl
                + Syntax : hostnamectl hostname [your-hostname]{.your-domain_name}
                ```console
                hostnamectl hostname MyHostname{.localhost}
                ```

- Set Hosts
    - Notes
        + The /etc/hosts file contains information about the Network Environment
        + Helps in resolving Host names to IP addresses for hosts that are not resolved by the name server
    - Structure
        ```console
        # This defines the current system and must be set
        127.0.0.1   localhost
        
        # Optional definitions of extra systems on the network
        # Map IP Address in the network to a hostname (aka Device)
        # Syntax : [ip-address] [hostname].[network-domain-name] [hostname]
        ::1         localhost
        127.0.1.1   hostname.homenetwork  hostname
        ```
    - File : /etc/hosts
    - Manual
        - Edit the /etc/hosts file
            ```console
            $EDITOR /etc/hosts
            ```
        - Add your network information into the file
        - Save your file
    - Automatic
        ```console
        echo "127.0.0.1 hosntmae.localhost" | sudo tee -a /etc/hosts
        echo "::1       hostname.localhost" | sudo tee -a /etc/hosts
        echo "127.0.1.1 hostname.your-network-domain-name your-hostname" | sudo tee -a /etc/hosts
        ```

- (OPTIONAL) Set Domain name
    - Notes
        + The /etc/conf.d/net file does not exist by default, it needs to be created
        + Set the domain name in '/etc/conf.d/net'
        - This is only necessary if the ISP or Network Administrator says its necessary
            + And/or if the network has a DNS server BUT not a DHCP server (static IP)
        - If no domain name is configured
            - Users will notice they get "This is hostname.(none)" messages at their login screen
            - This should be fixed by editing /etc/issue and deleting the string '.\O' from that file
    - OpenRC
        - Variable : 'dns_domain_lo'
        - Edit the file '/etc/conf.d/net' with your domain name
            ```console 
            echo "dns_domain_lo=\"Your-Network-Domain\""
            ```
    - Systemd
        - Edit the file '/etc/hostname' with your domain name in the following structure
            ```console
            # /etc/hostname
            your-hostname.network-domain-name
            ```

- (OPTIONAL) Set NIS Domain name
    - OpenRC
        - Edit the file '/etc/conf.d/net' with your NIS domain name
            ```console
            echo "nis_domain_lo=\"Your-NIS-Domain\""
            ```

- Configure Network Interface
    - (OPTIONAL) Enable DHCP Server
        - Pre-Requisites
            + net-misc/dhcpcd
                ```console
                emerge --ask net-misc/dhcpcd
                ```
        - Enable Service
            - OpenRC
                ```console
                rc-update add dhcpcd default
                ```
            - Systemd
                ```console
                systemctl enable --now dhcpcd
                ```
        - Start Service
            - OpenRC
                ```console
                rc-service dhcpcd start
                ```
            - Systemd
                ```console
                systemctl start dhcpcd
                ```
                
+ For More Networking Configuration, please refer to [Configure Network](#configure-network)

#### System Information

- Set Root Password
    - Notes
        + If the username is left out in passwd, the d
    - Syntax : passwd [username]
    - Parameters
        + username : The target user to change password. 
            - DEFAULT: If this field is empty, the command will change the root password
    ```console
    passwd
    ```

#### Init and Boot Configuration
- OpenRC
    - OpenRC System Configuration
        - Files:
            + /etc/rc.conf : To configure the services, startup and shutdown of a system
        - Edit /etc/rc.conf and review settings
            ```console
            $EDITOR /etc/rc.conf
            ```
    
    - Keyboard Configuration
        - File : /etc/conf.d/keymaps
        - Edit /etc/conf.d/keymaps to set keymappings
            ```console
            $EDITOR /etc/conf.d/keymaps
            ```
    - Hardware Clock Options
        - Notes:
            + If the hardware clock is not using UTC, then it is necessary to set clock="local" in the file
        - File : /etc/conf.d/hwclock
        - Edit /etc/conf.d/hwclock to set clock options
            ```console
            $EDITOR /etc/conf.d/hwclock
            ```
- Systemd
    - Run systemd-firstboot to ensure the system is setup correctly
        ```console
        systemd-firstboot --prompt --setup-machine-id
        ```
    - Run preset-all
        ```console
        systemctl preset-all
        ```

### Installing System Tools
```
Some tools are missing from the stage-3 archives because several packages provide the same functionality

- The user now has to choose which ones to install
```

- (OPTIONAL) System Logger
    - NOTE: 
        + Systemd users can usually skip this step unless they specifically want a syslog (system logger).
        + Systemd includes journald which handles the same functionality
    - Packages:
        + app-admin/sysklogd    : Offers the traditional set of system logging daemons; good option for beginners
        + app-admin/syslog-ng   : An advanced system logger; Requires additional configuration for anything beyond logging to one big file; For more advanced users
            - Recommended Add-ons:
                + logrotate
        + app-admin/metalog     : A highly configurable system logger
    
- (OPTIONAL) Cron Daemon
    + Cron is used in GNU/Linux as a timer utility, in that it activates specified commands after a specified amount of time
    - Packages
        + sys-process/bcron
        + sys-process/dcron
        + sys-process/fcron
        + sys-process/cronie (Recommended)
            - To Install
                ```console
                emerge --ask sys-process/cronie
                ```
            - To Enable
                - OpenRC
                    ```console
                    rc-update add cronie default
                    ```
                - Systemd
                    ```console
                    systemctl enable cronie
                    ```

- (OPTIONAL) File Indexing
    + In order to index the filesystem to provide faster file location capabilities (i.e. B* Tree in Database Systems)
    - Packages
        + sys-apps/mlocate
            ```console
            emerge --ask sys-apps/mlocate
            ```
            
- (OPTIONAL) Remote Access
    - SSH
        + Pre-Requisite
            + ssh
        + sshd is the SSH Daemon
        - Enable sshd
            - OpenRC
                ```console
                rc-update add sshd default
                ```
            - Systemd
                ```console
                systemctl enable sshd
                ```
        - Start sshd
            - OpenRC
                ```console
                rc-update start sshd
                ```
            - Systemd
                ```console
                systemctl start sshd
                ```
        - If serial console access is needed
            - configure agetty
            - OpenRC
                + Uncomment the serial console section in /etc/inittab
                    - Manual
                        - Edit /etc/inittab
                            ```console
                            $EDITOR /etc/inittab
                            ```
                        - Uncomment '# SERIAL CONSOLES'
            - Systemd
                - Enable 'getty@tty1.service'
                    ```console
                    systemctl enable getty@tty1.service
                    ```

- (OPTIONAL) Time Synchronization
    - Done via the NTP protocol
    - Packages:
        - net-misc/chrony
    - Install Chrony
        ```console
        emerge --ask net-misc/chrony
        ```
    - Enable Chrony Daemon
        - OpenRC
            ```console
            rc-update add chronyd default
            ```
        - Systemd
            ```console
            systemctl enable chronyd
            ```

- (OPTIONAL) Filesystem Tools
    - To use certain filesystems in other systems (i.e. Remote/Network File (Sharing) Server)
        + You need to install the required filesystem utilities
        - Generally for 
            + checking the filesystems integrity
            + creating additional filesystems
            + Compatibility for that filesystem
    - Filesystem Packages
        ============================================
        | filesystem       | package               |
        | :=============== | ====================: |
        | Ext4             | sys-fs/e2fsprogs      |
        | Btrfs            | sys-fs/btrfs-progs    |
        | NTFS             | sys-fs/ntfs3g         |
        | XFS              | sys-fs/xfsprogs       |
        | ReiserFS         | sys-fs/reiserfsprogs  |
        | JFS              | sys-fs/jfsutils       |
        | VFAT (FAT32,...) | sys-fs/dosfstools     |
        | ZFS              | sys-fs/zfs            |
        ============================================

### Configuring the Bootloader

- Select a bootloader
    - General
        - GRUB (commonly known as GRUB2) : sys-boot/grub
        - LILO
        - syslinux
    - (U)EFI Bootloaders
        - EFIBootmgr
    
- Configure Bootloader
    - GRUB2
        - (U)EFI 
            - (OPTIONAL) Create /boot/grub folder if not created  
                ```console
                mkir -p /boot/grub
                ```
            - (OPTIONAL) If your system supports UEFI partition tables
                - Ensure that GRUB_PLATFORMS="efi-64" is enabled (should be by default) in /etc/portage/make.conf
                    ```console
                    cat /etc/portage/make.conf
                    ```
                    
                - Add GRUB_PLATFORMS="efi-64" into /etc/portage/make.conf if not enabled
                    + To build EFI functionality in package
                    ```console
                    echo  'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
                    ```
            - Merge bootloader
                - (OPTIONAL) If GRUB2 was emerged without enabling GRUB_PLATFORMS="efi-64"
                    - Add the line above into /etc/portage/make.conf
                    - Recalculate @world set package
                        ```console
                        emerge --ask --update --newuse --verbose sys-boot/grub
                        ```
            - Install Bootloader
                - Notes
                    + Install the necessary GRUB files to the /boot/grub/ directory
                    - If the /boot partition was not formatted as a FAT variant (FAT32 etc.)
                        + Modify the '--efi-directory' option to the root of the EFI System Partition
                - Pre-Requisite
                    + Ensure that EFI system partition has been mounted before running grub-install
                ```console
                grub-install --target=x86_64-efi --efi-directory=/boot 
                ```
        - MBR/BIOS (MSDOS) 
            + No Configuration required
            - Merge bootloader
                ```console
                emerge --ask --verbose sys-boot/grub
                ``` 
            - Install Bootloader
                + Install the necessary GRUB files to the /boot/grub/ directory
                ```console
                grub-install [disk/device_label]
                ```
        - Configure Bootloader
            - Generate GRUB Bootloader Config into /boot/grub/grub.cfg
                - Generate the GRUB2 configuration based on the user configuration specified in the 
                    + '/etc/default/grub' file and
                    + '/etc/grub.d/scripts'
                - Normally no configuration is needed by users as GRUB2 will automatically detect which kernel to boot
                    + The highest one available in /boot/ and what the root filesystem is
                - Possible to append Kernel Parameters in /etc/default/grub using the Environment Variable 
                    + GRUB_CMDLINE_LINUX
                ```console
                grub-mkconfig -o /boot/grub/grub.cfg
                ```
    - LILO
        + Stands for the LInuxLOader
        + Tried and true workhorse of Linux Bootloaders
        + It lacks features when compared to GRUB
        - Install Bootloader Package
            ```console
            emerge  --ask sys-boot/lilo
            ```
        - Configure Bootloader
            - Notes
                - If the root filesystem is 'JFS'
                    + Add an 'append="ro"' line after each boot item since
                        + JFS needs to replay its log before it allows read-write mounting
            - Edit /etc/lilo.conf
                ```console
                $EDITOR /etc/lilo.conf
                ```
        - Install Bootloader
            ```console
            /sbin/lilo
            ```
    - efibootmgr
        + sys-boot/efibootmgr application is not a bootloader
            + it is a tool to interact with UEFI firmware and update its settings
        + EFIBootmgr is for the minimalist 
        + After reboot, a boot entry called "Gentoo" will be available
        - Install Package
            ```console
            emerge --ask sys-boot/efibootmgr
            `````
        - Create /boot/efi/boot/ locate
            ```console
            mkdir -p /boot/efi/boot
            ```
        - Copy the kernel into this location, call it 'bootx64.efi'
            ```console
            cp /boot/vmlinuz-* /boot/efi/boot/bootx64.efi
            ```
        - Tell the UEFI firmware that a boot entry called 'Gentoo' is to be created
            ```console
            efibootmgr --create --disk [disk-label] --part [partition_number] --label [UEFI Partition Label] --loader "\efi\boot\bootx64.efi"
            ```
        - (OPTIONAL) If an initiram RAM filesystem (initramfs) is used, add the proper boot option to it
            ```console
            efibootmgr -c -d [disk-label] -p [partition_number] -L [UEFI Partition Label] -l "\efi\boot\bootx64.efi" initrd='\initramfs-file-name'
            ```

### Finishing Touches

- Disk Cleanup
    - Removing tarballs
        ```console
        rm /stage3-*.tar.*
        ```

- Exit chroot
    ```console
    exit
    ```
    
- Unmount mounted partitions
    ```console
    sudo umount -l $GENTOO_ROOT_MOUNT_PATH/dev{/shm,/pts,}
    sudo umount -l $GENTOO_ROOT_MOUNT_PATH
    ```

- Reboot system (if in the same device)
    ```console
    sudo reboot now
    ```
        
### Post-Installation

- User Management
    - Groups
        + wheel : Have super user support and able to use 'su(do)'
        + users : For regular user
        + audio : Be able to access the audio devices
        + cdrom : Be able to directly access optical devices
        + floppy : Be able to directly access floppy devices
        + games : Be able to play games
        + portage : Be able to access portage restricted resources
        + usb : Be able to access USB devices
        + video : Be able to access video capturing hardware and doing hardware acceleration
    - Create User and add into group
        - Create User Account
            ```
            Syntax: useradd -m -g [primary-group] -G [secondary-groups] -d [home-directory] -s [shell] {username}
            ```
            + useradd -m -g wheel -G user,audio,video -d /home/profiles/user -s /bin/bash user

        - Change user password
            ```
            Syntax: passwd [username]
            Parameters:
                username : The user you want to change password
                    Default: Change root password
            ```
            + passwd user

        - Verify User
            - Switch to user
                ```
                Syntax: su - {username}
                ```
                + su - user

            - Check if can use sudo
                ```
                - Result should be 'root'
                ```
                + sudo whoami

- Install essential packages
    - Essential Packages
        + sudo
        ```console
        emerge --ask sudo
        ```
        
- Device and Driver Support
    - (OPTIONAL) Get PCMCIA working
        - Install 'sys-apps/pcmciautils'
            ```console
            sudo emerge --ask sys-apps/pcmciautils
            ```

- Set sudo priviledges
	- Manual:
		```
		use 'visudo' to enter the sudo file safely
		- Uncomment to allow members of group wheel to execute any command
		```
		visudo
		> uncomment: %wheel ALL=(ALL:ALL) ALL
	- Automatic:
		```
		CHANGELOG:
			ALL=(ALL) => ALL=(ALL:ALL) as of 2022-03-15 update
		```
		sed -i 's/^#\s*\(%wheel\s\+ALL=(ALL:ALL)\s\+ALL\)/\1/' /etc/sudoers
  
## Resources

+ [Gentoo CD Images Downloads](https://www.gentoo.org/downloads/)
+ [Gentoo List of Gentoo Release Engineering team public keys (Signatures Page)](https://www.gentoo.org/downloads/signatures/)
+ [Gentoo Downloads - Mirrors](https://www.gentoo.org/downloads/mirrors)
+ [Gentoo Wiki - Mirrors](https://wiki.gentoo.org/wiki/GENTOO_MIRRORS)
+ [Gentoo Downloads](https://www.gentoo.org/downloads/#other-arches)
+ [Gentoo Wiki - @world set](https://wiki.gentoo.org/wiki/World_set_(Portage)
+ [Gentoo Wiki - Localization](https://wiki.gentoo.org/wiki/Localization/Guide)
+ [Gentoo Wiki - Microcodes](https://wiki.gentoo.org/wiki/Microcode)
+ [Gentoo Kernel Configuration Guide](https://wiki.gentoo.org/wiki/Kernel/Gentoo_Kernel_Configuration_Guide)
+ [Gentoo Base Installation - AMD64 - Kernel](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Kernel)
+ [Gentoo Wiki - Config Snippets](https://wiki.gentoo.org/wiki/Project:Distribution_Kernel#Using_.2Fetc.2Fkernel.2Fconfig.d)
+ [Gentoo Wiki - Syslinux](https://wiki.gentoo.org/wiki/Syslinux)

## References

## Remarks

