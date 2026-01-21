================================ To Shrink The Logical Volume ==================================

[root@rhel ~]# umount /mnt/data
# Unmount the filesystem before resizing. Replace /mnt/data with your actual mount point.

[root@rhel ~]# e2fsck -f /dev/vgname/lvname
# Check and repair filesystem to ensure no corruption before shrinking.

[root@rhel ~]# resize2fs /dev/vgname/lvname 20G
# Shrink the filesystem first. Replace 20G with your new required size.
# IMPORTANT: New size must be greater than used space.

[root@rhel ~]# lvreduce -L 20G /dev/vgname/lvname
# Now reduce the Logical Volume to match filesystem size.
# If this step is done before resize2fs, data loss will occur.

[root@rhel ~]# e2fsck -f /dev/vgname/lvname
# Run filesystem check again for consistency.

[root@rhel ~]# mount /dev/vgname/lvname /mnt/data
# Remount your filesystem after the resizing process completes.

[root@rhel ~]# df -h
# Verify the logical volume has been reduced successfully.

================================================================================================================

Important warning: Never shrink logical volume before shrinking filesystem or you will lose data. Backup recommended. XFS filesystem cannot be reduced.

================================================================================================================

=============================== To get the Logical Volume Path ===========================

[root@rhel ~]# lsblk
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                 8:0    0  100G  0 disk
├─sda1              8:1    0    1G  0 part /boot
└─sda2              8:2    0   99G  0 part
  ├─rhel-root     253:0    0   80G  0 lvm  /
  └─rhel-home     253:1    0   19G  0 lvm  /home

[root@rhel ~]# lvs
  LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rhel -wi-ao---- 80.00g
  home rhel -wi-ao---- 19.00g

[root@rhel ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/rhel/root
  LV Name                root
  VG Name                rhel
  LV Size                80.00 GiB

===========================================================================================