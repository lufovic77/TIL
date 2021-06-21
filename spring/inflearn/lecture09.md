# 테스트 코드 만들기와 약간의 DI 
## 테스트 코드 메소드 이름

### 클래스 이름 따르기

- 관습적으로 테스트 할 클래스 이름 뒤에 Test만 추가한다.
- `MemberService` → `MemberServiceTest`

### 한글 이름

테스트 코드는 실제 코드에 포함이 되지 않는다. 

그래서 보통 현업에서는 알아보기 쉽게 과감하게 함수 이름을 한글로 작성하기도 한다. 

```java
void 회원가입(){
}
```

## 테스트 코드 작성할 때 추천하는 구조

개발자 자체적으로 테스트 메소드를 아래 세가지 step으로 나누어 구현하는 것을 추천한다. 

- given
    - 테스트 할 변수들 준비하기.
- when
    - 이런 경우를 검증하는거구나.
- then
    - 실제로 검증하면 된다.

예를 들어 회원의 정보를 저장소에 저장하는 `save()`메소드를 테스트 해본다고 하자.

```java
@Test
public void save(){
		//given
    Member member = new Member();
    member.setName("Spring");
		//when
    repository.save(member);
		//then
    Member result = repository.findById(member.getId()).get();
    assertThat(result).isEqualTo(member);
}
```

## Annotation

### @Test

위에서 본 `save()`예시처럼 테스트 해보려는 메소드 위에 붙인다. 

### @AfterEach

각 테스트의 종료 후 실행되는 메소드 위에 달린다. 

예를 들어 

```java
@AfterEach
public void afterEach() {
    repository.clearStore();
}
```

이와 같이 테스트가 끝날때마다 저장소를 비우게 하는 메소드를 호출할 수도 있다. 

이 이유는 아래 [주의할 점](#주의할 점)에서 확인할 수 있다. 

### @BeforeEach

각 테스트 실행 전에 호출되는 메소드. 

보통 테스트 간의 영향이 없도록 객체를 새롭게 만들고, 의존 관계도 맺는 역할을 한다. 

## 주의할 점

테스트를 각각 실행 할수도 있지만, 테스트 메소드를 감싸고 있는 클래스를 한번에 빌드 및 실행 하면 여러개의 테스트가 한꺼번에 실행된다.

**이때, 순서는 보장되지 않는다!!!**

따라서 직전 테스트의 특정 operation이 다음 테스트에게 영향을 줄 수가 있다. 

테스트는 순서에 의존성이 있으면 안되며, 각각 독립적으로 실행되어야 한다. 

그래서 보통 위에서 소개한 `@AfterEach`를 이용해 직전 테스트의 결과를 지운다. 

## DI (의존성 주입 - Dependecy Injection)

스프링에서 정말 중요한 DI의 개념이 여기서 살짝 나온다. 

`@BeforeEach`가 달린 메소드를 잠깐 보면, 

```java
@BeforeEach
public void beforeEach(){
    memberRepository = new MemoryMemberRepository();
    memberService = new MemberService(memberRepository);
}
```

마지막에

`memberService = new MemberService(memberRepository);` 라는 이상한 코드가 보인다. 

`MemberService`클래스의 생성자에 `MemoryMemberRepository`클래스의 인스턴스를 넘긴다. 

`MemberService`클래스의 생성자 또한 `new`를 이용한 객체 생성이 아닌, 외부에서 주입하도록 바꾼다.

```java
public class MemberService {
		private final MemberRepository memberRepository;
		
		public MemberService(MemberRepository memberRepo){
		    this.memberRepository = memberRepo; //외부에서 주입
		}
}
```

이렇게 하는 이유는 

- 여러 번 `new`를 통해 `MemberService`클래스에서 저장소를 만들면 여러 오퍼레이션에서 같은 저장소에 접근하지 않는 이슈가 발생할 수도 있다.
- 하지만 외부에서 인스턴스를 직접 주입하게 하면 같은 레포를 사용함을 보장할 수 있다.

**이런식으로 
클래스 자체적으로 new를 통해 의존성이 있는 클래스를 객체화 하는게 아니라, 
외부에서 주입한 인스턴스를 가지고 작업하는 것을 의존성 주입(DI)라고 한다.**
