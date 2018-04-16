---
title: 15. Spring View
date: 2018-04-10 08:17:00
tags: spring, view, controller, pdf, json
---

# 15. Spring View

View는 사용자에게 최종적으로 보이는 영역입니다. 이 영역은 우리가 일반적으로 보는 Html이 될 수도 있고, Mobile App에게는 JSON format의 API 결과가 될 수도 있습니다.
기본적으로 Spring Servlet/MVC를 기반으로 동작하게 됩니다. View 계층 또는 Presentation 계층이라고 합니다.

Controller가 return한 결과에 대한 표현을 담당하는 영역으로 Controller가 넘겨주는 ModelAndView가 View에서 핵심 객체라고 할 수 있습니다.

기본적인 동작은 DispatcherServlet이 ModelAndView에서 넘어온 값 ViewResolver를 통해서 해석한 결과를 사용자에게 보내줍니다. ViewResolver는 기본적으로 jsp를 기반으로 하고 있고, jsp 이외에 모든 web context가 될 수 있습니다.

## View의 구조

### View의 기본 구조

기본적으로 Spring의 org.springframework.web.servlet.View interface를 구현한 객체를 View로 표현이 가능합니다. 일반적으로 View는 Spring 내부의 View를 확장하거나, pdf, rss, excel과 같은 다른 형태의 View를 표현하기 위해서 class를 확장해서 사용합니다. 또는 외부 View engine을 이용한 View의 표현 역시 가능합니다. 여기서 외부 View engine을 이용하는 것은 `velocity`, `freemarker`, `tiles`와 같은 외부의 View engine과의 연결 interface 및 설정을 구현함으로서 가능합니다.

### View의 기본 동작

Controller가 작업을 마친 후, View정보를 ModelAndView 객체에 담아서 보내주는 DispatcherServlet에 보내주는 것이 기본 동작입니다. 이에 대한 방법은 Spring @MVC는 두가지를 제공하고 있습니다. 첫번째는 View interface를 구현한 객체를 보내주는 방법이고, 다른 하나는 View의 이름만을 보내는 방법입니다. 첫번째 방법의 경우에는 View interface의 구현 방법에 따라 다양한 View를 표시하게 됩니다. DispatcherServlet과 Controller간에는 어떠한 동작도 존재하지 않습니다. 그렇지만 View의 이름만 보내주는 방식은 좀 다르게 동작하게 됩니다.

View의 이름으로 표현되는 논리적인 View를 실질적인 View interface를 구현한 객체로 변경하는 작업이 필요하게되는데요. 이를 담당하는 객체를 ViewResolver라고 합니다.

Spring에서 제공하는 주요 ViewResolver는 다음과 같습니다.

|ViewResolver 구현 class|Description|
|---|---|
|InternalResourceViewResolver|View Name에서 jsp나 tiles 연동을 위한 View 객체를 반환|
|BeanNameViewResolver|Bean name을 기준으로 View 객체를 반환|
|ResourceBundlleViewResolver|View 이름과 View 객체간의 mapping 정보를 저장하기 위해서 Resource 파일을 이용|
|XmlViewResolver|View 이름과 View 객체간의 mapping 정보를 저장하기 위해서 xml 파일을 이용|

기본적으로 만들어지는 ViewResolver의 interface는 다음과 같습니다.

```java
public interface ViewResolver {
    View resolveViewName(String viewName, Locale locale) throws Exception;
}
```

다국어 지원을 위한 locale 정보와 View 이름을 이용한 View 객체를 얻어내는 간단한 interface만을 가지고 있습니다. 이를 이용해서 새로운 ViewResolver를 만드는 것 역시 가능합니다.
구성되는 View는 다음과 같은 interface를 가지고 있습니다.

```java
public interface View {
    String getContentType();
    void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

들어오게되는 model을 어떻게 render하는지에 대한 interface 선언을 볼 수 있습니다.

## View의 표현법 - HTML

우리가 일반적으로 보는 브라우져를 이용해서 볼 수 있는 화면을 표시하는 방법입니다.

### JSP & JSTL(JSP Standard Tag Library)

기본적인 jsp view와 JSTL을 사용하기 위한 View입니다.

jsp와 jstl을 사용하기 위해서는 특별한 View 설정은 필요없지만, 기본적으로 Spring에서 요구되는 View는 WEB-INF 폴더 안에 view file들을 위치시키고, client에서 직접 접근할 수 없도록 하는 것이 일반적입니다. 이때는 기본적으로 `InternalResourceViewResolver`를 사용해서 View 파일의 위치를 설정하도록 합니다.

*build.gradle*

```groovy
compile group: 'javax.servlet.jsp.jstl', name: 'jstl', version: '1.2'
```

*applicationContext.xml*

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
  <property name="prefix" value="/WEB-INF/views/" />
  <property name="suffix" value=".jsp" />
  <property name="contentType" value="text/html; charset=UTF-8" />
  <property name="order" value="1" />
</bean>
```

