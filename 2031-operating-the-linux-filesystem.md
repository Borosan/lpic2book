# 203.1. Operating the Linux filesystem

* **Topic 203: Filesystem and Devices**

**203.1 Operating the Linux filesystem**

**Weight:**4

Description: Candidates should be able to properly configure and navigate the standard Linux filesystem. This objective includes configuring and mounting various filesystem types.

**Key Knowledge Areas:**

* The concept of the fstab configuration
* Tools and utilities for handling swap partitions and files
* Use of UUIDs for identifying and mounting file systems
* Understanding of systemd mount units

**Terms and Utilities:**

* /etc/fstab
* /etc/mtab
* /proc/mounts
* mount and umount
* blkid
* sync
* swapon
* swapoff

## swap, swapon, swapoff

Previously we said that RAM is like a gateway of a town so that is too busy. Also we introduced some of techniques which is used by linux operating system in order to manage Memory. One of them was swap. Swap is emulated memory in disk. But what are its benefits and how dose it work ? Usually swap is used if computer run out of memory, in this condition two possible solutions are available. First avoiding user from running program or programs which require memory bigger that existing physical memory size and the second, crashing ! Obviously none of these solutions are acceptable. swap let us running programs and allocating them more memory than what we really have, by writing data on the hard disk temporarily. How ever this technique dose not guarantee performance.

swap can be put on a file\(swap file\) or can be an entire disk partition\(swap partition\)

### swap file

Lets create a file and use it as swap space:

```text
root@server1:~# free -m
              total        used        free      shared  buff/cache   available
Mem:            971         623          67           3         280         159
Swap:          1020          10        1010
root@server1:~# swapon -s
Filename                Type        Size    Used    Priority
/dev/sda5                                  partition    1045500    10976    -1
```

swapon -s command give us a summary of allocated swap spaces.

```text
root@server1:~# touch myswapfile
```

For being able to use this file as swap space, it should be populated and some meta data should be added, and its better to change the permission so just root has access to it:

```text
root@server1:~# dd if=/dev/zero of=myswapfile bs=1M count=500
500+0 records in
500+0 records out
524288000 bytes (524 MB, 500 MiB) copied, 1.61505 s, 325 MB/s

root@server1:~# mkswap myswapfile 
Setting up swapspace version 1, size = 500 MiB (524283904 bytes)
no label, UUID=867546e7-febc-44bc-9444-f2b7ef07824a

root@server1:~# ls -l
total 512004
-rw-r--r-- 1 root root 524288000 Dec 25 03:02 myswapfile
root@server1:~# chmod 600 myswapfile
```

and now lets add it to swap space

```text
root@server1:~# swapon myswapfile
```

and the result:

```text
root@server1:~# swapon -s
Filename                Type        Size    Used    Priority
/dev/sda5                                  partition    1045500    26020    -1
/root/myswapfile                           file        511996    0    -2
root@server1:~# free -m
              total        used        free      shared  buff/cache   available
Mem:            971         621          72           2         277         162
Swap:          1520          25        1495
```

in opposite to swapon command we can use swapoff to turn off swap on myswapfile.

```text
root@server1:~# swapoff myswapfile 
root@server1:~# swapon -s
Filename                Type        Size    Used    Priority
/dev/sda5                                  partition    1045500    25844    -1
```

### swap partition

swap partition is usually made automatically during linux installation, but it is possible to manipulate that or add another swap partition as needed.

