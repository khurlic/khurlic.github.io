---
layout: post
title: "Debian install on dell 9550"
date: 2015-12-22 18:05:59 -0800
comments: true
categories: Linux Dell
---

Im old to linux just new to debian. Im a long time RedHat guy. I wanted to see what all the hype was about. So I decided to kick the tires on Debian. Here are the steps I've took to get things going for me. 

Things to know

1. This is not a dual boot how to.
2. The windows10 install will be completely erased.
3. Things are still not working 100% as of this writing.
4. There are no screen shots :(


## Hardware
--------------------------------------------------------------------------------
Dell XPS 15 9550

   * 6th Generation Intel Core i7 (Skylake)
   * RAM: 16GB ddr4
   * Disk: Samsung PCIe 512GB SSD 
   * Wifi: Broadcom PCIe Chipset BCM43602 (14e4:43ba) / DW1830 3x3 802.11ax 2.4/5GHz + Bluetooth 4.1
   * Display: 15.6" 4k Ultra HD InfinityEdge Touch Display
   * Battery: 84w
   * Ports: 2 usb 3.0, 1 Thunderbolt 3, SD card reader and 1 hdmi 

Apple usb to ethernet dongle


## Install Media 
--------------------------------------------------------------------------------
Im using a MacBookPro (MacBookPro11,5) running OSX 10.10.5 (Yosemite) to create my usb bootable install media. I used the following steps to create my bootable usb thumbdrive.


* ThumbDrive: SanDisk Extreme USB 3.0 32GB thumbdrive
* Install ISO: [debian-8.2.0-amd64-DVD-1.iso](http://cdimage.debian.org/debian-cd/8.2.0/amd64/iso-dvd/debian-8.2.0-amd64-DVD-1.iso)
* Install ISO SHA256SUM: [d70685dec63ad07499e046a77237dabd8949f0ba0fa047c358e7d66a691576cd](http://cdimage.debian.org/debian-cd/8.2.0/amd64/iso-dvd/SHA256SUMS)



Now lets start the fun

1. ** Verify the sha hash of your debian Jessie 8.2 dvd iso **

        $ shasum -a 256 debian-8.2.0-amd64-DVD-1.iso
        d70685dec63ad07499e046a77237dabd8949f0ba0fa047c358e7d66a691576cd  debian-8.2.0-amd64-DVD-1.iso

2. ** Insert your usb thumbdrive  **
3. ** Find your usb thumbdrive **

        $ diskutil list

        /dev/disk2
           #:                       TYPE NAME                    SIZE       IDENTIFIER
           0:     Apple_partition_scheme                        *31.4 GB    disk2
           1:        Apple_partition_map                         4.1 KB     disk2s1
           2:                  Apple_HFS                         426.0 KB   disk2s2

    As you can see my thumbdrive is seen as /dev/disk2. I know this because my thumbdrive is 32GB
Your result can be different depending on how many disk/devices you have attached to your Mac. Please make sure you have the correct disk before moving forward. If you use the wrong disk you will lose data.

4. ** Unmount your thumbdrive **

        diskutil umountdisk /dev/disk2

5. ** Write the Debian Jessie 8.2 DVD install image to the thumbdrive. ** _!!!Warning this will overwrite any data you have on your thumbdrive!!!_

        sudo dd if=~/Downloads/debian-8.2.0-amd64-DVD-1.iso of=/dev/rdisk2 bs=1m

    Why /dev/rdisk instead of /dev/disk [answer here](http://superuser.com/questions/631592/why-is-dev-rdisk-about-20-times-faster-than-dev-disk-in-mac-os-x)

6. ** unmount your usb install media ***

        $ diskutil umountDisk /dev/disk2

7. ** remove your thumbdrive from your system **


## BIOS setup on the Dell XPS 15 9550
--------------------------------------------------------------------------------
1. Power on your Dell xps 15
2. Rapidly press F2 (This is so you dont miss entering the BIOS setup screen)
3. Disable secure boot
   * Under Settings => Secure Boot => Secure Boot Enable
      * Check Disabled then click Apply


## Start the Debian install
--------------------------------------------------------------------------------
1. Insert your Debian usb install media that your created earlier
2. Power on your Dell XPS 15
3. Rapidly press F12
4. Select your USB install media
5. Run through the Debian setup. ** Select text install ** <- trust me you want it.


## Issues
--------------------------------------------------------------------------------
1. The debian installer failed installing grub
  An installation setp failed. You can try to run the failing item again from the menu, or skip it and choose somehting else. the failing step is: install the GRUB boot loader on a hard disk.
2. The Broadcom Wireless chipset does not work with kernel 3.16
3. The nouveau driver goes haywire and causes the system to lock up during boot



## Issue resolutions
--------------------------------------------------------------------------------
Fix the "Install the GRUB boot loader on a hard disk" issue:


1. boot from  usb media
select Advanced options => Resuce mode
2. You will be prompted to select a device to use for your root file system. 

    If you did not encrypt your hard drive you should see an options that looks like this
    /dev/nvme0n1p3 or /dev/${HOSTNAME}-vg/root. 
    1. select your root file system
    1. Answer yes to mounting your /boot partition
    1. select Execute a shell in /dev/${HOSTNAME}-vg/root or /dev/nvme0n1p3 (which ever applies to you)
    1. select continue

    If you encrypted your hard drive and used lvm, do the following
    1. press ctrl+f2
    1. type ``` # cryptsetup luksOpen /dev/nvme0n1p3 ssd_unlock ```
    1. type ``` # vgchange -ay ssd_unlock ```
    1. type ``` # vgchange -ay ```
    1. press ctrl+f1
    1. select do no use a root file system
    1. select Choose a different root file system
    1. select your root file system
    1. Answer yes to mounting your /boot partition
    1. select Execute a shell in /dev/${HOSTNAME}-vg/root
    1. select continue

    This will start a shell where you can now fix your grub install.
3. Fix the console font so you can see. By default it's super small due to the lovely 4k screen :)

    1. type ```dpkg-reconfigure console-setup```

        Select all the options that apply to you. 
        I choose "Terminus" 16x32 (framebuffer only). It's nice on my eyes.
        Okay now that's done. Let's get back to fixing this boot loader issue

4. Mount your /boot/efi partition.
    1. type ```# fdisk -l | grep -i efi```

        once you find your /boot/efi partition mount it. Your be 512M in size with a TYPE of EFI

    2. type ```# mount /dev/nvme0n1p1 /boot/efi```



1. open shell
2. ctrl+f2

        chroot /target
        bash
        apt-get install -y efibootmgr grub-efi-amd64
        apt-get install -y tree
        [ "$(mount  | grep -i efi &>/dev/null )" -eq "0" ] && echo "EFI boot partition is mounted" || "You need to mount the efi partition"
        
        # tree /boot/efi
        /boot/efi
        |-- EFI
        |   `__ debian
        |       `-- grubx64.efi
        `__ boot
            `-- bootx64.efi
  
        3 directories, 2 files
        exit
        exit

3. ctrl+f1
4. select "Continue without boot loader"
5. select "<Continue>" when prompted with "No boot loader installed"
6. Once system has rebooted. rapidly press F2
7. Once in BIOS select Settings => General => Boot Sequence
8. Select "Add Boot Option"
9. For the Boot Option Name: field type Debian
10. For the File Name: field type  \boot\bootx64.efi
12. Click ok
13. Click Apply
14. Click ok
15. Click Exit



Create your own offline DVD. (This is a bit misleading because you need a working computer and a internet connection to at least create this dvd).

change luks password
http://askubuntu.com/questions/109898/how-to-change-the-password-of-an-encrypted-lvm-system-done-with-the-alternate-i
