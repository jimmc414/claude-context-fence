---
description: "Users and permissions recipes reference (internal) - Receives prepared arguments from task-permissions with target and context included"
argument-hint: "[prepared argument from task-permissions router]"
context: fork
model: inherit
allowed-tools: Read
---

# Users and Permissions Tasks

**Tools:** chmod, chown, chgrp, chroot, id, whoami, groups, su, sudo, passwd, useradd, usermod, userdel, groupadd, groupmod, getfacl, setfacl, umask

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Make executable | `chmod +x script.sh` |
| Set permissions | `chmod 755 file` |
| Change owner | `chown user:group file` |
| Current user | `whoami` |
| User info | `id` |
| Add user | `sudo useradd -m username` |
| Set password | `sudo passwd username` |
| Add to group | `sudo usermod -aG groupname username` |

---

## File Permissions

```bash
chmod +x script.sh                    # Make executable
chmod 755 script.sh                   # rwxr-xr-x
chmod 644 file.txt                    # rw-r--r--
chmod 600 secrets.txt                 # rw-------
chmod -R 755 directory/               # Recursive
```

### Numeric Mode
```
4 = read (r)
2 = write (w)
1 = execute (x)
755 = rwxr-xr-x, 644 = rw-r--r--, 600 = rw-------
```

---

## Ownership

```bash
chown user file                       # Change owner
chown user:group file                 # Owner and group
chown -R user:group directory/        # Recursive
chgrp group file                      # Change group only
```

---

## User Management

```bash
sudo useradd -m username              # Create user
sudo passwd username                  # Set password
sudo usermod -aG groupname username   # Add to group
sudo userdel -r username              # Delete user
```

---

## Common Patterns

```bash
# Web server files
sudo chown -R www-data:www-data /var/www/
sudo chmod -R 755 /var/www/

# SSH keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```