```text
root@server1:~# swapon -s
Filename                Type        Size    Used    Priority
/dev/sda5                                  partition    1045500    45116    -1
root@server1:~# lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0    1G  0 disk 
sr0     11:0    1 1024M  0 rom  
sda      8:0    0   50G  0 disk 
├─sda2   8:2    0    1K  0 part 
├─sda5   8:5    0 1021M  0 part [SWAP]
└─sda1   8:1    0   49G  0 part /

root@server1:~# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.27.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x1f54816e.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-2097151, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-2097151, default 2097151): 

Created a new partition 1 of type 'Linux' and of size 1023 MiB.
Command (m for help): t
Selected partition 1
Partition type (type L to list all types): l

 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris        
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden or  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx         
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data    
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility   
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt         
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access     
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi ea  Rufus alignment
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         eb  BeOS fs        
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ee  GPT            
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        ef  EFI (FAT-12/16/
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f0  Linux/PA-RISC b
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f1  SpeedStor      
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f4  SpeedStor      
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      f2  DOS secondary  
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fb  VMware VMFS    
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fc  VMware VMKCORE 
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fd  Linux raid auto
1c  Hidden W95 FAT3 75  PC/IX           bc  Acronis FAT32 L fe  LANstep        
1e  Hidden W95 FAT1 80  Old Minix       be  Solaris boot    ff  BBT            
Partition type (type L to list all types): 82
Changed type of partition 'Linux' to 'Linux swap / Solaris'.
Command (m for help): w   
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

root@server1:~# fdisk -l /dev/sdb
Disk /dev/sdb: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x1f54816e

Device     Boot Start     End Sectors  Size Id Type
/dev/sdb1        2048 2097151 2095104 1023M 82 Linux swap / Solaris
```

And lets add required meta data to desired partion before making swap on that:

```text
root@server1:~# lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0    1G  0 disk 
└─sdb1   8:17   0 1023M  0 part 
sr0     11:0    1 1024M  0 rom  
sda      8:0    0   50G  0 disk 
├─sda2   8:2    0    1K  0 part 
├─sda5   8:5    0 1021M  0 part [SWAP]
└─sda1   8:1    0   49G  0 part /
root@server1:~# mkswap /dev/sdb1
Setting up swapspace version 1, size = 1023 MiB (1072689152 bytes)
no label, UUID=6a1c543e-3224-4cfb-82ba-d802051c4b76
```

and every thing is ready to put the swap on our new fresh partition:

```text
root@server1:~# swapon /dev/sdb1
root@server1:~# swapon -s
Filename                Type        Size    Used    Priority
/dev/sda5                                  partition    1045500    48440    -1
/dev/sdb1                                  partition    1047548    0    -2
root@server1:~# free -m
              total        used        free      shared  buff/cache   available
Mem:            971         625          69           4         276         156
Swap:          2043          47        1996
```

use swapon -p to change the priority of using swap devices :

```text
root@server1:~# swapoff /dev/sda5
root@server1:~# swapon -p -2 /dev/sda5
root@server1:~# swapon -s
Filename                Type        Size    Used    Priority
/dev/sda5                                  partition    1045500    0    -2
/dev/sdb1                                  partition    1047548    3280    -1

root@server1:~# swapoff /dev/sdb1
root@server1:~# swapon -p -2 /dev/sdb1
root@server1:~# swapon -s
Filename                Type        Size    Used    Priority
/dev/sda5                                  partition    1045500    0    -1
/dev/sdb1                                  partition    1047548    0    -2
```

## mount, umount , mtab, fstab

When we add a new internal hard disk to our computer it is bounded but it is not mounted. To make it usable first we need to make a partition on that, format it with a file system and then mount it, these is current setting of our computer:

```text
root@server2:~# ls /dev | grep sd
sda
sda1
sda2
sda5
root@server2:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0     11:0    1 1024M  0 rom  
sda      8:0    0   50G  0 disk 
├─sda2   8:2    0    1K  0 part 
├─sda5   8:5    0 1021M  0 part [SWAP]
└─sda1   8:1    0   49G  0 part /
```

as an example lets add a new 10GB hard disk :

