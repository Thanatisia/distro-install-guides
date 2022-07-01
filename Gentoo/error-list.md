## Linux Distro Common Installation Error List


## Errors
- sys-kernel/linux-firmware License install issue
    + Add 'sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE' to /etc/portage/package.license
        ```console
        # Error : All Ebuilds That Could Satisfy "sys-kernel/linux-firmware" Have Been Masked
        # + https://www.google.com/search?q=Gentoo+all+ebuilds+that+could+satisfyt+linux-firmware+has+been+masked&rlz=1C1CHBD_enSG945SG945&oq=Gentoo+all+ebuilds+that+could+satisfyt+linux-firmware+has+been+masked&aqs=chrome..69i57.14392j0j1&sourceid=chrome&ie=UTF-8
        # + https://www.reddit.com/r/Gentoo/comments/lxsovz/all_ebuilds_that_could_satisfy/
        echo "sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE" | tee -a /etc/portage/package.license
        ```

- UEFI grub-install Read-only filesystem error
    - If grub_install returns an error like 'Could not prepare Boot variable: Read-only file system'
        + Remount the efivars special mount as Read-Write
            ```console
            # Syntax : mount -o [[action](action)],[permission] [directory-to-mount]
            mount -o remount,rw /sys/firmware/efi/efivars
            ```