*@Configuration*

```java
@Bean
public ViewResolver viewResolver() {
    InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
    internalResourceViewResolver.setPrefix("/WEB-INF/jsp/");
    internalResourceViewResolver.setSuffix(".jsp");
    internalResourceViewResolver.setOrder(1);
    internalResourceViewResolver.setViewClass(JstlView.class);
    return internalResourceViewResolver;
}
```

JSTL은 다양한 tag를 지원하고 있습니다. 이는 기존 jsp에서 <%= %>에서 빈번하게 만들어지는 code들을 단순화하고 읽기 편한 간단한 언어로 만들기 위해서 제공되고 있습니다.  jsp는 다음과 같은 tag를 선언함으로서 사용할 수 있습니다.

```jsp
<%@taglib prefix="c"uri="http://java.sun.com/jsp/jstl/core"%>
```

* `c:out`  - 객체를 출력한다.

```jsp
<c:out value="${name}"/><!--기본 문법-->
<c:out value="${age}" default="Null or empty"/> <!--  default 속성 : 값이 없을때 기본출력값을 정의-->
<c:out value="${name}" escapeXml="false"/><!-- escapeXml 속성 : 기본적으로 XML 문자를 escape하는데 Escape 하지 않으려면  false로 설정-->
```

* `c:if` - 조건문

```jsp
<!--기본문법-->
<c:if test="${조건}">
    조건 만족시 이 영역을 수행
</c:if>
```

jstl에서 if문은 조금 문제가 있습니다. 그건 else가 존재하지 않는 문제인데요. 우리가 주로 사용하는 if~else 문을 갖추기 위해서는 아래의 choose문을 사용해야지 됩니다.

* `c:choose, c:when, c:otherwise` - switch 문

```jsp
<!--기본문법-->
<c:choose>
    <c:when test="${value == 1}">
        value가 1이면 이 영역을 수행
    </c:when>
    <c:when test="${value == 2}">
        value가 2이면 이 영역을 수행
    </c:when>
    <c:otherwise>
        value가 1,2가 아니면 이 영역을 수행(기본값)
    </c:otherwise>
</c:choose>
```

* `c:foreach` - 반복문

```jsp
<!--기본문법(0~9까지 출력)-->
<c:forEach begin="0" end="9" var="i">
    <c:out value="${i}"/>
</c:forEach>
<!-- step 속성 : 정의된 수만큼 증가를 시킨다.(1,3,5,7,9출력)-->
<c:forEach var="test" begin="1" end="10" step="2" >
     <b>${test }</b>
</c:forEach>
```

* `c:forTokens` - 구분자로 반복문

```jsp
<!--기본문법(변수를 ','로 구분하여 출력한다. )-->
<c:forTokens var="alphabet" items="a,b,c,d,e,f,g,h,i,j,k" delims="," varStatus="idx" >
     <b>${alphabet }</b>
</c:forTokens>
```

* `c:url, c:param`  - URL을 처리

```jsp
<c:url value="index.jsp"/>
<!-- value의 속성값이 /로 시작하면 컨텍스트를 포함한다-->
<!--Context가 Root이라면 /Root/index.jsp로 출력-->
<c:url value="/index.jsp"/>
<!--context 속성 : 다른 컨텍스트로 출력하고자할때 사용-->
<!-- /newRoot/index.jsp로 출력-->
<c:url value="/index.jsp" context="/newRoot"/>
```

* `c:import` - JSP파일을 인클루드한다.

```jsp
<!-- partialView.jsp의 내용을 출력 -->
<c:import url="partialView.jsp"/>
<!-- charEncoding 속성:   인코딩에 문제가 있을 시 사용 -->
<c:import url="UserForm.jsp" charEncoding="UTF-8"/>
```

* `c:redirect` - 리다이렉션

```jsp
<!-- 기본 문법 -->
<c:redirect url="http://kkams.net"/>
```

* `c:catch` - 예외 발생시 처리

```jsp
<!-- 기본 문법 -->
<c:catch var="err">
    <%=10 / 0%> <!--0으로 나누면 에러 발생-->
</c:catch>
<!--에러 출력-->
<c:out value="${err }"/>
```

다양해보이지만, 상당히 빈약한 문법과 코드를 가지고 있는 것이 JSTL입니다. 자주 사용되기도 하고, 일단 표준이기 때문에 몰라서는 안되는 View 표현 방법입니다. 앞서 Controller에서 보내지는 Model의 값을 JSTL을 이용해서 표현하게 되는 것이 일반적입니다.

