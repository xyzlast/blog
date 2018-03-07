---
title: 09. Hibernate
date: 2018-03-07 16:54:35
tags: java, spring, hibernate
---
# Hibernate

Hibernate는 지금까지 SQL을 이용한 DB query 방법이 아닌, 객체를 이용한 SQL auto generate를 통한 DB 접속 방법을 제공합니다. 이런 객체를 이용한 방법을 ORM(object relation model)이라고 합니다. 먼저, ORM에 대해서 간단한 소개를 해보도록 하겠습니다.

## ORM(Object Relation Model)

java 언어의 발전은 sw 개발에 있어서 객체지향의 개발 방법으로 오는 혜택에 매료가 되었습니다. 그렇지만, 모든 웹 및 기업의 데이터가 저장되어 있는 RDBMS와 OOP간의 괴리의 차이에 의한 비용의 증가가 계속해서 발생하게 되었습니다. OOP적 개발 방법과 RDBMS의 관계형 개발 방법론이 끊임없이 충돌하게 되는 것이지요. 서로간에 전쟁(war) 라는 표현을 사용할 정도로 논란이 매우 큰 문제입니다. java 측에서는 relation 기술의 탓으로 돌리고, data 전문가들은 OOP 기술의 문제라고 약간은 소모적인 논쟁으로 계속해서 가게 되었지요.

이때, ORM(Object Relation Model)은 이러한 불일치 기술에 대한 solution을 지칭할 때 사용됩니다.

ORM은 4가지로 구성이 되어 있습니다.

* Persistence class에 대한 기본적인 CRUD를 수행하는 API
* class 자체나 class의 property를 참조하는 query를 작성할 수 있는 API
* mapping meta data 작성을 위한 기반 API
* ORM 구성이 Transaction과 상호 동작하며 최적화를 수행하도록 돕는 API

왜 ORM을 사용해야지 되는가?

* 생산성 : SQL관련 코드를 제거하고 객체간의 관계를 명확하게 그릴수 있습니다. 그리고 Domain에 집중할 수 있는 설계 구조를 가지고 옵니다.
* 유지보수성 : Domain을 구현했기 때문에 Domain에 대한 명확한 정의가 나타납니다. Domain의 BL이 변경, 추가 되었을 때 그에 대한 수정이 쉽습니다.
* 성능 : 가장 큰 이슈입니다. VM을 설명할 때, VM보다 일반 assembly로 compile되는 언어가 더 빠르지만 이제 사용되지 않는 이유랑 동일합니다. ORM 자체에 이미 많은 최적화 방법들이 구현되어 있고, 그 현인들의 지식을 사용할 수 있다는 점에서 더욱더 큰 장점을 가지고 올 수 있습니다.
* 밴더 독립성 : ORM은 DB와 DB의 방언(각 DB만의 함수들)로부터 추상화 되어 있습니다. DB의 독립성은 매우 큰 장점을 가지고 오는데, 개발에 있어서 오히려 더 나은 장점을 가지고 옵니다. 개발자들은 자신의 개발 PC에 가벼운 DB(mysql)를 설치해서 개발을 하고, 실 서버에서는 Data에 최적화된 DB를 이용해서 서비스를 할 수 있는 장점을 가지고 있습니다. 이는 생산성과도 직결되는 문제입니다.
* Java 표준 : ORM은 JSR 220에 의해서 java에 표준적인 기술로 인정을 받았습니다.

## DDD(Domain Driven Design)

에릭 에반스 (Eric Evans) 는 과거의 지혜와 경험들을 종합하여 도메인-드리븐 디자인 (Domain Driven Design, DDD) 이라는 방법론을 제시 했습니다.
단순 객체지향 세계에서 살던 개발자들은  이 굉장하지만 새로운 개념에 어려움을 느껴 발표된지 몇년이 지난 후에야 관심을 가지게 되었죠.
DDD가 도대체 뭔데? 어떻게 해서든지 돌아가기만 하면 되는거 아냐! 하면서 무심히 지나쳤던 것들에 대해 체계적으로 설명하는 방법론이죠. 하지만 여전히 DDD는 어려운 것, 그저 한때 유행하는 버즈 워드로 인식되고 있는 경향이 있습니다. DDD가 무엇인지 처음 듣는 개발자도 많을 것입니다. DDD의 전체적인 철학을 쉽게 요약하고 있는 블로그 포스트 DDD: How to tackle complexity  번역으로 DDD 카테고리를 시작합니다.

-----------------------------------------------------------------------------------------------------
DDD (Domain Driven Design) 에서는 어플리케이션 도메인을 표현하기 위한 오브젝트 모델을 만듭니다.
이 모델은 도메인의 모든 관계와 로직을 담고 있습니다. 이렇게 하는 목적은 도메인의 복잡성을 관리하기 위함 입니다. DDD 에는 매우 많은 개념과 패턴이 투입되어 있으나, 정제된 두개의 큰 그림으로 그 복잡성에 태클을 걸 수 있습니다.

1. 도메인의 개념을 명확하게 표현합니다.
2. 더욱 심도 있는 통찰을 위해 지속적인 리팩토링을 수행합니다.

