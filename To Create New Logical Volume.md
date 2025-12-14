
====================================== How to create new logical volume ========================================
# Step 1: Check available Volume Groups and free space
[root@rhel ~]# vgs
# This shows which VG has free space to create a new LV

Output : 

root@test:/mnt# vgs
  VG        #PV #LV #SN Attr   VSize  VFree
  ubuntu-vg   1   2   0 wz--n- 27.98g 7.99g

--------------------------------------------------------------------------

# Step 2: Create a new Logical Volume
[root@rhel ~]# lvcreate -L 10G -n lvnew vgname
# Replace 10G with required size
# Replace lvnew with logical volume name you want
# Replace vgname with your existing Volume Group name

Output : 

root@test:/mnt# lvcreate -L 5 -n test_lv2 ubuntu-vg
  Rounding up size to full physical extent 8.00 MiB
  Logical volume "test_lv2" created.

--------------------------------------------------------------------------

# Step 3: Format the new LV with a filesystem (example: ext4)
[root@rhel ~]# mkfs.ext4 /dev/vgname/lvnew
# This prepares it to store data

Output :

root@test:/mnt# mkfs.ext4 /dev/ubuntu-vg/test_lv2
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 2048 4k blocks and 2048 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

--------------------------------------------------------------------------

# Step 4: Create a mount point directory
[root@rhel ~]# mkdir /mnt/newdata
# This is where you will access the LV

--------------------------------------------------------------------------

# Step 5: Mount the new logical volume
[root@rhel ~]# mount /dev/vgname/lvnew /mnt/newdata
# LV is now mounted and usable

--------------------------------------------------------------------------

# Step 6: Verify the new LV
[root@rhel ~]# df -h
# You will see /dev/vgname/lvnew mounted on /mnt/newdata

===========================================================================

/dev/mapper/ubuntu--vg-test        5.9G   24K  5.5G   1% /mnt/test_lv1
/dev/mapper/ubuntu--vg-test_lv2    172M   24K  158M   1% /mnt/test_lv2


# The “24K used” you are seeing is NORMAL.
# You did not store any files, but the filesystem itself requires some space.

# When you create a filesystem like ext4:
# - Metadata
# - Superblock
# - Journaling information
# - Inode tables
# - Filesystem structures

# All of these take a small amount of space, even before you add any data.

# That is why you see something like:
5.9G total, 24K used, 5.5G available
172M total, 24K used, 158M available

# This 24K is filesystem overhead — not user data.

# You can confirm by checking the filesystem metadata usage:
root@test:~# tune2fs -l /dev/mapper/ubuntu--vg-test_lv2 | grep 'Block count\|Reserved'

# Also check inode usage:
root@test:~# df -i /mnt/test_lv2

# Conclusion:
# 24K used after mkfs is expected. Nothing to worry.

=============================================================================================


Summary :

[root@rhel ~]# lvcreate -L 50 -n lvname volumegroup
[root@rhel ~]# mkfs.ext4 /dev/ubuntu-vg/test_v2
[root@rhel ~]# mkdir -p /test_v2
[root@rhel ~]# mount /dev/ubuntu-vg/test_v2 /test_v2
[root@rhel ~]# df -h | grep test_v2