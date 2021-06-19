# API
API와 MVC의 가장 큰 차이점으로는 

- API에서는 `@ResponseBody`annotation을 사용하면 viewResolver를 사용하지 않고 그대로 내려가서 표현된다.
- 현재는 json 반환이 디폴트로 되어 있다 (XML도 선택할 수 있기는 함)
- 앞서 했던 것처럼 문자열이 아닌 **'객체'**를 반환할 수 있다.

## 동작 원리
![동작원리](https://github.com/lufovic77/TIL/blob/main/spring/inflearn/images/api.png)

- 톰캣을 거친 후 컨트롤러를 실행하는 것 까진 똑같다.
- 이때, ResponseBody란 annotation이
    - 없으면 viewResolver 호출
    - 있다면 HttpMesseageConverter 호출
- 그 후, 객체를 받았다면 JsonConverter를, 단순 문자열을 받았다면 StringConverter를 호출!

# Appendix

## IntelliJ 단축키 꿀팁

- `cmd+shift+enter` 치면 뒤에꺼 인텔리제이가 알아서 자동완성해준다.
    - ex) `public static void hello ()` 까지만 치고 단축키 실행하면 중괄호가 생긴다.
- `ctrl+enter`를 사용하면 자동으로 코드를 작성해준다.
    - ex1) 클래스 상속해서 메소드 오버라이딩 할 때 implements 하고 자동완성하면 메소드 쭉 만들어준다.
    - ex2) 클래스 내에서 private 변수 선언하고 자동완성하면 getter와 setter 만들어준다.
    - 이게 왜 필요하냐면

    ```bash
    static class Hello{
            private String name;

            public String getName() {
                return name;
            }

            public void setName(String name) {
                this.name = name;
            }
        }
    ```

    이게 Java bean 표준 규약 중에 하나다. 

    name 변수가 private 이니까 세팅하고, 값 가져올때 public 메소드가 필요한거임.
