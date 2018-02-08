---
title: 05-04.SampleApplication(4)
date: 2018-02-05 08:58:40
tags: study, java, refactoring
---
# SampleApplication(4) - TemplateCallback pattern

## 코드 정리 - 객체간의 중복 코드 정리

지금까지 우리는 books라는 한개의 table에 CRUD action을 진행해왔습니다. 그렇지만, 개발에서는 한개의 Table만에 CRUD를 하는 경우보다, 한 Action에 대하여 여러개의 Table을 CRUD 하게 되는 것이 일반적입니다.

그래서, 이 두개의 개념을 나누게 되는데요.
하나의 Action이라는 것은 Business Logic이라고 할 수 있고, 한 Table에 대한 CRUD는 DB에 종속적인 작업이라고 할 수 있습니다.
전자를 일반적으로 Service라고 지칭하고, 후자를 DAO (Data Access Object)라고 지칭하는 것이 일반적입니다. 그리고, DB를 통해서 처리되는 Table 단위 또는 Query 단위의 Object들을 Entity(or model)라고 명명합니다.

이와 같은 이름으로 package로 나누게 되는 것이 일반적입니다. dao, entities, service package를 나누것과 같이 이런 식으로 나눠주는 것이 좋습니다.

1개의 Service는 여러개의 DAO를 가지고 있고, DAO를 이용한 BL을 서술하는 것이 일반적입니다. DAO는 최대한 단순하게 Table에 대한 Query들로 구성이 되고, 그 Query를 통해서 결과를 얻어내는 수단으로 주로 사용됩니다. bookstore에 Business Logic(BL)을 추가하기 전에 book의 상태를 서술할수 있는 property와 현재 책을 빌려간 사용자의 정보를 저장할 수 있는 Column을 두개 추가하고, users table을 추가해서 Dao를 좀더 구성해보도록 하겠습니다.

변경된 table의 구조가 반영된 것으로 docker에 새로운 image를 올려뒀습니다. 기존 container를 삭제시키고, 새로운 docker container를 실행시켜주세요.

```cmd
docker run -d -p 4306:3306 192.168.94.18:15000/study-container:003 mysqld_safe
```

새로운 Docker Image안에 들어있는 db의 table 구조는 다음과 같습니다.

```cmd
mysql> show tables;
+---------------------+
| Tables_in_bookstore |
+---------------------+
| books               |
| histories           |
| users               |
+---------------------+
3 rows in set (0.01 sec)

mysql> describe books;
+-------------+--------------+------+-----+-------------------+-----------------------------+
| Field       | Type         | Null | Key | Default           | Extra                       |
+-------------+--------------+------+-----+-------------------+-----------------------------+
| id          | int(11)      | NO   | PRI | NULL              | auto_increment              |
| name        | varchar(255) | NO   |     | NULL              |                             |
| author      | varchar(50)  | NO   |     | NULL              |                             |
| publishDate | timestamp    | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| comment     | varchar(255) | YES  |     | NULL              |                             |
| status      | int(11)      | NO   |     | NULL              |                             |
| rentUserId  | int(11)      | YES  | MUL | NULL              |                             |
+-------------+--------------+------+-----+-------------------+-----------------------------+

mysql> describe users;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| id       | int(11)     | NO   | PRI | NULL    | auto_increment |
| name     | varchar(50) | NO   |     | NULL    |                |
| password | varchar(12) | NO   |     | NULL    |                |
| point    | int(11)     | NO   |     | NULL    |                |
| level    | int(11)     | NO   |     | NULL    |                |
+----------+-------------+------+-----+---------+----------------+
5 rows in set (0.00 sec)

mysql> describe histories;
+------------+-----------+------+-----+-------------------+-----------------------------+
| Field      | Type      | Null | Key | Default           | Extra                       |
+------------+-----------+------+-----+-------------------+-----------------------------+
| id         | int(11)   | NO   | PRI | NULL              | auto_increment              |
| userId     | int(11)   | NO   | MUL | NULL              |                             |
| bookId     | int(11)   | NO   | MUL | NULL              |                             |
| actionType | int(11)   | NO   |     | NULL              |                             |
| insertDate | timestamp | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+------------+-----------+------+-----+-------------------+-----------------------------+
5 rows in set (0.00 sec)
```

