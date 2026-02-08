---
description: "Users and permissions - can't access file, operation not permitted, locked out of directory, directory not writable, permission denied errors, file permissions, ownership, and access control (chmod, chown, setfacl, sudo)"
when_to_use: "Use when: can't access a file or directory, getting operation not permitted error, locked out of a directory or system, directory is not writable, permission denied errors, need to change file permissions, change ownership, create users, manage groups, configure sudo access, set up ACLs, debug access denied errors. Triggers: can't access file, operation not permitted, locked out, directory not writable, permission denied, access denied, can't write, can't execute, chmod, chown, permissions, user, group, sudo, acl, umask, setuid, sticky bit, capabilities, sudoers, passwd, adduser, rwx, 755, ownership, getfacl, setfacl, file mode."
when-to-use: "Use when: can't access a file or directory, getting operation not permitted error, locked out of a directory or system, directory is not writable, permission denied errors, need to change file permissions, change ownership, create users, manage groups, configure sudo access, set up ACLs, debug access denied errors. Triggers: can't access file, operation not permitted, locked out, directory not writable, permission denied, access denied, can't write, can't execute, chmod, chown, permissions, user, group, sudo, acl, umask, setuid, sticky bit, capabilities, sudoers, passwd, adduser, rwx, 755, ownership, getfacl, setfacl, file mode."
argument-hint: "[describe the permission or user task]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Permissions Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the permissions recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Analyze the conversation to identify:

### 1. Operation
What permission operation is needed?
- Set file/directory permissions, change ownership
- Create/modify users or groups
- Configure ACLs, check current permissions
- Debug "access denied" errors

### 2. Target
What are we modifying?
- File or directory path
- User account
- Group membership

### 3. Permission Model
What level of access control?
- **Basic**: chmod numeric (755) or symbolic (u+x)
- **ACL**: setfacl/getfacl for fine-grained control
- **Capabilities**: Linux capabilities for specific privileges

### 4. Current State
What's the starting point?
- Error messages ("permission denied" from earlier in conversation)
- Current permissions unknown (need to check first)

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| Set file permissions | chmod | Numeric (755) or symbolic (u+x) |
| Change ownership | chown | User and/or group |
| Change group only | chgrp | Group ownership only |
| Fine-grained ACLs | setfacl / getfacl | Per-user/group access |
| Check current perms | stat / ls -la | File metadata |
| Default permissions | umask | Set creation defaults |
| Add user | useradd -m | Create with home directory |
| Add to group | usermod -aG | Append to supplementary group |
| Set password | passwd | Interactive password change |
| Check identity | id / whoami | Current user info |

---

## Argument Format

Construct a structured argument string:

```
<operation> on <target> - permission: <value>, options: <settings>
```

**Include only relevant components.**

### Examples

**Set file permissions:**
```
set permissions 755 on /home/user/scripts/*.sh - recursive: no
```

**Change ownership:**
```
change ownership to www-data:www-data on /var/www/html - recursive: yes
```

**Debug access denied:**
```
diagnose permission denied on /var/log/app.log - current user: appuser, error from earlier in conversation
```

**ACL setup:**
```
grant read access to user deploy on /opt/app/ - using ACL, recursive
```

**User management:**
```
create user webadmin - add to groups: sudo,www-data, create home directory
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-permissions-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Target path is unclear
- Permission value is ambiguous
- Operation could be destructive (recursive chmod)

**Use Read tool first if:**
- Need to check current permissions
- Need to verify user/group exists
