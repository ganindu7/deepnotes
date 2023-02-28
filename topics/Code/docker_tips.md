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


## Commit changes to an image 

When we modify an image (i.e install dependencies) and want the image to retain it's modified state.

* Identify image: `$ docker ps` (hint: zoom the image below :p)
![docker_ps_output](docker_tips/docker-ps.png)

* Commit Changes `docker commit -m "your-commit-message" eager_hodgkin nvcr.io/nvidia/tensorflow:21.10-tf1-py3-ganindu_matplotlib` 


## Docker on Nvidia Jetson 

### Setting up the runtime 

Important prerequisite: 
sudo vim /etc/docker/daemon.json 
```
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```
1.  Connect all devices and cables. 
2.  Connect power to the Device 
3.  Connect power to additional equipment that needs separate power (if applicable).
4.  set the "DISPLAY" env variable if you  are using a remote shell (e.g. export DISPLAY=:1)
     if this does not work run `xhost +` in the jetson to allow remote displays. (`xhost +` can only run directly via a keyboard attached to jetson for security reasons)

5.  test display functionality with `xeyes`: (There should be an output on the screen attached to the jetson
6.  in a shell in jetson `cd /Workspace/your/docker/folder/with/the/compose/file`
7.  `docker-compose up`
8.  `sudo xhost +si:localuser:root` # if you use this you'll have to export the display variable e.g. `export DISPLAY=:1`
9.  `docker exec -it <the-container-name> /bin/bash --login`
10. try if `xeyes` work (optional)
11. do your stuff
12. After running do `xhost -` to shut down access to xhost by anyone 


TIPS FOR ACESSING DOCKER CONTAINER FROM A REMOTE:

Remember to do [docker-post installation steps](https://docs.docker.com/engine/install/linux-postinstall/)

Create the docker group (If already not done).
* run `sudo groupadd docker`

Add your user to the docker group.

* run `sudo usermod -aG docker $USER`


## Docker and Jupyter notebooks

if you haven't installed already use 

`python3 -m pip install ipykernel` to install the ipykernel 
`python3 -m pip install jupyter` to install Jupyter 

because you used python install the PATH variable may not be updated so try using 
`python3 -m jupyter notebook .`


Nvidia Userguide 
https://docs.nvidia.com/deeplearning/frameworks/user-guide/index.html


## Migrating docker storage 

Sometimes to use the available disk space optimally we need to move things around and docker storage directory is no exception, as of writing docker by default uses  the `/var/lib/docker` directory. let's say we mounted our additional storage onto `/mnt/my/new/storage` and we want to place the docker storage in the `/docker/storage` directory relative to that. 

[orignal_source]()

#### STEP 1 

stop docker (you can pick when perform this action)
 ``` sudo service docker stop ```


#### STEP 2 

 create the 'daemon.json' on `/etc/docker` (verify with docker docks that this is the correct location)

 ```
 sudo vim /etc/docker/daemon.json
 ```

 and then edit according to your use case.

 ```
{
    "data-root": "/mnt/my/new/storage/docker/storage"
}

 ```

#### STEP 3 

Copy the files to the destination 

```
sudo rsync -vaP /var/lib/docker /mnt/my/new/storage/docker/storage

```

#### STEP 4
we can rename the old directory to a make sure the changes have taken effect.  

```
sudo mv  /var/lib/docker /var/lib/docker.old
```

#### STEP 5. 
Start the docker service again to see if everything works 

```
sudo service docker start
```

### IMPORTANT 

* Doing this with debilitating hardware constraints can lead to run time difficulties in execution. 
* Be aware the that an active docker container uses more space in `/var/lib/docker` (you can confirm this with a command such as `du -h d 1` before and after activating a container.). 


