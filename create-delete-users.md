# Ubuntu User Management — Why & How

---

## Why Create Users Instead of Using Root?

The **root user** is the superuser in Ubuntu with unrestricted access to every file, command, and system setting. While it may seem convenient, operating as root daily is considered a bad practice. Here's why:

---

### 1. Security

The root user has **zero restrictions** — it can read, modify, or delete any file on the system. If you are logged in as root and:

- Your session is **hijacked** by an attacker
- You accidentally run a **malicious script**
- A **software vulnerability** is exploited

...the attacker or script gets **full control** of your entire server. There is no safety boundary.

A regular user is sandboxed — even if compromised, the damage is limited to that user's own files and permissions.

> **Best Practice:** Always operate day-to-day as a regular user. Elevate with `sudo` only when absolutely necessary.

---

### 2. Prevent Accidental Damage

As root, there is **no safety net**. A single typo can be catastrophic.

**Example — a dangerous typo as root:**

```bash
# Intended: remove a temp folder
rm -rf /tmp/myapp

# Typo (space after /) — destroys the entire OS!
rm -rf / tmp/myapp
```

As a regular user, the same command would be stopped with:

```
Permission denied
```

Root executes commands **instantly and silently** — no confirmation, no undo. Regular users are naturally protected from wiping critical system files.

---

### 3. Allow Multiple Users to Manage the VPS Efficiently

When a team manages a server, using a **single root account** creates serious problems:

| Problem            | Impact                                                                 |
| ------------------ | ---------------------------------------------------------------------- |
| No accountability  | You can't tell _who_ made a change                                     |
| Shared credentials | Revoking one person's access means changing root password for everyone |
| No audit trail     | Logs show `root` for every action — useless for debugging              |

With **individual user accounts**:

- Each admin has their own login
- You can grant or revoke access per person independently
- Every `sudo` command is logged with the username in `/var/log/auth.log`
- You can track exactly _who did what and when_

---

## Normal User vs Sudo User

| Feature             | Normal User | Sudo User            |
| ------------------- | ----------- | -------------------- |
| Can log in          | ✅ Yes      | ✅ Yes               |
| Access own files    | ✅ Yes      | ✅ Yes               |
| Install software    | ❌ No       | ✅ Yes (with `sudo`) |
| Modify system files | ❌ No       | ✅ Yes (with `sudo`) |
| Manage other users  | ❌ No       | ✅ Yes (with `sudo`) |

### Normal User

A regular user can only read/write their own files under `/home/username`. They cannot install packages, edit system config files, or affect other users.

### Sudo User

A sudo user is still a regular user but is added to the **sudo group**, allowing them to temporarily run commands with root privileges by prefixing `sudo`.

```bash
# Regular user — denied
apt update
# E: Could not open lock file ... Permission denied

# Sudo user — works
sudo apt update
# [sudo] password for john:
```

Every `sudo` command requires the user's **own password** and is **logged** — giving you security + accountability.

---

## How to Create a User

### Step 1 — Create the user

```bash
sudo adduser john
```

This will prompt you to set a password and fill in optional details (Full Name, Phone, etc.). Press `Enter` to skip optional fields.

```
Adding user 'john' ...
Adding new group 'john' (1001) ...
Adding new user 'john' (1001) with group 'john' ...
Creating home directory '/home/john' ...
New password:
Retype new password:
passwd: password updated successfully
```

### Step 2 — Verify the user was created

```bash
id john
```

Output:

```
uid=1001(john) gid=1001(john) groups=1001(john)
```

### Step 3 — Grant sudo privileges (optional)

To make the user a **sudo user**, add them to the `sudo` group:

```bash
sudo usermod -aG sudo john
```

Verify:

```bash
id john
# uid=1001(john) gid=1001(john) groups=1001(john),27(sudo)
```

Now `john` can run commands with `sudo`.

---

## How to Switch to a User

```bash
# Switch to another user
su - john

# Switch back to your original user
exit
```

---

## How to Remove a User

### Remove user only (keep home directory)

```bash
sudo deluser john
```

### Remove user AND their home directory

```bash
sudo deluser --remove-home john
```

> ⚠️ **Warning:** `--remove-home` permanently deletes all files in `/home/john`. Make sure to back up any important data first.

### Verify the user is removed

```bash
id john
# id: 'john': no such user
```

---

## Quick Reference

```bash
# Create a user
sudo adduser username

# Grant sudo access
sudo usermod -aG sudo username

# Switch to a user
su - username

# Remove user (keep files)
sudo deluser username

# Remove user + home directory
sudo deluser --remove-home username

# List all users
cat /etc/passwd | grep /home
```

---

## Summary

| Action              | Command                               |
| ------------------- | ------------------------------------- |
| Create user         | `sudo adduser username`               |
| Add to sudo group   | `sudo usermod -aG sudo username`      |
| Switch user         | `su - username`                       |
| Remove user         | `sudo deluser username`               |
| Remove user + files | `sudo deluser --remove-home username` |

> Always use individual user accounts on your VPS. Reserve root access for emergencies only, and use `sudo` for day-to-day administrative tasks.
