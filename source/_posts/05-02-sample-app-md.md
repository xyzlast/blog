---
title: 05-02.SampleApplication(2) - 테스트/코드평가
date: 2018-02-01 12:47:40
tags: study,java
---
# SampleApplication(2) - 테스트/코드평가

## Test를 통한 코드 평가

올바른 테스트란, 사람이 개입되어서는 안됩니다. 사람이 눈으로 테스트를 확인하는 경우는, 그것을 보지 않는다면 또는 테스트가 너무나 많아져서 테스트의 정보를 확인할 수 없다면 완전히 무용지물이 되어버리고 맙니다. 그리고, 개발을 해보시면 좀더 느끼시지만 하나를 만드니 다른쪽에서 에러가 발생할 수도 있습니다. 마지막으로 CI 환경에서 자동화된 build를 지원할 수가 없습니다.

따라서, 테스트는 확인 가능하나 완전 자동적인 코드로서 구성이 되어야지 됩니다. 테스트는 크게 2가지로 나눌 수 있습니다. 전에 CI에 대해서 간략하게 설명을 할 때, QA 부서에서 담당하기 전 단계라고 할 수 있는 단위 테스트(Unit TEST)와 QA 부서에서 진행하는 통합 테스트가 존재합니다. 잘 만들어진 개발 조직에서, 개발자는 자신의 단위 테스트를 유지하고 관리할 의무를 가지고 있습니다. 그리고 자신의 코드가 단위적으로는 에러가 발생하지 않는다는 결과를 보여줄 수 있는 방법이 있어야지 됩니다. 이를 위해 java에서는 junit이라는 테스트 도구를 배포하고 있고, 이 테스트 코드를 사용하면 자동화 되고, 확인이 가능한 코드로서 사용할 수가 있습니다.

따라서, 기본 folder구조에 왜 `src/test` 폴더가 존재하는지 알 수 있습니다. 테스트 코드 역시 관리 되어야지 되며, 이를 통해 코드의 변경이 일어났을 때, 그 코드의 문제점을 확인할 수 있어야지 됩니다.

java에서는 `JUnit`이라는 멋진 테스트 툴이 있습니다. 이 툴에 대해서 한번 알아보도록 하겠습니다.

junit을 사용할 때 기억할 annotation 목록입니다.

* @Test : Test method를 지정할 때 사용합니다. Test method는 반드시 public에 return type은 void, input 값은 하나도 없는 형태여야지 됩니다.
* @Before : Test method를 시작하기 전에 반드시 실행될 method입니다.
* @After : Test method를 수행 후, 실행될 method입니다.
* @BeforeClass : 전체 테스트 코드 method가 수행되기 전에 실행됩니다. 반드시 public, static type의 method여야지 됩니다.
* @AfterClass : 전체 테스트 코드 method가 수행된 후에 실행됩니다. 반드시 public, static type의 method여야지 됩니다.

테스트 코드의 annotation을 모두 사용한 테스트 코드입니다. 한번 내용을 확인해보도록 하겠습니다.

```java
public class AppTest {
    @BeforeClass
    public static void beforeClass() {
        System.out.println("#1. BeforeClass");
    }

    @AfterClass
    public static void afterClass() {
        System.out.println("#6. AfterClass");
    }

    @Before
    public void before() {
        System.out.println("#2. Before");
    }

    @After
    public void after() {
        System.out.println("#3. After");
    }

    @Test
    public void test01() {
        System.out.println("#4. Test01");
    }

    @Test
    public void test02() {
        System.out.println("#5. Test02");
    }

    @Test
    public void test03() {
        System.out.println("#6. Test03");
    }
}
```

결과는 다음과 같습니다.

```
#1. BeforeClass
#2. Before
#4. Test01
#3. After
#2. Before
#5. Test02
#3. After
#2. Before
#6. Test03
#3. After
#6. AfterClass
```

신규 테스트를 추가해보도록 하겠습니다. `BookService`에서 우클릭 후, `GoTo > Test`를 통해 `BookServiceTest`를 만듭니다.
그리고, `BookService`를 테스트 하는 코드를 작성해보도록 하겠습니다.

![](/images/05/intellij08.png)

