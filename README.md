# Docker image for ARM64(aarch64) based on x86_64
This Docker image provides the Open Source edition of the Kong Gateway (see
 https://github.com/Kong/kong) for ARM64(aarch64) based architectures.
If you want to run kong on arm, you can find help [here](https://github.com/svenwal/kong-arm)

## Prerequesites
When compiling this image on a non ARM machine you need to have QEMU https://www.qemu.org being installed and activated.
Just execute
```bash
# Only need to be executed once
docker run --rm --privileged multiarch/qemu-user-static:register --reset
```
See https://blog.hypriot.com/post/setup-simple-ci-pipeline-for-arm-images for more details.

## Usage
### build kong 1.0.0
```bash
# Only need to be executed once
docker run --rm --privileged multiarch/qemu-user-static:register --reset
# build image for kong-1.0.0
git clone https://github.com/Bevisy/kong-arm64.git

cd 1.0.0
docker build -t kong:1.0.0_arm64 .
```
PS: Because we can't use luarocks to install kong-1.0.0, we install it from
 source.
 **This version is not veritified.**


### build kong 1.3.0
```bash
# Only need to be executed once
docker run --rm --privileged multiarch/qemu-user-static:register --reset
# build image for kong-1.3.0
git clone https://github.com/Bevisy/kong-arm64.git

cd 1.3.0
docker build -t kong:1.3.0_arm64 .
```
PS: install kong by luarocks. You can install other version by luarocks refer
 to https://luarocks.org/modules/kong/kong.  
 If you don't want to build again, you can try this image [bevisy/kong:1.3.0_arm64](https://hub.docker.com/layers/bevisy/kong/1.3.0_arm64/images/sha256-b00a0b13b9c9d6ac217009d57d000942cdc423da9d855919b7d4cb96829deb76).   
 **I have already tested it on my own arm64 machine.**
