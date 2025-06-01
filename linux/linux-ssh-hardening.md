# 🔒 SSH Hardening on Ubuntu Server

This document outlines the process and purpose of hardening SSH access on my Ubuntu Server as part of a hands-on learning lab.

---

## 🛡️ Why SSH Hardening?

Hardening SSH is a critical step in securing remote server access. For this project, my goal was to practice strengthening system security and to simulate the responsibilities of a system administrator protecting digital infrastructure from unauthorized access.

By modifying the SSH configuration and applying firewall rules, I aimed to:

- Reduce potential attack vectors  
- Practice real-world system administration skills  
- Deepen my understanding of how SSH and system services interact

---

## 🔧 Hardening Process

### 1. Back Up the SSH Config File (Ubuntu Server)

All configuration changes in this section are performed on the **Ubuntu Server**, since this is the system being secured through SSH hardening.

Before making any changes to the config file, I made sure to create a backup. While preparing for this, I noticed there are two different SSH configuration files:

- `ssh_config`
- `sshd_config`

Out of curiosity, I looked up the difference between them to understand which one I should be editing.

**Why `sshd_config` and not `ssh_config`?**

- `sshd_config` controls the behavior of the **SSH server daemon (`sshd`)**, including listening ports, authentication rules, and session restrictions.
- `ssh_config` configures **SSH client behavior** — for example, how your system initiates SSH connections to other machines.

By learning the difference between these two files, I gained a better understanding of where to apply configuration changes. Since I'm securing the **Ubuntu Server**, it makes sense to work with the `sshd_config` file.