복잡성이란 자체의 복잡한 정도를 의미합니다. 복잡한 것은 이해하기 어렵습니다. 이해하기 어려우면 금방 알아 들을 수 없습니다.

이것이 실제 이슈 입니다 : 복잡한 소프트웨어는 이해하기 어렵습니다. 이것이 바로 모든 사람이 업데이트 하기를 두려워 하여 아예 처음부터 다시 만드는 이유입니다. 아마 첫번째나 두번째는 해킹 하듯이 코드를 추가해서 원하는 바를 이룰 수 있을지 모르지만, 각각의 해킹은 더 이상 시도하는 것이 의미가 없을 때까지 복잡성과 추잡함을 증가시킵니다. 이것을 다른 말로 실패 라고 합니다.

그래서 우리는 이 복잡성을 극복해야 합니다. DDD의 첫번째 방법은 객체지향, 모델 과 추상화의 장점을 얻는 것 입니다. 하지만 이건 매우 광범위 하죠. 우리는 이 오브젝트와 모델들을 어떻게 구조화 해야 하는지 알아내야 합니다. 이것이 DDD가 도메인의 개념을 명시적으로 표현하자는 아이디어의 입니다.

아이디어는 간단합니다. 여러분의 도메인에 새로이 적용되는 개념이 있다면 모델에서 확인할 수 있어야 합니다. 중요한 개념을 확인하기 위해서 코드를 뒤져서는 안됩니다. 그 개념은 모델에서 오브젝트로써 표현되어야 합니다. 특정 조건에서만 발생하는 액션이 있다고 합시다, 이 조건들이 중요하지 않다면 그 액션을 수행하도록 그저 IF Statement 메소드로 처리하면 됩니다. 하지만, 그 조건들이 도메인에서 중요하다면 코드로 부터 감추는 것만으로는 부족합니다. 그 조건들을 수행하도록 Policy Object가 조건들을 표현해야 합니다. 이제 조건들은 당신의 도메인에서 명시적으로 표현됩니다.

이 아이디어 들은 Factories, Repositories, Services, Knowledge Levels 등등으로 표현될 수 있습니다. 이 것은 여러분의 시스템을 이해 가능하도록 만드는 중요한 부분입니다.

DDD가 작동하도록 만드는 두번째 아이디어는 "Deeper Insight"를 위한 지속적인 리팩토링 입니다. Deeper Insight 란 이미 가지고 있는 도메인 모델에서 새로운 어떤 것을 발견하게 된다면 대충 끼워넣지 말고 도메인에서 중요한 요소인지 반드시 알아내라는 것을 의미합니다. 만약 중요하다면 새로 이해한 것이 명확하게 표현 되도록 모델을 리팩토링 해야 합니다. 이 리팩토링은 사소할 때도 있고, 매우 중요 할 때도 있습니다.

도메인 모델이 표현성을 잃게 되면 점점 더 부서지고, 점점더 복잡해 지며 점점 더 어려워 집니다. 여러분의 모델이 단순하며  표현력과 정확성을 유지할 수 있도록 항상 싸워야 합니다. 당신이 운이 좋다면 에릭 에반스가 말하는 Break Through [전에는 불가능 했던 것이 새로운 가능성과 통찰력이 갑자기 나타나는 일] 것을 경험할  수도 있습니다. 그렇게 된다면 진짜 운이 좋은 것입니다. 당신이 운이 좋지 않더라도 리팩토링은 적어도 모델이 유연성을 요구할때 유연함을 만족 시킬 수는 있습니다. 이것은 미래에 나타날 통찰력과 리팩토링 요소를 더 쉽게 핸들링 할 수 있음을 의미합니다.

-----------------------------------------------------------------------------------------------------

위 내용과 같이, DDD는 우리가 표현하고자 하는 실세계의 데이터 프로세스 자체를 컴퓨터 언어로 옮겨가는 과정입니다.
예를 들어, SQL로 작업을 하게 되면 다음과 같은 대화가 나오게 됩니다.

**TB_USER에서 SELECT를 할때, POINT Column으로 Order By DESC로 얻어오고, 그 POINT값이 100점 이상인 경우에는 LEVEL column값을 2로 업데이트를 시키면 됩니다.**

자, 방금 말한 내용을 실제 BL을 설계한 기획자에게 이야기를 해보도록 합시다. 과연 어떤 말인지 알아들을수 있을까요? SQL 개발자들이라면 가능하겠지요. DDD로 모델링을 거치면 다음과 같은 대화를 할 수 있습니다.

**User를 표로 보여줄 때, Book Point값이 높은 순서대로 보여주고, Book Point값이 100점이상인 경우에는 Reader로 사용자 Level을 높여줘서 보여주면 됩니다.**

어떤 대화가 좀 더 쉽습니까? 기존의 데이터 중심으로 보는 것과 모델링을 중심으로 보는것. 이 두가지의 차이는 그 데이터의 구조를 모르는 사람이 로직을 읽을수 있느냐, 없느냐에 따라 갈리게 됩니다. 그리고 우리가 문장으로 설명할 수 있는 BL을 code에 어떻게 녹여내느냐를 고민을 해야지 됩니다.

