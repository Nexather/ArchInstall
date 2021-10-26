# ArchInstall
Installation guide for my arch linux project

# Creating the VM
1.	Create VM using the .iso and custom settings. I had to give it extra RAM or else I'd receive errors upon initial booting. Later I read the powerpoint and felt dumb.
2.	Select linux version 5.x 64-bit. Make sure to give it upwards of 20 gigabytes.
3.	Edit the .vmx file, and add `firmware = "efi"`
4.	Selected Arch Medium
5.	Check boot mode:
`ls /sys/firmware/efi/efivars`
Result: EFI mode
5.	`ip link`, Ping archlinux.org, data returned
6.	Use `timedatectl set-ntp true`, `timedatectl status`, `timedatectl list-timezones`, and `timedatectl set-timezone America/Chicago` to set current timezone to America/Chicago
7.	Check drives with `fdisk -l`

This is where I took my first image

# Configure the Partitions.
1.	Create GPT partition table on /dev/sda using `cfdisk /dev/sda/`, selecting GPT
  - Select free space, new, change it to 500M
  * Change the type of the partition to EFI
  * Repeat in free space, with 8GB and type Linux Swap
  * Repeat in free space, with any remaining data and type Linux filesystem
  * Write to save changes, then Quit
2.	Format the sda1 partition as FAT32 with `mkfs.fat -F32 /dev/sda1`
3.	Format the sda2 partition as swap space with `mkswap /dev/sda2` and then `swapon /dev/sda2`
4.	Format the sda3 partition as root EXT4 with `mkfs.ext4 /dev/sda3`
5.	Mount the partitions
  * `mount /dev/sda3 /mnt`
  * `mkdir /mnt/boot`
  * `mkdir /mnt/boot/efi`
  * `mount /dev/sda1 /mnt/boot/efi`

# Configure the Sytem
I took another image here
1.	Set up mirrors
  * First I fetched active US mirrors into the mirrorlist using `curl -o /etc/pacman.d/mirrorlist "https://archlinux.org/mirrorlist/?country=US&protocol=https&use_mirror_status=on`
  * This gave me all active US HTTPS mirrors
  * Then I went through in vim and uncommented the mirrors
2.	Install packages: `pacstrap /mnt base linux linux-firmware e2fsprogs dosfstools mdadm lvm2 dhcpcd iwd vi vim man-db man-pages texinfo` 
  * Not all of these are necessary, but I felt it was better to be safe than sorry. Especially since I had to revert to this step's image to install many of them after realizing I couldn't progress without vim
3.	File system table `genfstab -U /mnt >> /mnt/etc/fstab`
4.	Change root `arch-chroot /mnt`
  * This changes console context to [root@archiso /]
5. Now let's set the timezone again. 
  * `ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime` then `hwclock --systohc`
  * You can check time with hwclock
6. do `cd ~`
7. Generate localizations with `locale-gen`
8. Now, vim into the locale.gen (`vim /etc/locale.gen`) and uncomment en_US.UTF-8 UTF-8
9. Re-run `locale-gen`, it will generate US English now
10. Next, we'll create a file in /etc that contains our languages.
  * `cd /etc`, `touch locale.conf`, `vim local.conf`, and enter `LANG=en_US.UTF-8`
11. Next, create another file called vconsole.conf containing our keyboard data
  * `echo "KEYMAP=us" > vconsole.conf`

# Networking
1. Next we'll create a hostname
  * `echo "sagurka" > hostname`, or whatever hostname you want
2. Now a file containing hosts, containing
```
127.0.0.1 localhost
::1 localhost
127.0.0.1 sagurka.localdomain sagurka
```
3. Install network manager
 * `pacman -S networkmanager`
4. Enable the manager on boot
 * `systemctl enable NetworkManager`

# Root and Bootloader
1. Change root password
 * `passwd`
2. Install a bootloader
 * `pacman -S grub efibootmgr`
 * `grub-install --target=x86_64-efi --efi-directory=/boot/efi`
3. Create a config file for the bootloader
 * `grub-mkconfig -o /boot/grub/grub.cfg`
4. Reboot and sign in with root
 * `exit`
 * `reboot`

# Create a User
1. Install sudo
 * `pacman --sync sudo`
2. Give the group 'wheel' sudo access
 * wheel is often used for administration according to the wiki
3. Create users 'sagurka', 'sal', and 'codi'
 * `useradd -m -G wheel sagurka`
 * `useradd -m -G wheel sal`
 * `useradd -m -G wheel codi`
 * `passwd sagurka` 
 * `passwd sal` set to GraceHopper1906
 * `passwd codi` set to GraceHopper1906
 * `passwd -e sal` to expire on next login
 * `passwd -e codi` to expire on next login

# Desktop Environment
1. Install KDE
 * It's the most modern looking one
 * We need a few packages, so do `pacman -S xorg plasma plasma-wayland-session kde-applications`
 * Default/recommended settings
2. Configure Startup
 * Enable Display Manager and Network Manager Services on boot
 * `systemctl enable sddm.service`
 * `systemctl enable NetworkManager.service`
3. Reboot

# Customization
1. I immediately changed my theme to Sweet, and then changed my display size to 1280x600 to maintain my laptop's 16:10 ratio
2. I also installed zsh with `sudo pacman -S zsh` and used default settings with a vim keymap
3. Install git with `sudo pacman -S git`
4. Now install zsh-syntax-highlighting
 * I regret doing this, bash would probably have been easier to give colors to
 * Do this stuff
```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
echo "source ${(q-)PWD}/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
source ./zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```
 * It sometimes works?
