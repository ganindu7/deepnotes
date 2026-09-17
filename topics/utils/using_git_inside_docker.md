---
layout: default
title: Using Git inside Docker
parent: Utilities
permalink: /topics/utils/using_git_inside_docker
nav_order: 2
---

## Using Git inside Docker

Status: Draft | Updated: {{ page.path | file_date | date_to_string }}

1. inside the `dgx` create a key-pair compatible with [github](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux#generating-a-new-ssh-key)

```shell
ssh-keygen -t ed25519 -f ~/.ssh/will-github-key -C "your_email@example.com"
```

After that add the newly minted ssh key to the agent.

```shell
eval "$(ssh-agent -s)" # make sure ssh is running
ssh-add ~/.ssh/will-github-key
```

Afterwards get the key (so you can copy the public portion)

```shell
cat ~/.ssh/will-github-key.pub
```

then add the key using the github UI

![GitHub "Add new SSH Key" form](using_git_inside_docker/github-add-ssh-key.png)

then you can go ahead with adding a docker compose override file and populating that
