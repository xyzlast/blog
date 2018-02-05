---
title: 05-03.SampleApplication(3)
date: 2018-02-01 12:48:40
tags: study,java,DIP,SOLID
---
# SampleApplication(3) - DIP

지금까지 간단한 코드와 그 코드에 대한 테스트를 행하는 방법에 대해서 알아봤습니다. 그리고 간단한 table의 CRUD를 하는 방법에 대해서 조금 깊게 들어가봤습니다. 또한 저번 시간의 최대 포인트는 테스트입니다. 테스트를 어떻게 작성을 하는지에 대한 논의와 테스트 코드를 직접 사용해보는 시간을 가져봤습니다. 이번 시간에는 드디어 Spring을 이용한 코드에 대해서 논의해보도록 하겠습니다.기존 코드의 가장 큰 문제는 무엇인가요?

* DB connection이 변경되는 경우, compile을 다시 해줘야지 된다.
* DB connection의 open/close가 모든 method에서 반복된다.

크게 보면 이 두가지의 문제가 나오게 됩니다.

Connection은 외부에서 설정할 수 있는 영역입니다. 특히 이런 부분은 config file로 관리가 되는 것이 일반적이고, Enterprise 구성환경에서는 JNDI를 이용한 DB Connection이 제공되는 경우도 많습니다. 따라서, 외부에서 설정이 가능한 것이라고 생각해도 좋습니다. 또한, BookApp은 books table에 CRUD를 하는 것을 목적으로 하는 객체입니다. 객체에 대한 가장 큰 원칙인 단일 책임의 원칙에 의해서, BookApp에서 Connection까지 관리가 되는 것은 영역을 넘어가게 됩니다. 또한, books 이외의 table이 존재할 때, Connection에 대한 코드는 중복될 수 밖에 없는 코드가 됩니다. 따라서 Connection을 제공하는 객체로 따로 분리를 해줍시다.

Connection을 관리, 생성하는 객체이기 때문에 ConnectionFactory라고 명명하고, 객체를 구성합니다. 객체의 코드는 다음과 같습니다.

```java
public class ConnectionFactory {
    private String connectionString;
    private String driverName;
    private String username;
    private String password;

    public Connection getConnection() throws InstantiationException, IllegalAccessException,
                                             ClassNotFoundException, SQLException {
        Class.forName(this.driverName).newInstance();
        return DriverManager.getConnection(this.connectionString, this.username, this.password);
    }

    public String getConnectionString() {
        return connectionString;
    }

    public void setConnectionString(String connectionString) {
        this.connectionString = connectionString;
    }

    public String getDriverName() {
        return driverName;
    }

    public void setDriverName(String driverName) {
        this.driverName = driverName;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

DB의 connection을 관리하는 객체입니다. conneciton만을 관리하기 때문에 `단일책임원칙`에 맞는 코드라고 할 수 있을 것입니다. 여기서 잠시 다른 곳으로 가서 코드를 좀 더 편하게 적을 방법을 찾아봅시다.

```cmd
compileOnly group: 'org.projectlombok', name: 'lombok', version: '1.16.18'
```

를 추가합시다.

이제 추가 한 후에는 다음과 같이 코드를 작성할 수 있습니다.

```java
public class ConnectionFactory {
    @Getter
    @Setter
    private String connectionString;
    @Getter
    @Setter
    private String driverName;
    @Getter
    @Setter
    private String username;
    @Getter
    @Setter
    private String password;

    public Connection getConnection() throws InstantiationException, IllegalAccessException,
                                             ClassNotFoundException, SQLException {
        Class.forName(this.driverName).newInstance();
        return DriverManager.getConnection(this.connectionString, this.username, this.password);
    }
}
```

`@Getter`, `@Setter`라는 annotation으로 원 코드를 대신해서 사용할 수 있습니다. 코드의 가독성을 높일 수 있는 장점을 갖습니다. 개인적으로 추천하는 library입니다.
그리고, 이 객체는 `connectionString`, `driverName`, `username`, `password` 중 하나라도 없다면 실행할 수 없는 객체입니다. 따라서 생성자에 전 값을 다 추가하는 것이 좋습니다. 이는 객체의 사용방법을 명확히 지정할 수 있습니다.

이제 간략화된 코드를 다시 구성합니다.

```java
@AllArgsConstructor
public class ConnectionFactory {
    @Getter
    private final String driverName;
    @Getter
    private final String connectionString;
    @Getter
    private final String username;
    @Getter
    private final String password;

