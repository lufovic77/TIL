# Part 3: Update the application
작은 기능 요청으로, todo 리스트에 아무것도 없으면 "empty text"를 띠우게 변경해 달라는 요청을 받았다. 다음과 같이 바꾸길 원한다:

> You have no todo items yet! Add one above!

꽤 간단하니 한번 해보자 ㅎㅎ

# Update the source code

1) `src/static/js/app.js` 파일에서, 56번 라인을 새로운 텍스트를 사용하도록 수정하자

```java
-   <p className="text-center">No items yet! Add one above!</p>
+   <p className="text-center">You have no todo items yet! Add one above!</p>
```

2) 예전에 썼던 동일한 명령어로 업데이트 된 이미지를 빌드해보자. 

```java
$ docker build -t getting-started .
```

3) 업데이트 된 코드로 새로운 컨테이너를 시작해보자. 

```java
$ docker run -dp 3000:3000 getting-started
```

아마도 다음과 같은 에러를 만날 수도 있다!

```java
docker: Error response from daemon: driver failed programming external connectivity on endpoint laughing_burnell 
(ShaID): Bind for 0.0.0.0:3000 failed: port is already allocated.
```

이거 내가 전에 정리 했던건데, 옛날 컨테이너가 아직 실행중이라서 새로운 컨테이너는 시작할 수 없는거다. 기존의 컨테이너가 여전히 호스트 머신의 3000번 포트를 사용중이며 하나의 프로세스만이 특정 포트로 listen할 수 있어서 에러가 난 것이다. 옛날 컨테이너를 종료하고 삭제한 다음에 다시 해보자 ㅎㅎ

# Replace the old container

컨테이너를 삭제하려면, 일단 멈춰야 한다. 멈추고 나서 지울 수 있다. 지우는 두가지 방법이 있는데, 아무 방법이나  괜찮으니 가장 편한 방법으로 하자. 

## CLI로 삭제하기

1) `docker ps`명령어를 이용해 컨테이너의 ID를 가져오자

```java
$ docker ps
```

아래와 같은 결과를 볼 수 있다. 

```java
kyungmin@kyungminny app % docker ps -al
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                                       NAMES
9959f66fb46c   getting-started   "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   fervent_aryabhata
```

2) `docker stop`명령어로 컨테이너를 멈춘다. 

```java
$ docker stop <본인_컨테이너_ID>
```

나 같은 경우는 `9959f66fb46c`이 ID다.

3) 컨테이너 멈추고 나면, `docker rm`을 이용해서 지울 수 있다. 

```java
$ docker rm <본인_컨테이너_ID_>
```

## Docker Dashboard로 삭제하기

이건 진짜 간단하다. 그냥 도커 대시보드 열어서 클릭해서 삭제하면 된다. 

1. 대시보드 열고, 왼쪽 메뉴의 Containers/Apps 보면 컨테이너 리스트가 쭉 보인다. 
2. 커서 올리고 휴지통 모양 클릭
3. 삭제 확인! 끝!

![https://docs.docker.com/get-started/images/dashboard-removing-container.png](https://docs.docker.com/get-started/images/dashboard-removing-container.png)

## Start the updated app container

1) 이제, 업데이트 된 앱을 시작하자

```java
$ docker run -dp 3000:3000 getting-started
```

2) [http://localhost:3000](http://localhost:3000/) 를 다시 열면 아래와 같이 업데이트 된 텍스트가 보인다!

![https://docs.docker.com/get-started/images/todo-list-updated-empty-text.png](https://docs.docker.com/get-started/images/todo-list-updated-empty-text.png)

# Recap

업데이트 빌드해보면서 두 가지 사항을 아마 눈치 챘을 것이다. 

- 우리가 일전에 만들었던 todo 리스트가 모두 사라졌다. 이러면 안되지~
- 작은 변화를 주기 위해 많은 단계들이 있었다. 다가오는 섹션에서는 수정 소요가 있을 때마다 다시 빌드하고 새로운 컨테이너를 시작하지 않고 코드를 어떻게 업데이트 하는지에 대해 배워본다.
