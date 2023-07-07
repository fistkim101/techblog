# CH07 HTTP 헤더2 - 캐시와 조건부 요청

캐시 처리와 관련된 헤더를 알아본다. 크게 검증헤더와 조건부 요청 헤더로 나눌 수 있다.

* 검증헤더
  * 캐시 데이터와 서버 데이터가 같은지 검증하는 데이터
  * Last-Modified, ETag
* 조건부 요청 헤더
  * 검증 헤더로 조건에 따른 분기
  * If-Modified-Since : Last-Modified
  * If-None-Match : ETag\


## **If-Modified-Since : Last-Modified 활용한 캐시 처리 절차**

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

### **body가 없다는 것에서 속도 향상 = 네트워크 부소하 감소**

Last-Modified, If-Modified-Since 단점

* 1초 미만(0.x초) 단위로 캐시 조정이 불가능
* 날짜 기반의 로직 사용
* 데이터를 수정해서 날짜가 다르지만, 같은 데이터를 수정해서 데이터 결과가 똑같은 경우 서버에서 별도의 캐시 로직을 관리하고 싶은 경우
  * 예) 스페이스나 주석처럼 크게 영향이 없는 변경에서 캐시를 유지하고 싶은 경우



## **If-None-Match : ETag 활용한 캐시 처리 절차**

* ETag(Entity Tag)
* 캐시용 데이터에 임의의 고유한 버전 이름을 달아둠
  * 예) ETag: “v1.0”, ETag: “a2jiodwjekjl3”
* 데이터가 변경되면 이 이름을 바꾸어서 변경함(Hash를 다시 생성)
  * 예) ETag: “aaaaa” -> ETag: “bbbbb”
* 진짜 단순하게 ETag만 보내서 같으면 유지, 다르면 다시 받기!

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>



아래는 인프라 공방 수강시 강의 자료에 있던 내용이다. Etag 관련 내용이라 옮겨왔다.

**최초 요청**

<figure><img src="https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/4b0db36dbf0b498dac9866457d053030" alt=""><figcaption></figcaption></figure>

**두번째 요청**

\


<figure><img src="https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/31006899bb5b498bac87b9c69d183f99" alt=""><figcaption></figcaption></figure>

## **Cache-Control**

* Cache-Control: max-age => 캐시 유효 시간, 초 단위
* Cache-Control: no-cache => 데이터는 캐시해도 되지만, 항상 원(origin) 서버에 검증하고 사용(max-age=0 과 같다)
* Cache-Control: no-store => 데이터에 민감한 정보가 있으므로 저장하면 안됨 (메모리에서 사용하고 최대한 빨리 삭제)
* Cache-Control: no-cache, no-store, must-revalidate => 캐시 무효화.
  * 캐시 만료후 최초 조회시 원 서버에 검증해야함
  * 원 서버 접근 실패시 반드시 오류가 발생해야함 - 504(Gateway Timeout) => 간혹 원 서버 실패시 프록시 캐시(CDN 서버)로 답을 하는 경우가 있는데 아예 오류가 나야 한다는 의미이다.

## 최적의 Cache-Control 정책 정의

아래 의사 결정 트리는 NEXTSTEP 의 인프라공방 수강시 제공된 강의 자료에 포함되어 있던 것이다. 관련 내용이라서 같이 정리한다.

<figure><img src="https://techcourse-storage.s3.ap-northeast-2.amazonaws.com/03d7194f009e43999c108da9ff94e718" alt=""><figcaption></figcaption></figure>
