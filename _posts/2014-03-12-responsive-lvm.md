---
layout: post
title:  "Responsive LVM"
date:   2014-03-12 16:02:00
categories: linux, lvm, storage
---

Rebooting is a sin.

One of the nice things about Linux is it's agility and stability in the face of changes and the fact that it rarely needs rebooting. Possesing the right knowledge allows us sysadmins to be responsive and adapt our systems to change without rebooting.

The Logical Volume Manager provides a powerful storage subsystem in Linux, including advanced features like spanning of partitions across disks, live resizing of volumes and taking snapshots.

This page covers the basic concepts of LVM and demonstrates a way to resize a volume without rebooting the system, even the root volume. The last bit is key. The power and flexibility of Linux allows making significant changes to the system while it is running, yet certain subtleties must be understood to accomplish certain changes on a live system.

Logical Volume Manager

First, I’ll outline the basics of LVM then move onto a step by step example of resizing a root volume.

PVs VGs and LVs

LVM can be thought of as a hierarchy of containers. The phsyical volumes contain volume groups, which in turn contain logical volumes. As such, the basic steps in creating storage is to create physical volumes on one or more block storage devices, create volume groups than span one or more physical volumes, then create logical volumes within volume groups.

Creating Physical Volumes

Start by allocating block devices for use with LVM. Locate the devices you wish to use:

dmesg | grep -i 'attached scsi'
sd 0:2:0:0: Attached scsi disk sda
sd 0:2:1:0: Attached scsi disk sdb
sd 0:2:2:0: Attached scsi disk sdc

The pvcreate command is used to make a phsyical volume. Let’s create physical volumes on sdb and sdc.

pvcreate /dev/sdb /dev/sdc
Writing physical volume data to disk "/dev/sdb"
Physical volume "/dev/sdb" successfully created
Writing physical volume data to disk "/dev/sdc"
Physical volume "/dev/sdc" successfully created

Creating a Volume Group

Now you can make a volume group that uses sdb and sdc. This VG will be a single entity that spans 2 physical disks, /dev/sdb and /dev/sdc.

vgcreate VolGroup01 /dev/sdb /dev/sdc
Volume group "VolGroup01" successfully created
You can use vgdisplay to get information about the VG and verify what you have:

vgdisplay
--- Volume group ---
VG Name VolGroup01
System ID
Format lvm2
Metadata Areas 2
Metadata Sequence No 1
VG Access read/write
VG Status resizable
MAX LV 0
Cur LV 0
Open LV 0
Max PV 0
Cur PV 2
Act PV 2
VG Size 1.91 TB
PE Size 4.00 MB
Total PE 500062
Alloc PE / Size 0 / 0
Free PE / Size 500062 / 1.91 TB
VG UUID vbBVrG-rrX1-nM0C-KYqN-Lfkg-F4ci-2S4vmb

Creating a Logical Volume

Next, inside the volume group created above we will create 2 logical volumes that are 950 GB each.

lvcreate -L 950G -n LogVol_Data1 VolGroup01
Logical volume "LogVol_Data1" created
lvcreate -L 950G -n LogVol_Data2 VolGroup01
Logical volume "LogVol_Data2" created

Format the Volume

The logical volume is almost ready for use. Create a filesystem on the LV as follows:

mkfs -t ext3 /dev/VolGroup01/LogVol_Data1
mke2fs 1.39 (29-May-2006)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
124518400 inodes, 249036800 blocks
12451840 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=4294967296
7600 block groups
32768 blocks per group, 32768 fragments per group
16384 inodes per group
Superblock backups stored on blocks:
32768, 98304...

Writing inode tables: done



Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done



The filesystem can now be mounted at /dev/VolGroup01/LogVol_Data1. Let’s create a direcotry for the mount point and do so now:

mkdir /data
mount -t ext3 /dev/VolGroup01/LogVol_Data1 /data
Various lvm commands are available as pv*, vg*, and lv*. Some useful ones being:

 

pvcreate
 

pvdisplay
 

 

pvremove
 

 

pvresize
 

 

vgcreate
 

 

vgdisplay
 

 

vgremove
 

 

vgextend
 

 

lvcreate
 

 

lvdisplay
 

 

lvreduce
 

 

 

Expanding a Logical Volume
Logical volumes can be expanded without unmounting the underlying filesystems they contain. Even the / filesystem can be expanded live, thereby eliminating the need to reboot.
To expand a logical volume onto a new disk:
Add the New Disk
After adding a the new drive, rescan the SCSI bus to force the kernel detect the new disk:

echo "- - -" > /sys/class/scsi_host/host0/scan
Locate the device node of the new disk with dmesg:

dmesg | grep -i "attached scsi"
sd 0:0:0:0: Attached scsi disk sda
sd 0:0:0:0: Attached scsi generic sg0 type 0
sd 0:0:1:0: Attached scsi disk sdb

In the above example, an existing single disk sda was present before adding the new disk, so the new volume is sdb.

Partition the Disk

Use fdisk to create a single partition on the new device, using type 8e (LVM) with the following commands:

fdisk /dev/sdb
n - new partition
p - primary type
t - set hex code to 8e
w - write changes
fdisk /dev/sdb





Command (m for help): n
Command action



e extended
p primary partition (1–4)
p

