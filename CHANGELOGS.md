# CHANGELOGS

## Table of Contents
+ [2024-03-02](#2024-03-02)
+ [2024-03-03](#2024-03-03)
+ [2024-03-04](#2024-03-04)

## Contents
### 2024-03-02
#### 2159H
- New
    - Added new directory 'Linux-From-Scratch/LFS-12.0/' for documentations of Linux From Scratch v12.0 Book
        - Added new document 'base-install.md' for a complete LFS 12.0 base installation (sysv) documentation
        - Added new document 'lfs-book-differences-between-sysv-and-systemd.md' for a write-up on the differences between the init systems 'sysv' and 'systemd'
- Updates
    - Updated document 'base-installation.md' in 'NixOS/amd64/'
        + NixOS Base Installation from Command Line via Bootstrapping

#### 2202H
- Updates
    - Updated document 'base-installation.md' in 'NixOS/amd64/'
        + Added Error Recovery steps

#### 2231H
- Updates
    - Updated document 'base-installation.md' in 'NixOS/amd64/'
        + Updated Partition Management steps with GPT/UEFI setup

#### 2234H
- Updates
    - Updated document 'base-installation.md' in 'NixOS/amd64/'
        + Added defaults information

#### 2239H
- Updates
    - Updated document 'base-installation.md' in 'NixOS/amd64/'
        - Added '--privileged' to docker run parameter
        - Added section 'Things to Note'
            + Added information about privileged
#### 2249H
- Updates
    - Updated document 'base-installation.md' in 'NixOS/amd64/'
        - Added 'configuration.nix' and 'hardware-configuration.nix' to step 'Edit the configuration file' 
            + To emphasize the need to edit both files
        - Added 'configuration.nix' and 'hardware-configuration.nix' to '### Files'

#### 2258H
- Updates
    - Updated document 'base-installation.md' in 'NixOS/amd64/'
        + Added information about dependency 'unshare'
        + Added installation information for package 'util-linux'
        + Added Resources for Nix Package manager installation

### 2024-03-03
#### 1510H
- Updates
    - Updated document 'base-installation.md' in 'NixOS/amd64/'
        + Added new dependency: squashfs-tools
        + Added new host system development environment method: Using the NixOS disk image

#### 1544H
- Updates
    - Updated document 'base-installation.md' in 'NixOS/amd64/' 
        + Added fix/bypass for 'you encountered an error regarding '--no-sandbox'

### 2024-03-04
#### 1534H
- Updates
    - Updated document 'base-installation.md' in 'NixOS/amd64/' 
        - Replaced 'nixos/nix' with the community-managed nix docker images 'nixpkgs/nix-unstable' (and nixpkgs/nix by extension)
            + Installable using 'nixpkgs/nix-unstable' as nixos/nix uses BusyBox and nix-unstable uses rootfs

