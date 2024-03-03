# NixOS Base Installation from Command Line via Bootstrapping

## Table of Contents
- Information
    - Introduction
    - Project
- Preparation
    - Host
        - Network
        - Pre-Requisite and Dependency Installation
    - Disk/Filesystem
        - Partitions
        - Mounting
    - Basically the same as the other distro installation guides
- Installing
    - Base Filesystem
- Documentations
- Wiki
- Resources
- References
- Remarks

## Information
### Introduction
```
NixOS is a 'containerized' distribution, not in the sense of being an immutable distribution,
but a distribution that contains a configuration that is like the "recipe" for the system.

To backup the configuration file is to backup the instructions to recreate the whole system. (Not a backup, please proceed to do backup beforehand)
```

### Project
- Defaults
    - NixOS uses (U)EFI and GPT specifications by default
        + Bootloader and partitioning uses UEFI settings

## Setup
### Dependencies
- nixos-generate-config : Base System Bootstrapper (like pacstrap for ArchLinux and debootstrap for Debian) 
    + Found in the nixos install tools (github.com/NixOS/nixos-install-tools)
- unshare
    + Found in the [nixos install tools](https://github.com/NixOS/nixpkgs/blob/nixos-23.11/pkgs/os-specific/linux/util-linux/default.nix#L127)
+ docker : (Optional) If your host is on a non-NixOS system and does not natively use the nix package manager
+ docker-compose : (Optional) If your host is on a non-NixOS system and does not natively use the nix package manager
- squashfs-tools : (Optional) If you are bootstrap installing NixOS from an existing non-NixOS distribution system
    + Contains 'squashfs' and 'unsquashfs'

### Pre-Requisites and Preparation
#### Host
- Disk Management
    - Format disk label
        ```bash
        sudo parted [disk-label] mklabel [msdos|gpt]
        ```

- Partition Management
    - Create new partitions
        - Syntax
            - MSDOS/MBR
                ```bash
                sudo parted [disk-label] mkpart [partition-type] [partition-filesystem] [starting-size] [ending-size]
                ```
            - GPT (UEFI)
                ```bash
                sudo parted [disk-label] mkpart [partition-label] [partition-filesystem] [starting-size] [ending-size]
                ```
        - Boot Partition
            - MSDOS/MBR
                ```bash
                sudo parted [disk-label] mkpart [partition-type] ext4 0% 1024MiB
                ```
            - GPT (UEFI)
                ```bash
                sudo parted [disk-label] mkpart [partition-label] ext4 0% 1024MiB
                ```
        - Root Partition
            ```bash
            sudo parted [disk-label] mkpart primary ext4 1024MiB 50%
            ```
        - (Optional) Home Partition
            ```bash
            sudo parted [disk-label] mkpart primary ext4 50% 100%
            ```

    - Format partitions
        - Syntax
            - Ext4
                ```bash
                sudo mkfs.ext4 [disk-label]{boot-partition-number}
                ```
            - FAT{8|16|32}
                ```bash
                sudo mkfs.fat -f {8|16|32} [disk-label]{boot-partition-number}
                ```
        - Boot Partition
            ```bash
            sudo mkfs.ext4 [disk-label]{boot-partition-number}
            ```
        - Root Partition
            ```bash
            sudo mkfs.ext4 [disk-label]{root-partition-number}
            ```
        - (Optional) Home Partition
            ```bash
            sudo mkfs.ext4 [disk-label]{home-partition-number}
            ```

    - Set boot partition as bootable
        - BIOS
            ```bash
            sudo parted [disk-label] set [boot-partition-number] boot on
            ```
        - ESP (Default)
            ```bash
            sudo parted [disk-label] set [boot-partition-number] esp on
            ```

#### Disk Mounting
- Create root partition mount point
    ```bash
    mkdir -pv /mnt
    ```

- Mount root partition to mount point
    ```bash
    mount [disk-label]{root-partition-number} [root-mount-point]
    ```

- Create boot and home partition mount point
    ```bash
    mkdir -pv /mnt/{boot,home}
    ```

- Mount remaining partitions partition to mount point
    ```bash
    mount [disk-label]{boot-partition-number} /mnt/[boot-mount-point]
    mount [disk-label]{home-partition-number} /mnt/[home-mount-point]
    ```

#### Setup System Development Environment
> Note: This section is option. If you are using a NixOS-based system, you may proceed to the Base/Root Filesystem installation step

- Using nix on host system
    - Install nix
        - Using package manager
            - apt-based
                ```bash
                apt install nix
                ```
            - pacman-based
                ```bash
                pacman -S nix
                ```
    - User Management
        - Create a dedicated user for nix packaging and add the user to the group 'nix-users'
            + To access the daemon socket
            ```bash
            useradd -g nix-users nixuser
            ```
        - Set password
            ```bash
            passwd nixuser
            ```
    - Enable and start service 'nix-daemon'
        - Enable service
            ```bash
            service nix-daemon enable
            ```
        - Start service
            ```bash
            service nix-daemon start
            ```
    - Restart system
        ```bash
        sudo reboot now
        ```
    - Add nix channel
        ```bash
        nix-channel --add https://nixos.org/channels/nixpkgs-unstable
        ```
    - Verify channel
        ```bash
        nix-channel --list
        ```
    - Update channel
        ```bash
        nix-channel --update
        ```

- Using docker
    - Notes
        - The nix docker image is using busybox
            + Hence, you will need to perform some pre-requisites first
    - Startup 'nixos/nix:latest' docker container
        ```bash
        docker run -itd --name=nix \
            --restart=unless-stopped \
            --privileged \
            -v /path/to/workdir:/workdir \
            -v [root-mount-point]:[root-mount-point]
            nixos/nix:[tag|version]
        ```

    - Enter the nix container
        ```bash
        docker exec -it nix /bin/sh
        ```

- Using NixOS disk image
    - Create directory to mount the ISO image files
        ```bash
        mkdir -p iso
        ```

    - Create directory to store the unsquashed root filesystem
        ```bash
        mkdir -pv squashfs/nix
        ```

    - Download the NixOS minimal disk image/iso from the [website](https://nixos.org/download.html)
        - Notes
            + Please note the latest version from the website
        ```bash
        wget https://channels.nixos.org/nixos-[version]/latest-nixos-minimal-x86_64-linux.iso
        ```

    - Modprobe loopback devices
        ```bash
        modprobe loop
        ```

    - Mount the ISO image to a media mount directory as loopback device
        ```bash
        sudo mount -o loop latest-nixos-minimal-x86_64-linux.iso [ISO-image-mount-point]/iso
        ```

    - Unsquash the squashfs file
        ```bash
        unsquashfs -d squashfs/nix/store [ISO-image-mount-point]/iso/nix-store.squashfs '*'
        ```

    - Change directory into 'squashfs'
        ```bash
        cd squashfs
        ```

    - Mount Virtual Kernel Devices from host system to container
        - Explanation
            - To have a working network connection, copy /etc/resolv.conf to squashfs/nix/etc. 
                + For a working chroot, you also need to bind /dev, /proc and /sys directories to the target system.
        - Make directories
            ```bash
            mkdir -pv etc dev proc sys
            ```
        - Copy /etc/resolv.conf from host to 'squashfs/nix/etc'
            ```bash
            cp /etc/resolv.conf etc/
            ```
        - Mount Virtual Kernel Devices
            ```bash
            for devices in dev proc sys; do
                mount --bind "/${devices}" "${devices}";
            done
            ```

    - Circumventing regular init sequence
        - Explanation
            - To properly chroot into the host system you must locate the packages named nixos and bash. 
                + The following commands may prove helpful
        - Setting INIT
            ```bash
            INIT=$(find . -type f -path '*nixos*/init')
            ```
        - Setting BASH
            ```bash
            BASH=$(find . -type f -path '*/bin/bash' | tail -n 1)
            ```

    - Replace further mentions of these files with your own results 
        - Notes 
            + Note the missing prefix in some uses.
        - Explanation
            - Edit the target system init script to start a bash session instead of systemd. 
                + As that is the last thing the script does, adding an interactive program does not pose a problem.
        ```bash
        sed -i "s,exec /.*systemd,exec /$BASH," $INIT
        ```

    - Chroot into root filesystem
        ```bash
        chroot . /$INIT
        ```

    - Add nix channel
        ```bash
        nix-channel --add https://nixos.org/channels/nixpkgs-unstable
        ```

    - Verify channel
        ```bash
        nix-channel --list
        ```

    - Update channel
        ```bash
        nix-channel --update
        ```

    - Set environment variable 'NIX_PATH'
        ```bash
        export NIX_PATH="nixpkgs=channel:nixos-[nixos-version]"
        ```

    - (Optional) After complete, unmount the mounted loopback device
        ```bash
        sudo umount {[mount-point]|/path/to/iso/image}
        ```

#### Toolings
- Install the Installation tool package using Nix package manager
    - Using nix-env
        - On NixOS
            ```bash
            nix-env -iA nixos.nixos-install-tools
            ```
        - On Non-NixOS
            ```bash
            nix-env -iA nixpkgs.nixos-install-tools
            ```
    - Using the NixOS configuration
        - Append the following Nix code to your NixOS configuration in '/etc/nixos/configuration.nix'
            ```nix
            environment.systemPackages = [
                pkgs.nixos-install-tools
            ];
            ```
    - Using nix-shell
        ```bash
        nix-shell -p nixos-install-tools
        ```

- Install the 'util-linux' package using Nix package manager
    - Using nix-env
        - On NixOS
            ```bash
            nix-env -iA nixos.util-linux
            ```
        - On Non-NixOS
            ```bash
            nix-env -iA nixpkgs.util-linux
            ```
    - Using the NixOS configuration
        - Append the following Nix code to your NixOS configuration in '/etc/nixos/configuration.nix'
            ```nix
            environment.systemPackages = [
                pkgs.util-linux
            ];
            ```
    - Using nix-shell
        ```bash
        nix-shell -p util-linux
        ```

## Installation
### Base/Root Filesystem
- Generate the root filesystem to your root partition
    - Explanation
        - This will bootstrap the distribution's base filesystem into the mount point's root directory
            + as well as the configuration file containing NixOS's recipe/installing configs
    - Synopsis/Syntax
        ```bash
        nixos-generate-config --root [root-mount-point]
        ```
    - Bootstrap Install to root mount point
        ```console
        nixos-generate-config --root /mnt
        ```

- Edit the generated configuration file
    - Explanation
        + Uncomment all settings you want in your system
    - Software flakes configuration.nix file
        ```console
        $EDITOR /mnt/etc/nixos/configuration.nix
        ```
    - Hardware flakes configuration.nix file
        ```console
        $EDITOR /mnt/etc/nixos/hardware-configuration.nix
        ```

- Install and Bootstrap NixOS into your base filesystem according to the configuration you edited
    - Explanation
        + The system will detect the mount point containing your NixOS configuration file and partition and 
        + install the base filesystem into the mount point's root directory
    - Options
        + `--root [root-filesystem-mountpoint]` : Explicitly specify the root partition mount point
    ```console
    nixos-install
    ```

## Documentation
### Commands
- nix-* : Handles the nix handling/systems
    - `nix-env {options} [package-name]` : Handles the Nix environment values such as Packages
        - Options
            + -A : 
            + -i : Install the specified package
- nixos-* : Handles the Nix operating system controls such as bootstrapping, installation to other systems etc
    - `nixos-generate-config {options}` : Generate the NixOS system configuration file that defines the system.
        - Options
            + `--root [mount-point]` : Explicitly specify the root filesystem to generate the configuration file in; Usually within the root partition to bootstrap the base filesystem into
    + `nixos-install` : Begin installation and bootstrapping of the Nix base packages into the mounted root partition

### Error Checking
- `error: path '/nix/store/[store-hash]-[package|file]' is not valid`
    - Error Type
        + This suggests a potential store issue
    - Possible Solution(s)
        1. Verify and Repair stores
            ```bash
            nix-store --verify --check-contents --repair
            ```

### Error Recovery
- If you accidentally destroyed your bootloader configurations 
    - Possible Scenarios
        - Because you accidentally wrote the wrong disk label (i.e. /dev/sda instead of /dev/sdb) in the NixOS flake configuration file
            + Which caused NixOS to run 'grub-install' in the wrong disk label
    - Solutions
        - Attach your drive to a secondary bootable device and start it up
            - Mount the device to your mount directory
                - Notes
                    + Just mount your root partition and boot partition
                - Mount root partition
                    ```bash
                    sudo mount [disk-label] [mount-point]
                    ```
                - Mount boot partition
                    ```bash
                    sudo mount [disk-label] [mount-point]/boot
                    ```
            - Chroot into the root filesystem
                - Using 'arch-chroot'
                    ```bash
                    sudo arch-chroot [root-filesystem-mount-point]
                    ```
            - Install Bootloader to the correct disk label
                - GRUB
                    ```bash
                    grub-install [disk-label]
                    ```
            - Generate Bootloader configurations to the boot partition
                - GRUB
                    ```bash
                    grub-mkconfig -o /boot/grub/grub.cfg
                    ```

## Wiki

### Things to note
- NixOS (or Nix-related system operations) wont run properly on docker (without --privileged or the likes) because it requires systemd
    - So functionalities like
        + Base/Root Filesystem Bootstrap Installation via nixos-install
    + will not work properly without '--privileged' passed into 'docker run', or 'privileged: true' set in docker-compose.yaml

### Files
+ /etc/nixos/configuration.nix : NixOS Software configuration
+ /etc/nixos/hardware-configuration.nix : NixOS Hardware configuration

### Folders
- /etc/ : System configuration and settings directory
    - nixos : Primary NixOS configurations and settings directory

## Resources

## References
+ [GitHub Gist - Vincibean - my-nixos-installation.md](https://gist.github.com/Vincibean/baf1b76ca5147449a1a479b5fcc9a222)
+ [NixOS - Wiki - Installation Guide](https://nixos.wiki/wiki/NixOS_Installation_Guide)
+ [NixOS - Wiki - Installing from Linux](https://nixos.wiki/wiki/Installing_from_Linux)
+ [Nix Package Manager - Wiki - Installation Guide](https://nixos.wiki/wiki/Nix_Installation_Guide)

## Remarks

