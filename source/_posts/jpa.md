---
title: 10. JPA
date: 2018-03-15 00:51:30
tags: spring, java, jpa, hibernate
---
Java Persistence API의 약자입니다.
영속성객체에 대한 java의 접근방법을 정의한 API로, 객체를 통한 Persistence 영역으로 접근방법을 정의하고 있습니다.
객체를 통한 Persistence 영역에 대한 접근을 하기 위해, 사용되는 것이 ORM Framework 들이고, 그중에서 가장 선두를 달리고 있는 것이 Hibernate입니다.

## JPA vs Hibernate

Hibernate와 JPA간의 차이를 한번 알아보도록 하겠습니다.

|방법|Hibernate|JPA|
|---|---|---|
|DB에 대한 접근(DataSource)|SessionFactory|EntityManagerFactory|
|DB의 query executor|Session|EntityManager|
|INSERT 대응 method|save, saveOrUpdate|merge|
|UPDATE 대응 method|update, saveOrUpdate|merge|
|DELETE 대응 method|delete, remove|remove|
|query 적용 방법| ~~Criteria~~, HQL (Hibernate query Language), Native SQL|Criteria, JPQL (java persistence query language), Native SQL|

접근 하는 방식은 거의 1:1로 동일합니다. 다만 query의 적용방법에서 차이를 보이게 되는데요. JPQL이 JPA의 표준이 되면서, 기존 Hibernate에서 주로 사용되고 있던 Criteria/HQL을 거의 사용하지 않습니다. 현 Hibernate 5.0 이상에서는 `Creteria`와 `HQL`은  `deprecated`되었습니다.

먼저 `JPA`를 구현하는 방법에 대해서 알아보도록 하겠습니다. 이는 `JAVA`의 표준기술입니다. 잘 알아둘 필요가 있습니다.
dependency는 다음과 같습니다.

```groovy
compileOnly group: 'org.projectlombok', name: 'lombok', version: '1.16.18'

compile 'com.google.guava:guava:23.0'
compile group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: '2.9.3'
compile group: 'org.mariadb.jdbc', name: 'mariadb-java-client', version: '2.2.0'
compile group: 'org.hibernate', name: 'hibernate-core', version: '5.2.12.Final'
compile group: 'org.hibernate', name: 'hibernate-entitymanager', version: '5.2.12.Final'
compile group: 'org.hibernate.javax.persistence', name: 'hibernate-jpa-2.1-api', version: '1.0.0.Final'

compile group: 'com.zaxxer', name: 'HikariCP', version: '2.7.4'
compile group: 'org.springframework', name: 'spring-context', version: '5.0.2.RELEASE'
compile group: 'org.springframework', name: 'spring-orm', version: '5.0.2.RELEASE'

testCompile 'junit:junit:4.12'
testCompile 'org.assertj:assertj-core:3.8.0'
testCompile group: 'org.springframework', name: 'spring-test', version: '5.0.2.RELEASE'
```

JPA는 표준 API로서, 기존에 `de facto`로 사용되고 있던 `Hibernate`와 거의 완벽하게 동일합니다. `@Configuration`은 다음과 같이 구성됩니다.

```java
@Configuration
@ComponentScan(value = {
    "com.xyzlast.bookstore.repository"
})
@EnableTransactionManagement
public class BookStoreConfig {
    @Bean
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setDriverClassName("org.mariadb.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://127.0.0.1:4306/bookstore");
        dataSource.setUsername("root");
        dataSource.setPassword("qwer12#$");
        dataSource.setMaximumPoolSize(10);
        dataSource.setMinimumIdle(10);
        return dataSource;
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryBean() {
        Properties properties = new Properties();
        properties.setProperty("hibernate.dialect", "org.hibernate.dialect.MariaDB53Dialect");
        properties.setProperty("hibernate.show_sql", "true");
        properties.setProperty("hibernate.id.new_generator_mappings","false");

        LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
        em.setDataSource(dataSource());
        em.setPackagesToScan("com.xyzlast.bookstore.entity");

        JpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        em.setJpaVendorAdapter(vendorAdapter);
        em.setJpaProperties(properties);

        return em;
    }

    @Bean
    public PlatformTransactionManager transactionManager() {
        JpaTransactionManager jpaTransactionManager = new JpaTransactionManager();
        jpaTransactionManager.setEntityManagerFactory(entityManagerFactoryBean().getObject());
        jpaTransactionManager.setDataSource(dataSource());
        return jpaTransactionManager;
    }

    @Bean
    public PersistenceExceptionTranslationPostProcessor exceptionTranslation() {
        return new PersistenceExceptionTranslationPostProcessor();
    }
}
```

