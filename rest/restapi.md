# RESTful
추후 기재 
# REST API를 사용하는 이유

REST API를 사용하는 이유와 RESTful함에 대해서는 토이프로젝트나 면접 준비를 하면서 대강 공부 했었다.

그러나 직접 하나의 서비스를 만들면서 REST API를 정의해보는 경험은 이번이 처음이어서

왜 REST API를 사용하고 명세서를 작성해야하는가? 에 대한 의문점이 있었다.

지금까지 느낀바로는

-   프론트엔드와 백엔드를 분리할 수 있다는 것.
-   이는 구현상의 이슈와 원활한 협업에서 큰 장점을 지닌다.

이 가장 큰 이유였다.

## REST API 명세서를 작성하는 이유
지금까지 나의 개발 경험은 개인 프로젝트가 주가 되었으므로 그냥 서버단에 프론트 요소들과 백엔드 요소들을 섞어두어도 내가 다 알기 때문에 문제가 없었다.  

다만 협업을 하는 과정에 있어서는 프론트엔드 개발자들과 백엔드 개발자들이 따로 일하기 때문에  

프론트엔드 개발자들이 서버에 요청을 보내면 받아오는 결과 값(json이 대표적)만 보고 상황을 간단히 유추할 수 있는 시스템이 필요했다.  

이는 추후 스프링을 이용해서 개발을 진행할 때 테스트 코드를 작성하는 데에도 이점을 줄 것 같다는 생각이 들었다 (간단하게 응답 값을 비교하면 되니까?)          

스프링을 이번에 처음 접해보는 터라 컨트롤러와 도메인 간의 의존성 설계를 어떻게 해야하나.. 고민을 했었는데       

REST API 명세서를 작성해보니까 각 도메인에 해당하는 각 HTTP 메소드마다 컨트롤러를 만들어 구현하면 되겠다는 계획이 세워졌다.       

# 스프링 컨트롤러에 적용
만약 본인이 스프링을 이용해서 개발하고 있다면, 컨트롤러를 구현할 때 REST API 명세서를 작성해 놓았다면 큰 도움을 얻을 수 있을 것 같다.        

-   각 api 메소드 호출 마다 컨트롤러를 만들어주어야 한다.
    -   이때 annotation으로 메소드에 따라 @GetMapping @PatchMapping 등 다르게 사용해야 한다.

-   만약 patch ~/api/adGroups/{adGroup_id}/policy 이런식의 api 호출이 있다면, 스프링에서는
    -   @PatchMapping("api/adGroups/{id}/policy")
    -   이런식으로 annotation을 정의하면 id 값을 파라미터로 받을 수 있다.