# CHAPTER 5

# 5.1 데이터 무결성

- 커다란 데이터가 들어오면 어쩔수 없이 데이터가 손상될 가능성도 높아진다.
- 체크섬
    - 손상된 데이터를 검출하는 일반적인 방법
    - 원본과 새롭게 생성된 체크섬을 비교하여 손상 여부 판단
    - 다만, 원상복구하는 방법은 아니고 단순한 에러 검출용
        - 일반적으로 CRC-32라는걸 이용해 모든 크기의 입력에 대해 체크섬을 계산
    - 체크섬 자체가 손상될 가능성도 있지만, 체크섬은 데이터보다 훨씬 작아서 그럴 가능성은 매우 낮긴하다.

## 5.1.1 HDFS의 데이터 무결성

- HDFS는 모든 데이터를 쓸때 내부적으로 체크섬을 계산 → 읽을때 체크섬을 검증
- 기본적으로 바이트 데이터의 크기는 512 바이트, 체크섬은 4바이트. 오버헤드가 크진 않다.
- 체크섬 검증하는 시나리오
    - 데이터 노드 또한 수신한 데이터를 검증할 책임이 있다.
        - 클라이언트로 수신한 데이터
            - 데이터노드 파이프라인의 마지막 DN이 체크섬 검증 수행
        - 복제할 때 다은 DN으로부터 받은 데이터
    - 클라이언트가 DN으로부터 데이터를 읽을때
        - 읽을때 에러가 검출되면 훼손된 블록, 데이터노드의 정보를 NN에 보고하고 exception 발생
        - 네임노드느 새로운 복제본을 만들고 손상된 복제본을 삭제한다.
    - 저장된 모든 블록을 주기적으로 검증
        - 백그라운드 스레드로 돌린다
        - 물리적인 저장 장치에서 발생가능한 bit rot를 방지하기 위함.
            - [https://ko.wikipedia.org/wiki/데이터_열화](https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%97%B4%ED%99%94)
- 블록이 손상 되었을때 HDFS의 대처방법
    - HDFS는 복제본을 가지고 있기 때문에 복제계수를 유지할 수 있다.
    - 손상된 블록 표시 및 네임노드로 보고
    - 새로운 블록에 복제복 복사
    - 손상된 블록은 삭제

## 5.1.2 LocalFileSystem

- 얘는 클라이언트에서 체크섬을 수행한다.
- 특정 파일을 쓴다고 하자
    - 파일과 같은 위치에 파일의 청크별 체크섬을 저장한다
    - 청크의 크기는 기본적으로 512바이트
        - 청크 크기가 파일 내부에 메타데이터로 저장되어서 추후 청크 크기가 변경되어도 문제없다.
    - 읽을때 체크섬을 검증한다.
- 오버헤드가 그리 크지 않아서 납득할만한 수준이다.
    - 그래도 기존 FS가 체크섬 검증을 지원해서 비활성화하고 싶으면 `RawLocalFileSystem`을 사용하면 된다.

# 5.2 압축

- 압축은 두가지 이점이 있는데, 이는 대용량 데이터를 처리할 때 매우 중요
    - 파일 저장 공간을 줄인다.
    - 네트워크 또는 디스크로부터 데이터 전송을 고속화 한다.
- 모든 압축 알고리즘은 공간 - 시간 트레이드 오프가 존재한다
    - 압축과 해제가 빨라질수록 공간이 늘어난다.
        - -1 옵션으로 빠르게
        - -9 옵션으로 공간 최적화
    - gzip이 그 중간
    - bzip2는 효율적으로 압축하지만 좀 더 느리다

## 5.2.1 코덱

- 코덱이란 압축-해제 알고리즘을 구현한것
    - GzipCodec이란 gzip의 알고리즘을 구현

### CompressionCodec

- 출력 → 압축해서 내보내기
    - 데이터를 압축된 형태로 내부 스트림에 쓴다
- 입력 → 읽어드린 데이터를 압축 해제하기

### CompressionCodecFactory로 코덱 유추하기

- 파일 확장명을 보면 코덱을 유추할 수 있다.
    - .gz는 GzipCodec
- 파일 확장명에 맞는 CompressionCodec을 찾아준다 (등록된 코덱 중에서 일치하는 것을 찾아본다)

### 원시 라이브러리

- 성능면에서 볼때는 자바로 구현하는 것보다 원시 라이브러리를 이용하는게 더 좋다.
    - 다만, 모든 포맷은 원시 구현체가 있지만 모두 자바 구현체가 있는건 아니다.
- 기본적으로 하둡은 플랫폼에 맞는 원시 라이브러리를 찾는다.
- 코덱 풀
    - 압축과 해제 작업이 매우 많이 있으면 압축기와 해제기를 재사용 할 수 있는 CodecPool을 사용해보자

## 5.2.2 압축과 입력 스플릿

- 파일의 압축 포맷이 분할을 지원하는지 여부는 중요하다.
- 1GB의 비압축 파일 → 128MB 크기의 블록 8개로 흩어져 저장
    - MR잡의 맵태스크는 8개의 입력 스플릿 생성
- 1GB의 단일 gzip 압축 파일 → 8개의 블록으로 저장
    - 다만, 블록별로 스플릿을 생성할 수 없다.
    - gzip은 DEFLATE를 사용
    - DEFLATE는 데이터를 일련의 압축된 블록으로 저장
        - DEFLATE는 각 블록의 시작점을 구별할수가 없다.
        - 리더가 다음 블록의 시작으로 이동할수가 없다.
        - gzip은 분할을 지원하지 않는다.
    - 그렇기 때문에 gzip을 쓴다면 → 파일 분할 x → 지역성 저하
- 즉, gzip은 분할을 지원하지 않기 때문에 HDFS 단에서는 파일을 나누어서 저장을하기는 하지만, 실질적으로는 맵 태스크 하나가 모든 나누어진 파일들을 읽으러 다녀야한다.
    - 지역성이 매우 저하된다.
- 어떤 압축 포맷을 사용할지는 상황에 따라 모두 다르다.

## 5.2.3 맵리듀스에서 압축 사용하기

### 맵 출력 압축

- 맵 단계에서 임시 출력을 압축하면 이점이 있다.
    - 맵 출력은 디스크에 기록, 네트워크를 타고 리듀서로 전송
    - 전송할 데이터 양을 줄이면 성능이 향상된다.

# 5.3 직렬화

- 직렬화: 네트워크 전송을 위해 구조화된 객체 → 바이트 스트림으로 전환
    - 프로세스간 통신
    - 영속적인 저장소
    - 이 둘과 같은 분산 데이터 처리에서 나타난다.
- 역직렬화: 역으로 바이트 스트림을 구조화된 객체로 역전환
- RPC (Remote Procedure Call)
    - 하둡에서 노드 사이의 프로세스간 통신하는 구현 방식
    - 직렬화
        - 원격노드로 보낼 메세지를 하나의 바이너리 스트림으로 구성
    - 역직렬화
        - 원격 노드에서 받아서 원본 메세지로 재구성
    - 왜 RPC가 좋은가?
        - 간결성
            - 간결한 포맷을 통해 네트워크 대역폭 절약
        - 고속화
            - 가능하면 오버헤드가 작게 빨라야한다
        - 확장성
            - 프로토콜은 계속 변경된다.
            - 통신 방식도 유동적이여야한다.
        - 상호운용성
            - 다양한 언어에서도 운용이 가능해야한다.
- 다만, 직렬화된 프레임워크 vs 영속적인 데이터는 다르다.
    - RPC는 1초 보다 작은 수명
    - 영속 데이터는 계속 저장된다.
    - 그래도 RPC의 네가지 장점은 영속 데이터에도 중요하다.
        - 간결 → 저장 공간을 효율적으로 사용
        - 빠르다 → 큰 데이터를 읽고 쓰는 오버헤드
        - 확장 가능 → 예전 포맷도 읽을 수 있어야한다.
        - 상호운용 → 다양한 언어를 사용
- 하둡에서는 Writable로 자체 직렬화를 한다 (자바)

## 5.3.1 Writable 인터페이스

- DataOutput: 바이너리 스트림으로 상태정보를 쓴다
    - `byte[] bytes = serialize(writable)`
- DataInput: 스트림에서 정보를 읽는다.
- 자바 int의 래퍼에 해당하는 IntWritable로 실습 진행

### WritableComparable과 비교자

- `IntWritable`은 `WritableComparable` 인터페이스를 구현한다.
- 이떄, 비교자도 중요한다.
    - 맵리듀스에서의 타입비교는 중요하다.
    - 키를 서로 비교하는 정렬과정이 있기 때문.
    - 하둡은 자바의 Comparator를 확장한 `RawComparator` 제공
- RawComparator의 인터페이스는 compare 메소드가 있고, WritableCompartor는 두가지 방법으로 비교 가능하게 구현해두었다.
    1. 스트림으로부터 비교할 객체를 deserialize 한 뒤에 객체의 compare 메소드로 비교
        - 기본 방식으로 두개의 IntWritable 객체를 비교하는 것이 예시
    2. 직렬화된 표현으로 비교하기
        1. 객체로 deserialize 안하고 직접 비교할 수 있어서 객체 만드는 오버헤드를 절감할 수 있다. 
        2. 바이트 스트림 두개와 시작위치/길이를 넘겨주어 직접 비교하도록 한다. 

## 5.3.2 Writable 클래스

- 하둡에는 매우 많은 Writable 클래스가 존재한다.

### 자바 기본 자료형을 위한 Writable 래퍼

- char을 제외하고는 모든 자바 기본형에 Writable 래퍼 존재
    - get(), set() 메소드로 값 다룬다
- 특이한 점은 정수(int, long)를 다룰때 길이 설정 포맷을 선택할 수 있다.
    - 고정길이 (IntWritabe, LongWritable)
    - 가변길이 (VIntWriatble, VLongWritable)
        - 값이 충분히 작으면
            - 단일 바이트만 사용
        - 크면
            - 첫번째 바이트로 양수/음수 + 얼마나 많은 바이트를 쓰는지 표시
            - ex) 163 → 1000.1111.1010.0011
                - 1바이트 = 8비트
                - 1000 → 양수
                - 1111 → 이게 얼마나 많은 바이트를 쓰냐인데 ..왜 그런지는 모르겠네
    - 그럼 선택하는 기준은?
        - 고정길이
            - 값들이 균일하게 분포되어있을때
        - 다만, 일반적으로 대부분 불균일하다.
            - 가변길이를 쓰자!
        - 가변길이의 장점
            - VIntWriatble를 VLongWritable로 변환 가능
            - 둘의 인코딩이 동일해서 가능하다.

