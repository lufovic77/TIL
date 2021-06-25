# 스프링 Jdbc Template
지금부터 코드를 어떻게 단순화 하고 효율적으로 작성할지에 대한 내용들이 이어진다. 

> 순수 Jdbc 코드 작성 → 스프링의 Jdbc Template → JPA → 스프링 데이터 JPA

왼쪽에서 오른쪽으로 갈수록 훨씬 간단하게 작성이 가능하다.

순수 Jdbc 코드 작성은 매우 옛날에 하던 방법이므로 생략하고 Jdbc Template 부터 살펴보자. 

## 특징

- 순수 Jdbc에 비해서 반복 코드를 상당 부분 제거해준다.
- 그러나 SQL 문은 여전히 개발자가 직접 작성해야 한다.
- 데이터베이스와 연결을 성립시키는 부분이 매우 단축되었다.
- 생성자가 하나 존재하면 bean에 등록 시키는 AutoWired annotation을 생략해도 된다(사실 이거는 Jdbc Template만의 특징은 아님).

## 비교

> 모든 코드는 강의 교안에서 가져왔습니다.

### 순수 Jdbc

- 데이터베이스 연결

```java
try {
  conn = getConnection();
  pstmt = conn.prepareStatement(sql);
  pstmt.setString(1, name);

  rs = pstmt.executeQuery();
}
catch{}
```

이런식으로 쿼리 날리기 전에 연결을 try catch로 감싸서 성립하고 진행해야 한다. 

연결을 종료하는 부분도 자원 관리를 위해 많은 try catch를 이용해서 잡아줘야 한다. 

- 쿼리

```java
public Optional<Member> findById(Long id) {
    String sql = "select * from member where id = ?";
    Connection conn = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    try {
        conn = getConnection();
        pstmt = conn.prepareStatement(sql);
        pstmt.setLong(1, id);
        rs = pstmt.executeQuery();
        if(rs.next()) {
            Member member = new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));
            return Optional.of(member);
        } else {
            return Optional.empty();
        }
    } catch (Exception e) {
        throw new IllegalStateException(e);
    } finally {
        close(conn, pstmt, rs);
    } 
}
```

### 스프링 Jdbc Template

- 데이터베이스 연결

```java
public JdbcTemplateMemberRepository(DataSource dataSource) {
		jdbcTemplate = new JdbcTemplate(dataSource);
}
```

레포 맨 처음 부분에 `DataSource`를 이용해서 연결을 열어주기만 하면 된다. 

- 쿼리

```java
@Override
public Optional<Member> findById(Long id) {
    List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
    return result.stream().findAny();
}
```

그냥 한눈에 봐도 엄청 짧아졌다.......ㅠㅠ 

## 결론

이처럼 불필요한 부분을 제거해서 코드를 매우 단축시켜 주지만, 

SQL문의 경우는 여전히 직접 작성해줘야 한다. 

다음 번에는 기본적인 SQL문도 알아서 생성해주는 JPA로 넘어가보겠다.