    public Connection getConnection() throws InstantiationException, IllegalAccessException,
        ClassNotFoundException, SQLException {
        Class.forName(this.driverName).newInstance();
        return DriverManager.getConnection(this.connectionString, this.username, this.password);
    }
}
```

그리고, 이에 대한 테스트 코드를 작성합니다.

```java
@Test
public void doConnectionFactoryTest() throws Exception {
    ConnectionFactory connectionFactory = new ConnectionFactory("org.mariadb.jdbc.Driver",
        "jdbc:mysql://127.0.0.1:4306/bookstore", "root", "qwer12#$");
    assertThat(connectionFactory).isNotNull();
    assertThat(connectionFactory.getConnection()).isNotNull();
}
```

를 이용해서 테스트를 진행합니다.

`ConnectionFactory`를 적용한 `BookService`를 만들어보도록 합니다.

```java
public class BookService {

    private final ConnectionFactory connectionFactory;

    public BookService(ConnectionFactory connectionFactory) {
        this.connectionFactory = connectionFactory;
    }

    public void add(Book book) throws InstantiationException, IllegalAccessException, ClassNotFoundException, SQLException {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement("insert books(name, author, publishDate, comment) values(?, ?, ?, ?)");
        st.setString(1, book.getName());
        st.setString(2, book.getAuthor());
        java.sql.Date sqlDate = new java.sql.Date(book.getPublishDate().getTime());
        st.setDate(3, sqlDate);
        st.setString(4, book.getComment());
        st.execute();

        st.close();
        conn.close();
    }
    ......
}
```

이제 테스트 코드 역시 같이 업데이트 시켜야지 됩니다. 우리가 만든 코드를 테스트할 수 있어야지 되니까요.

```java
@Before
public void setUp() {
    ConnectionFactory connectionFactory = new ConnectionFactory("org.mariadb.jdbc.Driver",
        "jdbc:mysql://127.0.0.1:4306/bookstore", "root", "qwer12#$");
    bookService = new BookService(connectionFactory);
}
```

## IoC, DI

지금 우리는 테스트 코드내에서 코드가 실행되고 있습니다. 테스트 코드입장에서 생각해보도록 하겠습니다.

우리는 지금 테스트 코드에서 `BookService`를 실행시키고 있습니다. 그리고 `BookService`에서 사용할 `ConnectionFactory`를 `BookServiceTest`에서 구성해서 사용하고 있습니다.
조금 말을 바꿔보면 `사용하는 Instance에서 사용할 Instance의 구성을 결정하고 있습니다.` 이를 `IoC`라고 합니다.

여기서, Spring의 가장 주요한 기능을 사용할 때가 되었습니다.

Spring은 `IoC`를 지원하는 `light container`입니다.

조금 더 말을 풀어서 표현하면 **Spring은 `IoC`를 `DI(dependency injection)`를 통해 지원하는 `light container`입니다.**
말이 어려워서 예시를 이용해서 이야기한다면 다음과 같습니다.

* BookService은 ConnectionFactory에 의존한다.
* BookServiceTest는 BookService에 의존한다.
* BookServiceTest는 BookService의 ConnectionFactory를 어떻게 바꾸어서 사용할지까지 결정이 가능하다.

이는 `BookServiceTest`는 `BookService`를 사용하고 있지만, `BookService`가 의존하고 있는 `ConnectionFactory`를 자신이 결정해서 사용하고 있습니다. 이때, 의존하는 객체를 변경하는 행위를 `DI`라고 합니다. 따라서 이 경우를 조금더 어렵게 말하면 다음과 같습니다.

**BookServiceTest는 BookService를 DI를 통해 ConnectionFactory를 주입(inject)하여 사용하는 코드입니다.**

지금 이야기드린 `IoC`와 `DI`는 Spring에서 굉장히 중요한 개념입니다. Spring의 본질이라고 할 수 있습니다. 단순히 `WebMvc`를 제공하고 있는 것이 Spring이 아닙니다.
좀 더 깊게 들어가보도록 하겠습니다.

### IoC(Inversion of Control)

IoC는 전통적인 방식에 반대되는 흐름으로 코드가 진행되는 것을 말하는 일반적인 용어입니다. Dependency Injection과 많이 혼동되는데 다른 개념입니다. 오히려 Dependency Inversion Principle이 IoC의 한 형태입니다.

라이브러리와 프레임워크의 관계는 IoC를 설명하는 단골 예입니다. 라이브러리를 사용할 때는 내 코드가 라이브러리 코드(외부 코드)를 호출하지만 프레임워크를 사용할 때는 프레임워크(외부 코드)가 내 코드를 호출합니다. 상호작용(interactive) 프로그래밍 모델과 반응형(reactive) 프로그래밍 모델도 하나의 예입니다.

IoC는 헐리웃에서 오디션이 끝난 후 오디션 참가자에게 하는 말때문에 `Hollywood Principle`로 불리기도 합니다

### DIP(Depencency Inversion Principal)

`IoC`는 `DIP` 원칙을 구현하는 대표적인 방법입니다. 이는 의존관계를 갖는 모듈 인스턴스의 구성이 추상화에 의존하는 것을 뜻합니다.

좀 더 풀어서 설명하면 추상성이 높고 안정적인 의존 주체 클래스는 구체적이고 불안정한 의존 대상 클래스에 의존해서는 안된다는 원칙으로서, 일반적으로 객체지향의 인터페이스를 통해서 이 원칙을 준수할 수 있게 된다. (상대적으로 고수준인) 클라이언트는 저수준의 클래스에서 추상화한 인터페이스만을 바라보기 때문에, 이 인터페이스를 구현한 클래스는 클라이언트에 어떤 변경도 없이 얼마든지 나중에 교체될 수 있다는 원칙입니다.

말이 계속해서 어려워집니다. 사전적 정의는 더 어렵습니다. 참고로 `DIP`는 객체지향 프로그래밍의 5대 원칙중 하나입니다.
> SOLID - SRP(단일 책임원칙), OCP(개방-폐쇄 원칙), LSP(리스코치환원칙), ISP(인터페이스분리원칙), DIP(의존성 역전 원칙)

> High-level modules should not depend on low-level modules. Both should depend on abstractions.
> Abstractions should not depend on details. Details should depend on abstractions.

우리가 개발하는 의존 주체 모듈 인스턴스(BookStore)가 의존 대상 모듈 인스턴스(ConnectionFactory)를 직접 생성하는 것을 전통적 흐름이라고 하면 이 전통적 흐름에서는 의존 대상 코드의 형식이 의존 주체 코드에 의해 결정되기 때문에 의존 대상 코드의 다형성이 동작하기 어렵습니다. 반면 `DIP` 원칙이 적용되면 구체적인 의존 관계가 추상화에 의해 사용시에 결정되기 때문에 다형성을 적극적으로 활용할 수 있으며 모듈의 재사용성이 높아집니다.

기본적으로 `DIP`를 따르기 위해서는 의존 주체 모듈 인스턴스와 의존 대상 모듈 인스턴스간의 직접적인 연결이 아닌 중간단계의 `interface`를 통해서 구현하게 됩니다.

말이 어렵습니다. 코드로서 보면서 이해하는 것이 더 쉽습니다. `BookService`에 `DIP`가 적용되지 않을때, 다음과 같이 사용되게 됩니다.

```java
class BookService {
    private ConnectionFactory connectionFactory;

