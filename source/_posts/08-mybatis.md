---
title: 08.myBatis
date: 2018-02-21 23:18:14
tags: java, spring, mybatis
---

# myBatis

지금까지는 Spring에서 제공하는 `JdbcTemplate`를 이용하는 방법을 알아봤습니다. 단점으로 볼 수 있는 것이, 직접적인 query들을 계속해서 코드안에 넣어야지 되는 단점이 있습니다. 그리고, 실제적인 코드와 SQL간의 분리가 되지 않는다는 단점을 가지고 있습니다. 국내에서 DB 접근에 가장 많이 사용하고 있는 기술인 `myBatis`에 대해서 알아보도록 하겠습니다.

myBatis는 `Repository Pattern`을 충실히 지킬 수 있으며, DB의 `StoreProcedure`, `Function`과 같은 legacy sql을 직접적으로 쉽게 사용할 수 있는 장점을 가지고 있습니다. 이를 통해, JDBC를 이용한 반복적인 코드를 획기적으로 줄이는 것을 목표로 가지고 있으며, 개발자들에게 게을러질 수 있는 권리를 보장하는 것이 목표입니다.

## myBatis의 구조

SqlSessionFactory, SqlSession이라는 두개의 객체를 가지고 있습니다. 기본적으로 mybatis.cfg.xml 파일을 이용해서 DB에 대한 연결 설정과 각각의 mapper를 구성하는 설정으로 구성되어 있습니다.

myBatis에서 DB에 대한 연결을 구성하는 방법은 다음과 같습니다.

1. SqlSessionFactory 구성
2. SqlSessionFactory를 통한 SqlSession을 구성
3. SqlSession을 통해 mapper를 구성

기본적인 `SqlSessionFactory`를 정의합니다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="org.mariadb.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://127.0.0.1:4306/bookstore"/>
                <property name="username" value="root"/>
                <property name="password" value="qwer12#$"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="mappers/bookDao.mapper.xml"/>
    </mappers>
</configuration>
```

그리고, 각 Repository의 method에 따른 SQL문을 xml에 정의합니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xyzlast.bookstore.repository.BookDao">
    <resultMap id="BookResult" type="com.xyzlast.bookstore.entity.Book">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="author" column="author"/>
        <result property="publishDate" column="publishDate"/>
        <result property="rentUserId" column="rentUserId"/>
        <result property="bookStatus" column="status" typeHandler="org.apache.ibatis.type.EnumOrdinalTypeHandler"/>
    </resultMap>
    <select id="getById" parameterType="int" resultMap="BookResult">
        SELECT * FROM books WHERE id = #{bookId}
    </select>
    <select id="getAll" resultMap="BookResult">
        SELECT * FROM books
    </select>
    <select id="countAll" resultType="int">
        SELECT count(*) FROM books
    </select>
    <insert id="add" parameterType="com.xyzlast.bookstore.entity.Book" useGeneratedKeys="true" keyColumn="id">
        INSERT INTO books(name, author, publishDate, comment, status, rentUserId)
        VALUES(#{name}, #{author}, #{publishDate}, #{comment}, #{bookStatus.value}, #{rentUserId})
    </insert>
    <delete id="delete" parameterType="int">
        DELETE FROM books WHERE id = #{bookId}
    </delete>
    <delete id="deleteAll">
        DELETE FROM books
    </delete>
    <update id="update" parameterType="com.xyzlast.bookstore.entity.Book">
        UPDATE books SET
        name = #{name}, author = #{author}, publishDate=#{publishDate}, status=#{bookStatus.value}, rentUserId=#{rentUserId}, comment = #{comment}
        WHERE id = #{id}
    </update>
</mapper>
```

xml이 모두 준비된 후, 다음과 같은 code pattern으로 처리됩니다.

```java
private SqlSessionFactory sqlSessionFactory;
SqlSession sqlSession;
private BookDao bookDao;

@Before
public void setUp() throws Exception {
    InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
    sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
    sqlSession = sqlSessionFactory.openSession();
    bookDao = sqlSession.getMapper(BookDao.class);
}
@After
public void tearDown() {
    if (sqlSession != null) {
        sqlSession.close();
    }
}
@Test
public void countAll() throws Exception {
    int bookCount = bookDao.countAll();
    assertThat(bookCount).isGreaterThan(0);
}
```

