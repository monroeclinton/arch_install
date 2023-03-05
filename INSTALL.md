# Arch Install

- Encrypted
- Works with UEFI

## Install Guide

1.  Create bootable USB with Arch
    ```
    dd if=ARCH.iso of=/dev/sdX status=progress
    ```
2.  Connect to Internet

    If you are using wifi, use [iwd](https://wiki.archlinux.org/title/Iwd)
    ```
    device list
    station *device* get-networks
    station *device* connect *SSID*
    ```
3.  Partition the disk
    ```
    lsblk # View a list of your block devices
    ```
    
    NOTE: Virtual machines in this tutorial, use `vdX` instead of `sdX`. Replace `X` with the proper letter you get from `lsblk`

    ```
    fdisk /dev/sdX
    ```
    Enter:
    ```
    (command) o # Create an empty partition table

    (command) n # Create boot
    (command) p
    (command) 1
    
    # Press enter for first sector
    (command) +256M # Offset for last sector

    (command) n # Contains everything else
    (command) p
    (command) 2 # Press enter after this till complete

    (command) w # Write
    ```
4.  Format boot:
    ```
    mkfs.fat -F32 /dev/sdX1
    ```
5.  Create luks:
    ```
    cryptsetup -c aes-xts-plain64 -y --use-random luksFormat /dev/sdX2
    cryptsetup luksOpen /dev/sdX2 luks
    ```
6.  Create LVM ([Logical Volume Management](https://wiki.archlinux.org/title/LVM)):
    ```
    pvcreate /dev/mapper/luks
    vgcreate vg0 /dev/mapper/luks
    lvcreate --size 8G vg0 --name swap
    lvcreate --size 80G vg0 --name root
    lvcreate -l +100%FREE vg0 --name home
    mkfs.ext4 /dev/mapper/vg0-root
    mkfs.ext4 /dev/mapper/vg0-home
    mkswap /dev/mapper/vg0-swap
    ```
7.  Mount:
    ```
    mount /dev/mapper/vg0-root /mnt
    mkdir /mnt/home
    mount /dev/mapper/vg0-home /mnt/home
    mkdir /mnt/boot
    mount /dev/sdX1 /mnt/boot
    swapon /dev/mapper/vg0-swap
    ```
8.  Install:
    ```
    # Add iwd if you need wifi
    pacstrap -i /mnt base base-devel linux linux-firmware git vim lvm2 grub efibootmgr
    ```
9.  Create fstab:
    ```
    genfstab -pU /mnt >> /mnt/etc/fstab
    ```
10. Enter chroot jail of new system:
    ```
    arch-chroot /mnt
    ```
11. Set hostname:
    ```
    echo machine > /etc/hostname
    ```
12. Map localhost in `/etc/hosts`:
    ```
    127.0.0.1	localhost
    ::1		localhost
    ```
13. Create user:
    ```
    useradd -m -g users -G wheel user
    passwd user
    visudo # Uncomment %wheel ALL=(ALL) ALL
    passwd # Set root password
    ```
14. Set locale
    ```
    vim /etc/locale.gen # uncomment en_US.UTF-8 UTF-8
    locale-gen
    echo LANG=en_US.UTF-8 > /etc/locale.conf
    export LANG=en_US.UTF-8
    ```
15. Set the system time
    ```
    ln -sf /usr/share/zoneinfo/Zone/SubZone /etc/localtime
    ```
16. Set hardware clock from system clock
    ```
    hwclock --systohc
    ```
17. Modify `mkinitcpio`:
    ```
    vim /etc/mkinitcpio.conf
    ```
    Add `ext4` to `MODULES=()`

    Add `encrypt lvm2` to `HOOKS=()` before `filesystems`

    Create initial ramdisk:
    ```
    mkinitcpio -p linux
    ```
18. Modify GRUB config in `/etc/default/grub`
    ```
    GRUB_CMDLINE_LINUX="cryptdevice=/dev/sdX2:luks:allow-discards root=/dev/mapper/vg0-root"
    ```
19. Install GRUB for UEFI: [Guide](https://wiki.archlinux.org/title/GRUB)

    Install GRUB for legacy
    ```
    grub-install /dev/sdX
    ```
20. Make GRUB config:
    ```
    grub-mkconfig -o /boot/grub/grub.cfg
    ```
21. Exit install
    ```
    exit
    umount -R /mnt
    swapoff -a
    reboot # Unplug USB
    ```