```java
public class BookServiceTest {

    private BookService bookService;

    @Before
    public void setUp() {
        bookService = new BookService();
    }

    private Book generateNewBook() {
        Book newBook = new Book();
        newBook.setName("newName01");
        newBook.setAuthor("newAuthor01");
        newBook.setComment("newComment01");
        newBook.setPublishDate(new Date());
        return newBook;
    }

    @Test
    public void getTest() throws Exception {
        List<Book> allBooks = bookService.getAll();
        if (allBooks.isEmpty()) {
            bookService.add(generateNewBook());
            allBooks = bookService.getAll();
        }

        Book checkBook = allBooks.get(allBooks.size() - 1);
        Book getBook = bookService.get(checkBook.getId());
        Assert.assertEquals(checkBook.getId(), getBook.getId());
    }

    @Test
    public void addTest() throws Exception {
        long preCount = bookService.countAll();
        Book newBook = generateNewBook();
        bookService.add(newBook);
        long afterCount = bookService.countAll();

        Assert.assertEquals("하나 더 추가 되었는지?", preCount + 1, afterCount);
    }

    @Test
    public void deleteTest() throws Exception {
        List<Book> allBooks = bookService.getAll();
        if (allBooks.isEmpty()) {
            bookService.add(generateNewBook());
            allBooks = bookService.getAll();
        }
        Book checkBook = allBooks.get(allBooks.size() - 1);
        bookService.delete(checkBook.getId());

        Book deletedBook = bookService.get(checkBook.getId());
        Assert.assertNull(deletedBook);
    }

    @Test
    public void getAll() throws Exception {
        List<Book> allBooks = bookService.getAll();
        long allCount = bookService.countAll();

        Assert.assertEquals(allBooks.size(), allCount);
    }

    @Test
    public void deleteAll() throws Exception {
        long allCount = bookService.countAll();
        if (allCount <= 10) {
            for (int i = 0 ; i < 10 ; i++) {
                bookService.add(generateNewBook());
            }
        }
        Assert.assertTrue(bookService.countAll() >= 10);
        bookService.deleteAll();
        Assert.assertEquals(bookService.countAll(), 0);
    }
}
```

테스트를 돌리는 방법은 IDE에서 실행하는 방법이 있고, `gradle`을 이용하는 방법이 있습니다. IDE에서는 다음과 같이 실행할 수 있습니다.

![](/images/05/intellij09.png)
![](/images/05/intellij10.png)

그리고, gradle을 통해 다음과 같이 실행할 수 있습니다.

```cmd
gradle test
```

`build/reports/tests/test` 폴더를 확인해보면 다음 파일들을 확인해볼 수 있습니다.

```cmd
.
├── classes
│   ├── AppTest.html
│   └── BookServiceTest.html
├── css
│   ├── base-style.css
│   └── style.css
├── index.html
├── js
│   └── report.js
└── packages
    └── default-package.html
```

brower를 열어서 확인하면 다음과 같은 내용을 확인할 수 있습니다.

![](/images/05/test-report01.png)

테스트가 정상적으로 다 통과가 된 것을 확인할 수 있습니다. 그런데, 테스트를 다 통과했다고 해서 테스트를 잘 작성했는지 확인을 할 수 있을까요?
테스트를 어떻게 하면 잘 작성했다고 할 수 있을까요. 여기에서 `Code Coverage`라는 개념이 나오게 됩니다.

이제 `gradle`에서 `Code Coverage`를 계산하기 위해서 `build.gradle`을 다음과 같이 변경하도록 하겠습니다. `java`에서 `Code Coverage`를 계산하기 위해서는 `jacoco`를 이용합니다.

```groovy
apply plugin: 'java'
apply plugin: 'application'
apply plugin: 'jacoco'

repositories {
    jcenter()
}

dependencies {
    compile 'com.google.guava:guava:23.0'
    compile group: 'org.mariadb.jdbc', name: 'mariadb-java-client', version: '2.2.0'
    testCompile 'junit:junit:4.12'
}
jacoco {
    toolVersion = "0.7.9"
}
jacocoTestReport {
    reports {
        xml.enabled = true
        csv.enabled = false
        html.enabled= true
        html.destination file("${jacoco.reportsDir}/html")
    }
}

test {
    jacoco {
        append = false
        destinationFile = file("$buildDir/jacoco/jacoco.exec")
        classDumpDir = file("$buildDir/jacoco/classpathdumps")
    }
    reports {
        junitXml.enabled = true
        html.enabled = true
    }
    ignoreFailures = true
}
mainClassName = 'App'
```

추가된 후, 다음 gradle 명령어를 통해 테스트를 다시 돌립니다.

