# Installing Arch linux with EFI

1. Verify boot mode:
    - `ls /sys/firmware/efi/efivars` (If the directory exist your computer supports EFI)

2. Ping some site on the Internet to verify connection:
    - `ping archlinux.org -c 5`

3. Update system clock:
    - `timedatectl set-ntp true`
    - You can verify the status with `timedatectl status`

4. Enable SSH (optional):
    - `systemctl start sshd`

5. Change root password:
    - `passwd`

6. Go to [https://archlinux.org/mirrorlist](https://archlinux.org/mirrorlist) and find the closest mirror that supports HTTPS:
    - Add the mirrors on top of the `/etc/pacman.d/mirrorlist` file.
    - `Server = https://mirror.arizona.edu/archlinux/$repo/os/$arch` (United States)

7. cfdisk

8. Create EFI partition:
    - New
    - 300M
    - Type
    - EFI System
    - Write

9. Create `/root` partition:
    - Select free space
    - New
    - 30G
    - Linux filesystem
    - Write

10. Create `/home` partition:
    - Select free space
    - New
    - Use rest of the space
    - Linux filesystem
    - Write

11. Create the filesystems:
    - `fdisk -l` to view the partitions for the next step
    - `mkfs.fat -F32 /dev/sda1`
    - `mkfs.ext4 /dev/sda2`
    - `mkfs.ext4 /dev/sda3`
    - 

12. Create the `/root` and `/home` directories:
    - `mount /dev/sda2 /mnt`
    - `mkdir /mnt/home`
    - `mount /dev/sda3 /mnt/home`

13. Install Arch linux base packages:
    - `pacstrap -i /mnt base linux linux-firmware sudo nano vim`

14. Generate the `/etc/fstab` file:
    - `genfstab -U -p /mnt >> /mnt/etc/fstab`

15. Chroot into installed system:
    - `arch-chroot /mnt`

16. Find available timezones:
    - `timedatectl list-timezones`
    
18. Set the timezone:
    - `ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`
    - or `timedatectl set-timezone America/Chicago`

17. Update the Hardware clock:
    - `hwclock --systohc`

## Set hostname

`echo ArchPC > /etc/hostname`

## Enable Network

`pacman -S networkmanager`

`systemctl enable NetworkManager`

18. Install boot manager and other needed packages:
    - `pacman -S grub efibootmgr dosfstools openssh os-prober mtools linux-headers linux-lts linux-lts-headers`

19. Set locale:
    - `sed -i 's/#en_US.UTF-8/en_US.UTF-8/g' /etc/locale.gen` (uncomment en_US.UTF-8)
    - `locale-gen`

20. Enable root login via SSH:
    - `sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config`
    - `systemctl enable sshd.service`
    - `passwd` (for changing the root password)

21. Create EFI boot directory:
    - `mkdir /boot/EFI`
    - `mount /dev/sda1 /boot/EFI`

22. Install GRUB on EFI mode:
    - `grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck`

23. Setup locale for GRUB:
    - `cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo`

24. Write GRUB config:
    - `grub-mkconfig -o /boot/grub/grub.cfg`

25. Create swap file:
    - `fallocate -l 2G /swapfile`
    - `chmod 600 /swapfile`
    - `mkswap /swapfile`
    - `echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab`

26. Exit, unount and reboot:
    - `exit`
    - `umount -a`
    - `reboot`
