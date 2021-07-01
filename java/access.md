# 접근 제어 지시자
변수, 메소드나 클래스 앞에 붙는 키워드 들인 public이나 private 같은 것들을 접근제어 지시자 라고 한다. 

이는 다음과 같이 네가지 경우가 있을 수 있다. 

- public
- protected
- default
- private

한번 하나씩 알아보자.

## public과 private

- public으로 선언이 되면 말 그래도 어디서든 접근할 수 있는 인스턴스 변수나 메소드가 된다.
- private으로 선언을 하는 것은 정보 은닉(information hiding)의 매우 중요한 부분이다.
    - 선언된 클래스의 내부에서만 접근할 수 있는 것이 원칙이다.

```java
class kyungmin{
	private height;
	public age;
	public kyungmin(double h){
		height = h;
	}
}
```

## default

아무런 접근 지시 키워드가 붙지 않는 경우이다. 

public과 성질이 비슷하지만, 같은 패키지에 있는 클래스만 접근할 수 있는 것이 특징이다. 

다만, 상속을 하는 과정에 있어서는 허용되지 않는다. 

## protected

default와 상속에서의 특징을 제외하고는 동일하다. 

- protected 키워드가 붙은 변수는 상속 시 접근이 가능하다.

```java
class A{
	private double weight;
    protected int height;
}
class B extends A{
    public void func() {
        height = 180;
    }    
}
```

물론 `weight` 변수는 private이므로 B클래스에서 접근할 수 없다. 

> 상속을 하면 어쨋든 A의 인스턴스 변수들이 곧 B의 인스턴수 변수들이 되는데 왜 안될까?

: 접근 가능 여부는 인스턴스가 아니라, 클래스 기준으로 판단하는 것이기 때문!

## public 클래스, default 클래스

변수에서의 접근지시자처럼 클래스에 붙는 public과 default도 비슷한 의미를 가진다. 

### default 클래스

아무 접근 지시자도 붙지 않는 클래스를 지칭. 

같은 패키지 내의 클래스에서만 인스턴스를 생성할 수 있는것이 특징이다. 

### public 클래스

public으로 선언하게 되면 어디서나 해당 클래스의 인스턴스를 생성할 수 있다. 

다만, public 클래스에는 다음 두가지 조건이 붙게 된다. 

- 하나의 소스파일에는 하나의 클래스만이 public 선언이 가능하다.
- public 클래스 이름은 소스파일의 이름과 동일해야한다.

### 생성자에서의 혼용

생성자에도 접근지시자가 붙을 수 있다. 

예를 들어 

```java
class AAA{
		public AAA(){
				//
		}
}
```

이런식으로 public한 생성자도 만들 수 있으며, private한 생성자도 만들 수 있고 아무것도 붙지 않은 default 생성자도 만들 수 있다. 

> default 생성자는 다른 경우와 동일하게 동일한 패키지에서만 호출할 수 있다는 특징이 있다.

근데, 위 예시처럼 default 클래스의 생성자를 public하게 선언해버리면 어디서든 호출할 수 있는것처럼 보인다. 

하지만 클래스 자체는 default이므로 동일 패키지내에서만 생성자를 호출할 수 있다. 

범위의 문제가 발생할 수 있어서 이런 혼용은 방지하는게 좋으며, 웬만하면 맞춰서 선언해주자.
