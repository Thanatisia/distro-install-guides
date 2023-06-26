# Void Linux installation guide

## Table of Contents
- Information
    - Introduction
    - Project
- Setup/Preparation
    - Host
        - Network
        - Pre-Requisite and Dependency Installation
    - Disk/Filesystem
        - Partitions
        - Mounting
    - Basically the same as the other distro installation guides
- Installing 
    - Methods
        + XBPS
        + ROOTFS
        + TUI Installer
    - Base Filesystem
    - Configuration
- Documentation
- Wiki
- Resources
- References
- Remarks

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
- Host
    - Installing Pre-Requsites and Dependences
        - If you are using an existing distribution/system (non-NixOS)
            - Install Packages
                + xbps

## Installing
### Methods
+ XBPS Method    : Installs the base/root filesystem using the XBPS Package Manager to install the base system; Requires 'xbps' to be installed
+ ROOTFS Method  : Installs the base/root filesystem by unpacking a ROOTFS tarball
+ void-installer : Installs the base/root filesystem by executing the 'void-installer' TUI Installer

### Base Filesystem
> Please stick to just one method in this section
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
                - Setup
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
                - Generate Filesystems Table
                    ```console
                    genfstab -U [mount-point] | tee -a [mount-point]/etc/fstab
                    ```
    - Manually
        - (OPTIONAL) Copy and use the mount points from '/proc/mounts' in your fstab
            ```console
            cp [mount-point]/proc/mounts [mount-point]/etc/fstab
            ```
        - Edit your filesystems table file
            + Remove lines that refer to proc, sys, devtmpfs and pts
                ```console
                $EDITOR /etc/fstab
                ```
            - Add the UUID of the filesystem device/labels into the filesystems table 
                + Use 'blkid'
                ```console
                blkid [device-label]
                ```
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
        - Using arch-chroot
            ```console
            arch-chroot [mount-point] [shell]
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
    - Installation Configuration
        - Edit Hostname
            ```console
            $EDITOR /etc/hostname
            ```
        - Edit Hosts file
            ```console
            $EDITOR /etc/host
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
        - Edit sudoers file
            - Information
                + Sudoers file is in '/etc/sudoers'
                + Remove '%wheel ALL=(ALL:ALL) ALL'
            - Manually
                ```console
                EDITOR=[your-editor] sudo visudo
                ```
            - Automatically
                ```console

                ```
        - Set a root password
            ```console
            passwd
            ```
        - User Management
            - Create a new user account
                ```console
                useradd -m -g [primary-group] -G [secondary-groups,...] -d [home-directory-path] [username]
                ```
            - Change new user's password
                ```console
                passwd [username]
                ```
            - Change to new user
                ```console
                su - [username]
                ```
            - Test new user
                ```console
                sudo whoami
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