```java
@RequestMapping(value = "/book/list", method = RequestMethod.GET)
public ModelAndView getBookList() {
    List<Book> books = new ArrayList<>();
    for (int i = 0 ; i < 10 ; i++) {
        String name = String.format("책%d", i);
        String author = String.format("저자%d", i);
        String status = "상태";
        books.add(new Book(name, author, status));
    }
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.setViewName("book/list");
    modelAndView.addObject("books", books);
    return modelAndView;
}
```

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form"%>
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions"%>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
    <link href="/node_modules/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet" />
    <script src="/node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
</head>
<body>
    <h2>jstl test code 입니다.</h2>
    <div class="example-form">
    <h3>c:forEach example code</h3>
    <table class="table table-striped table-bordered table-hover">
        <thead>
            <tr>
                <th>이름</th>
                <th>상태</th>
                <th>저자</th>
            </tr>
        </thead>

        <tbody>
            <c:forEach var="book" items="${books}">
                <tr>
                    <td>${book.name}</td>
                    <c:set var="status" value="일반" />
                    <c:choose>
                        <c:when test="${book.status eq 'NORMAL'}">
                            <c:set var="status" value="일반" />
                        </c:when>
                        <c:when test="${book.status eq 'RENTNOW'}">
                            <c:set var="status" value="대여중" />
                        </c:when>
                        <c:otherwise>
                            <c:set var="status" value="분실중" />
                        </c:otherwise>
                    </c:choose>
                    <td>${status }</td>
                    <td>${book.author}</td>
                </tr>
            </c:forEach>
        </tbody>
    </table>
</div>
</body>
</html>
```

### JSTL fmt를 이용한 데이터의 가공

java의 원 데이터를 html로 표시할 때, 우리는 자주 데이터의 모양을 바꿔야지 되는 일들이 발생됩니다. 특히 통화량과 날짜, 소숫점의 표시에서 가장 일반적이라고 할 수 있습니다. 자리수 단락에서 ","를 추가한다던지, 소숫점 이하의 자리수를 어떻게 표시를 하는지에 대한 문제는 Controller와 같은 기술적인 issue와는 별개로 사용자들에게는 '모든 것'이 될 수 있는 문제입니다. 이런 일은 수치데이터를 직접 바꾸는 것이 아니라 수치의 표시를 어떻게 해줄 것인지에 대한 내용입니다. View에서 이런 일들을 해주는 것이 가장 좋습니다.

이 부분에 대해서는 issue가 있습니다. View에서 특정 데이터를 빨간색으로 보여야지 되는 action에 대한 처리를 무엇으로 보느냐에 대한 논쟁입니다. **이것은 보는 방법을 바꾸는 일**이기 때문에 View에서 봐야지 된다는 의견과 보는 방법이 아닌 이것 역시 Business Logic으로 보는 견해도 있습니다. BL이기 때문에 Server Code에서 구동이 되고 테스트가 가능한 방법으로 만들어야지 된다는 것이지요. 가장 좋은 위치는 Controller 영역으로 이야기하고 있습니다. 보여지는 Model을 변경하는 것이기 때문에 가장 좋은 Layer이니까요. 개인적인 의견으로는 View에 대한 영역은 모두 View에 넘기는 것이 좋지 않나. 라는 생각입니다. 저는 개인적으로는 전자의 의견에 좀더 한표를 주고 있는 편입니다. 이 부분에 대해서는 개발자들의 취향과 그 프로젝트의 PM에 따라서 많이 바뀌게 될 내용이라고 생각됩니다.

지금 소개하는 JSTL fmt의 경우에는 View Layer, 즉 JSP에서 보이는 방법을 바꿔주는 방법입니다.

선언 방법은 다음과 같습니다.

```jsp
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt"%>
```

선언된 데이터는 다음과 같이 사용이 가능합니다.

```jsp
<div class="example-form">
    <fieldset>
        <legend>FMT examples - Date</legend>
        <ul>
            <li><fmt:formatDate value="${date}" type="DATE" pattern="yyyy/MM/dd" /></li>
            <li><fmt:formatDate value="${date}" type="DATE" pattern="yyyy년 M월 dd일" /></li>
        </ul>
    </fieldset>
    <br />
    <fieldset>
        <legend>FMT example - Number</legend>
        <ul>
            <li>orginal : ${number}</li>
            <li><fmt:formatNumber value="${number}" groupingUsed="true" currencySymbol=","/></li>
            <li><fmt:formatNumber value="${number}" minFractionDigits="5"/></li>
            <li><fmt:formatNumber value="${number}" type="CURRENCY"/></li>
            <li><fmt:formatNumber value="234.3" pattern="△#,##0.00; ▼#,##0.00" /></li>
            <li><fmt:formatNumber value="-1234.56" pattern="△#,##0.00; ▼#,##0.00" /></li>
            <li><fmt:formatNumber value="0.99" type="percent"/></li>
        </ul>
    </fieldset>
