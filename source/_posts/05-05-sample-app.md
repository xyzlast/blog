---
title: 05-05.SampleApplication(5)
date: 2018-02-05 09:06:39
tags: study,java,spring
---
# SampleApplication(5) - using SpringDao

지금까지 우리는 Template-callback 구조의 SqlExecutor와 Connection을 처리하는 ConnectionFactory를 이용한 Dao 객체들을 구성하였습니다. 지금까지 만든 Dao 객체들을 Spring에서 제공하는 Jdbc객체들을 이용해서 변환시키는 과정을 한번 알아보도록 하겠습니다.

## Spring JDBC를 이용한 DAO의 개발

Spring JDBC는 지금까지 이야기한 모든 기능들이 다 포함되어있습니다.

* Template, callback 구조
* DataSource를 이용한 ConnectionFactory 구현
* Checked Exception을 Runtime Exception으로 변경

먼저, `build.gradle`에 다음 dependency를 추가합니다.

```groovy
compile group: 'org.springframework', name: 'spring-jdbc', version: '5.0.2.RELEASE'
```

### ConnectionFactory -> DataSource

ConnectionFactory를 대신할 객체로 Spring에서는 `DataSource` 인터페이스를 구현한 `DriverManagerDataSource`를 제공합니다. `@Configuration`에서 다음과 같이 정의할 수 있습니다.

```java
@Bean
public DataSource dataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();
    dataSource.setDriverClassName("org.mariadb.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://127.0.0.1:4306/bookstore");
    dataSource.setUsername("root");
    dataSource.setPassword("qwer12#$");
    return dataSource;
}
```

참고삼아, `JNDI`를 이용한 `DataSource`의 경우에는 다음과 같이 구성 가능합니다.

```java
@Bean
public DataSource jndiDataSource() {
    JndiDataSourceLookup lookup = new JndiDataSourceLookup();
    lookup.setResourceRef(true);
    return lookup.getDataSource(jndiName);
}
```

### SqlExecutor -> JdbcTemplate

구현된 `SqlExecutor`에서 제공되는 Template, Callback을 이용한 객체를 `Spring`에서는 `JdbcTemplate`로 제공하고 있습니다. 객체 Bean은 다음과 같이 등록이 가능합니다.

```java
@Bean
public JdbcTemplate jdbcTemplate() {
    return new JdbcTemplate(dataSource());
}
```

`JdbcTemplate`은 다음 method들을 가지고 있습니다.

* queryForList: select에 의해서 List return이 발생할 때 사용됩니다.
* queryForObject: 한개의 객체를 return하는 method입니다.
* update: insert/update/delete query에서 사용됩니다. return값은 영향 받는 row의 갯수를 반환합니다.

기존 코드를 이제 `JdbcTemplate`를 이용해서 모두 변경해보도록 하겠습니다. 기존 `BookDao`는 다음과 같이 변경이 가능합니다.

