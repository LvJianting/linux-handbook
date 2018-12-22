# The usage of LVM

## Overview

### physical volumes

These are your physical disks, or disk partitions, such as /dev/hda or /dev/hdb1. These are what you'd be used to using when mounting/unmounting things. Using LVM we can combine multiple physical volumes into volume groups.

### volume groups

A volume group is comprised of real physical volumes, and is the storage used to create logical volumes which you can create/resize/remove and use. You can consider a volume group as a "virtual partition" which is comprised of an arbitary number of physical volumes.

### logical volumes

These are the volumes that you'll ultimately end up mounting upon your system. They can be added, removed, and resized on the fly. Since these are contained in the volume groups they can be bigger than any single physical volume you might have. (ie. 4x5Gb drives can be combined into one 20Gb volume group, and you can then create two 10Gb logical volumes.)

## Create Logical Volume

1. create physical volume
2. create volume group
3. create logical volume in the volume group
4. build filesystem
5. mount logical volume

```bash
pvcreate /dev/xvde

vgcreate rook /dev/xvde

lvcreate -L 80G -n ceph-cluster rook

mkfs.ext4 /dev/rook/ceph-cluster

mkdir /var/rook/ceph-cluster
mount /dev/rook/ceph-cluster  /var/rook/ceph-cluster/

lvdisplay
```

## Extend Logical Volume

`Note`: Better backup before operating

1. umount
2. extend logical volume
3. check filesystem
4. expand filesystem
5. mount

```bash
df -hl
pvscan

umount /var/rook/ceph-cluster

lvextend -L +5G /dev/rook/ceph-cluster
lvextend -l +100%FREE /dev/rook/ceph-cluster

e2fsck -f /dev/rook/ceph-cluster

resize2fs /dev/rook/ceph-cluster

mount /dev/rook/ceph-cluster /var/rook/ceph-cluster
```

## Shrink Logical Volume

`Note`: Better backup before operating

1. umount
2. check filesystem
3. shrink filesystem
4. shrink logical volume
5. mount

```bash
df -hl
pvscan

umount /var/rook/ceph-cluster

e2fsck -f /dev/rook/ceph-cluster

resize2fs /dev/rook/ceph-cluster

lvresize -L 75G /dev/rook/ceph-cluster
lvereduce -L -5G /dev/rook/ceph-cluster

mount /dev/rook/ceph-cluster /var/rook/ceph-cluster
```

## Add Physical Volume to Logical Volume

```bash
pvcreate /dev/xvda3
vgextend rook /dev/xvda3

umount /var/rook/ceph-cluster
lvextend -l +100%FREE /dev/rook/ceph-cluster
e2fsck -f /dev/rook/ceph-cluster
resize2fs /dev/rook/ceph-cluster
mount /dev/rook/ceph-cluster /var/rook/ceph-cluster
```

## Remove Logical Volume

```bash
```

## Partition

```bash
fdisk /dev/xvda
# p
# F
# n

partprobe
```

## Trouble shooting

### Either the superblock or the partition table is likely to be corrupt

`Reason`: The size of filesystem is not matched with logival volume

1. umount
2. resize logical volume to match the size of filesystem
3. redo extend or shrink

```bash
umount /var/rook/ceph-cluster

lvresize -L 75G /dev/rook/ceph-cluster

# redo extend or shrink
```