최종적으로 DDD는 다음 목표들을 갖습니다.

### Ubiquitous Language

Domain 중심의 SW팀에서는 모든 참가자들(사용자, 도메인전문가, 설계자, 프로그래머, 분석가)간에 동일한 의미를 갖는 공통된 언어를 갖는다. 심지어 공통된 언어는 Code의 Object로 구현이 가능해야지 된다.

### Layered Architecture pattern

UI와 Model이 결합되어 있으면, UI가 바뀌는데에 따라 Model이 변경되어야지 됩니다. 따라서 UI, BL, Modeling간에 분리가 가능한 Layer들이 만들어져야지 됩니다. 기준에 따라 Layer를 나누고, 역할을 부여하는 작업이 필요합니다. 이에 대한 장점은 다음과 같습니다.

* Layer들이 재사용될 수 있어야지 됩니다.
* 표준을 지원한다.
* 종속성을 국지적으로 최소화한다.
* 교환 가능성이 확보된다.

일반적으로 DDD는 다음 4개의 Layer로 구분시켜서 사용하게 됩니다.

![](/images/hibernate/h01.png)

* Infrastructure Layer : 상위 계층을 지원하는 일반화된 기술적 기능을 제공. 공통 Library, Engine, Framework 영역
* Domain Layer : 업무 개념과 업무 상황에 대한 정보. 업무 규칙을 표현하는 Layer.
* Application Layer : 작업을 정의하고 조정하는 영역. Domain 객체로 작업을 위임하는 역활을 담당.
* UI Layer : 정보를 노출하고 입력을 받아들이는 영역

이러한 구조는 결국은 Domain의 격리, 즉 분리가 이루어지게 됩니다. 이러한 Layer architecture중 가장 대표적인 방식이 MVC 입니다.

### Smart UI anti pattern

모든 업무로직을 사용자 인터페이스에 넣는 설계 방법을 Smart UI pattern이라고 합니다.

특징은

* application을 작은 기능으로 잘게 나누고,
* 나뉜 기능을 분리된 UI로 구현. 업무 규칙이 분리된 UI에 들어가게 합니다.
* 분리된 업무 규칙은 RDBMS를 이용해서 데이터를 공유하고 실행합니다. 주로 SP가 이 용도로 사용됩니다.
* 주로 자동화된 UI 구축 도구와 시각적인 프로그래밍 도구를 이용합니다.

DDD pattern에서 가장 피해야지 되는 것이 바로 UI Pattern입니다.
UI pattern이라는 것은 지금도 매우 자주 쓰이고 있는 패턴입니다. 이 패턴의 가장 큰 문제는 우리가 사용하는 데이터 및 Domain에 대한 모든 로직을 보여지는 View에 맞추게 됩니다. 실질적인 데이터의 흐름을 방해하고, 언제나 바뀔 수 있는 View 영역에 Model이 영속되기 때문에 application의 변경에 취약한 약점을 갖게 됩니다.

대표적인 Smart UI pattern의 도구가 PowerBuilder입니다. Data Window의 Smart UI를 이용하기 위해, UI에 따른 Model과 로직이 꼬인 상태로 존재하게 됩니다. 이는 필연적으로 코드의 중복을 가지고 오게 되며, 아직까지 UI에 대한 명확한 테스트를 할 수 없는 현 시점에서 유지보수가 거의 불가능한 시스템을 만들게 됩니다. *anti pattern은 쓰지 말라고 있는겁니다.*

### Entities pattern

ID를 갖는 unique한 object를 갖는 pattern입니다. 이는 RDBMS의 PK와 연결시켜, 이 객체가 유일한 어떤 값임을 나타내는 방법을 제공합니다.

### Service pattern

서비스는 기능을 처리하거나, Entity로 구별할 수 없는 것을 지칭합니다. 가장 주로 사용되는 것은 BL의 Group을 표현하는 것이 가장 많습니다.

### Factory pattern

각각의 Object, Service의 생성방법에 대한 통일성을 갖는것을 목표로 합니다. 어떤 객체를 생성할 때, 초기화 해줘야지 되는 값이라던지 실행시켜줘야지 되는 method가 독특하게 존재한다면 개발자 및 참여자들은 Domain에 집중할 수 없습니다. Spring의 ApplicationContext는 이런 경우 가장 좋은 해결 방법이 됩니다.

### Repository pattern

생성된 객체나 Model이 외부(주로 DB)와 연동되어 생명주기를 가질때, 그 생명주기에 대한 관리 Focus가 되는 Layer가 존재해야지 됩니다. dao로 구현되는 것이 일반적이며, Spring에서는 @Repository로 지정하는 것이 일반적입니다.

DDD의 사상을 반영하기 위해 주로 사용되는 툴이 ORM이고, java에서 가장 오랜 시간동안 개발이 되고 사랑받고 있는 Hibernate가 그 선두 주자라고 할 수 있습니다.

## Hibernate를 이용한 Repository 개발

