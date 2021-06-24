# 스프링의 객체 지향 관점의 장점
우리가 이 어플리케이션 설계할 때 `MemberRepository`를 인터페이스로 두고 구현한 이유가 있다. 

틀만 만들어두고, 어떤 DB로 구현할지 혹은 메모리로 구현할지 확실하지 않아서 추후 구현하는 클래스만 따로 만들어서 붙이려고 한거다!

현재까지는 `MemoryMemberRepository`를 이용해 메모리에 값을 저장하는 방식으로 구현했지만, 이번에는 Jdbc를 이용해서 H2 데이터베이스에 연결하는 방식으로 구현한다. 

## 스프링의 엄청난 장점

그럼 기존의 메모리 저장 방식을 버리고 Jdbc를 이용해서 새롭게 구현하면, 기존의 코드들은 어떻게 될까?

**손 댈 필요 없이 그대로 재활용 할 수 있다!**

객체 지향의 엄청난 장점이 뭘까?

- 코드의 다형성 증가
- 즉, 기존의 코드를 조립만 새로하면 재활용 할 수 있다.

우리는 스프링 컨테이너와 스프링 Bean을 이용해서 객체를 관리하고, `SpringConfig` 파일을 이용해서 클래스 간의 의존성을 관리했다. 

다른 코드 수정은 필요 없이 기존의 config 파일을 아래와 같이 바꾸기만 하면 된다. 

```java
@Configuration
public class SpringConfig {

    private final DataSource dataSource;
    public SpringConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {

        //return new MemoryMemberRepository();
        return new JdbcMemberRepository(dataSource);
    }
}
```

- 마지막을 보면 기존의 `return new MemoryMemberRepository();`를 `return new JdbcMemberRepository(dataSource);` 로 바꾸기만 하면 된다.
- 첫 부분의 `DataSource`는 DB 연결을 위해 필요한 부분.

이것이 바로 스프링 컨테이너를 이용한 DI(의존성 주입)의 최대 강점이다. 

# 스프링 통합테스트

지금까지의 테스트 코드 작성은 유닛 테스트였다. 

즉, 기능별로 쪼개서 각각 테스트를 진행하는 거였다면 이번에는 실제로 스프링 컨테이너를 올려서 테스트를 진행하는 통합 테스트이다. 

## 통합 테스트 팁

### Annotation

테스트 클래스 위에 다음 두가지의 Annotation을 달게 된다. 

- @SpringBootTest
    - 테스트를 진행할 때 실제로 스프링 컨테이너를 올려 진행한다는 뜻.
    - 그래서 테스트임에도 아래 그림을 볼 수 있다.

    ```java
    .   ____          _            __ _ _
     /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
    ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
     \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
      '  |____| .__|_| |_|_| |_\__, | / / / /
     =========|_|==============|___/=/_/_/_/
     :: Spring Boot ::                (v2.5.1)
    ```

- @Transactional
    - 기존의 유닛 테스트에서 보았던 @AfterEach annotation을 대체한다.
    - 한번에 테스트를 진행할 때 각 테스트가 끝나면 해당 테스트에서 날린 쿼리 (트랜잭션)들을 모두 **롤백**한다.

### AutoWired

앞의 DI 설명 파트에서 우리는 의존성 주입을 '생성자 주입'으로 구현해야 함을 배웠다. 

그러나 테스트 코드는 그냥 구현하기 가장 편하게 하면 되므로, '필드 주입'으로도 충분하다. 

```java
@Autowired MemberService memberService;
@Autowired MemberRepository memberRepository;
```

## 테스트

결국 다음과 같이 정리할 수 있다. 

- 단위 테스트: 스프링 컨테이너를 안올리고 기능별로 단위별로 쪼개서 테스트
- 통합 테스트: 스프링 컨테이너를 진짜 올려서 테스트 진행

두개 중에 단위 테스트가 보통 더 좋은 테스트!
