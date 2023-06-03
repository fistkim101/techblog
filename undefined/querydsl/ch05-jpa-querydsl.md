# CH05 실무활용 (스프링 데이터 JPA와 Querydsl)

## 사용자 정의 리포지토리 <a href="#undefined" id="undefined"></a>

스프링 데이터 JPA 에서 이미 다뤘던 내용이다.

![](https://fistkim101.github.io/images/concept\_spring\_data\_querydsl.png)

굳이 Custom 에 넣어서 하나의 레포지토리 아래에서 동작하게 할 필요는 없고 때에 따라 독립적인 레포지토리를 만들어서 거기서 꺼내 쓰는 것도 좋다.

## 스프링 데이터 페이징 활용1 (Querydsl 페이징 연동) <a href="#1-querydsl" id="1-querydsl"></a>

### 전체 카운트를 한번에 조회하는 단순한 방법 <a href="#undefined" id="undefined"></a>

```java
public Page<MemberTeamDto> searchPageSimple (MemberSearchCondition condition,
        Pageable pageable) {
    QueryResults<MemberTeamDto> results = queryFactory
            .select(new QMemberTeamDto(
                    member.id,
                    member.username,
                    member.age,
                    team.id,
                    team.name))
            .from(member)
            .leftJoin(member.team, team)
            .where(usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe()))
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetchResults();
    List<MemberTeamDto> content = results.getResults();
    long total = results.getTotal();
    return new PageImpl<>(content, pageable, total);
}
```

* Querydsl이 제공하는 fetchResults() 를 사용하면 내용과 전체 카운트를 한번에 조회할 수 있다.(실제 쿼리는 2번 호출)
* fetchResult() 는 카운트 쿼리 실행시 필요없는 order by 는 제거한다.

### 데이터 내용과 전체 카운트를 별도로 조회하는 방법 <a href="#undefined" id="undefined"></a>

```java
@Override
public Page<MemberTeamDto> searchPageComplex (MemberSearchCondition condition,
        Pageable pageable){
    List<MemberTeamDto> content = queryFactory
            .select(new QMemberTeamDto(
                    member.id,
                    member.username,
                    member.age,
                    team.id,
                    team.name))
            .from(member)
            .leftJoin(member.team, team)
            .where(usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe()))
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetch();
    
    long total = queryFactory
            .select(member)
            .from(member)
            .leftJoin(member.team, team)
            .where(usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe()))
            .fetchCount();
    return new PageImpl<>(content, pageable, total);
}
```

* 전체 카운트를 조회 하는 방법을 최적화 할 수 있으면 이렇게 분리하면 된다. (예를 들어서 전체 카운트를 조회할 때 조인 쿼리를 줄일 수 있다면 상당한 효과가 있다.)
* 코드를 리펙토링해서 내용 쿼리와 전체 카운트 쿼리를 읽기 좋게 분리하면 좋다.

## 스프링 데이터 페이징 활용2 (CountQuery 최적화) <a href="#2-countquery" id="2-countquery"></a>

### PageableExecutionUtils.getPage()로 최적화 <a href="#pageableexecutionutilsgetpage" id="pageableexecutionutilsgetpage"></a>

```java
  JPAQuery<Member> countQuery = queryFactory
                  .select(member)
                  .from(member)
                  .leftJoin(member.team, team)
                  .where(usernameEq(condition.getUsername()),
                          teamNameEq(condition.getTeamName()),
                          ageGoe(condition.getAgeGoe()),
                          ageLoe(condition.getAgeLoe()));
 // return new PageImpl<>(content, pageable, total);
  return PageableExecutionUtils.getPage(content, pageable, countQuery::fetchCount);
```

* 스프링 데이터 라이브러리가 제공
* count 쿼리가 생략 가능한 경우 생략해서 처리
  * 페이지 시작이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때
  * 마지막 페이지 일 때 (offset + 컨텐츠 사이즈를 더해서 전체 사이즈 구함)
