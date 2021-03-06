# CH2 - 스트리밍 처리 기초
- 용어와 배경지식들에 대해서 먼저 알아보자

## 데이터플로우 그래프

- 말그대로 데이터가 어떻게 흐르는지에 대한 모식도
    - Directed Graph
    - node: 연산자
    - edge: 의존 관계
    - 입력이 없는 노드: 데이터 소스
    - 출력이 없는 노드: 싱크
        - 소스/싱크 최소 하나씩을 가지고 있어야 한다.
- logical dataflow graph
    - 논리적인, 개념적인 모식도이다
    - 이걸 실제로 실행하려면 물리적인 데이터 플로우로 변환해야한다.
- physical dataflow graph
    - 실질적으로 프로그램을 어떻게 실행하는지를 지정
    - 여기서의 노드: 태스크를 의미한다.
    - 예를 들어 두개의 병렬 머신으로 처리한다면 논리 그래프에서는 하나의 연산 노드지만, 여기선 태스크가 병렬적으로 두개가 떠있는 모습일 것.

## 데이터 병렬화와 태스크 병렬화

- 이 그래프는 여러 방식으로 병렬화 할 수 있다.
    - 데이터 병렬화 (data parallelism)
        - ‘동일 연산'을 하는 태스크를 병렬로 실행한다.
        - 같은 태스크를 여러개 병렬로 두는거
        - 대용량 데이터 처리시 유용
    - 태스크 병렬화 (task parallelism)
        - ‘여러 연산'을 수행하는 태스크를 할당
        - 태스크 자체가 여러가지 연산을 하는거

## 데이터 교환 전략

- physical 데이터 플로우에서 레코드를 어떤 태스크로 할당할지 정의
    - Forward
        - 한 태스크의 데이터를 다른 태스크로
        - 같은 물리 장비에 있으면 네트워크 비용 안들겠지
    - Broadcast
        - 모든 레코드를 모든 병렬 태스크로 내보낸다.
    - Key-based
        - 키 기준으로 같은 키 값을 가진 데이터를 같은 태스크로
    - Random
        - 각 태스크의 부하를 모든 태스크로 균등하게 분배

# 병렬 스트림 처리

- 데이터 스트림이란?
    - 무한 이벤트 순서열. 즉 계속해서 이벤트가 들어온다.

## 지연과 처리율

- 성능을 측정하는 기준이 다른데,
    - 배치 애플리케이션
        - 잡의 총 실행 시간이 중요하다.
    - 스트리밍 애플리케이션
        - 사실 전체 실행 시간은 중요하지 않다
        - latency, throughput

### 지연 (latency)

- 이벤트 전체를 처리하는데 얼마나 많은 시간이 걸리는지
    - 이게 이벤트 하나 단위가 아니였네 ㅇㅅㅇ …
    - 이벤트 수신 → 결과를 볼 때까지 걸리는 시간
    - ex) 카페에서 주문 → 커피 한 모금 마실때까지 걸리는 시간
- 보통 밀리초 단위로 측정
    - 평균지연
    - 최대지연
    - 퍼센타일 (percentile) 지연
        - 주로 이걸 쓴다는데
            - 이거 정확한 의미는 95%로 잘랐을 때 10ms안에 들어온다 라는 의미인듯
            - 95%를 처리하는데 10ms가 걸렸다는건 아니구
        - 95번째 퍼센타일 지연값이 10ms → 이벤트의 95% 처리하는데 10ms 걸림
- 스트리밍 app
    - 몇 ms 정도의 지연으로 제공
- 배치 처리
    - 이벤트를 모아서 처리하기 때문에 자연스레 마지막에 도달한 이벤트가 지연을 결정

### 처리율 (throughput)

- 시스템의 처리량을 측정
    - 단위 시간당 얼마나 많은 이벤트를 처리하는지
    - ex) 카페에서 한시간당 얼마나 많은 손님을 받을 수 있는지
    - 지연이 가능한 짧아야 처리율이 올라간다.
