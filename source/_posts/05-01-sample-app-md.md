---
title: 05-01.SampleApplication 개발(1) - 초난감 Project
date: 2018-02-01 12:46:33
tags: study
---

# spring을 이용한 Simple application의 제작 (1) - 초난감 프로젝트

이 장의 제목은 Toby의 Spring Framework에서 붙인 이름을 그대로 표절해봤습니다. 매우 큰 문제를 가진 간단한 프로그램이 뛰어난 확장성과 처음의 너저분한 코드에서 점차 깔끔하게 구성 되어가는 코드로 점차 변경되어가는 것을 볼 수 있을겁니다.
먼저, 간단한 application입니다. bookStore라고 하나의 Project를 만들고, books 라는 table에 대한 CRUD와 count를 하는 application을 간단히 작성해보도록 하겠습니다.

## 개발 환경 준비 & 초기화

### Docker를 이용한 Database 준비

먼저, 대상이 되는 database를 준비하도록 하겠습니다. docker를 이용해서 제가 준비를 해두었습니다. docker cmd를 통해서 다음 명령어를 실행합니다.

```cmd
docker run -d -p 4306:3306 192.168.94.18:15000/study-container:002 mysqld_safe
```

를 실행하면 docker container에서 실행되어 있을것입니다.
그 후, host machine으로 돌아와서(CTRL + P, CTRL + Q) 다음 `cmd`를 통해 db에 정상적으로 붙을 수 있는지 확인하도록 합니다.

```cmd
mysql -u root -pqwer12#$ -h 127.0.0.1 -P4306 bookstore
```

```cmd
mysql> use bookstore;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from books;
+----+-----------+-----------+---------------------+-------------------+
| id | name      | author    | publishDate         | comment           |
+----+-----------+-----------+---------------------+-------------------+
|  1 | TEST BOOK | 나야나     | 2017-12-18 00:00:00 | 이건 뭘까요.       |
+----+-----------+-----------+---------------------+-------------------+
1 row in set (0.00 sec)
```

### gradle을 이용한 project 초기화 작업 진행

이제 db가 준비된 것을 확인하였으니 시작할 folder를 지정하고 다음의 gradle 명령어를 넣어 시작합니다.

```cmd
gradle init --type java-application
```

몇개의 process들이 진행되고, 다음 실행 결과가 나오면 일단 종료된겁니다.

```cmd
Starting a Gradle Daemon (subsequent builds will be faster)

BUILD SUCCESSFUL in 3s
2 actionable tasks: 2 executed
```

만들어진 folder 구조는 다음과 같습니다.

```cmd
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── settings.gradle
└── src
    ├── main
    │   └── java
    │       └── App.java
    └── test
        └── java
            └── AppTest.java
```
폴더 기반으로 먼저 설명을 드린다면, 다음과 같습니다.

|name|description|
|---|---|
|gradle|gradle 실행 파일이 위치한 folder입니다. 신경쓰지 않으셔도 됩니다.|
|src/main|실제 코드가 들어갈 folder입니다. 우리가 주로 작성할 코드가 이곳에 위치합니다|
|src/test|테스트 코드가 들어갈 folder입니다. 작성된 코드가 정상적으로 동작하는지를 확인하기 위한 코드 작성 구역입니다.|
|build.gradle|build가 선언된 파일입니다.|

이 구조는 잘 알아두어야지 됩니다. 지금 java 개발에서 이 folder구조가 아닌 구조로 만약 개발이 되어있다면 그 project는 기본적으로 원칙을 지키지 않은 상태입니다. 현 상태에서는 이 구조가 표준입니다.

다음은 `build.gradle` 파일을 알아보도록 하겠습니다. 안드로이드 개발을 해보신 분들은 자주 보시던 구조일겁니다.

```groovy
apply plugin: 'java'
apply plugin: 'application'

repositories {
    jcenter()
}
dependencies {
    compile 'com.google.guava:guava:23.0'
    testCompile 'junit:junit:4.12'
}
mainClassName = 'App'
```

먼저 정의된 것은 `plugin`입니다. 이 build에서 어떤 것을 사용할지를 정의합니다. `java`이며, `application`을 정의하고 있습니다. 이 구조는 `jar`형태로 실행될 수 있는 java application을 개발할 때 사용되는 구조입니다.

repositories는 dependency library들을 다운 받을 위치를 지정합니다. `jcenter`를 기본값으로 사용하고 있으며, 다음 값들을 갖는 것이 일반적입니다.

