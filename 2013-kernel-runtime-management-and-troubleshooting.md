# 201.3. Kernel runtime management and troubleshooting

## **201.3 Kernel runtime management and troubleshooting**

**Weight:** 4

**Description:** Candidates should be able to manage and/or query a 2.6.x, 3.x or 4.x kernel and its loadable modules. Candidates should be able to identify and correct common boot and run time issues. Candidates should understand device detection and management using udev. This objective includes troubleshooting udev rules.

**Key Knowledge Areas:**

* Use command-line utilities to get information about the currently running kernel and kernel modules
* Manually load and unload kernel modules
* Determine when modules can be unloaded
* Determine what parameters a module accepts
* Configure the system to load modules by names other than their file name.
* /proc filesystem
* Content of /, /boot/ , and /lib/modules/
* Tools and utilities to analyze information about the available hardware
* udev rules

**Terms and Utilities:**

* /lib/modules/kernel-version/modules.dep
* module configuration files in /etc/
* /proc/sys/kernel/
* /sbin/depmod
* /sbin/rmmod
* /sbin/modinfo
* /bin/dmesg
* /sbin/lspci
* /usr/bin/lsdev
* /sbin/lsmod
* /sbin/modprobe
* /sbin/insmod
* /bin/uname
* /usr/bin/lsusb
* /etc/sysctl.conf, /etc/sysctl.d/
* /sbin/sysctl
* udevmonitor
* udevadm monitor
* /etc/udev/

## Kernel Modules

Linux Kernel is modular since version 2.0. Being modular make simplicity and efficiency, the code which is not needed wouldn't be loaded. Kernel Modules have different conditions. A module might be compiled, or not. If it has already compiled it might be loaded or not. So Loading an unload but compiled kernel is easy, otherwise required kernel manipulation.We use CentOS7 for demonstartion:

### lsmod

To list Kernel Loaded modules use lsmod

```text
root@server1:~# lsmod
Module                  Size  Used by
bnep                   20480  2
nfnetlink_queue        24576  0
nfnetlink_log          20480  0
nfnetlink              16384  2 nfnetlink_log,nfnetlink_queue
bluetooth             557056  7 bnep
vmw_vsock_vmci_transport    28672  2
vsock                  36864  3 vmw_vsock_vmci_transport
vmw_balloon            20480  0
crct10dif_pclmul       16384  0
crc32_pclmul           16384  0
ghash_clmulni_intel    16384  0
snd_ens1371            28672  4
snd_ac97_codec        131072  1 snd_ens1371
gameport               16384  1 snd_ens1371
ac97_bus               16384  1 snd_ac97_codec
pcbc                   16384  0
snd_pcm               102400  2 snd_ac97_codec,snd_ens1371
aesni_intel           167936  0
aes_x86_64             20480  1 aesni_intel
snd_seq_midi           16384  0
snd_seq_midi_event     16384  1 snd_seq_midi
crypto_simd            16384  1 aesni_intel
snd_rawmidi            32768  2 snd_seq_midi,snd_ens1371
snd_seq                65536  2 snd_seq_midi_event,snd_seq_midi
glue_helper            16384  1 aesni_intel
cryptd                 24576  3 crypto_simd,ghash_clmulni_intel,aesni_intel
intel_rapl_perf        16384  0
snd_seq_device         16384  3 snd_seq,snd_rawmidi,snd_seq_midi
snd_timer              32768  2 snd_seq,snd_pcm
input_leds             16384  0
joydev                 20480  0
serio_raw              16384  0
i2c_piix4              24576  0
shpchp                 36864  0
snd                    77824  15 snd_seq,snd_ac97_codec,snd_timer,snd_rawmidi,snd_ens1371,snd_seq_device,snd_pcm
soundcore              16384  1 snd
vmw_vmci               69632  2 vmw_balloon,vmw_vsock_vmci_transport
nfit                   49152  0
mac_hid                16384  0
parport_pc             32768  0
ppdev                  20480  0
lp                     20480  0
parport                49152  3 lp,parport_pc,ppdev
autofs4                40960  2
vmw_pvscsi             24576  0
vmxnet3                61440  0
hid_generic            16384  0
usbhid                 53248  0
hid                   118784  2 hid_generic,usbhid
vmwgfx                241664  3
ttm                    98304  1 vmwgfx
drm_kms_helper        151552  1 vmwgfx
mptspi                 24576  2
syscopyarea            16384  1 drm_kms_helper
mptscsih               40960  1 mptspi
sysfillrect            16384  1 drm_kms_helper
psmouse               139264  0
sysimgblt              16384  1 drm_kms_helper
mptbase               102400  2 mptscsih,mptspi
fb_sys_fops            16384  1 drm_kms_helper
ahci                   36864  0
e1000                 143360  0
drm                   352256  6 vmwgfx,ttm,drm_kms_helper
libahci                32768  1 ahci
scsi_transport_spi     32768  1 mptspi
pata_acpi              16384  0
fjes                   77824  0
```

