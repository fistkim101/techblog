# CH01 JPA 소개

## SQL 을 직접 다룰 때 발생하는 문제점

SQL 을 직접 다룰 때 발생하는 문제점은 곧 ORM 을 쓰지 않고 개발하는 경우 겪어야할 불편함이라는 말과 같은 의미이다.

우리는 객체 지향 프로그래밍을 한다. 이 말인즉 특정 시점에 객체가 데이터베이스의 데이터로 영속화 되어야 하고, 반대로 데이터베이스에서 꺼내온 데이터가 객체로 매핑 되어야한다는 뜻이다. 이 과정을 ORM 을 쓰지 않고 SQL 을 직접 사용하여 처리할 경우 SQL 에 의존적인 개발을 하게 된다.

SQL 에 의존적인 개발이란 일일이 필요한 SQL 을 직접 다 작성해야하고, 필요한 데이터들을 작성한 SQL 에 매핑도 다 해줘야하며 조회시에도 데이터를 객체로 변환하는 과정을 다 신경써줘야한다는 것이다. 또한 DAO로 계층을 나눠놓았다고 해도 정확한 파악을 위해서는 결국엔 SQL 을 직접 확인해봐야하는 문제도 있다. 비슷한 맥락에서 특정 엔티티에서 관계를 갖고 있는 다른 엔티티를 조회할 경우 SQL 에서 이 객체까지 같이 조회하지 않았다면 null exception 이 날 수 있다.

책에서는 이와 같은 문제들을 아래와 같이 문제를 항목화했다.

* 진정한 의미의 계층 분할이 어렵다.
* 엔티티를 신뢰할 수 없다.
* SQL 에 의존적인 개발을 피하기 어렵다.

## 패러다임의 불일치란 무엇이며 JPA가 이를 어떻게 해결해주나

### 패러다임의 불일치(상속, 연관, 비교)

객체 지향 패러다임과 데이터베이스의 데이터 구조는 차이가 존재한다. 이것이 패러다임의 불일치다. 패러다임 불일치를 항목으로 나누면 아래 세 항목으로 나눌 수 있다.

* 상속
* 연관
* 비교

#### 상속 패러다임 불일치

<figure><img src="../../.gitbook/assets/image (13) (1) (1).png" alt=""><figcaption></figcaption></figure>

왼쪽은 객체 패러다임이고 오른쪽은 데이터베이스 패러다임의 모습이다. 데이터베이스에서는 상속이라는 개념을 수용하고 있지 않다. 따라서 저렇게 테이블을 각각 만들어서 데이터를 보관해야한다.

만약 이를 JPA 를 사용하지 않고 처리할 경우에 Album 하나를 insert 한다면 Item, Album 두 테이블에 대한 insert 문들을 모두 만들어 줘야한다. 조회 역시 대상이 되는 두 테이블에 대한 join 문을 각각 만들어 줘야한다. 이러한 처리들이 모두 패러다임 불일치를 해결하기 위한 비용이다.

#### JPA 가 상속 패러다임 불일치를 해결해주는 방법

JPA 는 간단히 상속하고 있는 대상 객체를 저장만 해주면 된다.&#x20;

위에서 예시로 든 Album 을 저장하는 경우를 보면 SQL 을 직접 사용했을 때 두 객체에 대한 Insert 문을 작성해줘야했던 것과 달리 JPA 를 사용하면 Album 이라는 객체에 대한 저장만 해주면 두 테이블에 알아서 Insert 문이 실행된다.

```java
jpa.persist(album);
```

조회 역시 SQL 을 직접 사용하면 두 테이블에 대한 join 문을 만들어서 실행한뒤 결과를 객체에 매핑 해줬어야 했는데, JPA 를 사용할 경우 조회 로직 코드 한 줄이면 해결 된다.

```java
Album album = jpa.find(Album.class, albumId);
```

#### 연관 패러다임 불일치

객체 지향 패러다임에서는 연관관계란 단지 객체 내에서 참조값을 가지고 있는 것이 전부이다. member 라는 객체가 필드로 team 이라는 필드에 참조값을 지니고 있는 것이다. 그래서 객체 지향 패러다임에서는 연관된 객체를 조회하고 싶다면 반드시 그 객체에 대한 참조값을 해당 객체에서 가지고 있어야 한다.

반면에 데이터베이스에서는 외래키 값을 이용해서 어느 테이블이든 join 을 통해 조회가 가능하다. member 에서 team\_id 를 가지고 있으면 member join team, team join member 어떤 식으로든 조회가 가능하다. 객체에서는 team 이 member(1:N) 들을 조회하는 필드를 갖고있지 않으면 조회 자체가 불가능하다.

