

# summary

Enable virtual background in Linux (tested with Ubuntu 20.04, Nvidia GeForce 940MX)

so you can also participate in playing with 

## *** Credit goes to original author Benjamin Elder @BenTheElder ***


# reference

https://elder.dev/posts/open-source-virtual-background/


# installation

+ install docker (google this)
+ install nvidia-docker (https://github.com/NVIDIA/nvidia-docker)

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

+ test nvidia-docker is install properly
```
docker run --gpus all nvidia/cuda:10.0-base nvidia-smi
```

+ install and setup virtual video device as "/dev/video20", and assuming the actual video device is "/dev/video0"
``` 
sudo apt-get upgrad -y
sudo apt-get install -y v4l2loopback-dkms v4l2loopback-utils

sudo modprobe -r v4l2loopback
sudo modprobe v4l2loopback devices=1 video_nr=20 card_label="v4l2loopback" exclusive_caps=1
```

+ add root to group video
```
sudo usermod -aG video $USER
cat /etc/group | grep video
```

+ build via docker-compose
```
docker-compose build
```

+ start the fake camera via docker-compose
```
docker-compose up

```

+ launch zoom/teams/slack..., select v4l2loopback as webcam

+ live swap background by replacing file `data/background.jpg` - refresh rate hard coded at 3 seconds.


# development

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



