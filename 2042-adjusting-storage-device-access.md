# 204.2. Adjusting Storage Device Access

## **204.2 Adjusting Storage Device Access**

**Weight:** 2

**Description:** Candidates should be able to configure kernel options to support various drives. This objective includes software tools to view & modify hard disk settings including iSCSI devices.

**Key Knowledge Areas:**

* Tools and utilities to configure DMA for IDE devices including ATAPI and SATA
* Tools and utilities to configure Solid State Drives including AHCI and NVMe
* Tools and utilities to manipulate or analyse system resources (e.g. interrupts)
* Awareness of sdparm command and its uses
* Tools and utilities for iSCSI
* Awareness of SAN, including relevant protocols (AoE, FCoE)

**Terms and Utilities:**

* hdparm, sdparm
* nvme
* tune2fs
* fstrim
* sysctl
* /dev/hd\*, /dev/sd\*, /dev/nvme\*
* iscsiadm, scsi_id, iscsid and iscsid.conf
* WWID, WWN, LUN numbers

In this course many standards and storage concepts are discussed. Before begging lets classified some topics and then we realize where each tools for.

| Storage Types | Description                                                                                                                                                                                  |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| HDD           | Hard Disk Drive, traditional mechanical hard disks stores data on magnetic platter.                                                                                                          |
| SSD           | Solid state drives (SSDs) are storage devices that contain non-volatile flash memory. they are superior to mechanical hard disk drives in terms of performance, power use, and availability. |
| NVMe          | non-volatile memory expres (NVMe) devices are flash memory chips connected to a system via the PCI-E bus.                                                                                    |

and from another aspect :

| Storage Connectivity Protocols | Description                                                                                                                                                                              |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IDE/ATA                        | Popular Interface, used to connect hard disks and optical drives. speed: 133 MB/s                                                                                                        |
| Serial ATA                     | Serial Version of IDE/ATA typically used for internal connectivity.speed up to 16 Gb/s                                                                                                   |
| SCSI                           | Popular standard for compute-to-storage connectivity.speed up to 640 MB/s                                                                                                                |
| SAS                            | Serial attached SCSI, is a point-to-point serial protocol that provides alternative to SCSI.speed up to 12 Gb/s                                                                          |
| FC                             | Fibre Channel a widely-used-protocol for high-spedd communication to the storage device. Latest version speed up to 32 Gb/s                                                              |
| IP                             | IP is a network protocol that has been traditionally used for compute-to-compute traffic.ISCSI and FCIP protocols are common examples of using IP for compute-to-storage communications. |

Now lets get familiar with some linux tools for specific Hard Drives:

## hdparm

hdparm is a command line program to set and view ATA hard disk drive hardware parameters. First (in 2005) hdparm utility developed by Mrk Lord to test Linux drivers for IDE hard drives. Since then, the program has developed into a valuable tool for diagnosis and tuning of hard drives It can set parameters such as drive caches, sleep mode, power management and DMA settings.As hdparm interacts directly with hardware it can cause data lost and other file system damages.

hdparm has to be run with root privileges, otherwise it will either not be found or the requested actions will not be executed properly.