```groovy
jcenter()
mavenCentral()
mavenLocal()
maven {
    url: 'http://repo.mycompany.com/maven2',
    credentials: {
        username: 'user',
        password: 'password'
    }
}
```

dependencies는 dependency library들을 위치하는 곳입니다. 접미어가 붙어있는데 다음과 같은 의미를 갖습니다.

|name|description|
|---|---|
|compile|Compile시에 사용되는 library입니다. 이는 `src/main`에서 사용되는 library 들을 의미합니다.|
|testCompile|Test Compile시에 사용되는 library입니다. 이는 `src/test`에서 사용되는 library들을 의미합니다.|

### Project Import with IntelliJ

IntelliJ에서 이제 project를 import해서 개발을 할 수 있는 준비를 시작합니다.

실행 후, `Import Project`를 통해 `build.gradle`이 있는 folder를 import 시킵니다.

![](/images/05/intellij01.png)

![](/images/05/intellij02.png)

![](/images/05/intellij03.png)

선택할때, 다른 option은 크게 신경쓸 필요는 없지만, 다음 Option은 볼 필요가 있습니다.

|menu|description|
|---|---|
|Use default gradle wrapper(recommended)|gradle wrapper를 사용합니다.|
|Use gradle wrapper task configuration|gradle wrapper를 이용해서 그 안의 세부 task를 이용합니다.|
|Use local gradle distribution|local PC에 설치된 gradle을 이용합니다.|

recommanded가 첫번째로 되어 있지만, 저는 3번째 option을 사용하는 것을 추천합니다. android project의 경우에는 첫번째 option을 사용하는 것이 좋습니다. gradle이 버젼업이 되어가면서 groovy 문법의 version이 바뀐 것 때문에 구 버젼에서 만들어진 `build.gradle`이 local machine에서 동작하지 않는 경우가 발생할 수 있습니다. 그래서 gradle은 자신이 만들어질 때 사용한 gradle의 wrapper version을 project에 같이 배포하는 것으로 버젼의 문제를 해결할 수 있습니다. 단, 지금의 경우 local PC에 설치된 gradle이 속도가 가장 빠르기 때문에 이럴때는 그냥 local PC의 gradle을 사용하는 것을 권장합니다.

프로젝트가 로드되면 다음과 같은 화면을 볼 수 있습니다.

![](/images/05/intellij04.png)
![](/images/05/intellij05.png)

이제 intelliJ에서 coding을 할 준비가 모두 완료되었습니다.

### 외부 Library의 추가

docker에 의해 제공된 DB는 mariaDb입니다. mariadb의 JDBC connector를 구하기 위해서 `http://mvnrepository.com/`에서 mariadb을 검색해서 dependency library를 검색합니다.

![](/images/05/maven01.png)

`Gradle` 텝으로 가서 copy해서 `build.gradle`에 추가합니다.
추가 후, 우측 dock에 위치한 `Gradle`에서 `refresh all Gradle projects`를 실행시킵니다.

![](/images/05/intellij06.png)

실행시키면 아래 `External Libraries`에 보면 `org.mariadb.jdbc:mariadb-java-client:2.2.0`이 추가된 것을 볼 수 있습니다.

![](/images/05/intellij07.png)

### 초난감 프로젝트의 시작

먼저 우리의 프로젝트는 다음 일을 할 수 있는 객체를 만들것입니다.

* books table에 CRUD가 가능하도록 한다.
* books에 총 몇개의 row가 있는지 확인 할 수 있어야지 된다.
* books table을 clear 시킬 수 있도록 한다.

간단히 Book 객체를 만들겠습니다.

```java
public class Book {
    private int id;
    private String name;
    private String comment;
    private String author;
    private Date publishDate;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getComment() {
        return comment;
    }

    public void setComment(String comment) {
        this.comment = comment;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public Date getPublishDate() {
        return publishDate;
    }

    public void setPublishDate(Date publishDate) {
        this.publishDate = publishDate;
    }
}
```

기본적인 `VO` 형태로 `getter/setter`가 있는 단순한 코드형태입니다. 이제 이 객체를 이용한 객체를 만들어보도록 하겠습니다.

이제 여기에 CRUD, countAll, deleteAll method를 구현해보도록 하겠습니다.

