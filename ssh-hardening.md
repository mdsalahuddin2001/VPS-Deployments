# SSH Hardening

A guide to locking down SSH on a remote server: disabling root login, disabling password authentication, and changing the default SSH port.

> **Before you start:** Make sure you can already log in with an SSH key as a non-root sudo user. If you lock yourself out of these settings, you'll need console access (via your VPS provider) to recover. See [SSH Key Setup](./setup-ssh.md) and [Create / Delete Users](./create-delete-users.md).

All changes live in `/etc/ssh/sshd_config`. Edit it with:

```bash
sudo nano /etc/ssh/sshd_config
```

After editing, **test the config before restarting**:

```bash
sudo sshd -t
```

Then apply the changes:

```bash
sudo systemctl restart ssh
```

> Keep your current SSH session open while you test the new settings in a **second** terminal. If something is wrong, you still have a working session to fix it.

## 1. Disable Root Login

Find and set:

```
PermitRootLogin no
```

**Why:** `root` is the one account that exists on every Linux server, so it's the first target for brute-force attacks. Disabling root login means an attacker has to guess both a valid username *and* its credentials. Day-to-day work should go through a regular sudo user, which also gives you an audit trail of who did what.

## 2. Disable Password Authentication

Set:

```
PasswordAuthentication no
PubkeyAuthentication yes
```

> On some systems you should also set `ChallengeResponseAuthentication no` (or `KbdInteractiveAuthentication no` on newer OpenSSH) to fully turn off password prompts. Check for a `Include /etc/ssh/sshd_config.d/*.conf` line at the top of the file — a drop-in file there can override your changes.

**Why:** Passwords can be guessed, brute-forced, or reused across services. SSH keys are effectively impossible to brute-force. Once key-based login works, passwords only add risk — turning them off stops all password brute-force attempts dead, no matter how weak the password is.

## 3. Change the Default SSH Port

Set (pick any unused port, e.g. 2222):

```
Port 2222
```

> Pick a port in the **1024–49151** range (the registered-port range). Ports below 1024 are privileged/well-known and often reserved for other services, while the dynamic range **49152–65535** can be used by the OS for temporary outbound connections — avoiding both keeps the port stable and conflict-free.

If you use a firewall, open the new port **before** restarting SSH:

```bash
sudo ufw allow 2222/tcp
sudo ufw delete allow 22/tcp   # remove the old rule once the new port works
```

Connect using the new port:

```bash
ssh -p 2222 user@your_server_ip
```

> Add it to `~/.ssh/config` so you don't have to type `-p` every time:
>
> ```
> Host myserver
>     HostName your_server_ip
>     User youruser
>     Port 2222
> ```

**Why:** Automated bots constantly scan port 22. Moving SSH to a non-standard port won't stop a determined attacker, but it dramatically cuts the noise — most of those automated scans and log entries simply disappear. It's security through obscurity, so treat it as a complement to keys and disabled root login, not a replacement.

## 4. Install fail2ban

The config changes above stop weak credentials from working. fail2ban goes a step further: it watches the logs and **bans IPs** that keep trying, so attackers can't keep hammering the server at all.

Install it:

```bash
sudo apt update
sudo apt install fail2ban -y
```

Create a local override (never edit `jail.conf` directly — package updates overwrite it):

```bash
sudo nano /etc/fail2ban/jail.local
```

Add an SSH jail. Set `port` to whatever you chose in step 3:

```
[sshd]
enabled = true
port = 2222
maxretry = 5
bantime = 1h
findtime = 10m
```

> - `maxretry` — failed attempts before a ban.
> - `findtime` — the window those attempts are counted in. Use a unit suffix: `s` seconds, `m` minutes, `h` hours, `d` days (e.g. `600` or `10m` = 10 minutes, `1h` = 1 hour).
> - `bantime` — how long the IP stays banned (use `-1` for permanent). Same unit suffixes apply: `600`/`10m` = 10 minutes, `1h` = 1 hour, `1d` = 1 day.

Enable and start it:

```bash
sudo systemctl enable --now fail2ban
```

Check what it's doing:

```bash
sudo fail2ban-client status sshd
```

> **Don't lock yourself out:** whitelist your own IP so a few fat-fingered logins don't ban you. Add under a `[DEFAULT]` section in `jail.local`:
>
> ```
> [DEFAULT]
> ignoreip = 127.0.0.1/8 your.home.ip.address
> ```

**Why:** Even with keys-only login, bots will keep trying — filling your logs and wasting resources. fail2ban turns repeated failures into an automatic firewall ban, so a persistent attacker gets cut off instead of retrying indefinitely. It's the active defense that complements the passive hardening above.

## Verify

From a new terminal, confirm you can still log in with all settings applied:

```bash
ssh -p 2222 user@your_server_ip
```

If that works, you're done. Close the backup session.
