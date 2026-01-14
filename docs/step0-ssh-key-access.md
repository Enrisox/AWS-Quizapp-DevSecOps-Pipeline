# SSH Key-Based Authentication

**1. Generate SSH key pair on Windows**

Open PowerShell on Windows:

```powershell
ssh-keygen -t ed25519 -C "windows-to-raspberry"
```
When prompted:

- Press Enter to accept the default path
- Optionally set a passphrase 

This creates:

- Private key → ~/.ssh/id_ed25519
- Public key → ~/.ssh/id_ed25519.pub

View the public key:
```bash
type $env:USERPROFILE\.ssh\id_ed25519.pub
```
Copy the entire line (starts with ssh-ed25519).

**2. Access the Raspberry Pi via ssh remotely or locally with monitor and keyboard**

**Prepare SSH directory on linux server**

```bash
cd ~
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

**4. Add the public key to authorized_keys**
```bash
nano ~/.ssh/authorized_keys
```

Paste the public key copied from Windows, then save and exit.

**Set correct permissions:**

```bash
chmod 600 ~/.ssh/authorized_keys    #owner can read and write only, no execution
```
**5. Test SSH key authentication**

If login works without asking for a password, SSH key authentication is correctly configured.

**5. Hardening SSH security**

Edit SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```
Set:

```bash
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
```

- This action eliminates password-based brute-force attacks
- Private key never leaves the client machine
- Strong file permissions (600) prevent key leakage
- Follows Linux and DevSecOps security best practices

**Restart SSH:**

```bash
sudo systemctl restart ssh
```


