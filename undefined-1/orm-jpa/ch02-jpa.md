# CH02 JPA 시작

2장은 책의 목차와 내용이 생각보다 부실하고 오히려 인프런 강의가 정리가 잘 되어있어서, 강의 중심으로 정리한다.



## JPA에서 알아서 처리해주는 트랜잭션을 코드로 구현해보기 <a href="#entitymanager" id="entitymanager"></a>

편리한 기능을 제공해준다고 해서 결과만 인식하고 사용하다보면 원리를 깊게 이해하기도 어렵고 문제가 발생 했을때 원인을 찾을 내공이 부족하게 된다.

@Transactional 을 이용해서 한 단위의 트랜잭션으로 묶어서 처리를 하곤 하는데 이렇게 어노테이션으로 트랜잭션 단위를 정하는 행위가 실제 내부적으로 어떻게 처리가 되는 것인지에 대해 알 수 있도록 raw 한 수준에서 코드로 작성해보는 기회가 되었다.

```java
EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("persistenceUnitName");
EntityManager entityManager = entityManagerFactory.createEntityManager();
EntityTransaction transaction = entityManager.getTransaction();

try {
    transaction.begin();
    
    // business logic
    
    transaction.commit();
} catch (Exception exception) {
    transaction.rollback();
} finally {
    entityManager.close();
}
```

핵심은 트랜잭션의 시작과 커밋이 try 내에 존재한다는 것이다. 예외 발생시 catch 에서 롤백이 된다. 어노테이션에서 어떤 예외일때 롤백할지도 정할 수 있는데 그런 경우에는 여기서 catch 가 조금 더 세분화가 될 것이다.

나중에 백기선님 강의나 김영한님 강의에서도 계속 다루고 있는데, EntityManager 는 스레드에 따라 독립적으로 할당된다. (이 부분은 아래에 다시 정리)



## JPA 구성요소간 관계도

<figure><img src="../../.gitbook/assets/image (10) (1) (2) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

외울 것도 없고 흐름에 맞춰서 생각해보면 당연한 내용들이다. Persistence Unit 은 아래와 같이 Database 접속 정보를 가지고 있는 configuration 이라고 보면 된다.

```xml
    <persistence-unit name="hello">
        <properties>

            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver" />
            <property name="javax.persistence.jdbc.user" value="berry"/>
            <property name="javax.persistence.jdbc.password" value="straw"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost/jpa-practice-schema"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>

        </properties>

    </persistence-unit>
```

위 구조에서 entityManagerFactory의 경우 Bean으로 자동 생성된다. Runner 에서 아래와 같이 직접 확인해본다.

```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        String[] beans = applicationContext.getBeanDefinitionNames();

        for (String bean : beans) {
            System.out.println("bean : " + bean);
        }

    }
```

```bash
...(중략)...
bean : entityManagerFactoryBuilder
bean : entityManagerFactory
...(중략)...
```

EntityManager 는 @PersistenceContext 를 통해서 아래와 같이 entityManagerFactory 로 생성해서 주입받아 사용한다. 하지만 실무에서는 굳이 직접 EntityManager 를 사용할 일은 적어도 나의 경험 선에서는 없었다.

```java
    @PersistenceContext
    private EntityManager entityManager;
```



책에서도 강조하듯 EntityManager는 데이터 베이스 커넥션과 밀접하게 작동하므로 스레드간에 공유하거나 재사용을 하면 안된다. 아래와 같이 EntityManager 를 변수명을 달리 하여 두 개를 생성해봐도 결국 주소값을 비교해보면 동일한 EntityManager 임을 확인할 수 있다.

```java
@Component
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    private EntityManager entityManager1;

    @PersistenceContext
    private EntityManager entityManager2;


    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(">>>>>");
        System.out.println(entityManager1);
        System.out.println(entityManager2);
        System.out.println("<<<<<");
    }

}
```

```bash
>>>>>
Shared EntityManager proxy for target factory [org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@103478b8]
Shared EntityManager proxy for target factory [org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@103478b8]
<<<<<
```



결국 entityManagerFactory 만 빈으로 주입받아고 여기서 스레드마다 별도로 entityManager 를 할당 받고 이 entityManager 가 위에서 원형 코드를 본 것처럼 트랜잭션을 관리하기 때문에 1개의 트랜잭션을 만들고 이것을 try 내에서 시작 후 커밋까지 한다. catch 에서 롤백이 발생하며 결과가 어떻게 되든 finally 에서 리소스를 반환한다.
