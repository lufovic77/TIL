# Part 5: Persist the DB
눈치 못챘을수도 있지만, 우리의 todo 리스트는 매번 컨테이너 실행할 때마다 싹 다 지워지고 있었다. 왜 그럴까? 컨테이너가 어떻게 작동하는지 한번 알아보도록 하자. 

## The Container's filesystem

컨테이너가 실행될 때, 컨테이너는 자신의 파일시스템으로 이미지에서 다양한 층을 사용한다. 또한 각 컨테이너는 파일을 만들고/업데이트하고/지우기 위한 자신만의 "바닥 공간"을 가진다. 어떠한 변경도 다른 컨테이너는 볼 수 없으며, 심지어 같은 이미지를 사용하고 있어도 그렇다. 

### See this in practice

이걸 실제로 보기 위해서, 두 개의 컨테이너를 시작하고 각각에서 파일을 만들것이다. 우리가 확인 해볼것은 하나의 컨테이너에서 만든 파일이 다른 컨테이너에서는 사용할 수 없는 것이다. 

1) 1부터 10000 사이의 랜덤한 숫자를 가지고 있는 `/data.txt` 파일을 만드는 ubuntu 컨테이너를 시작해보자.

```java
$ docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
```

명령어에 대한 설명을 해보자면, 배쉬 쉘을 시작하여 두개의 명령어를 실행한다 (`&&`가 명령어에 있는 이유). 첫번째 부분은 하나의 랜덤한 숫자를 골라서 `/data.txt`에 쓴다. 두번째 명령어는 그냥 단순히 컨테이너를 계속 실행하도록 한다. 

> 이거 명령어 실행하구 CLI 버튼 눌러서 실행하면 우분투 이미지로 빌드한 컨테이너 - 배쉬 쉘 실행된다!

2) 컨테이너에 `exec`를 날려서 결과물을 볼 수 있는지 확인해보자. 대쉬보드를 열어서 `ubuntu` 이미지를 실행하고 있는 컨테이너의 첫번째 버튼 (CLI)를 클릭하자. 

![https://docs.docker.com/get-started/images/dashboard-open-cli-ubuntu.png](https://docs.docker.com/get-started/images/dashboard-open-cli-ubuntu.png)

우분투 컨테이너에서 터미널이 쉘 실행하고 있는걸 볼 수 있다. `/data.txt` 파일의 내용을 보기 위해 아래 명령어를 실행하자. 명령어 실행하고 나서 터미널을 닫아도 된다. 

```java
$ cat /data.txt
```

만약 명령어 줄이 더 편하면 `docker exec` 명령어를 사용해서 똑같은 결과를 얻을 수 있다. 컨테이너 ID를 얻어서 (`docker ps` 명령어로 얻자) 아래 명령어를 실행해보자. 

```java
$ docker exec <container-id> cat /data.txt 
```

랜덤한 숫자를 봐야한다!

3) 자, 이번에는 다른 `ubuntu` 컨테이너 (같은 이미지로)를 만들어보면 같은 파일이 없음을 볼 수 있다. 

```java
$ docker run -it ubuntu ls /
```

해당 명령어는 컨테이너를 알아서 만들고 결과만 보여준 후, 알아서 삭제된다. 아래가 결과다. 

```java
kyungmin@kakaoui-MacBookPro ~ % docker run -it ubuntu ls /
bin   dev  home  lib32	libx32	mnt  proc  run	 srv  tmp  var
boot  etc  lib	 lib64	media	opt  root  sbin  sys  usr
```

보이듯이 `data.txt` 파일이 없다! 그 이유는 우리가 처음 만든 컨테이너의 루트 디렉토리에만 쓰여졌기 때문 (사실 당연한 얘기)

4) `docker rm -f` 명령어를 이용해서 첫번째 컨테이너를 삭제해보자 (얘는 아직 남아 있음)

```java
$ docker rm -f <container-id>
```
