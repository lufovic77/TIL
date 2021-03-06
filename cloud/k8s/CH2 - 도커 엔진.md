# Ch2

## 2.1 이미지 vs 컨테이너

- 이미지랑 컨테이너는 뭣이며 차이점이 뭘까?
    - 이미지
        - 컨테이너 생성에 필요한 것 (이때 이미지는 read only)
        - 보통 저장소 이름/이미지 이름:태그 로 구성된다. dockerhub/ubuntu:latest 이런식
        - MySQL, 하둡, 스파크 등등 여러가지가 있다.
    - 컨테이너
        - 이미지를 이용해서 컨테이너를 생성한다.
        - 컨테이너는 격리된 시스템 자원과 네트워크를 사용할 수 있는 공간이다.
        - 컨테이너는 이미지에 영향을 주지 않는다.

## 2.2 도커 컨테이너 실습

- 도커 이미지를 받아오고 컨테이너를 만드는 명령어들을 본다.
- 대표적으로 컨테이너를 만드는 명령어는 run, create가 있는데 둘은 차이가 있다.
    - -i -t 옵션은 cli로 입출력을 사용한다는 의미
    - run
        - docker pull (이미지 없으면 받아오기) → docker create → docker start → docker attach
    - create
        - docker pull → docker create 여기서 끝.
        - 들어가고 싶으면 docker start → docker attach를 따로 해줘야한다.

### 2.2.4 컨테이너 외부에 노출시키기

- 컨테이너는 기본적으로 172.17.0.x 번대 ip를 할당 받는다.
- 외부에서 접속하고 싶으면 컨테이너의 ehh0의 IP랑 포트를 호스트에 바인딩 해야한다 (컨테이너↔ 호스트)
    - `docker run -it --name mywebserver -p 80:80 ubuntu:14.04`
    - 중간에 보면 80:80이 있는데, 이거는 [호스트 포트]:[컨테이너 포트] 이렇게 바인딩 한다는 의미
    - 이때, 아파치는 80번 포트로 서비스 되기 때문에 아파치 웹서버를 설치한 상태에서 80:81 이런식으로 바인딩 시키면 서비스가 불가하게 된다.

### 2.2.5 컨테이너 어플리케이션 구축

- 한 컨테이너에서 두개 이상의 어플리케이션을 구동할수도있고 컨테이너 당 어플리케이션으로 구현할 수도 있다.
    - 다만 한 컨테이너에 하나의 프로세스를 실행하는게 도커의 철학이다.
- docker run의 옵션들
    - -d
        - detached 모드로 컨테이너를 실행한다는 의미로, 백그라운드에서 동작하는 어플리케이션이다.
        - 이때, -d로 run을 실행하면 따로 쉘과의 입출력 상호작용이 없는채로 컨테이너를 실행한다.
        - 반드시 컨테이너 내부에서 실행되는 프로그램이 있어야 하며, 없다면 컨테이너는 종료된다.
        - `docker run -d —name detach_test ubuntu:14.04` 처럼 내부에서 실행되는 프로그램이 따로 없으면 컨테이너는 바로 종료된다.
        - 간단히 말해 하나의 컨테이너는 하나의 모니터에 연결된다고 생각하면 된다.
            - 따라서 하나의 컨테이너에 두개의 쉘에서 docker attach 하면 신기하게 하나에서 입력하면 다른 창에서도 똑같이 행동한다 ㅋㅋㅋ
        - 이렇게 백그라운드로 실행된 컨테이너에 쉘로 접속하고 싶으면 ??
            - `docker exec` 명령어를 `-i -t` 옵션과 함께 사용하면 된다.
            - 이 명령어로 접속하면 exit로 나와도 컨테이너는 종료되지 않는다. 그냥 쉘 프로세스 종료한거니까.
    - -e
        - 컨테이너 내부의 환경변수 설정

### 2.2.6 도커 볼륨

- 아까 말했듯이 이미지로 컨테이너를 생성 시 이미지는 RO가 되고 변경사항만 따로 저장한다.
- 그리고 그 위에 컨테이너를 올려서 디비 정보나 다양한 변경사항을 저장한다.
    - 근데 문제는, 컨테이너를 삭제하면 mysql에 저장된 정보들도 다 날라간다. 그래서 데이터를 좀 persistent하게 유지할 방안이 필요한데, 이게 바로 도커 볼륨이다.