새로운 project를 만들고, Hibernate, DAO 관련 library를 추가합니다.

```groovy
compile group: 'org.mariadb.jdbc', name: 'mariadb-java-client', version: '2.2.0'
compile group: 'org.hibernate', name: 'hibernate-core', version: '5.2.12.Final'
compileOnly group: 'org.projectlombok', name: 'lombok', version: '1.16.18'
```

기존에 만들어둔 `Book`, `BookStatus`, `User`, `History`를 copy해서 넣습니다.

`hibernate.cfg.xml` 파일을 만들어주고, 다음과 같은 내용을 넣습니다. `hibernate.cfg.xml` 파일은 기본적으로 db 연결방법들을 제공합니다.

```xml
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd"><hibernate-configuration>
    <session-factory>
        <property name="hibernate.connection.driver_class">org.mariadb.jdbc.Driver</property>
        <property name="hibernate.connection.password">qwer12#$</property>
        <property name="hibernate.connection.url">jdbc:mysql://127.0.0.1:4306/bookstore</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.dialect">org.hibernate.dialect.MariaDB53Dialect</property>
        <property name="hibernate.show_sql">true</property>
    </session-factory>
</hibernate-configuration>
```

Hibernate를 이용한 repository는 다음과 같은 pattern으로 동작합니다.

* SessionFactory를 통한 Session의 확보
* Session을 통한 자동생성된 Query의 실행
* Session close

`myBatis`와 동일한 패턴입니다. 이제 이 패턴대로 `BookDao`를 구성해보도록 하겠습니다.
먼저, DB의 Column과 구현되어 있는 Book간의 연결을 annotation을 이용해서 구성합니다.

```java
@Getter
@Setter
@Entity
@Table(name = "books")
public class Book {
    @Id
    @GeneratedValue
    @Column(name = "id")
    private int id;
    @Column(name = "name", nullable = false, length = 255)
    private String name;
    @Column(name = "comment", nullable = true, length = 255)
    private String comment;
    @Column(name = "author", nullable = false, length = 50)
    private String author;
    @Column(name = "publishDate", nullable = false)
    private Date publishDate;
    @Enumerated(EnumType.ORDINAL)
    @Column(name = "status", nullable = false)
    private BookStatus bookStatus;
    @Column(name = "rentUserId", nullable = true)
    private Integer rentUserId;
}
```

객체의 각 property와 DB의 물리적인 상관관계를 annotation을 통해서 relation을 갖게 하는 과정입니다. 이를 통해 객체지향적인 java 객체가 관계지향/물리적 DB와의 연결이 되게 됩니다.

다음은 `Repository` 코드입니다.

```java
public class BookDaoImpl implements BookDao {

    private final SessionFactory sessionFactory;

    public BookDaoImpl(SessionFactory sessionFactory) {
        this.sessionFactory = sessionFactory;
    }

    @Override
    public int countAll() {
        try (Session session = sessionFactory.openSession()) {
            CriteriaBuilder builder = session.getCriteriaBuilder();
            CriteriaQuery<Long> query = builder.createQuery(Long.class);
            query.select(builder.count(query.from(Book.class)));
            return session.createQuery(query).getSingleResult().intValue();
        }
    }

    @Override
    public List<Book> getAll() {
        try (Session session = sessionFactory.openSession()) {
            return session.createQuery("from Book", Book.class).list();
        }
    }

    @Override
    public Book getById(int id) {
        try (Session session = sessionFactory.openSession()) {
            Book book = session.byId(Book.class).load(id);
            return book;
        } catch (Exception ex) {
            return null;
        }
    }

    @Override
    public int add(Book book) {
        try (Session session = sessionFactory.openSession()) {
            session.save(book);
            return 1;
        }
    }

    @Override
    public int update(Book book) {
        try (Session session = sessionFactory.openSession()) {
            session.save(book);
            return 1;
        }
    }

    @Override
    public int delete(int bookId) {
        try (Session session = sessionFactory.openSession()) {
            Book deleteBook = new Book();
            deleteBook.setId(bookId);
            session.delete(deleteBook);
            return 1;
        }
    }

    @Override
    public void deleteAll() {
        try (Session session = sessionFactory.openSession()) {
            CriteriaBuilder builder = session.getCriteriaBuilder();
            CriteriaDelete<Book> query = builder.createCriteriaDelete(Book.class);
            query.from(Book.class);
            session.createQuery(query);
        }
    }
}
```

이 패턴은 전에 refactoring을 진행했던 Template-Callback pattern을 통해 Refactoring이 가능합니다. HibernateRefactoring을 진행해보도록 하겠습니다.
테스트 코드를 통해 실행되는 것을 확인하면 다음과 같이 구성 가능합니다.

