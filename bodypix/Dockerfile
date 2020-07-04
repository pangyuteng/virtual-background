# Base image with TensorFlow GPU requirements
FROM nvcr.io/nvidia/cuda:10.0-cudnn7-runtime-ubuntu18.04
#FROM tensorflow/tensorflow:1.15.2-gpu-py3

# Install node
RUN apt update && apt install -y curl make build-essential \
    && curl -sL https://deb.nodesource.com/setup_12.x | bash - \
    && apt-get -y install nodejs \
    && mkdir /.npm \
    && chmod 777 /.npm

# Install node package dependencies
WORKDIR /src
COPY package.json /src/
RUN npm install

# Setup our app as the entrypoint
COPY app.js /src/

# Ensure we can get enough GPU memory
# Unfortunately tfjs-node-gpu exposes no gpu configuration :(
#ENV TF_FORCE_GPU_ALLOW_GROWTH=true

ENTRYPOINT node /src/app.js
