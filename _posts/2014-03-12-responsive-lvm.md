---
layout: post
title:  "Responsive LVM"
date:   2014-03-12 16:02:00
categories:
---

# Linux Storage Systems

Herein are general guidelines for managing storage in Linux including the Logical Volume Manager (LVM,) disk partitioning, disk resizing, filesystem mounting, etc.

## Logical Volume Manager

LVM provides flexible storage management in Linux, allowing for advanced features like spanning of partitions across disks, live resizing of volumes and taking snapshots.

**PVs VGs and LVs**

LVM can be thought of as a hierarchy of containers. The **phsyical volumes** contain **volume groups**, which in turn contain **logical volumes.** As such, the basic steps in creating storage is to create physical volumes on one or more block storage devices, create volume groups than span one or more physical volumes, then create logical volumes within volume groups.

### Creating Physical Volumes

Start by allocating block devices for use with LVM. Locate the devices you wish to use:

    # dmesg | grep -i 'attached scsi' 
    sd 0:2:0:0: Attached scsi disk sda
    sd 0:2:1:0: Attached scsi disk sdb
    sd 0:2:2:0: Attached scsi disk sdc

Use pvcreate to make new phsyical volumes for use by LVM. Let's create volumes on sdb and sdc.

    # pvcreate /dev/sdb /dev/sdc
    Writing physical volume data to disk "/dev/sdb"
    Physical volume "/dev/sdb" successfully created
    Writing physical volume data to disk "/dev/sdc"
    Physical volume "/dev/sdc" successfully created

### Creating a Volume Group

Now you can make a volume group that uses sdb and sdc. 

    # vgcreate VolGroup01 /dev/sdb /dev/sdc
    Volume group "VolGroup01" successfully created

Use **vgdisplay** to get information about the VG. 

    --- Volume group ---
    VG Name               VolGroup01
    System ID
    Format                lvm2
    Metadata Areas        2
    Metadata Sequence No  1
    VG Access             read/write
    VG Status             resizable
    MAX LV                0
    Cur LV                0
    Open LV               0
    Max PV                0
    Cur PV                2
    Act PV                2
    VG Size               1.91 TB
    PE Size               4.00 MB
    Total PE              500062
    Alloc PE / Size       0 / 0
    Free  PE / Size       500062 / 1.91 TB
    VG UUID               vbBVrG-rrX1-nM0C-KYqN-Lfkg-F4ci-2S4vmb

### Creating a Logical Volume

Next, create 2 logical volumes inside the new VG.

    # lvcreate -L 950G -n LogVol_Data1 VolGroup01
    Logical volume "LogVol_Data1" created
 
    # lvcreate -L 950G -n LogVol_Data2 VolGroup02
    Logical volume "LogVol_Data2" created
 
    # lvdisplay

    --- Logical volume ---
    LV Name                /dev/VolGroup01/LogVol_Data1
    VG Name                VolGroup01
    LV UUID                xyNzCp-DqZ7-k2bG-P3ws-va9v-gVvO-ORM0Re
    LV Write Access        read/write
    LV Status              available
    # open                 0
    LV Size                950.00 GB
    Current LE             243200
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     256
    Block device           253:2

    --- Logical volume ---
    LV Name                /dev/VolGroup01/LogVol_Data2
    VG Name                VolGroup01
    LV UUID                Z0gsy1-gMz5-ttmr-5VP7-9Gsn-npoz-22JSl6
    LV Write Access        read/write
    LV Status              available
    # open                 0
    LV Size                950.00 GB
    Current LE             243200
    Segments               2
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     256
    Block device           253:3

### Create a Filesystem on the Logical Volumes

    # mkfs -t ext3 /dev/VolGroup01/LogVol_Data1

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
      32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
      4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
      102400000, 214990848

    Writing inode tables: done
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done

The filesystem can now be mounted at /dev/VolGroup01/LogVol_Data1

    mkdir /data && mount -t ext3 /dev/VolGroup01/LogVol_Data1 /data

Various lvm commands are available as pv\*, vg\*, and lv\*. Some useful ones being:

* pvcreate
* pvdisplay
* pvremove
* pvresize
* vgcreate
* vgdisplay
* vgremove
* vgextend
* lvcreate
* lvdisplay
* lvreduce

## Expanding a Logical Volume

Now I will describe how to expand a logical volume onto a newly added disk.

Up to this point I've covered basic practial uses of LVM. Now, to elaborate on this with a useful real world scenerio. Suppost you're server is running out of space on the root volume. Since logical volumes can be expanded without unmounting the underlying filesystems they contain, even the root filesystem can be expanded live, thereby eliminating the need to reboot.

