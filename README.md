# Debian-chroot-install-systemdboot-
Memos etc on chroot Debian Bullseye install using Systemd 

MAKE CONSOLE READABLE
# dpkg-reconfigre console-setup

PARTITION DISK
# parted -a optimal /dev/nvme0n1
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
         debootstrap --include=bluez,bluez-tools,network-manager,wget,gpg,gpg2,firmware-linux-free,firmware-linux-nonfree,locales,sudo,git,console,setup,i965-va-driver,thermald,fonts-ipafont,fonts-vlgothic --arch amd64 bullseye /mnt http://ftp.jp.debian.org
         wget https://github.com/glacion/genfstab/releases/download/1.0/genfstab; chmod +x genfstab
         ./genfstab -U /mnt >> /mnt/etc/fstab
         echo xxx >> /mnt/etc/apt/sources.list
         echo
         echo kernels >> /mnt/etc/apt/sources.list
         
 CHROOT
 
         chroot /mnt /bin/bash
         gpgkey
         apt update
         
