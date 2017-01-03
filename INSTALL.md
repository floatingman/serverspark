The following is a brief installation tutorial for [Arch Linux][1]. It assumes
familiarity with the Arch [Beginner's Guide][2] and [Installation Guide][3].

Two methods are presented here, the more traditional "BIOS mode", where
there is no separate `/boot` partition. The second method is "UEFI mode" which
will use a GPT.

Use your system's setup interface to choose UEFI or legacy/BIOS mode as appropriate.

On some newer systems (e.g. Dell XPS 15), set SATA operation mode to AHCI.

Boot into the Arch installer.

If your console font is tiny ([HiDPI][7] systems), set a new font.

    $ setfont sun12x22

Connect to the Internet.

Verify that the [system clock is up to date][8].

    $ timedatectl set-ntp true
    
(BIOS mode) Create a single partition.

    $ parted -s /dev/sda mklabel msdos
    $ parted -s /dev/sda mkpart primary 2048s 100%

(UEFI mode) Create partitions for EFI, boot, and root.

    $ parted -s /dev/sda mklabel gpt
    $ parted -s /dev/sda mkpart primary fat32 1MiB 513MiB
    $ parted -s /dev/sda set 1 boot on
    $ parted -s /dev/sda set 1 esp on
    $ parted -s /dev/sda mkpart primary 513MiB 1024MiB
    $ parted -s /dev/sda mkpart primary 1024MiB 100%
    $ mkfs.vfat -F32 /dev/sda1

Create and mount the root filesystem. Note that for UEFI systems
this will be partition 3.

    $ pvcreate /dev/sda
    $ vgcreate arch /dev/sda
    $ lvcreate -L 8G arch -n swap
    $ lvcreate -L 30G arch -n root
    $ lvcreate -l +100%FREE arch -n home
    $ lvdisplay
    $ mkswap -L swap /dev/mapper/arch-swap
    $ mkfs.ext4 /dev/mapper/arch-root
    $ mkfs.ext4 /dev/mapper/arch-home
    $ mount /dev/mapper/arch-root /mnt
    $ mkdir /mnt/home
    $ mount /dev/mapper/arch-home /mnt/home
    $ swapon /dev/mapper/arch-swap

(UEFI mode) Mount the boot and EFI partitions.

    $ mkfs.ext4 /dev/sda2
    $ mkdir /mnt/boot
    $ mount /dev/sda2 /mnt/boot
    $ mkdir /mnt/boot/efi
    $ mount /dev/sda1 /mnt/boot/efi

Optionally [edit the mirror list][9].

    $ grep -A1 --no-group-separator "United States" /etc/pacman.d/mirrorlist > /etc/pacman.d/mirrorlist.backup
    $ rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist

May need to run thisafter updating mirrorlist

    $ pacman-key --refresh-keys

Install the [base system][10].

    $ pacstrap -i /mnt base base-devel net-tools git grub ansible
    (UEFI mode) $ pacstrap /mnt efibootmgr

Generate and verify [fstab][11].

    $ genfstab -U -p /mnt >> /mnt/etc/fstab
    $ less /mnt/etc/fstab

Change root into the base install and perform [base configuration tasks][12].

    $ arch-chroot /mnt /bin/bash
    $ echo en_US.UTF-8 UTF-8 >> /etc/locale.gen
    $ locale-gen
    $ echo LANG=en_US.UTF-8 > /etc/locale.conf
    $ export LANG=en_US.UTF-8
    $ ln -s /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
    $ hwclock --systohc --utc
    $ echo mymachine > /etc/hostname
    $ systemctl enable dhcpcd.service
    $ passwd

Set your mkinitcpio lvm2 hooks and rebuild.

    $ sed -i 's/^HOOKS=.*/HOOKS="base udev autodetect modconf block keyboard lvm2 resume filesystems fsck"/' /etc/mkinitcpio.conf
    $ mkinitcpio -p linux

Configure GRUB.

    # BIOS mode
    $ sed -i 's/^GRUB_CMDLINE_LINUX=.*/GRUB_CMDLINE_LINUX="resume=\/dev\/mapper\/arch-swap"/' /etc/default/grub
    $ grub-install /dev/sda
    $ chmod -R g-rwx,o-rwx /boot

    # UEFI mode
    $ sed -i 's/^GRUB_CMDLINE_LINUX=.*/GRUB_CMDLINE_LINUX="root=\/dev\/mapper\/arch-root resume=\/dev\/mapper\/arch-swap"/' /etc/default/grub
    $ grub-mkconfig -o /boot/grub/grub.cfg
    $ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --recheck
    $ chmod -R g-rwx,o-rwx /boot

Cleanup and reboot!

    $ exit
    $ umount -R /mnt
    $ reboot

Run ansible!


[1]: https://www.archlinux.org/
[2]: https://wiki.archlinux.org/index.php/Beginners'_guide
[3]: https://wiki.archlinux.org/index.php/Installation_guide
[4]: https://wiki.archlinux.org/index.php/Encrypted_LVM#LVM_on_LUKS
[5]: http://www.pavelkogan.com/2014/05/23/luks-full-disk-encryption/
[6]: https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#Encrypted_boot_partition_.28GRUB.29
[7]: https://wiki.archlinux.org/index.php/HiDPI
[8]: https://wiki.archlinux.org/index.php/Beginners'_guide#Update_the_system_clock
[9]: https://wiki.archlinux.org/index.php/Beginners'_Guide#Select_a_mirror
[10]: https://wiki.archlinux.org/index.php/Beginners'_Guide#Install_the_base_system
[11]: https://wiki.archlinux.org/index.php/Beginners'_guide#Generate_an_fstab
[12]: https://wiki.archlinux.org/index.php/Beginners'_guide#Configure_the_base_system
