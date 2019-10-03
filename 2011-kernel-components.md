# 201.1. Kernel Components

## **201.1 Kernel Components**

**Weight:** 2

**Description:** Candidates should be able to utilize kernel components that are necessary to specific hardware, hardware drivers, system resources and requirements. This objective includes implementing different types of kernel images, identifying stable and development kernels and patches, as well as using kernel modules.

**Key Knowledge Areas:**

* Kernel 2.6.x, 3.x and 4.x documentation

**Terms and Utilities:**

* /usr/src/linux/
* /usr/src/linux/Documentation/
* zImage
* bzImage
* xz compression

### Kernel Version Numbering

Linux Kernel as heart of Linux Operating System was numbered from its beginning. The rule rule of version numbering was like this:

| 2 | 5 | 75 | 3 |
| :---: | :---: | :---: | :---: |
| Version Number | Major Revision | Minor Revision | Correction/Patch |

where the "odd" or "even" revision number had different concepts. "odd" numbers was used for "development versions" and this showed they where some how unstable and weren't suitable for production environment. On the other hand "even" reversion number showed "stable" version and so it gave hope for Reliability. But this story is for past. When Kernel 2.6 arrived they decided to release just "stable" kernels and so they stick to version 2.6.X. So all versions after 2.6 , weather they have "odd" or "even" numbers, they are all "stable. Kernel 2.6.X was alive till version number 2.6.39.4 , that is a long number huh ? Finally Linus Trovalds accepted to using version 3 and so on . and changed the rule:

| 3 | 0 | X |
| :---: | :---: | :---: |
| Version Number | Major Reversion | Minor Reversion |

and we don't need to be worried about correction/patch number. 3.19 is last version and version 4 is active now. Lets Check :

```text
root@server1:~# uname -a
Linux server1 4.10.0-40-generic #44~16.04.1-Ubuntu SMP Thu Nov 9 15:37:44 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
root@server1:~# uname -r
4.10.0-40-generic
root@server1:~#
```

as you can see we are on version "4", Major release "10" and ubuntu specified patch "0-40-generic"

To learn How Linux Kernels and its components how placed in different places of linux tree, think of octopus :\). This way you will have enough insight to work with kernel, upgrade it and troubleshoot linux system. Lets start learning theme little by little:

### initial Ram Disk / initial Ram File System \(/boot/\)

After Bios/UEFI done his job and pass the control to boot loader, boot loader tries to load the kernel. But loading kernel is not that much easy. Kernel need some drivers to be loaded. How Linux achieve that ? initrd. initrd \(initial ramdisk\)/initramfs is a scheme for loading a temporary root file system into memory, which may be used as part of the Linux startup process. initrd and initramfs refer to two different methods of achieving this but do the same thing. Both are commonly used to make preparations before the real root file system can be mounted.

```text
root@server1:~# cd /boot/
root@server1:/boot# ls -l
total 107160
-rw-r--r-- 1 root root  1443598 Jul 20 07:11 abi-4.10.0-28-generic
-rw-r--r-- 1 root root  1443962 Nov  9 12:56 abi-4.10.0-40-generic
-rw-r--r-- 1 root root   204970 Jul 20 07:11 config-4.10.0-28-generic
-rw-r--r-- 1 root root   204970 Nov  9 12:56 config-4.10.0-40-generic
drwxr-xr-x 5 root root     4096 Nov 29 00:17 grub
-rw-r--r-- 1 root root 41799650 Nov 29 00:15 initrd.img-4.10.0-28-generic
-rw-r--r-- 1 root root 41801572 Nov 29 00:17 initrd.img-4.10.0-40-generic
-rw-r--r-- 1 root root   182704 Jan 28  2016 memtest86+.bin
-rw-r--r-- 1 root root   184380 Jan 28  2016 memtest86+.elf
-rw-r--r-- 1 root root   184840 Jan 28  2016 memtest86+_multiboot.bin
-rw------- 1 root root  3718582 Jul 20 07:11 System.map-4.10.0-28-generic
-rw------- 1 root root  3722376 Nov  9 12:56 System.map-4.10.0-40-generic
-rw-r--r-- 1 root root  7398656 Nov 26 02:37 vmlinuz-4.10.0-28-generic
-rw------- 1 root root  7407392 Nov  9 12:56 vmlinuz-4.10.0-40-generic
```

