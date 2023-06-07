# (추가) 별칭(alias) 관련 정리

queryDsl 로 join 을 할 때, alias 를 걸어줘도 되고 안걸어줘도 무관하다.

```java
    /**
     * Create a left join with the given target.
     * Use fetchJoin() to add the fetchJoin parameter to this join.
     *
     * @param <P>
     * @param target target
     * @return the current object
     */
    <P> JPQLQuery<T> leftJoin(CollectionExpression<?,P> target);

    /**
     * Create a left join with the given target and alias.
     *
     * @param <P>
     * @param target target
     * @param alias alias
     * @return the current object
     */
    <P> JPQLQuery<T> leftJoin(CollectionExpression<?,P> target, Path<P> alias);
```

첫번째 파라미터가 target 으로 이름이 지어져 있는데, 말 그대로 from 에 걸어준 테이블과 해당 target 을 join 한다는 것이다.

만약 target 으로 정한 테이블의 컬럼과 추가적인 join 설정을 해야 할 경우 아래와 같이 alias 를 지정해주고, 지정해준 alias 의 path 를 이용해서 join 을 걸어주면 된다. 아래는 예시 코드이다.

```java
    public Meal findMealByMealId(Long mealId) {
        QMealFeedback mealFeedbackAlias = new QMealFeedback("mealFeedback");
        return from(meal)
                .distinct()
                .innerJoin(meal.user).fetchJoin()
                .leftJoin(meal.mealFeedbacks, mealFeedbackAlias).fetchJoin()
                .leftJoin(mealFeedbackAlias.user).fetchJoin()
                .where(meal.id.eq(mealId))
                .fetchOne();
    }
```

```sql
    select
        distinct m1_0.id,
        m1_0.created_at,
        m1_0.image_url,
        m1_0.intake_time,
        m2_0.meal_id,
        m2_0.id,
        m2_0.created_at,
        m2_0.feedback_contents,
        m2_0.feedback_type,
        m2_0.updated_at,
        u2_0.id,
        u2_0.created_at,
        u2_0.is_social_login_user,
        u2_0.phone_number,
        u2_0.updated_at,
        u2_0.active_score,
        u2_0.meal_feedback_total_count,
        u2_0.meal_total_count,
        u2_0.grade_expired_at,
        u2_0.user_grade,
        u2_0.description,
        u2_0.gender_type,
        u2_0.image_url,
        u2_0.nick_name,
        u2_0.user_resource_id,
        m1_0.memo,
        m1_0.updated_at,
        u1_0.id,
        u1_0.created_at,
        u1_0.is_social_login_user,
        u1_0.phone_number,
        u1_0.updated_at,
        u1_0.active_score,
        u1_0.meal_feedback_total_count,
        u1_0.meal_total_count,
        u1_0.grade_expired_at,
        u1_0.user_grade,
        u1_0.description,
        u1_0.gender_type,
        u1_0.image_url,
        u1_0.nick_name,
        u1_0.user_resource_id 
    from
        meal m1_0 
    join
        user u1_0 
            on u1_0.id=m1_0.user_id 
    left join
        meal_feedback m2_0 
            on m1_0.id=m2_0.meal_id 
    left join
        user u2_0 
            on u2_0.id=m2_0.user_id 
    where
        m1_0.id=?
```
