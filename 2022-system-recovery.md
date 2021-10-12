# 202.2. System Recovery

## **202.2 System Recovery**

**Weight:** 4

**Description:** Candidates should be able to properly manipulate a Linux system during both the boot process and during recovery mode. This objective includes using both the init utility and init-related kernel options. Candidates should be able to determine the cause of errors in loading and usage of bootloaders. GRUB version 2 and GRUB Legacy are the bootloaders of interest. Both BIOS and UEFI systems are covered.

**Key Knowledge Areas:**

* BIOS and UEFI
* NVMe booting
* GRUB version 2 and Legacy
* grub shell
* boot loader start and hand off to kernel
* kernel loading
* hardware initialisation and setup
* daemon/service initialisation and setup
* Know the different boot loader install locations on a hard disk or removable device.
* Overwrite standard boot loader options and using boot loader shells.
* Use systemd rescue and emergency modes.

**Terms and Utilities:**

* mount
* fsck
* inittab, telinit and init with SysV init
* The contents of /boot/, /boot/grub/ and /boot/efi/
* EFI System Partition (ESP)
* GRUB
* grub-install
* efibootmgr
* UEFI shell
* initrd, initramfs
* Master boot record
* systemctl

### Boot process overview

lets take a look at boot process and stick what ever we have learned till now:

```
BIOS / UEFI
    |
    Bootable Disk
        |
        Boot Loader(LILO,Grub,Grub2)
            |
            Kernel (initramfs/initd)
                |
                init --->systemd / upstart / SysV
                            |
                            everything---shell & other services and programs
```

if any problems occur on any of these steps, boot process fails. It might be motherboard problems, Disk problems or boot loader problem.lets start from very beginig.

### BIOS basic Input Output System vs UEFI(EFI) Unified Extensible Firmware Interface

When we start computer there should be away to wake up the kernel from sleep. But even our pour bootloader sleeps on the hard disk and has to access disk when no kernel driver is available. So there should be a hardware solution to access hard disk and starts boot loader and then bootloader wake up the kernel.

BIOS and UEFI are both interfaces for accessing disk.They sit between hardware/firmware and operating system and help computers to start. BIOS has become pretty old and UEFI has developed to retire it. UEFI has many advantages and has become default interface in modern PCs. UEFI is able to boot from large disks when we use GPT.

### MBR(Master Boot Record)

on BIOS system Master boot record is read from disk. MBR is not part of any file system and considered as metadata of your hard disk.How is it working ?Master boot record holds two things, some of or all of boot loader program and the partition table. At first stage just 446 kb of disk is read, (primary boot loader/IPL) and its job is to load second stage .as 512 bytes is so small to keep whole needed files to load the kernel, In second stage a MB of disk is read, which is meta data area of disk and every thing which is needed to load kernel should be exsited here. Boot loader(grub2) located in first 30 KB of hard disk immediately after MBR. grub load kernel and inintrd from /etc/boot/grub/grub.conf and loads other modules as needed. Grub loads GUI from /grub/splash.xpm.gz and starts the system.

