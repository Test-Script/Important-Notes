===============================================================================================================
1️⃣ PREREQUISITES
================================================================================================================

✔ Linux System (Ubuntu/CentOS/RHEL etc.)
✔ Install Azure CLI

curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash   # Ubuntu/Debian

# RHEL / CentOS:
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo tee /etc/yum.repos.d/azure-cli.repo <<EOF
[azure-cli]
name=Azure CLI
baseurl=https://packages.microsoft.com/yumrepos/azure-cli
enabled=1
gpgcheck=1
gpgkey=https://packages.microsoft.com/keys/microsoft.asc
EOF
sudo yum install azure-cli

✔ Login to Azure

az login

===============================================
2️⃣ ACCESS AZURE **BLOB STORAGE**
===============================================

🅐 Install **AzCopy** for file upload/download

wget https://aka.ms/downloadazcopy-v10-linux -O azcopy.tar.gz
tar -xvf azcopy.tar.gz
sudo cp ./azcopy_linux*/azcopy /usr/local/bin/
azcopy --version

🅑 Generate SAS URL (valid for limited time)

az storage blob generate-sas \
  --account-name <STORAGE_ACCOUNT_NAME> \
  --container-name <CONTAINER> \
  --name <BLOB_FILE> \
  --permissions r \
  --expiry 2025-12-31T23:59:00Z \
  --output tsv

🅒 Download/Upload Blob

# Download blob to Linux
azcopy copy "<SAS_BLOB_URL>" "/home/user/downloaded_file"

# Upload file to blob
azcopy copy "/home/user/file.txt" "<SAS_CONTAINER_URL>"

📌 **Blob is object storage → cannot mount like a local disk.**
Use Azure Files if you want direct mount.

========================================
3️⃣ ACCESS AZURE **FILE SHARE (SMB)**
(Works like a shared drive in Linux)
====================================

🅐 Install SMB client

sudo apt install cifs-utils  # Ubuntu
# or
sudo yum install cifs-utils  # RHEL/CentOS

🅑 Create mount folder

r -p /mnt/azfiles

🅒 Get Storage Account Key

az storage account keys list \
  --resource-group <RG_NAME> \
  --account-name <STORAGE_ACCOUNT_NAME> \
  -o table

Copy the `key1` value.

🅓 Mount Azure File Share

sudo mount -t cifs \
  // <STORAGE_ACCOUNT_NAME>.file.core.windows.net/<FILESHARE_NAME> \
  /mnt/azfiles \
  -o vers=3.0,username=<STORAGE_ACCOUNT_NAME>,password=<STORAGE_ACCOUNT_KEY>,dir_mode=0777,file_mode=0777,serverino

✔ Open folder

cd /mnt/azfiles
ls -al

📌 Now you can **copy/edit/delete files** from your Linux system directly.

🅔 Make the mount **permanent**
(Add to `/etc/fstab`)

//<STORAGE_ACCOUNT_NAME>.file.core.windows.net/<FILESHARE_NAME> /mnt/azfiles cifs vers=3.0,username=<STORAGE_ACCOUNT_NAME>,password=<STORAGE_ACCOUNT_KEY>,dir_mode=0777,file_mode=0777,serverino 0 0

========================================
4️⃣ ACCESS AZURE **FILE SHARE (NFS)**
(If NFS protocol enabled)
========================================

🅐 Install NFS utilities


sudo apt install nfs-common
# or
sudo yum install nfs-utils

🅑 Mount NFS

sudo mkdir /mnt/azfilesnfs

sudo mount \
  -t nfs \
  <STORAGE_ACCOUNT_NAME>.file.core.windows.net:/<FILESHARE_NAME> \
  /mnt/azfilesnfs \
  -o vers=4,minorversion=1,sec=sys

========================================
5️⃣ Decision Guide
========================================

| Requirement                  | Best Option                |
| ---------------------------- | -------------------------- |
| Upload / download files only | Azure Blob + AzCopy        |
| Mount like a local drive     | Azure File Share (SMB/NFS) |
| Scripting automation         | Azure CLI / AzCopy         |
| Access VM disk               | Managed Disk → Export VHD  |