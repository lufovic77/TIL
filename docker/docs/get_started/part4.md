# Part 4: Share the application
이미지를 빌드 했으니 이제 공유해보자! 도커 이미지들을 공유하려면, 도커 레지스트리를 사용해야 한다. 기본 레지스트리는 도커 허브이며 지금까지 우리가 썼던 이미지들이 있는 곳이다.

> Docker ID
: 도커 ID는 컨테이너 이미지들을 위한 세계에서 가장 큰 라이브러리와 커뮤니티인 도커 허브에 접근할 수 있게 한다.

## Create a repo

이미지를 푸시하려면, 먼저 도커 허브에서 저장소를 만들어야 한다.

1. [Sign up](https://www.docker.com/pricing?utm_source=docker&utm_medium=webreferral&utm_campaign=docs_driven_upgrade) 하고 도커 허브를 이용해서 이미지를 공유하자. 
    1. 앞에서 도커 대시보드를 사용한거랑은 별개임. 무료 플랜으로 가입하자. 
2. [Docker Hub](https://hub.docker.com/)에 로그인 하자. 
3. **Create Repository** 버튼을 클릭하자 (가운데 왼쪽에 있음)
4. 저장소(repo)이름으로는 `getting-started`으로 정하자. 꼭 `Public` 으로 설정하기!

> Private repositories
: 프라이빗 저장소를 이용하면 특정 유저나 팀들만 접속할 수 있다. 다만, 돈 내야함.

5. **Create** 버튼 누르기!

만들고 나서 오른쪽에 보면 **Docker Commands**라는 섹션이 보인다. 이 저장소에 푸시하기 위해서 실행하는 명령어다. 

![https://docs.docker.com/get-started/images/push-command.png](https://docs.docker.com/get-started/images/push-command.png)

## Push the image

1) 도커 허브에서 본 푸시 명령어를 한번 커맨드 라인에서 실행해보자. 여기서 주의할 점은, 명령어가 'docker'에 가서 작용하는게 아니라 본인의 호스트 머신, 네임스페이스에서 작용한다는 것이다. 

```java
$ docker push docker/getting-started
```

> 지금은 위 명령어가 안돼야 정상이다. 로컬 네임스페이스(본인의 머신)을 참조하니까. 근데 Part 1의 동영상을 따라서 해본 사람들은 아마 로컬에 docker/getting-started 이미지가 남아 있어서 될 수도 있다. 따라서 먼저 [Troubleshooting](#Troubleshooting)을 따라해보고 진행하자.

왜 안될까? 푸시 명령어는 docker/getting-started란 이름의 이미지를 찾고 있지만, 찾을 수가 없다. `docker image ls` 명령어를 실행해보면 역시 안보일 것이다 (그냥 getting-started는 보임).

해결하려면, 우리가 가진 이미지에게 다른 이름을 주기 위해 "태그"를 해야한다. 

2) `docker login -u YOUR-USER-NAME` 명령어를 이용해서 도커 허브에 로그인하자. 

3) `docker tag` 명령어를 이용해서 `getting-started` 이미지에게 새로운 이름을 부여하자. 아래에 예시로 나온 `YOUR-USER-NAME`을 꼭 꼭 본인의 도커 ID로 바꿔서 입력하자.

```java
$ docker tag getting-started YOUR-USER-NAME/getting-starteddocker tag getting-started YOUR-USER-NAME/getting-started
```

딱히 별다른 아웃풋은 없다. 

4) 이제 푸시 명령어를 다시해보자. 도커 허브세어 명령어 복사해서 가져오는 거면, 맨 뒤에 `:tagname`은 지워도 된다. 태그를 특정하지 않으면, 도커는 `latest`라는 태그를 이용할거다. 

> 역시 layer 별로 나뉘어서 푸쉬되는 것을 볼 수 있다.

```java
kyungmin@kyungminny docker % docker push lufovic77/getting-started
Using default tag: latest
The push refers to repository [docker.io/lufovic77/getting-started]
aaaac54b9687: Pushed 
bbbb4b73bee3: Pushed 
17bbbb0df5b6: Pushed 
b5b949bbbb24: Pushed 
c29549bbbb68: Mounted from library/node 
efc48abbbb42: Mounted from library/node 
33816ebbbb7a: Mounted from library/node 
9a5d14bbbb50: Mounted from library/node 
latest: digest: sha256:fe3735e537b567b452ede59c9561ba117ad767a528d03dd459d9d48bac256896 size: 2000
```

## Run the image on a new instance

이제 빌드된 이미지가 레지스트리로 푸시되었으니, 이번에는 우리의 컨테이너 이미지를 본적이 없는 완전히 새로운 인스턴스에서 앱을 실행해보자. 이걸 위해 우리는 Play with Docker를 사용할거다. 

1. [Play with Docker](https://labs.play-with-docker.com/)를 열자. 
2. **Login 누르고** 로그인 방법으로 **docker를 선택.** 
3. Docker Hub 계정이랑 연동.
4. 로그인 하면, **ADD NEW INSTANCE** 옵션을 왼쪽 사이드바에서 클릭하자. . 만약에 안보이면 브라우저를 조금 크게 만들어보자. 몇초 있으면 터미널이 브라우저에 뜰거다!!

![https://docs.docker.com/get-started/images/pwd-add-new-instance.png](https://docs.docker.com/get-started/images/pwd-add-new-instance.png)

5. 터미널에서 방금 우리가 푸시한 앱을 실행해보자. 역시 `YOUR-USER-NAME`은 잘 바꿔서 입력. 

```java
$ docker run -dp 3000:3000 YOUR-USER-NAME/getting-started
```

이미지가 다운되는게 보일거고, 결국에는 실행된다. 

먼저 로컬에서 이미지가 있는지 찾아보고, 없다면 허브에서 찾아서 다운받는다. 

6. 위에 IP 부분 옆에 3000이 뜰거다. 그거 누르면 우리가 만든 앱이 나올거! 만약에 자동적으로 안 나타나면 수동으로 "Open Port" 눌러서 3000을 입력해서 넣자. 

## Recap

이번 섹션에서는 이미지를 레지스트리에 푸시함으로서 공유하는 방법을 배웟다. 그리고 그 푸시한 이미지를 실행할 수 있는 인스턴스를 보았다. 이거는 CI 파이프라인에서 꽤나 일반적인데, 파이프라인이 이미지를 만들고 레지스트리, 그리고 생산 환경에 푸시해서 최신 버전의 이미지를 사용할 수 있게 한다. 

지금까지는 정보들을 메모리에 저장했기 때문에 앱을 끄면 다 날라간다. 다음에는 DB를 이용해서 데이터를 persistent하게 유지하는 방법을 알아본다. 

# Troubleshooting

## Push the image - (1)

1) 우선은 현재 로컬에 어떤 도커 이미지가 있는지 찾아보자.

```java
$ docker image ls
```

2) 나는 다음과 같이 나왔는데, 튜토리얼을 진행하려면 `docker/getting-started`가 있으면 안된다. 

```java
$ docker image ls
// 결과
kyungmin@kyungminny docker % docker image ls    
REPOSITORY               TAG       IMAGE ID       CREATED        SIZE
getting-started          latest    996768dd0009   47 hours ago   383MB
<none>                   <none>    1abff7ea3340   5 days ago     383MB
docker/getting-started   latest    083d7564d904   12 days ago    28MB
```

3) 지우고 싶은 이미지를 지운다.

```java
$ docker image rm docker/getting-started
```
