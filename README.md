# 🔐 Cross-Platform SSH Lab: Linux and Windows

This project demonstrates how to set up and use SSH (Secure Shell) across both Linux and Windows environments. My goal for this lab is to get hands-on experience with secure remote access, user authentication, basic SSH hardening, and file transfer operations — essential tasks in both system administration and cybersecurity roles.

---

## 📌 Objectives

- Set up SSH server and client on Ubuntu and Windows 11
- Connect between systems using password and SSH key authentication
- Harden SSH configuration on Linux
- Practice secure file transfer using SFTP

---

## 🧰 Tools Used

- Ubuntu Desktop / Server (22.04 LTS)
- Windows 11 (PowerShell + OpenSSH Client/Server)
- `ssh`, `sshd`, `sftp`, `ufw`, `PowerShell`

---

## 📂 Documentation Breakdown

- [`linux/linux-ssh-server-setup.md`](linux/linux-ssh-server-setup.md): SSH server installation and setup on Ubuntu Server
- [`linux/ssh-hardening.md`](linux/linux-ssh-hardening.md): SSH hardening steps including systemd override fix
- [`windows/windows_ssh_setup_and_sftp.md`](windows/windows_ssh_setup_and_sftp.md): OpenSSH server installation, SSH client usage via PowerShell and SFTP

---

## 🗂️ Repository Structure

```plaintext
ssh-lab/
├── README.md
├── linux/
│   ├── screenshots/
│   ├── linux-ssh-server-setup.md
│   ├── linux-ssh-hardening.md
└── windows/
    ├── screenshots/
    ├── openssh-installation.md
    ├── powershell-ssh.md
    └── putty-usage.md

```