</div>
```

![](/images/15/01.png)

### Freemarker

Freemarker는 `JSP Tag`보다 나은 가독성과 속도를 목적으로 하고 있는 ViewEngine입니다. 다음은 `Freemarker`로 만든 `loop`문입니다.

```ftl
<#list books as book>
  <tr>
    <td>${book.name}</td>
    <#if book.status == "NORMAL">
        <td>일반</td>
    <#elseif book.status == "RENTNOW">
        <td>대여중</td>
    <#else>
        <td>분실</td>
    </#if>
    <td>${book.author}</td>
  </tr>
</#list>
```

기존 `jsp`에서의 문과 비교해보면 가독성이 더 높은 것을 알 수 있습니다.

```jsp
<c:forEach var="book" items="${books}">
    <tr>
        <td>${book.name}</td>
        <c:set var="status" value="일반" />
        <c:choose>
            <c:when test="${book.status eq 'NORMAL'}">
                <c:set var="status" value="일반" />
            </c:when>
            <c:when test="${book.status eq 'RENTNOW'}">
                <c:set var="status" value="대여중" />
            </c:when>
            <c:otherwise>
                <c:set var="status" value="분실중" />
            </c:otherwise>
        </c:choose>
        <td>${status}</td>
        <td>${book.author}</td>
    </tr>
</c:forEach>
```

`if`, `foreach`와 같은 제어문이 더 좋습니다. 그리고 기본적으로 `HTML`과 거의 동일한 문법을 가지고 있기 때문에 좀 더 사용하기 편한 면도 있습니다.

`freemarker`를 사용하기 위한 `dependency`와 `@Configuration`은 다음과 같습니다.

```groovy
compile group: 'org.freemarker', name: 'freemarker', version: '2.3.23'
```

```java
@Bean(name = "freemarkerConfig")
public FreeMarkerConfigurer freemarkerConfig() {
    FreeMarkerConfigurer freemarkerConfigurer = new FreeMarkerConfigurer();
    freemarkerConfigurer.setTemplateLoaderPath("/WEB-INF/fmt/");
    freemarkerConfigurer.setDefaultEncoding("UTF-8");
    freemarkerConfigurer.setPreferFileSystemAccess(false);
    return freemarkerConfigurer;
}

@Bean(name = "freeMarkerViewResolver")
public FreeMarkerViewResolver freemarkerViewResolver() {
    FreeMarkerViewResolver viewResolver = new FreeMarkerViewResolver();
    viewResolver.setExposeSpringMacroHelpers(true);
    viewResolver.setExposeRequestAttributes(true);
    viewResolver.setContentType("text/html; charset=UTF-8");
    viewResolver.setCache(false);
    viewResolver.setSuffix(".ftl");
    return viewResolver;
}
```

### tiles

Tiles는 기본적으로 JSTL을 이용하지만, JSTL에서 중복되는 html code를 최소화하기 위해서 만들어진 Template Engine입니다. Tile는 기본적인 Html View 가 Composite View Pattern으로 동작할 때, 중복되는 코드들을 공통 요소로 뽑아서 이용가능합니다. 다음은 Tile에 대한 기본 개념의 정의입니다.

![](/images/15/02.png)

일반적인 Html Page의 기본 구조입니다. 이 구조에서 페이지가 바뀌게 된다면 Body부분의 Content는 계속해서 바뀌게 되지만, Menu, Header, Footer들은 크게 바뀌지 않는 것이 일반적입니다. 따라서, 각 부분을 재사용할 수 있다면 코드의 양은 매우 줄어들고, 유지보수에 용의함을 알 수 있습니다.

tiles역시 tag를 제공하고 있으며, `xml`을 통해서 view를 정의합니다.

먼저, 다음과 같이 layout을 만들어줍니다. `layout`은 `insertAttribute`를 갖고, 이는 각 `Component`를 의미합니다.

```jsp
<%@ taglib prefix="tiles" uri="http://tiles.apache.org/tags-tiles"%>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title><tiles:insertAttribute name="title"/></title>
  <link href="/node_modules/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet" />
  <script src="/node_modules/jquery/dist/jquery.min.js"></script>
  <script src="/node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
</head>
<body>
    <tiles:insertAttribute name="content"/>
</body>
</html>
```

기본적인 content의 attribute를 지정하고, 그 attribute에 원하는 값을 넣어주는 것이 가능합니다. attribute는 String 문자열 또는 jsp 파일이 될 수 있습니다.  이러한 attribute와 view name을 설정하기 위해서 tiles는 설정 xml을 가지게 되는데, 이 xml은 다음과 같이 구성될 수 있습니다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE tiles-definitions PUBLIC
       "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN"
       "http://tiles.apache.org/dtds/tiles-config_3_0.dtd">
<tiles-definitions>
    <definition name="main-layout" template="/WEB-INF/tiles/layouts/main-layout.jsp">
        <put-attribute name="title" value=""/>
        <put-attribute name="content" value=""/>
    </definition>
    <definition name="book/list" extends="main-layout">
        <put-attribute name="title" value="BOOK LIST"/>
        <put-attribute name="content" value="/WEB-INF/tiles/book/list.jsp"/>
    </definition>
    <definition name="book/add" extends="main-layout">
        <put-attribute name="title" value="Add BOOK"/>
        <put-attribute name="content" value="/WEB-INF/tiles/book/add.jsp"/>
    </definition>
</tiles-definitions>
```

