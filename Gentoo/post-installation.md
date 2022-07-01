# Linux Distro Post-Installation Guide

## Table of Contents
- [List](#list)
    - [Enable sudo](#enable-sudo)

## List

### Enable Sudo

- Manual
    + Open /etc/sudoers
    + Uncomment '%wheel (ALL:ALL) ALL'
    + save
    
- Use sed to uncommment '%wheel (ALL:ALL) ALL' in /etc/sudoers
    ```console 
    sed -i 's/^#\s*\(%wheel\s\+ALL=(ALL:ALL)\s\+ALL\)/\1/' /etc/sudoers
    ```
    

