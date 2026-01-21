
======================= To remove logical volume =======================

# Step 1: Unmount the Logical Volume if it is mounted
[root@rhel ~]# umount /mnt/newdata
# Replace /mnt/newdata with the actual mount point

# Step 2: Remove the Logical Volume
[root@rhel ~]# lvremove /dev/vgname/lvname
# Replace vgname and lvname with actual names
# It will ask for confirmation: type y

Output : 

root@test:/mnt# lvremove /dev/ubuntu-vg/test_lv2
Do you really want to remove and DISCARD active logical volume ubuntu-vg/test_lv2? [y/n]: t
  WARNING: Invalid input 't'.
Do you really want to remove and DISCARD active logical volume ubuntu-vg/test_lv2? [y/n]: y
  Logical volume "test_lv2" successfully removed.

# Step 3: Verify the LV is removed
[root@rhel ~]# lvs
# The logical volume should no longer appear

========================================================================