여기서 각각의 definition name은 ModelAndView의 view name으로 이용됩니다. extends로 선언된 View를 Template로 이용하고, 각각의 View의 조각(tile)을 붙이는 형식으로 사용할 수 있습니다.

여기에서 사용될 `book/list.jsp` 내용은 다음과 같이 구성될 수 있습니다.

```jsp
<div>
    <table class="table table-striped table-bordered table-hover">
        <thead>
            <tr>
                <th>이름</th>
                <th>상태</th>
                <th>저자</th>
            </tr>
        </thead>

        <tbody>
            <c:forEach var="book" items="${books}">
                <tr>
                    <td>${book.name}</td>
                    <c:set var="status" value="일반" />
                    <c:choose>
                        <c:when test="${book.status eq 'NORMAL'}">
                            <c:set var="status" value="일반" />
                        </c:when>
                        <c:when test="${book.status eq 'RENTNOW'}">
                            <c:set var="status" value="대여중" />
                        </c:when>
                        <c:otherwise>
                            <c:set var="status" value="분실중" />
                        </c:otherwise>
                    </c:choose>
                    <td>${status }</td>
                    <td>${book.author}</td>
                </tr>
            </c:forEach>
        </tbody>
    </table>
</div>
```

단순한 HTML의 경우에는 차이가 크지 않지만, HTML이 커질수록 관리면에서 매우큰 장점을 가지고 있습니다.
`tiles`를 사용하기 위한 `dependency`와 `@Configuration`은 다음과 같습니다.

```groovy
compile group: 'org.apache.tiles', name: 'tiles-jsp', version: '3.0.8'
```

```java
@Bean
public TilesConfigurer tilesConfigurer() {
    TilesConfigurer tilesConfigurer = new TilesConfigurer();
    tilesConfigurer.setDefinitions("/WEB-INF/tiles-configs.xml");
    return tilesConfigurer;
}

@Bean
public TilesViewResolver tilesViewResolver() {
    TilesViewResolver tilesViewResolver = new TilesViewResolver();
    tilesViewResolver.setOrder(1);
    return tilesViewResolver;
}
```

### 또다른 View Template Engine 들..

`Sitemesh`, `thymeleaf`, `Velocity`, `Pebble` 등의 다양한 `View Template Engine`이 있습니다.
프로젝트를 시작할 때, 어떤 것이 가장 좋은지 비교해보는 노력이 필요합니다.

예전에는 `View Template Engine`간의 속도차가 분명히 존재했습니다. 그러나, 지금 `java` 버젼이 높아진 상태에서는 `View Template Engine`의 속도는 거의 차이가 없습니다. 그러나 `java 6`이전 버젼의 경우에는 몇몇 `View Template`간의 속도차가 분명히 존재합니다.

개발자들의 숙련도와 가장 쉬운것이 뭘지를 고민해서 처리하는 것이 좋습니다.

## View의 표현법 - none HTML

우리가 일반적으로 보는 브라우져 이외의 방법들로 보여지는 방법입니다. 그럼 다른 타입의 View는 어떤 것들이 있을까요?
먼저, File(excel, pdf) download가 있습니다. 그리고, 최근에는 json 형태로 객체를 표현하여 `API`를 제공하는 것 역시 `View`를 통해서 진행됩니다.

### Excel

`Spring`에서는 Excel File을 위해 `Apache POI`를 이용합니다.

```groovy
compile group: 'org.apache.poi', name: 'poi', version: '3.17'
compile group: 'org.apache.poi', name: 'poi-ooxml', version: '3.17'
```

Excel 파일을 생성하기 위해서, Spring에서는 `AbstractXlsxView`를 제공합니다. 선언은 다음과 같이 구성되어 있습니다.

```java
public abstract class AbstractXlsxView extends AbstractXlsView {
    public AbstractXlsxView() {
        setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
    }
    @Override
    protected Workbook createWorkbook(Map<String, Object> model, HttpServletRequest request) {
        return new XSSFWorkbook();
    }
}
```

이제 이 class를 상속받아 다음과 같이 구성이 가능합니다.

