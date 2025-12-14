===================================================================
 Guide: How to Generate SSH Private Key for Ubuntu Server Access
===================================================================

SSH Authentication uses a Key Pair:
- Private Key  → Stored securely on your local machine
- Public Key   → Shared with the Ubuntu Server

This replaces password authentication with secure certificate access.

===================================================================
1️⃣ Generate a New SSH Key Pair (Certificate)
===================================================================

On Linux / macOS / Windows PowerShell:

Recommended:
ssh-keygen -t ed25519 -C "tejastarghale47@gmail.com"

(OR older RSA format)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

Press ENTER for default file path:
~/.ssh/id_ed25519

Enter passphrase (optional but recommended)
This enhances security.

Key Pair Generated:
---------------------------------------------
~/.ssh/id_ed25519            -> Private Key
~/.ssh/id_ed25519.pub        -> Public Key
---------------------------------------------

===================================================================
2️⃣ Secure Your Private Key
===================================================================

Run:
chmod 600 ~/.ssh/id_ed25519

Purpose:
Ensures nobody else can read your private key.

===================================================================
3️⃣ Upload Public Key to Ubuntu Server
===================================================================

Option A: If server allows password login
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@<server-ip>

Option B: Manual Upload
scp ~/.ssh/id_ed25519.pub ubuntu@<server-ip>:/home/ubuntu/
ssh ubuntu@<server-ip>
mkdir -p ~/.ssh
cat id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

===================================================================
4️⃣ Connect Using Your Private Key
===================================================================

ssh -i ~/.ssh/id_ed25519 ubuntu@<server-ip>

If Azure VM User:
ssh -i ~/.ssh/id_ed25519 azureuser@<server-ip>

If AWS Ubuntu AMI:
ssh -i ~/.ssh/id_ed25519 ubuntu@<server-ip>

===================================================================
 Supported Key Types (Recommendation Chart)
===================================================================

| Key Type  | Security | Performance | Best Use |
|-----------|----------|-------------|----------|
| ED25519   | ⭐⭐⭐⭐⭐  | ⭐⭐⭐⭐⭐   | Modern, Strong Security |
| RSA-4096  | ⭐⭐⭐⭐    | ⭐⭐⭐     | Legacy Compatibility   |

Recommendation: **Use ED25519**

===================================================================
 Important Notes
===================================================================
✔ Never share your private key
✔ Backup key securely to encrypted storage
✔ Public key can be freely shared to servers
✔ Private key may include a passphrase for extra safety

===================================================================
END OF DOCUMENT
===================================================================
