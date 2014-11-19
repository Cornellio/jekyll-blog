---
layout: post
title:  "Responsive LVM(html)"
date:   2014-03-12 16:02:00
categories: linux, lvm, storage
---

<p>Rebooting is a sin.</p>
<p>One of the nice things about Linux is it's agility and stability in the face of changes and the fact that it rarely needs rebooting. Possesing the right knowledge allows us sysadmins to be responsive and adapt our systems to change without rebooting.</p>
<p>The Logical Volume Manager provides a powerful storage subsystem in Linux, including advanced features like spanning of partitions across disks, live resizing of volumes and taking snapshots.</p>
<p>This page covers the basic concepts of LVM and demonstrates a way to resize a volume without rebooting the system, even the root volume. The last bit is key. The power and flexibility of Linux allows making significant changes to the system while it is running, yet certain subtleties must be understood to accomplish certain changes on a live system.</p>
<h2 class="">Logical Volume Manager</h2>
<p>First, I’ll outline the basics of LVM then move onto a step by step example of resizing a root volume.</p>
<p><strong>PVs VGs and LVs</strong></p>
<p>LVM can be thought of as a hierarchy of containers. The phsyical volumes contain volume groups, which in turn contain logical volumes. As such, the basic steps in creating storage is to create physical volumes on one or more block storage devices, create volume groups than span one or more physical volumes, then create logical volumes within volume groups.</p>
<h3 class="">Creating Physical Volumes</h3>