```java
@Component
public class BookListExcelView extends AbstractXlsxView {
    @Override
    protected void buildExcelDocument(Map<String, Object> model, Workbook workbook,
                                      HttpServletRequest request, HttpServletResponse response) throws Exception {
        SimpleDateFormat fileFormat = new SimpleDateFormat("yyyyMMdd");
        String fileName = "책리스트" + "-" + fileFormat.format(new Date()) + ".xlsx";
        response.setHeader("Content-Disposition", String.format("attachment; filename=\"%s\"", fileName));
        Sheet sheet = workbook.createSheet("책목록");

        Row header = sheet.createRow(0);
        header.createCell(0).setCellValue("이름");
        header.createCell(1).setCellValue("저자");
        header.createCell(2).setCellValue("상태");

        @SuppressWarnings("unchecked")
        List<Book> books = (List<Book>) model.get("books");

        int row = 1;
        for (Book book : books) {
            Row rowCell = sheet.createRow(row);
            rowCell.createCell(0).setCellValue(book.getName());
            rowCell.createCell(1).setCellValue(book.getAuthor());
            rowCell.createCell(2).setCellValue(book.getStatus());
            row++;
        }
    }
}
```

이제 구성된 `View`를 `Controller`에서 다음과 같이 사용할 수 있습니다.

```java
@Controller
public class BookController {
    private final BookListExcelView bookListExcelView;

    @Autowired
    public BookController(BookListExcelView bookListExcelView) {
        this.bookListExcelView = bookListExcelView;
    }

    @GetMapping(value = "/book/excel")
    public ModelAndView getBookExcel() {
        List<Book> books = getBooks();
        ModelAndView modelAndView = new ModelAndView();

        modelAndView.setView(this.bookListExcelView);
        modelAndView.addObject("books", books);
        return modelAndView;
    }
```

이에 대한 테스트 코드는 다음과 같습니다.

```java
@Test
public void getBookExcel() throws Exception {
    MvcResult mvcResult = mockMvc.perform(get("/book/excel"))
        .andExpect(status().isOk())
        .andExpect(model().attributeExists("books"))
        .andDo(MockMvcResultHandlers.print())
        .andReturn();
    assertThat(mvcResult.getModelAndView().getView()).isNotNull().isInstanceOf(BookListExcelView.class);
}
```

### PDF (using iText 2.1.7)

`Spring`에서 PDF는 미묘합니다. java에서 가장 많이 사용되고 있는 `iText`는 `2.1.7`까지는 `MPL`정책을 가지고 있으나, 다음 버젼부터는 `AGPL`을 따르고 있습니다. `AGPL`은 매우 강력한 `License` 정책입니다. 이 정책은 이 library가 사용된 product의 전체 코드를 모두 `Open Source`로 공개해야지 됩니다. 따라서, 일반적인 Solution이나 Service에서 사용할 수 없습니다. 내부내에서 사용되는 Tool 정도로만 사용 가능합니다.

dependency는 다음과 같습니다.

```groovy
compile group: 'com.lowagie', name: 'itext', version: '2.1.7'
```

그리고, Spring에서 제공하는 `AbstractPdfView`를 상속받아 다음과 같이 구성하면 됩니다.

```java
@Component
public class BookListITextPdfView extends AbstractPdfView {
    @Override
    protected void buildPdfDocument(Map<String, Object> model, Document document, PdfWriter writer,
                                    HttpServletRequest request, HttpServletResponse response) throws Exception {
        setFileNameToResponse(request, response, "bookList.pdf");
        Chapter chapter = new Chapter(new Paragraph("this is english"), 1);
        chapter.add(new Paragraph("이건 메세지입니다."));
        document.add(chapter);
    }

    private void setFileNameToResponse(HttpServletRequest request, HttpServletResponse response, String fileName) {
        String userAgent = request.getHeader("User-Agent");
        if (userAgent != null && userAgent.contains("MSIE 5.5")) {
            response.setContentType("doesn/matter");
            response.setHeader("Content-Disposition", "filename=\"" + fileName + "\"");
        } else {
            response.setHeader("Content-Disposition", "attachment; filename=\"" + fileName + "\"");
        }
    }
}
```

이렇게 구성된 `View`를 이용한 `Controller` 코드는 다음과 같습니다.

```java
@GetMapping(value = "/book/itext2/pdf")
public ModelAndView getBookPdf() {
    List<Book> books = getBooks();
    ModelAndView modelAndView = new ModelAndView();

    modelAndView.setView(this.bookListPdfView);
    modelAndView.addObject("books", books);
    return modelAndView;
}
```

위와 같이 구성하고 테스트 코드는 다음과 같이 구성될 수 있습니다.

```java
@Test
public void getBookItext2Pdf() throws Exception {
    MvcResult mvcResult = mockMvc.perform(get("/book/itext2/pdf"))
        .andExpect(status().isOk())
        .andExpect(model().attributeExists("books"))
        .andDo(MockMvcResultHandlers.print())
        .andReturn();
    assertThat(mvcResult.getModelAndView().getView()).isNotNull().isInstanceOf(BookListITextPdfView.class);
    Files.write(Paths.get("sample.pdf"), mvcResult.getResponse().getContentAsByteArray());
}
```