```java
public class BookDao implements BookStoreDao<Book, Integer> {

    private final JdbcTemplate jdbcTemplate;

    public BookDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public Integer countAll() {
        String sql = "select count(*) from books";
        return jdbcTemplate.queryForObject(sql, Integer.class);
    }

    @Override
    public void deleteAll() {
        String sql = "delete from books";
        jdbcTemplate.update(sql);
    }

    @Override
    public List<Book> getAll() {
        String sql = "select id, name, author, status, rentUserId, comment, publishDate from books";
        List<Book> books = new ArrayList<>();
        List<Map<String, Object>> maps = jdbcTemplate.queryForList(sql);
        for (Map<String, Object> map : maps) {
            books.add(convertBookFromResultSet(map));
        }
        return books;
    }

    @Override
    public Book getById(Integer id) {
        String sql = "select id, name, author, publishDate, comment, status, rentUserId from books where id=?";
        return jdbcTemplate.queryForObject(sql, new Object[] {id}, (rs, rowNum) -> convertBookFromResultSet(rs));
    }

    @Override
    public boolean update(Book book) {
        String sql = "update books set name = ?, author = ?, publishDate = ?, comment = ?, status = ?, rentUserId =? " +
            "where id = ?";
        return jdbcTemplate.update(sql,
            book.getName(), book.getAuthor(),
            new java.sql.Date(book.getPublishDate().getTime()),
            book.getComment(),
            book.getBookStatus().value(),
            book.getRentUserId(),
            book.getId()) == 1;
    }

    @Override
    public boolean add(Book book) {
        String sql = "insert books(name, author, publishDate, comment, status, rentUserId) values(?, ?, ?, ?, ?, ?)";
        return jdbcTemplate.update(sql,
            book.getName(), book.getAuthor(),
            new java.sql.Date(book.getPublishDate().getTime()),
            book.getComment(),
            book.getBookStatus().value(),
            book.getRentUserId()) == 1;
    }

    @Override
    public boolean delete(Book entity) {
        String sql = "delete from books where id = ?";
        return jdbcTemplate.update(sql, entity.getId()) == 1;
    }

    private Book convertBookFromResultSet(Map rs) {
        Book book = new Book();
        book.setId((int) rs.get("id"));
        book.setName((String) rs.get("name"));
        book.setAuthor((String) rs.get("author"));
        book.setPublishDate((Timestamp) rs.get("publishDate"));
        book.setBookStatus(BookStatus.parse((int) rs.get("status")));
        Integer rentUserId = (Integer) rs.get("rentUserId");
        book.setRentUserId(rentUserId);
        book.setComment((String) rs.get("comment"));
        return book;
    }

    private Book convertBookFromResultSet(ResultSet rs) throws SQLException {
        Book book = new Book();
        book.setId(rs.getInt("id"));
        book.setName(rs.getString("name"));
        book.setAuthor(rs.getString("author"));
        java.util.Date date = new java.util.Date(rs.getDate("publishDate").getTime());
        book.setPublishDate(date);
        book.setBookStatus(BookStatus.parse(rs.getInt("status")));
        Integer rentUserId = (Integer) rs.getObject("rentUserId");
        book.setRentUserId(rentUserId);
        book.setComment(rs.getString("comment"));
        return book;
    }
}
```

## Bean의 등록

`BookDao`, `UserDao`, `HistoryDao`를 각각 `ApplicationContext`에 등록하게 되면 다음과 같이 구성됩니다.

```java
@Configuration
public class BookStoreConfig {
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.mariadb.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://127.0.0.1:4306/bookstore");
        dataSource.setUsername("root");
        dataSource.setPassword("qwer12#$");
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dataSource());
    }

    @Bean
    public BookDao bookDao() {
        return new BookDao(jdbcTemplate());
    }

    @Bean
    public UserDao userDao() {
        return new UserDao(jdbcTemplate());
    }

    @Bean
    public HistoryDao historyDao() {
        return new HistoryDao(jdbcTemplate());
    }
}
```

만약 `Dao` 객체들이 여러개가 된다면 이러한 코드 패턴을 여러개를 계속해서 만들어줘야지 됩니다. 이를 간단하게 처리할 수 있는 방법으로 `ComponentScan`을 제공합니다. `ComponentScan`은 다음과 같이 사용 가능합니다.

```java
@Configuration
@ComponentScan(basePackages = {
    "com.xyzlast.bookstore.dao"
})
public class BookStoreConfig {
  ...
}
```

해당되는 `package`안의 객체들을 자동으로 `ApplicationContext`에 등록하게 되는데, 이는 annotation을 기준으로 갖습니다. Spring에서 `ComponentScan`에 의하여 자동 등록되는 기준 annotation은 다음과 같습니다.

|종류|설명|
|---|---|
|@Component|일반적인 객체에 사용됩니다.|
|@Repository|DB에 접근되는 객체에 사용됩니다. 일반적으로 `DAO`, `Repository` 객체에 적용이 됩니다.|
|@Service|Business Logic이 구성되는 Service 객체에 사용됩니다. BL은 여러개의 Repository의 구성으로 만들어집니다.|
|@Controller|WebMVC에서 사용되는 객체입니다. Url Request와 연결되는 객체를 지정할 때 사용됩니다.|

그리고, 자동등록이 지정된 객체들에 자동으로 추가되어야지 될 내용들을 어떻게 지정해주어야지 될까요? 지금 `Dao` 객체들은 생성자에 `JdbcTemplate`이 `ApplicationContext`에 등록된 객체를 사용할 수 있도록 지정해야 합니다. 이럴때 `@Autowired`를 이용합니다.