```
root@server1:~# hdparm -h

hdparm - get/set hard disk parameters - version v9.48, by Mark Lord.

Usage:  hdparm  [options] [device ...]

Options:
 -a   Get/set fs readahead
 -A   Get/set the drive look-ahead flag (0/1)
 -b   Get/set bus state (0 == off, 1 == on, 2 == tristate)
 -B   Set Advanced Power Management setting (1-255)
 -c   Get/set IDE 32-bit IO setting
 -C   Check drive power mode status
 -d   Get/set using_dma flag
 -D   Enable/disable drive defect management
 -E   Set cd/dvd drive speed
 -f   Flush buffer cache for device on exit
 -F   Flush drive write cache
 -g   Display drive geometry
 -h   Display terse usage information
 -H   Read temperature from drive (Hitachi only)
 -i   Display drive identification
 -I   Detailed/current information directly from drive
 -J   Get/set Western DIgital "Idle3" timeout for a WDC "Green" drive (DANGEROUS)
 -k   Get/set keep_settings_over_reset flag (0/1)
 -K   Set drive keep_features_over_reset flag (0/1)
 -L   Set drive doorlock (0/1) (removable harddisks only)
 -m   Get/set multiple sector count
 -M   Get/set acoustic management (0-254, 128: quiet, 254: fast)
 -n   Get/set ignore-write-errors flag (0/1)
 -N   Get/set max visible number of sectors (HPA) (VERY DANGEROUS)
 -p   Set PIO mode on IDE interface chipset (0,1,2,3,4,...)
 -P   Set drive prefetch count
 -q   Change next setting quietly
 -Q   Get/set DMA queue_depth (if supported)
 -r   Get/set device readonly flag (DANGEROUS to set)
 -R   Get/set device write-read-verify flag
 -s   Set power-up in standby flag (0/1) (DANGEROUS)
 -S   Set standby (spindown) timeout
 -t   Perform device read timings
 -T   Perform cache read timings
 -u   Get/set unmaskirq flag (0/1)
 -U   Obsolete
 -v   Use defaults; same as -acdgkmur for IDE drives
 -V   Display program version and exit immediately
 -w   Perform device reset (DANGEROUS)
 -W   Get/set drive write-caching flag (0/1)
 -x   Obsolete
 -X   Set IDE xfer mode (DANGEROUS)
 -y   Put drive in standby mode
 -Y   Put drive to sleep
 -z   Re-read partition table
 -Z   Disable Seagate auto-powersaving mode
 --dco-freeze      Freeze/lock current device configuration until next power cycle
 --dco-identify    Read/dump device configuration identify data
 --dco-restore     Reset device configuration back to factory defaults
 --dco-setmax      Use DCO to set maximum addressable sectors
 --direct          Use O_DIRECT to bypass page cache for timings
 --drq-hsm-error   Crash system with a "stuck DRQ" error (VERY DANGEROUS)
 --fallocate       Create a file without writing data to disk
 --fibmap          Show device extents (and fragmentation) for a file
 --fwdownload            Download firmware file to drive (EXTREMELY DANGEROUS)
 --fwdownload-mode3      Download firmware using min-size segments (EXTREMELY DANGEROUS)
 --fwdownload-mode3-max  Download firmware using max-size segments (EXTREMELY DANGEROUS)
 --fwdownload-mode7      Download firmware using a single segment (EXTREMELY DANGEROUS)
 --fwdownload-modee      Download firmware using mode E (min-size segments) (EXTREMELY DANGEROUS)
 --fwdownload-modee-max  Download firmware using mode E (max-size segments) (EXTREMELY DANGEROUS)
 --idle-immediate  Idle drive immediately
 --idle-unload     Idle immediately and unload heads
 --Istdin          Read identify data from stdin as ASCII hex
 --Istdout         Write identify data to stdout as ASCII hex
 --make-bad-sector Deliberately corrupt a sector directly on the media (VERY DANGEROUS)
 --offset          use with -t, to begin timings at given offset (in GiB) from start of drive
 --prefer-ata12    Use 12-byte (instead of 16-byte) SAT commands when possible
 --read-sector     Read and dump (in hex) a sector directly from the media
 --repair-sector   Alias for the --write-sector option (VERY DANGEROUS)
 --security-help   Display help for ATA security commands
 --trim-sector-ranges        Tell SSD firmware to discard unneeded data sectors: lba:count ..
 --trim-sector-ranges-stdin  Same as above, but reads lba:count pairs from stdin
 --verbose         Display extra diagnostics from some commands
 --write-sector    Repair/overwrite a (possibly bad) sector directly on the media (VERY DANGEROUS)
```

hdparm -I /dev/sda to get information about Hard Disk :

```
root@server1:~# hdparm -I /dev/sda

/dev/sda:

ATA device, with non-removable media
    Model Number:       Samsung SSD 850 EVO 500GB               
    Serial Number:      S2R9NX0H503451W     
    Firmware Revision:  EMT02B6Q
    Transport:          Serial, ATA8-AST, SATA 1.0a, SATA II Extensions, SATA Rev 2.5, SATA Rev 2.6, SATA Rev 3.0
Standards:
    Used: unknown (minor revision code 0x0039) 
    Supported: 9 8 7 6 5 
    Likely used: 9
Configuration:
    Logical        max    current
    cylinders    16383    16383
    heads        16    16
    sectors/track    63    63
    --
    CHS current addressable sectors:   16514064
    LBA    user addressable sectors:  268435455
    LBA48  user addressable sectors:  976773168
    Logical  Sector size:                   512 bytes
    Physical Sector size:                   512 bytes
    Logical Sector-0 offset:                  0 bytes
    device size with M = 1024*1024:      476940 MBytes
    device size with M = 1000*1000:      500107 MBytes (500 GB)
    cache/buffer size  = unknown
    Form Factor: 2.5 inch
    Nominal Media Rotation Rate: Solid State Device
Capabilities:
    LBA, IORDY(can be disabled)
    Queue depth: 32
    Standby timer values: spec'd by Standard, no device specific minimum
    R/W multiple sector transfer: Max = 1    Current = 1
    DMA: mdma0 mdma1 mdma2 udma0 udma1 udma2 udma3 udma4 udma5 *udma6 
         Cycle time: min=120ns recommended=120ns
    PIO: pio0 pio1 pio2 pio3 pio4 
         Cycle time: no flow control=120ns  IORDY flow control=120ns
Commands/features:
    Enabled    Supported:
       *    SMART feature set
            Security Mode feature set
       *    Power Management feature set
       *    Write cache
       *    Look-ahead
       *    Host Protected Area feature set
       *    WRITE_BUFFER command
       *    READ_BUFFER command
       *    NOP cmd
       *    DOWNLOAD_MICROCODE
            SET_MAX security extension
       *    48-bit Address feature set
       *    Device Configuration Overlay feature set
       *    Mandatory FLUSH_CACHE
       *    FLUSH_CACHE_EXT
       *    SMART error logging
       *    SMART self-test
       *    General Purpose Logging feature set
       *    WRITE_{DMA|MULTIPLE}_FUA_EXT
       *    64-bit World wide name
            Write-Read-Verify feature set
       *    WRITE_UNCORRECTABLE_EXT command
       *    {READ,WRITE}_DMA_EXT_GPL commands
       *    Segmented DOWNLOAD_MICROCODE
       *    Gen1 signaling speed (1.5Gb/s)
       *    Gen2 signaling speed (3.0Gb/s)
       *    Gen3 signaling speed (6.0Gb/s)
       *    Native Command Queueing (NCQ)
       *    Phy event counters
       *    READ_LOG_DMA_EXT equivalent to READ_LOG_EXT
       *    DMA Setup Auto-Activate optimization
            Device-initiated interface power management
       *    Asynchronous notification (eg. media change)
       *    Software settings preservation
       *    Device Sleep (DEVSLP)
       *    SMART Command Transport (SCT) feature set
       *    SCT Write Same (AC2)
       *    SCT Error Recovery Control (AC3)
       *    SCT Features Control (AC4)
       *    SCT Data Tables (AC5)
       *    reserved 69[4]
       *    DOWNLOAD MICROCODE DMA command
       *    SET MAX SETPASSWORD/UNLOCK DMA commands
       *    WRITE BUFFER DMA command
       *    READ BUFFER DMA command
       *    Data Set Management TRIM supported (limit 8 blocks)
Security: 
    Master password revision code = 65534
        supported
    not    enabled
    not    locked
        frozen
    not    expired: security count
        supported: enhanced erase
    2min for SECURITY ERASE UNIT. 8min for ENHANCED SECURITY ERASE UNIT. 
Logical Unit WWN Device Identifier: 5002538d40e515dc
    NAA        : 5
    IEEE OUI    : 002538
    Unique ID    : d40e515dc
Device Sleep:
    DEVSLP Exit Timeout (DETO): 50 ms (drive)
    Minimum DEVSLP Assertion Time (MDAT): 30 ms (drive)
Checksum: correct
```