마지막 라인의 경우, response를 파일로 저장시켜 완성된 pdf 파일을 확인할 수 있습니다. 그런데, 위 결과를 보시면 한글 메세지가 표시되지 않는 것을 확인할 수 있습니다. `iText2`의 경우, 한글 font가 정상적으로 들어가 있지 않습니다. 한글 메세지를 보기위해서는 다음과 같은 절차가 더 필요합니다.

먼저, 다음 url에서 `iTextAsian.jar`를 다운 받습니다.

`http://sourceforge.net/project/showfiles.php?group_id=15255`

다운받은 `jar`를 `mavenLocal` 또는 `nexus`와 같은 `repository storage`에 넣어줍니다.
`localMaven()`의 경우 다음과 같이 넣어줄 수 있습니다.

```bash
mvn install:install-file -DgroupId=com.lowagie -DartifactId=itextasian -Dversion=1.0 -Dpackaging=jar -Dfile=/Users/secucen/Downloads/iTextAsian.jar
```

그리고 `dependency`를 다음과 같이 수정하면 `localMaven`에서 위 jar를 참고해서 사용할 수 있습니다.

```groovy
repositories {
    mavenLocal()
    jcenter()
}

dependencies {
    .....
    compile 'com.lowagie:itextasian:1.0'
}
```

이제 `Font`를 사용하도록 코드를 수정합니다.

```java
@Override
protected void buildPdfDocument(Map<String, Object> model, Document document, PdfWriter writer,
                                HttpServletRequest request, HttpServletResponse response) throws Exception {
    setFileNameToResponse(request, response, "bookList.pdf");
    BaseFont objBaseFont = BaseFont.createFont("HYGoThic-Medium", "UniKS-UCS2-H", false);
    Font objFont = new Font(objBaseFont, 12);
    Chapter chapter = new Chapter(new Paragraph("this is english"), 1);
    chapter.add(new Paragraph("이건 메세지입니다.", objFont));
    document.add(chapter);
}
```

이제 한글을 정상적으로 표시할 수 있습니다. 이 때 폰트는 `iTextAsian`에서 제공되는 font만을 사용하게 됩니다.

만약 외부 한글 font를 이용하기 위해서는 다른 방법을 사용해야지 됩니다. itext는 거의 대부분의 ttf 파일을 지원합니다. ttf를 class path에 넣어주는 것이 필요합니다. 이때, ttf파일을 class path에 넣어주는 것이 중요합니다. 이는 font embedded가 되는 것으로 생각하시면 되고, 이는 한참 문제되고 있는 font 저작권문제와 연관이 깊습니다. 이렇게 배포하게 되는 경우에는 font의 저작권을 신경써주시는 것이 좋습니다. 외부 ttf 파일을 이용해서 사용하는 경우, 다음과 같은 코드를 이용하면 됩니다.

```java
private BaseFont getIdentityFont(String path) throws DocumentException, IOException {
    String fontPath = FontUtils.class.getClassLoader().getResource(path).toExternalForm();
    if (fontPath == null) {
        return null;
    }
    BaseFont baseFont = BaseFont.createFont(fontPath, BaseFont.IDENTITY_H, BaseFont.EMBEDDED);
    return baseFont;
}

@Override
protected void buildPdfDocument(Map<String, Object> model, Document document, PdfWriter writer,
                                HttpServletRequest request, HttpServletResponse response) throws Exception {
    setFileNameToResponse(request, response, "bookList.pdf");
    BaseFont objBaseFont = getIdentityFont("nanumGothic.ttf");
    if (objBaseFont == null) {
        throw new RuntimeException("font 파일이 존재하지 않습니다.");
    }
    Font objFont = new Font(objBaseFont, 12);
    Chapter chapter = new Chapter(new Paragraph("this is english"), 1);
    chapter.add(new Paragraph("이건 메세지입니다.", objFont));
    document.add(chapter);
}
```

### PDF (using iText 5 higher)

`iText` 5 부터는 `APGA`라는 매우 강력한 라이센스 정책을 가지고 있습니다. 이 정책의 경우에는 만든 제품의 코드를 공개하지 않으면 안됩니다. 이 라이센스정책의 경우, 솔루션에서 사용할 수 없습니다. 그런 이유인지는 모르겠지만, `Spring`에서 기본적으로 지원하고 있지는 않습니다. 그렇지만, 우리는 기존 `AbstractPdfView`를 기반으로 해서 새로운 `AbstractView`를 만들어낼 수 있습니다. 다음과 같이 구성해보도록 하겠습니다.

