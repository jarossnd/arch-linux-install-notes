# Installing Arch Linux with UEFI

## Overview

The steps I tool to triple boot Windows, Windows, and Arch Linux. Each OS has it's own hard drive. I also installed both Windows OS's first prior to installing Arch Linux.

> Note: This article assumes you are using a US keyboard layout. You should also read the official Arch installation guide https://wiki.archlinux.org/index.php/installation_guide. 

1. Set they keyboard
    - If you are not using a US Keyboard then you will need to set the appropriate keymap
    - You do not need to complete the below steps if you are already using a standard US keyboard
    - `ls /usr/share/kbd/keymaps/**/*.map.gz` (Find the keymap you want to use)
    - `loadkeys de-latin1`

2. Verify boot mode:
    - This is an important set when working with a UEFI system. Run the below command and if the directory does not exist, the system may not be booted in UEFI mode.
    - `ls /sys/firmware/efi/efivars` (If the directory exist your computer supports EFI)

3. Ping some site on the Internet to verify connection:
    - Use `ip link` to verify your network devices
    - If necessary, use `iwctl` to setup your wireless
    - Run `ping` to check to see if you already have an internet connection
    - `ping archlinux.org -c 5`

4. Update system clock:
    - View the current status of your clock by running `timedatectl status`
    - Run the following command to ensure your system clock is correct `timedatectl set-ntp true`
    - This will set your date and time to update remotely using network time protocal

5. Go to [https://archlinux.org/mirrorlist](https://archlinux.org/mirrorlist) and find the closest mirror that supports HTTPS:
    - Add the mirrors on top of the `/etc/pacman.d/mirrorlist` file.
    - `Server = https://mirror.arizona.edu/archlinux/$repo/os/$arch` (United States)

6. cfdisk /dev/<HHD>
    - Select `gpt` if asked for a label type

7. Create our swap partition:
    - Select free space
    - New
    - 4GB (this number will depend on your requirements for swap)
    - Type
    - Linux swap
    - Write

8. Create our linux partition:
    - Select free space
    - New
    - Use rest of the space
    - Type
    - Linux filesystem
    - Write

9. Create the filesystems and mount (replace sda* with the name of your HHD):
    - `fdisk -l` to view the partitions for the next step
    - `mkswap /dev/sda1`
    - `swapon /dev/sda1`
    - `mkfs.ext4 /dev/sda2`
    - `mount /dev/sda2 /mnt`

10. Mount the drive that contains the EFI partition (On the Windows HDD)
    - Run fdisk to find out what drive contains the existing EFI partition as we will need to mount it
    - `fdisk -l`
    - `mkdir /mnt/efi`
    - `mount /dev/sda1 /mnt/efi`

10. Install Arch linux base packages:
    - Use the following if you want to include VIM:
    - `pacstrap -i /mnt base linux linux-firmware sudo vim`
    - Use the following if you want to also include Nano:
    - `pacstrap -i /mnt base linux linux-firmware sudo nano vim`

11. Generate the `/etc/fstab` file:
    - `genfstab -U /mnt >> /mnt/etc/fstab`

12. Chroot into installed system:
    - `arch-chroot /mnt`

13. Find available timezones:
    - `timedatectl list-timezones`
    
14. Set the timezone:
    - `timedatectl set-timezone America/Chicago`

15. Update the Hardware clock:
    - `hwclock --systohc`

16. Set your hostname and configure hosts
    - `echo myhostname > /etc/hostname`
    - Update `/etc/hosts` with the following:

```
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

17. Set the root password
    - `passwd`

18. Install boot manager and other needed packages:
    - `pacman -S grub efibootmgr`

19. Install os-prober
    - `pacman -S os-prober`

20. Set locale:
    - `sed -i 's/#en_US.UTF-8/en_US.UTF-8/g' /etc/locale.gen` (uncomment en_US.UTF-8)
    - `locale-gen`
    - `vim /etc/locale.conf`
    - `LANG=en_US.UTF-8`

21. Create EFI boot directory:
    - `mkdir /boot/EFI`
    - `mount /dev/sda1 /boot/EFI` (replace /dev/sda1 with the drive that contains the Windows EFI partition)

22. Install GRUB on EFI mode:
    - `grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB`

23. Enable os-prober
    - `vim /etc/default/grub`
    - Uncomment `GRUB_DISABLE_OS_PROBER=false`

24. Write GRUB config:
    - `grub-mkconfig -o /boot/grub/grub.cfg`

25. Let's enable the network
    - If you don't do this before your reboot you won't have internet connection
    - `pacman -S networkmanager`
    - `systemctl enable NetworkManager`

26. Exit, unount and reboot:
    - `exit`
    - `umount -R /mnt`
    - `reboot`

# Create User Account

1. Create user account and add it to appropriate groups

    - `useradd -m username`
    - `passwd username`
    - `usermod -aG wheel,video,audio,storage username`

2. Add account to sudoers file

    - `vim /etc/sudoers`
    - Uncomment the below line to allow members of group wheel to execute any command
    - `# %wheel ALL=(ALL) ALL` to `%wheel ALL=(ALL) ALL`