    public BookService() {
        ConnectionFactory connectionFactory = new ConnectionFactory("org.mariadb.jdbc.Driver",
            "jdbc:mysql://127.0.0.1:4306/bookstore", "root", "qwer12#$");
    }
}
```

위 코드에서 `DIP`가 적용되지 않은 `BookService`는 `ConnectionFactory`에 의존적이게 됩니다. 만약에 `ConnectionFactory`가 변경이 되게 되면 이는 `BookService`에 영향을 미치게 됩니다. 이는 세부사항(detail)이 구현(Abstractions)를 변경시키는 영향을 미치게 됩니다. `DIP`를 적용하기 위한 방법으로 `DI(Dependency Injection)`와 `Service Locator` 방법이 있습니다.

### DI(Dependency Injection)

많은 사람들이 `DI`와 `DIP`를 혼동합니다. `DI`는 `DIP`를 구현하는 기법 중 하나입니다. 의존 주체 모듈의 노출된 멤버를 통해 의존 대상 모듈 인스턴스가 제공됩니다. Dependency Injection은 다시 세부적으로 구분될 수 있습니다.

#### Constructor Injection

Dependency Injection 중 가장 많이 사용되는 방법입니다. 의존 주체 모듈의 생성자 매개변수로 의존 대상 모듈 인스턴스가 주입됩니다. 의존 주체 인스턴스가 생성됨과 동시에 의존성이 해결되기 때문에 일단 의존 주체 인스턴스가 생성되면 의존성 해결 여부를 검사할 필요가 없습니다. 정적 언어의 경우 의존성이 해결되지 않으면 의존 주체 인스턴스를 생성할 수 없기 때문에 모듈간 의존 관계가 문법적으로 명확하게 드러나며 의존성이 누락될 위험이 컴파일러에 의해 차단됩니다.

#### Property Injection

공용으로 노출된 쓰기 가능한 속성을 통해 의존성을 주입하는 방법입니다. Setter Injection이라고도 부릅니다. 속성이 외부에 노출된 정보를 의미하는지 의존 대상 모듈을 의미하는지 별도의 설명이 없으면 구분하기 어렵습니다. 속성 설정은 의존 주체 인스턴스가 생성된 후에 진행되기 때문에 의존 주체 인스턴스는 필요한 의존성이 해결되지 않은 상태일 수 있습니다.

#### DI를 이용한 코드 구성

`Constructor Injection`과 `Property Injection` 중에서, 예전에는 `Property Injection`을 권장했습니다. 지금은 압도적으로 `Constructor Injection`을 선호합니다.
`Constructor Injection`을 이용해서 `BookService`를 구성하면 다음과 같이 구성이 가능합니다.

```java
public interface ConnectionFactory {
    Connnection getConnection();
}

