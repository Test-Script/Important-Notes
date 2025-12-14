
================= To Rename The Logical Volume ============================

# Step 1: Check current Logical Volume details
[root@rhel ~]# lvs
# Identify the current LV path (Example: /dev/vgname/oldlv)

---------------------------------------------------------------------------

# Step 2: Rename the Logical Volume
[root@rhel ~]# lvrename /dev/vgname/oldlv newlv
# Replace oldlv with the current LV name
# Replace newlv with the new name you want

---------------------------------------------------------------------------

# Step 3: Verify the rename
[root@rhel ~]# lvs
# You should now see the LV with the new name
# Note:
# If the LV is mounted, update /etc/fstab with the new LV path before reboot

============================================================================

Extra Infomation :

# If the LV is mounted, update /etc/fstab with the new LV path before reboot
[root@rhel ~]# vi /etc/fstab
# Update the old LV entry to the new LV name
# Example:
# FROM:
/dev/vgname/oldlv  /mnt/data  ext4  defaults  0 0
# TO:
/dev/vgname/newlv  /mnt/data  ext4  defaults  0 0

# Save and exit the file (:wq in vi)

# Step: Re-mount all filesystems to test the change
[root@rhel ~]# mount -a
# If no error, fstab update is correct

======================== Justification of changes to fstab ===================================

# Yes. If the LV is mounted using its LV path in /etc/fstab, you MUST update it.
# Because the path will change after LV rename.

# Example: Before rename
/dev/vgname/oldlv  /mnt/data  ext4  defaults  0 0

# After LV rename, update like this:
/dev/vgname/newlv  /mnt/data  ext4  defaults  0 0

# Then run:
[root@rhel ~]# mount -a
# If there are no errors, your fstab entry is correct.

# Note:
# If the /etc/fstab entry uses UUID or LABEL instead of LV path,
# then no need to update.

# Your /etc/fstab uses device UUID links (dm-uuid and by-uuid)
# This means your mounts do NOT depend on the logical volume NAME.

# So even if you rename the LV, the UUID stays the same.
# Therefore, NO CHANGE is required in fstab.

# In short:
# LV rename → UUID remains same → fstab entry still valid → No update needed

# You can verify UUID of your LV with:
root@test:/mnt# blkid

# If the same UUID appears in fstab, everything is fine.

=============================================================================================

root@test:/mnt# blkid
/dev/mapper/dm_crypt-0: UUID="0PWtrN-T2TS-Z1nN-uvTa-dCBC-Jo2b-ttiuQa" TYPE="LVM2_member"
/dev/mapper/ubuntu--vg-ubuntu--lv: UUID="981cf935-0076-45ab-994e-0dbcd9f5b697" BLOCK_SIZE="4096" TYPE="ext4"
/dev/sr0: BLOCK_SIZE="2048" UUID="2025-08-05-23-54-07-00" LABEL="Ubuntu-Server 24.04.3 LTS amd64" TYPE="iso9660" PTTYPE="PMBR"
/dev/sda2: UUID="109a6e99-f023-4542-97bf-2a020e78135b" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="ffc08a6a-b906-4ea8-8b19-a5ebd0ef677b"
/dev/sda3: UUID="ed0e0613-ad8a-4140-9ce8-809b273da93c" TYPE="crypto_LUKS" PARTUUID="f6047f63-9a79-4d78-b9ff-87a5032ed7bc"
/dev/mapper/ubuntu--vg-test_lv1: UUID="1e317b60-d20a-4d0b-a811-79dd8c0a7ab2" BLOCK_SIZE="4096" TYPE="ext4"
/dev/sda1: PARTUUID="32ab63c2-63e7-4def-9f99-81aab6bfb407"

=============================================================================================