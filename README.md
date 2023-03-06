# Linux Distribution Installation Guides

My collection base-installation & post-installation install + Setup guides

## Table of Contents:
- [Background](#background)
- [Guides](#guides)
    - [ArchLinux](#archlinux)
    - [Debian (Debootstrap)](#debian-debootstrap)
    - [Gentoo](#gentoo)
    - [Fedora](#fedora)
    - [VoidLinux](#voidlinux)
    - [Templates](#templates)
- [References](#references)
- [Resources](#resources)
- [Remarks](#remarks)

## Background

```
Please reference and follow the steps in [Pre-Requisites/General Flow.md](Pre-Requisites/General%20Flow.md) before proceeding if they are not referenced in the root filesystem installation steps.
```

- Used and referenced in my Install Scripts found in 
    - [Arch Distro Install Script](https://github.com/Thanatisia/distro-installscript-arch)

- This is related to and part of
    + [Linux Setup Guides - PostInstallations](https://github.com/Thanatisia/linux-postinstallations)
    + [Linux Setup Guides - Window Manager](https://github.com/Thanatisia/linux-postinstallations/tree/main/Setup%20-%20Window%20Managers)

## Guides
### ArchLinux

- 2022-03-08 2214H
    + Tested and basic-installation guide installs successfully

- Folders
    + [ArchLinux Installation Guide](ArchLinux)

### Debian (via Debootstrap)

- 2022-03-08 2214H
    + Ongoing Tests and rewriting for easier reference and understanding
- 2022-03-14 1716H :
    + Still a WIP, in testing
- 2023-02-16 1557H : 
    + Tested, Working and contains the plans, what will be the template to remodel the other distribution guides after.
    + Pre-Requisites are found in [Pre-Installation General Flow Guide](Pre-Requisites) which is a mirror/reference to [My SharedSpace Repository](https://github.com/Thanatisia/SharedSpace/blob/main/Docs/Linux/Guides/Setup/General%20Flow.md)
- 2023-02-17 1411H
    + Cleaned up folder to only contain debian-based or usable files

- Folders
    + [Debian Installation Guide via Debootstrap (amd64)](Debian/amd64)

### General
> Distribution-agnostic Documentation and guides

### Gentoo

- 2022-03-08 2214H : 
    + Rewriting - Not-bootable as of [2022-03-08]
- 2022-03-13 0044H : 
    + Completed first rewrite draft, to be tested
- 2022-03-14 1716H : 
    + Still a WIP, in testing

- Folders
    + [Gentoo Installation Guide](Gentoo)

### Fedora

- 2023-03-05 2308 : 
    + Finished writing documentation
    + Performing testing and debugging
    - Issues Encountered
        + using ArchLinux has 'systemd-fsck: e2fsck has incompatible feature' error

### VoidLinux

- Folders
    + [VoidLinux Installation Guide](VoidLinux)


### Templates

- 2022-03-13 1015H:
    + Created Templates folder for storing all templates (related to distro installation) for reuse

- Folders
    + [Distro Install Templates](Templates)

## References

## Resources
+ SharedSpace Repository: https://github.com/Thanatisia/SharedSpace
+ Pre-Installation General Flow Guide: https://github.com/Thanatisia/SharedSpace/blob/main/Docs/Linux/Guides/Setup/General%20Flow.md

## Remarks

