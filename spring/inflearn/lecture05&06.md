# 스프링 웹 개발의 세가지 방법

스프링으로 웹 개발을 한다고 할 때, 보통 다음 세가지의 방법이 존재한다. 

- 정적 컨텐츠
    - 파일 그대로를 브라우저 상에서 내려 보여주는거다.
    - 딱히 별다른 프로그래밍이 존재하지 않는다.
    - 우리가 [lecture03](./lecture03.md)에서 보았던 view가 그 예시.
- MVC와 템플릿 엔진
    - MVC (Model, View, Controller)
    - 비유를 하자면 PHP같은 언어가 들어가서 서버와 통신하는 템플릿 엔진 같은거.
    - HTML을 변형 시켜서 보여줄 수 있다. 값을 넣을 수도 있고. 주로 이걸 사용.
- API
    - 다음 내용 정리에서 자세하게 보겠다.

## 정적 컨텐츠

그대로의 html 파일을 브라우저 화면 상에 제공하는 방식. 

작동원리는 아래 그림과 같다. 
![정적 컨텐트](https://github.com/lufovic77/TIL/blob/main/spring/inflearn/images/static.png)
- 역시나 톰캣 웹 서버를 내장하고 있다.
- 주소창에 입력이 들어오면
- controller를 우선 찾아본다 (컨트롤러의 우선순위가 높다)
- 없다면 static 디렉토리를 뒤져서 일치하는 이름의 파일이 있는지 찾아본다.
- 있다면 해당 정적 페이지를 반환한다.

## MVC

MVC의 각 세 대문자는 아래를 상징한다. 

- Model
- View: 화면에 페이지를 띄우는 일에만 집중한다.
- Controller: 비즈니스 로직을 담거나 서버 뒷단의 일들을 한다.

과거에는 View와 Controller가 붙어 있어 View 내에서 작동을 담당하기도 했으나 이제 분리하는 것이 대세. 

### Controller 예시

```bash
//츨처: 강의 교안
@Controller
public class HelloController{
	@GetMapping("hello-mvc")
	    public String helloMvc(@RequestParam(value = "name", required = false) String name, Model model){
	        model.addAttribute("name", name);
	        return "hello-template";
	    }
}
```

사실 코드는 이전 강의 내용 요약에서 본 것과 거의 비슷하다. 

- `hello-mvc`라는 입력이 들어오면 `helloMvc` 메소드를 실행시키며, "name"이라는 입력을 기다린다.
    - `@RequestParam` 같이 annotation을 받는 파라미터 안에다 넣을 수도 있다.
    - 이때, `required = false`를 따로 명시 안하면 인자를 반드시 넘겨야 함
    - 따로 false로 명시 하지 않고 그냥 `localhost:8080/hello-mvc`   접속하면 워닝 뜬다.

    ```bash
    Required request parameter 'name' for method parameter type String is not present]
    ```

    - `localhost:8080/hello-mvc?name=spring!` 같이 인자 넘겨주면서 실행하면 문제없이 돌아감!
        - ? 표시는 HTTP GET방식에서 사용하는 방식. 자세한 내용은 [Appendix](#Appendix)를 보자.

### 작동 원리

![MVC](https://github.com/lufovic77/TIL/blob/main/spring/inflearn/images/mvc.png)
# Appendix

## HTTP의 GET 통신

GET 통신으로 서버에 요청을 보낼 때, 다음과 같이 보낼 수 있다. 

`www.developer.com/index.html?name=kyungmin&id=lufovic77`

- ? 물음표 표시는 URL이 종료됨을 의미한다.
- 서버에 클라이언트의 정보를 보낼 때 사용된다.
- ? 뒤의 매개변수 전달은 key, value 쌍으로 전달된다.
- HTTP 패킷의 헤더에 넣어져서 전달된다.
- 두개 이상의 변수를 전달할 때는 `&`로 구분한다.