1. 호스트와 도커가 볼륨을 공유
    
    ```java
    docker run -d \
    -e MYSQL_ROOT_PASSWORD=password \
    -e MYSQL_DATABASE=wordpress \
    --name wordpressdb_hostvolume \
    -v /Users/kyungmin/Desktop/coding/kube/wordpress_db:/var/lib/mysql \
    mysql:5.7
    ```
    
    - [호스트의 디렉토리]:[컨테이너의 디렉토리] 식으로 호스트와 공유하는것
    - 이렇게 하고 컨테이너 삭제해도 그대로 호스트에는 데이터가 남는다.
    - 이때, 이미 이미지 안의 해당 경로에 데이터가 있더라도 저런식으로 호스트 디렉토리를 마운트 하면 호스트의 디렉토리 정보로 덮어씌우게 된다.
2. 볼륨 컨테이너를 활용
    1. 이해가 어렵네 ..
3. 도커가 관리하는 볼륨을 생성
    - docker volume create 명령어를 이용하여 직접 볼륨을 생성한다.
    - 여기서는 [볼륨 이름]:[컨테이너 내의 디렉터리] 식으로 해야한다.
    - 이러면 디렉터리의 정보가 볼륨에 저장된다.
    - 어디에 저장되는지는 사용자는 몰라도 된다.
- 이런식으로 컨테이너 외부에 데이터를 저장하는게 stateless라고 한다.
- 컨테이너에 데이터가 있으면 stateful하다고 말한다.

### 2.2.7 도커 네트워크

- 컨테이너에서 ifconfig 해보면 eth에 네트워크 정보를 볼 수 있다.
    - 이 ip 주소는 컨테이너 재시작 시 항상 바뀐다.
    - 이때, veth라는 가상 인터페이스가 도커와 호스트를 연결해주기 위해 생긴다.
    - 호스트에서 조회해보면 veth 가 보인다.
    

### 2.2.8 컨테이너 로깅

- 컨테이너 내부의 로그를 확인하는 작업은 디버깅, 운영에서 중요
- json-file 로그를 사용 (default)
    - mysql 컨테이너 생성해보기
    
    ```java
    docker run -d --name mysql \
    > -e MYSQL_ROOT_PASSWORD=1234 \
    > mysql:5.7
    ```
    
    - 이렇게 -d 옵션을 주면 백그라운드에서 컨테이너가 돌고 있기 때문에 `docker logs` 명령어로 로그를 찍어서 상태를 확인할 수 있다.
        - `docker logs mysql`
        - `—tail n` 옵션
            - 마지막 n 줄만 출력
        - `—since {유닉스 시간}` 옵션
            - 특정 시간 이후의 로그 확인
        - `-f` 옵션
            - 실시간으로 로그를 출력한다. 실행해보면 표준 출력을 기다리고 있는걸 볼 수 있다.
    - 기본적으로 이런 컨테이너의 로그들은 JSON 형태로 도커 내부에 저장된다.
        - 도커 내부에서 /var/lib/docker/containers/... 위치에 저장된다.
        - 다만 이 로그들이 너무 쌓이면 저장 공간을 너무 많이 잡아먹을 수 있으므로 `--log-opt` 옵션을 이용해 로그 파일의 최대 크기를 지정 가능.
- syslog 로그를 사용
    - syslog는 유닉스 계열의 OS에서 로그를 수집하는 표준
    - `--log-drvier=syslog` 옵션으로 json-file에서 syslog로 변경 가능
    - 유닉스 계열마다 syslog가 저장되는 위치가 다른데, 맞게 찾아가면 호스트 OS에서 로깅이된걸 확인할 수 있다.
    - `rsyslog`
        - 로그를 원격 서버로 보낼 수 있는 방법
        - 로그 수집용 서버를 따로 만들어 수집할 수 있다.
