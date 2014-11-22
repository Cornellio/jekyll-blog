---
layout: post
title:  "Responsive LVM"
date:   2014-03-12 16:02:00
categories: linux lvm storage
---


In the Linux world, rebooting is kind of a sin.

<p>One of the nice things about Linux is it's agility and stability in the face of changes and the fact that it rarely needs rebooting. Possessing the right knowledge allows us sysadmins to be responsive and adapt our systems to change without rebooting.</p>
<p>The Logical Volume Manager provides a powerful storage subsystem in Linux, including advanced features like spanning of partitions across disks, live resizing of volumes and taking snapshots.</p>
<p>This page covers the basic concepts of LVM and demonstrates a way to resize a volume without rebooting the system, even the root volume. The last bit is key. The power and flexibility of Linux allows making significant changes to the system while it is running, yet certain subtleties must be understood to accomplish certain changes on a live system.</p>
<h2 class="">Logical Volume Manager</h2>
<p>First, I’ll outline the basics of LVM then move onto a step by step example of resizing a root volume.</p>

## Boxes within Boxes

<p>LVM can be thought of as a hierarchy of containers, or boxes within boxes. The physical volumes contain volume groups, which in turn contain logical volumes. As such, the basic steps in creating storage is to create physical volumes on one or more block storage devices, create volume groups than span one or more physical volumes, then create logical volumes within volume groups.</p>

## Getting Deeper

There are a lot of lvm commands available.

* pv*
* vg*
* lv*
 
Try them out, check the man pages and discover yet more capabilities of this storage system. Here are the 
most useful commands.

* pvs
* pvcreate
* pvdisplay
* pvremove
* pvresize
* vgs
* vgcreate
* vgdisplay
* vgremove
* vgextend
* lvs
* lvcreate
* lvdisplay
* lvreduce

====================