### 텍스트

- Text 클래스
    - UTF-8을 위한 Writable 구현체
    - 가변길이 인코딩으로 int를 사용해서 최대값이 2GB까지 간다.
    - 상호 운용성도 뛰어나다.
- 인덱스
    - Text 클래스 VS 자바의 String 클래스
    - Text에서의 인덱스
        - 인코딩된 바이트 시퀀스의 위치 관점
        - String에서 하던 것처럼의 위치 관점이 아니다.
    - charAt() 메소드
        - `Text.charAt(2)`
        - 반환값이 char가 아니라 int다
        - 유니코드 코드 포인트를 반환!
- 유니코드
    - 한바이트 이상을 인코딩 하는 예시
    - 유니코드 중에서 U+10400(보충문자) 이라는 애가 대행 쌍이라는 두개의 char로 표현된다.
    - 그래서 Text랑 String을 비교해보면
        - Text
            - 바이트 단위의 연산을 수행해서 준다.
            - length: 인코딩된 바이트 수
            - find: 바이트 오프셋
        - String
            - char 단위로 연산을 수행
            - length: char 코드 단위의 개수
            - charAt: char 코드 단위의 인덱스
- 반복
    - Text는 인덱스로 바이트 오프셋을 사용하잖나 인덱스를 쓰는게 아니라
    - 유니코드 문자를 반복할 때 꽤 복잡하다. 인덱스를 +1 하면 되는게 아니니까
    - 대충 Text → ByteBuffer → Text의 bytesToCodePoint 정적 메소드를 반복해서 호출한다.