Partition number (1-4): 1
First cylinder (1-4699, default 1):
Using default value 1
Last cylinder or +size or +sizeM or +sizeK (1-4699, default 4699):
Using default value 4699
Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 8e
Changed system type of partition 1 to 8e (Linux LVM)

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table.
The new table will be used at the next reboot.



Syncing disks.



The warning above can be ignored as you will now use partprobe to inform the kernel of the new partition. A reboot is not necessary.

partprobe /dev/sdb

Create the Physcial Volume

Initialize the new partition as a physical volume to enable its use by LVM:

pvcreate /dev/sdb1
Writing physical volume data to disk "/dev/sdb1"
Physical volume "/dev/sdb1" successfully created

Identify the volume group (VG) you wish to extend with vgdisplay.

vgdisplay
--- Volume group ---
VG Name VolGroup00
System ID
Format lvm2
Metadata Areas 2
Metadata Sequence No 6
VG Access read/write
VG Status resizable
MAX LV 0
Cur LV 2
Open LV 2
Max PV 0
Cur PV 2
Act PV 2
VG Size 28.18 GB
PE Size 32.00 MB
Total PE 614
Alloc PE / Size 592 / 18.50 GB
Free PE / Size 22 / 704.00 MB
VG UUID qDcbn4-cgXJ-cTeX-mXuh-V06I-AFOU-vZKeEH

Extend the Volume Group

Use vgextend to extend the VG identified above to include the new physical volume:

vgextend VolGroup00 /dev/sdb1
Volume group “VolGroup00” successfully extended
Run vgdisplay again and note the increased size (VG Size.)

vgdisplay
--- Volume group ---
VG Name VolGroup00
System ID
Format lvm2
Metadata Areas 3
Metadata Sequence No 7
VG Access read/write
VG Status resizable
MAX LV 0
Cur LV 2
Open LV 2
Max PV 0
Cur PV 3
Act PV 3
VG Size 55.16 GB
PE Size 32.00 MB
Total PE 1765
Alloc PE / Size 592 / 18.50 GB
Free PE / Size 1173 / 36.66 GB
VG UUID qDcbn4-cgXJ-cTeX-mXuh-V06I-AFOU-vZKeEH

Extend the Logical Volume

Identify the logical volume you wish to extend with lvdisplay

lvdisplay
-- Logical volume --
LV Name /dev/VolGroup00/LogVol00
VG Name VolGroup00
LV UUID T7lMW0-W1Rx-zRKw-evZy-tcSp-gSgF-9HAx3w
LV Write Access read/write
LV Status available
open 1
LV Size 18.00 GB
Current LE 576
Segments 2
Allocation inherit
Read ahead sectors auto
currently set to 256
Block device 253:0

Notice the difference in size between the VG and the LV. In this case the volume group has 37 GB of additional space that the logical volume can use.
Extend the logical volume to use the space.

lvextend -L 37G /dev/VolGroup00/LogVol00
Extending logical volume LogVol00 to 37.00 GB
Logical volume LogVol00 successfully resized

Resize the Filesystem

The underlying filesystem needs to be expanded to use the additional space using resize2fs.

resize2fs /dev/mapper/VolGroup00-LogVol00
resize2fs 1.39 (29-May-2006)
Filesystem at /dev/mapper/VolGroup00-LogVol00 is mounted on /; on-line resizing required
Performing an on-line resize of /dev/mapper/VolGroup00-LogVol00 to 9699328 (4k) blocks.
The filesystem on /dev/mapper/VolGroup00-LogVol00 is now 9699328 blocks long.
Verify the free space with df -h

df -h



Filesystem Size Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00
36G 16G 19G 47% /
/dev/sda1 99M 20M 74M 21% /boot
tmpfs 502M 0 502M 0% /dev/shm



Find the device using dmesg output

[root@cddsql-bak-sc9 ~]# dmesg | grep -i 'device sdb'
SCSI device sdb: 15623782400 512-byte hdwr sectors (7999377 MB)
SCSI device sdb: drive cache: write back
SCSI device sdb: 15623782400 512-byte hdwr sectors (7999377 MB)
SCSI device sdb: drive cache: write back 

Use parted to create a gpt partition table

parted /dev/sdb 
Write a gpt partition table

(parted) mklabel gpt
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost.      Do you want to continue?
Yes/No? yes
New disk label type?  [gpt]?  
Display partition table and note the size of the disk, in this case 7999GB
(parted) print                                                            
Model: DELL PERC H700 (scsi)
Disk /dev/sdb: 7999GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start  End  Size  File system  Name  Flags  
Using the disk size, create a primary partition
(parted) mkpart primary 0GB 7999GB  
Exit
(parted) quit 
Write an ext3 filesystem
[root@cddsql-bak-sac postfix]# mkfs.ext3  /dev/sdb1
mke2fs 1.39 (29-May-2006)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
976486400 inodes, 1952972791 blocks
97648639 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=4294967296
59600 block groups
32768 blocks per group, 32768 fragments per group
16384 inodes per group
Superblock backups stored on blocks: 
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
    4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
    102400000, 214990848, 512000000, 550731776, 644972544, 1934917632
Writing inode tables: done 
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done





This filesystem will be automatically checked every 37 mounts or
180 days, whichever comes first. Use tune2fs -c or -i to override. 
Write a label to reference the partition in fstab
e2label /dev/sdb1 vadata 
Add entry in /etc/fstab
LABEL=vadata /vadata ext3 defaults 0 0



Finally, create a mount point and mount it.

mkdir /vadata
mount -a 
