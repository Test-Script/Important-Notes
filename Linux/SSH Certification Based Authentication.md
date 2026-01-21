===================================================================
 GUIDE: Access Ubuntu Server using PuTTY with SSH Certificate (Keys)
===================================================================

Client: Windows (PuTTY / PuTTYgen)
Server: Ubuntu Linux

PuTTY uses .PPK key format, not OpenSSH format.

===================================================================
1️⃣ Generate Private Key using PuTTYgen
===================================================================

Steps:
1. Open PuTTYgen
2. Key type: Choose "ED25519" (Recommended)
   (Alternative: RSA 4096-bit)
3. Click "Generate" and move cursor until complete
4. Save the private key:
   Save Private Key → mykey.ppk
5. Copy the Public Key from PuTTYgen window

Protect with passphrase (optional, better security)

===================================================================
2️⃣ Install Public Key on Ubuntu Server
===================================================================

Requirements:
- Temporary password login
- Username: ubuntu (AWS Ubuntu AMI)
          azureuser (Azure VM)
          or your custom user

Commands:
---------------------------------------------
ssh ubuntu@<server-ip>
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
---------------------------------------------

Paste the Public Key from PuTTYgen

Save & Close:
CTRL + O → Enter
CTRL + X

Set correct permissions:
---------------------------------------------
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
---------------------------------------------

===================================================================
3️⃣ Connect to Ubuntu using PuTTY (Private Key)
===================================================================

Open PuTTY → Left panel settings:

Host Name:
---------------------------------------------
ubuntu@<server-ip>
---------------------------------------------

Go to:
Connection → SSH → Auth → Credentials

Browse → Select: mykey.ppk

Back to Session → Save session → Open

If first time: Accept the fingerprint

You should now log in without a password!

===================================================================
4️⃣ Delete SSH Keys (if needed)
===================================================================

Delete keys from local system:
---------------------------------------------
Windows Explorer path:
C:\Users\<YourUser>\Documents\ or .ssh folder
Delete: mykey.ppk
---------------------------------------------

To revoke access from server:
---------------------------------------------
ssh ubuntu@<server-ip>
nano ~/.ssh/authorized_keys
Remove the specific key line
sudo systemctl restart ssh  (optional)
---------------------------------------------

===================================================================
5️⃣ Troubleshooting
===================================================================

Error: "Server refused our key"
Cause: Wrong username / wrong public key
Fix: Ensure correct Ubuntu default username

Error: "No supported authentication methods available"
Cause: Key not loaded in PuTTY
Fix: Choose correct PPK under SSH → Auth → Credentials

Error: Permission denied
Cause: Incorrect ~/.ssh permission
Fix:
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

===================================================================
 Summary
===================================================================
| File                     | Location / Owner  | Purpose               |
|-------------------------|------------------|-----------------------|
| mykey.ppk               | Local (Windows)  | Private Key (secret)  |
| authorized_keys         | Ubuntu Server    | Allows access         |
| Public Key (copied)     | Added to server  | Trusted certificate   |

✔ Private key stays on Windows only  
✔ Public key goes to Ubuntu server only  
✔ No password required once configured

===================================================================
 END OF DOCUMENT
===================================================================