lets check how fast is my ssd drive with -t switch:

```
root@server1:~# hdparm -t /dev/sda

/dev/sda:
 Timing buffered disk reads: 1528 MB in  3.00 seconds = 509.06 MB/sec
```

Wow 509.06 MB/ sec.Not bad :). Some other amazing parameters and features that can be manipulated with hdparm:

| hdparam command switch | Description                                                                                                                                                                                                                                                       |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| hdparm -B 125 /dev/sda | Set the Advanced Power Management, valus <1-255>. While 1-127 permit spin-down, 128-254 no not allow spin-down and 255 disable feature completly                                                                                                                  |
| hdparm -S 240 /dev/sda | Set standby time.specifies how long to wait in idle (with no disk activity) before turning off the motor to save power. 0 can disbale feature,the values from 1 to 240 specify multiples of 5 seconds and values from 241 to 251 specify multiples of 30 minutes. |
| hdparm -d 1 /dev/sda   | set DMA (Direct Memory Access)on   or off,values 0 or 1                                                                                                                                                                                                           |

and many many other options.

## sdparm

scsi version of hdparm. sdparm manupulate scsi specific attributes of hard drive.

## NVMEe

Linux has NVMe driver which is natively included in the kernel since version 3.3. NVMe devices should show up under /dev/nvme\*

## tune2fs

tune2fs command is used by the system administrator to change/modify tunable parameters on ext2, ext3 and ext4 type filesystems. Incomparison to hdparm it manipulates disk drive from File System layer.

```
root@server1:~# tune2fs 
tune2fs 1.42.13 (17-May-2015)
Usage: tune2fs [-c max_mounts_count] [-e errors_behavior] [-g group]
    [-i interval[d|m|w]] [-j] [-J journal_options] [-l]
    [-m reserved_blocks_percent] [-o [^]mount_options[,...]] [-p mmp_update_interval]
    [-r reserved_blocks_count] [-u user] [-C mount_count] [-L volume_label]
    [-M last_mounted_dir] [-O [^]feature[,...]]
    [-Q quota_options]
    [-E extended-option[,...]] [-T last_check_time] [-U UUID]
    [ -I new_inode_size ] device
```

We have already seen tune2fs -l /dev/sda command and we have used -i and -c flags to change fsck check interval. Lets do something more interesting. By default every File System in Linux has some space reserved for root, so no regular user can fill file system up to 100%. As a standard, each File System reserve 5% of total space.Lets change it:

```
root@server1:~# tune2fs -l /dev/sda8 | grep -i reserved 
Reserved block count:     610355
Reserved GDT blocks:      1021
Reserved blocks uid:      0 (user root)
Reserved blocks gid:      0 (group root)

root@server1:~# tune2fs -m 6 /dev/sda8
tune2fs 1.42.13 (17-May-2015)
Setting reserved blocks percentage to 6% (732426 blocks)

root@server1:~# tune2fs -l /dev/sda8 | grep -i reserved 
Reserved block count:     732426
Reserved GDT blocks:      1021
Reserved blocks uid:      0 (user root)
Reserved blocks gid:      0 (group root)
```