JPA를 사용하지 않으면 member 에서 team\_id 에 해당하는 필드값을 갖게 하고 이를 이용해서 join 문으로 조회를 해와야하는데 member 에서 바로 team 을 조회하지 못하고, member 의 모델링 역시 SQL 에 의존적인 형태로 객체 지향의 특징과 멀어지게 된다.

쉽게 말해서 ORM 이 없었다면 객체 모델링에 외래키 값이 노골적으로 들어가는 식으로 데이터베이스에 종속된 모델링이 나오게 되는데 ORM 덕분에 객체 지향 방식대로 객체 모델링에 객체 참조를 하도록 해도 외래키로 컬럼이 관리가 되고, 조회시에도 객체 참조를 하지만 데이터베이스 상에상에서 join이 실행 되는 것이다.

#### JPA 가 연관 패러다임 불일치를 해결해주는 방법

객체 지향 패러다임에 따라 그대로 모델링을 해주고 객체 자체를 저장해주면 JPA 가 알아서 외래키를 변환해서 insert 문을 실행해준다. 예를 들어 member 에 team\_id 를 위한 필드 값을 만들고 여기에 team의 id 값을 세팅한 뒤 insert 문을 날릴 필요 없이 member 에 team 객체 자체를 참조하는 필드값을 만들고 member를 저장하면 JPA가 알아서 team의 외래키 값으로 변환하여 저장해주는 것이다.

조회도 마찬가지로 지연 로딩을 통해서 객체 그래프 탐색이 발생하면 적절한 join 문을 JPA에서 알아서 만들어서 실행시켜서 조회된 team 객체를 엔티티에 매핑해준다.

#### 비교 패러다임 불일치

```java
String memberId = "100";
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);
```

100이라는 동일한 pk 로 조회해온 row 를 각각 member1, member2 에 할당할 경우 두 엔티티는 사실상 동일한 객체이지만 member1 과 member2 는 객체 지향 패러다임에서는 member1 != member2 가 된다. 왜냐하면 다른 주소값을 지니게 되기 때문이다.

#### JPA 가 비교 패러다임 불일치를 해결해주는 방법

```java
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);
```

JPA 에서는 같은 트랜잭션에서 동일한 row 에 대해서 동일성 비교에 성공하도록 보장해준다. 위의 로직이 같은 트랜잭션에서 실행될 경우 member1과 member2의 주소값은 같다.

## JPA란 무엇인가

Java Persistence API 는 자바 진영의 ORM 기술 표준이다. JPA 를 애플리케이션 계층 구조에서 보면 아래와 같다. 뒤에 다루겠지만 JPA 는 단지 자바 진영에서 정한 ORM 기술 표준, 명세일 뿐이고 이를 실제 구현한 구현 프레임워크 중 하나가 하이버네이트이다. 하이버네이트는 JPA 에서 선택한 표준 프레임워크이다.

<figure><img src="../../.gitbook/assets/image (19) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

위에서 살펴본 바와 같이 JPA 가 객체 지향 패러다임과 데이터베이스 패러다임 간의 불일치를 해결해주는 덕분에 JPA 를 사용하면 객체 지향 패러다임에 입각한 개발에 더 집중할 수 있다.

아래는 백기선님 강의에서 정리한 내용인데, 같은 내용이어서 옮겨둔다. 각 계층에 대해서 짚어본다.

### Spring Data JPA

스프링 공식문서에도 잘 나와있다시피 Spring Data JPA 는 JPA based data access layer 를 support 하는 목적으로 만들어진 라이브러리이다.

> Spring Data JPA, part of the larger Spring Data family, makes it easy to easily implement JPA based repositories. This module deals with enhanced support for JPA based data access layers. It makes&#x20;

### JPA

Java Persistence API 는 자바 진영의 ORM 기술 표준이다. 자바 진영에서 정한 ORM 기술 표준, 명세이다.

### hibernate

ORM 프레임워크이다. 달리 표현하자면 JPA 의 구현체라고 할 수 있다. JPA 에서는 hibernate 를 내부적으로 가지고 있고 이를 활용하고 있다. (참고로 그래서 JPA를 거치지 않고 hibernate를 직접 활용하는 것도 가능하다.)

hibernate 공식 홈페이지에도 hibernate를 JPA 의 구현체라고 밝히고 있다.