public class BookService {
    private ConnectionFactory connectionFactory;
    public BookService(ConnectionFactory connectionFactory) {
        this.connectionFactory = connectionFactory;
    }
}

public class MySqlConnectionFactoryImpl implements ConnectionFactory {
    .....
}

public class OracleSqlConnectionFactoryImpl implements ConnectionFactory {
    .....
}

class App {
    public static void main(String[] args) {
        BookService bookServiceWithMySql = new BookService(new MySqlConnectionFactoryImpl());
        BookService bookServiceWithOracle = new BookService(new OracleMySqlConnectionFactoryImpl());
    }
}
```

위와 같이 구성하게 되면, `BookService`는 의존성이 `interface`만 연결이 되게 되며 이는 결합성이 매우 낮은 코드로 사용할 수 있게 됩니다.

### Service Locator

Service Locator는 Dependency Injection과는 다른 Dependency Inversion Principle 적용법입니다. Dependency Injection에서 의존 주체 모듈은 의존성을 외부의 주입에 의해 수동적으로 해결하는 반면 Service Locator를 사용하는 의존 주체 모듈은 의존성이 필요한 시점에 능동적으로 의존성 해결을 Service Locator에 요청합니다.

```java
public interface ConnectionFactory {
    Connnection getConnection();
}

public class BookService {
    public Connection getConnection() {
        IServiceLocator serviceLocator = IServiceLocator.current();
        return serviceLocator.resolve(ConnectionFactory.class).getConnection();
    }
}