lets try to load and unloaded module " jfs " that old journaling file system. for loading an unloaded but compiled module there are two solutions. An old solution is insmod , lets try it first

```text
root@server1:~# insmod jfs
insmod: ERROR: could not load module jfs: No such file or directory
```

But it doesn't work :\(\( so let get more information about "jfs" module with modinfo:

```text
root@server1:~# modinfo jfs
filename:       /lib/modules/4.10.0-40-generic/kernel/fs/jfs/jfs.ko
alias:          fs-jfs
license:        GPL
author:         Steve Best/Dave Kleikamp/Barry Arndt, IBM
description:    The Journaled Filesystem (JFS)
srcversion:     3249CD436BA490FF745560A
depends:        
intree:         Y
vermagic:       4.10.0-40-generic SMP mod_unload 
parm:           nTxBlock:Number of transaction blocks (max:65536) (int)
parm:           nTxLock:Number of transaction locks (max:65536) (int)
parm:           commit_threads:Number of commit threads (int)
```

modinfo shows a bunch of information about a module. Like its filename, dependencies and possible parameters.Till now we have realize that "jfs" module has been compiled and its available but it is not loaded. inorder to load a module with insmod we have to specify the full patch of module:

```text
root@server1:~# insmod /lib/modules/4.10.0-40-generic/kernel/fs/jfs/jfs.ko
root@server1:~# lsmod | grep jfs
jfs                   184320  0
```

Haha its loaded now, Lets remove it by using rmmod:

```text
root@server1:~# lsmod | grep jfs
jfs                   184320  0
root@server1:~# 
root@server1:~# rmmod jfs
root@server1:~# lsmod | grep jfs
```

But as you see using insmod is not that much easy, insmod has another problem and that is dependencies. "jfs" module has no dependency so we were able to load it very easily, if a module has some dependencies, insmod dosn't load those required dependencies automatically. So it would be administrator task. All these shortages cause we come to this conclusion that insmod is not perfect and we need a modern tool, which automatically know the patch and load required dependencies, that is modprobe.

```text
root@server1:~# lsmod | grep lp
lp                     20480  0
glue_helper            16384  1 aesni_intel
parport                49152  1 lp
drm_kms_helper        151552  1 vmwgfx
syscopyarea            16384  1 drm_kms_helper
sysfillrect            16384  1 drm_kms_helper
sysimgblt              16384  1 drm_kms_helper
fb_sys_fops            16384  1 drm_kms_helper
drm                   352256  6 vmwgfx,ttm,drm_kms_helper
root@server1:~# modinfo lp
filename:       /lib/modules/4.10.0-40-generic/kernel/drivers/char/lp.ko
license:        GPL
alias:          char-major-6-*
srcversion:     5273B3FB623D57CACC37C09
depends:        parport
intree:         Y
vermagic:       4.10.0-40-generic SMP mod_unload 
parm:           parport:array of charp
parm:           reset:bool
root@server1:~# modinfo parport
filename:       /lib/modules/4.10.0-40-generic/kernel/drivers/parport/parport.ko
license:        GPL
srcversion:     CED357203546F0957229054
depends:        
intree:         Y
vermagic:       4.10.0-40-generic SMP mod_unload 
root@server1:~# rmmod lp
root@server1:~# rmmod parport
root@server1:~# insmod /lib/modules/4.10.0-40-generic/kernel/drivers/char/lp.ko
insmod: ERROR: could not insert module /lib/modules/4.10.0-40-generic/kernel/drivers/char/lp.ko: Unknown symbol in module
root@server1:~# modprobe lp
root@server1:~# lsmod | grep lp
lp                     20480  0
parport                49152  1 lp
glue_helper            16384  1 aesni_intel
drm_kms_helper        151552  1 vmwgfx
syscopyarea            16384  1 drm_kms_helper
sysfillrect            16384  1 drm_kms_helper
sysimgblt              16384  1 drm_kms_helper
fb_sys_fops            16384  1 drm_kms_helper
drm                   352256  6 vmwgfx,ttm,drm_kms_helper
```

also for removing use modprobe -r lp. But how modprobe realize module dependencies and act such a smart tool? modules.dep

