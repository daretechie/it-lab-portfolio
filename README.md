# 🧪 Full-stack IT Support & SysAdmin Simulation Lab (VirtualBox)

**This repository contains a comprehensive simulation lab designed to provide hands-on experience in IT support and system administration tasks using VirtualBox. This lab covers user support, server setup, troubleshooting, networking, and basic DevOps tools.**

---

## 🖥️ Lab Overview

| OS         | Role                         |
| ---------- | ---------------------------- |
| Windows 11 | User workstation             |
| Ubuntu     | Admin/DevOps environment     |
| CentOS     | File server & NGINX web host |

---

## 🔧 Tasks Completed

### ✅ 1. Enable VM Communication

- Switched all VMs to **Bridged Adapter** mode for full inter-VM, host, and internet connectivity.
- **Issue**: NAT prevented ping between VMs.
- **Resolution**: Changed to Bridged Adapter.
- **Commands/Actions**:

  ```bash
  # In VirtualBox GUI: Settings → Network → Bridged Adapter → Select host interface
  # Inside each VM:
  ip a        # Verify IP address
  ping <other_VM_IP>
  ```

<!-- Screenshot: Bridged adapter settings and successful ping output -->

### ✅ 2. File Server on CentOS with Samba

- Installed and configured Samba for guest and authenticated shares.
- Created `/srv/samba/shared` and shared via `[SharedFiles]` section in `/etc/samba/smb.conf`.
- **Issue**: `chown nobody:nogroup` failed; no `nogroup` on CentOS.
- **Resolution**: Used `nobody:nobody` instead.
- **Commands**:

  ```bash
  sudo dnf install samba samba-client samba-common -y
  sudo systemctl enable smb --now
  sudo mkdir -p /srv/samba/shared
  sudo chown -R nobody:nobody /srv/samba/shared
  sudo chmod -R 0775 /srv/samba/shared
  # smb.conf share block:
  [SharedFiles]
     path = /srv/samba/shared
     browsable = yes
     writable = yes
     guest ok = yes
     map to guest = Bad User
  sudo firewall-cmd --permanent --add-service=samba
  sudo firewall-cmd --reload
  sudo systemctl restart smb nmb
  ```

- **Test**: Mapped drive in Windows Explorer to `\\192.168.43.10\SharedFiles`.

<!-- Screenshot: Windows mapped network drive dialog and folder contents -->

### ✅ 3. SSH Access Across Machines

- Enabled SSH on Ubuntu (`openssh-server`) and CentOS (`sshd`).
- **Commands**:

  ```bash
  # Ubuntu:
  sudo apt update && sudo apt install openssh-server -y
  sudo systemctl enable ssh --now
  # CentOS:
  sudo dnf install openssh-server -y
  sudo systemctl enable sshd --now
  sudo firewall-cmd --permanent --add-service=ssh && sudo firewall-cmd --reload
  ```

- Connected from Windows PowerShell and PuTTY.
- Set up SSH key-based login:

  ```powershell
  # On Windows:
  ssh-keygen
  type $env:USERPROFILE\.ssh\id_rsa.pub |
    ssh user@<VM_IP> "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
  ```

- **Test**: `ssh user@<VM_IP>` without password.

<!-- Screenshot: PowerShell SSH login and key-based login success -->

### ✅ 4. Host a Website on Ubuntu (Apache)

- Installed Apache2, created custom `index.html` & `styles.css`.
- Installed PHP (`libapache2-mod-php`) and verified `info.php`.
- **Commands**:

  ```bash
  sudo apt install apache2 libapache2-mod-php php -y
  sudo systemctl enable --now apache2
  # HTML & CSS:
  cd /var/www/html
  sudo mv index.html index.html.bak
  # index.html, styles.css, info.php created
  ```

- **Test**:

  - `http://<Ubuntu_IP>/` shows styled page.
  - `http://<Ubuntu_IP>/info.php` shows PHP info.

<!-- Screenshot: Styled page and PHP info page -->

### ✅ 5. Host the Same Site on CentOS (NGINX & PHP-FPM)

- Installed NGINX and PHP-FPM, configured `/etc/nginx/conf.d/default.conf`.
- **Fixed**: Placed `location` blocks inside `server {}`. Tested with `nginx -t`.
- **Commands**:

  ```bash
  sudo dnf install nginx php php-fpm -y
  sudo systemctl enable --now nginx php-fpm
  sudo firewall-cmd --permanent --add-service=http && sudo firewall-cmd --reload
  # default.conf server block updated
  sudo nginx -t && sudo systemctl restart nginx
  ```

- Copied files via `scp` and set proper ownership.
- **Test**: `http://<CentOS_IP>/` and `http://<CentOS_IP>/form.html`.

<!-- Screenshot: NGINX served styled page and form.html response -->

---

## 🛠️ Issues & Resolutions

| Task                              | Issue                                               | Resolution                        |
| --------------------------------- | --------------------------------------------------- | --------------------------------- |
| File Server (chown)               | `invalid group: nobody:nogroup`                     | Use `nobody:nobody`               |
| NGINX location directive position | `location directive is not allowed here`            | Moved `location` inside `server`  |
| Permission denied on NGINX folder | Cannot write to `/usr/share/nginx/html` as non-root | Use `scp` to home, then `sudo mv` |
| 404 on form.html                  | Files not in `/usr/share/nginx/html`                | Copied files; set correct root    |

---

### ✅ 6. Linux User Management (Ubuntu & CentOS)

- **Users Created**: `alice`, `bob`
- **Groups**:

  - Created group `devs`
  - Added users with `usermod -aG devs alice`

- **Password Policies**: Used `chage` to set expiry dates, min/max days
- **Sudo Permissions**:

  - **Ubuntu**: Added `alice` to `sudo` group
  - **CentOS**: Added `alice` to `wheel` group; edited `sudo visudo` to uncomment:

    ```bash
    %wheel ALL=(ALL) ALL
    ```

#### 🔐 Lock/Unlock User Accounts

```bash
sudo passwd -l alice    # Locks the account
sudo passwd -u alice    # Unlocks the account
```

#### ❌ User Deletion Issue

- Tried deleting user `alice`:

  ```bash
  sudo userdel alice
  userdel: user alice is currently used by process 2113
  ```

- **Resolution**:

  ```bash
  sudo kill -9 2113
  sudo userdel -r alice
  ```

> Confirmed deletion after killing user’s active session process.

<!-- Screenshot: `useradd`, `passwd`, `usermod`, locked shell output -->

---

## 🧠 Key Skills Practiced

- Networking with VirtualBox
- File sharing with Samba
- SSH remote access and key setup
- LAMP/NGINX website hosting
- Linux user/group management

---

🔗 **Connect with me:** [LinkedIn](#) | [GitHub](#)