- fluentd 로깅을 이용
    - 로그를 수집, 저장하는 오픈소스 도구. 도커 엔진이 공식적으로 제공한다.
    - JSON 형식으로 로그를 수집하므로 HDFS나 MongoDB같이 다양한 저장소에 싱크 가능.

    ![Untitled-5](https://user-images.githubusercontent.com/20508932/180609608-3feefa90-7857-4cde-8beb-92201489e544.png)
        
    - 도커 서버 1,2
        - `--log-driver=fluentd`
        - `--log-opt` 로 여러가지 옵션 지정 가능
    - 결론적으로 흐름은 도커 컨테이너가 실행중인 서버 → fluentd 서버 → 몽고DB에 저장
- 아마존 클라우드워치 로그 이용
    - **Identity and Access Management(IAM)**
    - 책에 있는대로 쭈욱 따라해서 클라우드워치를 설정해보자
    - EC2 인스턴스를 생성할 때 IAM을 연결할 수 있고, 해당 인스턴스에 도커 엔진을 설치하면 클라우드 워치로 로그가 수집되는 것을 UI로 볼 수 있다.
    

### 2.2.9 컨테이너 자원 할당 제한

- docker run, create를 할 때 옵션이 없다면 컨테이너는 호스트의 자원을 제한없이 쓰게 된다.
    - 다른 컨테이너와 심하면 호스트 자체에도 성능에 영향을 줄 수 있다.
- `docker inspect {컨테이너 이름}`
    - 컨테이너의 설정값들을 볼 수 있다.
- 메모리
    - `--memory` 옵션
        - 컨테이너의 최대 메모리 제한
        - 이때, 컨테이너의 프로세스가 할당된 메모리를 초과하면 자동으로 컨테이너가 종료된다.
        - `--memory-swap` 옵션
        - 일반적으로 메모리의 2배로 설정
- CPU
    - `--cpu-shares`
        - 컨테이너가 사용할 수 있는 CPU의 양.
        - 정량적인 값이 아니라 ‘상대적인' 값이다. 시스템에 존재하는 CPU를 나눠 가지는 것.
        - 만약 컨테이너A: 1024, 컨테이너B:512로 설정되었다면,
            - A: 69%
            - B: 33%
            - 의 비율에 해당하는 CPU를 나눠서 사용한다.
    - `--cpuset-cpu`
        - 특정 CPU를 사용하게 지정할 수 있다.
        - 호스트에 존재하는 CPU 개수에 맞게 0부터 넘버링을 해서 사용하면 된다.
    - `--cpu-period, --cpu-quota`
        - 얘도 비슷하게 상대적인 비율을 가지고 CPU를 할당해준다.
        - 결과적으로 quota/period 만큼의 CPU time을 할당 받는다.
    - `--cpus`
        - 사용하는 CPU의 개수를 직관적으로 지정한다.
- 파일 입출력 (Block I/O)
    - `--device` 로 시작하는 옵션을 사용하면 블록 입출력을 제한할 수 있다.
        - `--device-write-bps`, `--device-read-bps`, `--device-write-ops`, `--device-read-iops`
        

## 2.3 도커 이미지

- 도커허브
    - apt-get 했을 때 패키지 받는 것처럼 이미지들이 모여있는 중앙 저장소
    - `docker create, run, pull`
    - 단, 누구나 이미지를 올릴 수 있어서 Official 라벨이 없으면 동작을 안할수도 있다.
    - 또한 이미지를 검색해볼 수도 있다.
        - `docker search`

### 2.3.1 도커 이미지 생성

- 받아서 사용할수도 있지만 이번에는 사용자의 환경에서 컨테이너 안에서 작업한 내용을 이미지에 반영하는 방법을 다룬다.
- 컨테이너 내에서 작업한뒤에
    - `docker commit [OPTIONS] [컨테이너 이름] [저장소(이미지 이름):태그]`
        - -a: 작성자 정보
        - -m: 커밋 메세지
        - 이미지 이름의 prefix는 이미지가 저장되는 저장소 이름이여야 한다.
- `docker images`
    - 생성된 사용자의 이미지를 확인해볼 수 있다.

### 2.3.2 이미지 구조 이해

- 어떻게 commit 만으로 빠르고 간단하게 이미지를 만들 수 있을까?
- ubuntu 이미지를 가지고 commit_test:first를 만들고, 반복적으로 이를 이용해 commit_test:second를 만들었다고 하자.
    - ubuntu:14.04 → base 이미지
    - commit_test:first → ubuntu:14.04에서 first 파일을 추가
    - commit_test:second → commit_test:first에서 second 파일을 추가
- `docker inspect`를 이용해 이미지의 정보를 보면, Layers에 대한 정보를 볼 수 있다.
    
    ![Untitled-6](https://user-images.githubusercontent.com/20508932/180609625-3f74fd7a-03f4-4992-a09d-554a57c6f899.png)

    
    - 새로운 레이어가 하나씩 추가된것을 볼 수 있다.
    - 즉, 기존 레이어에서 변경된 사항을 새로운 레이어로 쌓게되는것.
    - 이러면 파일 크기에도 영향이 있는데, ubuntu:14.04가 100mb일때 세 파일의 크기는?
        - (100mb) + (100mb + first 파일 크기) + (100mb + first, second 파일 크기) 가 아니라
        - 100mb + first, second 파일 크기가 된다.
- `docker rmi {이미지 이름:태그}`
    - 이미지를 삭제하는 명령어
    - 해당 이미지를 가지고 실행중인 컨테이너가 있으면 안된다.
    - 이때, first를 삭제해도 second에는 first의 내용이 여전히 유지된다. 그 레이어가 삭제되지 않는다.
    - 비슷하게, second를 삭제한다고 해도 베이스 이미지인 ubuntu가 삭제되지는 않는다.

### 2.3.3 이미지 추출

- 도커 이미지를 추출하고, 다시 도커에 로드해올 수 있다.
- 이때, 이미지의 모든 메타데이터들이 포함되기 때문에 로드해도 이전과 완전히 동일한 이미지가 불러와진다.
    - `docker save -o [파일명] [이미지]`
    - `docker load -i [파일명]`
    - 이거랑 약간 다른게 export랑 import이다.
        - 얘네는 설정정보를 저장하지 않기 때문에 import 시 새로운 이미지로 다시 저장하게 된다.
        - 이렇게 계속 새로운 이미지를 찍어내는건 저장 공간 면에서 비효율적임!

### 2.3.4 이미지 배포

- `save, export`로 배포할 수 있지만, 이미지가 너무 크거나 배포 대상이 되는 도커 엔진이 너무 많으면 쉽지않다. → 이미지를 올려두자
    - 도커 허브 이미지 저장소
        - 개인 저장소
        - 팀 저장소
        - `docker push {저장소이름 접두어 + 이미지 이름:태그}`
    - 도커 사설 레지스트리 - 포트 5000번
        - 개인 서버에 이미지를 저장할 수 있다.
        - 이 사설 저장소용 도커 이미지가 존재해서 `registry:2.6` 으로 받아서 컨테이너를 만들 수 있다.
        - `docker push {ip:5000/이미지이름:태그}`
        - 기본적으로 HTTPS 통신만 허용한다. `--insecure-registry` 옵션으로 따로 지정해줘야 한다.

## 2.4 Dockerfile

### 2.4.1 이미지 생성하기

- 지금까지 우리가 봤던 이미지 생성하는 방법은 다음과 같은 flow 였다.
    - 베이스 이미지를 받자
    - 컨테이너를 만들어서 뚱땅뚱땅 작업하자
    - 잘 구동되는지 보고 컨테이너를 이미지에 커밋하자
    - 새로운 이미지를 만들수 있다.
- 하지만 이러면 좀 번거롭잖아
- 그래서 작업을 기록한 파일을 만들면 도커가 이를 읽어서 바로 이미지를 만들어준다.
- 이게 바로 Dockerfile !!

### 2.4.2 도커파일 명령어

- 한줄이 곧 명령어가 된다.
- 소문자, 대문자 상관없지만 보통 대문자 (RUN, FROM, ADD)
    - FROM
        - 베이스 이미지
        - 반드시 한번이상 입력해야한다.
        - docker run 이미지 이름. 이거랑 같은 의미
    - LABEL
        - 도커 이미지의 메타데이터를 의미
        - 키:밸류 형식으로 저장
        - 나중에 `--filter` 로 라벨을 손쉽게 찾을수도 있다.
    - RUN
        - 컨테이너 내부에서 실행할 명령어
        - 다만 설치 중간에 입력을 받는 패키지가 있을 수 있다. 이러면 뒤에 `-대답`을 붙여야 한다. 안붙이면 오류남!!
    - ADD
        - 이미지에 파일을 추가
        - ADD {호스트 파일경로} {컨테이너 파일 경로}
    - WORKDIR
        - cd랑 같은 의미.
        - 명령어를 실행할 디렉터리를 의미
    - EXPOSE
        - 이미지에서 노출할 포트 번호
    - CMD
        - 컨테이너가 실행될때마다 실행할 명령어
        - 한번만 지정할 수 있다.

### 2.4.3 Dockerfile 빌드

- 이미지 빌드
    - Dockerfile이 존재하는 디렉토리에 들어가서 `docker build -t 이미지이름:태그 {이미지를 저장할 경로}` 를 하면 이미지가 만들어진다.
    - 그 이후 `docker run -d -P` 을 하면 일전에 도커파일에서 EXPOSE로 지정한 포트가 호스트의 임의의 포트와 바인딩된다.
- 빌드 과정 살펴보기
    1. 빌드 컨텍스트 불러오기 
        1. 필요한 파일, 설정, 소스 코드 등
        2. Dockerfile이 위치한 디렉토리 - 하위 디렉토리까지 빌드에 전부 포함
            1. 따라서 Dockerfile이 위치하는 디렉토리에는 웬만하면 이미지 빌드에 필요한 파일만 놓자
            2. 그래서 `.dockerignore`을 이용해서 컨텍스트에서 제외할 수 있다. 
    2. 이미지를 생성하는 과정
        1. 도커파일에 기록된 명령어들이 있으면 (ADD, RUN 같은거) 각 명령어 마다 컨테이너를 생성, 삭제 한다. 
        2. 즉, 한 줄의 명령어를 실행할 때 컨테이너를 생성 → 명령어 실행 → 커밋 → 기존의 이미지에 새로운 레이어로 쌓음 → 컨테이너 삭제 를 반복하는거. 
        3. 명령어의 줄 수만큼 레이어가 쌓이고, 그 만큼 컨테이너의 생성 - 삭제가 반복된다. 
    3. 캐시로 이미지 빌드하기 
        1. 이미지 빌드를 마치고 나서 같은 빌드를 다시 진행하면.. 이전 빌드의 캐시를 이용한다. 
        2. 이전에 빌드했을 때 사용한 Dockerfile이랑 같은 명령어들이 포함되어 있으면 새로 빌드하지 않고 이전에 사용했던 이미지 레이어를 가져와서 빌드한다. 
        3. 문제는 .. 캐시를 사용하고 싶지 않을때도 자동적으로 이전 빌드 내용을 재사용한다는 점이다. 
            1. 예를 들어 git clone을 하여 이미지를 만들었는데, 원격 저장소에 변경 사항이 생겼다.
            2. 새로운 코드를 좀 받아오고 싶지만 새로운 이미지를 만들때의 git clone은 예전 버전의 소스코드를 가져오게 될것이다. 
            3. 이럴 때 `--no-cache`옵션을 추가하면 된다. 
- 멀티 스테이지를 사용하기
    - 간단한 프로그램이더라도 구동하는데 필요한 패키지나 프로그램들이 많으면 최종적으로 완성되는 이미지의 크기는 매우 클 수 있다.
    - 이런 경우 최종 이미지의 크기를 줄이기 위해서 멀티 스테이지 빌드라는 개념을 가져온다.
    - 이는 하나의 Dockerfile안에 여러개의 FROM {이미지}를 사용하는 방식이다.

### 2.4.4 기타 Dockerfile 명령어

- ENV
    - 환경변수 설정
    - ENV var {?}
    - docker run 시 `-e`옵션으로 덮어쓸 수 있다.
- VOLUME
    - 컨테이너를 생성시 호스트와 공유할 컨테이너 내부의 디렉토리 지정
- USER
    - 사용자의 계정을 설정하는 옵션.
    - 지정안하면 기본적으로 root 권한을 부여하기 때문에 웬만하면 이미지 생성 시 계정을 만들어주자.
- ONBUILD
    - 이미지 A의 마지막에 `ONBUILD RUN {명령어}` 를 추가하고 이미지 A를 이용해 이미지 B를 생성한다고 하자.
    - A 빌드시에는 명령어가 출력되지 않는다.
    - 하지만 A의 마지막에 ONBUILD로 추가된 명령어가 B 빌드 시 출력된다.
- ADD와 COPY
    - 기능적인 측면으로 봤을때는 같다. 로컬의 컨텍스트로부터 이미지에 파일을 복사한다.
    - 하지만 다음과 같은 차이가 있다.
        - COPY: 로컬의 파일만 이미지에 추가할 수 있다.
        - ADD: 외부 URL이나 tar 파일(자동으로 압축 해제)에서도 파일을 추가할 수 있다.
        - ADD가 더 넓은 개념 !
    - ADD가 더 유연하긴 하지만 어떤 파일들이 포함될지 모르니까 사용을 권장하지는 않는다.
- ENTRYPOINT와 CMD
    - CMD: docker run 뒤의 명령어처럼 컨테이너 시작 시 실행할 명령어를 지정
    - ENTRYPOINT: 얘를 설정하고 docker run 맨 뒤에 명령어를 치면 그 명령어가 인자로 치환된다.
        - `docker run -it --entrypoint"echo" --name yes_entrypoint ubuntu:14.04 /bin/bash`
        - entrypoint가 없다면 쉘이 실행되면서 입출력을 할 수 있다.
        - 다만 저렇게 둘이 공존하면 cmd가 인자로 치환되면서 `echo /bin/bash`가 되어 “/bin/bash”를 출력하게 된다.
    - 둘중에 하나는 꼭 지정해야한다!
    - 이렇게 단일 명령어를 실행할수도 있지만.. 보통을 스크립트를 실행하도록 한다.
        - entrypoint에 스크립트의 이름을 넣어준다.
        - 이때, 이 스크립트는 컨테이너 내부에 있어야 한다.
        - 도커 파일에서 ADD나 COPY로 미리 넣어주어야 한다.
        - 스크립트에서 필요한 인자들은 cmd를 활용해서 넣어주자.

### 2.4.5 깔끔한 Dockerfile을 사용하자

- 만약 하나의 도커파일에서
    1. 100mb 크기의 파일 생성
    2. 파일 지우기
- 두개의 명령어가 연속적으로 존재한다고 해보자.
- 이럴때 최종 이미지의 크기네 100mb가 반영이 될까 안될까 ?
    - 반영이 된다...!
- 그 이유는 하나의 명령어 당 하나의 이미지 레이어를 쌓기 때문이다.
- 그래서 결과적으로 이미지에 파일은 존재하지 않지만 두번째 레이어에 ‘파일 지움!’의 변경사항이 저장될 뿐 첫번째 레이어에는 파일이 여전히 남아 있다.
- 이걸 방지하려면 ..?
    - 명령어를 하나로 퉁쳐서 사용하면 된다.
    - `&&`를 이용하면 여러개의 명령어를 하나로 합칠 수 있다!

## 2.5 도커 데몬

- 도커 자체에 대해서 자세히 알아보자

### 2.5.1 도커의 구조

- 그냥 docker가 있고, 도커 데몬이라는 dockerd가 있다.
    - 흐음 ..... 맥에서는 경로가 조금 다른거같다
- 도커를 클라이언트 + 서버로 분리할 수 있다.
    - 도커서버
        - 컨테이너를 생성하고 실행, 이미지를 관리
        - 서버로서 입력을 받을 준비가 된 상태를 도커 데몬이라고 한다.
    - 도커 클라이언트
        - CLI로 명령을 받아 이를 dockerd에게 API를 통해 요청한다
        - 유닉스 소켓을 이용해 통신한다
- 흐름
    - 사용자가 `docker`로 시작하는 명령어를 도커 클라이언트에게 입력
    - 유닉스 소켓을 이용해 도커 서버에게 전송
    - 도커 데몬은 명령어를 파싱하고 수행
    - 결과를 클라이언트에게 반환

### 2.5.2 도커 데몬 실행

- 우분투에서는 도커가 설치되면 자동으로 실행된다.
    - `service docker start/stop`
    - 위 명령어로 실행할수도 있고, 간단하게 `dockerd`만 쳐도 도커데몬이 실행된다.

### 2.5.3 도커 데몬 설정

- 이전 절에서 `dockerd`로 도커 데몬을 실행하는 방법과 `service`로 실행하는 방법을 봤다.
- 도커 데몬에는 지정할 수 있는 옵션들이 매우 많다
    - `dockerd`명령어를 직접 사용해서 도커 데몬을 실행할 때는 뒤에 쭉 옵션을 적으면 된다.
    - 실제로 사용하는 방법은 `/etc/default/docker`설정 파일을 수정하여 `DOCKER_OPTS`에 옵션을 추가해 서비스 명령어로 도커 데몬을 실행한다.
- `-H` 옵션
    - 도커 클라이언트가 dockerd와 통신하는 방법은 디폴트로 유닉스 소켓을 이용
    - -H 옵션을 이용해 IP 주소와 포트번호를 입력하면 원격 API를 통해 로컬이 아니더라도 원격의 도커 데몬을 제어할 수 있다.
        - RESTful API + HTTP 요청으로 제어
    - 다만, 이거 하나만 설정하면 유닉스 소켓을 비활성화 되어서 docker 명령어를 사용할 수 없다.
        - 따라서 유닉스 소켓과 IP 주소 둘다 입력해서 제어하는게 일반적
    - 어차피 로컬 도커데몬한테 요청할때도 API를 사용하고, 이는 원격에 요청 보낼때도 동일한 API를 사용하게 된다.
        - curl로 API 콜하는 것처럼 -H에 넘긴 ip 주소에 요청 날리면 동일한 결과를 받을 수 있다.
- 보안 적용 `--tlsverify`
    - 도커 데몬에 TLS 보안을 적용하고, 인증을 도입해 클라이언트를 제어해보자
        - `openssl`을 이용하여 키도 만들고, 인증서 파일도 만들수 있다!
    - 키를 만들어서 docker 명령어를 날릴때 지정해서 날리면 된다.
        - 다만 매번 키 정보를 붙이기 번거롭기 때문에 이 또한 환경변수로 설정해서 저장할수 있다 .
- 스토리지 드라이버 변경 `--storage-driver`
    - 운영체제 마다 사용하는 스토리지 드라이버가 다르다 (맥은 overlay2)
    - 드라이버도 변경할 수 있는데, overlayFS, AUFS, Btrfs 등등이 있다.
        - 드라이버 별로 컨테이너와 이미지가 생성된다.
        - 다른 드라이버로 만든 컨테이너와 이미지를 사용할수 없다.
    - 스토리지 드라이버의 원리
        - 이미 배웠던 것은
            - 이미지는 RO
            - 컨테이너를 실행시 이미지 위헤 컨테이너 레이어를 생성한다
        - 실제 어떻게 동작하는지는 CoW(Copy-on-Write)와 RoW(Redirect-on-Write)를 살펴봐야한다.
            - snapshot
                - 원본 파일의 특정 상태를 RO상태로 사용하고, 변경되는 사항들을 새로운 공간을 사용해 기록
            - CoW
                - 말그대로 복사해서 쓰기 작업을 하는것
                - 원본 파일을 스냅숏 공간으로 복사한다. (쓰기작업 +1)
                - 수정한다
                - 이를 다시 원본 파일에 적용한다.  (쓰기작업 +1)
                - 총 두번의 쓰기작업. 오버헤드가 크다.
            - RoW
                - 한번의 쓰기작업만 일어난다.
                - 변경점을 기록해서 그대로 스냅숏 공간에 기록한다. (쓰기작업 +1)
                - 이게 바로 컨테이너가 작동하는 방식과 비슷
                - 이미지 레이어 → 각 snapshot
                - 컨테이너 → snapshot의 변경점 (이미지를 수정해서 새로운 스냅숏을 만든다)
    - 지금부터 여러가지 스토리지 드라이버를 살펴보자
    - AUFS
        - 가장 일반적으로 사용하는 드라이버
        - 여러개의 이미지 레이어를 union mount point로 제공
            - 컨테이너 레이어가 여기에 마운트해서 이미지를 사용
        - 이미지를 변경해야한다면
            - 컨테이너 레이어로 전체 파일을 복사 후 변경
            - 이미지 레이어가 여러개 있기 떄문에 변경해야하는 레이어 찾으려면 쭉 훑어야해서 시간이 걸릴수 있다.
            - 그래도 한번 컨테이너 레이어로 복사해두고 나면 이걸로 계속 작업해서 빠르다
    - Devicemapper
        - CentOS등에서 보편적으로 잘 쓰였지만 이제는 deprecated 된 드라이버
        - 하나의 큰 data라는 파일을 만들어서 이를 자원 풀로 설정
            - 이미지와 컨테이너에게 블록 단위로 공간을 할당한다.
            - allocate-on-demand (64KB 크기의 블록 개수를 필요한만큼 풀에서 받아 쓰기작업 수행)
        - 변경하려는 블록만 원본파일에서 컨테이너로 복사해 수정한다.
            - AUFS는 전체 파일 복사했었는데, 얘는 그게 아니니까 성능상의 이점이 있긴해도 컨테이너의 생성과 삭제가 빠른편은 아니다.
        - 단점은 하나의 스토리지 풀을 사용하니까 각 컨테이너와 이미지를 분리해 관리할수가 없다는것.
    - OverlayFS (overlay, overlay2)
        - 대부분의 운영체제에서 자동으로 사용하는 드라이버
        - overlay2가 성능이 좀 더 좋고 이미지 레이어를 여러개 제공한다.
        - overlay
            - lowerdir
                - 단일화된 이미지 레이어 사용
                - 여러개의 이미지 레이어를 컨테이너 마운트 지점에서 통합해서 사용
            - upperdir
                - 컨테이너 레이어
                - 컨테이너에서 발생한 변경사항 (overlayfile)
            - merged
                - lowerdir과 upperdir 이 함께 마운트되어 최종적으로 컨테이너 내부에 보여진다.
        - 얘도 이미지에 변경사항 적용할 때 AUFS에서처럼 이미지를 upperdir로 복사해와서 적용한다.
            - 다만, 이미지 레이어가 하나이기 때문에 그렇게 오래 시간이 걸리지는 않는다!
    - Btrfs
        - 앞에서와 다르게 별도로 FS를 구성해야 도커에서 사용할 수 있다.
            - 도커가 위치한 디렉토리를 btrfs 파일시스템을 사용하는 공간에 마운트 해야만 한다.
        - 이미지와 컨테이너를 서브 볼륨, 스냅숏 단위로 관리
            - 서브볼륨: 이미지의 가장 아래에 있는 base layer
            - 스냅숏: 그 위에 쌓이는 child layer들
            - 따라서 새로운 컨테이너 생성시 맨 위에 스냅숏으로 쌓인다.
        - RoW 방식 (원본 파일은 보존)
    - ZFS
        - 자세한건 생략!
        - RoW 방식 사용
        - 안정성이 뛰어나긴한데 꽤나 무겁다. 메모리 사용량이 높음.
- 컨테이너 저장 공간 설정
    - 각 스토리지 드라이버마다 컨테이너에게 저장 공간을 할당하는 방식이 다르다.
        - AUFS, overlay2: 호스트의 저장공간과 컨테이너의 저장공간 크기가 같다
        - devicemapper: 기본적으로 10GB 할당, 저장공간의 크기를 제한할수도 있다.
        - overlay2
            - project quota 기능을 이용해 저장공간을 제한할 수 있다. (다만 파일 시스템을 xfs로 포맷해야한다.)

### 2.5.4 도커 데몬 모니터링

- 도커 자체가 지원하는 모니터링도 있고, 각종 오픈소스도 있다.
- 도커 데몬 디버그 모드
    - 도커 데몬을 디버그 옵션으로 실행 (`-D` 옵션)
    - API의 입출력 뿐만 아니라 모든 명령어를 로그로 출력
    - 다만, 너무 많은 정보가 나오고 도커 데몬을 포그라운드로 해야 콘솔 출력으로 나오므로 한계가 있다.
- `events`, `stats`, `system df` 명령어
    - events
        - 도커 자체가 제공하는 기능
        - 도커 데몬에 일어나는 일을 실시간 스트림 로그로 보여준다.
        - `docker events`
        - 다만 클라이언트에서 입력하느 모든 명령어가 나오지는 않는다.
            - 컨테이너 관련 명령어 (attach, commit...)
            - 이미지 관련 명령어 (delete, import ...)
            - 볼륨, 네트워크 관련 명령어 등등이 나온다.
        - 얘도 `--filter`로 원하는 정보만 볼 수 있다.
    - stats
        - `docker stats`
        - 실행중인 모든 컨테이너의 자원 사용량을 출력 (스트림 형태 출력)
    - system df
        - 도커에서 사용중인 이미지, 컨테이너 등 사용중인 공간과 사용 안하는 것들을 삭제해서 확보할 수 있는 공간을 보여준다.
        - `docker system df`
        - 앞에서도 봤지만 `docker container/volume/image prune`으로 사용중이지 않은 대상을 삭제할 수 있다.
- CAdvisor
    - 구글이 만든 컨테이너 모니터링 도구
    - 호스트의 8080번 포트로 웹 UI로 확인이 가능하다.
    - 꽤 자세한 정보를 보여주는데 어떻게 그럴수 있을까?
        - 호스트의 모든 디렉토리를 CAdvisor 컨테이너에 마운트했기 때문
    - 하지만 단일 도커 호스트만을 모니터링 할 수 있다.
        - 여러개의 호스트가 존재하는 도커 클러스터에서는 용도에 맞지 않다.
        - 일반적으로는 오케스트레이션 툴을 설치하고 프로메테우스, influxDB로 데이터를 수집한다.

### 2.5.5 Remote API 라이브러리를 이용한 도커 사용

- 다양한 언어로 API를 호출해 도커를 제어할 수 있다.
- 자바, 파이썬