```text
root@server1:~# cd /lib/modules/
root@server1:/lib/modules# ls
4.10.0-28-generic  4.10.0-40-generic
root@server1:/lib/modules# cd 4.10.0-40-generic/
root@server1:/lib/modules/4.10.0-40-generic# ls
build          modules.alias.bin    modules.dep.bin  modules.symbols
initrd         modules.builtin      modules.devname  modules.symbols.bin
kernel         modules.builtin.bin  modules.order    vdso
modules.alias  modules.dep          modules.softdep
root@server1:/lib/modules/4.10.0-40-generic# depmod -a
```

we can use depmod -a to create and update modules.dep with all new modules and dependencies. Now that we have learned alot of tools, let play game with cdrom parameters \[in ubunto cdrom is not a module, so use other distro\]:

```text
[root@server1 ~]# lsmod | grep cdrom
cdrom                  42556  1 sr_mod
[root@server1 ~]# modinfo cdrom
filename:       /lib/modules/3.10.0-693.el7.x86_64/kernel/drivers/cdrom/cdrom.ko.xz
license:        GPL
rhelversion:    7.4
srcversion:     BE3BD0D17D080229D55B173
depends:        
intree:         Y
vermagic:       3.10.0-693.el7.x86_64 SMP mod_unload modversions 
signer:         CentOS Linux kernel signing key
sig_key:        DA:18:7D:CA:7D:BE:53:AB:05:BD:13:BD:0C:4E:21:F4:22:B6:A4:9C
sig_hashalgo:   sha256
parm:           debug:bool
parm:           autoclose:bool
parm:           autoeject:bool
parm:           lockdoor:bool
parm:           check_media_type:bool
parm:           mrw_format_restart:bool
[root@server1 ~]# modprobe cdrom lookdoor=1
```

We as users like modules to load automatically, mean while parameters should be loaded automatically, to make parameters persistence create a file in /etc/modprobe.d :

```text
 root@server1:~# cd /etc/modprobe.d/
root@server1:/etc/modprobe.d# ls
alsa-base.conf           blacklist-framebuffer.conf   blacklist-watchdog.conf
blacklist-ath_pci.conf   blacklist-modem.conf         fbdev-blacklist.conf
blacklist.conf           blacklist-oss.conf           iwlwifi.conf
blacklist-firewire.conf  blacklist-rare-network.conf  mlx4.conf
root@server1:/etc/modprobe.d# vi cdroom-lookdoor.conf
```

and insert:

```text
options cdroom lookdoor=1
```

If you have noticed there is blacklist.conf file in /etc/modfprobe.d , if you don't wana let a specific module be loaded use it:

```text
# This file lists those modules which we don't want to be loaded by
# alias expansion, usually so some other driver will be loaded for the
# device instead.

# evbug is a debug tool that should be loaded explicitly
blacklist evbug

# these drivers are very simple, the HID drivers are usually preferred
blacklist usbmouse
blacklist usbkbd

# replaced by e100
blacklist eepro100

# replaced by tulip
blacklist de4x5

# causes no end of confusion by creating unexpected network interfaces
blacklist eth1394

# snd_intel8x0m can interfere with snd_intel8x0, doesn't seem to support much
# hardware on its own (Ubuntu bug #2011, #6810)
blacklist snd_intel8x0m

# Conflicts with dvb driver (which is better for handling this device)
blacklist snd_aw2

# causes failure to suspend on HP compaq nc6000 (Ubuntu: #10306)
blacklist i2c_i801

# replaced by p54pci
blacklist prism54

# replaced by b43 and ssb.
blacklist bcm43xx

# most apps now use garmin usb driver directly (Ubuntu: #114565)
blacklist garmin_gps

# replaced by asus-laptop (Ubuntu: #184721)
blacklist asus_acpi

# low-quality, just noise when being used for sound playback, causes
# hangs at desktop session start (Ubuntu: #246969)
blacklist snd_pcsp

# ugly and loud noise, getting on everyone's nerves; this should be done by a
# nice pulseaudio bing (Ubuntu: #77010)
blacklist pcspkr

# EDAC driver for amd76x clashes with the agp driver preventing the aperture
# from being initialised (Ubuntu: #297750). Blacklist so that the driver
# continues to build and is installable for the few cases where its
# really needed.
blacklist amd76x_edac
```

go to /sys/module/&lt;module name&gt; directory to see current running environment parameters.

### sysctl

We discussed that by manipulating /proc/sys/kernel we can change some current running kernel parameters. But as we said these settings are not permanent and they all gone after reboot. sysctl lets us make permanent settings . sysctl is loaded from /sbin/sysctl and read setting from /etc/sysctl.conf. As user we can change settings inside sysctl.conf file or make our custom file with desired parameters in /etc/sysctl.d/ directory. sysctl has a command interface and let us to see and write paramets trough /proc/sys/kernel/.

sysctl -a to show all tuneables.