When we have very huge partition, ext file system sparse lots of super blocks bakcups in different places of hard disk which consume noticeable space, we can Limit the number of backup superblocks to save space on large filesystems using `tune2fs -s 0 /dev/sda10` command.

note that after any manipulation fsck should be run in order to changes take effect.

## sysctl

We have previously seen sysctl as a tool to view and change kernel parameters at the run time. sysctl deals with /proc directory .

```
root@blackbird:~# sysctl 

Usage:
 sysctl [options] [variable[=value] ...]

Options:
  -a, --all            display all variables
  -A                   alias of -a
  -X                   alias of -a
      --deprecated     include deprecated parameters to listing
  -b, --binary         print value without new line
  -e, --ignore         ignore unknown variables errors
  -N, --names          print variable names without values
  -n, --values         print only values of a variables
  -p, --load[=<file>]  read values from file
  -f                   alias of -p
      --system         read values from all system directories
  -r, --pattern <expression>
                       select setting that match expression
  -q, --quiet          do not echo variable set
  -w, --write          enable writing a value to variable
  -o                   does nothing
  -x                   does nothing
  -d                   alias of -h

 -h, --help     display this help and exit
 -V, --version  output version information and exit

For more details see sysctl(8).
```

We used sysctl -a to list all parameters ,and as an example we know sysctl -w net.ipv4.ipforward=1 is the same as echo "1" > /proc/sys/net/ipv4/ip_forward. Then for marking changes persistent we learned that sysctl has a configuration file which is /etc/sysctl.conf and kernel parameters are defined there.

## What is LUN?

a logical unit number, or LUN, is a number used to identify a logical unit, which is a device addressed by the SCSI protocol or Storage Area Network protocols which encapsulate SCSI, such as Fibre Channel or iSCSI.

## iscsi "EYE-skuzzy"

Befor talking about iscsi, lets talk about scsi to understand it better, Whats scsi? SCSI "skuzzy" ( Small Computer System Interface), is a set of American National Standards Institute (ANSI) standard electronic interfaces that allow personal computers (PCs) to communicate with peripheral hardware such as disk drives, tape drives, CD-ROM drives, printers and scanners faster and more flexibly than previous parallel data transfer interfaces . But what if we could do it remotely?

iSCSI (Internet Small Computer System Interface), works on top of the TCP (Transport Control Protocol) and allows the SCSI command to be sent end-to-end over LANs and WANs or the Internet.

How dose it work? iSCSI works by transporting block-level data between an iSCSI initiator on a computer(as client) and an iSCSI target on a storage device(as server). The iSCSI protocol encapsulates SCSI commands and assembles the data in packets for the TCP/IP layer. Packets are sent over the network using a point-to-point connection. When packets arrived, the iSCSI protocol disassembles the packets, take out SCSI commands so the operating system will see the storage as a local SCSI device that can be formatted as usual.

![](.gitbook/assets/iscsi.jpg)

### iqn,EUI

Each iscsi target or iscsi initirator is called iscsi node.All iSCSI nodes are identified by an iSCSI name. An iSCSI name is not IP address or DNS name of that host. Names enable iSCSI storage resources to be managed regardless of other kind of addresses, Also it is used in authentication of targets to initiators and initiators to targets.

iSCSI addresses can be one of two types:

1. iSCSI Qualified Name (iQN) 
2. IEEE Naming convention (EUI)

iQN format ‐ iqn.yyyy‐mm.com.xyz.aabbccddeeffgghh :

* iqn ‐ Naming convention identifier
* yyyy‐nn ‐ Point in time when the .com domain was registered
* com.xyz ‐ Domain of the node backwards
*   aabbccddeeffgghh ‐ Device identifier (can be a WWN, the system name, or any other vendorimplemented

     standard)

EUI format ‐ eui.64‐bit WWN:

* eui ‐ Naming prefix
* 64‐bit WWN ‐ FC WWN of the host

Okey enough introduction lets start.To keep it simple we use CentOS 7 server (target) with additional 10 gig Disk Drive and then get Our Ubuntu client (initiator) connected to it. First We need to add EPEL repository:

```
###Server Side , CentOS with 10 gig Disk, iscsitarget
[root@server1 ~]# yum install epel-release
```

and now lets install iscsi target:

