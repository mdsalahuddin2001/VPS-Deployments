# Firewall Setup (UFW)

### What is Firewall ?

A firewall is a security device or software that monitors and controls incoming and outgoing network traffic based on set rules.

### UFW

UFW stands for Uncomplicated Firewall, a user-friendly command-line tool used to manage firewall rules on Linux systems.

> **Before you enable the firewall:** allow your SSH port _first_. If you enable UFW with SSH blocked, you'll lock yourself out and need console access (via your VPS provider) to recover. See [SSH Hardening](./ssh-hardening.md).

### Install

UFW is usually pre-installed. If not:

```bash
sudo apt update
sudo apt install ufw -y
```

### Check the current ufw status

Before making any changes, check whether ufw is already enabled.

```bash
sudo ufw status
```

For more detailed information:

```bash
sudo ufw status verbose
```

### Allow SSH port before enabling ufw

If you are connected to your vps through SSH, never enable UFW before allowing SSH access

If your SSH port is 22 (default) , first run:

```bash
sudo ufw allow 22/tcp
```

SSH use TCP protocol, and by default it runs on port 22. If you have changed the SSH port, use that port number instead of 22.
If we omit /tcp and run

```bash
sudo ufw allow 22
```

It will add the rule for both TCP and UDP.But since ssh only use TCP protocol, it is better to specify /tcp.This way we can deny UDP or other traffic to port 22.

## Set Default Policies

Deny everything coming in, allow everything going out. This is the safe baseline — you then open only the ports you need.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

It means deny all incoming traffic and allow all outgoing traffic. This is the safe baseline — you then open only the ports you need to allow.

### Allow HTTP and HTTPS

If the vps hosts a website, you normally need:

- HTTP → port 80
- HTTPS → port 443

Allow them:

```bash
sudo ufw allow 80/tcp
```

```bash
sudo ufw allow 443/tcp
```

Or, if the corresponding UFW application profile exists:

```bash
sudo ufw allow 'Nginx Full'
```

This allows both HTTP and HTTPS traffic.

### Enable UFW

once the required ports have been allowed, run:

```bash
sudo ufw enable
```

to enable the firewall. Type 'y' and press Enter to proceed.
Then verify:

```bash
sudo ufw status verbose
```

**Default:** deny (incoming), allow (outgoing), disabled (routed)

| To      | Action | From     |
| ------- | ------ | -------- |
| 22/tcp  | ALLOW  | Anywhere |
| 80/tcp  | ALLOW  | Anywhere |
| 443/tcp | ALLOW  | Anywhere |

### Allow a specific port

Suppose your application needs port `3000`.

You can allow it with:

```bash
sudo ufw allow 3000/tcp
```

However, **don't automatically expose every application port to the public internet**.

For example, if Node.js runs on port `3000` and Nginx is acting as a reverse proxy, you usually don't need:

```bash
sudo ufw allow 3000/tcp
```

Instead, the architecture can look like this:

```text
Internet
   ↓
443 HTTPS
   ↓
Nginx
   ↓
localhost:3000
   ↓
Node.js
```

In this architecture, port `3000` can remain inaccessible from the public internet.

### Allow a port only from a specific ip

Somethime you wanta service to be accessible only from your Own/Specific IP.

For example: You have a mongodb database hosted in a dedicated vps, and nodejs server in another vps. You want to connect to mongodb from nodejs server. You should allow the mongodb port only from the nodejs server's IP.

To do this, you can use the following command:

```bash
sudo ufw allow from 203.0.113.10 to any port 27017 proto tcp
```

This allows MOngoDB port 27017 only from 203.0.113.10 ip address. No one else can access the mongodb port.

This is much safer than:

```bash
sudo ufw allow 27017/tcp
```

which exposes MongoDB to the internet.

### Restrict SSH to a specific IP

If your server should only be accessible from your own/specific IP. You can use the following command:

```bash
sudo ufw allow from 203.0.113.10 to any port 22 proto tcp
```

This means:

203.0.113.10 → SSH → ALLOWED

Everyone else → SSH → BLOCKED

Only do his if you source IP is stable. If your ISP changes your public IP, you could lock yourself out.

### Allow UDP Ports

UFW can manage both TCP and UDP traffic. If you need to allow a UDP port, you can do so with the following command:

```bash
sudo ufw allow 1194/udp
```

### Allow a Range of Ports

You can allow a range:

```bash
sudo ufw allow 6000:6100/tcp
```

This allows:

6000
6001
6002
...
6100

Be careful with large port ranges because they increase the server's exposed attack surface.

### Deny a Port

You can explicitly deny a port:

```bash
sudo ufw deny 8080/tcp
```

However, remember that if you already set default policy running:

```bash
sudo ufw default deny incoming
```

then an unallowed port is already blocked. Therefore, explicit deny rules are often unnecessary.

### Delete a rule

First list the rules with numbers:

```bash
sudo ufw status numbered
```

You will see something like this:

```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW       Anywhere
[ 2] 80/tcp                     ALLOW       Anywhere
[ 3] 443/tcp                    ALLOW       Anywhere
[ 4] 3000/tcp                   ALLOW       Anywhere
```

Then delete the rule by number:

```bash
sudo ufw delete 4
```

you can also delete a rule using the original rule:

```bash
sudo ufw delete allow 3000/tcp
```

### Reset UFW

If you want to remove UFW's rules and disable it (exactly like before we have added any rule and enable it):

```bash
sudo ufw reset
```

ufw will ask for confirmation.

After resetting, ufw becomes inactive and its rules are removed.

Then you can configure it again

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

### Disable UFW

To temporarily disable UFW:

```bash
sudo ufw disable
```

Check:

```bash
sudo ufw status
```

You should see:

Status: inactive

You can re enable it with:

```bash
sudo ufw enable
```
