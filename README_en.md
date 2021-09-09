# MAW
Manjaro in Arch Way - Manjaro Linux installation guide through CLI
---
### INITIAL DOWNLOAD
Download the [Manjaro Linux image](https://manjaro.org/download/)  
Enter the live system, connect to the internet and open Konsole  
`sudo su` enter the root  
`timedatectl set-ntp true` checking the accuracy of the system clock  
`timedatectl status` checking the service status  

### DISK PARTITIONING
`fdisk -l` to identify all drives  
`fdisk *your drive*` partitioning (example - `fdisk /dev/sda`)  

Partitioning examples from [Arch Wiki](https://wiki.archlinux.org/title/Installation_guide#Example_layouts):  
>##### BIOS with MBR:
>| Mount Point | Partition 		| Partition Type | Suggested size 	   |
>|-------------|-------------------------|----------------|-------------------------|
>|   `[SWAP]`  | `/dev/swap_partition`   |   Linux swap   | More than 512 MiB       |
>|   `/mnt`    | `/dev/root_partition`   |   Linux        | Remainder of the device |
>##### UEFI with GPT
>| Mount Point | Partition 		| Partition Type | Suggested size 	   |
>|-------------|-------------------------|----------------|-------------------------|
>|`/mnt/boot` or `/mnt/efi`|`/dev/efi_system_partition`|EFI system partition |At least 260 MiB |
>|`[SWAP]`  		  | `/dev/swap_partition`     |   Linux swap        |More than 512 MiB|
>|`/mnt`    		  | `/dev/root_partition`     |Linux x86-64 root (/)| Remainder of the device|

Manjaro Linux automatic partitioning for UEFI systems:
| Mount Point | Partition 		| Partition Type | Suggested size	           |
|-------------|-------------------------|----------------|---------------------------------|
|`/mnt/boot/efi`| `/dev/efi_system_partition`|EFI system partition |300 MiB                |
|`[SWAP]`       | `/dev/swap_partition`      |Linux swap           |More than 512 MiB      |
|`/mnt`         | `/dev/root_partition`      |Linux x86-64 root (/)|Remainder of the device|

### FORMATTING PARTITIONS
`lsblk` listing all created partitions  
`mkfs.ext4 /dev/root_partition` formatting root partition with ext4  
You can also use a different file system (for example, btrfs) and set partition name, for example: `mkfs.btrfs -L "Root" /dev/root_partition`  
`mkfs.fat -F32 /dev/efi_system_partition` formatting efi partition with fat32  
`mkswap /dev/swap_partition` formatting swap partition  

### MOUNTING FILESYSTEMS
`mount /dev/root_partition /mnt` mounting root partition  
`swapon /dev/swap_partition` enabling swap partition  
`mkdir -p /mnt/boot/efi`  
`mount /dev/efi_system_partition /mnt/boot/efi` mounting efi partition  
Also, using `lsblk`, you can check the correctness of the previously performed actions  

### INSTALLATION
#### MIRRORS CHOOSING
To download from faster mirrors, use the `pacman-mirrors` utility  
An example of choosing the fastest mirrors using the https protocol:  
`pacman-mirrors --fasttrack --api --protocol https && pacman -Syyu`  
You can read more about the options for choosing mirrors on the [Manjaro Wiki](https://wiki.manjaro.org/index.php/Pacman-mirrors)  

#### INSTALLING BASIC PACKAGES
`basestrap /mnt base linux510 mhwd linux-firmware nano` installing basic packages  

#### SYSTEM CONFIGURATION
`fstabgen -U /mnt >> /mnt/etc/fstab` fstab generation with UUID identification  
`manjaro-chroot /mnt /bin/bash` changing root to a new system  
`mhwd -i pci video-nvidia` driver installation (ONLY FOR NVIDIA CARDS)  
`ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime` setting the time zone (example with Moscow)  

#### LOCALIZATION
To install localization, go to `/etc/locale.gen` and uncomment the required locales  
`nano /etc/locale.gen`  
To install English and Russian localization, you need to uncomment the following lines:  
`en_US.UTF-8 UTF-8`  
`ru_RU.UTF-8 UTF-8`  
`locale-gen` command to generate all previously uncommented locales  
Further, if you need a system language other than English, in `/etc/locale.conf` you need to replace the LANG value with one of the configured locales (example with Russian localization):  
`nano /etc/locale.conf`  
`LANG=ru_RU.UTF-8`  

#### NETWORK CONFIGURATION
Create a name for your computer (hostname):  
`nano /etc/hostname`  
`myhostname`  
Edit `/etc/hosts` with the following [pattern](https://wiki.archlinux.org/title/Installation_guide#Network_configuration):  
>127.0.0.1  localhost  
>::1        localhost  
>127.0.1.1  myhostname.localdomain  myhostname  

#### SUPERUSER PASSWORD
`passwd` setting superuser password  

#### BOOTLOADER
The following is an example of installing a bootloader for UEFI systems, to install a bootloader for BIOS systems, check the details on the [Arch Wiki](https://wiki.archlinux.org/title/GRUB)  
`pacman -S grub efibootmgr` grub installation  

You can also optionally install microcode for intel or amd processors:  
`pacman -S amd-ucode`  microcode for amd processors  
`pacman -S intel-ucode`  microcode for intel processors  

Installing a bootloader for uefi systems with an efi partition on /boot/efi, bootloader-id you can put any - this is the name of your system in the choice of boot options in the BIOS  
`grub-install --target=x86_64-efi --efi-directory=/boot/efi  --bootloader-id=manjaro`  
`update-grub` config file generation  

### SETTING THE GRAPHIC ENVIRONMENT
#### CREATING AN ADMINISTRATOR
`useradd -m username` create user username  
`usermod -a -G wheel username` adding username to wheel group  
`passwd username` setting password for username  
`pacman -S sudo` installing sudo and nano editor (if it was not there before)  
`EDITOR=nano visudo` editing sudo options  
Next, you need to uncomment the line associated with the wheel group:  
`%wheel ALL=(ALL) ALL`  

#### KDE PLASMA INSTALLATION
`pacman -S xorg-server plasma-desktop sddm-kcm` installing xorg-server and minimal suite of plasma packages  
  
For a complete set of KDE Plasma, as well as its applications, on the Manjaro KDE packages pages page (link in TIPS & TRICKS) you need to install the packages from Plasma5 and KDE applications sections  
If you are not worried about the "clutter" of your system, then you can run the following command, which will install all the necessary packages for KDE Plasma and KDE Applications:  
`pacman -S xorg-server plasma-meta kde-applications`  
  
`systemctl enable sddm.service` enable sddm  
`systemctl enable NetworkManager.service` enable NetworkManager  

#### GNOME INSTALLATION
`pacman -S xorg-server gnome` installing xorg-server and gnome  
`systemctl enable gdm.service` enable gdm  
`systemctl enable NetworkManager.service` enable NetworkManager  

### FINISHING INSTALLATION
`exit` exit chroot  
`shutdown now` system shutdown 

### TIPS & TRICKS
If you want to get the same set of installed programs as in regular Manjaro Linux images, then without leaving the chroot, you can go through the list of installed packages in [KDE](https://gitlab.manjaro.org/profiles-and-settings/iso-profiles/-/blob/master/manjaro/kde/Packages-Desktop), [GNOME](https://gitlab.manjaro.org/profiles-and-settings/iso-profiles/-/blob/master/manjaro/gnome/Packages-Desktop) and [XFCE](https://gitlab.manjaro.org/profiles-and-settings/iso-profiles/-/blob/master/manjaro/xfce/Packages-Desktop)  

Also on KDE, problems may arise when changing user options in settings  
If it gives an error while saving changes, then in `/etc/login.defs` you need to change the `CHFN_RESTRICT` field, changing the value to `frwh`  

If btrfs was selected as the file system during the markup, then, if necessary, you will have to create the subvolume yourself, following [this article](https://wiki.archlinux.org/title/Btrfs#Subvolumes)  

Various installation details and more can be found at [ArchWiki](https://wiki.archlinux.org/)  
  
---
Created by @eightbyte81