- 최대 처리율
    - 최대 처리율로 이벤트를 처리하길 바란다.
    - 다만, 자원을 모두 사용할 정도의 유입량 때문에 시스템에 부하가 걸린다면?
    - 버퍼링을 하다가 한계를 넘어서면 데이터를 잃을 수도 있다 (backpressure)

### 지연과 처리율

- 높은 지연 → 높은 처리율을 기대할 수 없다.
- 낮은 처리율 → 이벤트가 버퍼링 되므로 낮은 지연을 기대할 수 없다.
- 그렇다면 .. 낮은 지연 + 높은 처리율이 가능?
    - 성능 좋은 계산 머신
    - 병렬화

## 데이터 스트림 연산

- 인입, 변환, 출력에서 사용하는 연산들에 대해서 알아보자.
- 상태
    - 상태가 없는 연산
        - 내부 상태가 유지되지 않는다.
        - 독립적이므로 병렬화하기 쉽다.
    - 상태가 있는 연산
        - 최신 이벤트가 내부 상태를 갱신하고 이걸 다시 사용.
        - 따라서 상태가 있는 스트림 처리기에서는 병렬화와 내고장성 유지가 중요하다.

### 데이터 인입과 방출

- 인입 (ingestion)
    - 데이터 소스에서 데이터를 가져와 변환
    - TCP 소켓, 카프카 토픽 등
- 방출 (egress)
    - 소비 형태에 맞춰 변환하여 내보내기 (데이터 싱크)
    - 파일, DB, 메세지 큐 등

### 변환 연산

- 단일 경로 연산
- 이벤트를 하나씩 소비하면서 각각에 변환 연산을 적용 후 내보냄

### 롤링 집계 연산

- 합계, 최소, 최대 같은 집계 연산
- 상태가 있다 (현재 상태와 최신 이벤트를 결합해야한다)
- 결합 법칙, 교환 법칙이 성립해야만 이걸 사용할 수 있음
- 한번에 하나의 이벤트 소비

### 윈도우 연산

- 변환/롤링 연산: 한번에 하나의 이벤트만 처리
- 일정량의 이벤트를 모아 처리할 필요가 있을 수 있다.
    - 최근의 데이터만을 모아서 보고 싶다면?
- 버킷
    - 이벤트의 유한 집합을 지속적으로 생성, 거기서 연산을 수행
    - 그 정책은 시간 (마지막 몇초), 개수(마지막 몇개) 등이 될 수 있다.
- 윈도우의 종류들
    - 텀블링 윈도우 (Tumbling Window)
        - 고정길이(개수 or 시간), 서로 겹치지 않는 bucket
        - 개수 기반
            - 윈도우를 트리거링 (연산에 보내기)전에 얼마나 많은 이벤트가 모여야 하는지에 따름
        - 시간 기반
            - 정해진 시간 간격으로 데이터를 버퍼링
        
      <img width="478" alt="스크린샷 2022-07-20 오전 9 39 30" src="https://user-images.githubusercontent.com/20508932/180367852-c1a867fe-3d66-4578-a131-d16c23df02ef.png">

    - 슬라이딩 윈도우 (Sliding WIndow)
        - 고정 길이, 서로 겹치는 
        - 두 버킷에 동시에 포함되는 이벤트도 있다.
        - 슬라이드
            - 새로운 버킷이 생성될 때까지의 간격
        
      <img width="483" alt="스크린샷 2022-07-20 오전 9 41 43" src="https://user-images.githubusercontent.com/20508932/180367969-101ae7f7-6106-405f-a899-f9242567dd24.png">

        
    - 세션 윈도우 (Session Window)
        - 현실 세계에서는 위 두개처럼 데이터가 착착 모이지가 않는다.
        - 세션
            - 활동하는 기간 동안 발생하는 이벤트들을 하나의 세션이라고 보면 된다.
            - 이 길이를 예측할 수 있는것도 아니고 실제 발생한 데이터를 기반으로 봐야한다.
            - 따라서 하나의 세션이 비활성 기간에 들어가는 그 간격을 session gap이라고 정의하여 세션들을 그룹화 한다.