위 `Service`와 `DAO`에 대한 정의를 보면 지금까지 작성된 `BookService`는 `DAO`의 성격이 더 강한 것을 알 수 있습니다. 그렇다면 package에 대한 전체적인 변경을 해보도록 하겠습니다.

```cmd
├── dao
├── entity
└── service
```

이 구조는 `Business Logic`을 다루는 package의 일반적인 구조가 될 수 있습니다. 추후에 `WebApplication`에서 package에 대한 구조를 좀 더 깊게 다뤄보도록 하겠습니다.

그리고 3개의 Table에 해당되는 `DAO`인 `BookDao`, `UserDao`, `HistoryDao`를 추가합니다. 그리고, 각 `Dao`에서 해당되는 `Entity`들을 모두 추가하면 일단 기본 포멧은 꾸며질 수 있게 됩니다.

## Entity (or Model)

Table 단위의 접근 객체를 의미합니다. 이는 DAO에서 처리하는 객체 단위가 될 수 있습니다. 조금 더 깊은 내용들은 `ORM(Oriented Relation Model)`에서 좀 더 깊게 다루도록 하겠습니다.

기존 Book의 경우, `status`와 `rentUserId`가 추가되었습니다. 이를 단순히 `int` 값으로 처리가 가능합니다. 그런데, 이렇게 int 값으로 처리를 하게 되면 int 값에 대한 내용 서술이 되지 않습니다. 이런 경우 이 값을 `enum` 형태로 만들어서 처리해주면 좋습니다. 이 부분에 대해서는 논쟁이 조금 있습니다. 상태의 경우 `code table`을 통해서 처리가 되는 것이 일반적입니다. 이렇게 `enum`을 이용할 때의 장점은 무엇이 있을까요? 이 부분에 대해서는 각 개발자들이 한번 공부를 해보는 것이 어떨까 생각을 하고 있습니다. 약간의 논쟁이 되는 부분중 하나라고 생각합니다. (http://woowabros.github.io/tools/2017/07/10/java-enum-uses.html)

BookStatus를 enum으로 정의하면 다음 코드로 표현이 가능합니다.

```java
public enum BookStatus {
    CanRent(0),
    RentNow(1),
    Missing(2);

    private int value;
    BookStatus(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }

    public static BookStatus parse(int value) {
        switch(value) {
            case 0 : return CanRent;
            case 1 : return RentNow;
            case 2 : return Missing;
            default:
                throw new IllegalArgumentException();
        }
    }
}
```

`BookStatus`를 반영한 `Book` 코드는 다음과 같이 구성됩니다. (lombok을 찬양합시다.)

```java
@Getter
@Setter
public class Book {
    private int id;
    private String name;
    private String comment;
    private String author;
    private Date publishDate;
    private BookStatus bookStatus;
}
```

이제 `User`, `History`에 대한 `Entity`를 구성해보도록 합니다.

```java
@Getter
@Setter
public class User {
    private int id;
    private String name;
    private String password;
    private int point;
    private int level;
}

@Getter
@Setter
public class History {
    private int id;
    private int bookId;
    private int userId;
    private HistoryActionType actionType;
    private Date date;
}
```

## DAO의 기능

`DAO`는 일반적으로 Data에 접근하고 처리할 수 있는 기능들을 제공합니다. 이는 Table의 CRUD기능을 의미하게 됩니다. 이제 일반적인 CRUD의 경우, 다음 Code로 정의가 가능할 수 있습니다.

```java
public interface BookStoreDao<T, K> {
    int countAll() throws Exception;
    void deleteAll() throws Exception;

    List<T> getAll() throws Exception;
    T getById(K id) throws Exception;
    boolean update(T entity) throws Exception;
    boolean add(T entity) throws Exception;
    boolean delete(T entity) throws Exception;
}
```

이렇게 `interface`를 제공하게 되면, 객체들을 만들때 어떤 객체들을 다 만들어줘야지 되는지를 좀 더 명확히 해줄 수 있습니다. 그리고 구현이 되는 객체가 어떤 일을 할 수 있는지에 대한 선언을 해줄 수 있습니다. 이제 구현되는 `BookDao`를 한번 구현해 보도록 하겠습니다.

```java
public class BookDao implements BookStoreDao<Book, Integer> {

    private final ConnectionFactory connectionFactory;

    public BookDao(ConnectionFactory connectionFactory) {
        this.connectionFactory = connectionFactory;
    }

    @Override
    public int countAll() throws Exception {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement("select count(*) from books");
        ResultSet rs = st.executeQuery();
        rs.next();

        Long count = rs.getLong(1);

        rs.close();
        st.close();
        conn.close();

        return count.intValue();
    }

    @Override
    public void deleteAll() throws Exception {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement("delete from books");
        st.execute();
        st.close();
        conn.close();
    }

    @Override
    public List<Book> getAll() throws Exception {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement("select id, name, author, status, rentUserId, comment, publishDate from books");
        ResultSet rs = st.executeQuery();

        List<Book> books = new ArrayList<>();
        while (rs.next()) {
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
            books.add(book);
        }
        rs.close();
        st.close();
        conn.close();
        return books;
    }

    @Override
    public Book getById(Integer id) throws Exception {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement("select id, name, author, publishDate, comment, status, rentUserId from books where id=?");
        st.setInt(1, id);
        ResultSet rs = st.executeQuery();

        Book book = null;
        if (rs.next()) {
            book = new Book();
            book.setId(rs.getInt("id"));
            book.setName(rs.getString("name"));
            book.setAuthor(rs.getString("author"));
            java.util.Date date = new java.util.Date(rs.getDate("publishDate").getTime());
            book.setPublishDate(date);
            book.setPublishDate(date);
            book.setBookStatus(BookStatus.parse(rs.getInt("status")));
            Integer rentUserId = (Integer) rs.getObject("rentUserId");
            book.setRentUserId(rentUserId);
            book.setComment(rs.getString("comment"));
        }
        rs.close();
        st.close();
        conn.close();
        return book;
    }

    @Override
    public boolean update(Book book) throws Exception {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement("update books set name = ?, author = ?, publishDate = ?, comment = ?, status = ?, rentUserId =? where id = ?");
        st.setString(1, book.getName());
        st.setString(2, book.getAuthor());
        st.setDate(3, new java.sql.Date(book.getPublishDate().getTime()));
        st.setString(4, book.getComment());
        st.setInt(5, book.getBookStatus().value());
        st.setObject(6, book.getRentUserId());
        st.setInt(7, book.getId());
        st.execute();

        st.close();
        conn.close();

        return true;
    }

    @Override
    public boolean add(Book book) throws Exception {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement("insert books(name, author, publishDate, comment, status, rentUserId) values(?, ?, ?, ?, ?, ?)");
        st.setString(1, book.getName());
        st.setString(2, book.getAuthor());
        java.sql.Date sqlDate = new java.sql.Date(book.getPublishDate().getTime());
        st.setDate(3, sqlDate);
        st.setString(4, book.getComment());
        st.setInt(5, book.getBookStatus().value());
        st.setObject(6, book.getRentUserId());
        st.execute();
        st.close();
        conn.close();

        return true;
    }

    @Override
    public boolean delete(Book entity) throws Exception {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement("delete from books where id = ?");
        st.setInt(1, entity.getId());
        st.execute();

        st.close();
        conn.close();

        return true;
    }
}
```

이제 여기서 중복 코드를 제거해보는 절차를 진행해보고자 합니다. 먼저 계속해서나오는 `ResultSet`을 `Book`으로 변환시키는 코드가 계속해서 중복되고 있는 것을 볼 수 있습니다.

이를 따로 뽑아내면 다음과 같습니다.

```java
private Book convertFromResultSet(ResultSet rs) throws Exception {
    Book book = new Book();
    book.setId(rs.getInt("id"));
    book.setName(rs.getString("name"));
    book.setAuthor(rs.getString("author"));
    java.util.Date date = new java.util.Date(rs.getDate("publishDate").getTime());
    book.setPublishDate(date);
    book.setPublishDate(date);
    book.setBookStatus(BookStatus.parse(rs.getInt("status")));
    Integer rentUserId = (Integer) rs.getObject("rentUserId");
    book.setRentUserId(rentUserId);
    book.setComment(rs.getString("comment"));

    return book;
}
```

DB에서 Query를 실행시키는 코드는 두가지의 패턴을 가지고 있습니다. `add`, `update`, `delete`와 같이 query를 실행시키는 코드와 `getById`, `getAll`과 같이 query의 실행 후 그 결과(`ResultSet`)를 이용해서 객체를 만들어주는 코드로 나눌 수 있습니다.

### Query를 단순 실행하는 코드

DB에서 값을 처리하는 과정은 다음과 같습니다.  `Connection`을 얻어내고 `PreparedStatement`를 구성하고, `PreparedStatement`를 `Close`시켜주고, `Connection`을 `Close`시켜주게 됩니다. 이는 정해진 패턴이고 계속되는 중복 코드를 가지고 오게 됩니다.

이제 이 코드는 다음과 같이 보여줄 수 있습니다.

```java
interface InnerPreparedStatementProcess {
    void doProcess(PreparedStatement st) throws Exception;
}

private void executeProcess(String sql, InnerPreparedStatementProcess process) throws Exception {
    Connection conn = connectionFactory.getConnection();
    PreparedStatement st = conn.prepareStatement(sql);
    process.doProcess(st);
    st.execute();
    st.close();
    conn.close();
}
```

이렇게 수정하면 다음 코드들은 다음과 같이 수정이 가능합니다.

```java
@Override
public boolean update(Book book) throws Exception {
    executeProcess("update books set name = ?, author = ?, publishDate = ?, comment = ?, status = ?, rentUserId =? where id = ?", (st) -> {
        st.setString(1, book.getName());
        st.setString(2, book.getAuthor());
        st.setDate(3, new java.sql.Date(book.getPublishDate().getTime()));
        st.setString(4, book.getComment());
        st.setInt(5, book.getBookStatus().value());
        st.setObject(6, book.getRentUserId());
        st.setInt(7, book.getId());

    });
    return true;
}

@Override
public boolean add(Book book) throws Exception {
    executeProcess("insert books(name, author, publishDate, comment, status, rentUserId) values(?, ?, ?, ?, ?, ?)", (st) -> {
        st.setString(1, book.getName());
        st.setString(2, book.getAuthor());
        java.sql.Date sqlDate = new java.sql.Date(book.getPublishDate().getTime());
        st.setDate(3, sqlDate);
        st.setString(4, book.getComment());
        st.setInt(5, book.getBookStatus().value());
        st.setObject(6, book.getRentUserId());
    });
    return true;
}

@Override
public boolean delete(Book entity) throws Exception {
    executeProcess("delete from books where id = ?", (st) -> {
        st.setInt(1, entity.getId());
    });
    return true;
}
```

기존 코드와 비교해서 어느정도 코드가 줄어드는지 확인해보시길 바랍니다. 이렇게 시작 Process와 종료 Process가 동일한데 중간 Action 만이 변경되는 코드를 위와 같이 구성하는 패턴을 `Template Pattern`이라고 합니다.

### Query의 실행결과를 이용하는 경우

Query를 단순 실행하는 패턴과 거의 동일합니다. 패턴의 코드는 다음과 같습니다.

```java
private <T> T executeProcess(String sql, InnerPreparedStatementAndResultSetProcess<T> process) throws Exception {
    Connection conn = connectionFactory.getConnection();
    PreparedStatement st = conn.prepareStatement(sql);

    process.doProcess(st);

    ResultSet rs = st.executeQuery();

    T result = process.convertFromResultSet(rs);

    rs.close();
    st.close();
    conn.close();

    return result;
}
```

적용된 Pattern을 적용하면 `BookDao`는 다음과 같이 변경할 수 있습니다.

```java
public class BookDao implements BookStoreDao<Book, Integer> {

    private final ConnectionFactory connectionFactory;

    public BookDao(ConnectionFactory connectionFactory) {
        this.connectionFactory = connectionFactory;
    }

    @Override
    public int countAll() throws Exception {
        String sql = "select count(*) from books";
        return executeProcess(sql, new InnerPreparedStatementAndResultSetProcess<Integer>() {
            @Override
            public void doProcess(PreparedStatement st) throws Exception {

            }

            @Override
            public Integer convertFromResultSet(ResultSet rs) throws Exception {
                Long count = rs.getLong(1);
                return count.intValue();
            }
        });
    }

    @Override
    public void deleteAll() throws Exception {
        executeProcess("delete from books", (st) -> {});
    }

    @Override
    public List<Book> getAll() throws Exception {
        String sql = "select id, name, author, status, rentUserId, comment, publishDate from books";
        return executeProcess(sql, new InnerPreparedStatementAndResultSetProcess<List<Book>>() {
            @Override
            public void doProcess(PreparedStatement st) throws Exception {

            }

            @Override
            public List<Book> convertFromResultSet(ResultSet rs) throws Exception {
                List<Book> books = new ArrayList<>();
                while (rs.next()) {
                    books.add(convertBookFromResultSet(rs));
                }
                return books;
            }
        });
    }

    @Override
    public Book getById(Integer id) throws Exception {
        String sql = "select id, name, author, publishDate, comment, status, rentUserId from books where id=?";
        return executeProcess(sql, new InnerPreparedStatementAndResultSetProcess<Book>() {
            @Override
            public void doProcess(PreparedStatement st) throws Exception {
                st.setInt(1, id);
            }
            @Override
            public Book convertFromResultSet(ResultSet rs) throws Exception {
                Book book = null;
                if (rs.next()) {
                    book = convertBookFromResultSet(rs);
                }
                return book;
            }
        });
    }

    @Override
    public boolean update(Book book) throws Exception {
        executeProcess("update books set name = ?, author = ?, publishDate = ?, comment = ?, status = ?, rentUserId =? where id = ?", (st) -> {
            st.setString(1, book.getName());
            st.setString(2, book.getAuthor());
            st.setDate(3, new java.sql.Date(book.getPublishDate().getTime()));
            st.setString(4, book.getComment());
            st.setInt(5, book.getBookStatus().value());
            st.setObject(6, book.getRentUserId());
            st.setInt(7, book.getId());
        });
        return true;
    }

    @Override
    public boolean add(Book book) throws Exception {
        executeProcess("insert books(name, author, publishDate, comment, status, rentUserId) values(?, ?, ?, ?, ?, ?)", (st) -> {
            st.setString(1, book.getName());
            st.setString(2, book.getAuthor());
            java.sql.Date sqlDate = new java.sql.Date(book.getPublishDate().getTime());
            st.setDate(3, sqlDate);
            st.setString(4, book.getComment());
            st.setInt(5, book.getBookStatus().value());
            st.setObject(6, book.getRentUserId());
        });
        return true;
    }

    @Override
    public boolean delete(Book entity) throws Exception {
        executeProcess("delete from books where id = ?", (st) -> {
            st.setInt(1, entity.getId());
        });
        return true;
    }

    interface InnerPreparedStatementProcess {
        void doProcess(PreparedStatement st) throws Exception;
    }

    interface InnerPreparedStatementAndResultSetProcess<T> {
        void doProcess(PreparedStatement st) throws Exception;
        T convertFromResultSet(ResultSet rs) throws Exception;
    }

    private <T> T executeProcess(String sql, InnerPreparedStatementAndResultSetProcess<T> process) throws Exception {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement(sql);
        process.doProcess(st);
        ResultSet rs = st.executeQuery();
        T result = process.convertFromResultSet(rs);
        rs.close();
        st.close();
        conn.close();

        return result;
    }

    private void executeProcess(String sql, InnerPreparedStatementProcess process) throws Exception {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement(sql);
        process.doProcess(st);
        st.execute();
        st.close();
        conn.close();
    }

    private Book convertBookFromResultSet(ResultSet rs) throws Exception {
        Book book = new Book();
        book.setId(rs.getInt("id"));
        book.setName(rs.getString("name"));
        book.setAuthor(rs.getString("author"));
        java.util.Date date = new java.util.Date(rs.getDate("publishDate").getTime());
        book.setPublishDate(date);
        book.setPublishDate(date);
        book.setBookStatus(BookStatus.parse(rs.getInt("status")));
        Integer rentUserId = (Integer) rs.getObject("rentUserId");
        book.setRentUserId(rentUserId);
        book.setComment(rs.getString("comment"));

        return book;
    }
}
```

여기서 반영된 코드는 이제 `Query`를 실행시키는 역활만을 담당하게 됩니다. 이제 좀더 `OOP`적인 코드가 될 수 있게 됩니다.
`DAO`는 DB에 `CRUD`를 행합니다. `DAO`에서 `CRUD`를 처리할 때, 이제 DB에 대한 연결, Query의 전달만을 처리하는 것으로 역활을 담당하면 이 역활을 타 DAO(UserDao, HistoryDao)가 같이 사용할 수 있게 됩니다.

## Refactoring SQL 실행부

이제 DB에서의 실행부를 분리해보도록 하겠습니다. `SqlExecutor`라는 객체로 만들어서 따로 뽑아주고, 그에 대한 interface 역시 정리해줍니다. 그럼 코드는 다음과 같은 형태가 될 수 있습니다.

```java
public class SqlExecutor {

    private final ConnectionFactory connectionFactory;

    public SqlExecutor(ConnectionFactory connectionFactory) {
        this.connectionFactory = connectionFactory;
    }

    public <T> T executeProcess(String sql, InnerPreparedStatementAndResultSetProcess<T> process) throws Exception {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement(sql);
        process.doProcess(st);
        ResultSet rs = st.executeQuery();
        T result = process.convertFromResultSet(rs);
        rs.close();
        st.close();
        conn.close();

        return result;
    }

    public void executeProcess(String sql, InnerPreparedStatementProcess process) throws Exception {
        Connection conn = connectionFactory.getConnection();
        PreparedStatement st = conn.prepareStatement(sql);
        process.doProcess(st);
        st.execute();
        st.close();
        conn.close();
    }
}
```

이제 `DAO`에서는 `ConnectionFactory`가 아닌 `SqlExecutor`를 이용해서 처리하도록 변경합니다. 생성자가 다음과 같이 변경될 필요가 있습니다.

```java
private final SqlExecutor sqlExecutor;

public BookDao(SqlExecutor sqlExecutor) {
    this.sqlExecutor = sqlExecutor;
}

@Override
public int countAll() throws Exception {
    String sql = "select count(*) from books";
    return sqlExecutor.executeProcess(sql, new InnerPreparedStatementAndResultSetProcess<Integer>() {
        @Override
        public void doProcess(PreparedStatement st) throws Exception {

        }

        @Override
        public Integer convertFromResultSet(ResultSet rs) throws Exception {
            Long count = rs.getLong(1);
            return count.intValue();
        }
    });
}
```

이제 구성된 `BookDao`, `UserDao`, `HistoryDao`를 구성하는 `applicaiton-context.xml`은 다음과 같습니다. `@Configuration`도 같이 첨부합니다. 참고해보시길 바랍니다.

## 코드 정리 - Exception의 처리

지금까지 보시면 SqlExecutor를 이용한 코드의 간결화를 해온것을 알 수 있습니다.

마지막으로, 지금 저희 코드에 중복이 되는 코드를 찾아보도록 합시다.
딱히 문제가 특출나게 보이지는 않습니다. 지금까지 상당한 refactoring을 통해서 변경시킨 코드에 문제가 쉽게 보이면 그것 역시 문제가 될 수 있습니다.

지금 모든 코드에 나타나 있는 Exception이 선언되어 있습니다. DB access 코드에 원래 일괄적으로 들어가 있어야할 Exception들은 다음과 같습니다.

* InstantiationException : Class.forName 에서 객체의 이름이 아닌, interface의 이름이 들어간 경우에 발생하는 에러.
* IllegalAccessException : Db Connection시, 권한이 없거나 id/password가 틀린 경우에 발생하는 에러
* ClassNotFoundException : Class.forName 을 이용, DB Connection 객체를 생성할 때 객체의 이름이 틀린 경우에 발생하는 에러
* SQLException : SQL query가 잘못된 Exception입니다.

Java는 2개의 Exception type을 가지고 있습니다. checked exception과 Runtime Exception인데요. checked exception의 경우, 이 exception이 발생하는 경우에는 반드시 exception을 처리해줘야지 됩니다. 또는 상위 method로 throw를 해줘야지 됩니다.

Runtime exception은 상위에서 처리를 안해줘도 되고요. 대표적인 것은 NullPointerException, UnsupportedOperationException, IllegalArgumentException 등이 있습니다.

이 부분은 매우 중요한 개념입니다. java에서의 exception은 사용자가 처리해줘야지 될 것(체크 해야지 될 exception)과 Runtime 시(실행시에) 확인되어야지 될 것들로 나뉘게 됩니다. exception에 대하여 보다더 확실한 처리를 해주길 바란 java의 설계 원칙이지만, 근간에는 비판이 좀 많은 부분이기도 합니다. java의 exception에 대한 정리를 한번 해주시는 것이 필요합니다.

그럼, 지금 저희가 사용한 코드중에서 getConnection() method를 확인해보겠습니다. Exception은 원래 이렇게 처리가 되어야지 됩니다.

```java
public Connection getConnection() throws InstantiationException, IllegalAccessException, ClassNotFoundException, SQLException {
    Connection conn = DriverManager.getConnection (this.connectionString, this.username, this.password);
    return conn;
}
```

이 method는 이 4개의 exception을 반드시 처리하도록 되어 있습니다. 또는 이 4개의 exception을 사용한 method로 던져줘서 상위 method에서 처리하도록 되어 있는데요. DB 접속과 SQL query가 잘못된 경우에 대한 exception 처리는 과연 할 수 있을까요? 이 Exception은 처리할 수 없는 Exception을 넘겨줘서 되는 것이 아닌가. 라는 생각을 할 수 있습니다. 잘 보면 대부분의 DB 접속시에 나오는 대부분의 Exception은 처리가 불가능한 Exception이라고 할 수 있습니다. 에러의 내용은 도움이 될 수 있지만, 이 에러가 발생했을 때 어떠한 처리를 하지를 못하는 경우가 대다수라는거지요.

어떤 방법으로 Exception을 처리할지는 이 코드를 사용하는 곳에서 어떻게 할지를 정해보는것이 좋습니다. 예를 들어, `SqlExecutor`에서 발생하는 에러를 모두 `DaoException`으로 만들어서 처리하고, log를 남기는 것도 방법입니다.

## Summary

* 객체간의 코드중복은 최대한 줄여야지 됩니다. 이를 남겨두면 결국은 코드의 기술부채로서 남습니다.
* 코드중복을 없애는 방법중에서 유용하지만, 어렵지만 좋은 Pattern이 Template Pattern을 이용해서 처리하는 것입니다.
* Java의 Exception은 Check-Exception과 Runtime-Exception으로 구분되고 있습니다. java나 library들에서 제공되는 객체들의 Check-Exception은 남겨두는 것이 좋지만, BL Logic에서의 Check-Exception은 독이 되는 것이 대부분입니다. Runtime-Exception 으로 변경시켜서 처리하는 것이 좋습니다.

다음은 지금까지 구성된 코드를 `Spring`으로 구성되는 BookStore를 만나보실 수 있을 것입니다.

ps: UserDao, HistoryDao 코드를 모두 만들어서 와주시길 바랍니다.