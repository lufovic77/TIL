# Part 2: Sample application
> 원문: [https://docs.docker.com/get-started/02_our_app/](https://docs.docker.com/get-started/02_our_app/)

이 튜토리얼의 나머지 부분에서는, 우리는 Node.js로 구동되는 간단한 todo 리스트 매니저를 만들어 볼 것이다. Node.js랑 익숙하지 않더라도, 걱정말자. 자바스크립트의 경험은 필요하지 않다. 

이 시점에서, 당신의 개발 팀은 꽤 작을 것이고 당신은 단순히 당신의 MVP (minimum viable product - 실행이 되는 최소한의 프로덕트)를 증명하기 위해 앱을 만드는 것 뿐이다. 대규모 팀, 여러 개발자에게 어떻게 작동하는지 생각할 필요 없이 어떻게 작동하는지, 무엇을 할 수 있는지 보여주려고 합니다.

# Get the app

어플리케이션을 실행하기 전에, 우리의 머신으로 어플리케이션의 소스코드를 받아올 필요가 있다. 진짜 프로젝트에서는 통상적으로 레포를 클론해올 것이다. 그러나 이번 튜토리얼에서는, 어플리케이션을 포함하는 ZIP 파일을 만들어두었다. 

1. [앱 내용물을 다운받는다](https://github.com/docker/getting-started/tree/master/app). 전체 프로젝트를 pull 하든 zip파일로 다운받아서 앱 폴더에 압축을 풀어놔도 된다. 
2. 압축 풀고나면, 좋아하는 코드 에디터로 프로젝트를 열어보자. 에디터가 필요하면, Visual Studio Code를 다운받아서 쓰자. `package.json` 과 두개의 서브 디렉토리가 보일 것이다(`src`랑 `spec`)

![https://docs.docker.com/get-started/images/ide-screenshot.png](https://docs.docker.com/get-started/images/ide-screenshot.png)

# Build the app’s container image

어플리케이션을 빌드하려면, `Dockerfile`이 필요하다. 도커파일은 컨테이너 이미지를 만드는데 사용되는 단순한 텍스트 기반의 스크립트 파일이다. 도커파일을 만들어본 적이 있다면, 아래 도커파일에서 몇 가지 흠들을 찾아볼 수 있다. 걱정하지 말자. 계속 해볼 것이다.

1) `Dockerfile`이라는 이름의 파일을 아까 받은 `package.json`이 있는 폴더에 아래의 내용으로 만들어주자.

```java
# syntax=docker/dockerfile:1
 FROM node:12-alpine
 RUN apk add --no-cache python g++ make
 WORKDIR /app
 COPY . .
 RUN yarn install --production
 CMD ["node", "src/index.js"]
```

`Dockerfile`이 `.txt`와 같은 파일 확장자가 없도록 하자. 어떤 에디터들은 자동으로 파일 확장자를 추가해서 다음 단계에서 에러가 나게 할수도 있다. 

2) 터미널을 열어서 `Dockerfile`이 있는 `app` 디렉토리(우리가 위에서 받은 프로젝트에서)로 가자. 이제 `docker build`명령어로 컨테이너 이미지를 빌드해보자. 

```java
$ docker build -t getting-started .
```

> 마지막에 .(dot) 포함해서 치자.
아래는 쭉 Dockerfile에 대한 설명.

이 명령어는 도커파일로 새로운 컨테이너 이미지를 만드는데 사용된다. 다운 받는거 보면 아마 엄청 많은 "layers"들이 다운된 것을 볼 수 있을 것이다. 이것은 우리가 도커파일에서 `node:12-alpine`으로 시작하고 싶다고 빌더에게 알려주었기 때문이다. 그러나, 아직 그게 우리의 머신에는(본인 컴퓨터) 없기 때문에, 이미지를 다운받아야 한 것이다.

이미지를 다운 받은 후에는, 어플리케이션 내부로 복사하고 의존성을 어플리케이션의 dependecies를 설치하기 위해 `yarn`을 이용했다. `CMD`지시어는 이미지로부터 컨테이너를 시작할 때 실행할 기본 명령어를 지정한다. 

마지막으로, `-t` 플래그는 이미지에 태그를 지정한다 (뒤의 getting-started). 이걸 그냥 최종 이미지의 사람이 읽을 수 있는 이름이라고 생각하자. 이미지를 `getting-started`라고 이름 지었기 때문에, 컨테이너를 실행할 때 해당 이미지를 참조할 수 있다. 

`docker build` 명령어의 마지막에 있는 `.`은 도커가 현재 디렉토리에 있는 `Dockerfile`을 찾아봐야 함을 알려준다. 

# Start an app container

이제 이미지가 있으니, 어플리케이션을 실행해보자. 그러려면 `docker run`명령어를 사용해야 한다. 

1) `docker run` 명령어를 이용해서 컨테이너를 시작하고 우리가 방금 만든 이미지를 이름을 이용해 지정하자:

```java
$ docker run -dp 3000:3000 getting-started
```

`-d`와 `-p`플래그를 기억하는가? 새로운 컨테이너를 "detached"모드 (백그라운드에서 실행)로 실행중이며, 호스트 머신의 포트 3000번과 컨테이너의 포트 3000번을 맵핑한다. 포트 맵핑 없이는, 어플리케이션에 접근할 수가 없다. 

2) 몇초 뒤에 브라우저를 열어서 [http://localhost:3000](http://localhost:3000)를 열어보자. 우리가 실행한 앱이 보일 것이다. 

![https://docs.docker.com/get-started/images/todo-list-empty.png](https://docs.docker.com/get-started/images/todo-list-empty.png)

3) 한 두개의 아이템을 추가해봐서 예상한대로 작동하는지 보자. 아이템을 완료되었다고 표시할 수도 있고, 지울수도 있다. 프론트 단이 성공적으로 아이템을 백엔드 단에 저장하고 있다. 꽤 쉽죠잉~?

이 시점이 되면, 너가 빌드한 todo 리스트를 실행할 수 있게 된다. 이제, 몇가지 수정을 해보고 컨테이너를 관리하는 법에 대해 배워보자. 

Docker Dashboard를 보면 (직전 컨텐츠에서 설명했다), 두개의 컨테이너가 보일 것이다 (하나는 튜토리얼 시작 때 만든거랑 하나는 이번에 새로 만든거)

![https://docs.docker.com/get-started/images/dashboard-two-containers.png](https://docs.docker.com/get-started/images/dashboard-two-containers.png)

# Recap

이번 섹션에서, 컨테이너를 빌드하는 아주 기본적인 것들과 그러는데 필요한 Dockerfile을 만드는 법에 대해 배웟다. 이미지를 한번 빌드하고 나면, 컨테이너를 시작했고 실행되는 앱을 볼 수 있었다. 

다음으로는, 우리의 앱을 수정하고 어플리케이션을 새로운 이미지로 업데이트 하는 방법에 대해 배운다.