- 지금까지는 모든 이벤트를 대상으로 동작하는 윈도우 연산을 봤다.
    - 이걸 논리적으로 분리해서 병렬로 처리할 수도 있다. (병렬 윈도우)
    - 각 파티션별로 다른 윈도우 정책 적용 가능
- 윈도우 연산의 문제는 상태를 모아서 처리를 하기 때문에 장애 발생 시에 이걸 안정적으로 복구하는게 중요하다.
    - 이번 장의 뒤에서는 그걸 위주로 볼것 !

# 시간 시멘틱

- 스트리밍에서 ‘시간'의 의미에 대해 보고,
    - 순서가 뒤바뀌어서 들어온 이벤트를 어떻게 처리하는지를 보자.

## 스트리밍 처리에서 1분

- 예를 들면 이해하기 쉬운게
    - 지하철에서 온라인 게임을 하다가 30초 동안 터널속에 들어가서 연결이 끊김.
    - 그 30초 동안의 이벤트들은 핸드폰에 버퍼링 되어 있는데 .. 이걸 어떻게 볼까 ?
- 데이터를 ‘처리'하는 시간을 기준으로 하면 처리 속도나 네트워크 연결 속도가 결과에 영향을 미치게 된다.
- **데이터 자체의 실제 시간**을 고려해야 함!

## 처리 시간 (processing time)

- 스트림 처리 연산자가 실행 중인 장비의 로컬 시간
- 예시를 보면 터널에 들어가도 처리시간은 계속 흐른다.
    - 터널에 나와서 보낸다고 하더라도 그건 인정되지 않지 (다른 윈도우로 들어감)

## 이벤트 시간

- 이벤트가 실제 발생한 시간
    - 이벤트 안의 타임 스탬프를 기반으로 한다.
- 처리 결과를 처리 속도랑 분리해서 생각하는 것
    - 따라서 이벤트가 언제 도착하든, 연산자가 얼마나 빠르든 관계 없이 항상 동일한 결과를 만든다.
    - 과거의 이벤트도 돌려볼 수 있음.

## 워터마크

- 그렇다면 언제 이벤트 시간 윈도우를 끊고 트리거링 (연산자로 보내기)할 수 있을까?
    - 즉, 이벤트가 다 도착했다 !! 라고 확신할 수 있는 근거를 어떻게 볼까 ?
- 워터마크
    - 이벤트가 더 지연되지 않고 도착할 것이라고 확신하는 시점
    - T라는 워터마크 → 여기 이후로는 이벤트를 안받는다!
    - 이 값은 신뢰성 - 지연 그 사이의 균형을 잘 맞춰야함.
- 다만, 워터마크를 결정할 정보가 실제 상황에선 너무 부족함.
    - 그 이벤트가 비행기를 타서 몇시간동안 안오는건지, 지하철 때문에 몇초 안오는건지를 모른다.
    - 따라서 워터마크보다 늦게 도착한 이벤트를 처리하는 방법을 제공한다 (무시 or 보정).

## 처리 시간과 이벤트 시간

- 지금까지 내용을 보니까 이벤트 시간만 있으면 될 것 같은데 처리시간은 왜 있을까?
- 처리 시간도 이벤트의 순서를 고려하지 않는 어플리케이션이라면 매우 유용하다
    - 정확보다 속도가 중요한 경우
    - ex) 정확도가 별로 안중요한 실시간 보고서
- 처리시간도 충분히 스트림을 표현할 수 있고, 실제로 처리 시간의 의미를 원하는 경우도 있다.
    - 이해 안간 부분
    
    > 이거의 예시로 ‘정전을 감지하고자 스트림 애플리케이션이 스트림에서 초당 얼마나 많은 이벤트가 발생하는지 관찰할 수 있다'라고 했는데.. 이거면 이벤트 시간 아닌가 ..?
    > 