- 스트링 변환
    - String으로 변환할수도 있다.

### BytesWritable

- 바이너리 데이터의 배열에 대한 래퍼
    - 데이터의 바이트 길이를 지정 (4바이트 정수 필드)
    - 데이터가 뒤따른다
    - ex) 3,5
        - 4바이트 정수 필드 (00000002) - 길이 2
        - + 03, 05가 뒤따른다.
        - 결과적으로 000000020305
- 가변적이라 set()로 값 변경이 된다.

### NullWritable

- 길이가 0인 직렬화
    - 어떤 바이트도 읽거나 쓸 수 없다.
    - 그럼 왜있을까? → placeholder(위치 표시자)로 사용된다.
- MR에서 필요없는 키, 값 → NullWritable로 선언 (빈 상숫값으로 저장)

### ObjectWritable과 GenericWritable

- 자바의 기본 자료형, String, enum, Writable, null 혹은 이런 애들의 배열을 위한 래퍼
- 어떤 필드가 두개 이상의 자료형을 가질 때 유용

### Writable 컬렉션

- 하둡 패키지에는 6개의 Writable 콜렉션이 존재한다.

## 5.3.3 커스텀 Writable 구현하기

- WritableComparable의 구현체인 TextPair가 나왔고 얘네가 구현해야하는 메소드들에 대한 정보들이 나왔다.
- RawComparator
    - 역시 맵 태스크에서 사용하려면 compareTo() 메소드를 호출할 수 있어야한다.
    - 두 문자열이 주어지기 때문에 첫번째 문자열의 길이를 알고 오프셋을 넘겨주면 역직렬화하지 않고서도 두 문자열을 비교할 수 있다.

