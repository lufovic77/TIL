# JPA (Java Persistence API)
앞서 살펴본 Jdbc Template을 이용하면 불필요한 부분들을 쳐내므로 코드가 매우 짧아졌지만, SQL같은 부분들은 여전히 `.query()` 형태로 개발자가 직접 작성해야 했다. 

JPA를 사용하면, 기본적인 SQL문 까지도 자동으로 만들어주게 된다!

> 모든 코드는 강의 교안에서 가져왔습니다.

## 특징

- 반복 코드 제거라는 Jdbc Template의 장점에 더불어 JPA는 기본적인 SQL문도 자동으로 작성해준다.
- 기존의 SQL 문은 테이블을 대상으로 날리는게 목적이였다면, JPA에서 사용되는 JSQL은 객체를 대상으로 쿼리가 진행된다.
- JPA는 자바의 표준 인터페이스일 뿐이다. 이를 구현하는 것은 각 회사마다 다르며, 우리는 이번에 hibernate를 이용할 것이다.

## Annotation 맵핑

- 도메인을 JPA가 관리한다고 표현해주기 위해 `@Entity`라는 annotation을 Member 클래스에 맵핑해준다.
- 테이블에 대한 정보도 맵핑할 수 있는데, column이나 PK같은 정보들을 맵핑할 수 있다.

    ```java
    @Entity
    public class Member {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        private String name;
        //...
    }
    ```

    - ID를 PK로 맾핑했으며, 자동으로 값이 증가하게 해주는 IDENTITY를 사용했다.

이런식으로 annotation으로 맵핑해두면 JPA가 알아서 다 해준다 ㅎㅎ

## EntityManager

JPA를 사용하려면 `EntityManager`를 주입받아야 한다. 

```java
public JpaMemberRepository(EntityManager em) {
    this.em = em;
}
```

SpringConfig 파일에서 주입해준다. 

## JPA의 SQL 자동 작성

단적인 예로 우리가 구현한 메소드들중 DB에 `Member` 객체를 저장하는 `save()`메소드를 한번 보자.

```java
@Override
public Member save(Member member) {
    em.persist(member);
    return member;
}
```

아래는 Jdbc Template 사용할 때의 코드이다.

```java
@Override
public Member save(Member member) {
    SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
    jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");

    Map<String, Object> parameters = new HashMap<>();
    parameters.put("name", member.getName());

    Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));
    member.setId(key.longValue());
    return member;
}
```

- 눈에 띄게 짧아졌다;;
- `persist()`를 이용하면 insert 쿼리를 JPA가 알아서 만들어서 DB에 저장한다.

### JSQL

위에서 본 것처럼 `save()`같이 기본적인 메소드는 JPA에서 알아서 작성해주지만, 당연히 우리가 임의로 추가한 기능인 `findAll()`혹은 `findByName()` 같은 메소드 들은 여전히 직접 쿼리를 직접 짜서 날려야 한다. 

이때, 기존의 SQL을 짜서 날리는게 아니라 JSQL이라는 것을 날리게 된다. 

- 테이블 대상이 아니라 객체 대상이다
- `select m from Member m` 이런식으로 작성

```java
public Optional<Member> findByName(String name) {
          List<Member> result = em.createQuery("select m from Member m where
  m.name = :name", Member.class)
                  .setParameter("name", name)
                  .getResultList();
          return result.stream().findAny();
}
```

## 주의 사항

JPA를 이용하게 된다면 모든 데이터 변경은 트랜잭션 내에서 처리가 되어야 한다. 

따라서 `MemberService()`클래스에 `@Transactional` annotation을 맵핑해야 한다.