```java
@Repository
public class BookDao implements BookStoreDao<Book, Integer> {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public BookDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    ...
}
```

`@Autowired`가 지정된 객체들은 `ApplicationContext`에서 생성될 순서가 뒤로 밀리게 됩니다. 우선 다른 객체들의 dependency가 없는 객체들부터 우선 생성하고, `@Autowired`가 지정된 객체들을 찾아서 처리하게 됩니다. 이때, 지정된 객체를 찾는 것은 다음과 같은 순서를 갖습니다.

1. `@Autowired`될 객체의 Bean 이름을 지정한 경우
2. `@Autowired`될 객체의 type이 동일한 경우

위 케이스의 경우, 객체의 type이 동일한 경우입니다. `Bean`의 이름이 지정되어 있지 않기 때문입니다. 만약에 `UserDao`, `BookDao`가 각각 다른 `JdbcTemplate`를 사용해야지 된다면, `@Qualifier`를 이용해서 각기 사용할 Bean을 지정해줄 수 있습니다.

```java
@Bean("jdbcTemplate")
public JdbcTemplate jdbcTemplate() {
    return new JdbcTemplate(dataSource());
}

@Bean("jdbcTemplate2")
public JdbcTemplate jdbcTemplate2() {
    return new JdbcTemplate(dataSource());
}
```

```java
@Repository
public class BookDao implements BookStoreDao<Book, Integer> {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public BookDao(@Qualifier(value = "jdbcTemplate") JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    ...
}

@Repository
public class UserDao implements BookStoreDao<User, Integer> {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public UserDao(@Qualifier(value = "jdbcTemplate2") JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    ...
}
```

## DataSource의 변경

`DriverManagerDataSource`는 매우 단순한 `DataSource`입니다. 1개의 `connection`에 대한 `open`/`close`만을 담당하고 있는 매우 단순한 형태이지요. 실무에서는 절대로 사용되지 않는 형태의 `DataSource`입니다. 실무에서는 `JNDI` 또는 `Connection Pool`을 사용합니다.

`Connection Pool`은 미리 `Connection`을 준비해두고, `DAO`에서 사용된 `Connection`을 `Pool`에 넣어서 관리하는 형태입니다. 이 방식은 다음과 같은 장점을 갖습니다.

* DB에 대한 가장 큰 부하인 Connection open / close 횟수를 줄여, System의 부하를 줄일수 있습니다.
* Web과 같은 동시접근성이 보장되어야지 되는 시스템에서 Connection의 여유분을 만들어서, 시스템의 성능을 높일 수 있습니다.
* DB System에 대한 max connection 숫자를 파악할 수 있기 때문에, DB System에 대한 부하 및 성능에 대한 예측이 가능합니다.

주로 사용되는 ConnectionPool Library는 다음 5가지를 볼 수 있습니다.

* apache common-dhcp
* tomcat jdbc
* boneCP
* HikiriCP
* c3p0

주로 사용되고 있는 `apache common-dhcp`는 사용버젼에 주의할 필요가 있습니다. `1.4.1`이전 버젼의 경우 memoryLeak이 발생합니다. 그런데, `1.4.1` 정식버젼이 아직 release되고 있지 않습니다. 따라서, 다음 선언을 모두 변경하는 것이 좋습니다.

```groovy
compile group: 'commons-dbcp', name: 'commons-dbcp', version: '1.4'
```

```groovy
compile group: 'org.apache.commons', name: 'commons-dbcp2', version: '2.1.1'
```

개인적으로는 `HikariCP`를 추천합니다. 성능이 좋습니다.