## 5.3.4 직렬화 프레임워크

- 대부분의 MR 프로그램이 Writable 키/값 타입을 사용한다.
    - 다만 의무사항은 아니여서 모든 타입이 사용될 수 있다.
- 이를 위해서 하둡은 (역)직렬화 프레임워크 API를 제공한다.
- 자바 객체 직렬화도 있는데, Writable 만큼 효율적이지는 않음
    - 직렬화는 하둡에서 너무 중요한 만큼 객체에 대한 정교한 제어 능력을 얻을 필요가 있다.

### 직렬화 IDL(Interface Description Language)

- 다른 방식으로 이 문제에 접근하는 여러 직렬화 프레임워크가 있다.
- IDL
    - 언어 중립적이라 다양한 언어로 타입 생성이 되고 상호 운용성도 좋다.
- 에이브로
    - 하둡에 저장된 대규모 데이터 처리에 적합한 IDL 기반의 직렬화 프레임워크

# 5.4 파일 기반 데이터 구조

- MR로 데이터 처리를 하려면 어떻게 저장할지에 대한 고민이 필요하다.
    - 하나의 파일에 blob을 몽땅 때려넣을수는 없는 모냥이지

## 5.4.1 SequenceFile

- 로그 파일을 저장한다고 해보자
    - 로그의 한 레코드 → 한행의 텍스트
- SequenceFile
    - binary key/value쌍에 대한 영속적인 데이터 구조 제공
    - 로그에 딱이다!
    - 키: 타임스탬프 (LongWritable)
    - 밸류: 텍스트 (Writable)
- 이때, 키/밸류 타입은 반드시 Writable일 필요는 없다.
    - 직렬화/역직렬화 할 수 있는 대상이라면 충분히 가능하다.

### SequenceFile 읽기