```
[root@server1 ~]# yum search scsi target
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.us.leaseweb.net
 * epel: mirrors.lug.mtu.edu
 * extras: mirror.us.leaseweb.net
 * updates: mirror.us.leaseweb.net
============================== N/S matched: scsi, target ==============================
netbsd-iscsi.x86_64 : User-space implementation of iSCSI target from NetBSD project
scsi-target-utils.x86_64 : The SCSI target daemon and utility programs
scsi-target-utils-gluster.x86_64 : Support for the Gluster backstore to
                                 : scsi-target-utils
python-rtslib.noarch : API for Linux kernel LIO SCSI target

  Full name and summary matches only, use "search all" for everything.

[root@server1 ~]# yum install -y scsi-target-utils
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.steadfast.net
 * epel: fedora-epel.mirror.iweb.com
 * extras: mirror.steadfast.net
 * updates: mirror.steadfast.net
Resolving Dependencies
--> Running transaction check
---> Package scsi-target-utils.x86_64 0:1.0.55-4.el7 will be installed
--> Processing Dependency: perl(Config::General) for package: scsi-target-utils-1.0.55-4.el7.x86_64
--> Processing Dependency: sg3_utils for package: scsi-target-utils-1.0.55-4.el7.x86_64
--> Running transaction check
---> Package perl-Config-General.noarch 0:2.61-1.el7 will be installed
---> Package sg3_utils.x86_64 0:1.37-12.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=======================================================================================
 Package                     Arch           Version                 Repository    Size
=======================================================================================
Installing:
 scsi-target-utils           x86_64         1.0.55-4.el7            epel         209 k
Installing for dependencies:
 perl-Config-General         noarch         2.61-1.el7              epel          75 k
 sg3_utils                   x86_64         1.37-12.el7             base         644 k

Transaction Summary
=======================================================================================
Install  1 Package (+2 Dependent packages)

Total download size: 927 k
Installed size: 2.3 M
Downloading packages:
(1/3): perl-Config-General-2.61-1.el7.noarch.rpm                |  75 kB  00:00:03     
(2/3): scsi-target-utils-1.0.55-4.el7.x86_64.rpm                | 209 kB  00:00:05     
(3/3): sg3_utils-1.37-12.el7.x86_64.rpm                         | 644 kB  00:00:06     
---------------------------------------------------------------------------------------
Total                                                     131 kB/s | 927 kB  00:07     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : perl-Config-General-2.61-1.el7.noarch                               1/3 
  Installing : sg3_utils-1.37-12.el7.x86_64                                        2/3 
  Installing : scsi-target-utils-1.0.55-4.el7.x86_64                               3/3 
  Verifying  : sg3_utils-1.37-12.el7.x86_64                                        1/3 
  Verifying  : perl-Config-General-2.61-1.el7.noarch                               2/3 
  Verifying  : scsi-target-utils-1.0.55-4.el7.x86_64                               3/3 

Installed:
  scsi-target-utils.x86_64 0:1.0.55-4.el7                                              

Dependency Installed:
  perl-Config-General.noarch 0:2.61-1.el7        sg3_utils.x86_64 0:1.37-12.el7       

Complete!
[root@server1 ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb               8:16   0   10G  0 disk 
sr0              11:0    1 1024M  0 rom  
sda               8:0    0   50G  0 disk 
├─sda2            8:2    0   49G  0 part 
│ ├─centos-swap 253:1    0  3.9G  0 lvm  [SWAP]
│ └─centos-root 253:0    0 45.1G  0 lvm  /
└─sda1            8:1    0    1G  0 part /boot
```

Now lets configure target vi /etc/tgt/target.conf:

```
# This is a sample config file for tgt-admin.
#
# The "#" symbol disables the processing of a line.

# Set the driver. If not specified, defaults to "iscsi".
default-driver iscsi

# Set iSNS parameters, if needed
#iSNSServerIP 192.168.111.222
#iSNSServerPort 3205
#iSNSAccessControl On
#iSNS On

# Continue if tgtadm exits with non-zero code (equivalent of
# --ignore-errors command line option)
#ignore-errors yes
###############################
#Our added configuration:
<target 192.168.10.134:target00>
    backing-store /dev/sdb
    initiator-address 192.168.10.151
    incominguser myuser mypassword
</target>
```

lets restrat service:

```
[root@server1 ~]# systemctl restrat tgtd.service 
Unknown operation 'restrat'.
[root@server1 ~]# systemctl restart tgtd.service 
[root@server1 ~]# systemctl status tgtd.service 
● tgtd.service - tgtd iSCSI target daemon
   Loaded: loaded (/usr/lib/systemd/system/tgtd.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2018-01-09 03:56:23 EST; 7s ago
  Process: 5601 ExecStop=/usr/sbin/tgtadm --op delete --mode system (code=exited, status=0/SUCCESS)
  Process: 5592 ExecStop=/usr/sbin/tgt-admin --update ALL -c /dev/null (code=exited, status=0/SUCCESS)
  Process: 5589 ExecStop=/usr/sbin/tgtadm --op update --mode sys --name State -v offline (code=exited, status=0/SUCCESS)
  Process: 5644 ExecStartPost=/usr/sbin/tgtadm --op update --mode sys --name State -v ready (code=exited, status=0/SUCCESS)
  Process: 5614 ExecStartPost=/usr/sbin/tgt-admin -e -c $TGTD_CONFIG (code=exited, status=0/SUCCESS)
  Process: 5612 ExecStartPost=/usr/sbin/tgtadm --op update --mode sys --name State -v offline (code=exited, status=0/SUCCESS)
  Process: 5610 ExecStartPost=/bin/sleep 5 (code=exited, status=0/SUCCESS)
 Main PID: 5609 (tgtd)
   CGroup: /system.slice/tgtd.service
           └─5609 /usr/sbin/tgtd -f

Jan 09 03:56:17 server1 systemd[1]: Starting tgtd iSCSI target daemon...
Jan 09 03:56:17 server1 tgtd[5609]: tgtd: iser_ib_init(3436) Failed to initialize...es?
Jan 09 03:56:17 server1 tgtd[5609]: tgtd: work_timer_start(146) use timer_fd base...ler
Jan 09 03:56:17 server1 tgtd[5609]: tgtd: bs_init_signalfd(267) could not open ba...ore
Jan 09 03:56:17 server1 tgtd[5609]: tgtd: bs_init(386) use signalfd notification
Jan 09 03:56:23 server1 tgtd[5609]: tgtd: device_mgmt(246) sz:14 params:path=/dev/sdb
Jan 09 03:56:23 server1 tgtd[5609]: tgtd: bs_thread_open(408) 16
Jan 09 03:56:23 server1 systemd[1]: Started tgtd iSCSI target daemon.
Hint: Some lines were ellipsized, use -l to show in full.
```

