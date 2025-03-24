---
title: Install nvidia container toolkit in Ubuntu 22.04
date: 2022-10-09
categories: [WSL]
tags: [wsl, docker, ubuntu]     # TAG names should always be lowercase
---

## Introduction
우분투 22.04 이상에서 nvidia-container-toolkit을 설치하기 위해 nvidia에서 제공하고 있는 설치 방법을 따라하면 `apt-key`가 deprecated 되었다는 Warning이 계속 뜨기 때문에 이를 해결하고자 함.
  
## Solution
* nvidia의 가이드에서는 apt-key를 이용하는데 이는 Warning을 유발함
* 따라서 nvidia의 gpgkey를 개인 키링에 `nvidia.gpg` 이름으로 저장함 (line 2)
```
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | gpg --dearmor | sudo tee /usr/share/keyrings/nvidia.gpg > /dev/null
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```
* 위 작업 완료 후 키 파일 명시를 위해 다음 작업 실행
```
$ vi /etc/apt/sources.list.d/nvidia-docker.list
```
* 위와 같이 텍스트 편집기 실행 후 아래와 같이 키링 파일 내용 삽입
* `[signed-by=/usr/share/keyrings/nvidia.gpg]`을 다음과 같이 삽입하면 됨.
```
deb [signed-by=/usr/share/keyrings/nvidia.gpg] https://nvidia.github.io/libnvidia-container/stable/ubuntu18.04/$(ARCH) /
#deb [signed-by=/usr/share/keyrings/nvidia.gpg] https://nvidia.github.io/libnvidia-container/experimental/ubuntu18.04/$(ARCH) /
deb [signed-by=/usr/share/keyrings/nvidia.gpg] https://nvidia.github.io/nvidia-container-runtime/stable/ubuntu18.04/$(ARCH) /
#deb [signed-by=/usr/share/keyrings/nvidia.gpg] https://nvidia.github.io/nvidia-container-runtime/experimental/ubuntu18.04/$(ARCH) /
deb [signed-by=/usr/share/keyrings/nvidia.gpg] https://nvidia.github.io/nvidia-docker/ubuntu18.04/$(ARCH) /
```
* 주석 처리된 부분은 안해도 되는데 그냥 했음
```
$ sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
```
* 패키지 정상 설치 및 Warning 안뜨는거 확인.

## References
1. [INSTALLING DOCKER AND THE DOCKER UTILITY ENGINE FOR NVIDIA GPUS - NVIDIA](https://docs.nvidia.com/ai-enterprise/deployment-guide/dg-docker.html#installing-docker-and-the-docker-utility-engine-for-nvidia-gpus)
1. [Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead - Stack Overflow](https://stackoverflow.com/questions/68992799/)