```text
root@server1:~# sysctl -a | wc
sysctl: reading key "net.ipv6.conf.all.stable_secret"
sysctl: reading key "net.ipv6.conf.default.stable_secret"
sysctl: reading key "net.ipv6.conf.ens33.stable_secret"
sysctl: reading key "net.ipv6.conf.lo.stable_secret"
    947    2960   35450
```

that is a large file you can see.To set a new value to a parameter use sysctl -w :

```text
root@server1:~# sysctl -a | grep swap
sysctl: reading key "net.ipv6.conf.all.stable_secret"
sysctl: reading key "net.ipv6.conf.default.stable_secret"
sysctl: reading key "net.ipv6.conf.ens33.stable_secret"
sysctl: reading key "net.ipv6.conf.lo.stable_secret"
vm.swappiness = 60
root@server1:~# sysctl -w vm.swappiness=80
vm.swappiness = 80
root@server1:~# sysctl -a | grep swap
sysctl: reading key "net.ipv6.conf.all.stable_secret"
sysctl: reading key "net.ipv6.conf.default.stable_secret"
sysctl: reading key "net.ipv6.conf.ens33.stable_secret"
sysctl: reading key "net.ipv6.conf.lo.stable_secret"
vm.swappiness = 80
```

Before quit lets take a look at sysctl.conf :

```text
# /etc/sysctl.conf - Configuration file for setting system variables
# See /etc/sysctl.d/ for additional system variables.
# See sysctl.conf (5) for information.
#

#kernel.domainname = example.com

# Uncomment the following to stop low-level messages on console
#kernel.printk = 3 4 1 3

##############################################################3
# Functions previously found in netbase
#

# Uncomment the next two lines to enable Spoof protection (reverse-path filter)
# Turn on Source Address Verification in all interfaces to
# prevent some spoofing attacks
#net.ipv4.conf.default.rp_filter=1
#net.ipv4.conf.all.rp_filter=1

# Uncomment the next line to enable TCP/IP SYN cookies
# See http://lwn.net/Articles/277146/
# Note: This may impact IPv6 TCP sessions too
#net.ipv4.tcp_syncookies=1

# Uncomment the next line to enable packet forwarding for IPv4
#net.ipv4.ip_forward=1

# Uncomment the next line to enable packet forwarding for IPv6
#  Enabling this option disables Stateless Address Autoconfiguration
#  based on Router Advertisements for this host
#net.ipv6.conf.all.forwarding=1

###################################################################
# Additional settings - these settings can improve the network
# security of the host and prevent against some network attacks
# including spoofing attacks and man in the middle attacks through
# redirection. Some network environments, however, require that these
# settings are disabled so review and enable them as needed.
#
# Do not accept ICMP redirects (prevent MITM attacks)
#net.ipv4.conf.all.accept_redirects = 0
#net.ipv6.conf.all.accept_redirects = 0
# _or_
# Accept ICMP redirects only for gateways listed in our default
# gateway list (enabled by default)
# net.ipv4.conf.all.secure_redirects = 1
#
# Do not send ICMP redirects (we are not a router)
#net.ipv4.conf.all.send_redirects = 0
#
# Do not accept IP source route packets (we are not a router)
#net.ipv4.conf.all.accept_source_route = 0
#net.ipv6.conf.all.accept_source_route = 0
#
# Log Martian Packets
#net.ipv4.conf.all.log_martians = 1

#net.ipv4.ip_forwarding = 1
```

as you can see includes /etc/sysctl.d/ directory setting files but don't for get that setting in sysctl.conf file always over write /etc/sysctl.d/ directory and Win :\) You can uncomment or add setting weather in sysctl.conf or go and create custom file in /etc/sysctl.d/ directory but Take care of contrasts.

### udev

During boot process when kernel is loaded and Device Driver start setting up hardware, next step of hardware initializing is udev. So kernel initial device loading and send uevents to udev daemon. udev keep these event and handle them base on attributes it has received in the event. Then udev take action based on its rules .there are two types of rules in two different locations :

/lib/udev/rules.d "default rules"

/etc/udev/rules.d "custom rules/ user defined"

After Finishing boot process udev is active and receive kernel messages an uevent messages from hot-pluggable devices.

### udevadm monitor , udevadm

udev has a monitoring tool, and it monitor and shows event received from the kernel after hardware initializing and uevent which are udev events after processing udev rules, lets attach a usb and monitor what it shows:

```text
root@server1:~# udevadm monitor 
monitor will print the received events for:
UDEV - the event which udev sends out after rule processing
KERNEL - the kernel uevent

KERNEL[3520.586975] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1 (usb)
KERNEL[3520.589831] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0 (usb)
UDEV  [3520.610064] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1 (usb)
KERNEL[3520.653543] add      /module/usb_storage (module)
UDEV  [3520.655921] add      /module/usb_storage (module)
KERNEL[3520.655983] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33 (scsi)
KERNEL[3520.656293] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/scsi_host/host33 (scsi_host)
KERNEL[3520.656671] add      /bus/usb/drivers/usb-storage (drivers)
UDEV  [3520.659555] add      /bus/usb/drivers/usb-storage (drivers)
KERNEL[3520.664336] add      /module/uas (module)
KERNEL[3520.664472] add      /bus/usb/drivers/uas (drivers)
UDEV  [3520.665071] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0 (usb)
UDEV  [3520.666828] add      /module/uas (module)
UDEV  [3520.669972] add      /bus/usb/drivers/uas (drivers)
UDEV  [3520.671676] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33 (scsi)
UDEV  [3520.673836] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/scsi_host/host33 (scsi_host)
KERNEL[3521.702618] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0 (scsi)
KERNEL[3521.703797] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0 (scsi)
KERNEL[3521.703877] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/scsi_disk/33:0:0:0 (scsi_disk)
KERNEL[3521.703990] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/scsi_device/33:0:0:0 (scsi_device)
KERNEL[3521.706049] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/scsi_generic/sg2 (scsi_generic)
KERNEL[3521.707046] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/bsg/33:0:0:0 (bsg)
UDEV  [3521.709816] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0 (scsi)
UDEV  [3521.715283] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0 (scsi)
UDEV  [3521.728137] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/scsi_disk/33:0:0:0 (scsi_disk)
UDEV  [3521.728500] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/scsi_device/33:0:0:0 (scsi_device)
UDEV  [3521.732039] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/bsg/33:0:0:0 (bsg)
UDEV  [3521.732504] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/scsi_generic/sg2 (scsi_generic)
KERNEL[3522.394218] add      /devices/virtual/bdi/8:16 (bdi)
UDEV  [3522.405809] add      /devices/virtual/bdi/8:16 (bdi)
KERNEL[3522.446634] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/block/sdb (block)
UDEV  [3522.871787] add      /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/block/sdb (block)
KERNEL[3523.061202] add      /module/nls_iso8859_1 (module)
UDEV  [3523.066143] add      /module/nls_iso8859_1 (module)
```

You can see both kernel messages and udev events. And finally our usb flash is mounted as sdb. For having more readable data:

```text
root@server1:~# udevadm info --query=all --name=/dev/sdb
P: /devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/block/sdb
N: sdb
S: disk/by-id/usb-Kingston_DT_101_II_0013729982D5B97196320049-0:0
S: disk/by-label/MYLINUXLIVE
S: disk/by-path/pci-0000:02:03.0-usb-0:1:1.0-scsi-0:0:0:0
S: disk/by-uuid/3879-A38A
E: DEVLINKS=/dev/disk/by-uuid/3879-A38A /dev/disk/by-label/MYLINUXLIVE /dev/disk/by-id/usb-Kingston_DT_101_II_0013729982D5B97196320049-0:0 /dev/disk/by-path/pci-0000:02:03.0-usb-0:1:1.0-scsi-0:0:0:0
E: DEVNAME=/dev/sdb
E: DEVPATH=/devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/block/sdb
E: DEVTYPE=disk
E: ID_BUS=usb
E: ID_FS_LABEL=MYLINUXLIVE
E: ID_FS_LABEL_ENC=MYLINUXLIVE
E: ID_FS_TYPE=vfat
E: ID_FS_USAGE=filesystem
E: ID_FS_UUID=3879-A38A
E: ID_FS_UUID_ENC=3879-A38A
E: ID_FS_VERSION=FAT32
E: ID_INSTANCE=0:0
E: ID_MODEL=DT_101_II
E: ID_MODEL_ENC=DT\x20101\x20II\x20\x20\x20\x20\x20\x20\x20
E: ID_MODEL_ID=1625
E: ID_PATH=pci-0000:02:03.0-usb-0:1:1.0-scsi-0:0:0:0
E: ID_PATH_TAG=pci-0000_02_03_0-usb-0_1_1_0-scsi-0_0_0_0
E: ID_REVISION=PMAP
E: ID_SERIAL=Kingston_DT_101_II_0013729982D5B97196320049-0:0
E: ID_SERIAL_SHORT=0013729982D5B97196320049
E: ID_TYPE=disk
E: ID_USB_DRIVER=usb-storage
E: ID_USB_INTERFACES=:080650:
E: ID_USB_INTERFACE_NUM=00
E: ID_VENDOR=Kingston
E: ID_VENDOR_ENC=Kingston
E: ID_VENDOR_ID=0951
E: MAJOR=8
E: MINOR=16
E: SUBSYSTEM=block
E: TAGS=:systemd:
E: USEC_INITIALIZED=3522768898
```