I then backed up the original SSH daemon configuration file with:
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```


---

### 2. Modify SSH Config Settings

I opened the SSH configuration file using vim:

```bash
sudo vim /etc/ssh/sshd_config
```

Changes made:

**Change SSH Port:**  
  ```diff
  - #Port 22
  + Port 2222

  ```
When most people think of SSH, port 22 is the default that comes to mind. From an attacker's point of view, it's typically the first port scanned for vulnerabilities.
Switching to a non-standard port like 2222 helps reduce unwanted traffic and brute-force attempts from automated bots. While not a substitute for real security, it's a simple and effective first layer of deterrence.

I chose 2222 specifically because:
- It’s easy to remember.
- It's high enough to avoid conflicts with standard service ports.
- It's commonly used in labs and non-production setups as an SSH alternative.

**Disable Password Authentication And Enforce Key-Based Login:**  
  ```diff
  - #PasswordAuthentication yes
  + PasswordAuthentication no
  ```

Disabling password authentication helps prevent brute-force attacks by bots attempting to guess user credentials.
Using SSH keys is far more secure, as the private key must match the public key on the server.
Setting `PasswordAuthentication no` ensures that only users with valid SSH keys can log in.

**Disable Root Login:**
  ```diff
  - #PermitRootLogin yes
  + PermitRootLogin no
  ```

Disabling root login blocks direct access to the server as the root user.
Allowing root access would give attackers superuser privileges if compromised, which is a serious security risk.
It also prevents accidental misconfigurations by users (including myself) when working in a lab environment with elevated privileges.

**Permit Empty Passwords**
  ```diff
  - #PermitEmptyPasswords no
  + PermitEmptyPasswords no
  ```

Setting this ensures that no user account with an empty password can log in over SSH.
Even if such an account is mistakenly created, this setting ensures it's not accessible remotely.

**Enable Banner**
  ```diff
  - #Banner none
  + Banner /etc/ssh/custom_ssh_banner.txt
  ```

Although it doesn't enhance security directly, enabling a login banner is good practice in professional settings.
A custom banner provides a warning or legal disclaimer before login, informing users (or intruders) that the system is monitored or restricted.

I created a file called `custom_ssh_banner.txt` containing a short compliance message similar to what you might see in enterprise environments.

![Custom Banner File](./screenshots/linux/06_ssh_custom_banner.png)


---

### 3. Update UFW Firewall Rules

After modifying the SSH configuration, I updated the firewall rules to reflect the new port:

```bash
sudo ufw allow 2222/tcp
sudo ufw delete allow 22/tcp
```

This allows incoming connections on port 2222 while removing access on port 22, which is commonly targeted in brute-force scans.
With the firewall adjusted, I restarted the SSH service to apply all changes:

```bash
sudo systemctl restart ssh
```

---

## ❗ Trouble Encountered: Connection Refused on Port 2222

After restarting the SSH service, I attempted to connect from my Ubuntu Desktop with another terminal window using:

```bash
ssh -p 2222 kailin@192.168.xxx.xxx
```

However, I encountered a critical issue: **"Connection refused."**

![Connection Refused](./screenshots/linux/07_ssh_connection_refused.png)

Even though the SSH configuration had been updated and I had restarted the SSH service with: `sudo systemctl restart ssh`, the server was still **not listening on port 2222**. 
To confirm this, I ran:

```bash
ss -tuln | grep 2222
```

![Port Not Used](./screenshots/linux/07_tuln_grep_2222.png)

The command returned no output, indicating that **port 2222 was not in use**, despite the changes made in the config file.

---

## 🛠️ Troubleshooting and Fix: SSH Server Not Listening on Port 2222

After modifying the `sshd_config` file and restarting the SSH service, the server was still not listening on port 2222. Below is the step-by-step breakdown of how I diagnosed and resolved the issue.

### 1. Check the SSH Configuration File for Port Declaration

I used `grep` to confirm whether the correct port was set in the SSH daemon configuration file.

```bash
grep -in "Port" /etc/ssh/sshd_config
```

This returned:
![Grep Port](./screenshots/linux/07_grep_port.png)

```
14:Port 2222
```

Which confirmed that port 2222 was properly defined and uncommented in the file.

---

### 2. Check UFW Firewall Status

Next, I checked the firwall status by using `sudo ufw status` to see if there are any issues.

![ufw status](./screenshots/linux/07_ssh_ufw_status.png)

Output showed that port 2222 was allowed and port 22 had been removed. This confirmed that UFW was not blocking my custom port.

---

### 3. Use `sshd -T` and `sshd -t` for Validation

To double-check what the SSH daemon thinks it's using, I ran:

```bash
sudo sshd -T | grep port
```

This should display:
```
port 2222
```

I also tested for syntax errors with:

```bash
sudo sshd -t
```

It returned no errors, confirming that the configuration syntax was valid.

![sshd T](./screenshots/linux/07_sshd_T.png)


---

### 4. Check File Permissions

I confirmed the permissions of `sshd_config`:

![sshd Permission](./screenshots/linux/07_sshd_config_permission.png)

The permissions were correct and not preventing the SSH service from reading the config.

---

### 5. Confirm Listening Ports

Once again, I checked whether the system was actually listening on port 2222:

![Port Not Used](./screenshots/linux/07_tuln_grep_2222.png)

Unfortunately, this returned nothing, confirming that SSH was **still not listening** on the custom port.

---

### 6. Inspect Journal Logs for SSH Restart

After doing some research to find the root cause of the problem, I decided to inspect the logs for any issues that might occur during SSH restart by running:

```bash
sudo systemctl restart ssh | journalctl -xe | grep sshd
```

#### What these commands do:
- `systemctl restart ssh`: restarts the SSH daemon
- `journalctl -xe`: shows the system journal logs in detail (with recent errors and warnings)
- `grep sshd`: filters output to only show entries related to the SSH daemon

![Listening Port 22](./screenshots/linux/07_listening_port_22.png)

The logs revealed that SSH was **still binding to port 22** — despite all configuration pointing to 2222.

---

### 7. Investigate systemd Override

At this point, I suspected the issue was with the way the SSH service was being launched. I checked the override configuration using:

```bash
systemctl cat ssh
```

![systemctl cat ssh](./screenshots/linux/07_cat_ssh.png)

The result showed that `ExecStart` was defined as `ExecStart=/usr/sbin/sshd -D $SSHD_OPTS`
This means the systemd was using the $SSHD_OPTS as the variable to provide additional configuration options, such as pointing to the correct SSH config file. However, $SSHD_OPTS was likely undefined or empty, meaning the SSH daemon started without explicitly using the updated configuration I had set in /etc/ssh/sshd_config.
As a result, the SSH service **defaulted to its built-in behavior**, which listens on port 22. This explained why my changes weren't taking effect, even though the config file itself was correctly configured.

---

### 8. Apply systemd Override Fix

I used the following command to open an editable override file for the SSH service:

```bash
sudo systemctl edit ssh
```

There, I updated the `ExecStart` line to explicitly specify the correct configuration file:

```bash
[Service]
ExecStart=
ExecStart=/usr/sbin/sshd -D -f /etc/ssh/sshd_config
```

![SSH Override](./screenshots/linux/07_ssh_Override.png)

The first `ExecStart=` line is intentionally left empty to clear any previous `ExecStart` directives. This prevents systemd from throwing a conflict error due to multiple `ExecStart` entries, which is only allowed for `Type=oneshot` services.
Then, instead of using `$SSHD_OPTS`, I directly specified the config file using `-f /etc/ssh/sshd_config` to ensure that the SSH daemon explicitly loads the correct configuration file.

After saving the override file, I reloaded systemd and restarted the SSH service:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart ssh
```

---

### 9. Final Confirmation

Once again, I verified that SSH was finally listening on port 2222:

```bash
sudo ss -tuln | grep 2222
```

![Issue Solved](./screenshots/linux/07_issue_solved.png)


✅ This time, the port was **successfully being used**. I opened a new terminal in my Ubuntu Desktop and was able to connect using the custom SSH port.

![Log In Success](./screenshots/linux/08_LoggedIn.png)

---

## ✅ Conclusion: What I Learned

Through this hardening process, I learned:

- How to properly modify SSH server configurations
- The importance of verifying systemd overrides when services ignore expected settings
- How to use tools like `ss`, `systemctl`, and `sshd -T` for low-level troubleshooting
- Why key-based authentication is essential for secure remote access
- How to update firewall rules to match SSH service changes

---

SSH is now running securely on port 2222 with password-based login disabled and key-based login enforced — exactly how a production-level SSH service should be configured.
