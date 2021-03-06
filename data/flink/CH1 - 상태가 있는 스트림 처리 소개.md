# CH1 - 상태가 있는 스트림 처리 소개
# 전통적인 데이터 인프라

- 두 종류의 데이터 처리로 구분할 수 있다.
    - 트랜잭션  처리
    - 분석 처리

## 트랜잭션 처리

- 기존에는 두 계층으로 아키텍쳐가 구성되었다.
    - 애플리케이션
    - DB
- 문제는 여러 어플리케이션이 동일한 DB에 접근하기 떄문에 여러 제약이 있었고,
    - 이런 결합으로 인한 제약을 풀기 위한 노력이 MSA의 등장.
- 각 마이크로 서비스들을 독립적으로 분리하여 REST로 통신하게끔 한다.
    - 각 서비스들은 서로 다른 기술 스택 (언어, 라이브러리, 데이터 저장소 등)들로 이루어질 수 있다.

## 분석 처리

- 데이터들을 저장한 다음에는 분석을 하는것도 매우 중요하다.
- 이때 데이터 저장소에 직접 연결해서 분석 작업을 하기보다는
    - 데이터 웨어하우스 - **분석 질의 전용 데이터 저장소**를 활용
    - 이때, 데이터 웨어하우스로 데이터를 옮기는걸 ETL 처리라고 한다.
- ETL
    - Extract
    - Transformation
    - Load
    - 동기화를 위해 주기적으로 실행되어야 한다.
- 이런 데이터 웨어하우스에 질의하는 두가지 방법
    - 데이터를 주기적으로 보고하는 질의
    - 특정 문의에 응답하거나 빠른 결정이 필요할 떄 사용

# 상태가 있는 스트림 처리

> 용어가 참 애매한데, ‘이벤트 로그 == 카프카' 라고 생각하면 된다.
> 
- 모든 데이터는 이벤트 스트림이 지속적으로 생성한 데이터
    - 상태가 있는 스트림 처리는 이런 ‘무한' 이벤트 스트림 처리에 좋다.
- 아파치 플링크는 상태를 로컬 메모리나 DB에 저장한다.
    - 근데 분산 시스템이기 때문에 백업이 필요하므로
    - 원격 저장소에 상태의 체크포인트를 주기적으로 기록한다.
- 주로 ‘상태가 있는 스트림 처리 app(플링크)’은 이벤트 로그(카프카)에서 이벤트를 가져온다.
    - ex) 플링크는 카프카에서 이벤트를 가져온다
    - 이런 이벤트 로그들은 append-only 특성을 가진다.
    - 여러 번 다른 사용자가 읽는다고 하더라도 동일한 순서가 보장된다.
- 장애가 발생하면
    - 직전 체크포인트로 복구하고
    - 카프카(이벤트 로그)의 읽는 위치를 이전으로 재설정해서 다시 끝까지 읽는다.
- 일반적으로 많이 구현되는 다음 세가지 패턴의 app을 알아보자.
    - 이벤트 주도 app
    - 데이터 파이프라인 app
    - 데이터 분석 app

> 여기부터는 왜 플링크를 쓰는게 좋은지에 대한 얘기를 한다.
> 

## 이벤트 주도 애플리케이션 (Event Driven Application)

- 이벤트 스트림을 입력으로 받아 비즈니스 로직을 처리한다.
- MSA와는 조금 다르게
    - REST 호출 말고 kafka 같은 이벤트 로그를 통해 서로 통신한다.
    - 상태를 외부 저장소에 저장하지 않고 로컬에서 관리한다.
- 이벤트 로그
    - 전송/수신을 분리
    - 비동기, 논블록킹으로 이벤트를 전송한다.
- 여러가지 이점이 있는데,
    - 로컬에 상태를 저장: 훨씬 빠른 성능
    - 이벤트 로그가 입력 소스이므로 장애극복이 된다.
    - 상태를 이전 savepoint로 설정. 업그레이드나 수평 확장이 가능하다.

## 데이터 파이프라인

- 오늘날 상태나 정보를 RDBMS, HDFS 등 여러곳에 저장하게 된다.
    - 이렇게 동일 데이터를 여러 데이터 저장소에 저장하면 성능도 좋고 해서 일반적이다.
    - 다만, 동기화를 해야하는데 ..
- 원래는 주기적으로 ETL 수행.
    - 지연 시간이 좀 길다.
    - 이벤트 로그를 이용해 변경 내용을 분산!
    - 이렇게 보낼 때 데이터를 가져와서 변환하는게 데이터 파이프라인.

## 스트리밍 분석

- ETL을 이용한 분석 작업은 .. 배치 처리 잡이다.
    - 따라서 상당히 긴 지연시간을 발생시키는 원인
- 지속적으로 이벤트 스트림을 가져와서 짧은 지연시간으로 이벤트를 처리

## 스트림 처리의 역사

- 1세대 처리기는 빠르지만, 정확한 결과를 포기했다.
    - 장애 시 유실을 허용
    - 일관성 있는 결과를 제공하지 않음.
- 람다 아키텍처
    - 정확도를 위한 ‘배치 레이어’ - 배치 잡으로 일배치 보고서를 만들 때 사용하는것
    - 빠른 속도를 위한 ‘스피드 레이어' - 실시간 보고서 만들때 사용
    - 서빙 계층: 두개 합쳐가지구 보여주는거
    - 두개를 운용
        - 배치 테이블에 있는 정확한 결과를 바탕으로 스피드 테이블의 부정확한 결과를 버린다.
    - 다만, 단점이 있는데
        - 동일한 기능을 하는 두 벌의 app을 구현해야한다.
        - 여전히 결과는 대략적일 뿐
        - 설치, 유지보수가 어려움
- 2세대 처리기
    - 고수준 API
    - 장애 복구 기능 및 정확히 한번 반영됨을 보장
    - 결과가 여전히 시간과 이벤트 도착 순서에 따라 다르다.
- 3세대 처리기 - 플링크(Kappa 아키텍처)
    - 정확히 한번 장애복구
    - 정확한 결과를 만들어냄
    - 정확도, 빠른 속도 둘다 잡아냄