```cmd
gradle clean test jacocoTestReport
```

테스트가 종료된 후, `build/reports/jacoco/html`을 에서 `index.html`을 확인하면 다음과 같은 결과들을 볼 수 있습니다.

![](/images/05/jacoco01.png)
![](/images/05/jacoco02.png)
![](/images/05/jacoco03.png)

테스트 결과를 확인 할 수 있으며, 테스트가 통과되지 않은 코드들을 확인 할 수 있습니다.

Test Coverage를 확인하면 자신의 코드가 어떻게 테스트가 되었는지 확인할 수 있습니다.
좋은 테스트를 작성하게 되면, 에러발생상황을 쉽게 시뮬레이션이 가능합니다. 좋은 테스트 코드를 관리해갈 수 있어야지 됩니다.

## 코드 평가

이제 작성되어 있는 코드를 평가해보도록 하겠습니다. 코드에 대한 평가는 주로 정적프로그램분석(static program analysis)를 통해서 이루어집니다. 테스트코드의 경우 동적프로그램분석(dynamic program analysis)에 해당됩니다.

java에서 주로 사용되는 정적프로그램분석방법은 다음 3가지를 뽑습니다.

* PMD
* FindBugs
* CheckStyle

### PMD

정해진 규칙에 따라 소스코드를 검사해주고 이에 대한 결과를 Report 시켜줍니다. PMD는 잠재적인 버그, 사용되지 않는 코드, 너무 복잡한 코드, 중복된 코드를 찾습니다.

`build.gradle`에 다음과 같은 설정을 추가합니다.

```groovy
apply plugin: 'pmd'
pmd {
    ignoreFailures = true
}

pmdMain {
    reports {
        xml.destination = file("${pmd.reportsDir}/pmd.xml")
        html.enabled = true
        xml.enabled = true
    }
}
```

이제 `gradle pmdMain`을 통해 `PMD`결과를 보여줍니다. 지금까지 작성된 코드에서의 PMD Report는 다음과 같은 결과를 보여줍니다.

![](/images/05/pmd.png)

### FindBugs

잠재적 버그를 찾습니다. FindBugs의 경우 source code가 아닌 compile된 class파일을 이용해서 버그를 찾아내기 때문에 compile이 반드시 필요합니다.

`build.gradle`에 다음과 같은 설정을 추가해줍니다.

```groovy
apply plugin: 'findbugs'

findbugs {
    ignoreFailures = true
    toolVersion ='3.0.1'
}

findbugsMain {
    reports {
// HTML,XML 둘중 하나의 Report만 작성 가능합니다.
//        xml.enabled = true
//        xml.destination = file("${findbugs.reportsDir}/findbugs.xml")
        html.enabled = true
        html.destination = file("${findbugs.reportsDir}/findbugs.html")
    }
}
```

이제 `gradle findbugsMain`을 통해 FindBugs결과를 볼 수 있습니다. 지금까지 작성된 코드에서의 FindBugs Report는 다음 결과를 보여줍니다.

![](/images/05/findbugs01.png)
![](/images/05/findbugs02.png)

### CheckStyle

CheckStyle은 코딩 패턴을 확인합니다. 버그를 찾는 코드분석이 아닙니다. class의 이름이나 method의 이름, 코드의 길이등을 확인해서 정해진 규칙대로 코드가 작성되었는지를 확인합니다.
전에 잠시 소개했었던 `secucen-checkstyle.xml`이 이를 정의하고 있습니다.
사내 Redmine: `https://192.168.54.104/attachments/47/secucen-checkstyle.xml`에서 다운받을 수 있습니다.

