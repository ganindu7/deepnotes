---
layout: default
title: Docker Tips
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

##  Docker 

Usually the first issue is running docker without admin rights 

Please refer to [this](https://github.com/pyenv/pyenv) 


If you want to attach your shell to a running container try [this](https://phase2.github.io/devtools/common-tasks/ssh-into-a-container/)

```bash
$ docker ps # use that to get the name of the running container (i.e. "cranky_dirac")
$ docker exec -it <container name> /bin/bash  # attach shell
```

