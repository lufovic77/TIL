# Part 1: Getting started
> 원문: [https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)

# Orientation and setup

이 페이지는 도커를 시작하는 방법을 단계별로 안내합니다. 이번 튜토리얼에서는 다음을 배울 것입니다:

- 이미지를 컨테이너로서 빌드하고 실행하기
- 도커 허브를 이용해서 이미지 공유하기
- 여러가지 컨테이너와 데이터베이스를 이용해서 도커 어플리케이션 배포하기
- Docker Compose를 이용해서 어플리케이션 실행하기

추가적으로, 이미지를 빌드하는 연습과 이미지의 보안 취약점을 검사하는 방법도 배우게 될 것이다. 

언어에 따라 어플리케이션을 containerize 하는 방법도 있으니 [여길](https://docs.docker.com/language/) 참조하면 된다. 

아래 동영상을 보는것도 추천!

[DockerCon 2020](https://youtu.be/iqqDU2crIEQ)

## 동영상 요약

동영상 내용을 간단히 요약하면 아래와 같다.

- Dockerfile을 통해 이미지를 configure 할 수 있다.
- Docker hub를 이용하면 깃허브 레포처럼 이미지들의 push, pull이 가능하다.
- docker로 시작하는 명령어들은 생각보다 간단하며, 기존 리눅스 커맨드들과 닮아있는 점이 많다.
    - docker image ls: 로컬에 있는 도커 이미지들 보기
    - docker ps: 현재 실행중인 이미지 보기

동영상을 보고 컨테이너의 확실한 강점을 깨달았다. 

동영상에서 빌드한 컨테이너는 아래 Dockerfile에 기반한다. 

```bash
1 FROM node:12.16.3
2 WORKDIR /code
3 ENV PORT 80
4 COPY package.json /code/package.json
5 RUN npm install
6 COPY . /code
7 CMD ["node", "src/server.js"]
```

이 이미지는 node를 이용한 웹 서버로, 구동된 컨테이너에 접속하면 바로 웹 서버처럼 작동하게 된다. 

**즉, 따로 이것저것 내가 설치하고 환경 설정할 필요 없이 그냥 node를 가지고 있는 이미지를 받아와 컨테이너를 빌드하면 바로 웹 서버로 사용할 수 있다는 것!**

# Download and install Docker

아래 튜토리얼을 따라하기 전에 [이전 포스트](https://github.com/lufovic77/TIL/blob/main/docker/docs/Get_Docker.md)를 보고 도커 엔진을 설치해 봅시다.

## Start the tutorial

설치하고 나서, 아래 명령어를 치면 도커가 실행이 되면서 예제 홈페이지에 들어갈 수 있게 된다. 

```java
$ docker run -d -p 80:80 docker/getting-started
```

> 만약 AWS 같은 클라우드 환경을 이용해서 따라해보고 있다면 중간에 있는 포트를 
80:80이 아닌 8080:80으로 바꿔서 해보자. 
그 이유는 [아래](#도커 실행)에서 확인!

명령어에 사용된 몇 가지 플래그들에 대한 정보는 아래와 같다:

- `-d` 컨테이너를 백그라운드에서 실행시킨다.
- `-p 80:80` 왼쪽이 호스트의 포트, 오른쪽이 컨테이너의 포트. 둘을 맵핑 시킨다.
- `docker/getting-started` 사용할 이미지. 여기서는 도커 허브에서 받아와서 쓰는거다.

> 하나의 character로 이루어진 플래그는 합칠 수 있다!

```java
$ docker run -dp 80:80 docker/getting-started
```

## 실행 결과 공유

> 아래는 내가 실행하면서 얻은 결과들이다. 
비교해보면서 실행해보자.

### 로컬에 가지고 있는 도커 이미지 확인하기

```java
kyungmin@ip-172-31-13-114:~/docker_test$ sudo docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
<none>        <none>    fd63e34e4970   6 minutes ago   916MB
hello-world   latest    d1165f221234   3 months ago    13.3kB
node          12.16.3   bdca973cfa07   13 months ago   916MB
```

### 도커 실행

```java
kyungmin@ip-172-31-13-114:~/docker_test$ sudo docker run -d -p 80:80 docker/getting-started
Unable to find image 'docker/getting-started:latest' locally
latest: Pulling from docker/getting-started
ww0db60ca938: Pull complete 
0ae30075c5da: Pull complete 
ssa81141e74e: Pull complete 
b2e41dd2de20: Pull complete 
as40e80wfb2d: Pull complete 
75sd4dc48411: Pull complete 
ddded5c3e3fe: Pull complete 
38a847d4d941: Pull complete 
Digest: sha256:10555bb0c50e13fc4dd965ddb5f00e948ffa53c13ff15dcdc85b7ab65e1f240b
Status: Downloaded newer image for docker/getting-started:latest
4b1459adsadff5f6eff0b8sd0cb2assabd792c8def4dc051sdfsdf8werr0c58
docker: Error response from daemon: driver failed programming external connectivity on endpoint jovial_swanson (d31f23bb8d45b263421103f8156c4f4834fb6ffe91a568c6d512d8b1dd64a6ba): Error starting userland proxy: listen tcp4 0.0.0.0:80: bind: address already in use.
```

결과를 보면 맨 마지막에 

> docker: Error response from daemon: driver failed programming external connectivity on endpoint jovial_swanson (d31f23bb8d45b263421103f8156c4f4834fb6ffe91a568c6d512d8b1dd64a6ba): Error starting userland proxy: listen tcp4 0.0.0.0:80: bind: address already in use.

에러가 난다. 

이미 aws가 80번 포트를 사용해서 나의 로컬 맥북과 통신하고 있기 때문에 사용할 수 없는 포트기 때문이다. 

즉, 내 랩탑 → 80번 포트 → AWS 서버 → 80번 포트 → 도커 컨테이너 

이런식으로 이용하려니까 겹치는거. 

내 랩탑 → 80번 포트 → AWS 서버 → 8080번 포트 → 도커 컨테이너 

이렇게 바꾸면 된다. 

```java
kyungmin@ip-172-31-13-114:~/docker_test$ sudo docker run -d -p 8080:80 docker/getting-started
39bac43e3e8860c738b4ae53b6b203ca1f1369ca9fce866d3ebd2e6a2904e381
```

### 실행중인 컨테이너 확인

```java
kyungmin@ip-172-31-13-114:~/docker_test$ sudo docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS         PORTS                                   NAMES
39bac43e3e88   docker/getting-started   "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   naughty_lamport
```

# The Docker Dashboard

더 가기 전에, 자신의 머신에서 실행되고 있는 컨테이너들에 대한 요약을 보여주는 Docker Dashboard를 소개해본다. Docker Dashboard는 Mac과 윈도우에서 사용할 수 있다. 컨테이너 로그를 손쉽게 볼 수 있으며, 컨테이너 내에서 쉘을 실행할 수 있고, 컨테이너의 생명주기 (stop, remove 같은)를 쉽게 제어할 수 있게 해준다. 

> 대시보드에 접속하려면, 맥이나 윈도우에 맞는 안내서를 보자. 
일단은 아래에 간략하게 요약해보겠다.

## 도커 대쉬보드 접속하기

### Docker Desktop 다운

[도커 홈페이지](https://www.docker.com/get-started)에 들어가서 Docker Desktop을 다운받자. 

### 터미널에서 도커 실행

위의 튜토리얼에서 도커를 실행했다면, 대시보드에 자동으로 내가 띄운 컨테이너가 잡힌다!

---

Dashboard를 열어보면, 현재 튜토리얼이 돌아가고 있는게 보인다. 컨테이너 이름은 실행할 때마다 랜덤하게 부연되며, 이름 (나의 경우 hungry_austin) 오른쪽에 보면 빌드한 이미지 이름을 확인할 수 있다. 

# What is a container?

이제 컨테이너를 실행했는데, 컨테이너란 무엇일까? 간단히 말해서, 컨테이너란 호스트 머신의 다른 모든 프로세스들과는 고립된 프로세스이다. 이런 고립성은 [kernel namespaces and cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504)을 활용한건데, 이건 사실 리눅스에서 오랫동안 사용된 온걸 따온건다. 도커는 이러한 능력들을 접근할 수 있게 하고, 사용하기 쉽게 만들려고 한거다. 

# What is a container image?

컨테이너를 실행할 때, 얘는 독립된 파일시스템을 이용한다. 이런 커스텀 파일시스템은 **컨테이너 이미지**가 제공한다. 이미지는 컨테이너의 파일시스템을 포함하기 때문에, 어플리케이션을 실행하는데 필요한 모든 것들을 가지고 있어야 한다 - 모든 의존성(dependencies), 설정(configuration), 스크립트(scripts), 바이너리(binaries), 등등. 이미지는 또한 컨테이는 위한 다른 설정도 가지고 있어야 하는데, 예를 들어 환경 변수라든지, 실행할 수 있는 기본적인 명령어들, 그리고 다른 메타 데이터들이 있다. 

이미지에 대해서는 나중에 더 깊게 알아볼 것이며, layering, best practices 같은 주제들을 다룰 것이다. 

> 만약 `chroot`에 익숙하다면, 컨테이너를 `chroot`의 확장된 버전이라고 생각해도 된다. 파일시스템은 단순히 이미지로부터 온다. 그러나, 컨테이너는 단순히 chroot를 사용했을 때 가능하지는 않은 추가적인 고립성(isolation)을 제공한다.
