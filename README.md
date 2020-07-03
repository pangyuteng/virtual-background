

# summary

tl;dr. Enable virtual background in Linux (tested with Ubuntu 20.04, Nvidia GeForce 940MX, and a crappy web cam).

Check out @BenTheElder's post for a !tl;dr.
https://elder.dev/posts/open-source-virtual-background/

## *** Credit goes to Benjamin Elder @BenTheElder, the original author! ***

# installation

+ install docker (https://docs.docker.com/engine/install/ubuntu)

+ install nvidia-docker (https://github.com/NVIDIA/nvidia-docker)

+ test nvidia-docker is install properly
```
docker run --gpus all nvidia/cuda:10.0-base nvidia-smi
```

+ install and setup virtual video device as "/dev/video20", and assuming the actual video device is "/dev/video0"
``` 
sudo apt-get upgrade -y
sudo apt-get install -y v4l2loopback-dkms v4l2loopback-utils

sudo modprobe -r v4l2loopback
sudo modprobe v4l2loopback devices=1 video_nr=20 card_label="v4l2loopback" exclusive_caps=1
```

+ add root to group video (likely unecessary...)
```
sudo usermod -aG video root
cat /etc/group | grep video
```

+ clone repo
```
git clone git@github.com:pangyuteng/virtual-background.git vbkgd
cd vbkgd
```

+ build via docker-compose
```
docker-compose build
```

+ start the virtual camera via docker-compose (assuming gpu is present at `/dev/nvidia0`, physical video device at `/dev/video0` and virtual video device at `/dev/video20`)
```
docker-compose up
```

+ launch zoom/teams/slack..., select `v4l2loopback` as webcam

+ live swap background by replacing file `data/background.jpg` - refresh rate hard coded at 3 seconds.


# misc/development

+ build docker
```
docker build -t bodypix ./bodypix
docker build -t fakecam ./fakecam
```

+ dev mode, remove entrypoint from Dockerfile, rebuild and run below to edit code
```
# in terminal 
docker run   --name=bodypix   --network=fakecam   -p 9000:9000   --gpus=all --shm-size=1g --ulimit memlock=-1 --ulimit stack=67108864 -v ${PWD}/bodypix:/src -it bodypix /bin/bash

# in another terminal
docker run  --name=fakecam   --network=fakecam  $(find /dev -name 'video*' -printf "--device %p ") -v ${PWD}/fakecam:/src -it fakecam /bin/bash
```

# reference

https://elder.dev/posts/open-source-virtual-background/