```text
root@server2:~# ls /dev | grep sd
sda
sda1
sda2
sda5
sdb
root@server2:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0   10G  0 disk 
sr0     11:0    1 1024M  0 rom  
sda      8:0    0   50G  0 disk 
├─sda2   8:2    0    1K  0 part 
├─sda5   8:5    0 1021M  0 part [SWAP]
└─sda1   8:1    0   49G  0 part /
root@server2:~# fdisk /dev/sdb 

Welcome to fdisk (util-linux 2.27.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x21fcd674.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-20971519, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-20971519, default 20971519): 

Created a new partition 1 of type 'Linux' and of size 10 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

root@server2:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0   10G  0 disk 
└─sdb1   8:17   0   10G  0 part 
sr0     11:0    1 1024M  0 rom  
sda      8:0    0   50G  0 disk 
├─sda2   8:2    0    1K  0 part 
├─sda5   8:5    0 1021M  0 part [SWAP]
└─sda1   8:1    0   49G  0 part /

root@server2:~# mkfs.ext
mkfs.ext2     mkfs.ext3     mkfs.ext4     mkfs.ext4dev  
root@server2:~# mkfs.ext4 /dev/sdb1
mke2fs 1.42.13 (17-May-2015)
Creating filesystem with 2621184 4k blocks and 655360 inodes
Filesystem UUID: a3be5874-890b-46a6-b4e5-a3f88998ad91
Superblock backups stored on blocks: 
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

Now lets mount /dev/sdb1 on a mount point to use it:

```text
root@server2:~# mkdir /mnt/my10ghdd
root@server2:~# mount -t ext4 /dev/sdb1 /mnt/my10ghdd/

