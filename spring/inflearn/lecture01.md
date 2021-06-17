# IntelliJ

## 기본

- 요즘에는 eclipse 거의 안쓰고 다들 intelliJ로 넘어왔다고 합니다.
    - 단축기도 그렇고 여러가지 개발 편의성을 추가해줬다는데, 써보니 진짜 좋긴 합니다 ㅎㅎ
- [start.spring.io](http://start.spring.io)
    - spring boot 기반으로 프로젝트를 생성해주는 사이트.
    - Maven Project, Java, 2.5.1 버전으로 선택했다.
    - Dependecies
        - 이번 강의에서는 Spring Web, thymeleaf 사용
            - Spring Web: 말그대로 스프링으로 웹서버 구현하려는게 목적
            - thymeleaf: html 만들 때 필요한 것들
    - 이제 GENERATE를 해서 다운 받고 IntelliJ에서 임포트 하면 끝!

## 코드 & 빌드

- 거의 표준화 되어 있는게 src 안에 main, test가 넣어져 있다.
    - main: 실행 코드 작성
    - test: 테스트 코드 작성
- build.gradle
    - 요즘에는 [start.spring.io](http://start.spring.io) 때매 자동으로 만들어준다.
    - dependencies

        ```bash
        repositories {
        	mavenCentral()
        }

        dependencies {
        	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
        	implementation 'org.springframework.boot:spring-boot-starter-web'
        	testImplementation 'org.springframework.boot:spring-boot-starter-test'
        }
        ```

        - dependencies에 명시된 의존성은 위에 나온 레포에서 다운받게 된다. 여기서는 mavenCentral()
    - .gitignore도 자동으로 만들어준다 ㅎㅎ
- 만약 클래스 빌드하고 실행하는게 너무 느리다면
    - 설정 → Build, Execution, Deployment → Build Tools → Gradle
      - Build and run using
      - Run tests using
    - 둘다 IntelliJ IDEA로 바꿔준다.
    - 이러면 Gradle을 거치지 않고 바로 자바로 실행해서 빠르다고 한다.
### 라이브러리
- Gradle은 의존 관계가 있는 라이브러리들을 모두 가져온다.  
- 위에서 나는 spring web, thymeleaf만 추가를 했지만 external libraries를 확인해보면 매우 많은 라이브러리들이 추가로 다운되어 있다.   
- 대표적인 로깅 라이브러리 (system.out.println은 이제 지양하자..)
    - logback
    - slf4j
- 대표적인 테스트 라이브러리
    - junit
    - mockito
    - assertj
    - sprint-test



# Appendix

- SpringBootApplication이 톰캣을 내장하고 있어서 실행하면 톰캣 웹서버가 실행된다.
    - 즉, 스프링 부트가 톰캣 웹서버를 내장하고 있다는 사실. 이는 뒤에서도 계속 나온다.
- 위에서 만든 프로젝트의 메인 클래스를 실행하면, 웹서버가 하나 구동된다.
    - [`localhost:8080`](http://localhost:8080) 으로  접속하면 일단은 에러가 뜨는데, 그게 제대로 된거다!