<p>Start by allocating block devices for use with LVM. Locate the devices you wish to use:</p>
<pre>dmesg | grep -i 'attached scsi'
<p>sd 0:2:0:0: Attached scsi disk sda<br>sd 0:2:1:0: Attached scsi disk sdb<br>sd 0:2:2:0: Attached scsi disk sdc<code><br></code><br><span style="font-family: 'Helvetica Neue';">The </span><code>pvcreate</code><span style="font-family: 'Helvetica Neue';"> command is used to make a phsyical volume. Let’s create physical volumes on sdb and sdc.</span></p></pre><p></p>
<pre><code>pvcreate /dev/sdb /dev/sdc</code>
<p><code>Writing physical volume data to disk "/dev/sdb"<br>Physical volume "/dev/sdb" successfully created<br>Writing physical volume data to disk "/dev/sdc"<br>Physical volume "/dev/sdc" successfully created<br></code></p></pre><p></p>
<h3 class="">Creating a Volume Group</h3>
<p>Now you can make a volume group that uses sdb and sdc. This VG will be a single entity that spans 2 physical disks, <code>/dev/sdb</code> and <code>/dev/sdc.</code></p>
<pre><code>vgcreate VolGroup01 /dev/sdb /dev/sdc
Volume group "VolGroup01" successfully created
</code></pre>
<p>You can use <code>vgdisplay</code> to get information about the VG and verify what you have:</p>
<pre><code>vgdisplay</code>
<p><code>--- Volume group ---<br>VG Name               VolGroup01<br>System ID<br>Format                lvm2<br>Metadata Areas        2<br>Metadata Sequence No  1<br>VG Access             read/write<br>VG Status             resizable<br>MAX LV                0<br>Cur LV                0<br>Open LV               0<br>Max PV                0<br>Cur PV                2<br>Act PV                2<br>VG Size               1.91 TB<br>PE Size               4.00 MB<br>Total PE              500062<br>Alloc PE / Size       0 / 0<br>Free  PE / Size       500062 / 1.91 TB<br>VG UUID               vbBVrG-rrX1-nM0C-KYqN-Lfkg-F4ci-2S4vmb<br></code></p></pre><p></p>
<h3 class="">Creating a Logical Volume</h3>
<p>Next, inside the volume group created above we will create 2 logical volumes that are 950 GB each.</p>
<pre><code>lvcreate -L 950G -n LogVol_Data1 VolGroup01
Logical volume "LogVol_Data1" created</code>
<p><code>lvcreate -L 950G -n LogVol_Data2 VolGroup01<br>Logical volume "LogVol_Data2" created<br></code></p></pre><p></p>
<h3 class="">Format the Volume</h3>
<p>The logical volume is almost ready for use. Create a filesystem on the LV as follows:</p>
<pre><code>mkfs -t ext3 /dev/VolGroup01/LogVol_Data1
<p>mke2fs 1.39 (29-May-2006)<br>Filesystem label=<br>OS type: Linux<br>Block size=4096 (log=2)<br>Fragment size=4096 (log=2)<br>124518400 inodes, 249036800 blocks<br>12451840 blocks (5.00%) reserved for the super user<br>First data block=0<br>Maximum filesystem blocks=4294967296<br>7600 block groups<br>32768 blocks per group, 32768 fragments per group<br>16384 inodes per group<br>Superblock backups stored on blocks:<br>32768, 98304...</p>
<p>Writing inode tables: done</p></code><p><code></code></p><p><code>Creating journal (32768 blocks): done<br>Writing superblocks and filesystem accounting information: done<br></code></p><br><br><p></p></pre><p></p>
<p>The filesystem can now be mounted at /dev/VolGroup01/LogVol_Data1. Let’s create a direcotry for the mount point and do so now:</p>
<pre><code>mkdir /data
mount -t ext3 /dev/VolGroup01/LogVol_Data1 /data
</code></pre>
<p>Various lvm commands are available as pv*, vg*, and lv*. Some useful ones being:</p>
<p>&nbsp;</p>
<ul>
<ul>
<li>pvcreate</li>
</ul>
</ul>
<p>&nbsp;</p>
<ul>
<li>pvdisplay</li>
</ul>
<p>&nbsp;</p>
<p>&nbsp;</p>
<ul>
<li>pvremove</li>
</ul>
<p>&nbsp;</p>
<p>&nbsp;</p>
<ul>
<li>pvresize</li>
</ul>
<p>&nbsp;</p>
<p>&nbsp;</p>
<ul>
<li>vgcreate</li>
</ul>
<p>&nbsp;</p>
<p>&nbsp;</p>
<ul>
<li>vgdisplay</li>
</ul>
<p>&nbsp;</p>
<p>&nbsp;</p>
<ul>
<li>vgremove</li>
</ul>
<p>&nbsp;</p>
<p>&nbsp;</p>
<ul>
<li>vgextend</li>
</ul>
<p>&nbsp;</p>
<p>&nbsp;</p>
<ul>
<li>lvcreate</li>
</ul>
<p>&nbsp;</p>
<p>&nbsp;</p>
<ul>
<li>lvdisplay</li>
</ul>
<p>&nbsp;</p>
<p>&nbsp;</p>
<ul>
<li>lvreduce</li>
</ul>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>Expanding a Logical Volume<br>Logical volumes can be expanded without unmounting the underlying filesystems they contain. Even the / filesystem can be expanded live, thereby eliminating the need to reboot.<br>To expand a logical volume onto a new disk:<br>Add the New Disk<br>After adding a the new drive, rescan the SCSI bus to force the kernel detect the new disk:</p>
<pre><code>echo "- - -" &gt; /sys/class/scsi_host/host0/scan
</code></pre>
<p>Locate the device node of the new disk with dmesg:</p>
<pre><code>dmesg | grep -i "attached scsi"</code>
<p><code>sd 0:0:0:0: Attached scsi disk sda<br>sd 0:0:0:0: Attached scsi generic sg0 type 0<br>sd 0:0:1:0: Attached scsi disk sdb<br></code></p></pre><p></p>
<p>In the above example, an existing single disk sda was present before adding the new disk, so the new volume is sdb.</p>
<p>Partition the Disk</p>
<p>Use fdisk to create a single partition on the new device, using type 8e (LVM) with the following commands:</p>
<pre><code>fdisk /dev/sdb
n - new partition
p - primary type
t - set hex code to 8e
w - write changes
<p>fdisk /dev/sdb</p></code><p><code></code></p><p><code>Command (m for help): n<br>Command action<br></code></p><br><br><p></p></pre><p></p>
<p>e extended<br>p primary partition (1–4)<br>p</p>
<pre><code>Partition number (1-4): 1
First cylinder (1-4699, default 1):
Using default value 1
Last cylinder or +size or +sizeM or +sizeK (1-4699, default 4699):
Using default value 4699
<p>Command (m for help): t<br>Selected partition 1<br>Hex code (type L to list codes): 8e<br>Changed system type of partition 1 to 8e (Linux LVM)</p>
<p>Command (m for help): w<br>The partition table has been altered!</p>
<p>Calling ioctl() to re-read partition table.</p>
<p>WARNING: Re-reading the partition table failed with error 16: Device or resource busy.<br>The kernel still uses the old table.<br>The new table will be used at the next reboot.</p></code><p><code></code></p><p><code>Syncing disks.<br></code></p><br><br><p></p></pre><p></p>
<p>The warning above can be ignored as you will now use partprobe to inform the kernel of the new partition. A reboot is not necessary.</p>
<p>partprobe /dev/sdb</p>
<h3 class="">Create the Physcial Volume</h3>
<p>Initialize the new partition as a physical volume to enable its use by LVM:</p>
<pre><code>pvcreate /dev/sdb1</code>
<p><code>Writing physical volume data to disk "/dev/sdb1"<br>Physical volume "/dev/sdb1" successfully created<br></code></p></pre><p></p>
<p>Identify the volume group (VG) you wish to extend with vgdisplay.</p>
<pre><code>vgdisplay</code>
<p><code>--- Volume group ---<br>VG Name               VolGroup00<br>System ID<br>  Format                lvm2<br>  Metadata Areas        2<br> Metadata Sequence No  6<br> VG Access             read/write<br> VG Status             resizable<br> MAX LV                0<br> Cur LV                2<br> Open LV               2<br> Max PV                0<br> Cur PV                2<br> Act PV                2<br> VG Size               28.18 GB<br> PE Size               32.00 MB<br> Total PE              614<br> Alloc PE / Size       592 / 18.50 GB<br>Free  PE / Size       22 / 704.00 MB<br>VG UUID               qDcbn4-cgXJ-cTeX-mXuh-V06I-AFOU-vZKeEH<br></code></p></pre><p></p>
<p>Extend the Volume Group</p>
<p>Use vgextend to extend the VG identified above to include the new physical volume:</p>
<pre><code>vgextend VolGroup00 /dev/sdb1
</code></pre>
<p>Volume group “VolGroup00” successfully extended<br>Run vgdisplay again and note the increased size (VG Size.)</p>
<pre><code>vgdisplay</code>
<p><code>--- Volume group ---<br>  VG Name               VolGroup00<br>  System ID<br>  Format                lvm2<br>  Metadata Areas        3<br>  Metadata Sequence No  7<br>  VG Access             read/write<br>  VG Status             resizable<br>  MAX LV                0<br>  Cur LV                2<br>  Open LV               2<br>  Max PV                0<br>  Cur PV                3<br>  Act PV                3<br>  VG Size               55.16 GB<br>  PE Size               32.00 MB<br>  Total PE              1765<br>  Alloc PE / Size       592 / 18.50 GB<br>  Free  PE / Size       1173 / 36.66 GB<br>  VG UUID               qDcbn4-cgXJ-cTeX-mXuh-V06I-AFOU-vZKeEH<br></code></p></pre><p></p>
<p>Extend the Logical Volume</p>
<p>Identify the logical volume you wish to extend with lvdisplay</p>
<pre><code>lvdisplay</code>
<p><code>-- Logical volume --<br>  LV Name                /dev/VolGroup00/LogVol00<br>  VG Name                VolGroup00<br>  LV UUID                T7lMW0-W1Rx-zRKw-evZy-tcSp-gSgF-9HAx3w<br>  LV Write Access        read/write<br>  LV Status              available<br>open                 1<br>  LV Size                18.00 GB<br>  Current LE             576<br>  Segments               2<br>  Allocation             inherit<br>  Read ahead sectors     auto<br>  currently set to     256<br>  Block device           253:0<br></code></p></pre><p></p>
<p>Notice the difference in size between the VG and the LV. In this case the volume group has 37 GB of additional space that the logical volume can use.<br>Extend the logical volume to use the space.</p>
<pre><code>lvextend -L 37G /dev/VolGroup00/LogVol00</code>
<p><code>Extending logical volume LogVol00 to 37.00 GB<br>Logical volume LogVol00 successfully resized<br></code></p></pre><p></p>
<p>Resize the Filesystem</p>
<p>The underlying filesystem needs to be expanded to use the additional space using resize2fs.</p>
<pre><code>resize2fs /dev/mapper/VolGroup00-LogVol00
<p>resize2fs 1.39 (29-May-2006)<br>Filesystem at /dev/mapper/VolGroup00-LogVol00 is mounted on /; on-line resizing required<br>Performing an on-line resize of /dev/mapper/VolGroup00-LogVol00 to 9699328 (4k) blocks.<br>The filesystem on /dev/mapper/VolGroup00-LogVol00 is now 9699328 blocks long.<br>Verify the free space with df -h</p>
<p>df -h</p></code><p><code></code></p><p><code>Filesystem            Size  Used Avail Use% Mounted on<br>/dev/mapper/VolGroup00-LogVol00<br>                   36G   16G   19G  47% /<br>/dev/sda1              99M   20M   74M  21% /boot<br>tmpfs                 502M     0  502M   0% /dev/shm<br></code></p><br><br><p></p></pre><p></p>
<p>Find the device using dmesg output</p>
<pre><code>[root@cddsql-bak-sc9 ~]# dmesg | grep -i 'device sdb'</code>
<p><code>SCSI device sdb: 15623782400 512-byte hdwr sectors (7999377 MB)<br>SCSI device sdb: drive cache: write back<br>SCSI device sdb: 15623782400 512-byte hdwr sectors (7999377 MB)<br>SCSI device sdb: drive cache: write back <br></code></p></pre><p></p>
<p>Use parted to create a gpt partition table</p>
<pre><code>parted /dev/sdb 
</code></pre>
<p>Write a gpt partition table</p>
<pre><code>(parted) mklabel gpt
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost.      Do you want to continue?
Yes/No? yes
New disk label type?  [gpt]?  
Display partition table and note the size of the disk, in this case 7999GB
(parted) print                                                            
</code></pre>
<p>Model: DELL PERC H700 (scsi)<br>Disk /dev/sdb: 7999GB<br>Sector size (logical/physical): 512B/512B<br>Partition Table: gpt</p>
<pre><code>Number  Start  End  Size  File system  Name  Flags  
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
<p>Writing inode tables: done                           <br>Creating journal (32768 blocks): done<br>Writing superblocks and filesystem accounting information: done</p></code><p><code></code></p><p><code>This filesystem will be automatically checked every 37 mounts or<br>180 days, whichever comes first.  Use tune2fs -c or -i to override. <br>Write a label to reference the partition in fstab<br>e2label /dev/sdb1 vadata <br>Add entry in /etc/fstab<br>LABEL=vadata            /vadata                  ext3    defaults        0 0<br></code></p><br><br><p></p></pre><p></p>
<p>Finally, create a mount point and mount it.</p>
<pre><code>mkdir /vadata
mount -a 
</code></pre><p></p><br>