```java
@SuppressWarnings("unused")
public class BookDaoImplTest {
    private ServiceRegistry serviceRegistry;
    private SessionFactory sessionFactory;

    @Before
    public void setUp() throws Exception {
        Configuration configuration = new Configuration();
        configuration.configure("hibernate.cfg.xml");
        configuration.addAnnotatedClass(Book.class);
        configuration.addAnnotatedClass(User.class);
        configuration.addAnnotatedClass(History.class);
        serviceRegistry = new StandardServiceRegistryBuilder()
            .applySettings(configuration.getProperties()).build();
        sessionFactory = configuration.buildSessionFactory(serviceRegistry);
    }

    @Test
    public void getAll() {
        BookDao bookDao = new BookDaoImpl(sessionFactory);
        bookDao.getAll();
    }
}
```

기존과 좀 다른 형태의 `DaoImpl`이 만들어집니다. 이렇게만 작성하면 `ORM`과 전에 나온 `myBatis`간의 차이가 무엇인지 모르게됩니다. 이제 더 깊이 가보도록 하겠습니다.

## Hibernate를 이용한 Service의 개발 - DDD

이제 서비스의 개발을 확인해보도록 하겠습니다. 다음과 같은 BL을 생각해보도록 하겠습니다.

** 사용자가 책을 대여한다. 대여하면서 사용자 point가 10점씩 증가된다. **

서비스 코드는 다음과 같이 구성될 수 있을 것입니다.

```java
public void rentBook(int userId, int bookId) {
    Book rentBook = bookDao.getById(bookId);
    User user = userDao.getById(userId);

    rentBook.setRentUser(user);
    user.setPoint(user.getPoint() + 10);

    userDao.update(user);
    bookDao.update(rentBook);
}
```

위와 같이 절차적으로 구성이 가능합니다.

> 책을 얻어낸다.
> 사용자를 얻어낸다.
> 책에 대여사용자를 셋팅한다.
> 사용자의 point 값을 증가시킨다.
> Dao를 이용해서 update 시킨다.

그렇지만 이는 객체지향적이지 못합니다.

`ORM`으로, 아니 `DDD`로 구성을 한다면 처음 이야기한 BL 그대로 해석이 가능한 코드가 서비스에서 녹아들어갈 수 있어야지 됩니다. 다음과 같이요.

```java
public void rentBook(int userId, int bookId) {
    Book book = bookDao.getById(bookId);
    User user = userDao.getById(userId);
    user.rentBook(book, 10, bookDao, userDao);
}
```

위와 같이 `Entity` 객체들에 단순한 property만을 넣는 것이 아닌, action에 대한 처리 역시 `Entity`에 구현해서 보다더 확장된 `Entity`를 만들어주게되면 코드의 확장이 더욱더 잘 됩니다.
위와 같이 확장된 `User`의 코드는 다음과 같이 구성 가능합니다.

```java
public void rentBook(Book book, int incrementPoint, BookDao bookDao, UserDao userDao) {
    book.setRentUser(this);
    bookDao.update(book);
    this.setPoint(this.getPoint() + incrementPoint);
    userDao.update(this);
}
```

이에 대한 테스트를 작성한다면 다음과 같습니다.

```java
@Test
public void rentTest() {
    UserDao userDao = mock(UserDao.class);
    BookDao bookDao = mock(BookDao.class);

    User user = new User();
    user.setPoint(100);
    Book book = new Book();

    user.rentBook(book, 10, bookDao, userDao);

    //book에 대여자가 정상적으로 설정되었는지 확인
    assertThat(book.getRentUser()).isEqualTo(user);
    //user에 정상적으로 Point 값이 설정되었는지 확인
    assertThat(user.getPoint()).isEqualTo(110);
    //Repository를 통해서 객체 값이 업데이트가 되고 있는지 체크 필요
    verify(userDao, times(1)).update(user);
    verify(bookDao, times(1)).update(book);
}
```

BL 로직 자체를 DB가 없이 가능하게 되며, Entity간의 Domain Logic에 의해서 처리되고 있는 것을 볼 수 있습니다. 이는 서비스의 구현에 있어서 용어의 통일 및 로직의 확장에 용의하게 됩니다.

## Entity의 정의

Entity의 정의는 기존 `VO`나 `DTO`를 만들어낼때 사용되는 간단한 getter/setter만을 사용하지 않고 BL의 action의 확장 역시 가능합니다. 또한 Entity는 다른 Entity간의 Relation을 가지게 됩니다.

```java
@Getter
@Setter
@Entity
@Table(name = "books")
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private int id;
    @Column(name = "name", nullable = false, length = 255)
    private String name;
    @Column(name = "comment", nullable = true, length = 255)
    private String comment;
    @Column(name = "author", nullable = false, length = 50)
    private String author;
    @Column(name = "publishDate", nullable = false)
    private Date publishDate;
    @Enumerated(EnumType.ORDINAL)
    @Column(name = "status", nullable = false)
    private BookStatus bookStatus;
    @ManyToOne
    @JoinColumn(name = "rentUserId", nullable = true)
    private User rentUser;
    @OneToMany(mappedBy = "book")
    private List<History> histories;
}

@Getter
@Setter
@Entity
@Table(name = "histories")
public class History {
    @Id
    @GeneratedValue
    @Column(name = "id")
    private int id;
    @ManyToOne
    @JoinColumn(name = "bookId")
    private Book book;
    @ManyToOne
    @JoinColumn(name = "userId")
    private User user;
    @Enumerated(EnumType.ORDINAL)
    @Column(name = "actionType")
    private HistoryActionType actionType;
    @Column(name = "insertDate")
    private Date date;
}

@Getter
@Setter
@Entity
@Table(name = "users")
public class User {
    @Id
    @Column(name = "id")
    @GeneratedValue
    private int id;
    @Column(name = "name", length = 50, nullable = false)
    private String name;
    @Column(name = "password", length = 12, nullable = false)
    private String password;
    @Column(name = "point", nullable = false)
    private int point;
    @Column(name = "level", nullable = false)
    private int level;
    @OneToMany(mappedBy = "user")
    private List<History> histories;

    public void rentBook(Book book, int incrementPoint, BookDao bookDao, UserDao userDao) {
        book.setRentUser(this);
        bookDao.update(book);
        this.setPoint(this.getPoint() + incrementPoint);
        userDao.update(this);
    }
}
```