```xml
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC "-//Puppy Crawl//DTD Check Configuration 1.3//EN" "http://www.puppycrawl.com/dtds/configuration_1_3.dtd">
<module name = "Checker">
    <property name="charset" value="UTF-8"/>
    <property name="severity" value="warning"/>
    <property name="fileExtensions" value="java, properties, xml"/>
    <module name="FileTabCharacter">
        <property name="eachLine" value="true"/>
    </module>
    <module name="TreeWalker">
        <module name="OuterTypeFilename"/>
        <module name="IllegalTokenText">
            <property name="tokens" value="STRING_LITERAL, CHAR_LITERAL"/>
            <property name="format" value="\\u00(09|0(a|A)|0(c|C)|0(d|D)|22|27|5(C|c))|\\(0(10|11|12|14|15|42|47)|134)"/>
            <property name="message" value="Consider using special escape sequence instead of octal value or Unicode escaped value."/>
        </module>
        <module name="AvoidEscapedUnicodeCharacters">
            <property name="allowEscapesForControlCharacters" value="true"/>
            <property name="allowByTailComment" value="true"/>
            <property name="allowNonPrintableEscapes" value="true"/>
        </module>
        <module name="LineLength">
            <property name="max" value="120"/>
            <property name="ignorePattern" value="^package.*|^import.*|a href|href|http://|https://|ftp://"/>
        </module>
        <module name="AvoidStarImport">
            <property name="excludes" value="java.io,java.net,java.lang.Math,org.assertj.core.api.Assertions,org.mockito.Mockito,org.springframework.test.web.servlet.result.MockMvcResultHandlers,org.springframework.test.web.servlet.setup.MockMvcBuilders,org.hamcrest.CoreMatchers,org.junit.Assert,org.springframework.test.web.servlet.result.MockMvcResultMatchers"/>
            <property name="allowClassImports" value="true"/>
            <property name="allowStaticMemberImports" value="false"/>
        </module>
        <module name="OneTopLevelClass"/>
        <module name="NoLineWrap"/>
        <module name="EmptyBlock">
            <property name="option" value="TEXT"/>
            <property name="tokens" value="LITERAL_TRY, LITERAL_FINALLY, LITERAL_IF, LITERAL_ELSE, LITERAL_SWITCH"/>
        </module>
        <module name="NeedBraces"/>
        <module name="RightCurly">
            <property name="id" value="RightCurlySame"/>
            <property name="tokens" value="LITERAL_TRY, LITERAL_CATCH, LITERAL_FINALLY, LITERAL_IF, LITERAL_ELSE, LITERAL_DO"/>
        </module>
        <module name="RightCurly">
            <property name="id" value="RightCurlyAlone"/>
            <property name="option" value="alone"/>
            <property name="tokens" value="CLASS_DEF, METHOD_DEF, CTOR_DEF, LITERAL_FOR, LITERAL_WHILE, STATIC_INIT, INSTANCE_INIT"/>
        </module>
        <module name="WhitespaceAround">
            <property name="allowEmptyConstructors" value="true"/>
            <property name="allowEmptyMethods" value="true"/>
            <property name="allowEmptyTypes" value="true"/>
            <property name="allowEmptyLoops" value="true"/>
            <message key="ws.notFollowed"
             value="WhitespaceAround: ''{0}'' is not followed by whitespace. Empty blocks may only be represented as '{}' when not part of a multi-block statement (4.1.3)"/>
             <message key="ws.notPreceded"
             value="WhitespaceAround: ''{0}'' is not preceded with whitespace."/>
        </module>
        <module name="OneStatementPerLine"/>
        <module name="MultipleVariableDeclarations"/>
        <module name="ArrayTypeStyle"/>
        <module name="MissingSwitchDefault"/>
        <module name="FallThrough"/>
        <module name="UpperEll"/>
        <module name="ModifierOrder"/>
        <module name="EmptyLineSeparator">
            <property name="allowNoEmptyLineBetweenFields" value="true"/>
        </module>
        <module name="SeparatorWrap">
            <property name="id" value="SeparatorWrapDot"/>
            <property name="tokens" value="DOT"/>
            <property name="option" value="nl"/>
        </module>
        <module name="SeparatorWrap">
            <property name="id" value="SeparatorWrapComma"/>
            <property name="tokens" value="COMMA"/>
            <property name="option" value="EOL"/>
        </module>
        <module name="PackageName">
            <property name="format" value="^[a-z]+(\.[a-z][a-z0-9]*)*$"/>
            <message key="name.invalidPattern"
             value="Package name ''{0}'' must match pattern ''{1}''."/>
        </module>
        <module name="TypeName">
            <message key="name.invalidPattern"
             value="Type name ''{0}'' must match pattern ''{1}''."/>
        </module>
        <module name="MemberName">
            <property name="format" value="^[a-z][a-z0-9][a-zA-Z0-9]*$"/>
            <message key="name.invalidPattern"
             value="Member name ''{0}'' must match pattern ''{1}''."/>
        </module>
        <module name="ParameterName">
            <property name="format" value="^[a-z]([a-z0-9][a-zA-Z0-9]*)?$"/>
            <message key="name.invalidPattern"
             value="Parameter name ''{0}'' must match pattern ''{1}''."/>
        </module>
        <module name="CatchParameterName">
            <property name="format" value="^[a-z]([a-z0-9][a-zA-Z0-9]*)?$"/>
            <message key="name.invalidPattern"
             value="Catch parameter name ''{0}'' must match pattern ''{1}''."/>
        </module>
        <module name="LocalVariableName">
            <property name="tokens" value="VARIABLE_DEF"/>
            <property name="format" value="^[a-z]([a-z0-9][a-zA-Z0-9]*)?$"/>
            <message key="name.invalidPattern"
             value="Local variable name ''{0}'' must match pattern ''{1}''."/>
        </module>
        <module name="ClassTypeParameterName">
            <property name="format" value="(^[A-Z][0-9]?)$|([A-Z][a-zA-Z0-9]*[T]$)"/>
            <message key="name.invalidPattern"
             value="Class type name ''{0}'' must match pattern ''{1}''."/>
        </module>
        <module name="MethodTypeParameterName">
            <property name="format" value="(^[A-Z][0-9]?)$|([A-Z][a-zA-Z0-9]*[T]$)"/>
            <message key="name.invalidPattern"
             value="Method type name ''{0}'' must match pattern ''{1}''."/>
        </module>
        <module name="InterfaceTypeParameterName">
            <property name="format" value="(^[A-Z][0-9]?)$|([A-Z][a-zA-Z0-9]*[T]$)"/>
            <message key="name.invalidPattern"
             value="Interface type name ''{0}'' must match pattern ''{1}''."/>
        </module>
        <module name="NoFinalizer"/>
        <module name="GenericWhitespace">
            <message key="ws.followed"
             value="GenericWhitespace ''{0}'' is followed by whitespace."/>
             <message key="ws.preceded"
             value="GenericWhitespace ''{0}'' is preceded with whitespace."/>
             <message key="ws.illegalFollow"
             value="GenericWhitespace ''{0}'' should followed by whitespace."/>
             <message key="ws.notPreceded"
             value="GenericWhitespace ''{0}'' is not preceded with whitespace."/>
        </module>
        <module name="Indentation"/>
        <!--<module name="CommentsIndentation"/>-->
        <module name="AbbreviationAsWordInName">
            <property name="ignoreFinal" value="false"/>
            <property name="allowedAbbreviationLength" value="1"/>
        </module>
        <module name="OverloadMethodsDeclarationOrder"/>
        <module name="VariableDeclarationUsageDistance"/>
        <module name="MethodParamPad"/>
        <module name="ParenPad"/>
        <module name="OperatorWrap">
            <property name="option" value="NL"/>
            <property name="tokens" value="BAND, BOR, BSR, BXOR, DIV, EQUAL, GE, GT, LAND, LE, LITERAL_INSTANCEOF, LOR, LT, MINUS, MOD, NOT_EQUAL, PLUS, QUESTION, SL, SR, STAR, METHOD_REF "/>
        </module>
        <module name="AnnotationLocation">
            <property name="id" value="AnnotationLocationMostCases"/>
            <property name="tokens" value="CLASS_DEF, INTERFACE_DEF, ENUM_DEF, METHOD_DEF, CTOR_DEF"/>
        </module>
        <module name="AnnotationLocation">
            <property name="id" value="AnnotationLocationVariables"/>
            <property name="tokens" value="VARIABLE_DEF"/>
            <property name="allowSamelineMultipleAnnotations" value="true"/>
        </module>
        <module name="NonEmptyAtclauseDescription"/>
        <module name="JavadocTagContinuationIndentation"/>
        <module name="SummaryJavadoc">
            <property name="forbiddenSummaryFragments" value="^@return the *|^This method returns |^A [{]@code [a-zA-Z0-9]+[}]( is a )"/>
        </module>
        <module name="JavadocParagraph"/>
        <module name="AtclauseOrder">
            <property name="tagOrder" value="@param, @return, @throws, @deprecated"/>
            <property name="target" value="CLASS_DEF, INTERFACE_DEF, ENUM_DEF, METHOD_DEF, CTOR_DEF, VARIABLE_DEF"/>
        </module>
        <module name="JavadocType">
            <property name="severity" value="ignore"/>
        </module>
        <module name="JavadocMethod">
            <property name="severity" value="ignore"/>
        </module>
        <module name="JavadocVariable">
            <property name="severity" value="ignore"/>
        </module>
        <module name="MethodName">
            <property name="format" value="^[a-z][a-z0-9][a-zA-Z0-9_]*$"/>
            <message key="name.invalidPattern"
             value="Method name ''{0}'' must match pattern ''{1}''."/>
        </module>
        <module name="EmptyCatchBlock">
            <property name="exceptionVariableName" value="expected"/>
        </module>
    </module>
</module>
```