기존의 `SessionFactoryBean`과 `HibernateTransactionManager`, `HibernateExceptionTranslationPostProcessor`가 각각 `EntityManagerFactoryBean`, `JpaTransactionManager`, `PersistenceExceptionTranslationPostProcessor`로 변경되게 됩니다.
주의해서 보실 내용은 `HibernateJpaVendorAdapter`입니다. 이는 JPA 표준을 따르는 `Vendor`를 지정해줍니다. Hibernate에서 제공하는 Vendor Adapter입니다.

|Hibernate|JPA|설명|
|---|---|---|
|SessionFactoryBean|EntityManagerFactoryBean|DB 접근방법|
|HibernateTransactionManager|JpaTransactionManager|Transaction 관리자|
|HibernateExceptionTranslationPostProcessor|PersistenceExceptionTranslationPostProcess|DB내에서 발생된 Exception의 Converter|

지금 `JPA`에 적합한 Framework는 2개입니다. `Hibernate`와 `EclipseLink`가 있습니다.

|JpaVendorAdaptor|Hibernate|EclipseLink|
|---|---|---|
||HibernateJpaVendorAdapter|EclipseLinkJpaVendorAdapter|

`BookDao`의 구성은 다음과 같이 됩니다.

```java
@Repository
public class BookDaoImpl implements BookDao {

    @PersistenceContext
    private EntityManager em;

    @Override
    public int countAll() {
        return em.createQuery("SELECT COUNT(*) FROM Book", Long.class).getSingleResult().intValue();
    }

    @Override
    public List<Book> getAll() {
        return em.createQuery("FROM Book", Book.class).getResultList();
    }

    @Override
    public Book getById(int id) {
        return em.find(Book.class, id);
    }

    @Override
    public int add(Book book) {
        em.merge(book);
        return 1;
    }

    @Override
    public int update(Book book) {
        em.merge(book);
        return 1;
    }

    @Override
    public int delete(Book book) {
        em.remove(book);
        return 0;
    }

    @Override
    public void deleteAll() {
        em.createQuery("DELETE FROM Book").executeUpdate();
    }
}
```

Hibernate의 `Session`을 이용하는 코드와 거의 유사합니다. 단 차이가 있다면 생성자로 `SessionFactory`를 전달했던 코드가 `@PersistenceContext`로 바뀌는 차이가 있습니다. 이는 JPA에 정의된 annotation으로 PersistenceContext의 주입을 지정합니다.

## Querydsl

ORM을 통해서, 우리는 RDBMS를 OOP적 사상으로 옮기는 것이 가능해졌습니다. 그렇지만, `FROM Book`과 같은 `JPQL`이라는 문자열식 query에 종속되고 있는 것이 사실입니다. 또한, `Entity`의 변경시에 문자열식 query는 변경되지 않기 때문에, 실행하기 전까지 오류를 찾아낼 수 없습니다. 그리고, 문법이 꽤나 복잡한 것이 사실입니다.

개발상 요구사항은 계속해서 바뀌게 되고, 개발도중에는 Schema가 계속해서 변경되게 되는 것이 사실입니다. 그때마다 `JPQL`에 문제가 있는지를 확인하는 것은 빼먹기쉬운 부분입니다. Querydsl은 `Entity`객체와 연동되는 'Q-class`라는 query 객체를 생성하고, 이를 통해서 query를 만들어내게 됩니다. 이를 통해 Entity의 변경에 따른 query문제를 없앨 수 있으며, Query의 동적인 생성을 원활하게 합니다.

