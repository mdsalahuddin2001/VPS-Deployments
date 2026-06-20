# SSH Key Setup

A guide to generating an SSH key pair and registering it with GitHub/GitLab or a remote server.

## Mac / Linux

Check for existing SSH keys:

```bash
cd ~/.ssh
ls
```

Generate an SSH key pair:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

> **Legacy systems** that don't support Ed25519 should use RSA instead:
>
> ```bash
> ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
> ```

Start the SSH authentication agent:

```bash
eval "$(ssh-agent -s)"
```

Create the SSH config file if it doesn't already exist:

```bash
touch ~/.ssh/config
```

Add the following to `~/.ssh/config`. Adjust the `IdentityFile` path if your key has a different name.

```
Host *

AddKeysToAgent yes

UseKeychain yes

IdentityFile ~/.ssh/id_ed25519
```

> **`UseKeychain yes` is Mac only** — remove this line on Linux. It only applies if your key has a passphrase.

Add your private key to the ssh-agent:

```bash
# Mac (stores passphrase in keychain)
ssh-add --apple-use-keychain ~/.ssh/id_ed25519

# Linux
ssh-add ~/.ssh/id_ed25519
```

## Windows

Generate an SSH key pair:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

> **Legacy systems** that don't support Ed25519 should use RSA instead:
>
> ```bash
> ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
> ```

When prompted, press **Enter** to accept the default file location. If you already have keys, give the new one a custom name to avoid overwriting:

Enter file in which to save the key (/c/Users/YOU/.ssh/id_ed25519): [Press Enter]

Enter passphrase (empty for no passphrase): [Type a passphrase]

Enter same passphrase again: [Type passphrase again]

Ensure the ssh-agent is running (PowerShell **as Administrator**):

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

Add your private key to the ssh-agent (Git Bash or PowerShell):

```bash
ssh-add ~/.ssh/id_ed25519
```

## Add the Key to GitHub / GitLab / Server

Copy your **public** key:

```bash
pbcopy < ~/.ssh/id_ed25519.pub    # Mac
clip < ~/.ssh/id_ed25519.pub      # Windows (Git Bash)
cat ~/.ssh/id_ed25519.pub         # Linux (copy manually)
```

Paste it into your provider's SSH key settings, then test the connection:

```bash
ssh -T git@github.com
```
