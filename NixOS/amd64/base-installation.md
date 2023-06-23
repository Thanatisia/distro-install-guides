# NixOS installation guide

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
- Configuration Folder: /etc/nixos/
- Configuration File: /etc/nixos/configuration.nix

## Preparation
- Host
    - Installing Pre-Requsites and Dependences
        - If you are using an existing distribution/system (non-NixOS)
            - Install Packages
                + nix

## Installing
### Base Filesystem
- Generate distribution Base Filesystem configuration file
    - This will bootstrap the distribution's base filesystem into the mount point's root directory
        + as well as the configuration file containing NixOS's recipe/installing configs
    + Using `nixos-generate-config --root [mount-point]`
    ```console
    nixos-generate-config --root /mnt
    ```
- Edit the generated configuration file
    ```console
    $EDITOR /mnt/etc/nixos/configuration.nix
    ```
- Install and Bootstrap NixOS into your base filesystem according to the configuration you edited
    + The system will detect the mount point containing your NixOS configuration file and partition and 
    + install the base filesystem into the mount point's root directory
    ```console
    nixos-install
    ```

## Documentation
### Commands
- nix-* : Handles the nix handling/systems
    - `nix-env {options} [package-name]` : Handles the Nix environment values such as Packages
        - Options
            + 
- nixos-* : Handles the Nix operating system controls such as bootstrapping, installation to other systems etc
    - `nixos-generate-config {options}` : Generate the NixOS system configuration file that defines the system.
        - Options
            + `--root [mount-point]` : Explicitly specify the root filesystem to generate the configuration file in; Usually within the root partition to bootstrap the base filesystem into
    + `nixos-install` : Begin installation and bootstrapping of the Nix base packages into the mounted root partition

## Wiki

## Resources

## References

## Remarks
