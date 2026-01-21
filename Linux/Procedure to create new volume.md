
=================== How to create New Physical Volume =================================

# Step 1: Identify the disk or free partition where you want to create LVM
[root@rhel ~]# lsblk
# Example output will show disks like /dev/sdb

# Step 2: Create a physical volume (PV) on the disk or partition
[root@rhel ~]# pvcreate /dev/sdb
# /dev/sdb becomes an LVM physical volume

# Step 3: Create a volume group (VG)
[root@rhel ~]# vgcreate vgdata /dev/sdb
# Here "vgdata" is the name of the new Volume Group

# Step 4: Create a logical volume (LV)
[root@rhel ~]# lvcreate -L 20G -n lvdata vgdata
# Creates LV named "lvdata" of size 20GB in VG "vgdata"
# You can replace 20G with the size you want

# Step 5: Format the logical volume with a filesystem (ext4 example)
[root@rhel ~]# mkfs.ext4 /dev/vgdata/lvdata
# Format so it can store files

# Step 6: Create a mount directory
[root@rhel ~]# mkdir /mnt/data
# This folder will be used to access the LV

# Step 7: Mount the logical volume
[root@rhel ~]# mount /dev/vgdata/lvdata /mnt/data
# The LV is now mounted and ready to use

# Step 8: Verify
[root@rhel ~]# df -h
# You will see the new LV mounted under /mnt/data

==============================================================================================