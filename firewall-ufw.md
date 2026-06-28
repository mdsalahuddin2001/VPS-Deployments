# Firewall Setup (UFW)

A guide to configuring a basic firewall on a remote server with UFW (Uncomplicated Firewall). UFW is a friendly front end for `iptables` and ships with Ubuntu/Debian.

> **Before you enable the firewall:** allow your SSH port *first*. If you enable UFW with SSH blocked, you'll lock yourself out and need console access (via your VPS provider) to recover. See [SSH Hardening](./ssh-hardening.md).

## Install

UFW is usually pre-installed. If not:

```bash
sudo apt update
sudo apt install ufw -y
```

## Set Default Policies

Deny everything coming in, allow everything going out. This is the safe baseline — you then open only the ports you need.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

## Allow SSH (do this before enabling!)

If you're on the default port:

```bash
sudo ufw allow OpenSSH
```

If you changed the SSH port (e.g. to 2222):

```bash
sudo ufw allow 2222/tcp
```

## Allow Web Traffic (if running a web server)

```bash
sudo ufw allow 80/tcp     # HTTP
sudo ufw allow 443/tcp    # HTTPS
```

> Or use the bundled app profile if Nginx is installed:
>
> ```bash
> sudo ufw allow 'Nginx Full'
> ```

## Enable the Firewall

```bash
sudo ufw enable
```

Confirm SSH still works in a **second terminal** before closing your current session.

## Check Status

```bash
sudo ufw status verbose      # full view with default policies
sudo ufw status numbered     # numbered list, useful for deleting rules
```

## Remove a Rule

By name/spec:

```bash
sudo ufw delete allow 80/tcp
```

Or by number (from `ufw status numbered`):

```bash
sudo ufw delete 3
```

## Common Commands

```bash
sudo ufw disable             # turn the firewall off
sudo ufw reset               # wipe all rules and start over
sudo ufw allow from 1.2.3.4  # allow all traffic from a specific IP
sudo ufw allow from 1.2.3.4 to any port 2222   # allow one IP to one port
```

**Why use a firewall:** Default-deny on incoming traffic means only the services you explicitly open are reachable from the internet. Even if a service is accidentally left running on some port, the firewall blocks access to it — shrinking your attack surface to exactly what you intend to expose.