```java
public abstract class AbstractIText5PdfView extends AbstractView {
    public AbstractIText5PdfView() {
        setContentType("application/pdf");
    }

    @Override
    protected boolean generatesDownloadContent() {
        return true;
    }

    @Override
    protected final void renderMergedOutputModel(Map<String, Object> model, HttpServletRequest request,
                                                 HttpServletResponse response) throws Exception  {
        ByteArrayOutputStream baos = createTemporaryOutputStream();

        Document document = new Document(PageSize.A4.rotate(), 36, 36, 54, 36);
        PdfWriter writer = PdfWriter.getInstance(document, baos);
        prepareWriter(model, writer, request);
        buildPdfMetadata(model, document, request);

        document.open();
        buildPdfDocument(model, document, writer, request, response);
        document.close();

         // make browser to ask for download/display
        response.setHeader("Content-Disposition", "attachment");
        writeToResponse(response, baos);
    }

    protected void prepareWriter(Map<String, Object> model, PdfWriter writer,
                                 HttpServletRequest request) throws DocumentException {
        writer.setViewerPreferences(getViewerPreferences());
    }

    protected int getViewerPreferences() {
        return PdfWriter.ALLOW_PRINTING | PdfWriter.PageLayoutSinglePage;
    }

    protected abstract void buildPdfMetadata(Map<String, Object> model, Document document, HttpServletRequest request);

    protected abstract void buildPdfDocument(Map<String, Object> model, Document document, PdfWriter writer,
                                             HttpServletRequest request, HttpServletResponse response) throws Exception;

}
```

### JSON

JSON (JavaScript Object Notation)은 경량의 `DATA-교환형식`이다. 이 형식은 사람이 읽고 쓰기에 용이하며, 기계가 분석하고 생성함에도 용이합니다. JavaScript Programming Language, Standard ECMA-262 3rd Edition - December 1999의 일부에 토대를 두고 있습니다. JSON은 완벽하게 언어로 부터 독립적이지만 C-family 언어 - C, C++, C#, Java, JavaScript, Perl, Python 그외 다수 - 의 프로그래머들에게 친숙한 관습을 사용하는 텍스트 형식을 가지고 있습니다.

JSON은 두개의 구조를 기본으로 가지고 있습니다.

* name/value 형태의 쌍으로 collection 타입. 다양한 언어들에서, 이는 object, record, struct(구조체), dictionary, hash table, 키가 있는 list, 또는 연상배열로서 구현되었습니다.
* 값들의 순서화된 리스트. 대부분의 언어들에서, 이는 array, vector, list, 또는 sequence로서 구현되었습니다.

`DATA-교환형식`으로서의 `JSON`은 오늘날의 개발에서 매우 중요한 역활을 가지고 있습니다. 단순히 `frontend-backend`간의 데이터 교환뿐이 아니라 mongoDb, mySQL의 경우 기본 데이터 형식으로 `JSON`을 가지고 있습니다.

Spring에서는 JSON을 View로 사용하기 위해서는 다음과 같은 dependency가 필요합니다.

```groovy
    compile group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: '2.9.4'
```

Spring에서는 `jackson`이 포함되어 있고, method에 `@ResponseBody`가 지정되어 있는 경우에 `JSON`으로 변경된 return값을 보내줍니다.

```java
@GetMapping(value = "/book/json")
@ResponseBody
public Object getBookJson() {
    List<Book> books = getBooks();
    return books;
}
```

```java
@Test
public void getBookJson() throws Exception {
    MvcResult mvcResult = mockMvc.perform(get("/book/json"))
        .andDo(print())
        .andExpect(status().isOk())
        .andReturn();
}
```

```bash
MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = {Content-Type=[application/json;charset=UTF-8]}
     Content type = application/json;charset=UTF-8
             Body = [{"name":"책0","author":"저자0","status":"NORMAL"},{"name":"책1","author":"저자1","status":"NORMAL"},{"name":"책2","author":"저자2","status":"NORMAL"},{"name":"책3","author":"저자3","status":"NORMAL"},{"name":"책4","author":"저자4","status":"NORMAL"},{"name":"책5","author":"저자5","status":"NORMAL"},{"name":"책6","author":"저자6","status":"RENTNOW"},{"name":"책7","author":"저자7","status":"RENTNOW"},{"name":"책8","author":"저자8","status":"NORMAL"},{"name":"책9","author":"저자9","status":"NORMAL"}]
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

## Summary

Spring은 다양한 View를 제공합니다. 크게 두가지 타입으로 나눈다면 `HTML형태로 보여지는 View`와 `특별한 형태로 변환되는 View`로 볼 수 있습니다.
`HTML형태로 보여지는 View`는 다양한 View Engine을 이용할 수 있습니다. View Engine 종류로는 `tiles`, `velocity`, `freemarker`를 주로 사용합니다. 여담으로 회사내부에서는 `sitemash`를 사용하고 있습니다.
`특별한 형태로 변환되는 View`는 매우 다양한 형태로 데이터를 표현하는 방식입니다. pdf, excel, json 등 다양한 데이터 형태를 제공하고 있습니다.