Querydsl에서의 query pattern은 .NET의 linq 사상을 옮겨왔습니다.

```csharp
var queryLondonCustomers = from cust in customers
                           where cust.LastName.startsWith('A') && cust.active
                           orderby cust.lastName ascending
                           orderby cust.firstName descending
                           select cust;
```

```java
List<Customer> result = query.from(customer)
    .where(customer.lastName.like("A%"), customer.active.eq(true))
    .orderBy(customer.lastName.asc(), customer.firstName.desc())
    .list(customer);
```

```sql
select * from customer where lastname like 'A%' and active = 1 order by lastname asc, firstname desc;
```

보시면 query문과 매우 비슷한 구조를 가지고 있습니다.
Querydsl은 `EntityManager`를 한번 더 감싸서 얻을 수 있는 `JPAQuery`와 `Q-class`라는 Query 객체화 class를 이용합니다.

* Hibernate ORM 규칙에 따른 JPA annotation을 이용한 객체 정의
* generate Q-class
* Query Definition을 이용한 query 작성

기존 JPA Project를 QueryDsl을 지원하는 프로젝트로 변경해보도록 하겠습니다.
먼저, `Querydsl`을 지원하기 위한 dependency 적용은 다음과 같습니다.

```groovy
compile group: 'com.querydsl', name: 'querydsl-apt', version: '4.1.4'
compile group: 'com.querydsl', name: 'querydsl-jpa', version: '4.1.4'
```

Querydsl을 지원하기 위한 `Q-Class`를 생성하기 위해 `build.gradle`에 새로운 `task`를 추가합니다.

```groovy
sourceSets {
    main {
        java {
            srcDirs 'src/main/java', 'src/main/generated'
        }
    }
}

task generateQueryDSL(type: JavaCompile, group: 'build', description: 'Generates the QueryDSL query types') {
    file(new File(projectDir, "/src/main/generated")).deleteDir()
    file(new File(projectDir, "/src/main/generated")).mkdirs()
    source = sourceSets.main.java
    classpath = configurations.compile + configurations.compileOnly
    options.compilerArgs = [
            "-proc:only",
            "-processor", "com.querydsl.apt.jpa.JPAAnnotationProcessor"
    ]
    destinationDir = file('src/main/generated')
}
```

최종적으로 다음 gradle task를 실행합니다.

```groovy
gradle generateQuerydsl
```

그 후, 다음과 같은 결과를 보실 수 있습니다.

```cmd
BUILD SUCCESSFUL in 1s
1 actionable task: 1 up-to-date
 secucen  ~/dev/code/secucen-study/step05   feature/is10 ●✚  gradle generateQueryDSL

> Task :generateQueryDSL
Note: Running JPAAnnotationProcessor
Note: Serializing Entity types
Note: Generating com.xyzlast.bookstore.entity.QUser for [com.xyzlast.bookstore.entity.User]
Note: Generating com.xyzlast.bookstore.entity.QBook for [com.xyzlast.bookstore.entity.Book]
Note: Generating com.xyzlast.bookstore.entity.QHistory for [com.xyzlast.bookstore.entity.History]
Note: Running JPAAnnotationProcessor
Note: Running JPAAnnotationProcessor
BUILD SUCCESSFUL in 2s
```

이제 `main/generated`안을 보시면 Q-class 들이 생성되어 있는 것을 보실 수 있습니다.

![](/images/jpa/01.png)

이제 `Repository` class를 수정해보도록 하겠습니다.