What lets our kernel to boot up is initrd.img-4.10.0-40-generic which populate RAM during boot process.do not forget that each kernel version requires its initrd with the same version. Now it is Kernel turn to be loaded. But it has its own story, kernel must be some how compressed inorder to fit in ram. first lets start with some terminology lesson:

### XZ Compression

Sine very beginning versions of Kernel, there where some methods to compress it. As time passes and kernel grows and new need and performance issues appears, different methods used like gzip, bzip2, LZMA, XZ, LZO. All of us are familiar with compression tools and have worked with at least one of them like RAR, Zip, 7-Zip , ... . Like others XZ Compression is a compression program with its file format. It incorporates with LZMA/LZMA2 compression algorithms which is used by famous 7-zip how ever, they are not compatible. For LPI exam we are requested to know about it existence and we will see it again when we talk about kernel in next lessons.

## Whats is zImage an bzImage?

During old days Computers where in lack of enough memory, so there should be a way to fit and populate RAM with pre-compiled kernel before root fs mounted. zImage was a name that used to call compressed tiny pre-compiled kernel .azImage was so small and it fit in to just 512k of memory.

Little by little as kernel grows and memories get bigger, 512k wasn't enough. So bzImage come across and if you have more than 512k zImage, is called "Big zImage". Nowadays zImage just might be seen in some embedded linux systems and bzImage might used more.bzImag dosn't refer to special compression tool like bzip2, bzImage under the hood might used different tool for compressing .Keep it for now and we will back to this topic in next lessnons.

## Kernel Modules \(/lib/modules/\)

in /lib/modules/ you can see Folders contains each kernel's modules:

```text
root@server1:~# cd /lib/modules
root@server1:/lib/modules# ls -l
total 8
drwxr-xr-x 5 root root 4096 Nov 26 02:46 4.10.0-28-generic
drwxr-xr-x 5 root root 4096 Nov 29 00:16 4.10.0-40-generic
root@server1:/lib/modules# ls -l 4.10.0-40-generic/
total 4956
lrwxrwxrwx  1 root root      40 Nov  9 13:46 build -> /usr/src/linux-headers-4.10.0-40-generic
drwxr-xr-x  2 root root    4096 Nov  9 13:10 initrd
drwxr-xr-x 14 root root    4096 Nov 29 00:14 kernel
-rw-r--r--  1 root root 1166900 Nov 29 00:16 modules.alias
-rw-r--r--  1 root root 1150091 Nov 29 00:16 modules.alias.bin
-rw-r--r--  1 root root    7317 Nov  9 12:56 modules.builtin
-rw-r--r--  1 root root    9142 Nov 29 00:16 modules.builtin.bin
-rw-r--r--  1 root root  523389 Nov 29 00:16 modules.dep
-rw-r--r--  1 root root  741387 Nov 29 00:16 modules.dep.bin
-rw-r--r--  1 root root     285 Nov 29 00:16 modules.devname
-rw-r--r--  1 root root  196852 Nov  9 12:56 modules.order
-rw-r--r--  1 root root     429 Nov 29 00:16 modules.softdep
-rw-r--r--  1 root root  558199 Nov 29 00:16 modules.symbols
-rw-r--r--  1 root root  680971 Nov 29 00:16 modules.symbols.bin
drwxr-xr-x  3 root root    4096 Nov 29 00:14 vdso
```

## Kernel \(/usr/src/\)

And Finally where is the Kernel itself ?

```text
root@server1:~# cd /usr/src/
root@server1:/usr/src# ls -l
total 16
drwxr-xr-x 27 root root 4096 Aug  1 04:23 linux-headers-4.10.0-28
drwxr-xr-x  7 root root 4096 Aug  1 04:23 linux-headers-4.10.0-28-generic
drwxr-xr-x 27 root root 4096 Nov 29 00:14 linux-headers-4.10.0-40
drwxr-xr-x  7 root root 4096 Nov 29 00:14 linux-headers-4.10.0-40-generic
```