use tgtadmin to scan what is result:

```
[root@server1 ~]# tgtadm --mode target --op show
Target 1: 192.168.10.134:target00
    System information:
        Driver: iscsi
        State: ready
    I_T nexus information:
    LUN information:
        LUN: 0
            Type: controller
            SCSI ID: IET     00010000
            SCSI SN: beaf10
            Size: 0 MB, Block size: 1
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: null
            Backing store path: None
            Backing store flags: 
        LUN: 1
            Type: disk
            SCSI ID: IET     00010001
            SCSI SN: beaf11
            Size: 10737 MB, Block size: 512
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: rdwr
            Backing store path: /dev/sdb
            Backing store flags: 
    Account information:
        myuser
    ACL information:
        192.168.10.151
```

Do not forget to `sysemctl enable tgt.service` to keep iscsi target service Active even after reboot. Before testing we have to make tcp port 3260 open, CentOS 7 use firewalld on top of iptables so we use firewall-cmd to manipulate ip table:

```
[root@server1 ~]# firewall-cmd --add-port=3260/tcp --zone=public --permanent
success
[root@server1 ~]# firewall-cmd --reload
success
 [root@server1 ~]# firewall-cmd --zone=public --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: ssh dhcpv6-client
  ports: 3260/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:
```

okey its time to configure our client as iscsi initiator , we use ubuntu 16 here:

```
###Client side, ubuntu 16.04, initiator
root@server2:~# apt install open-iscsi
```

lets configure iscsi initiator `vi /etc/iscsi/iscsid.conf` :