- 걀과적으로
    - 처리시간: 짧은 지연, 결과는 결정적이지 않다.
    - 이벤트 시간: 결정적 결과를 보장

# 상태와 일관성 모델

- 이번에는 ‘상태'라는 개념에 대해서 알아보자.
    - 뭐 내부에 임시적으로 계산 값을 들고 있는 것도 상태라고 할 수 있지.
- 원래 기존의 배치 처리잡에서는 하나의 배치잡이 종료되면 그 결과는 영속적인 저장소로 간다.
    - 즉, 잡의 상태가 사라지게 된다.
- 다만, 이제 요즘에는 무한 이벤트에서 실행되는 스트림 처리 잡이잖아.
    - 여기서는 항상 상태가 존재하고 모든 이벤트가 상태에 접근할 수 있게 된다.
- 상태가 있는 경우에는 다음 과제들을 극복해야함
    - 상태 관리
        - 상태 관리를 효율적, 상태를 보호
        - 예를 들어 내부 상태가 무한으로 커지면 안된다
    - 상태 분할
        - 병렬 처리할 때 복잡하다
        - 키로 상태를 분할
        - 파티션별로 독립적으로 상태를 관리
    - 상태 복구
        - 상태를 복구하고 장애 극복 → 지금부터 이걸 볼거임

## 태스크 실패

- 장애 시 연산자의 상태를 잃어버리면 복구가 불가능하다.

### 태스크 실패

- 태스크는 일반적으로 다음과 같은 절차로 이루어진다.
    1. 이벤트를 수신해 로컬 버퍼에 저장
    2. 경우에 따라 내부 상태 갱신
    3. 출력 레코드 생산
- 실패는 이 중 어디에서나 발생할 수 있다.
- 배치 처리잡이야 걍 처음부터 다시 하면 된다. 근데 스트리밍에서는 그리 쉬운일이 아니다.

## 결과 보장

- 자세히 알아보기 전에 확실히 짚고 넘어 가야하는것이
    - ‘결과 보장'의 의미
    - 스트림 처리기 내부 상태의 일관성이다.
    - 즉, 코드가 바라보는 상태의 일관성
    - 이게 싱크돼서 출력될때의 일관성은 보장하지 않는다.
        - 그건 싱크 시스템의 역할

### 최대 한번 (at-most-once)

- 이벤트를 최대 한번만 처리
- 이벤트가 유실되어도 아무것도 하지 않는 낮은 수준의 보장 방법
- 대략적인 결과나 지연시간 최소화가 중요하면 나름 괜찮다.

### 최소 한번 (at-least-once)

- 모든 이벤트를 처리하고 일부는 한 번 이상 처리할 수 있다.
    - 즉, 중복처리도 가능함
- 결과만 중요하면 나름 쓸만함
    - ex) 이벤트의 발생 여부만 알고 싶다면 유용
- 다만, 이걸 하려면 이벤트를 어딘가에 저장하고 재생을 해야한다.

### 정확히 한 번 (exactly-once)

- 딱 한번만 이벤트를 처리
    - 장애가 나도 실패가 없었던 것처럼 보이도록한다.
- 이걸 하려면?
    - 위에서 말한 이벤트 재생은 필수다
    - 내부 상태의 일관성 보장
        - 즉, 어떤 이벤트가 상태에 반영됐는지를 알아햐한다.
- 플링크에서는 lightweight snapshot이란 방식으로 보장한다.

### 단대단 정확히 한번 (end-to-end exactly-once)

- 지금까지는 스트림 처리의 각 컴포넌트에서 발생하는 실패를 보장하는 방법을 봄
- 이거는 소스에서 싱크까지의 파이프라이닝이 완전히 보장되는 것을 의미
    - ex) 가장 쉬운건 멱등 처리로 할 수 있다.

지금까지는 스트리밍 처리의 개념을 봤고, 다음부터는 ‘플링크’에 대해서 세부적으로 배운다.
