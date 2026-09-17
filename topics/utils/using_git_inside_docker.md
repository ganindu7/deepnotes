---
layout: default
title: Using Git inside Docker
parent: Utilities
permalink: /topics/utils/using_git_inside_docker
nav_order: 2
---

## Using Git inside Docker

Status: Draft | Updated: {{ page.path | file_date | date_to_string }}

The goal: let a user (here `will`) clone, pull and push GitHub repositories from inside a container on a shared machine (the `dgx`) without leaving any credential on that machine, and only for as long as his SSH session is alive.

### The idea

The private key never leaves Will's laptop. Will connects to the dgx with SSH agent forwarding, which gives his shell on the dgx a socket (`$SSH_AUTH_SOCK`) that can *sign* with his key but cannot read it. The container gets that one socket bind-mounted and nothing else. When Will logs out the socket disappears and the container loses GitHub access.

What not to do: generate a key on the dgx, or copy a key (or `~/.ssh`) into a container or an image. On a shared host everyone in the `docker` group is effectively root and can read any container's filesystem, and a key stored there outlives the session and has to be revoked by hand.

### 1. On the laptop: create a key and add it to GitHub

```shell
ssh-keygen -t ed25519 -C "will@example.com"
```

If you have a FIDO2 hardware key, `ssh-keygen -t ed25519-sk` makes a key that also needs a touch for every use, which is the strongest option here.

Then copy the *public* half (`cat ~/.ssh/id_ed25519.pub`) into the GitHub UI under Settings, SSH and GPG keys:

![GitHub "Add new SSH Key" form](using_git_inside_docker/github-add-ssh-key.png)

### 2. On the laptop: forward the agent, but only to the dgx

```
# ~/.ssh/config
Host dgx
    HostName <dgx address>
    User will
    ForwardAgent yes
```

```shell
ssh-add ~/.ssh/id_ed25519   # macOS: ssh-add --apple-use-keychain ~/.ssh/id_ed25519
ssh dgx
```

Only enable `ForwardAgent` for hosts you trust. While you are connected, root on that host can *use* the forwarded agent (it cannot extract the key). `ssh-add -c` makes the agent ask for confirmation on every use if you want that extra check.

### 3. On the dgx: check the forwarded agent

```shell
echo $SSH_AUTH_SOCK     # a socket under /tmp/ssh-XXXX/
ssh-add -l              # lists the laptop key
ssh -T git@github.com   # "Hi will! You've successfully authenticated..."
```

No key files were created on the dgx. If `ssh-add -l` reports no agent, forwarding is off: check `ForwardAgent` on the laptop and `AllowAgentForwarding` in `/etc/ssh/sshd_config` on the dgx.

Set the identity git should use for commits (this is not a secret):

```shell
git config --global user.name "Will Example"
git config --global user.email "will@example.com"
```

### 4. The compose override: hand the socket to the container

Next to the project's compose file add `compose.override.yaml` (or `docker-compose.override.yml`, matching the base file's naming). Compose merges it automatically, so the shared compose file stays untouched and everyone can keep their own override.

```yaml
services:
  dev:   # the service name from the base compose file
    volumes:
      - ${SSH_AUTH_SOCK:?SSH agent forwarding is not active}:/run/ssh-agent.sock
      - ${HOME}/.gitconfig:/etc/gitconfig:ro
    environment:
      SSH_AUTH_SOCK: /run/ssh-agent.sock
```

The `:?` makes compose refuse to start with a clear message if you logged in without forwarding, instead of mounting an empty path.

Start a container for the session:

```shell
docker compose run --rm dev bash
```

Inside the container:

```shell
ssh-add -l   # same key as on the dgx, still no key file anywhere
mkdir -p ~/.ssh && ssh-keyscan -t ed25519 github.com >> ~/.ssh/known_hosts
ssh -T git@github.com
git clone git@github.com:org/private-repo.git
```

Check the fingerprint against [GitHub's published SSH key fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints) the first time. The `known_hosts` line is public information and can be baked into the image.

### Notes

* The socket path changes on every login. A container started from an earlier session loses GitHub access; start a fresh one (`docker compose run --rm ...`, or `docker compose up -d --force-recreate`). That is the "scoped to the session" behaviour we want.
* Run the container as yourself so files you create in mounted volumes are owned by you: `export LOCAL_UID=$(id -u) LOCAL_GID=$(id -g)` and add `user: "${LOCAL_UID}:${LOCAL_GID}"` to the override. The socket is owned by you, so it still works. Some images assume root, so check that the tools you need still run.
* Unattended jobs (no SSH session) cannot use agent forwarding. For those use a [fine-grained personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token) limited to the repositories needed, with an expiry, passed in as an environment variable, or a per-repository [deploy key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/managing-deploy-keys#deploy-keys). Never bake either into an image.
* To revoke access, delete the key in the GitHub UI. Nothing needs cleaning up on the dgx or in any container.
