# 스프링 빈과 의존성 주입
이번에는 어노테이션을 활용해서 객체를 스프링 빈에 등록해 공유하는 식으로 관리하는 법에 대해 정리해본다. 

## 의존성 주입 (Dependecy Injection)

지금까지는 생성자에서 `new`키워드를  이용해서 객체를 만들어 주었다. 

이제부터는 스프링 빈에 등록한 객체를 스프링이 알아서 찾아서 주입을 해준다!

```java
@Controller
public class MemberController {
    private final MemberService memberService;
    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```

- @Controller
    - 스프링 컨테이너가 생성자 호출해서 객체를 만들어서 관리해준다.
- @Autowired
    - 필요한 객체들을 스프링 컨테이너에서 찾아와 주입한다.
    - 생성자가 하나만 존재하는 경우 생략해도 무관하다.

이것처럼 개발자가 직접 객체를 주입하는게 아닌 스프링이 자동적으로 주입하는게 DI

### 오류 발생

어노테이션으로 컨테이너에 등록 안한 상태로 Autowired 하니까 아래 같은 에러가 나옴 

```java
***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of constructor in hello.hellospring.controller.MemberController required a bean of type 'hello.hellospring.service.MemberService' that could not be found.
```

하나씩 따라 들어가면서 @Component 혹은 필요한 어노테이션 맵핑해서 컨테이너에 등록해주면 빌드 잘된다

## 스프링 빈에 등록하는 방법

### 컴포넌트 스캔 vs 직접 구현

스프링 빈을 등록하는 방법은 두가지가 있다

- 컴포넌트 스캔(@Component 활용) 및 자동 의존관계 설정(@Autowired 활용)
    - 기본적으로 스프링 컨테이너에 등록되는 스프링 빈은  하나만 등록해서 공유한다.

        > 즉, 같은 스프링 빈에서 가져온 인스턴스는 모두 같은 인스턴스다.

- 자바 코드로 직접 config 파일 만들기

이때, 

- 컴포넌트 스캔: 변경 소요가 거의 없는 정형화된 코드인 컨트롤러, 서비스, 레포 같은 경우는 이걸 사용해도 된다.
- config 파일: 상황에 따라 수정 소요가 있는 코드같은 경우는 컴포넌트 스캔 사용 시 의존성이 수정되면 어노테이션을 모두 바꿔야 하지만 직접 config 파일을 작성한 경우에는 아래와 같이 변경하면 된다.

```java
@Configuration
public class SpringConfig {

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```

- 이제 인터페이스 대신 DB를 선정해서 저장소로 활용하게 바뀌었다면

```java
@Configuration
public class SpringConfig {

    @Bean
    public MemberRepository memberRepository() {
        return new DBMemberRepository();
    }
}
```

- 이름만 살짝 바꿔주면 기존의 코드를 변경할 필요없이 적용 가능

## DI의 세가지 방법

지금까지 DI를 하는 이유는 어쨌든 의존성이 있는 클래스들에서 여러개의 중복되는 인스턴스를 만드는 것을 방지하고 하나만 만들어서 공유하게 하려는 목적이 있음!

- 필드 주입
    - 변수 앞에다가 annotation을 붙이는 경우

    ```java
    @Controller
    public class MemberController {

        @Autowired private MemberService memberService;

    }
    ```

    - 별로 안 좋음. 되기는 되네.
    - 이거 나중에 보겠지만 테스트 코드 작성할 때는 이렇게 하기도 함 ㅎㅎ 별로 중요하지는 않기 때문에.
- setter 주입
    - setXXX를 setter라고 함. setName 이런것두 setter.
    - `ctrl+enter` 하면 setter 자동으로 만들어준다.

    ```java
    @Controller
    public class MemberController {

        private MemberService memberService;

        @Autowired
        public void setMemberService(MemberService memberService) {
            this.memberService = memberService;
        }
    }
    ```

    - 문제는, setter 함수가 public이라는거.
    - 이러면 아무때나 호출할 수 있기 때문에 서비스 중간에 바뀔 수가 있다는 문제가 발생한다.
- 생성자 주입 (요즘엔 이걸 권장함)
    - 우리가 한 방법

    ```java
    @Controller
    public class MemberController {

        private final MemberService memberService;

        @Autowired
        public MemberController(MemberService memberService) {
            this.memberService = memberService;
        }
    }
    ```

    처음 컨트롤러 클래스 실행될 때만 생성자에서 의존성 주입.
