# Get Docker
- Docker Engine을 설치하는 방법에 대해 알아봅니다.
- 도커 엔진을 설치하면 [Docker overview](https://github.com/lufovic77/TIL/blob/main/docker/docs/Docker_overview.md)에서 본 도커 데몬과 CLI에서 docker 명령어를 사용할 수 있게 됩니다.

이번 설치 과정에서는 Ubuntu 환경, 특히 CLI를 이용한 설치를 안내합니다. 

> 원문: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

> 아래 명령어들을 따라할 때 맨 앞의 $ 표시는 제외하고 입력해주세요

# Install Docker Engine on Ubuntu

도커 엔진을 우분투에 설치하기 전에, 우선 최소 사양들을 맞추고 시작하자.

## Prerequisites

### OS requirements

일단 도커 엔진은 64bit OS에서만 설치가 된다:

- Ubuntu Hirsute 21.04
- Ubuntu Groovy 20.10
- Ubuntu Focal 20.04 (LTS)
- Ubuntu Bionic 18.04 (LTS)
- Ubuntu Xenial 16.04 (LTS)

나는 Ubuntu Bionic 18.04 (LTS) 버전으로 설치를 진행했다. 

### Uninstall old versions

도커의 옛날 버전들인 `docker` , `[docker.io](http://docker.io)` 나 `docker-engine` 들이 설치되어 있으면 지워야한다:

```bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

만약에 아무것도 안 깔려있다고 나오면 괜찮다. 

## Installation methods

도커 엔진을 설치하는 여러가지 방법 중에 이번에는 저장소와 CLI를 이용해서 설치해보겠다. 

도커 엔진 설치가 처음이면 도커 저장소를 먼저 세팅해줘야 한다. 

### Set up the repository

1)  `apt` 패키지 인덱스를 업데이트하자

```bash
$ sudo apt-get update
```

2) 여러가지 패키지들을 설치하자

```bash
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

3) 도커의 공식적 GPG 키를 추가하자

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

4) 아래 명령어를 이용해서 `stable` 저장소를 세팅하자

```bash
$ echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker Engine

1) 아까 한 것처럼 `apt` 패키지 인덱스를 업데이트하자. 

```bash
$ sudo apt-get update
```

2) 만약 도커 엔진과 containerd의 *최신 버전*을 설치하고 싶다면:

```bash
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

2-1) 그게 아니라 레포에 있는 특정 버전을 설치하고 싶다면, 선택해서 설치할 수 있다:

a) 레포에 있는 버전들을 나열하자

```bash
$ apt-cache madison docker-ce
```

b) 그럼 아래 같은 결과들이 쭉 나온다

```bash
docker-ce | 5:20.10.7~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:20.10.6~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:20.10.5~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:20.10.4~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:20.10.3~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:20.10.2~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:20.10.1~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:20.10.0~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.15~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.14~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.13~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.12~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.11~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.10~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.9~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.8~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.7~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.6~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.5~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.4~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
docker-ce | 5:19.03.3~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
```

c) 두번째 열에 나와있는 내용이 버전이다. 아래 명령어의 <VERSION_STRING>을 원하는 버전으로 바꿔서 실행하면 된다. 예를 들어`5:20.10.7~3-0~ubuntu-bionic` 로 바꿔서 실행하면 된다. 

```bash
$ sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```

3) 도커 엔진이 제대로 설치 되었는지 확인하기 위해 `hello-world` 이미지를 실행해본다

```bash
$ sudo docker run hello-world
```

정상적으로 설치되었고 실행되었다면 아래와 같은 결과를 나타낸다. 

```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
