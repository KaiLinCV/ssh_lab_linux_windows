# 🔐 Cross-Platform SSH Lab: Linux and Windows

This project demonstrates how to set up and use SSH (Secure Shell) across both Linux and Windows environments. My goal for this lab is to get hands-on experience with secure remote access, user authentication, basic SSH hardening, and file transfer operations — essential tasks in both system administration and cybersecurity roles.

---

## 📌 Objectives

- Set up SSH server and client on Ubuntu and Windows 11
- Connect between systems using password and SSH key authentication
- Harden SSH configuration on Linux
- Practice secure file transfer using SCP
- Document both GUI and CLI usage for cross-platform familiarity

---

## 🧰 Tools Used

- Ubuntu Desktop / Server (22.04 LTS)
- Windows 10/11 (PowerShell + OpenSSH Client/Server)
- PuTTY (optional, for GUI-based SSH on Windows)
- Visual Studio Code (Remote SSH Plugin)
- `ssh`, `sshd`, `scp`, `ufw`, `PowerShell`

---

## 🗂️ Repository Structure

```plaintext
ssh-lab/
├── README.md
├── linux/
│   ├── ssh-server-setup.md
│   ├── ssh-client-usage.md
│   ├── ssh-hardening.md
├── windows/
│   ├── openssh-installation.md
│   ├── powershell-ssh.md
│   └── putty-usage.md
├── screenshots/
│   ├── linux-ssh-login.png
│   ├── windows-powershell-ssh.png
│   └── config-example.png
└── config/
    ├── sshd_config_linux_example
    └── authorized_keys_example
```

---

---

## 📂 Documentation Breakdown

- [`linux/linux-ssh-server-setup.md`](linux/linux-ssh-server-setup.md): SSH server installation and setup on Ubuntu Server
- [`linux/ssh-hardening.md`](linux/linux-ssh-hardening.md): SSH hardening steps including systemd override fix
- [`windows/openssh-installation.md`](windows/openssh-installation.md): OpenSSH server installation and setup on Windows
- [`windows/powershell-ssh.md`](windows/powershell-ssh.md): SSH client usage via PowerShell
- [`windows/putty-usage.md`](windows/putty-usage.md): (Optional) GUI-based SSH using PuTTY

## 📸 Screenshots

All screenshots used in documentation can be found in the [`screenshots/`](screenshots/) folder.