So next time you're low on space and your server has an uptime of 987 days, keep that system running and follow these steps.

### Add the New Disk

After adding the disk you usually need to rescan the SCSI bus to signal the kernel to recognize the new disk. This bit of magic prevents the need to reboot:

    # echo "- - -" > /sys/class/scsi_host/host0/scan

Locate the device node of the new disk with dmesg:

    # dmesg | grep -i "attached scsi"

    sd 0:0:0:0: Attached scsi disk sda
    sd 0:0:0:0: Attached scsi generic sg0 type 0
    sd 0:0:1:0: Attached scsi disk sdb

In the above example, an existing single disk sda was present before adding the new disk, so the new volume is sdb.

### Partitioning

Usually I don't partition at this stage, as I typically use the entire disk for LVM but if you wish you can slice it up with `fdisk` or `parted`.

### Create the Physcial Volume

Initialize the new partition as a physical volume to enable its use by LVM:

    # pvcreate /dev/sdb1
  
    Writing physical volume data to disk "/dev/sdb1"
    Physical volume "/dev/sdb1" successfully created


Identify the volume group (VG) you wish to extend with **vgdisplay.**

    # vgdisplay

    --- Volume group ---
    VG Name               VolGroup00
    System ID
    Format                lvm2
    Metadata Areas        2
    Metadata Sequence No  6
    VG Access             read/write
    VG Status             resizable
    MAX LV                0
    Cur LV                2
    Open LV               2
    Max PV                0
    Cur PV                2
    Act PV                2
    VG Size               28.18 GB
    PE Size               32.00 MB
    Total PE              614
    Alloc PE / Size       592 / 18.50 GB
    Free  PE / Size       22 / 704.00 MB
    VG UUID               qDcbn4-cgXJ-cTeX-mXuh-V06I-AFOU-vZKeEH

### Extend the Volume Group

Use **vgextend** to extend the VG identified above to include the new physical volume:

    # vgextend VolGroup00 /dev/sdb1

    Volume group "VolGroup00" successfully extended

Now run **vgdisplay** again and note the increased size shown at **VG Size.**

    # vgdisplay

    --- Volume group ---
    VG Name               VolGroup00
    System ID
    Format                lvm2
    Metadata Areas        3
    Metadata Sequence No  7
    VG Access             read/write
    VG Status             resizable
    MAX LV                0
    Cur LV                2
    Open LV               2
    Max PV                0
    Cur PV                3
    Act PV                3
    VG Size               55.16 GB
    PE Size               32.00 MB
    Total PE              1765
    Alloc PE / Size       592 / 18.50 GB
    Free  PE / Size       1173 / 36.66 GB
    VG UUID               qDcbn4-cgXJ-cTeX-mXuh-V06I-AFOU-vZKeEH

### Extend the Logical Volume

Identify the logical volume you wish to extend with **lvdisplay**

    # lvdisplay

    -- Logical volume --
    LV Name                /dev/VolGroup00/LogVol00
    VG Name                VolGroup00
    LV UUID                T7lMW0-W1Rx-zRKw-evZy-tcSp-gSgF-9HAx3w
    LV Write Access        read/write
    LV Status              available
    # open                 1
    LV Size                18.00 GB
    Current LE             576
    Segments               2
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     256
    Block device           253:0

Notice the difference in size between the VG and the LV. In this case the volume group has 37 GB of additional space that the logical volume can use.

Extend the logical volume to use the space.

    # lvextend -L 37G /dev/VolGroup00/LogVol00

    Extending logical volume LogVol00 to 37.00 GB
    Logical volume LogVol00 successfully resized


### Resize the Filesystem

The underlying filesystem needs to be expanded to use the additional space using **resize2fs.**

    # resize2fs /dev/mapper/VolGroup00-LogVol00

    resize2fs 1.39 (29-May-2006)
    Filesystem at /dev/mapper/VolGroup00-LogVol00 is mounted on /; on-line resizing required
    Performing an on-line resize of /dev/mapper/VolGroup00-LogVol00 to 9699328 (4k) blocks.
    The filesystem on /dev/mapper/VolGroup00-LogVol00 is now 9699328 blocks long.

Verify the free space with **df -h**

    # df -h
    Filesystem            Size  Used Avail Use% Mounted on
    /dev/mapper/VolGroup00-LogVol00
                           36G   16G   19G  47% /
    /dev/sda1              99M   20M   74M  21% /boot
    tmpfs                 502M     0  502M   0% /dev/shm

1. Add entry in `/etc/fstab`
    `LABEL=vadata /vadata ext3 defaults 0 0`
2. Create mount point
    `mkdir /vadata`
3. Mount the partition
    `mount -a`