class App {
    public static void main(String[] args) {
        IServiceLocator serviceLocator = IServiceLocator.current();
        serviceLocator.add(new MySqlConnectionFactoryImpl());
        BookService bookServiceWithMySql = new BookService();
    }
}
```

Service Locator는 의존 주체 모듈이 `Service Locator`를 통해 능동적으로 자신이 사용할 객체들을 찾아서 사용합니다.

Dependency Injection과의 차이점은 다음과 같습니다.

* `Service Locator`가 적용된 의존 주체 모듈은 외부 모듈 의존관계가 노출되지 않습니다. 위 코드에서는 `ConnectionFactory`를 사용하는 것을 알아낼 수 없습니다.
* 의존 주체 모듈은 `Service Locator`를 직접 사용하기 때문에 `Service Locator`의 존재를 알고 있으며 필수적으로 의존합니다.
* `Service Locator`가 정적인(static) 코드를 통해 제공되기 때문에 병렬(parallel) 테스팅이 어렵습니다.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Spring을 이용한 IoC/DI

Spring에서는 `IoC`를 제공하기 위해서 `DI`를 이용합니다.
Spring을 이용하는 방법을 알아보도록 하겠습니다. 먼저 Spring의 Core만을 추가해서 진행해보도록 하겠습니다.

```cmd
compile group: 'org.springframework', name: 'spring-context', version: '5.0.2.RELEASE'
testCompile group: 'org.springframework', name: 'spring-test', version: '5.0.2.RELEASE'
```

Spring에서는 `IoC`를 위한 `DI` 구성을 `Configuration`이라고 합니다. 이는 `xml`과 `class`로 처리가 가능합니다. 먼저 옛날부터 사용하고 있던 `xml`을 우선 알아보도록 하겠습니다.

### xml

`xml`파일을 다음과 같이 선언합니다.

```xml
<?xml version = "1.0" encoding = "UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">
<beans>
    <bean id="connectionFactory" class="com.xyzlast.bookstore.ConnectionFactory">
        <constructor-arg index="0" type="String" value="org.mariadb.jdbc.Driver"/>
        <constructor-arg index="1" type="String" value="jdbc:mysql://127.0.0.1:4306/bookstore"/>
        <constructor-arg index="2" type="String" value="root"/>
        <constructor-arg index="3" type="String" value="qwer12#$"/>
    </bean>
    <bean id="bookService" class="com.xyzlast.bookstore.BookService">
        <constructor-arg index="0" ref="connectionFactory"/>
    </bean>
</beans>
```

Spring은 `ApplicationContext`라는 개념을 가지고 있습니다. `ApplicationContext`는 이름 그대로 Application에서 사용되는 내용들을 정의하는 역활을 담당합니다. 전에 설명했던 `IoC`를 위한 `DI`구성을 정의합니다. 그리고, 정의된 파일을 이용해서 객체를 사용하는 방법은 다음과 같습니다.

```java
public class SpringBookServiceTest {
    private ApplicationContext context;

    @Before
    public void setUp() {
        context = new GenericXmlApplicationContext("/application-context.xml");
        assertThat(context).isNotNull();
    }