SqlSessionFactory를 통해, SqlSession을 얻어내고 update/delete/insert에 대한 commit을 직접 행하도록 코드를 작성했습니다. 또한 모든 method가 완료되면 반드시 Session을 닫아줘야지만 됩니다.

## Spring-myBatis의 이용

Spring과 myBatis를 연결하기 위해서는 추가 Library가 필요합니다.

```groovy
compile group: 'org.mybatis', name: 'mybatis-spring', version: '1.3.1'
```

Spring에서 제공하는 `@Transaction`을 사용하기 위해서, `SqlSessionTemplate`를 이용합니다. 이는 `@Transaction`으로 지정된 method내 같은 `Session`으로 `@Transaction`이 지정될 수 있도록 만들어줍니다.
이를 지정하기 위한 `@Configruation`은 다음과 같습니다.

```java
@Configuration
@ComponentScan(value = {
    "com.xyzlast.bookstore.repository"
})
@EnableTransactionManagement
public class BookStoreConfig {
    private ApplicationContext context;

    @Autowired
    public BookStoreConfig(ApplicationContext applicationContext) {
        this.context = applicationContext;
    }

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
        return new DataSourceTransactionManager(dataSource());
    }

    @Bean
    public SqlSessionFactoryBean sqlSessionFactory() {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource());
        sqlSessionFactoryBean.setConfigLocation(context.getResource("classpath:mybatis-config.xml"));
        return sqlSessionFactoryBean;
    }
}
```

그리고, `BookDaoImpl`을 구성해야지 됩니다. `BookDaoImpl`은 다음과 같습니다.

```java
@Repository
public class BookDaoImpl extends SqlSessionDaoSupport implements BookDao {

    @Autowired
    public BookDaoImpl(SqlSessionFactory sqlSessionFactory) {
        super.setSqlSessionFactory(sqlSessionFactory);
    }

    @Override
    public int countAll() {
        return getSqlSession().getMapper(BookDao.class).countAll();
    }

    @Override
    public List<Book> getAll() {
        return getSqlSession().getMapper(BookDao.class).getAll();
    }

    @Override
    public Book getById(int id) {
        return getSqlSession().getMapper(BookDao.class).getById(id);
    }

    @Override
    public int add(Book book) {
        return getSqlSession().insert("com.xyzlast.bookstore.repository.BookDao.add", book);
    }

    @Override
    public int update(Book book) {
        return getSqlSession().update("com.xyzlast.bookstore.repository.BookDao.update", book);
    }

    @Override
    public int delete(int bookId) {
        return getSqlSession().delete("com.xyzlast.bookstore.repository.BookDao.delete", bookId);
    }
}
```

재미있는 코드를 하나 발견할 수 있습니다. `BookDaoImpl`의 생성자는 `SqlSessionFactory`를 인자로 받게 됩니다. 그런데, `SqlSessionFactory`를 `Bean`에 등록하는 코드는 `@Configuration`에 없습니다. 이건 어떻게 되는 것일까요. 이는 `SqlSessionFactoryBean`에서 확인할 수 있습니다. `SqlSessionFactoryBean`의 내부 코드는 다음과 같습니다.

```java
public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {
  ....
}
```

구현된 interface중에 `FactoryBean`이 있는 것을 보실 수 있습니다. Spring Configuration 내부에 `Factory Pattern`을 통해서 객체를 얻어내야지 될 필요가 있을 때, `FactoryBean` interface를 이용해서 구현하면 Spring은 이를 감지하여 `Factory Pattern`을 통해 객체를 얻어내게 됩니다.

마지막으로, query를 지정하는 방법은 다음 2가지로 볼 수 있습니다. `getMapper`를 통하거나, xml의 full-name을 기술하는 것. 두 방법 모두를 이용해서 구성이 가능합니다.

## Summary

국내에서 가장 많이 사용되는 `myBatis`를 이용한 DB 접근 방식에 대해서 알아봤습니다. 매우 간편한 설정 및 사용의 편의성, 무엇보다도 java 코드와 sql query간의 명확한 분리가 가장 큰 장점입니다.

## HOMEWORK

기존 코드의 Dao를 모두 `myBatis`를 이용해서 구성해주시길 바랍니다.