```java
@Repository
public class BookDaoImpl implements BookDao {

    @PersistenceContext
    private EntityManager em;

    @Override
    public int countAll() {
        JPAQuery<Book> jpaQuery = new JPAQuery<>(em);
        Long count = jpaQuery.from(QBook.book).fetchCount();
        return count.intValue();
    }

    @Override
    public List<Book> getAll() {
        JPAQuery<Book> jpaQuery = new JPAQuery<>(em);
        return jpaQuery.from(QBook.book).fetch();
    }

    @Override
    public Book getById(int id) {
        JPQLQuery<Book> jpaQuery = new JPAQuery<>(em);
        QBook q = QBook.book;
        return jpaQuery.from(q).where(q.id.eq(id)).fetchFirst();
    }

    @Override
    public int add(Book book) {
        em.merge(book);
        return 1;
    }

    @Override
    public int update(Book book) {
        QBook q = QBook.book;
        JPAUpdateClause updateClause = new JPAUpdateClause(em, q);
        updateClause.where(q.id.eq(book.getId()))
            .set(q.name, book.getName())
            .set(q.author, book.getAuthor())
            .set(q.comment, book.getComment())
            .set(q.publishDate, book.getPublishDate())
            .execute();
        return 1;
    }

    @Override
    public int delete(Book book) {
        QBook q = QBook.book;
        JPADeleteClause jpaDeleteClause = new JPADeleteClause(em, q);
        jpaDeleteClause.where(q.id.eq(book.getId())).execute();
        return 0;
    }

    @Override
    public void deleteAll() {
        JPADeleteClause jpaDeleteClause = new JPADeleteClause(em, QBook.book);
        jpaDeleteClause.execute();
    }
}
```

`BookDao`의 경우, 매우 단순한 select query만을 가지고 있기 때문에 큰 차이를 보이지 않을 수 있습니다. querydsl을 이용하면 조건등에 의한 동적인 query 작성에 매우 탁월합니다.

## Spring Data JPA

`http://spring.io`에서는 여러가지 project를 볼 수 있습니다. 그 중에서 `JPA`에 관련된 프로젝트가 있습니다. `Spring Data`가 바로 그것입니다. Spring Data JPA는 ORM Framework입니다. JPA 기반의 repository를 아주 빨리, 쉽게 개발할 수 있는 방법을 제시하고 있습니다. 지금까지 보던 모든 기술들(JdbcTemplate, Hibernate, myBatis)에서 repository는 객체만 바뀔뿐, 코드의 중복은 계속해서 나타나게 됩니다. 특히 우리가 CRUD라고 부르는 영역의 CUD 코드의 경우에는 거의 대부분이 중복이 되는 경우가 많습니다. 이러한 문제점을 착안하여 Spring Data - JPA project가 시작되게 되었습니다.

Spring Data JPA는 단독으로 동작하는 ORM Framework가 아닙니다. JPA ORM engine을 기반으로 동작하는 ORM Framework입니다. 여기에서 JPA ORM engine이 될 수 있는 표준적인 JSR 220 을 만족하는 ORM engine은 다음과 같습니다.

* Hibernate
* Google App Engine for JAVA
* TopLink

Google App Engine이 RDBMS를 완벽하게 지원하지 못하고, TopLink의 경우에는 Oracle에서 판매하는 상용제품이며 Oracle DB에만 특화가 되어있는 ORM입니다. 그렇다면, 실질적으로 지금 사용할 수 있는 선택의 폭은 Hibernate 밖에는 존재하지 않습니다. 지금 상태로는 Spring Data JPA는 Hibernate를 기반으로 동작하는 ORM Framework이다. 라고 생각해주시면 좋을 것 같습니다.

신규 프로젝트에서부터 시작해서 `Spring Data JPA`를 적용한 프로젝트를 구성해보도록 하겠습니다.

기존 querydsl 프로젝트를 그대로 copy 후, repository들을 모두 지워줍니다. `spring-data-jpa`만을 추가합니다.

```groovy
compile group: 'org.springframework.data', name: 'spring-data-jpa', version: '2.0.2.RELEASE'
```

`@Configuration`은 다음과 같습니다.

```java
@Configuration
@ComponentScan(value = {
    "com.xyzlast.bookstore.repository"
})
@EnableTransactionManagement
@EnableJpaRepositories(basePackages = {
    "com.xyzlast.bookstore.repository"
})
public class BookStoreConfig {
    @Bean
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setDriverClassName("org.mariadb.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://127.0.0.1:4306/bookstore");
        dataSource.setUsername("root");
        dataSource.setPassword("qwer12#$");
        dataSource.setMaximumPoolSize(10);
        dataSource.setMinimumIdle(10);
        return dataSource;
    }
    .....
}
```