use udevadm info attribute-walk --name=/dev/sdb \| less to have summary of device attributes:

```text
root@server1:~# udevadm info --attribute-walk --name=/dev/sdb

Udevadm info starts with the device specified by the devpath and then
walks up the chain of parent devices. It prints for every device
found, all possible attributes in the udev rules key format.
A rule to match, can be composed by the attributes of the device
and the attributes from one single parent device.

  looking at device '/devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0/block/sdb':
    KERNEL=="sdb"
    SUBSYSTEM=="block"
    DRIVER==""
    ATTR{alignment_offset}=="0"
    ATTR{badblocks}==""
    ATTR{capability}=="51"
    ATTR{discard_alignment}=="0"
    ATTR{events}=="media_change"
    ATTR{events_async}==""
    ATTR{events_poll_msecs}=="-1"
    ATTR{ext_range}=="256"
    ATTR{inflight}=="       0        0"
    ATTR{range}=="16"
    ATTR{removable}=="1"
    ATTR{ro}=="0"
    ATTR{size}=="15679488"
    ATTR{stat}=="     344    15135    17586     3160        2        0        2      268        0     1792     3428"

  looking at parent device '/devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0/33:0:0:0':
    KERNELS=="33:0:0:0"
    SUBSYSTEMS=="scsi"
    DRIVERS=="sd"
    ATTRS{device_blocked}=="0"
    ATTRS{device_busy}=="0"
    ATTRS{dh_state}=="detached"
    ATTRS{eh_timeout}=="10"
    ATTRS{evt_capacity_change_reported}=="0"
    ATTRS{evt_inquiry_change_reported}=="0"
    ATTRS{evt_lun_change_reported}=="0"
    ATTRS{evt_media_change}=="0"
    ATTRS{evt_mode_parameter_change_reported}=="0"
    ATTRS{evt_soft_threshold_reached}=="0"
    ATTRS{inquiry}==""
    ATTRS{iocounterbits}=="32"
    ATTRS{iodone_cnt}=="0x590"
    ATTRS{ioerr_cnt}=="0x2"
    ATTRS{iorequest_cnt}=="0x590"
    ATTRS{max_sectors}=="240"
    ATTRS{model}=="DT 101 II       "
    ATTRS{queue_depth}=="1"
    ATTRS{queue_type}=="none"
    ATTRS{rev}=="PMAP"
    ATTRS{scsi_level}=="0"
    ATTRS{state}=="running"
    ATTRS{timeout}=="30"
    ATTRS{type}=="0"
    ATTRS{vendor}=="Kingston"

  looking at parent device '/devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33/target33:0:0':
    KERNELS=="target33:0:0"
    SUBSYSTEMS=="scsi"
    DRIVERS==""

  looking at parent device '/devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0/host33':
    KERNELS=="host33"
    SUBSYSTEMS=="scsi"
    DRIVERS==""

  looking at parent device '/devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1/1-1:1.0':
    KERNELS=="1-1:1.0"
    SUBSYSTEMS=="usb"
    DRIVERS=="usb-storage"
    ATTRS{authorized}=="1"
    ATTRS{bAlternateSetting}==" 0"
    ATTRS{bInterfaceClass}=="08"
    ATTRS{bInterfaceNumber}=="00"
    ATTRS{bInterfaceProtocol}=="50"
    ATTRS{bInterfaceSubClass}=="06"
    ATTRS{bNumEndpoints}=="02"
    ATTRS{supports_autosuspend}=="1"

  looking at parent device '/devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1/1-1':
    KERNELS=="1-1"
    SUBSYSTEMS=="usb"
    DRIVERS=="usb"
    ATTRS{authorized}=="1"
    ATTRS{avoid_reset_quirk}=="0"
    ATTRS{bConfigurationValue}=="1"
    ATTRS{bDeviceClass}=="00"
    ATTRS{bDeviceProtocol}=="00"
    ATTRS{bDeviceSubClass}=="00"
    ATTRS{bMaxPacketSize0}=="64"
    ATTRS{bMaxPower}=="300mA"
    ATTRS{bNumConfigurations}=="1"
    ATTRS{bNumInterfaces}==" 1"
    ATTRS{bcdDevice}=="0110"
    ATTRS{bmAttributes}=="80"
    ATTRS{busnum}=="1"
    ATTRS{configuration}==""
    ATTRS{devnum}=="6"
    ATTRS{devpath}=="1"
    ATTRS{idProduct}=="1625"
    ATTRS{idVendor}=="0951"
    ATTRS{ltm_capable}=="no"
    ATTRS{manufacturer}=="Kingston"
    ATTRS{maxchild}=="0"
    ATTRS{product}=="DT 101 II"
    ATTRS{quirks}=="0x0"
    ATTRS{removable}=="unknown"
    ATTRS{serial}=="0013729982D5B97196320049"
    ATTRS{speed}=="480"
    ATTRS{urbnum}=="3230"
    ATTRS{version}==" 2.00"

  looking at parent device '/devices/pci0000:00/0000:00:11.0/0000:02:03.0/usb1':
    KERNELS=="usb1"
    SUBSYSTEMS=="usb"
    DRIVERS=="usb"
    ATTRS{authorized}=="1"
    ATTRS{authorized_default}=="1"
    ATTRS{avoid_reset_quirk}=="0"
    ATTRS{bConfigurationValue}=="1"
    ATTRS{bDeviceClass}=="09"
    ATTRS{bDeviceProtocol}=="00"
    ATTRS{bDeviceSubClass}=="00"
    ATTRS{bMaxPacketSize0}=="64"
    ATTRS{bMaxPower}=="0mA"
    ATTRS{bNumConfigurations}=="1"
    ATTRS{bNumInterfaces}==" 1"
    ATTRS{bcdDevice}=="0410"
    ATTRS{bmAttributes}=="e0"
    ATTRS{busnum}=="1"
    ATTRS{configuration}==""
    ATTRS{devnum}=="1"
    ATTRS{devpath}=="0"
    ATTRS{idProduct}=="0002"
    ATTRS{idVendor}=="1d6b"
    ATTRS{interface_authorized_default}=="1"
    ATTRS{ltm_capable}=="no"
    ATTRS{manufacturer}=="Linux 4.10.0-40-generic ehci_hcd"
    ATTRS{maxchild}=="6"
    ATTRS{product}=="EHCI Host Controller"
    ATTRS{quirks}=="0x0"
    ATTRS{removable}=="unknown"
    ATTRS{serial}=="0000:02:03.0"
    ATTRS{speed}=="480"
    ATTRS{urbnum}=="88"
    ATTRS{version}==" 2.00"

  looking at parent device '/devices/pci0000:00/0000:00:11.0/0000:02:03.0':
    KERNELS=="0000:02:03.0"
    SUBSYSTEMS=="pci"
    DRIVERS=="ehci-pci"
    ATTRS{acpi_index}=="16777752"
    ATTRS{broken_parity_status}=="0"
    ATTRS{class}=="0x0c0320"
    ATTRS{companion}==""
    ATTRS{consistent_dma_mask_bits}=="32"
    ATTRS{d3cold_allowed}=="0"
    ATTRS{device}=="0x0770"
    ATTRS{dma_mask_bits}=="32"
    ATTRS{driver_override}=="(null)"
    ATTRS{enable}=="1"
    ATTRS{irq}=="17"
    ATTRS{label}=="ehci"
    ATTRS{local_cpulist}=="0-3"
    ATTRS{local_cpus}=="00000000,00000000,00000000,0000000f"
    ATTRS{msi_bus}=="1"
    ATTRS{numa_node}=="-1"
    ATTRS{revision}=="0x00"
    ATTRS{subsystem_device}=="0x0770"
    ATTRS{subsystem_vendor}=="0x15ad"
    ATTRS{uframe_periodic_max}=="100"
    ATTRS{vendor}=="0x15ad"

  looking at parent device '/devices/pci0000:00/0000:00:11.0':
    KERNELS=="0000:00:11.0"
    SUBSYSTEMS=="pci"
    DRIVERS==""
    ATTRS{broken_parity_status}=="0"
    ATTRS{class}=="0x060401"
    ATTRS{consistent_dma_mask_bits}=="32"
    ATTRS{d3cold_allowed}=="0"
    ATTRS{device}=="0x0790"
    ATTRS{dma_mask_bits}=="32"
    ATTRS{driver_override}=="(null)"
    ATTRS{enable}=="1"
    ATTRS{irq}=="0"
    ATTRS{local_cpulist}=="0-3"
    ATTRS{local_cpus}=="00000000,00000000,00000000,0000000f"
    ATTRS{msi_bus}=="1"
    ATTRS{numa_node}=="-1"
    ATTRS{revision}=="0x02"
    ATTRS{subsystem_device}=="0x0790"
    ATTRS{subsystem_vendor}=="0x15ad"
    ATTRS{vendor}=="0x15ad"

  looking at parent device '/devices/pci0000:00':
    KERNELS=="pci0000:00"
    SUBSYSTEMS==""
    DRIVERS==""
```

