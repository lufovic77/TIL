# 스프링 데이터 JPA
스프링 데이터 JPA는 JPA를 좀 더 편리하게 구현하게 해주는 기술이다. 

즉, 어찌되었든 JPA 공부가 선행되어야만 한다. 

## 특징

- 클래스를 구현할 필요도 없다. 인터페이스로 모든게 끝!
- JPA에서 기본적인 CRUD(Create, Read, Update, Delete) 기능들을 미리 만들어두고 제공한다.
- 진짜 충격적인 거는 대강 이름 규칙을 비슷하게 만들어서 선언만 해둬도 알아서 구현을 해준다.

## 인터페이스로 구현

새로운 레포를 만들건데, 인터페이스로 선언해서 만들거다. 

```java
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {
    Optional<Member> findByName(String name);
}
```

이러면 끝이다. 

- 인터페이스는 두개 이상의 인터페이스 상속이 가능하다.
- `JpaRepository`로 구현체를 받으면 알아서 스프링 빈에 등록해준다.
- findByName이라고 이름 비슷하게 만들어서 선언해두면 알아서 구현된다 ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ
    - 다만 이거는 JPA에서 정의한 규칙을 따라서 이름을 지어야하기는 함!
    - 해당 내용은 [여기](https://docs.spring.io/spring-data/jpa/docs/1.3.0.RELEASE/reference/html/jpa.repositories.html)를 참조해보자
    ![스프링데이터JPA](https://github.com/lufovic77/TIL/blob/main/spring/inflearn/images/springDataJPA.png)

- JpaRepository를 보면 기본적인 것들은 모두 구현이 되어 있음을 확인할 수 있다.
