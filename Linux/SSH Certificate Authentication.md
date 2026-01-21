===================================================================
 Guide: Access Ubuntu Server using SSH Certificate Authentication
===================================================================

There are 2 main methods:

1️⃣ Standard SSH Key Based Access (Personal / Cloud VMs)
2️⃣ SSH CA-Signed Certificate (Enterprise / AD / Fleet Servers)

-------------------------------------------------------------------
1️⃣ Standard SSH Certificate Login (SSH Keys)
-------------------------------------------------------------------

Step 1: Generate Private + Public Certificate (Key Pair)
-------------------------------------------------------
ssh-keygen -t ed25519 -C "ubuntu-access-cert"

This creates:
~/.ssh/id_ed25519            -> Private Certificate (keep safe)
~/.ssh/id_ed25519.pub        -> Public Certificate (upload to server)

-------------------------------------------------------
Step 2: Install Public Certificate on Ubuntu Server
-------------------------------------------------------

Option A: If password login works
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@<server-ip>

Option B: Manual method
scp ~/.ssh/id_ed25519.pub ubuntu@<server-ip>:/home/ubuntu/
ssh ubuntu@<server-ip>
mkdir -p ~/.ssh
cat id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

-------------------------------------------------------
Step 3: Connect using Certificate
-------------------------------------------------------
ssh -i ~/.ssh/id_ed25519 ubuntu@<server-ip>

===================================================================
2️⃣ SSH CA-Signed Certificate Method (For Organizations)
===================================================================

Used when many users access many servers.
Server trusts the CA instead of individual keys.

-------------------------------------------------------
Step 1: User creates a key
-------------------------------------------------------
ssh-keygen -t ed25519 -C "user@company"

-------------------------------------------------------
Step 2: CA Admin signs the public key
-------------------------------------------------------
ssh-keygen -s ca_key -I user_id -n ubuntu id_ed25519.pub

This creates:
id_ed25519-cert.pub -> Signed Certificate

-------------------------------------------------------
Step 3: Server trusts the CA
-------------------------------------------------------
Edit the SSH configuration on server:
/etc/ssh/sshd_config

Add:
TrustedUserCAKeys /etc/ssh/ca.pub

Restart SSH:
sudo systemctl restart ssh

-------------------------------------------------------
Step 4: User Connects with Certificate
-------------------------------------------------------
ssh -i ~/.ssh/id_ed25519 -o CertificateFile=id_ed25519-cert.pub ubuntu@<server-ip>

===================================================================
 Important Notes
===================================================================
✔ Never share private key (id_ed25519)
✔ Backup securely
✔ Always set correct permissions:
  chmod 600 ~/.ssh/id_ed25519
✔ Certificate authentication is more secure than passwords

===================================================================
END OF DOCUMENT
===================================================================