Oh. That is a long long list of attributes, but we notice that udev consider all of these attributes when it checks it rules!

Thanks udev, but udev dose another great job for us, can you remember? udev populate /dev/ and create Hardware Abstraction, which is used my other programs. Udev has made world easier place for us and other programs.Thank you udev :\)

### dmesg

In LPIC1 Course we discussed that during boot process \(when operating system hasn't been started yet\) dmesg logs kernel ring buffer. Let go deeper, during the boot process kernel is loaded into RAM. At this stage there is a Device Driver presents in kernel which set up to drive relevant hardware. As any other elements of kernel, device driver produce messages. These messages have information about modules presents and value of any parameters. As booting process is so fast, dmesg help us to review what has happened during boot process.

Even after booting kernel might generate some messages, for example when an I/O devicefacing the problem, or a hot-pluggable device like USB is attached to system. Again dmesg can be used to review what kernel has generated during doing its jobs.

```text
tion="profile_load" profile="unconfined" name="/usr/lib/lightdm/lightdm-guest-session" pid=638 comm="apparmor_parser"
[    5.013133] audit: type=1400 audit(1512370178.278:3): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/lightdm/lightdm-guest-session//chromium" pid=638 comm="apparmor_parser"
[    5.032577] audit: type=1400 audit(1512370178.302:4): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/bin/evince" pid=640 comm="apparmor_parser"
[    5.032579] audit: type=1400 audit(1512370178.302:5): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/bin/evince//sanitized_helper" pid=640 comm="apparmor_parser"
[    5.032580] audit: type=1400 audit(1512370178.302:6): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/bin/evince-previewer" pid=640 comm="apparmor_parser"
[    5.032581] audit: type=1400 audit(1512370178.302:7): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/bin/evince-previewer//sanitized_helper" pid=640 comm="apparmor_parser"
[    5.032582] audit: type=1400 audit(1512370178.302:8): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/bin/evince-thumbnailer" pid=640 comm="apparmor_parser"
[    5.032583] audit: type=1400 audit(1512370178.302:9): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/bin/evince-thumbnailer//sanitized_helper" pid=640 comm="apparmor_parser"
[    5.046032] audit: type=1400 audit(1512370178.314:10): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/sbin/cups-browsed" pid=668 comm="apparmor_parser"
[    5.662430] random: crng init done
[    5.668377] Adding 1045500k swap on /dev/sda5.  Priority:-1 extents:1 across:1045500k FS
[    6.567089] IPv6: ADDRCONF(NETDEV_UP): ens33: link is not ready
[    6.574513] IPv6: ADDRCONF(NETDEV_UP): ens33: link is not ready
[    6.575691] e1000: ens33 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: None
[    6.576992] IPv6: ADDRCONF(NETDEV_CHANGE): ens33: link becomes ready
[    6.866049] NET: Registered protocol family 40
[   12.878693] Bluetooth: Core ver 2.22
[   12.878785] NET: Registered protocol family 31
[   12.878787] Bluetooth: HCI device and connection manager initialized
[   12.878792] Bluetooth: HCI socket layer initialized
[   12.878795] Bluetooth: L2CAP socket layer initialized
[   12.878805] Bluetooth: SCO socket layer initialized
[   12.887893] Netfilter messages via NETLINK v0.30.
[   13.014635] device ens33 entered promiscuous mode
[ 2076.880855] Bluetooth: BNEP (Ethernet Emulation) ver 1.3
[ 2076.880856] Bluetooth: BNEP filters: protocol multicast
[ 2076.880860] Bluetooth: BNEP socket layer initialized
[ 2080.858589] ISO 9660 Extensions: Microsoft Joliet Level 3
[ 2080.875428] ISO 9660 Extensions: RRIP_1991A
[ 2098.732330] usb 1-1: new high-speed USB device number 2 using ehci-pci
[ 2098.920575] usb 1-1: device descriptor read/64, error 18
[ 2099.260910] usb 1-1: device descriptor read/64, error 18
[ 2099.597146] usb 1-1: new high-speed USB device number 3 using ehci-pci
[ 2099.839122] usb 1-1: device descriptor read/64, error 18
[ 2100.182250] usb 1-1: device descriptor read/64, error 18
[ 2100.522218] usb 1-1: new high-speed USB device number 4 using ehci-pci
[ 2100.546399] usb 1-1: Invalid ep0 maxpacket: 9
[ 2100.779187] usb 1-1: new high-speed USB device number 5 using ehci-pci
[ 2100.813183] usb 1-1: Invalid ep0 maxpacket: 9
[ 2100.819476] usb usb1-port1: unable to enumerate USB device
```

result above is abstracted, how ever you can see that dmesg dosn't workout well to give information about modules and parameters.

There are some user space commands from lpic1, lets just review:

| command | Description |
| :--- | :--- |
| lspci | List PCi devices on the computer |
| lsusb | Shows Connected USB Devices |
| lsdev | List all Devices and Device Information |