- Reader 생성하고 next를 반복 호출하면서 읽으면 된다.
- 읽어온 객체가 null이 아니라면 getCurrentValue()로 읽으면 된다.
- null이면 끝까지 읽었다는 뜻!
- 동기화 포인트 (sync point)
    - 리더가 자신의 위치를 잃어버렸을때 레코드의 경계를 재동기화하는데 사용될 수 있는 스트림의 한 지점
    - 부하도 매우 미약하다
    - 항상 레코드의 경계에 맞춰진다
- 파일의 위치를 찾는 방법은 두가지가 있다.
    1. seek()
        1. 파일의 지정된 위치에 리더를 위치시킨다 
        2. 인자로 레코드 경계를 넘기면 된다 (정수값)
        3. 근데 경계를 안넘기면 exception 발생한다. 
    2. 동기화 포인트를 이용
        1. sync(position)
        2. position 이후의 다음 동기화 포인트로 이동
        3. 얘는 다행인거는 어느 위치던 실행할 수 있지. 다음 동기화 포인트를 찾아가니까

### SequenceFile 포맷

- 단일 헤더 + 하나 이상의 레코드로 구성된다.
- 레코드의 내부 포맷은 압축 가능 여부에 따라 다르고,
    - 압축할 수 있다면 레코드 압축/블록 압축인지에 따라 또 다르다.
- 비압축 (default)
    - 레코드 → 레코드 길이 (바이트 단위) + 키 길이 + 키 + 값
    - 길이는 4바이트 정수
- 압축
    - 비압축과 거의 동일하다
    - 마지막 키/값 부분이 다른데 ..
        - 압축안한 키 + 압축된 값
        - 으로 이루어진다.
- 블록 압축
    - 다수의 레코드를 한번에 압축
        - 압축 밀도가 더 높고 레코드 간 유사성을 찾을 확률이 더 높아 선호된다.

### 5.4.2 MapFile

- 키를 기준으로 검색을 지원하는 정렬된 SequenceFile (index 포함)
- SequenceFIle이랑 읽기/쓰기가 매우 유사하다
    - 다만, 쓸때의 주의점은 순서를 지켜서 매 항목을 추가해야한다는점

### 5.4.3 기타 파일 포맷과 칼럼 기반 파일 포맷

- SequenceFile, MapFile 말고도 새로운 파일 포맷들도 존재한다.
    - 에이브로
        - SequnceFIle은 매우 자바 중심적
        - 에이브로는 이기종 프로그래밍 언어에도 이식할 수 있다.
        - 그 자체의 스키마에 의해 기술된다.
- 셋다 row 기반 파일 포맷이다.
    - 즉, 하나의 행의 값들은 파일에서 연속된 위치에 있다.
- column 기반 파일 포맷은 (ex. RCFile)?
    - 먼저 row 기준으로 여러 조각으로 분리
    - 각 조각은 column 기준으로 저장
    - 따라서 각 행의 첫번째 칼럼 저장 → 각 행의 두번째 칼럼 저장
    
    ![image](https://user-images.githubusercontent.com/20508932/175976079-b9ecf640-3f8e-4b58-b0de-245c22c97bbb.png)
)
    
- column 기반이 좋은점은?
    - row 기반은 특정 column만 보고 싶어도 모든 행을 메모리로 읽어와야한다.
    - column 기반일때는 오직 필요한 부분만 로드할 수 있다.
    - 따라서 일반적으로 테이블에서 소수의 column에 접근할때 더 유리하다.
- 나쁜점은?
    - 읽거나 쓸 때 더 많은 메모리가 필요하다.
    - 그냥 한 행을 쭉 읽어오는게 아니라 데이터를 각 행으로 분리하는 버퍼가 추가로 필요
    - 쓰는 도중에 flush, sync가 일반적으로 불가능하며 장애시 파일 복구가 불가능해 스트리밍 쓰기에는 적합하지 않다.
        - row 중심인 SequenceFIle이나 에이브로는 동기화 포인트를 이용해서 쓰기 실패시에도 읽을 수 있다.
