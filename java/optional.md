# Optional 클래스
다음을 import하고 사용하자.

`import java.util.Optional;`

> 참조: [https://jdm.kr/blog/234](https://jdm.kr/blog/234)

## 의미

자바에서는 반환 타입으로 `Optional<T>`을 사용할 수 있다. 

이 Optional의 주된 특징은 바로 '**Null 값 반환에 대응**'하는 의미가 있다.

### 예시 - `ofNullable`

전국에 있는 해장국 매장을 저장하는 간단한 `Map`이 있다고 해보자. 

매장의 고유 `Id`를 가지고 매장의 `Store`객체를 받아오는 함수이다. 

```java
Map<Long, Store> map = new HashMap<>();
public Optional<Store> findById(Long id) {
    return Optional.ofNullable(map.get(id));
}
```

물론 찾는 `Id`에 해당하는 매장 객체가 있어서 정상적으로 객체를 반환할 수도 있다. 

하지만 받아온게 NULL일수도 있는데, 이때 `.ofNullable`을 사용하면 발생하는 `NullPointerException` 예외를 막을 수 있다.

반환한 값이

- Null이 아니면
    - `Store` 객체를 가지는 `Optional` 객체 반환
- Null이면
    - 비어있는 `Optional` 객체 반환

## Optional 클래스의 메소드

위에서 보았듯이 `Optional` 객체 안의 객체에 접근할 소요가 발생하며, 

이는 여러가지 메소드를 통해서 이루어질 수 있다. 

### get()

위에서 보았듯이 어찌되었든 반환되는 값은 `Optional` 객체이다.

따라서 `Optional` 안의 객체에 접근을해야하는데, 

`.get()`을 사용하면 안에 있는 객체를 꺼낼 수 있다. 

### 예시

위에서 정의한 `findById(Ling id)` 메소드를 사용한다고 가정했을때, 

```java
public void output(Long id){
		Store store = findById(id).get();
}
```

`findById`의 반환형이 `Optional` 객체이며, 이제 `store` 객체 안에 적합한 값 혹은 `Null` 값이 들어있을 것이다.
