# SSH Password Authentication Guide

A step-by-step guide to disable private key-only SSH authentication and enable password-based login on a Linux server.

---

## Overview

This guide explains how to:
- Switch from private key (PEM) authentication to password-based SSH login
- Modify SSH daemon configuration to accept password authentication
- Test and verify password-based login works correctly

**Use case**: Temporary access, fallback authentication, or development environments where key management is not available.

---

## ⚠️ Security Warning

Password-based SSH authentication is less secure than key-based authentication. Use this only when:
- Key-based authentication is not available
- You use **strong, unique passwords** (12+ characters with mixed case, numbers, symbols)
- You restrict SSH access with firewalls or security groups
- You consider this a temporary solution for development/testing

**Best practice**: Keep key-based authentication as your primary method and use password login only for emergencies or temporary access.

---

## Prerequisites

- SSH access to a Linux server (Ubuntu, CentOS, Debian, etc.)
- `sudo` or root privileges
- A text editor (`vi`, `nano`, or `vim`)
- The ability to restart the SSH service

---

## Step 1: Current Key-Based SSH Login

Your current login method (using a private key):

```bash
ssh -i <your-key.pem> <USER_NAME>@<PUBLIC_IP>
```

**Parameters**:
- `<your-key.pem>` – Your private key file (e.g., `id_rsa`, `aws-key.pem`)
- `<USER_NAME>` – Linux username on the server (e.g., `ubuntu`, `ec2-user`, `root`)
- `<PUBLIC_IP>` – Public IP address or hostname of the server

**Example**:
```bash
ssh -i ~/aws-key.pem ubuntu@203.0.113.45
```

---

## Step 2: Set or Change the User Password

Log into the server using your private key, then set a password for the user:

```bash
sudo passwd <USER_NAME>
```

Replace `<USER_NAME>` with the actual username (e.g., `ubuntu`).

**Example**:
```bash
sudo passwd ubuntu
```

The system will prompt you to:
1. Enter a new password
2. Confirm the password

Use a strong password with:
- At least 12 characters
- Uppercase and lowercase letters
- Numbers and special characters
- No dictionary words

---

## Step 3: Edit SSH Server Configuration

Connect to the server and open the SSH daemon configuration file:

```bash
sudo vi /etc/ssh/sshd_config.d/00-default.conf
```

**Note**: On some systems, the file might be `/etc/ssh/sshd_config` instead. Use whichever exists on your system.

---

## Step 4: Enable Password Authentication

Find and modify these configuration directives:

```text
PasswordAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
```

### If using vi/vim editor:

1. Press `i` to enter insert mode
2. Navigate to the relevant lines (use arrow keys or `/` to search)
3. Change `PasswordAuthentication` from `no` to `yes` (if it exists)
4. Add the lines above if they don't exist
5. Press `Esc` to exit insert mode
6. Type `:wq` and press Enter to save and exit

### Full example configuration:

```text
# Allow password authentication
PasswordAuthentication yes

# Disable challenge-response authentication
ChallengeResponseAuthentication no

# Enable PAM (Pluggable Authentication Modules)
UsePAM yes
```

---

## Step 5: Restart SSH Service

Apply the changes by restarting the SSH daemon:

```bash
sudo systemctl restart sshd
```

**Alternative command** (on older systems):
```bash
sudo service ssh restart
```

---

## Step 6: Test Password-Based Login

In a **new terminal window** (keep your current session open as a fallback), test password login:

```bash
ssh <USER_NAME>@<PUBLIC_IP>
```

**Example**:
```bash
ssh ubuntu@203.0.113.45
```

The system will prompt for a password:
```
ubuntu@203.0.113.45's password:
```

Enter the password you set in **Step 2**.

### Success indicators:
- Prompt changes to show you're logged in
- No errors about authentication failures
- You can run commands on the remote server

---

## Verification

Once logged in via password, verify you're connected:

```bash
whoami                    # Shows current username
hostname                  # Shows server hostname
pwd                       # Shows current directory
```

---

## Reverting to Key-Only Authentication (Optional)

If you want to disable password authentication again and use only keys:

```bash
sudo vi /etc/ssh/sshd_config.d/00-default.conf
```

Change:
```text
PasswordAuthentication yes
```

Back to:
```text
PasswordAuthentication no
```

Save and restart SSH:
```bash
sudo systemctl restart sshd
```

---

## Troubleshooting

### "Permission denied (password)"
- Verify password is correct (check caps lock)
- Ensure you set the password with `sudo passwd <USER_NAME>`
- Password-based auth might not be enabled yet (re-check Step 4)

### SSH service won't restart
```bash
# Check for configuration errors
sudo sshd -t

# View systemd logs
sudo journalctl -u ssh -n 20
```

### Can't connect with password but key still works
- Password authentication might be in a different config file
- Check: `grep -r "PasswordAuthentication" /etc/ssh/`
- Restart SSH after making changes: `sudo systemctl restart sshd`

### Locked out completely
- Use a serial console, KVM, or cloud provider's web terminal
- Edit SSH config files directly as root
- Reset to allow key-based login

---

## SSH Configuration Directives Reference

| Directive | Value | Purpose |
|-----------|-------|---------|
| `PasswordAuthentication` | `yes` / `no` | Allow password-based login |
| `PubkeyAuthentication` | `yes` / `no` | Allow key-based login |
| `ChallengeResponseAuthentication` | `yes` / `no` | Allow interactive challenges |
| `UsePAM` | `yes` / `no` | Use Linux PAM for authentication |
| `PermitRootLogin` | `yes` / `no` / `prohibit-password` | Allow root login |
| `Port` | `22` | SSH listening port |
| `X11Forwarding` | `yes` / `no` | Allow X11 GUI forwarding |

---

## Best Practices

1. **Use strong passwords**: Min 12 characters, mixed case, numbers, symbols
2. **Limit SSH access**: Use firewall rules or security groups to restrict SSH connections
3. **Change default port**: Consider moving SSH to a non-standard port (e.g., 2222) to reduce bot attacks
4. **Keep key-based auth enabled**: As a backup method for recovery
5. **Monitor login attempts**: Check `/var/log/auth.log` regularly
6. **Disable password timeout**: On servers used for scripting, but always with firewall protection
7. **Use SSH keys for automation**: Scripts should always use keys, not passwords

---

## Additional Resources

- [OpenSSH Manual](https://man.openbsd.org/sshd_config) – Complete SSH configuration reference
- [Linux Security Best Practices](https://linux-audit.com/ssh-hardening-guide/) – SSH hardening guide
- [Password Security](https://www.ncsc.gov.uk/collection/mobile-device-guidance/using-built-in-platform-features/managing-built-in-platform-features-securely/password-authentication) – Creating secure passwords

---

## License

This guide is provided as-is for educational and practical purposes.