각 객체들간의 새로운 연관관계가 보이는 것을 볼 수 있습니다. `Book`이라는 객체를 통해서, RentUser를 얻어낼 수 있으며, Book의 연관 History를 모두 얻어낼 수 있습니다.
이러한 연관관계를 정의하는 annotation들은 다음과 같습니다.

|이름|설명|
|---|---|
|@Entity|entity 객체임을 지정하는 annotation입니다.|
|@Table|entitty 객체와 연결되는 table을 지정하는 annotation입니다.|
|@Id|PK를 지정하는 annotation입니다.|
|@Column|DB의 Column과 1:1로 mapping되는 Property를 지정하는 annotation입니다.|
|@GeneratedValue|PK의 값이 지정되는 방법을 결정할 수 있습니다. AUTO의 경우 mysql의 AUTO_INCREMENT에 해당됩니다. 각각의 DB에 따라 다른 값을 넣어주는 것도 가능합니다.(ex : oracle의 sequence)|
|@Enumerated|enum 값과 1:1로 mapping이 되는 것을 지정합니다.|
|@ManyToOne|자신과 같은 객체들이 다른 한개의 객체에 연관이 있음을 지정합니다. 이는 N:1의 속성을 지정하게 됩니다.|
|@JoinColumn|@ManyToOne과 같이 사용됩니다. Join 되는 Column을 지정합니다. (FK column)|
|@OneToMany|자신이 다른 객체들에 연관이 있음을 지정합니다. 이는 1:N 또는 N:N의 속성을 지정합니다.|

그리고, Entity에 대한 개념을 다시 한번 정리해보겠습니다.

### Entity의 요구사항

하나의 엔티티 클래스는 다음의 요구조건을 충족해야 합니다:

* 클래스 선언부에 javax.persistence.Entity 어노테이션을 반드시 명시하여야 합니다.
* 기본 생성자를 반드시 포함해야 합니다. 기본생성자는 인수가 없는 생성자를 의미합니다. 만일, 인수를 포함한 생성자를 사용한다면 명시적으로 기본 생성자를 만들어야만 합니다.
* 클래스를 fianl로 선언해서는 안됩니다.
* 엔티티 인스턴스가 세션빈의 리모트 비지니스 인터페이스와 같이 detached object형태로 전달되는 경우, 클래스는 반드시 Serializble 인터페이스를 구현해야 합니다.
* 엔티티는 엔티티 또는 non-엔티티 클래스 모두 확장(extend)이 가능하며, non-엔티티 클래스는 엔티티 클래스를 확장할 수 있습니다.
* 영속화 인스턴스 변수는 반드시 private, protected, package-private중 하나로 선언되어야 하며, 엔티티 클래스의 메소드에 의해 직접 참조될 수 있습니다. 클라이언트는 엔티티의 상태를 접근자(accessor)또는 비지니스 메소드를 통해 접근이 가능합니다.

또한, 영속상태의 엔티티는 엔티티의 인스턴스 변수 또는 자바빈 스타일의 속성에 의해 접근할 수 있습니다. 필드 또는 속성은 다음의 자바 언어 타입에 따라야지 됩니다.

* 자바 원시 타입
* java.lang.String
* 그외 직렬화 가능(Serializable) 타입들
* 자바 원시타입의 Wrappers
* java.math.BigInteger
* java.math.BigDecimal
* java.util.Date
* java.util.Calendar
* java.sql.Date
* java.sql.Time
* java.sql.Timestamp
* 사용자 정의 직렬화 타입들
* byte[]
* Byte[]
* char[]
* Character[]
* 열거형(Enumerated) 타입들
* 다른 엔티티 또는 엔티티 컬렉션
* 내장형(Embeddable) 클래스

### Entity간의 Relation

각 엔티티들은 다음과 같은 관계를 가질 수 있습니다.