```
#
# Open-iSCSI default configuration.
# Could be located at /etc/iscsi/iscsid.conf or ~/.iscsid.conf
#
# Note: To set any of these values for a specific node/session run
# the iscsiadm --mode node --op command for the value. See the README
# and man page for iscsiadm for details on the --op command.
#

######################
# iscsid daemon config
######################
# If you want iscsid to start the first time a iscsi tool
# needs to access it, instead of starting it when the init
# scripts run, set the iscsid startup command here. This
# should normally only need to be done by distro package
# maintainers.
#
# Default for Fedora and RHEL. (uncomment to activate).
# iscsid.startup = /etc/rc.d/init.d/iscsid force-start
# 
# Default for upstream open-iscsi scripts (uncomment to activate).
iscsid.startup = /usr/sbin/iscsid


#############################
# NIC/HBA and driver settings
#############################
# open-iscsi can create a session and bind it to a NIC/HBA.
# To set this up see the example iface config file.

#*****************
# Startup settings
#*****************

# To request that the iscsi initd scripts startup a session set to "automatic".
# node.startup = automatic
#
# To manually startup the session set to "manual". The default is manual.
node.startup = manual

# For "automatic" startup nodes, setting this to "Yes" will try logins on each
# available iface until one succeeds, and then stop.  The default "No" will try
# logins on all availble ifaces simultaneously.
node.leading_login = No

# *************
# CHAP Settings
# *************

# To enable CHAP authentication set node.session.auth.authmethod
# to CHAP. The default is None.
#node.session.auth.authmethod = CHAP

# To set a CHAP username and password for initiator
# authentication by the target(s), uncomment the following lines:
#node.session.auth.username = username
#node.session.auth.password = password

# To set a CHAP username and password for target(s)
# authentication by the initiator, uncomment the following lines:
#node.session.auth.username_in = username_in
#node.session.auth.password_in = password_in

# To enable CHAP authentication for a discovery session to the target
# set discovery.sendtargets.auth.authmethod to CHAP. The default is None.
#discovery.sendtargets.auth.authmethod = CHAP

# To set a discovery session CHAP username and password for the initiator
# authentication by the target(s), uncomment the following lines:
#discovery.sendtargets.auth.username = username
#discovery.sendtargets.auth.password = password

# To set a discovery session CHAP username and password for target(s)
# authentication by the initiator, uncomment the following lines:
#discovery.sendtargets.auth.username_in = username_in
#discovery.sendtargets.auth.password_in = password_in

# ********
# Timeouts
# ********
#
# See the iSCSI REAME's Advanced Configuration section for tips
# on setting timeouts when using multipath or doing root over iSCSI.
#
# To specify the length of time to wait for session re-establishment
# before failing SCSI commands back to the application when running
# the Linux SCSI Layer error handler, edit the line.
# The value is in seconds and the default is 120 seconds.
# Special values:
# - If the value is 0, IO will be failed immediately.
# - If the value is less than 0, IO will remain queued until the session
# is logged back in, or until the user runs the logout command.
node.session.timeo.replacement_timeout = 120

# To specify the time to wait for login to complete, edit the line.
# The value is in seconds and the default is 15 seconds.
node.conn[0].timeo.login_timeout = 15

# To specify the time to wait for logout to complete, edit the line.
# The value is in seconds and the default is 15 seconds.
node.conn[0].timeo.logout_timeout = 15

# Time interval to wait for on connection before sending a ping.
node.conn[0].timeo.noop_out_interval = 5

# To specify the time to wait for a Nop-out response before failing
# the connection, edit this line. Failing the connection will
# cause IO to be failed back to the SCSI layer. If using dm-multipath
# this will cause the IO to be failed to the multipath layer.
node.conn[0].timeo.noop_out_timeout = 5

# To specify the time to wait for abort response before
# failing the operation and trying a logical unit reset edit the line.
# The value is in seconds and the default is 15 seconds.
node.session.err_timeo.abort_timeout = 15

# To specify the time to wait for a logical unit response
# before failing the operation and trying session re-establishment
# edit the line.
# The value is in seconds and the default is 30 seconds.
node.session.err_timeo.lu_reset_timeout = 30

# To specify the time to wait for a target response
# before failing the operation and trying session re-establishment
# edit the line.
# The value is in seconds and the default is 30 seconds.
node.session.err_timeo.tgt_reset_timeout = 30


#******
# Retry
#******

# To specify the number of times iscsid should retry a login
# if the login attempt fails due to the node.conn[0].timeo.login_timeout
# expiring modify the following line. Note that if the login fails
# quickly (before node.conn[0].timeo.login_timeout fires) because the network
# layer or the target returns an error, iscsid may retry the login more than
# node.session.initial_login_retry_max times.
#
# This retry count along with node.conn[0].timeo.login_timeout
# determines the maximum amount of time iscsid will try to
# establish the initial login. node.session.initial_login_retry_max is
# multiplied by the node.conn[0].timeo.login_timeout to determine the
# maximum amount.
#
# The default node.session.initial_login_retry_max is 8 and
# node.conn[0].timeo.login_timeout is 15 so we have:
#
# node.conn[0].timeo.login_timeout * node.session.initial_login_retry_max =
#                                120 seconds
#
# Valid values are any integer value. This only
# affects the initial login. Setting it to a high value can slow
# down the iscsi service startup. Setting it to a low value can
# cause a session to not get logged into, if there are distuptions
# during startup or if the network is not ready at that time.
node.session.initial_login_retry_max = 8

################################
# session and device queue depth
################################

# To control how many commands the session will queue set
# node.session.cmds_max to an integer between 2 and 2048 that is also
# a power of 2. The default is 128.
node.session.cmds_max = 128

# To control the device's queue depth set node.session.queue_depth
# to a value between 1 and 1024. The default is 32.
node.session.queue_depth = 32

##################################
# MISC SYSTEM PERFORMANCE SETTINGS
##################################

# For software iscsi (iscsi_tcp) and iser (ib_iser) each session
# has a thread used to transmit or queue data to the hardware. For
# cxgb3i you will get a thread per host.
#
# Setting the thread's priority to a lower value can lead to higher throughput
# and lower latencies. The lowest value is -20. Setting the priority to
# a higher value, can lead to reduced IO performance, but if you are seeing
# the iscsi or scsi threads dominate the use of the CPU then you may want
# to set this value higher.
#
# Note: For cxgb3i you must set all sessions to the same value, or the
# behavior is not defined.
#
# The default value is -20. The setting must be between -20 and 20.
node.session.xmit_thread_priority = -20


#***************
# iSCSI settings
#***************

# To enable R2T flow control (i.e., the initiator must wait for an R2T
# command before sending any data), uncomment the following line:
#
#node.session.iscsi.InitialR2T = Yes
#
# To disable R2T flow control (i.e., the initiator has an implied
# initial R2T of "FirstBurstLength" at offset 0), uncomment the following line:
#
# The defaults is No.
node.session.iscsi.InitialR2T = No

#
# To disable immediate data (i.e., the initiator does not send
# unsolicited data with the iSCSI command PDU), uncomment the following line:
#
#node.session.iscsi.ImmediateData = No
#
# To enable immediate data (i.e., the initiator sends unsolicited data
# with the iSCSI command packet), uncomment the following line:
#
# The default is Yes
node.session.iscsi.ImmediateData = Yes

# To specify the maximum number of unsolicited data bytes the initiator
# can send in an iSCSI PDU to a target, edit the following line.
#
# The value is the number of bytes in the range of 512 to (2^24-1) and
# the default is 262144
node.session.iscsi.FirstBurstLength = 262144

# To specify the maximum SCSI payload that the initiator will negotiate
# with the target for, edit the following line.
#
# The value is the number of bytes in the range of 512 to (2^24-1) and
# the defauls it 16776192
node.session.iscsi.MaxBurstLength = 16776192

# To specify the maximum number of data bytes the initiator can receive
# in an iSCSI PDU from a target, edit the following line.
#
# The value is the number of bytes in the range of 512 to (2^24-1) and
# the default is 262144
node.conn[0].iscsi.MaxRecvDataSegmentLength = 262144

# To specify the maximum number of data bytes the initiator will send
# in an iSCSI PDU to the target, edit the following line.
#
# The value is the number of bytes in the range of 512 to (2^24-1).
# Zero is a special case. If set to zero, the initiator will use
# the target's MaxRecvDataSegmentLength for the MaxXmitDataSegmentLength.
# The default is 0.
node.conn[0].iscsi.MaxXmitDataSegmentLength = 0

# To specify the maximum number of data bytes the initiator can receive
# in an iSCSI PDU from a target during a discovery session, edit the
# following line.
#
# The value is the number of bytes in the range of 512 to (2^24-1) and
# the default is 32768
# 
discovery.sendtargets.iscsi.MaxRecvDataSegmentLength = 32768

# To allow the targets to control the setting of the digest checking,
# with the initiator requesting a preference of enabling the checking, uncomment# one or both of the following lines:
#node.conn[0].iscsi.HeaderDigest = CRC32C,None
#node.conn[0].iscsi.DataDigest = CRC32C,None
#
# To allow the targets to control the setting of the digest checking,
# with the initiator requesting a preference of disabling the checking,
# uncomment one or both of the following lines:
#node.conn[0].iscsi.HeaderDigest = None,CRC32C
#node.conn[0].iscsi.DataDigest = None,CRC32C
#
# To enable CRC32C digest checking for the header and/or data part of
# iSCSI PDUs, uncomment one or both of the following lines:
#node.conn[0].iscsi.HeaderDigest = CRC32C
#node.conn[0].iscsi.DataDigest = CRC32C
#
# To disable digest checking for the header and/or data part of
# iSCSI PDUs, uncomment one or both of the following lines:
#node.conn[0].iscsi.HeaderDigest = None
#node.conn[0].iscsi.DataDigest = None
#
# The default is to never use DataDigests or HeaderDigests.
#

# For multipath configurations, you may want more than one session to be
# created on each iface record.  If node.session.nr_sessions is greater
# than 1, performing a 'login' for that node will ensure that the
# appropriate number of sessions is created.
node.session.nr_sessions = 1

#************
# Workarounds
#************

# Some targets like IET prefer after an initiator has sent a task
# management function like an ABORT TASK or LOGICAL UNIT RESET, that
# it does not respond to PDUs like R2Ts. To enable this behavior uncomment
# the following line (The default behavior is Yes):
node.session.iscsi.FastAbort = Yes

# Some targets like Equalogic prefer that after an initiator has sent
# a task management function like an ABORT TASK or LOGICAL UNIT RESET, that
# it continue to respond to R2Ts. To enable this uncomment this line
# node.session.iscsi.FastAbort = No
######################
### Copy and uncoment and modify:


node.session.auth.username = myuser 
node.session.auth.password = mypassword
```