root@server2:~# touch /mnt/my10ghdd/{myfile1,myfile2,myfile3}
root@server2:~# ls /mnt/my10ghdd/
lost+found  myfile1  myfile2  myfile3
```

-t means what type of file system is going to be mounted, /dev/sdb1 is mount device and /mnt/my10ghdd is mount point.

| mount command switches | Description |
| :--- | :--- |
| mount -V | Output version |
| mount -v | Verbose mode |
| mount -h | Prints help message |
| mount -a | mount all file systems mentioned in /etc/fstab file |
| mount -n /dev/sda7 /mnt/newpartition | mounting without writing in /etc/mtab file |
| mount -t &lt;File System Type&gt; /dev/sda7 /mnt/newpartition | indicates the File System ext2, ext3, ext4, iso9660,  ntfs, swap, auto |
| mount -o &lt;options&gt; /dev/sda7 /mnt/newpartition | ro, rw, exec/noexec, suid/nosuid, dev/nodev, sync/async, user/users |

Before exploring mount command options lets talk about sync/async concept:

### sync/async

As spped difference between CPU and Hard Disk or other lazy devices, RAM are used. But Still there is a gap and latency between RAM and 3rd storage media. The solutions for omitting this speed gap are caches and buffers. Imagine CPU wants to write some thing on a poor floppy Disk. The data can be first stored in RAM and CPU can invest its valuable time on other things and than Data is writed down on floppy disk from ram.This is what logic accepts and usually happens, which is called "async". In opposite to "async" we have "sync" option which writes down dataat the same time on the floppy, and obviously take more time.

### mount command options:

| mount option | Description |
| :--- | :--- |
| ro | read-only |
| rw | read-write |
| exec/noexec | Permit/Prevent execution of binaries |
| suid/nosuid | Permit/Block the operation of suid, guid bits |
| dev/nodev | Interpret/Do not interpret character or block special devices on the file system |
| sync/async | I/O to the file system is done \(a\)synchronously |
| defaults | Use default options: rw, suid, dev, exec, auto, nouser, and async |
| remount | Attempt to remount an already-mounted file system, usually used to change mount options |

### /etc/mtab \(contraction of mounted file systems table\)

mtab is a file which is kept update with the mount subsystem.It lists currently mounted file system, how ever kernel doesn't do any thing with the mtab file. kernel puts its settings in /proc/mounts and /proc/self/mounts.

```text
root@server1:~# cat /etc/mtab 
sysfs /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
udev /dev devtmpfs rw,nosuid,relatime,size=475204k,nr_inodes=118801,mode=755 0 0
devpts /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000 0 0
tmpfs /run tmpfs rw,nosuid,noexec,relatime,size=99492k,mode=755 0 0
/dev/sda1 / ext4 rw,relatime,errors=remount-ro,data=ordered 0 0
securityfs /sys/kernel/security securityfs rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /dev/shm tmpfs rw,nosuid,nodev 0 0
tmpfs /run/lock tmpfs rw,nosuid,nodev,noexec,relatime,size=5120k 0 0
tmpfs /sys/fs/cgroup tmpfs ro,nosuid,nodev,noexec,mode=755 0 0
cgroup /sys/fs/cgroup/systemd cgroup rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/lib/systemd/systemd-cgroups-agent,name=systemd 0 0
pstore /sys/fs/pstore pstore rw,nosuid,nodev,noexec,relatime 0 0
cgroup /sys/fs/cgroup/net_cls,net_prio cgroup rw,nosuid,nodev,noexec,relatime,net_cls,net_prio 0 0
cgroup /sys/fs/cgroup/memory cgroup rw,nosuid,nodev,noexec,relatime,memory 0 0
cgroup /sys/fs/cgroup/cpu,cpuacct cgroup rw,nosuid,nodev,noexec,relatime,cpu,cpuacct 0 0
cgroup /sys/fs/cgroup/pids cgroup rw,nosuid,nodev,noexec,relatime,pids 0 0
cgroup /sys/fs/cgroup/devices cgroup rw,nosuid,nodev,noexec,relatime,devices 0 0
cgroup /sys/fs/cgroup/blkio cgroup rw,nosuid,nodev,noexec,relatime,blkio 0 0
cgroup /sys/fs/cgroup/cpuset cgroup rw,nosuid,nodev,noexec,relatime,cpuset 0 0
cgroup /sys/fs/cgroup/freezer cgroup rw,nosuid,nodev,noexec,relatime,freezer 0 0
cgroup /sys/fs/cgroup/hugetlb cgroup rw,nosuid,nodev,noexec,relatime,hugetlb 0 0
cgroup /sys/fs/cgroup/perf_event cgroup rw,nosuid,nodev,noexec,relatime,perf_event 0 0
systemd-1 /proc/sys/fs/binfmt_misc autofs rw,relatime,fd=26,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=14873 0 0
debugfs /sys/kernel/debug debugfs rw,relatime 0 0
mqueue /dev/mqueue mqueue rw,relatime 0 0
hugetlbfs /dev/hugepages hugetlbfs rw,relatime 0 0
fusectl /sys/fs/fuse/connections fusectl rw,relatime 0 0
vmware-vmblock /run/vmblock-fuse fuse.vmware-vmblock rw,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other 0 0
tmpfs /run/user/1000 tmpfs rw,nosuid,nodev,relatime,size=99492k,mode=700,uid=1000,gid=1000 0 0
gvfsd-fuse /run/user/1000/gvfs fuse.gvfsd-fuse rw,nosuid,nodev,relatime,user_id=1000,group_id=1000 0 0
```

When we type mount the content of mtab file is shown:

```text
root@server1:~# mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=475204k,nr_inodes=118801,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,noexec,relatime,size=99492k,mode=755)
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro,data=ordered)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/lib/systemd/systemd-cgroups-agent,name=systemd)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=26,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=14873)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
mqueue on /dev/mqueue type mqueue (rw,relatime)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime)
fusectl on /sys/fs/fuse/connections type fusectl (rw,relatime)
vmware-vmblock on /run/vmblock-fuse type fuse.vmware-vmblock (rw,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,size=99492k,mode=700,uid=1000,gid=1000)
gvfsd-fuse on /run/user/1000/gvfs type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
```

They are the same, as mtab list includes some dynamically mounted system objects it shouldn't be edited by the Administrators.

### /etc/fstab \(file systems table\)

Every thing seems right but our mounted devices are not persistent and wouldn't be accessible after reboot so far.To make a persistent mount we should use fstab. fstab is very similar to mtab but they are not related. fstab is more easier to edit:

```text
root@server1:~# cat /etc/fstab 
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
UUID=142a64e5-96f3-4789-9c91-1dc1570057b7 /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=b4801c8b-ca75-4548-8697-182d1b6d895c none            swap    sw              0       0
/dev/fd0        /media/floppy0  auto    rw,user,noauto,exec,utf8 0       0
# for making permanent swap on /dev/sdb1
UUID="b4801c8b-ca75-4548-8697-182d1b6d895c" none    swap    sw    0 0
```

Desciption:

| Item | Example | Description |
| :--- | :--- | :--- |
| &lt;file system&gt; | /dev/floppy0 or UUID | Device/partion that contains file system |
| &lt;mount point&gt; | /media/floppy | Where do we wana mount device/partition |
| &lt;type&gt; | ext4 | Type of File system |
| &lt;option&gt; | rw, user, noauto, exec, ... | mount options of accessing device/partition |
| &lt;dump&gt; | 0 or 1 | enable/disbale backing up device/pertition |
| &lt;pass&gt; | 0 or 1 or 2 | Control the order of fsck check partition/device during boot process |

Lets take a look at fstab mount options:

### fstab mount options:

Obviously fstab mount options and mount command options are the same but there some options which are meaning full if we are talking about fstab configuration:

| mount option | Description |
| :--- | :--- |
| user | Allow an ordinary user to mount the file system. The name of the mounting user is written to mtab so that he can unmount the file system again.This option Implies the options noexec, nosuid, and nodev \(unless overridden by subsequent options, as in the option line user,exec,dev,suid\) |
| users | Allow every user to mount and unmount the file system. Implies the options noexec, nosuid, and nodev \(unless overridden by subsequent options, as in the option line users,exec,dev,suid\). |
| nouser | Forbin an ordinary user to mount the File System, this is the default |
| auto | File System can be mounted Automatically after boot . using mount -a option also mount Device/partition if it is not mounted |
| noauto | The File System will NOT be automatically mounted after reboot, mount -a wouldn't considering this Device/Partition.You must explicitly mount the filesystem. |
| \_netdev | filesystem resides on a device that requires network access \(used to prevent the system from attempting to mount these filesystems until the network has been enabled on the system\) |

### blkid

blkid show all information of block devices in our system,

```text
/dev/sda1: UUID="142a64e5-96f3-4789-9c91-1dc1570057b7" TYPE="ext4" PARTUUID="101c66bb-01"
/dev/sda5: UUID="b4801c8b-ca75-4548-8697-182d1b6d895c" TYPE="swap" PARTUUID="101c66bb-05"
/dev/sdb1: UUID="6a1c543e-3224-4cfb-82ba-d802051c4b76" TYPE="swap" PARTUUID="1f54816e-01"
```

in fstab we can use UUID \(Universally Unique Identifier\) instead of Device abstract name from /dev/ directory. This way we reduce the mount fails because HAL might change the name as time passes and other Disks are installed.

And Finnally lets go back and make the swap partition permanent by editing fstab file :

```text
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
UUID=142a64e5-96f3-4789-9c91-1dc1570057b7 /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=b4801c8b-ca75-4548-8697-182d1b6d895c none            swap    sw              0       0
/dev/fd0        /media/floppy0  auto    rw,user,noauto,exec,utf8 0       0
# for making permanent swap on /dev/sdb1
UUID="b4801c8b-ca75-4548-8697-182d1b6d895c" none        swap    sw      0 0
```

and another way to see UUID of devices is:

```text
root@server1:/# ls -l /dev/disk/by-uuid/
total 0
lrwxrwxrwx 1 root root 10 Dec 26 22:17 142a64e5-96f3-4789-9c91-1dc1570057b7 -> ../../sda1
lrwxrwxrwx 1 root root 10 Dec 26 22:17 6a1c543e-3224-4cfb-82ba-d802051c4b76 -> ../../sdb1
lrwxrwxrwx 1 root root 10 Dec 26 22:17 b4801c8b-ca75-4548-8697-182d1b6d895c -> ../../sda5
```