* One-to-one : 각 엔티티 인스턴스는 하나의 인스턴스가 다른 엔티티와 연관됩니다. One-to-one 관계는 javax.persistence.OneToOne 어노테이션으로 해당 필드에 정의합니다.
* One-to-many : 하나의 엔티티 인스턴스가 다수의 다른 엔티티 인스턴스와 연관됩니다. 영업주문의 경우 다수의 라인 아이템을 가집니다. 즉, Order 엔티티는 여러개의 LineItem 을 가지므로 이들 사이에는 One-to-many 관계가 선언되어야 하므로 javax.persistence.OneToMany 어노테이션을 사용합니다.
* Many-to-one : 다수의 인스턴스가 하나의 다른 엔티티와 연관됩니다. 이것은 One-to-many 와 반대입니다. 영업주문의 경우 LineItem은 Order에 대해 Many-to-one 관계가 성립되므로 javax.persistence.ManyToOne 어노테이션을 지정합니다.
* Many-to-many : 이 인스턴스는 다수의 인스턴스가 각기 다른 엔티티들과 연관됩니다. 예를들어, 학교에서 각 수업들은 다수의 학생들과 연관이 있으며, 학생 역시 다수의 수업을 듣고 있습니다. 이경우 수업과 학생은 Many-to-Many 관계가 성립되며 javax.persistence.ManyToMany 어노테이션을 사용합니다.

Entity-Entity Relation은 매우 중요한 개념입니다. 이를 통해 RDBMS와 OOP간의 간극을 없애주는 핵심 개념입니다.

## Spring과 Hibernate 연결

지금까지 작성되었던 Hibernate를 이용한 dao, service layer를 spring을 이용하도록 수정해보도록 하겠습니다.

먼저 조금 소개를 한다면, Spring과 Hibernate는 원래 사이가 좋은 편은 아니였습니다.
예전 Spring 1.x와 Hibernate 2.x 간에는 개발자간에 매우 심각한 대립이 존재를 했었고, 그로 인한 키보드 배틀이 엄청나게 있었지요.
가장 큰 이유는 예전 Spring에서는 HibernateTemplate을 제공하고 있었습니다. Hibernate는 DB에 접근하는 방법을 Session을 바로 얻어서 사용하도록 되어 있는데, 이러한 Raw level 접근을 Spring을 사용함으로 Spring에서 Hibernate의 기본 사상을 망가트리고 있다는 불만이 가장 컷지요.

그렇지만, Hibernate 3.x대로 넘어가고, Spring이 2.x대로 넘어가면서 화해(?)를 하게 됩니다. Spring 측에서 HibernateTemplate을 사용하지 않고, Spring에서 Hibernate를 사용할 수 있도록 Spring Framework의 큰 부분을 변경했습니다. 그리고 그에 맞추어 Hibernate에서는 SessionFactory에서 getCurrentSession() method를 추가함으로서, Spring의 기본적인 Transaction 방법에 맞추어 Hibernate를 이용한 DB 접근을 가능하게 하였습니다.

둘간의 관계는 Spring 자체는 application framework입니다. Hibernate는 ORM framework고요. Hibernate 자체가 Spring보다 범위가 작은 Framework라고도 할 수 있습니다. Java의 모든 application은 Spring을 사용할 수 있지만, DB를 사용하지 않는 Application은 Hibernate를 사용하지 않을테니까요. 그럼에도 불구하고, Java의 최고 양대 open source framework는 spring과 hibernate입니다. 둘은 open source가 세상을 얼마나 바꾸어 놓을 수 있는지 보여주었고, 상업적으로도 엄청난 성공을 거뒀습니다. spring은 지금 vmware에서 제공되고 있고, hibernate는 weblogic을 제공하는 JBOSS에서 제공되고 있습니다.

잡설이 길었습니다. spring은 이러한 hibernate를 위한 library를 따로 분리해서 사용하고 있습니다.

구성될 dependencies 입니다.

```groovy
compileOnly group: 'org.projectlombok', name: 'lombok', version: '1.16.18'

compile 'com.google.guava:guava:23.0'
compile group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: '2.9.3'
compile group: 'org.mariadb.jdbc', name: 'mariadb-java-client', version: '2.2.0'
compile group: 'org.hibernate', name: 'hibernate-core', version: '5.2.12.Final'
compile group: 'com.zaxxer', name: 'HikariCP', version: '2.7.4'
compile group: 'org.springframework', name: 'spring-context', version: '5.0.2.RELEASE'
compile group: 'org.springframework', name: 'spring-orm', version: '5.0.2.RELEASE'

testCompile 'junit:junit:4.12'
testCompile 'org.assertj:assertj-core:3.8.0'
testCompile group: 'org.springframework', name: 'spring-test', version: '5.0.2.RELEASE'
```

