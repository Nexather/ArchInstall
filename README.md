# ArchInstall
Installation guide for my arch linux project

1.	Create VM using the .iso, linux version 5.x 64-bit.
2.	Selected Arch Medium
3.	Check boot mode:
# ls /sys/firmware/efi/efivars
Result: BIOS mode
4.	Ping archlinux.org, data returned
5.	Use timedatectl set-ntp true, status, list-timezones, and set-timezone to set current timezone to America/Chicago
6.	Create GPT partition table on /dev/sda using fdisk
7.	Create partition with default start and end
8.	Write and exit fdisk
9.	Format the partition as FAT32 with 
# mkfs.fat -F32 /dev/sda1
10.	Mount root to /mnt
11.	Set up mirrors
12.	Install essential packages 
# pacstrap /mnt base linux linux-firmware