```java
public class BookService {

    public void add(Book book) throws InstantiationException, IllegalAccessException, ClassNotFoundException, SQLException {
        String url = "jdbc:mysql://127.0.0.1:4306/bookstore";
        Class.forName("org.mariadb.jdbc.Driver").newInstance();
        Connection conn = DriverManager.getConnection (url, "root", "qwer12#$");

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

    public Book update(Book book) throws InstantiationException, IllegalAccessException, ClassNotFoundException, SQLException {
        String url = "jdbc:mysql://127.0.0.1:4306/bookstore";
        Class.forName("org.mariadb.jdbc.Driver").newInstance();
        Connection conn = DriverManager.getConnection (url, "root", "qwer12#$");

        PreparedStatement st = conn.prepareStatement("update books set name = ?, author = ?, publishDate = ?, comment = ? where id = ?");
        st.setString(1, book.getName());
        st.setString(2, book.getAuthor());
        st.setDate(3, new java.sql.Date(book.getPublishDate().getTime()));
        st.setString(4, book.getComment());
        st.setInt(5, book.getId());
        st.execute();

        st.close();
        conn.close();

        return book;
    }

    public boolean delete(int bookId) throws InstantiationException, IllegalAccessException, ClassNotFoundException, SQLException {
        String url = "jdbc:mysql://127.0.0.1:4306/bookstore";
        Class.forName("org.mariadb.jdbc.Driver").newInstance();
        Connection conn = DriverManager.getConnection (url, "root", "qwer12#$");

        PreparedStatement st = conn.prepareStatement("delete from books where id = ?");
        st.setInt(1, bookId);
        st.execute();

        st.close();
        conn.close();

        return true;
    }

    public Book get(int id) throws ClassNotFoundException, SQLException, IllegalAccessException, InstantiationException {
        String url = "jdbc:mysql://localhost:4306/bookstore";
        Class.forName("org.mariadb.jdbc.Driver").newInstance();
        Connection conn = DriverManager.getConnection (url, "root", "qwer12#$");

        PreparedStatement st = conn.prepareStatement("select id, name, author, publishDate, comment from books where id=?");
        st.setInt(1, id);
        ResultSet rs = st.executeQuery();

        Book book = null;
        if (!rs.next()) {
            book = new Book();
            book.setId(rs.getInt("id"));
            book.setName(rs.getString("name"));
            book.setAuthor(rs.getString("author"));
            java.util.Date date = new java.util.Date(rs.getDate("publishDate").getTime());
            book.setPublishDate(date);
            book.setComment(rs.getString("comment"));
        }
        rs.close();
        st.close();
        conn.close();
        return book;
    }

    public long countAll() throws SQLException, ClassNotFoundException, IllegalAccessException, InstantiationException {
        String url = "jdbc:mysql://localhost:4306/bookstore";
        Class.forName("org.mariadb.jdbc.Driver").newInstance();
        Connection conn = DriverManager.getConnection (url, "root", "qwer12#$");

        PreparedStatement st = conn.prepareStatement("select count(*) from books");
        ResultSet rs = st.executeQuery();
        rs.next();

        Long count = rs.getLong(1);

        rs.close();
        st.close();
        conn.close();

        return count;
    }
}
```

작성된 코드를 한번 확인해보시길 바랍니다. 이제 이 코드가 정상적으로 돌아가는지 어떻게 하면 확인할 수 있을까요?

먼저 위의 코드의 가장 큰 문제점은 *실행하기 전까지는 정상적으로 코드를 작성했는지 확인할 수가 없다* 입니다.
실행하기 위해서 `BookApp.java`를 추가해서 다음과 같은 코드를 작성했습니다.

```java
public class App {
    public static void main(String[] args) throws ClassNotFoundException, SQLException, InstantiationException, IllegalAccessException {
        BookService bookService = new BookService();
        System.out.println(bookService.countAll());
        Book book = new Book();
        book.setId(1);
        book.setName("ChangedBookName");
        book.setAuthor("나야나");
        book.setComment("이건 뭘까요.");
        book.setPublishDate(new Date());

        bookService.update(book);
    }
}
```

이런 식으로 모든 코드를 실행하는 코드를 만들어주면, 이것을 우리는 어떻게하면 확인을 할 수 있을까요. 이 방법은 매우 많은 사람들이 사용하고 있는 방법입니다. 그렇지만, 많은 사람들이 사용하고 있다고 해서 좋은 방법은 아닌거지요. 실행해봐야지 아는 것은 절대로 좋은 코드가 될 수 없습니다. 실행 후, 확인하는 것은 최종 제품테스트에서나 하는 것이지, 개발중에 하는 것은 아닙니다. 우리는 우리의 코드를 잘 만들어서 테스트 할 필요성이 있습니다.

## Summary

개발환경을 꾸미고, 간단한 코드를 작성해보았습니다.
이제 다음은 만들어진 코드를 테스트하고, 평가하는 방법에 대해서 알아보도록 하겠습니다.

