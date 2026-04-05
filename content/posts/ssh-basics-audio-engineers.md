---
date: 2026-04-05T00:00:00Z
draft: false
categories:
  - Broadcast Technology
  - Security
tags:
  - SSH
  - Linux
  - Networking
  - Audio Engineering
  - IP
title: "SSH Basics for Audio Engineers"
description: "If you work with Linux-based audio systems, remote servers, or broadcast infrastructure, SSH is an essential tool. Here is a practical guide to generating, copying, and managing SSH keys."
cover:
  image: ssh.png
  alt: "SSH"
---

If you spend any time working with Linux-based audio systems - streaming servers, audio consoles with embedded Linux systems, remote monitoring tools, or Raspberry Pi-based devices - you will eventually need to connect to them over SSH. Getting comfortable with key-based authentication makes that process faster, more secure, and less friction than typing a password every time.

This is not a deep dive into cryptography. It is a practical reference for the commands you will actually use.

## What SSH Is

SSH (Secure Shell) is a protocol for connecting to remote machines over a network. It gives you a terminal session on the remote machine as if you were sitting in front of it. Everything sent over the connection is encrypted.

The default authentication method is a password. The better method is a key pair - a private key that stays on your machine and a public key that you place on the remote server. When you connect, the server checks that your private key matches the public key it has on file. No password is ever sent over the network.

## Generating a Key Pair

```bash
ssh-keygen -t ed25519 -C "your@email.com"
```

This generates a key pair using the Ed25519 algorithm, which is modern, fast, and produces short keys. The `-C` flag adds a comment to the public key - useful for identifying which key belongs to whom when you have multiple keys on a server.

You will be prompted to choose a location (the default `~/.ssh/id_ed25519` is fine) and an optional passphrase. A passphrase encrypts the private key on disk, so even if someone gets hold of the file they cannot use it without the passphrase. For anything other than a throwaway dev machine, use one.

This produces two files:

- `~/.ssh/id_ed25519` - your private key. Never share this.
- `~/.ssh/id_ed25519.pub` - your public key. This is what you put on remote servers.

If you are working with older systems that do not support Ed25519, RSA is the fallback:

```bash
ssh-keygen -t rsa -b 4096 -C "your@email.com"
```

## Copying Your Public Key to a Remote Server

```bash
ssh-copy-id user@hostname
```

This copies your public key to the remote server and appends it to `~/.ssh/authorized_keys` on that machine. After this, you can connect without a password.

If the remote server uses a non-standard port:

```bash
ssh-copy-id -p 2222 user@hostname
```

If `ssh-copy-id` is not available (it is absent on macOS by default unless installed via Homebrew), you can do it manually:

```bash
cat ~/.ssh/id_ed25519.pub | ssh user@hostname "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

## Connecting

```bash
ssh user@hostname
```

If your key is in the default location, SSH picks it up automatically. If you have multiple keys and need to specify one:

```bash
ssh -i ~/.ssh/id_ed25519 user@hostname
```

## Managing Keys with ssh-agent

If you set a passphrase on your key, you will be prompted for it each time you connect. The SSH agent keeps your decrypted key in memory for the duration of your session, so you only enter the passphrase once.

Start the agent and add your key:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

On macOS, Keychain integration means the agent persists across reboots. On Linux, the agent typically lives for the duration of your login session.

To see which keys are currently loaded:

```bash
ssh-add -l
```

To remove all keys from the agent:

```bash
ssh-add -D
```

## The SSH Config File

If you regularly connect to several servers, typing out full hostnames and usernames gets old quickly. The SSH config file at `~/.ssh/config` lets you define shortcuts:

```
Host streamserver
    HostName 192.168.1.50
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519

Host rpi-audio
    HostName rpi-audio.local
    User pi
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

With this in place, `ssh streamserver` and `ssh rpi-audio` work without any additional flags.

## Useful Day-to-Day Commands

Check what keys are authorised on a remote server:

```bash
cat ~/.ssh/authorized_keys
```

Remove a specific host from your known hosts file (useful after reimaging a device):

```bash
ssh-keygen -R hostname
```

Copy a file to a remote server:

```bash
scp localfile.wav user@hostname:/path/to/destination/
```

Copy a file from a remote server:

```bash
scp user@hostname:/path/to/file.wav ./
```

Run a single command on a remote server without starting an interactive session:

```bash
ssh user@hostname "systemctl status liquidsoap"
```

## In an Audio Engineering Context

A few places where this comes up regularly:

**Cloud infrastructure** - Linux VMs on AWS, GCP, Azure, or DigitalOcean are almost always accessed exclusively via SSH. Most cloud providers inject your public key at provisioning time, so password auth is often disabled by default. Managing keys properly matters more here because these machines are directly internet-facing.

**Broadcast codecs and embedded Linux devices** - Some codec and processing units run Linux under the hood and expose SSH access. Check the documentation - if it is there, key auth is worth setting up.

The main thing is to get out of the habit of relying on password authentication for anything you access regularly. Key-based SSH is faster, harder to brute-force, and removes the risk of reusing passwords across systems.