    @Test
    public void checkBeans() {
        ConnectionFactory connectionFactory01 = (ConnectionFactory) context.getBean("connectionFactory");
        ConnectionFactory connectionFactory02 = (ConnectionFactory) context.getBean("connectionFactory");
        assertThat(connectionFactory01).isEqualTo(connectionFactory02);

        BookService bookService = context.getBean(BookService.class);
        assertThat(bookService).isNotNull();
    }
}
```

`checkBeans` 내부를 보면 `new`를 이용하지 않고, 모든 객체를 사용할 수 있는 것을 볼 수 있습니다. 특이한점이 `ConnectionFactory`입니다. 두번 객체를 얻어내도 같은 객체로 얻어지는 것을 볼 수 있습니다. 여기서 `BookService`를 보면 `ref` 키워드를 이용해서 기존에 만들어진 `ConnectionFactory`를 `BookService`에 주입할 수 있는것을 볼 수 있습니다.

### @Configuration을 이용

Spring에서는 `xml` 파일이 아닌, `@Configuration`을 이용하는 것을 좀 더 권장하는 편입니다. `@Configuration`으로 구성하는 방법은 다음과 같습니다.

```java
@Configuration
public class BookServiceConfiguration {
    @Bean
    public ConnectionFactory connectionFactory() {
        ConnectionFactory connectionFactory = new ConnectionFactory("org.mariadb.jdbc.Driver",
            "jdbc:mysql://127.0.0.1:4306/bookstore", "root", "qwer12#$");
        return  connectionFactory;
    }
    @Bean
    public BookService bookService() {
        return new BookService(connectionFactory());
    }
}
```

구성한 `@Configuration`은 다음과 같이 활용 가능합니다.

```java
@Test
public void checkBeansWithConfiguration() {
    ApplicationContext context = new AnnotationConfigApplicationContext(BookServiceConfiguration.class);
    assertThat(context).isNotNull();

    ConnectionFactory connectionFactory01 = (ConnectionFactory) context.getBean("connectionFactory");
    ConnectionFactory connectionFactory02 = (ConnectionFactory) context.getBean("connectionFactory");
    assertThat(connectionFactory01).isEqualTo(connectionFactory02);

    BookService bookService = context.getBean(BookService.class);
    assertThat(bookService).isNotNull();
}
```

### xml vs @Configuration

이 방법들중에서 어떤 방법이 더 좋을지는 개발자들의 취향에 따라서 다르게 할 수도 있습니다. 그런데, 지금 Spring에서 제공되는 Sample이나 강의 대부분이 이젠 `@Configuration`을 사용하고 있습니다.

예전 코드에 대한 유지보수의 경우에는 'xml'을 사용하는 것이 좋겠지만, 신규 Application이나 `SpringBoot`를 이용하는 Application은 `@Configuration`을 이용하는것이 좋습니다.

### ApplicationContext

Spring을 사용한 객체의 사용의 장점은 크게 3가지로 볼 수 있습니다.

1. 정해진 규칙에 따른 객체의 선언
2. 정의된 객체의 dependency 파악이 가능
3. 테스트의 편의성

먼저 정해진 규칙에 따른 객체의 선언은 팀단위의 개발자들에게 일정한 개발 패턴을 만들어줍니다. 정해진 개발 패턴은 정형화된 코드를 만들고, 서로간에 코드의 공유가 원활하게 할 수 있습니다. 그리고, xml을 통한 객체의 의존성 관리는 객체들이 어떠한 구조를 가지고 있는지를 파악하는데 도움을 줍니다. 마지막으로 테스트의 편의성을 들 수 있습니다. spring은 테스트 코드를 작성하는데, 지금까지 보던 코드보다 더욱 깔끔하고 쉬운 테스트 패턴을 제공하고 있습니다. Spring을 사용한 테스트 코드를 다시 한번 알아보도록 하겠습니다.

Spring은 spring-test를 통해 테스트를 구성할 수 있습니다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = BookServiceConfiguration.class)
public class SpringBookServiceTest2 {

    @Autowired
    private ApplicationContext context;

    @Before
    public void setUp() {
        assertThat(context).isNotNull();
    }

    @Test
    public void checkBeansWithConfiguration() {
        ConnectionFactory connectionFactory01 = (ConnectionFactory) context.getBean("connectionFactory");
        ConnectionFactory connectionFactory02 = (ConnectionFactory) context.getBean("connectionFactory");
        assertThat(connectionFactory01).isEqualTo(connectionFactory02);

        BookService bookService = context.getBean(BookService.class);
        assertThat(bookService).isNotNull();
    }
}
```

`@RunWith`와 `@ContextConfiguration`을 통해 로드된 `application-context`를 재사용이 됩니다. 여러개의 테스트코드를 작성할때, 속도를 위해 꼭 사용되어야지 되는 코드입니다.

## 객체의 생명주기

spring에서의 객체의 생명 주기는 기본적으로 한번 사용된 객체를 재사용합니다. 테스트 코드를 통해서 이를 확인해보도록 하겠습니다.
구성된 코드에 System.out.println 을 이용해서 bookApp 객체의 instance id를 확인해보도록 하겠습니다.

```cmd
Dec 23, 2017 1:23:48 PM org.springframework.context.support.AbstractApplicationContext prepareRefresh
INFO: Refreshing org.springframework.context.support.GenericApplicationContext@59f99ea: startup date [Sat Dec 23 13:23:48 KST 2017]; root of context hierarchy
com.xyzlast.bookstore.ConnectionFactory@4802796d
com.xyzlast.bookstore.ConnectionFactory@4802796d
com.xyzlast.bookstore.ConnectionFactory@4802796d
Dec 23, 2017 1:23:49 PM org.springframework.context.support.AbstractApplicationContext doClose
```