`@EnableJpaRepositories`를 추가하고, `Repository`가 위치할 package들만 추가해주면 됩니다.

Spring JPA Data는 Dao/Repository의 반복적인 코드가 될 수 있는 CUD를 자동화된 코드로 지원합니다. Spring JPA Data에서 제공하는 interface는 다음과 같습니다.

* Repository
* CrudRepository
* JpaRepository

이 중에서 우리가 사용할 녀석은 다른 interface를 모두 상속한 JpaRepository<T, C>입니다. JapRepository의 선언은 전에 Hibernate의 GenericDao의 선언과 거의 동일합니다. 단 두번째 인자가 PK에 대한 객체 Type을 넣어주는것만 다릅니다. Book, User, History 모두 int를 사용하고 있기 때문에, Integer를 두번째 Type에 넣어주면 됩니다.

이제 각 `BookDao`, `UserDao`, `HistoryDao`를 pattern에 맞도록 `BookRepository`, `UserRepository`, `HistoryRepository`로 만들어줍시다. 각 Repository 코드는 다음과 같습니다.

```java
@Repository
public interface BookRepository extends JpaRepository<Book, Integer>, QuerydslPredicateExecutor<Book> {

}

@Repository
public interface UserRepository extends JpaRepository<User, Integer>, QuerydslPredicateExecutor<User> {

}

@Repository
public interface HistoryRepository extends JpaRepository<History, Integer>, QuerydslPredicateExecutor<History> {

}
```

interface 구현이 아닌 interface 상속을 통해서 구현이 되어 있는 것을 주의해주세요. JpaRepository<T,C> 의 T에는 Target이 되는 Class가 들어가고, C에는 Target Class의 @Id property의 객체값이 들어갑니다. (int인 경우, Integer)

자. 이제 지금까지 만들었던 BookDaoImpl의 코딩은 모두 끝났습니다. interface에 대한 코딩 작업이 전혀 필요하지 않습니다. 그 이유는 기본적으로 전에 보셨던 GenericDao에 대한 확장과 비슷한 방법입니다. Spring Data JPA는 JpaRepository, Repository, CrudRepository를 상속받은 interface를 모두 찾아내서, 동적인 객체를 생성합니다. 우리가 따로 코드를 만들어줄 필요가 전혀 없는것이지요. GenericDao의 결정판입니다. ^^

이제 테스트 코드를 작성해보도록 하겠습니다. 기존의 테스트 코드에서 약간의 변경만 있으면 됩니다. countAll()을 count()로, add, update를 모두 save로 변경만 시켜주면 테스트 코드가 모두 완성됩니다.

### findByXXX method의 확장

지금 만들어진 `Repository`는 매우 단순한 객체입니다. 만약에 `BookRepository`를 이용해서 `Book.name`을 이용해 검색을 한다면 다음과 같은 코드를 작성해야지 됩니다.

```java
@Test
public void findByName01() {
    QBook qBook = QBook.book;
    Optional<Book> bookOptional = bookRepository.findOne(qBook.name.eq("ykyoon"));
    if (bookOptional.isPresent()) {
        System.out.println(bookOptional.get());
    }
}
```

서비스코드에 이렇게 검색하는 루틴이 들어가 있는 것은 코드의 가독성을 떨어트립니다. Entity의 property만을 이용해서 검색하는 경우에 `spring-data-jpa`는 아주 멋진 방법을 제공합니다. `BookRepository`에 다음과 같은 method를 추가합니다.

```java
@Repository
public interface BookRepository extends JpaRepository<Book, Integer>, QuerydslPredicateExecutor<Book> {
    Book findByName(String name);
}
```

이렇게만 넣어주는 것으로도 위 코드와 동일한 검색을 가능하게 할 수 있습니다. 이렇게 만들어줄 수 있는 패턴들은 다음과 같습니다.

