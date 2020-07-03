```
this repo was created as part of an answer for 
https://askubuntu.com/questions/1228501/how-can-i-choose-the-zoom-virtual-background-feature-using-ubuntu/1255945#1255945
```

# summary

Enable virtual background in Linux during your zoom/teams/slack meetings.

+ branch `master` is tested with Ubuntu 20.04, Nvidia GeForce 940MX, and a crappy web cam.
+ branch `cpu-friendly` is tested with Ubuntu 20.04 and no gpu.

Check out the original author @BenTheElder's post for a detailed explaination on what the code is doing.
https://elder.dev/posts/open-source-virtual-background

Also check out a cpu-friendly derivation by fangfufu.
https://github.com/fangfufu/Linux-Fake-Background-Webcam

Finally, for those interested in the underlying bodypix. Here is a detailed blog post on this amazing open source library, there is even a live demo page!
https://blog.tensorflow.org/2019/11/updated-bodypix-2.html

If you just want a tl;dr... we grab image frames from the physical camera with OpenCV.  For each image, the face/body is cropped with a pretrained neural net via TensorFlow.js Bodypix, and merged with a specified background image.  The merged images are then continously generated and used to create a video feed via pyfakewebcam and v4l2loopback.  The implementation is dockerized to 2 containers, with the `bodypix` container handling the body detection, while the `fakecam` container handles the rest of the image processing, and also interfacing with the physical and virtual camera devices.

--


## *** Credit goes to Benjamin Elder @BenTheElder, the original author! ***

# instructions

+ install docker (https://docs.docker.com/engine/install/ubuntu)

+ (skip if you have no GPU) install nvidia-docker (https://github.com/NVIDIA/nvidia-docker)

+ (skip if you have no GPU) test nvidia-docker is install properly
```
docker run --gpus all nvidia/cuda:10.0-base nvidia-smi
```

+ install v4l2loopback
``` 
sudo apt-get upgrade -y
sudo apt-get install -y v4l2loopback-dkms v4l2loopback-utils
```

+ setup virtual video device as `/dev/video20`, and assuming the actual video device is `/dev/video0`. (for me, `/dev/video20` disappears after reboot, so these commands need to be run on boot if you want this device to always appear).
```
sudo modprobe -r v4l2loopback
sudo modprobe v4l2loopback devices=1 video_nr=20 card_label="v4l2loopback" exclusive_caps=1
```

+ confirm virtual camera is created, for my laptop i see `/dev/video0`,`/dev/video1` and `/dev/video20`.
```
find /dev -name 'video*'
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




# misc/notes for development

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