보시면 ConnectionFactory의 모든 객체 Id가 동일함을 알 수 있습니다. ApplictionContext에 저장된 객체를 얻을 때, 새로 생성하는 것이 아닌 기존의 객체를 계속해서 사용하는 것을 알 수 있습니다.
왜 모든 객체들을 기본적으로 재사용하게 될까요? 기본적으로 spring은 enterprise development framework입니다. enterprise급의 대규모 시스템에서는 객체의 생성/삭제가 많은 부담을 주게 됩니다. 이에 대한 해결 방법으로 spring은 application context가 로드 될 때, 기본 객체들을 모두 생성, 로드 하는 것을 기본으로 하고 있습니다. 물론, 다른 방법역시 가능합니다.

Spring에서의 객체의 생명주기는 scope로 불리고, 다음 4가지로 관리가 가능합니다.

1.singleton
2.prototype
3.session
4.request

singleton은 default 값입니다. scope를 따로 설정하지 않으면 모두 singleton으로 동작합니다. 이는 객체를 static object와 동일하게 사용하게 됩니다.

prototype은 일반적으로 저희가 사용하던 객체의 생성방법과 동일합니다. new 를 통해서 객체를 생성하고, property 값을 모두 설정시킨 후, 그 객체를 넘겨주게 됩니다.

session과 request는 web programming에서 사용되는 scope입니다. 새로운 session이 생성될 때, 새로운 request가 생성이 될때 사용될 수 있는 scope 입니다.

크게는 singleton과 prototype을 이용하면 대부분의 객체 생명주기 관리는 가능합니다. 생명주기선언은 다음과 같습니다.

```java
@Bean
@Scope(value = BeanDefinition.SCOPE_SINGLETON)
public ConnectionFactory connectionFactoryWithSingleton() {
    ConnectionFactory connectionFactory = new ConnectionFactory("org.mariadb.jdbc.Driver",
        "jdbc:mysql://127.0.0.1:4306/bookstore", "root", "qwer12#$");
    return  connectionFactory;
}

@Bean
@Scope(value = BeanDefinition.SCOPE_PROTOTYPE)
public ConnectionFactory connectionFactoryWithProxy() {
    ConnectionFactory connectionFactory = new ConnectionFactory("org.mariadb.jdbc.Driver",
        "jdbc:mysql://127.0.0.1:4306/bookstore", "root", "qwer12#$");
    return  connectionFactory;
}
```

이제 생성된 코드에 대한 테스트는 다음과 같이 진행할 수 있습니다.

```java
@Test
public void checkBeanLifeCycle() {
    ConnectionFactory singleton01 = (ConnectionFactory) context.getBean("connectionFactoryWithSingleton");
    ConnectionFactory singleton02 = (ConnectionFactory) context.getBean("connectionFactoryWithSingleton");
    assertThat(singleton01.hashCode()).isEqualTo(singleton02.hashCode());

    ConnectionFactory proxy01 = (ConnectionFactory) context.getBean("connectionFactoryWithProxy");
    ConnectionFactory proxy02 = (ConnectionFactory) context.getBean("connectionFactoryWithProxy");
    assertThat(proxy01.hashCode()).isNotEqualTo(proxy02.hashCode());
}
```

이 코드를 확인해보시면 아시겠지만 Factory Pattern에서의 Factory와 동일한 기능을 가지게 되는 것을 알 수 있는데요. **ApplicationContext는 bean의 Map과 Factory를 지원한다.** 라고 할 수 있습니다.
spring을 사용하게 되면, 객체들을 new로 새롭게 할당하는 일들이 얼마 없습니다. 아니 new를 안쓰고 만들 수 있어야지 됩니다. 모든 객체 bean들은 spring을 통해서 관리가 되고, bean들간의 데이터 교환을 위한 POJO(Plain Old Java Object)들만 new로 생성되어서 데이터 교환이 되는 것이 일반적인 패턴입니다.

## Summary

이번 장에서는 Spring을 이용한 객체의 관리를 위주로 Simple Application을 작성해봤습니다. 다음 개념들은 반드시 다시 정리해보시는 것이 좋습니다.

* ApplicationContext : Spring에서 제공하는 bean의 Map/ObjectFactory
* property, constructor, init-method를 이용한 객체 초기화 방법
* IoC, DIP, DI