|KEYWORD|SAMPLE	JPQL|SNIPPET|
|---|---|---|
|And|findByLastnameAndFirstname|… where x.lastname = ?1 and x.firstname = ?2|
|Or|findByLastnameOrFirstname|… where x.lastname = ?1 or x.firstname = ?2|
|Between|findByStartDateBetween|… where x.startDate between 1? and ?2|
|LessThan|findByAgeLessThan|… where x.age < ?1|
|GreaterThan|findByAgeGreaterThan|… where x.age > ?1|
|IsNull|findByAgeIsNull|… where x.age is null|
|IsNotNull,NotNull|findByAge(Is)NotNull|… where x.age not null|
|Like|findByFirstnameLike|… where x.firstname like ?1|
|NotLike|findByFirstnameNotLike|… where x.firstname not like ?1|
|OrderBy|findByAgeOrderByLastnameDesc|… where x.age = ?1 order by x.lastname desc|
|Not|findByLastnameNot|… where x.lastname <> ?1|
|In|findByAgeIn(Collection<Age> ages)|… where x.age in ?1|
|NotIn|findByAgeNotIn(Collection<Age> age)|… where x.age not in ?1|

### Predicate의 추가

`findByXXX`로 해결되지 않는 쿼리를 만들어내야지 될 때 사용하는 방법입니다. Domain Logic중에서 다음과 같은 경우를 가정해보도록 하겠습니다.

```cmd
UserId, BookId 입력받아, null인 경우 where 조건이 없이 진행한다.
```

이 경우, Service code에 넣으면 다음과 같이 처리가 되어야지 됩니다.

```java
@Transactional
@Override
public Page<History> listHistories(Integer bookId, Integer userId, int pageIndex, int pageSize) {
    BooleanBuilder queryBuilder = new BooleanBuilder();
    if (bookId != null) {
        queryBuilder.and(QBook.book.id.eq(bookId));
    }
    if (userId != null) {
        queryBuilder.and(QUser.user.id.eq(userId));
    }
    return historyRepository.findAll(queryBuilder, PageRequest.of(pageIndex, pageSize));
}
```

와 같은 코드로 만들어져야지 됩니다. parameter의 검사 및 query를 만드는 코드가 Service에 있다는 것은 원 서비스 코드의 목적인 BL에 대한 구현과 어울리지 않는 코드가 만들어집니다. 따라서 저런 코드들을 Predicate로 넣어서 만드는 것이 일반적입니다.

```java
public class HistoryPredicate {
    private HistoryPredicate() {

    }

    public static Predicate buildQueryByBookIdAndUserId(Integer bookId, Integer userId) {
        BooleanBuilder queryBuilder = new BooleanBuilder();
        if (bookId != null) {
            queryBuilder.and(QBook.book.id.eq(bookId));
        }
        if (userId != null) {
            queryBuilder.and(QUser.user.id.eq(userId));
        }
        return queryBuilder;
    }
}
```

구성된 코드를 이용한 서비스 코드는 다음과 같이 만들어집니다.

```java
@Transactional
@Override
public Page<History> listHistories(Integer bookId, Integer userId, int pageIndex, int pageSize) {
    return historyRepository.findAll(HistoryPredicate.buildQueryByBookIdAndUserId(bookId, userId),
        PageRequest.of(pageIndex, pageSize));
}
```

## Summary

지금까지 `myBatis`, `Hibernate`, `JPA`, `JPA + Querydsl`, `Spring Data JPA + Querydsl`로 구성한 코드를 알아봤습니다.

`Spring Data JPA + Querydsl`를 이용한 코드 패턴은 지금까지 보이는 코드들중 가장 복잡한 구조를 가지고 있지만, 현재 가장 선호되는 패턴입니다.
이렇게 복잡하게 계속해서 묶어주는 이유는 무엇일까요. 가장 큰 이유는 전에 이야기드린 것 처럼, 관계지향적 데이터를 우리가 가지고 있는 `OOP`적 방법으로 사용하기 위해서 입니다.

이 코드 패턴을 꼭 자신의 것으로 만드시길 바랍니다. `Repository Pattern`의 경우, 거의 대부분의 DB 어플리케이션개발의 중심이 됩니다.
