############################################
# REDUCE LV SIZE & ADD TO ANOTHER LV (LVM) #
############################################

# Important Notes:
# 1) Backup data before resizing (Risk of Data Loss)
# 2) Make sure the FS on the LV is not FULL (leave space before shrinking)
# 3) Ensure the LV is not mounted while shrinking (except XFS cannot shrink)

===============================================================
STEP-1: Check LVM & Filesystem Details
===============================================================
$ lsblk
$ df -h
$ sudo vgdisplay
$ sudo lvdisplay

===============================================================
STEP-2: Unmount the LV that you want to shrink
===============================================================
$ sudo umount /mnt/test_lv1

# If busy:
$ sudo lsof | grep /mnt/test_lv1
$ sudo fuser -vk /mnt/test_lv1

===============================================================
STEP-3: Run Filesystem Check (Mandatory before shrinking)
===============================================================
# For EXT4 filesystem:
$ sudo e2fsck -f /dev/ubuntu-vg/test_lv1

===============================================================
STEP-4: Shrink Filesystem FIRST
===============================================================
# Example: Reduce by 1GB (set final size as needed)
$ sudo resize2fs /dev/ubuntu-vg/test_lv1 4G     # new desired size

===============================================================
STEP-5: Shrink Logical Volume
===============================================================
# Use same size as filesystem
$ sudo lvreduce -L 4G /dev/ubuntu-vg/test_lv1 -y

# OR shrink by amount:
$ sudo lvreduce -r -L -1G /dev/ubuntu-vg/test_lv1

===============================================================
STEP-6: Extend Another Logical Volume
===============================================================
# Example: Adding 1GB to other LV
$ sudo lvextend -L +1G /dev/ubuntu-vg/test_lv2

===============================================================
STEP-7: Extend Filesystem to Use New Space
===============================================================
# For EXT4
$ sudo resize2fs /dev/ubuntu-vg/test_lv2

===============================================================
STEP-8: Mount Back & Verify
===============================================================
$ sudo mount /dev/ubuntu-vg/test_lv1 /mnt/test_lv1
$ df -h
$ lsblk

###############################################################
# NOTE FOR XFS FILESYSTEM:
# XFS CANNOT SHRINK (You must backup → recreate LV → restore)
###############################################################
# To only extend XFS:
$ sudo lvextend -L +1G /dev/ubuntu-vg/test_lv2
$ sudo xfs_growfs /mnt/test_lv2   # mount path
###############################################################