```
root@server1:~# xxd -l 512 /dev/sda
00000000: eb63 9010 8ed0 bc00 b0b8 0000 8ed8 8ec0  .c..............
00000010: fbbe 007c bf00 06b9 0002 f3a4 ea21 0600  ...|.........!..
00000020: 00be be07 3804 750b 83c6 1081 fefe 0775  ....8.u........u
00000030: f3eb 16b4 02b0 01bb 007c b280 8a74 018b  .........|...t..
00000040: 4c02 cd13 ea00 7c00 00eb fe00 0000 0000  L.....|.........
00000050: 0000 0000 0000 0000 0000 0080 0100 0000  ................
00000060: 0000 0000 fffa 9090 f6c2 8074 05f6 c270  ...........t...p
00000070: 7402 b280 ea79 7c00 0031 c08e d88e d0bc  t....y|..1......
00000080: 0020 fba0 647c 3cff 7402 88c2 52bb 1704  . ..d|<.t...R...
00000090: f607 0374 06be 887d e817 01be 057c b441  ...t...}.....|.A
000000a0: bbaa 55cd 135a 5272 3d81 fb55 aa75 3783  ..U..ZRr=..U.u7.
000000b0: e101 7432 31c0 8944 0440 8844 ff89 4402  ..t21..D.@.D..D.
000000c0: c704 1000 668b 1e5c 7c66 895c 0866 8b1e  ....f..\|f.\.f..
000000d0: 607c 6689 5c0c c744 0600 70b4 42cd 1372  `|f.\..D..p.B..r
000000e0: 05bb 0070 eb76 b408 cd13 730d 5a84 d20f  ...p.v....s.Z...
000000f0: 83d0 00be 937d e982 0066 0fb6 c688 64ff  .....}...f....d.
00000100: 4066 8944 040f b6d1 c1e2 0288 e888 f440  @f.D...........@
00000110: 8944 080f b6c2 c0e8 0266 8904 66a1 607c  .D.......f..f.`|
00000120: 6609 c075 4e66 a15c 7c66 31d2 66f7 3488  f..uNf.\|f1.f.4.
00000130: d131 d266 f774 043b 4408 7d37 fec1 88c5  .1.f.t.;D.}7....
00000140: 30c0 c1e8 0208 c188 d05a 88c6 bb00 708e  0........Z....p.
00000150: c331 dbb8 0102 cd13 721e 8cc3 601e b900  .1......r...`...
00000160: 018e db31 f6bf 0080 8ec6 fcf3 a51f 61ff  ...1..........a.
00000170: 265a 7cbe 8e7d eb03 be9d 7de8 3400 bea2  &Z|..}....}.4...
00000180: 7de8 2e00 cd18 ebfe 4752 5542 2000 4765  }.......GRUB .Ge
00000190: 6f6d 0048 6172 6420 4469 736b 0052 6561  om.Hard Disk.Rea
000001a0: 6400 2045 7272 6f72 0d0a 00bb 0100 b40e  d. Error........
000001b0: cd10 ac3c 0075 f4c3 bb66 1c10 0000 8020  ...<.u...f..... 
000001c0: 2100 83fe ffff 0008 0000 0000 2006 00fe  !........... ...
000001d0: ffff 05fe ffff fe0f 2006 02e8 1f00 0000  ........ .......
000001e0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001f0: 0000 0000 0000 0000 0000 0000 0000 55aa  ..............U.
```

### GPT ( GUID Partition Table)

in UEFI system, GPT partition table is used. GPT partition table contains an EFI System Partition (ESP).Inside that there is always /efi directory. /efi directory might contains one or many boot loaders. Each boot loader has its own identifier and corresponding directory. So if you are using boot linux and microsoft windows, its usual to have both /efi/grub and /efi/microsoft directories.Boot loaders in this partition has .efi extension. So as you can see UEFI/EFIgive more flexibility and you can support more than one operating system by just having an ESP partition on a hard disk. no more pain :).

```
root@server3:~# gdisk  /dev/sda
GPT fdisk (gdisk) version 1.0.1

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): p
Disk /dev/sda: 62914560 sectors, 30.0 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 3F67AA1F-8E90-4EA6-AB2E-2B8287545A35
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 62914526
Partitions will be aligned on 2048-sector boundaries
Total free space is 4029 sectors (2.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1050623   512.0 MiB   EF00  EFI System Partition
   2         1050624        58722303   27.5 GiB    8300  
   3        58722304        62912511   2.0 GiB     8200  

Command (? for help): q

root@server3:~# mount | grep efi
efivarfs on /sys/firmware/efi/efivars type efivarfs (rw,nosuid,nodev,noexec,relatime)
/dev/sda1 on /boot/efi type vfat (rw,relatime,fmask=0077,dmask=0077,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
```

```
root@server3:/boot/efi# tree
.
└── EFI
    └── ubuntu
        ├── fw
        ├── fwupx64.efi
        ├── grub.cfg
        ├── grubx64.efi
        ├── mmx64.efi
        └── shimx64.efi

3 directories, 5 files
```

GPT is backward compatible and it has MBR .It is not used. it is MBR which is GPT compatible.

```
root@server3:~# xxd -l 512 /dev/sda
00000000: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000010: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000030: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000040: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000050: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000060: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000070: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000080: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000090: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000a0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000b0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000c0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000d0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000e0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000f0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000100: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000110: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000120: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000130: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000140: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000150: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000160: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000170: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000180: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000190: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001a0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001b0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001c0: 0100 eefe ffff 0100 0000 ffff bf03 0000  ................
000001d0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001e0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001f0: 0000 0000 0000 0000 0000 0000 0000 55aa  ..............U.
```

## MBR vs GPT

Lets compare these two:

| item                           | mbr                            | gpt                                       |
| ------------------------------ | ------------------------------ | ----------------------------------------- |
| Number of Supported partitions | 4                              | 128 (modifiable )                         |
| Stored in                      | mbr (out side of file system)  | GPT  partition                            |
| Partition Types                | Primary, Logical, Extended     | No more(Primary, logical, Extended)       |
| Maximum Partition Size         | 2^64 \* 512=2 T                | 2^64 \* 512 =>hit file system limitations |
| Backup                         | no, nothing, easily damaged :( | store a backup at the end of disk         |
| Work With                      | BIOS                           | UEFI and BIOS (Backward compatible)       |

## Boot Loader Recovery General Notes

Recovering boot loader needs to specify three major required elements. We have already got familiar with all 3:

1. root partion
2. kernel and boot partition as its argument (usually same as root)
3. initrd/initramfs 

how ever some commands are different in grub version 1 and version 2.

* **Recovering Grub Legacy**

grub version1 dosn't have "ls" command.Do not forget it starts counting hard disk partitions from "Zero".

```
grub>help ###to see grub v1 commands

###1.root,What the root is ?
grub>root ### if nothing showed up, we have to boot system by using another media

###2.Kernel, which kernel we want to boot from? nad where is the boot partition? consider lvm :(
grub> kernel /vmlinuz.x.y.z root=/dev/sdaX

###3.initramfs/initrd , Which initramdisk is going to be loaded? It should be the same as kernel version
grub>initrd /initrd-x.y.z 

### and finally boot the system
grub>boot
```

after rebooting system install grub again:

```
[root@localhost ~]# grub-install /dev/sda
Installation finished. No error reported.
This is the contents of the device map /boot/grub/device.map.
Check if this is correct or not. If any of the lines is incorrect,
fix it and re-run the script `grub-install'.

# this device map was generated by anaconda
(hd0)     /dev/sda
```

* **Recovering Grub2**

Grub2 is more enhanced than legacy grab and if boot problems occur, grub might left system in different states.

| grub command prompt | Description                                 |                                                                                                                                                                                                                                                                                                                                      |
| ------------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| grub>               | prompt                                      | GRUB 2 loaded modules but was unable to find the grub.cfg file. = has already found root partition                                                                                                                                                                                                                                   |
| grub rescue>        | prompt                                      | GRUB 2 failed to find its grub folder, or failed to load the normal module. = has not found root partition                                                                                                                                                                                                                           |
| grub>:              | The grub prompt on a blank screen           | GRUB 2 has found the boot information but has been either unable to locate or unable to use an existing GRUB 2 configuration file (usually grub.cfg)                                                                                                                                                                                 |
| grub rescue>:       | the rescue mode                             | GRUB 2 is unable to find the grub folder or its contents are missing/corrupted. The grub folder contains the GRUB 2 menu, modules and stored environmental data.                                                                                                                                                                     |
| GRUB-               | a single word, with no prompt and no curser | GRUB has failed to find even the most basic information, usually contained in the MBR or boot sector.                                                                                                                                                                                                                                |
|                     | Busybox or Initramfs                        | GRUB 2 began the boot process but there was a problem passing control to the operating system. Possible causes include an incorrect UUID or root= designation in the 'linux' line or a corrupted kernel.                                                                                                                             |
|                     | Frozen splash screen                        | blinking cursor with no grub> or grub rescue prompt. Possible video issues with the kernel. While these failures are not of GRUB 2's making, it may still be able to help. GRUB 2 allows pre-boot editing of its menu and the user may restore functionality by adding and/or removing kernel options in a menuentry before booting. |

each failer might need different remedies here lets take a look at the first one. Grub2 supports "ls" , "cat" command and support tab tab completion. it starts counting partitions from "One" . Strange world huuh ?

```
###1.root, Finding root partition --- no "root command" in grub2
grub>ls ### to see partitions, (hdX,A) (hdX,B) (hdX,C) , ...
grub>ls (hdX,A) ###try to guess based on content, setting root partition in grub 2 is different from grub 1
grub>set ###it show all current grub2 variables, we should check/create/fix two of them:
         ###prefix=(hdX,A)/boot/grub
         ###root=hdX,A
grub>set root=(hdA,Y) ###set/modify required variables base on information that we have got from ls
grub>set ###check results

###2.kernel, introducing kernel --- no "kernel command" in grub 2
grub>linux /boot/vmlinux.x.y.z root=/dev/sdY

###3.initramfs/initrd, give appropriate initramdisk for kernel , consider versions 
grub>initrd /boot/init.img.x.y.z

###and finally
grub>boot
```

and finally do some post configuration :

```
root@server1:~# update-grub
Generating grub configuration file ...
Warning: Setting GRUB_TIMEOUT to a non-zero value when GRUB_HIDDEN_TIMEOUT is set is no longer supported.
Found linux image: /boot/vmlinuz-4.10.0-42-generic
Found initrd image: /boot/initrd.img-4.10.0-42-generic
Found linux image: /boot/vmlinuz-4.10.0-40-generic
Found initrd image: /boot/initrd.img-4.10.0-40-generic
Found linux image: /boot/vmlinuz-4.10.0-28-generic
Found initrd image: /boot/initrd.img-4.10.0-28-generic
Found memtest86+ image: /boot/memtest86+.elf
Found memtest86+ image: /boot/memtest86+.bin
done
```

and install grub:

```
root@server1:~# grub-install /dev/sda
Installing for i386-pc platform.
Installation finished. No error reported.
```
