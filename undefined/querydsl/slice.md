# (+) Slice 쿼리

목록을 호출 할 경우 Paging 으로 많이 처리하게 된다. 하지만 요즘은 많이 사용되는 무한스크롤 형식을 취할 경우 프론트에서 페이지 사이즈와 해당 페이지 사이즈를 기준으로한 페이지 번호 등을 제공할 필요가 없다.

즉, 프론트에서 요청하는 페이지 사이즈, 페이지 번호에 맞는 데이터와 다음 데이터가 존재하는지 여부만 응답해주면 된다. 이 의미는 Paging 처리시 실행되었던 카운트 쿼리가 실행될 필요가 없다는 뜻이다. 페이징 처리시 2번 실행 되어야했던 쿼리의 횟수가 1번으로 줄어들게 된다.

스프링 부트에서는 이를 Slice 로 제공해주고 있는데 이를 querydsl 로 구현하면 아래와 같이 처리할 수 있다.

```java
public Slice<Meal> getSliceOfImageMeals(Long userId, Pageable pageable) {
    final List<Meal> meals = from(meal)
            .innerJoin(meal.user).fetchJoin()
            .where(meal.imageUrl.isNotEmpty().and(meal.user.id.eq(userId)))
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize() + 1)
            .orderBy(meal.createdAt.desc())
            .fetch();

    final boolean hasNext = meals.size() > pageable.getPageSize();
    if (hasNext) {
        final int lastElementIndex = meals.size() - 1;
        meals.remove(lastElementIndex);
    }

    return new SliceImpl<>(meals, pageable, hasNext);
}
```

카운트 쿼리가 일단 없는 것이 보이고, limit 을 요청 받은 것 보다 1개 더 많이 설정해서 데이터를 가져온 뒤 실제로 그만큼 가져왔는지 판단하고 hasNext 를 설정해준뒤 응답시에는 요청한 사이즈 만큼만 돌려주는 방식이다.