이제 `build.gradle`에 checkstyle을 추가할 차례입니다.

```groovy
apply plugin: 'checkstyle'

checkstyle {
    toolVersion = '8.2'
    ignoreFailures = true
    configFile = rootProject.file("secucen-checkstyle.xml")
    reportsDir = file("${buildDir}/checkstyle-output")
}
```

`gradle checkstyleMain`을 통해 결과를 확인할 수 있습니다.

```cmd
> Task :checkstyleMain
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/App.java:8: Line is longer than 120 characters (found 136). [LineLength]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:7: Line is longer than 120 characters (found 124). [LineLength]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:10:55: '(' is preceded with whitespace. [MethodParamPad]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:12: Line is longer than 120 characters (found 124). [LineLength]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:24: Line is longer than 120 characters (found 127). [LineLength]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:27:55: '(' is preceded with whitespace. [MethodParamPad]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:29: Line is longer than 120 characters (found 137). [LineLength]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:43: Line is longer than 120 characters (found 131). [LineLength]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:46:55: '(' is preceded with whitespace. [MethodParamPad]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:58: Line is longer than 120 characters (found 121). [LineLength]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:61:55: '(' is preceded with whitespace. [MethodParamPad]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:63: Line is longer than 120 characters (found 124). [LineLength]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:83: Line is longer than 120 characters (found 124). [LineLength]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:112:55: '(' is preceded with whitespace. [MethodParamPad]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:118: Distance between variable 'count' declaration and its first usage is 4, but allowed 3.  Consider making that variable final if you still need to store its value in advance (before method calls that might have side effects on the original value). [VariableDeclarationUsageDistance]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:127: Line is longer than 120 characters (found 121). [LineLength]
[ant:checkstyle] [WARN] /Users/secucen/dev/code/secucen-study/step01/src/main/java/BookService.java:130:55: '(' is preceded with whitespace. [MethodParamPad]
```