![](https://github.com/brettwooldridge/HikariCP/wiki/HikariCP-bench-2.6.0.png)

`ConnectionPool`은 다음과 같은 일반적인 속성들을 갖습니다.

* idleConnectionTestPeriodInMinutes : connection이 쉬고 있는 상태인지 확인하는 주기입니다.
* idleMaxAgeInMinutes : connection이 유휴상태로 놓어지는 최대시간입니다. 이 시간 이후, connection은 소멸됩니다.
* maxConnections : 최대 connection의 갯수입니다.
* minConnections : 최소 connection의 갯수입니다.
* acquireIncrement : 한번에 connection을 얻어낼 때, 얻어내는 숫자입니다.

application의 최대 성능을 높이기 위해서는 `maxConnections`와 `minConnections`의 숫자를 일치시키는 것이 강력 권장합니다. `HikariCP`를 이용하는 경우 DataSource를 변경하면 다음과 같이 `@Configuration`의 변경이 필요합니다.

```java
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
```

`DataSourc`를 변경한 이후에도 기존 코드에 아무런 변경이 없이 적용이 가능합니다. 이는 `DI`에 의한 `IoC`의 대표적인 한 예가 될 수 있습니다.

## Repository Pattern

`Spring`에서 Logic의 구성은 일반적으로 `Repository Pattern`을 따를 것을 권고하고 있습니다. `Repository Pattern`은 BL(Business Logic)과 데이터의 저장(Repository)에 대한 Layer를 명확히 구분시키는 pattern 입니다. 일반적인 구성은 다음과 같이 만들어집니다.

![](https://i-msdn.sec.s-msft.com/dynimg/IC340233.png)

데이터의 검색/저장 영역과 그 데이터를 이용하는 BL을 철저하게 구분합니다. 이는 `DAO Pattern`과는 구분 됩니다. `DAO Pattern`의 경우, 데이터에 Access를 어떤 Layer에서나 접근이 가능한 차이를 가지고 있습니다.

그래서, Spring으로 구성된 프로젝트를 보면 다음과 같은 package들로 구성되는 경우가 많습니다.

```cmd
.
├── config
├── controller
├── entity
├── repository
└── service
```

이제 몇가지 BL을 가지고 Spring에서의 Service를 구현해보도록 하겠습니다.

## Service

이제 BookStore의 BL을 정의해보도록 하겠습니다.

* user는 book을 빌리거나 반납할 수 있다.
* user가 book을 빌리면, point가 10점씩 쌓인다.
* point가 100점이 넘어가면 LEVEL이 READER(1)가 된다.
* point가 300점이 넘어가면 MVP(2)가 된다.
* point가 100점 이하인 경우, 일반유저(NORMAL)이다.
* 전체 book을 list up 할 수 있으며, 대출이 가능한 책 우선으로 Sort된다.
* book은 대출 가능, 대출중, 분실의 3가지의 상태를 갖는다.
* user는 자신이 지금까지 빌린 book들의 기록(대출,반납)을 최신 순으로 볼 수 있다.
* user의 RENT/RETURN은 모두 History가 남는다.

매우 간단한 BL입니다. 그리고, 이 BL에 대한 Action 주체를 기준으로 다음과 같이 명명한 서비스들을 구상할 수 있습니다.

* UserService: User가 Action의 주체가 되는 서비스입니다.
* BookService: Book이 Action의 주체가 되는 서비스입니다.

서비스의 명명법은 일반적으로 영문법을 따르게 되며 다음과 같은 영어 문장과 같은 구성을 갖는것이 좋습니다.

```java
userService.listupRentHistories();
userService.rentBook(Book book);
userService.returnBook(Book book);
bookService.listup();
```

이에 대한 서비스는 다음과 같이 설계할 수 있습니다.

```java
public interface UserService {
    boolean rent(int userId, int bookId);
    boolean returnBook(int userId, int bookId);
    List<User> listup();
    List<History> getHistories(int userId);
}

public interface BookService {
    public List<Book> listup();
}
```

이와 같이 interface는 단순히 객체에 대한 프로그래밍적 요소로만 사용되는 것이 아닌, 프로그램의 in/out에 대한 설계로서 사용이 가능합니다. 우리가 어떠한 application을 작성을 할때, input/output에 대한 정의를 명확히 할 수 있는 경우, interface를 이용해서 코드를 명확히 구성하는 것이 가능합니다. 이제 interface를 기반으로 서비스를 구현해보면 다음과 같습니다.

```java
@Service
public class UserServiceImpl implements UserService {
    private UserDao userDao;
    private BookDao bookDao;
    private HistoryDao historyDao;

    @Autowired
    public UserServiceImpl(UserDao userDao, BookDao bookDao, HistoryDao historyDao) {
        this.userDao = userDao;
        this.bookDao = bookDao;
        this.historyDao = historyDao;
    }

    @Override
    public boolean rent(int userId, int bookId) {
        User user = userDao.getById(userId);
        Book book = bookDao.getById(bookId);
        if (book.getBookStatus() != BookStatus.CanRent || book.getRentUserId() != null) {
            return false;
        }

        user.setPoint(user.getPoint() + 10);
        if (user.getPoint() >= 100 && user.getPoint() < 300) {
            user.setLevel(1);
        } else if (user.getPoint() >= 300) {
            user.setLevel(2);
        } else {
            user.setLevel(0);
        }
        book.setRentUserId(userId);
        book.setBookStatus(BookStatus.RentNow);

        userDao.update(user);
        bookDao.update(book);

        //NOTE: History 기록
        History history = new History();
        history.setDate(new Date());
        history.setActionType(HistoryActionType.RENT_BOOK);
        history.setUserId(userId);
        history.setBookId(bookId);
        historyDao.add(history);

        return true;
    }
  .....
}
```

이제 구현된 서비스를 `@Service`로 등록됩니다. 이제 `@Configuration`에 등록하는 코드를 확인해보겠습니다.

```java
@Configuration
@ComponentScan(basePackages = {
    "com.xyzlast.bookstore.dao",
    "com.xyzlast.bookstore.service"
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

    @Bean
    public JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dataSource());
    }
}
```

이제 정상적으로 등록되었는지 확인해보도록 하겠습니다.

```java
@SuppressWarnings("unused")
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = BookStoreConfig.class)
public class UserServiceImplTest {
    @Autowired
    private ApplicationContext applicationContext;

    @Before
    public void setUp() throws Exception {
        UserService userService = applicationContext.getBean(UserService.class);
        assertThat(userService).isNotNull();
    }

    @Test
    public void doTest01() {
        ...
    }
}
```

## Service Code의 테스트 방법

`@Service` 객체들을 모두 가져다 사용이 가능한 것을 볼 수 있습니다. 이제 우리는 `Repository Pattern`으로 개발할 준비가 모두 마쳐진 상태입니다. 그렇다면 이와 같은 코드를 어떻게 테스트를 할 수 있을까요?

`rent` 코드가 정상적으로 동작하는 것을 확인하기 위해서는 다음 상황을 확인할 수 있어야지 됩니다.

* `Book`의 상태에 따라 대여가 가능하거나 불가능해야지 됩니다.
* `User`의 Point 점수에 따라 Level이 변경이 될 수 있어야지 됩니다.
* `History`의 추가가 되어야지 됩니다.

위 3가지를 확인하기 위해서는 `Repository`에 올바른 데이터가 입력되어 있어야지 됩니다. 그리고, 테스트가 정상적으로 동작할 수 있도록 데이터를 준비하더라도 테스트가 반복적으로 실행될 수는 없는 구조입니다. `User`의 상태가 변경되기 때문이지요. 그렇다면 이런 문제를 회피하면서 테스트를 진행하기 위해서는 어떻게 해야지 될까요?

여기서 한번 우리가 기본적으로 알고 있는 `OOP`에 대해서 다시 생각해볼 필요가 있습니다. 우리의 테스트 코드는 테스트하고자 하는 객체에 대한 테스트를 진행해야지 됩니다. `Service` 테스트 코드가 굳이 `Repository`에 대한 테스트를 진행할 필요가 없다는 것입니다. 그렇다면 `Repository`를 우리가 원하는 객체만을 반환하게 해줘서 테스트를 진행하면 이 문제를 해결할 수 있습니다.

다시한번 위 3가지의 테스트 조건이 어떻게 통과될 수 있는지 확인해보도록 하겠습니다.

|상황|해결방안|
|---|---|
|`Book`의 상태에 따라 대여가 가능하거나 불가능해야지 됩니다.|bookDao.getById에서 원하는 상태를 갖는 `Book`을 얻어낼 수 있으면 가능합니다.|
|`User`의 Point 점수에 따라 Level이 변경이 될 수 있어야지 됩니다.|`User.point`, `User.level`이 테스트 하고자 하는 상태를 갖는 `User`를 얻어낼 수 있으면 가능합니다.|
|`History`의 추가가 되어야지 됩니다.|`historyDao.add`가 호출되어야지 됩니다.|

이렇게 설정된 값을 반환하는 테스트 객체를 이용하는 테스트를 `mockTest`라고 합니다. `mockTest`는 매우 중요합니다. 모든 `application`은 `Dependency`를 갖게 되는데 이 `Dependency`의 영향을 받지 않는 테스트를 구성하지 않으면, 좋은 테스트 코드를 구현할 수 없습니다. `mockTest`를 구현하기 위해 자주 사용되는 library는 `mokito`입니다.

```groovy
testCompile group: 'org.mockito', name: 'mockito-all', version: '1.10.19'
```

다음은 구성된 테스트 코드입니다.

```java
import static org.assertj.core.api.Assertions.*;
import static org.mockito.Mockito.*;

public class MockUserServiceImplTest {
    @Test
    public void BOOK상태가_정상적이지_않은_경우_테스트() throws Exception {
        Book book = new Book();
        //대여되어 있는 상태
        book.setBookStatus(BookStatus.RentNow);

        BookDao bookDao = mock(BookDao.class);
        when(bookDao.getById(any())).thenReturn(book);
        UserDao userDao = mock(UserDao.class);
        HistoryDao historyDao = mock(HistoryDao.class);

        UserServiceImpl userService = new UserServiceImpl(userDao, bookDao, historyDao);
        assertThat(userService.rent(0, 0)).isFalse();
    }

    @Test
    public void BOOK의_RENTUSERID가_NULL이_아닌경우_테스트() throws Exception {
        int targetUserId = 0;
        Book book = new Book();
        book.setBookStatus(BookStatus.CanRent);
        book.setRentUserId(0);

        BookDao bookDao = mock(BookDao.class);
        when(bookDao.getById(any())).thenReturn(book);
        UserDao userDao = mock(UserDao.class);
        HistoryDao historyDao = mock(HistoryDao.class);

        UserServiceImpl userService = new UserServiceImpl(userDao, bookDao, historyDao);
        assertThat(userService.rent(targetUserId, 0)).isFalse();
    }

    @Test
    public void 정상적인_RENT가_되는_경우_사용자Level이_READER가_되는_경우() throws Exception {
        //User.point = 90, level이 1로 변경되는 것 확인
        testRentWithUserCondition(90, 0, 1);
        //User.point = 290, level이 2로 변경되는 것 확인
        testRentWithUserCondition(290, 1, 2);
        //User.point = 50, level이 0으로 유지되는 것 확인
        testRentWithUserCondition(50, 0, 0);
    }

    private void testRentWithUserCondition(int currentPoint, int currentLevel, int expectLevel) {
        Book book = new Book();
        book.setBookStatus(BookStatus.CanRent);
        book.setRentUserId(null);

        User user = new User();
        user.setPoint(currentPoint);
        user.setLevel(currentLevel);

        BookDao bookDao = mock(BookDao.class);
        when(bookDao.getById(any())).thenReturn(book);
        UserDao userDao = mock(UserDao.class);
        when(userDao.getById(any())).thenReturn(user);
        HistoryDao historyDao = mock(HistoryDao.class);

        UserServiceImpl userService = new UserServiceImpl(userDao, bookDao, historyDao);
        //Rent가 정상적으로 진행되는 것을 확인
        assertThat(userService.rent(0, 0)).isTrue();
        //Rent가 진행된 후, 사용자의 Level이 바뀐것을 확인
        assertThat(user.getLevel()).isEqualTo(expectLevel);

        //book, user가 update되는 코드가 호출되는 것을 확인
        verify(bookDao, times(1)).update(book);
        verify(userDao, times(1)).update(user);
        //History가 추가 되는것을 확인
        verify(historyDao, times(1)).add(any());
    }
}
```

`mockTest`는 의존성을 제거하고 로직을 테스트할 수 있는 좋은 방법입니다. 그렇다고 `Repository`를 직접 이용하는 테스트 코드가 필요없다는 것은 아닙니다. 어떤 항목을 테스트해야될지 개발자들이 확인할 필요가 있습니다.

## Transaction

지금까지 구현된 Service, Repository Layer는 결정적인 문제를 가지고 있습니다. 예를 들어, rentBook action에서 book의 상태를 업데이트 한 후에, DB의 문제나 application의 exception이 발생했다면 어떤 문제가 발생할까요?
지금의 JdbcTemplate은 각 Repository Layer단으로 Connection이 분리 되어 있습니다. 따라서 한쪽에서 Exception이 발생하더라도, 기존 update 사항에 대해서는 DB에 그대로 반영이 되어버립니다. 이건 엄청난 문제를 발생시킵니다. Repository는 우리가 서비스를 만드는 도구이고, 결국은 사용자나 BL의 한개의 action으로 DB의 Transaction이 적용이 되어야지 되는데, 이러한 규칙을 모두 날려버리게 되는 것입니다.

다시 한번 정리하도록 하겠습니다. Service의 method는 BL의 기본 단위가 됩니다. 따라서, Transaction의 단위가 Service의 method 단위가 되어야지 됩니다. 간단히 sudo 코드를 작성한다면 rent method는 다음과 같이 작성되어야지 됩니다.

```java
@Override
public boolean rent(int userId, int bookId) {
    Transaction.begin();
    User user = userDao.get(userId);
    Book book = bookDao.get(bookId);

    user.setPoint(user.getPoint() + 10);
    user.setLevel(getUserLevel(user.getPoint()));
    book.setRentUserId(user.getId());
    book.setStatus(BookStatus.RentNow);

    UserHistory history = new UserHistory();
    history.setUserId(userId);
    history.setBookId(book.getId());
    history.setAction(HistoryActionType.RENT);

    userDao.update(user);
    bookDao.update(book);
    userHistoryDao.add(history);

    Transaction.commit();

    return true;
}
```

Transaction은 매우 골치아픈 개념입니다. 먼저 지금 사용중인 DataSource는 직접적으로 JDBC에 연결되는 Connection입니다. 그런데, 이를 Hibernate의 session 또는 iBatis의 ObjectMapper들을 사용한다면 완전히 다른 Transaction 기술을 사용해야지 됩니다. 지금까지 기술에 종속적이지 않은 `Service`를 작성하고 있는데, 이제 Transaction에 의한 기술 종속적 코드로 변경이 되어야지 되는 상황이 되어버린것입니다. 그래서, 이 경우를 해결하기 위해서 Transaction의 기술들에 대한 interface를 spring은 제안하고 있습니다. 바로 `org.springframework.transaction.PlatformTransactionManager`가 바로 그 interface입니다.

```java
public interface PlatformTransactionManager {
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
    void commit(TransactionStatus status) throws TransactionException;
    void rollback(TransactionStatus status) throws TransactionException;
}
```

Transaction을 얻어내고, Transaction을 commit, rollback하는 간단한 구조로 되어 있습니다. 이러한 기본 구조는 우리가 Dao Layer를 이용해서 Transaction을 사용하는데 충분합니다.
PlatformTransactionManager를 이용한 Transaction 구현을 간단한 sudo code로 구현하면 다음과 같습니다.

```java
public void doSomething() {
    TransactionStatus status = transactionManager.getTransaction(definition);
    try {
        // ..do something
        transactionManager.commit(status);
    }
    catch(Exception ex) {
        transactionManager.rollback(status)
    }
}
```

전에 보던 Template-callback pattern과 동일한 패턴의 코드가 완성됩니다. TransactionManager의 생성자에는 DataSource interface를 구현하고 있기 때문에 Dao Layer에서 사용하는 Connection을 한번에 묶어서 처리가 가능합니다. Spring Transaction을 이용한 코드는 다음과 같이 구현되어야지 됩니다.

먼저 `@Configuration`에 다음과 같이 등록합니다.

```java
@Bean
public PlatformTransactionManager transactionManager() {
    return new DataSourceTransactionManager(dataSource());
}
```

UserService는 다음과 같이 변경되어야지 됩니다.

```java
@Override
public boolean rent(int userId, int bookId) {
    TransactionStatus transaction = transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
        User user = userDao.getById(userId);
        Book book = bookDao.getById(bookId);
        if (book.getBookStatus() != BookStatus.CanRent || book.getRentUserId() != null) {
            return false;
        }

        user.setPoint(user.getPoint() + 10);
        if (user.getPoint() >= 100 && user.getPoint() < 300) {
            user.setLevel(1);
        } else if (user.getPoint() >= 300) {
            user.setLevel(2);
        } else {
            user.setLevel(0);
        }
        book.setRentUserId(userId);
        book.setBookStatus(BookStatus.RentNow);

        userDao.update(user);
        bookDao.update(book);

        //NOTE: History 기록
        History history = new History();
        history.setDate(new Date());
        history.setActionType(HistoryActionType.RENT_BOOK);
        history.setUserId(userId);
        history.setBookId(bookId);
        historyDao.add(history);
        transactionManager.commit(transaction);
    } catch (Exception ex) {
        transactionManager.rollback(transaction);
    }
    return true;
}
```

이제 Transaction이 적용된 코드가 완성되었습니다.

## annotation을 이용한 Transaction의 구현

그런데, 지금까지 구현된 코드에는 한가지 문제가 있습니다. Transaction은 기술적인 영역으로 Service 객체에는 어울리지 않는 내용입니다. Service는 BL의 집합이라는 것을 다시 한번 상기해주시길 바랍니다. BL에 기술적인 요소가 들어가게 되면, 기술적인 요소에 따른 BL의 수정이 가해질 수 있습니다. 따라서, Spring에서는 이를 분리하는 것을 제안하고 있으며, 특히 Transaction에서는 @Transactional annotaion을 이용한 분리를 제안하고 있습니다.

@Transactional은 method, class에 모두 적용 가능한 annotation입니다. @Transactional을 사용하기 위해서는 `@Configuration`이 다음과 같이 변경이 필요합니다.

```java
@Configuration
@ComponentScan(basePackages = {
    "com.xyzlast.bookstore.dao",
    "com.xyzlast.bookstore.service"
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
    public JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dataSource());
    }
    @Bean
    public PlatformTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }
}
```

`@EnableTransactionManagement`에 주의해주시길 바랍니다. 이제 Transaction을 다음과 같이 적용하면 됩니다.

```java
@Override
@Transactional
public boolean rent(int userId, int bookId) {
    User user = userDao.getById(userId);
    Book book = bookDao.getById(bookId);
    if (book.getBookStatus() != BookStatus.CanRent || book.getRentUserId() != null) {
        throw new RuntimeException("BOOK의 상태가 정상적이지 않습니다.");
    }

    user.setPoint(user.getPoint() + 10);
    if (user.getPoint() >= 100 && user.getPoint() < 300) {
        user.setLevel(1);
    } else if (user.getPoint() >= 300) {
        user.setLevel(2);
    } else {
        user.setLevel(0);
    }
    book.setRentUserId(userId);
    book.setBookStatus(BookStatus.RentNow);

    userDao.update(user);
    bookDao.update(book);

    //NOTE: History 기록
    History history = new History();
    history.setDate(new Date());
    history.setActionType(HistoryActionType.RENT_BOOK);
    history.setUserId(userId);
    history.setBookId(bookId);
    historyDao.add(history);

    return true;
}
```

## Summary

이번에는 Spring JDBC를 이용한 DB 접근 방법에 대해서 알아봤습니다. Spring을 사용중이지만, 외부 Library의 도움을 받지 못하는 경우에 매우 좋은 선택입니다. 그리고 DataSource에 대해서도 알아봤습니다. DataSource는 매우 중요한 Library입니다. DB Connection을 관리하고 있기 때문에 이에 대한 세부설정은 전체 솔루션 성능에 큰 영향을 미치게 됩니다. Transaction은 DB를 다룰때 매우 중요한 이슈입니다.

마지막으로 `Repository Pattern`에 대해서 알아봤습니다. `Service`, `Repository`로 만들어지는 구조는 Spring을 사용한 BL Logic을 구성하는 application의 기본 구조이기도합니다. 그리고, `Repository Pattern`을 테스트하는 방법들 중에서 `mockTest`에 대해서도 알아봤습니다.

하나하나가 매우 중요한 개념입니다. 몸으로도 개념으로도 익히는 것이 중요합니다.