and lets start the service:

```
root@server2:~# systemctl start iscsid.service
```

and check with iscsiadm tool:

```
root@server2:~# iscsiadm --mode discovery -t sendtargets --portal 192.168.10.134
192.168.10.134:3260,1 192.168.10.134:target00
```

see current hard disk of our client and lets add new hard disk using iscsiadm:

```
root@server2:~# ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sda2  /dev/sda5

root@server2:~# iscsiadm --mode node --targetname 192.168.10.134:target00 --portal 192.168.10.134 --login
Logging in to [iface: default, target: 192.168.10.134:target00, portal: 192.168.10.134,3260] (multiple)
Login to [iface: default, target: 192.168.10.134:target00, portal: 192.168.10.134,3260] successful.

root@server2:~# ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sda2  /dev/sda5    /dev/sdb
```

okey /dev/sdb is added, if you have noticed. to confirm:

```
root@server2:~# iscsiadm -m session -o show
tcp: [1] 192.168.10.134:3260,1 192.168.10.134:target00 (non-flash)
root@server2:~# cat /proc/partitions 
major minor  #blocks  name
   8        0   52428800 sda
   8        1   51380224 sda1
   8        2          1 sda2
   8        5    1045504 sda5
  11        0    1048575 sr0
   8       32   10485760 sdb
```

and if we reboot the system BoOoMM! every thing is gone! to make it persistent change `node.startup = manual`\
to automatic inside /etc/iscsi/iscsi.conf file. and copy iscsiadm script to /etc/rc.local and enable it if required:

```
root@server2:~# echo "iscsiadm --mode node --targetname 192.168.10.134:target00 --portal 192.168.10.134 --login" >> /etc/rc.local
root@server2:~# systemctl enable rc-local.service 
root@server2:~# chmod 750 /etc/rc.local
```

and we are done. Now we can format mount the partition or we can add it to fstab using `_netdev` option.
