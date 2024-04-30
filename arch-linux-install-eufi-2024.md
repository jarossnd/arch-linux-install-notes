# Installing Arch Linux

> Note: This article assumes you are using a US keyboard layout. You should also read the official [Arch installation guide](https://wiki.archlinux.org/index.php/installation_guide).

1. Set the keyboard
    - If you are not using a US keyboard then you will need to set the appropriate keymap
    - You do not need to complete the below steps if you are already using a standard US keyboard
    - `ls /usr/share/kbd/keymaps/**/*.map.gz` (Find the keymap you want to use)
    - `loadkeys de-latin1`

2. Verify boot mode:
    - If you get a results after running the below command then your computer is configured to use UEFI and you should review my other instructions on how to install Arch Linux on a UEFI system.
    - `ls /sys/firmware/efi/efivars` (If you receive an error then you machine is booted in BIOS and we can continue. If you get output then follow the UEFI directions in this repo.)

3. Ping some site on the Internet to verify connection:
    - Use `ip link` to verify your network devices
    - If necessary, use `iwctl` to setup your wireless
    - Run `ping` to check to see if you already have an internet connection
    - `ping archlinux.org -c 5`

4. Update system clock:
    - View the current status of your clock by running `timedatectl status`
    - Run the following command to ensure your system clock is correct `timedatectl set-ntp true`
    - This will set your date and time to update remotely using network time protocal

5. Setting up partitions with `fdisk`

**EFI Partition**
- use `fdisk -l` to list your storage devices
- Once you have identified the disk you want to use do `fdisk /dev/nvme1n1`
- Note: Replace /dev/nvme1n1 with the disk you want to use
- Press `n` to create a new partition. We will create this partition as our EFI partition
- Press `p` for primary
- Use partition number `1` which should be the default option
- Use the default for the first sector
- Set the last sector. I am going to use `+4G`
- Type `t` to set the type and choose `ef` for EFI (FAT-12/16/32)

**Swap Partition**
- use `fdisk -l` to list your storage devices
- Once you have identified the disk you want to use do `fdisk /dev/nvme1n1`
- Note: Replace /dev/nvme1n1 with the disk you want to use
- Press `n` to create a new partition. We will create this partition as our Swap partition
- Press `e` for extended
- Use partition number `2` which should be the default option
- Use the default for the first sector
- Set the last sector. If you want your swap file to be 8GB then type `+8G`
- Type `t` to set the type and choose `82` for Linux swap

**root Partition**
- Now we will create our root partition so type `n` to create a new partition
- Partition type for our root partition `p` will be primary
- Set the partition number to equal `3`
- Use the default for the first sector
- Use the default for the last sector to use all remaining space
- Press `t` to set the type then choose `83`
- Last, press `w` to write your partitions

6. Create the filesystems
    - `fdisk -l` to view the partitions for the next step
    - `mkfs.fat -F32 /dev/nvme1n1p1`
    - `mkswap /dev/nvme1n1p2`
    - `swapon /dev/nvme1n1p2`
    - `mkfs.ext4 /dev/nvme1n1p3`
    - `mount /dev/nvme1n1p3 /mnt`

7. Mount the drive that contains the EFI partition (On the Windows HDD)
    - Run fdisk to find out what drive contains the existing EFI partition as we will need to mount it
    - `fdisk -l`
    - `mkdir /mnt/efi`
    - `mount /dev/nvme0n1p1 /mnt/efi`

8. Install Arch linux base packages:
    - Use the following if you want to include VIM:
    - `pacstrap -i /mnt base linux linux-firmware sudo vim`
    - Use the following if you want to also include Nano:
    - `pacstrap -i /mnt base linux linux-firmware sudo nano vim`

9. Generate the `/etc/fstab` file:
    - `genfstab -U /mnt >> /mnt/etc/fstab`

10. Chroot into installed system:
    - `arch-chroot /mnt`

11. Set the timezone:
    - `ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`
    - If you are unsure, do a `ls /usr/share/zoneinfo` to find country
    - Once you have found the country do another `/usr/share/zoneinfo/America` to find the city

12. Update the Hardware clock:
    - `hwclock --systohc`

13. Set locale:
    - `vim /etc/locale.gen` (uncomment en_US.UTF-8)
    - `locale-gen`
    - `vim /etc/locale.conf`
    - `LANG=en_US.UTF-8`

14. Set your hostname and configure hosts
    - `echo myhostname > /etc/hostname`
    - Update `/etc/hosts` with the following:

```
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

15. Set the root password
    - `passwd`

16. Install boot manager and os-prober:
    - `pacman -S grub efibootmgr os-prober`

17. Create EFI boot directory:
    - `mkdir /boot/EFI`
    - `mount /dev/nvme0n1p1 /boot/EFI` (replace /dev/nvme0n1p1 with the drive that contains the Windows EFI partition)

18. Install GRUB on EFI mode:
    - `grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB`

19. Enable os-prober
    - `vim /etc/default/grub`
    - Uncomment `GRUB_DISABLE_OS_PROBER=false`

20. Write GRUB config:
    - `grub-mkconfig -o /boot/grub/grub.cfg`

21. Let's enable the network
    - If you don't do this before your reboot you won't have internet connection
    - `pacman -S networkmanager`
    - `systemctl enable NetworkManager`

22. Exit, unount and reboot:
    - `exit`
    - `umount -R /mnt`
    - `reboot`

23. After a reoobt I did not see Windows in the Grub boot loader menu so I ran the following:

`sudo grub-mkconfig -o /boot/grub/grub.cfg`

# Create User Account

1. Create user account and add it to appropriate groups

    - `useradd -m username`
    - `passwd username`
    - `usermod -aG wheel,video,audio,storage username`

2. Add account to sudoers file

    - `vim /etc/sudoers`
    - Uncomment the below line to allow members of group wheel to execute any command
    - `# %wheel ALL=(ALL) ALL` to `%wheel ALL=(ALL) ALL`