이제 `@Configuration`을 구성하겠습니다.

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
    public PlatformTransactionManager transactionManager() {
        HibernateTransactionManager transactionManager = new HibernateTransactionManager();
        transactionManager.setDataSource(dataSource());
        transactionManager.setSessionFactory(sessionFactoryBean().getObject());
        return transactionManager;
    }

    @Bean
    public LocalSessionFactoryBean sessionFactoryBean() {
        Properties properties = new Properties();
        properties.setProperty("hibernate.dialect", "org.hibernate.dialect.MariaDB53Dialect");
        properties.setProperty("hibernate.show_sql", "true");

        LocalSessionFactoryBean localSessionFactoryBean = new LocalSessionFactoryBean();
        localSessionFactoryBean.setHibernateProperties(properties);
        localSessionFactoryBean.setPackagesToScan("com.xyzlast.bookstore.entity");
        localSessionFactoryBean.setDataSource(dataSource());
        return localSessionFactoryBean;
    }

    @Bean
    public HibernateExceptionTranslator hibernateExceptionTranslator() {
        return new HibernateExceptionTranslator();
    }
}
```

이제 변경되는 `Repository` 코드는 다음과 같습니다.

```java
@Repository
public class BookDaoSpringImpl implements BookDao {
    private final SessionFactory sessionFactory;

    @Autowired
    public BookDaoSpringImpl(SessionFactory sessionFactory) {
        this.sessionFactory = sessionFactory;
    }

    @Override
    public int countAll() {
        Session session = sessionFactory.getCurrentSession();
        CriteriaBuilder builder = session.getCriteriaBuilder();
        CriteriaQuery<Long> query = builder.createQuery(Long.class);
        query.select(builder.count(query.from(Book.class)));
        return session.createQuery(query).getSingleResult().intValue();
    }

    @Override
    public List<Book> getAll() {
        Session session = sessionFactory.getCurrentSession();
        return session.createQuery("from Book", Book.class).list();
    }

    @Override
    public Book getById(int id) {
        Session session = sessionFactory.getCurrentSession();
        return session.byId(Book.class).load(id);
    }

    @Override
    public int add(Book book) {
        Session session = sessionFactory.getCurrentSession();
        session.save(book);
        return 1;
    }

    @Override
    public int update(Book book) {
        Session session = sessionFactory.getCurrentSession();
        session.update(book);
        return 1;
    }

    @Override
    public int delete(Book book) {
        Session session = sessionFactory.getCurrentSession();
        session.delete(book);
        return 1;
    }

    @Override
    public void deleteAll() {
        Session session = sessionFactory.getCurrentSession();
        session.createQuery("DELETE FROM Book").executeUpdate();
    }
}
```

차이는 `Session session = sessionFactory.openSession()`에서 `Session session = sessionFactory.getCurrentSession()`으로 바뀐것입니다. 그 이외에는 별 차이가 없습니다.
이제 만들어진 `Dao`를 가지고 Test code를 돌려보도록 합니다. 기존 test 코드를 그대로 사용 가능할 것입니다.

```java
@SuppressWarnings("unused")
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = BookStoreConfig.class)
public class BookDaoSpringImplTest {
    @Autowired
    private BookDao bookDao;

    @Test
    public void getTest() {
        List<Book> allBooks = bookDao.getAll();
        if (allBooks.isEmpty()) {
            bookDao.add(TestValueGenerator.generateNewBook());
            allBooks = bookDao.getAll();
        }
        Book checkBook = allBooks.get(allBooks.size() - 1);
        Book getBook = bookDao.getById(checkBook.getId());
        Assert.assertEquals(checkBook.getId(), getBook.getId());
    }
  ......
}
```

돌리면 바로 다음과 같은 에러가 발생할 것입니다.

```cmd
Jan 11, 2018 9:46:18 AM org.hibernate.engine.jdbc.env.internal.LobCreatorBuilderImpl makeLobCreatorBuilder
INFO: HHH000422: Disabling contextual LOB creation as connection was null
org.hibernate.HibernateException: Could not obtain transaction-synchronized Session for current thread
	at org.springframework.orm.hibernate5.SpringSessionContext.currentSession(SpringSessionContext.java:136)
	at org.hibernate.internal.SessionFactoryImpl.getCurrentSession(SessionFactoryImpl.java:465)
```

에러가 발생되는 원인은 바로 변경된 코드인 `getCurrentSession()` 입니다. 현 상태는 Session이 `Open` 되어 있지 않은 상태입니다. `@Transactional` annotation을 통해 Session을 `Open`시켜줍니다. 이를 통해 `@Transactional`로 묶여 있는 한개의 method가 하나의 BL이 되는 명확한 지정이 됩니다.

## Summary

ORM을 이용한 개발에 대해서 알아봤습니다. ORM을 이용한다는 것을 DDD(Domain Driven Development)와 밀접한 연관을 가지고 있습니다. BL과 우리가 개발하는 코드간의 언어적 갭을 최대한 줄이는 것이 목적입니다. 새로운 언어가 나오면 그에 대한 Application Framework가 나오게 되고, 거의 동시에 ORM Framework가 나오는 것이 대부분의 추세입니다. 특히 Spring에서의 `Repository Pattern`은 기본적으로 ORM을 사용하는 것을 기본으로 구성하는 경우가 많습니다. 비록 국내 SI에서는 잘 사용되고 있지 않지만, 최근 스타트업 붐에 보이는 거의 대부분의 기업들이 ORM을 사용하고 있는 것이 현실입니다. 잘 익혀둬야지 됩니다.

ORM의 사용은 단순 코드의 개발로 끝나지 않습니다. 개발 패턴에 대한 내용이고, 사상에 대한 이슈입니다.
