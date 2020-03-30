# Debian-chroot-install-systemdboot-
Memos etc on chroot Debian Bullseye install using Systemd 

MAKE CONSOLE READABLE
# dpkg-reconfigre console-setup

PARTITION DISK

parted -a optimal /dev/nvme0n1
          mklabel gpt
          unit mib
          mkpart esp fat32
          1 512
          mkpart 512 12512
          mkpart 12512 -1
          set 1 boot on
          quit
          
FORMAT DISKS

         mkfs.vfat -F32 /dev/nvme0n1p1
         mkswap /dev/nvme0n1p2
         swapon /dev/nvme0n1p2
         mkfs.ext4 /dev/nvme0n1p3
 
INSTALL BASE SYSTEM

         apt update
         apt install -y debootstrap
         mount /dev/nvme0n1p3 /mnt
         debootstrap --include=intel-microcode,bluez,bluez-tools,network-manager,wget,gpg,gpg2,firmware-linux-free,firmware-linux-nonfree,locales,sudo,git,console,setup,i965-va-driver,thermald,fonts-ipafont,fonts-vlgothic --arch amd64 bullseye /mnt http://ftp.jp.debian.org
         mount /dev/nvme0n1p1 /mnt/boot
         mount --bind /dev /mnt/dev
         mount -t sysfs sysfs /mnt/sys
         mount --bind /sys/firmware/efi/efivars /mnt//sys/firmware/efi/efivars
         mount -t proc proc /mnt/proc
         wget https://github.com/glacion/genfstab/releases/download/1.0/genfstab; chmod +x genfstab
         ./genfstab -U /mnt >> /mnt/etc/fstab
         echo deb http://ftp.jp.debian.org/debian bullseye main contrib non-free > /mnt/etc/apt/sources.list
         echo deb-src http://ftp.jp.debian.org/debian/ bullseye main contrib non-free >> /mnt/etc/apt/sources.list
         echo deb http://ftp.jp.debian.org/debian/ bullseye-updates main contrib non-free >> /mnt/etc/apt/sources.list
         echo deb-src http://ftp.jp.debian.org/debian/ bullseye-updates main contrib non-free >> /mnt/etc/apt/sources.list
         echo deb http://security.debian.org bullseye-security main contrib non-free >> /mnt/etc/apt/sources.list
         echo deb-src http://security.debian.org bullseye-security main contrib non-free  >> /mnt/etc/apt/sources.list
         echo deb [arch=amd64] https://pkg.surfacelinux.com/debian release main >> /mnt/etc/apt/sources.list
         echo kernels >> /mnt/etc/apt/sources.list
         
 CHROOT
 
         
         export LANGUAGE=en_GB.UTF-8
         export LANG=en_GB.UTF-8
         export LC_ALL=en_GB.UTF-8
         dpkg-reconfigure tzdata
         chroot /mnt /bin/bash
         wget -qO - https://raw.githubusercontent.com/linux-surface/linux-surface/master/pkg/keys/surface.asc \
    | sudo apt-key add -
         apt update
         apt install linux-headers-surface-lts linux-image-surface-lts linux-libc-dev-surface surface-ipts-firmware linux-surface-secureboot-mok libwacom-surface
         update-initramfs -c -k 4.19.112-surface-lts
         sudo echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/nvme0n1p3) rw" >> /boot/efi/loader/entries/debian.conf