꽤나 많은 것들이 걸리는 것을 볼 수 있습니다.

## SonarQube

이런 정적분석툴 결과를 수집하고, 관리하는 application입니다. 이 툴은 특별한 설명이 없이 바로 실행해서 확인해보도록 하겠습니다.
SonarQube를 통해서 추가합니다.

```groovy
apply plugin: 'org.sonarqube'
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath "org.sonarsource.scanner.gradle:sonarqube-gradle-plugin:2.5"
    }
}
```

그리고 `gradle.properties`파일을 추가합니다. `gradle.properties`파일은 다음과 같습니다.

```properties
systemProp.sonar.host.url=http://192.168.94.18:9000
systemProp.sonar.login=cbb2cca03f69cac6bea892228129433bb0240bb6
systemProp.sonar.projectKey=sampleApp01-이름을넣어주세요
```

`gradle clean sonarqube`을 통해서 실행합니다. `http://192.168.94.18:9000`에서 결과를 확인해볼 수 있습니다. 자신의 Project의 이름을 보고 확인해보시면 좋습니다.

![](/images/05/sonarqube.png)

정적 코드 분석에 대해서 한번 정리해보도록 하겠습니다.

||FingBugs|PMD|CheckStyle|
|---|---|---|---|
|목적|잠재적버그찾기(java파일이 아닌 class파일 이용)|잠재적인 문제들, 버그가능성이 있는 부분들, 사용되지 않았거나 최적화되지 않은 코드를 검색|java code를 style.xml을 이용해서 규칙 위반 여부를 체크|
|장점|실제 결함을 잘 찾아줌</br>정확성이 높음</br>속도가 빠름|실제 결함을 찾는 경우가 많음</br>나쁜 패턴을 잘 찾습니다.|정해진 코딩 규약에 위반되는 것을 찾아줍니다.|
|단점|compile된 class를 이용하기 때문에 build과정이 필수적입니다.|속도가 늦습니다.|실제 버그를 찾는 과정이 아닙니다.|
|규칙수|408|234|xml에 따라 다름|

이 3가지 모두를 다 사용할 수 있는 것이 좋습니다. 자신의 문제를 객관적으로 확인해볼 수 있습니다.
