---
title: 14. Spring MVC의 구조
date: 2018-03-28 08:19:24
tags: java, web, spring-mvc
---
# Spring MVC의 구조

지금까지 Spring을 이용한 DB에 대한 접근법과 다루는 법에 대해서 알아보았습니다. 엄밀히 말하면 이 부분은 MVC pattern의 M에 해당되는 내용들입니다. Spring은 구조화된 MVC 구조를 제공하며, Layer 기반의 잘 정형화된 서비스 패턴을 제공합니다. Spring은 MVC를 기반으로 하는 Web Framework를 제공하고 이는 org.springframework.web 안에 제공되어 있습니다.
이제부터 사용할 package인 org.springframework.web에서는 C와 V에 대한 내용들을 다루고 있습니다.

## Spring MVC 동작 모델

![](/images/14/01.jpg)

### Front Controller

기존 servlet application 작성시에, url에 각각의 servlet을 등록해서 사용하는 것을 알고 있습니다. 일반적인 MVC 구조의 Web Application은 `Front Controller pattern`이라고 해서, 가장 최 상단 `FrontController`가 모든 Web의 Request를 처리합니다. 그리고, 처리된 Request에 따라 맞는 Controller를 찾아서 호출해주는 형식입니다.

Spring에서는 `org.springframework.web.servlet.DispatcherServlet`에서 `FrontController`를 담당합니다. 참고로, .NET에서는 ASP .NET MVC에서는 `ControllerFactory`가 이 일을 합니다.

### Controller

`org.springframework.web.servlet.DispatcherServlet`에서 처리된 request가 전달되는 영역입니다. 여기에서 HTML의 경우, 화면에 보여줄 model을 생성 하고, 또는 REST API의 경우에는 전달될 객체를 결정하게 됩니다.

간단하게 말해서, Controller는 URL과 상호동작하는 Class입니다.
/wpwoms/appuser/getshoplist 라는 URL이 호출되었을 때, Http Request가 연결되는 Class를 의미합니다.
기술적으로는 굉장히 어려운 영역입니다. 이는 Servlet container의 Http Request가 어떻게 해석이 되며, 이 해석된 Http Request에 따라 어떤 class의 어떤 method를 선택할 지에 대한 복잡한 연결을 계속해서 이어가야지 됩니다.
Controller는 기본적으로 다음과 같은 일을 합니다.

* Servlet이 넘겨주는 HttpServletRequest를 처리한다.
* Servlet으로 HttpServletResponse를 보내준다.
* Cookie, Session 에 대한 동기화 작업을 지원한다.

### View

사용자, Client에게 전달될 내용입니다. Html이 될 수도 있고, JSON이 될 수도 있고, Excel File이 될 수도 있습니다.
어찌보면 단순한 영역일수도 있지만... 이 부분이 어마어마한 노가다를 요구하는 부분입니다. ㅠ-ㅠ
View는 Controller가 보내주는 데이터를 정해진 포멧으로 변경하여 표현하는 영역이라고 생각하면 됩니다. 일반적으로 `HTML`, `JSON`, `FILE` 등이 있습니다.

### model

MVC에서의 M과는 다른 개념입니다. 기존 MVC에서의 Model은 BL 또는 Persistance Layer의 각 model 또는 entity를 의미하지만, 이곳의 model은 View Model입니다. View에 표시되는 model은 Controller에서 작성이 되는 경우도 있고, Domain Model에서 넘겨온 값을 그대로 이용하는 경우도 있습니다. 기존에 Application의 데이터 흐름에서 이야기한 DTO, VO가 여기에 속하게 됩니다.

## Spring MVC Web Request Process

Spring MVC는 Web Request를 다음과 같은 순서로 처리되게 됩니다.

![](/images/14/02.gif)

* Request -> DispatcherServlet : Request가 처음 들어오게 됩니다.
* DispatcherServlet -> Handler Mapping : DispatcherServlet은 Request를 분석하고 Mapping 중에서 Request와 동일한 Mapping을 찾아냅니다.
* Handler Mapping -> DispatcherServlet : 찾아진 Handler Mapping을 DispatcherServlet에 전달해줍니다.
* DispatcherServlet -> Controller : Handler Mapping에서 찾아진 Controller측에 Http Request를 전달해줍니다.
* Controller -> DispatcherServlet : ModelAndView 객체를 보내줍니다. ModelAndView객체는 View의 이름과 View에 표시될 데이터를 가지고 있습니다.
* DispatcherServlet -> View Resolver : View이름을 가지고 View를 처리하는 View Resolver에 ViewName을 전달합니다.
* View Resolver -> DispatcherServlet : 이름에 맞는 View를 return 시켜줍니다.
* DispatcherServlet -> View : View에 표시될 데이터를 가지고 있는 model을 보내줍니다.
* View -> DispatcherServlet : model을 이용해서 View를 render 시키고, 그 결과를 DispatcherServlet에 보내줍니다.
* DispatcherServlet -> Response : View로부터 받은 render된 결과를 Client에게 보내주게 됩니다.

위 그림과 순서는 머리에 익혀두고 있어야지 됩니다. Spring이 우리에게 감춰버리는 부분이기 때문에, 개발을 할때에는 자주 쓰이는 부분이 아니지만 디버그나 문제가 발생했을 때, 어떤 영역에서의 문제인지 확인할 필요가 있을 때 사용되는 지식입니다.

## Spring WebApplication에서의 ApplicationContext의 로드

먼저, 기본적인 Spring이 적용된 `web.xml`을 볼 필요가 있습니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns="http://java.sun.com/xml/ns/javaee"
          xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
          xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
          id="WebApp_ID" version="3.0">
    <display-name>mvc.spring01</display-name>
    <listener>
        <display-name>ContextLoader</display-name>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>spring</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>spring</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
</web-app>
```

먼저, listener입니다. web application이 실행될 때, 기본 bean들을 load 하기 위한 listener를 등록합니다. 기본적으로 main/webapp/WEB-INF/applicationContext.xml 파일을 읽어 bean들을 load 시켜줍니다.

다음은 servlet 설정입니다. front controller로 spring 이라는 이름으로 DispatcherServlet을 로드하고 있는 것을 볼 수 있습니다.

마지막으로 servlet-mapping입니다. Servlet이 사용될 url path를 설정합니다. 위 xml에서는 /app/* path에서 DispatcherServlet이 실행됨을 지정하고 있습니다. 그리고, DispatcherServlet가 로드될때, 자동으로 spring-servlet.xml bean 정의를 로드합니다. 이는 DispatcherServlet 내부에서 사용할 child appliation context입니다. 파일 이름은 `servlet 이름 + "-servlet.xml"` 로 자동 결정됩니다.

따라서, 위 설정의 경우 src/main/webapp 폴더는 다음과 같이 구성되게 됩니다.

```cmd
├── META-INF
└── WEB-INF
    ├── applicationContext.xml
    ├── spring-servlet.xml
    └── web.xml
```

지금보시면 applicationContext가 2개가 등록되게 됩니다. 이 구조는 root context - child context 구조입니다. child 끼리는 서로간에 간섭되지 않는 bean들이 등록될 수 있으며, root의 bean들은 child에서 공유하는 구조입니다. 이런 root-child context 구조는 기존에는 spring application context를 사용하지만, web 기술은 spring @MVC를 하지 않고 structure 와 같은 다른 web framework를 사용하는 경우를 지원하기 위해서 만들어진 구조입니다.

그런데 왜 이렇게 2개의 application context를 가지도록 설계가 되어 있을까요? 그 이유는 Spring은 먼저 ApplicationContext를 제공하는 Spring-core와 Spring-Bean으로 시작했기 때문입니다. spring-mvc의 경우에는 후에 추가가 된 것이고, 초기에는 spring-core, spring-bean을 이용한 DI와 AOP만을 이용하고, web기술은 struct 와 같은 다른 web framework를 이용했기 때문입니다. 그래서 두개의 설정이 나뉘게 되었고, 그게 지금까지 고정되어 사용되고 있는 것입니다.

## Hello WebApplicaiton - Spring(old)

기본적으로 `web.xml` + `applicationContext`를 이용한 HelloWorld를 만들어보도록 하겠습니다.
시작은 `gradle init --type=java-application`입니다. 후에 'build.gradle'을 이렇게 만들어줍니다.

```groovy
apply plugin: 'java'
apply plugin: 'war'
apply from: 'https://raw.github.com/akhikhl/gretty/master/pluginScripts/gretty.plugin'

repositories {
    jcenter()
}

dependencies {
    compile 'com.google.guava:guava:23.0'
    providedCompile group: 'javax.servlet', name: 'javax.servlet-api', version: '4.0.0'
    compile group: 'org.springframework', name: 'spring-webmvc', version: '5.0.2.RELEASE'

  	testCompile group: 'org.springframework', name: 'spring-test', version: '5.0.2.RELEASE'
    testCompile 'junit:junit:4.12'
    testCompile group: 'org.assertj', name: 'assertj-core', version: '3.9.0'
}

gretty {
	contextPath = '/'
}
```

Controller를 만들어줍니다. 위에 설명드린것처럼, HttpRequest를 처리하는 영역입니다. Spring에서는 `org.springframework.web.servlet.mvc.Controller`라는 interface를 제공하고 있으며, 이는 `HttpServlet`과 거의 유사한 기능을 제공합니다. 다음은 `Controller` 코드입니다.

```java
public class HomeController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request,
                                      HttpServletResponse response) throws Exception {
        String name = request.getParameter("name");
        String message = "이름이 입력되지 않았습니다.";
        if (!Strings.isNullOrEmpty(name)) {
            message = "Hello World!! " + name;
        }
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("message", message);
        modelAndView.setViewName("hello");
        return modelAndView;
    }
}
```

이제 `spring-servlet.xml`에 bean들을 등록시켜줘야지 됩니다. `applicationContext.xml`이 아닌, `spring-servlet.xml`에 등록시켜야지 된다는 것을 잊지 말아주세요.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean name="/hello.do" class="com.xyzlast.web.controller.HomeController"/>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
        <property name="order" value="1" />
    </bean>
</beans>
```

아래 정의된 `InternalResourceViewResolver`는 추후에 다시 설명하도록 하겠습니다. 지금은 pass 하시지요.
이제 controller에서 사용할 jsp 파일을 넣어줍니다. jsp 파일의 위치는 `/webapp/WEB-INF/jsp/hello.jsp`입니다.

```jsp
<h1>${message}</h1>
```

이제 마지막으로 rootContext가 될 `applicationContext.xml`파일을 넣어줍니다. 내용은 다음과 같습니다. 공통 Bean이 하나도 없기 때문에 xml 선언만 들어갑니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns:mvc="http://www.springframework.org/schema/mvc"
     xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

최종적으로 구성된 `webapp`의 폴더구조입니다.

```cmd
.
├── META-INF
└── WEB-INF
    ├── applicationContext.xml
    ├── jsp
    │   └── hello.jsp
    ├── spring-servlet.xml
    └── web.xml
```

이제 `gradle appRun`을 실행시키고, `http://localhost:8080/hello.do`를 열면 다음과 같은 화면을 보실 수 있습니다.

![](/images/14/04.png)

자. 지금까지 옛날 방식의 Spring MVC의 기본 페이지 작성법을 따라가봤습니다. 특이할 점은 딱 하나입니다. `spring-servlet.xml`파일의 정의입니다. 그리고, 그 중에서 아래 한줄입니다.

```xml
<bean name="/hello.do" class="com.xyzlast.web.controller.HomeController"/>
```

bean을 name으로 정의하고, 그 name이 url과 1:1로 mapping 됩니다. 특이하게 `name`속성으로 binding하는 것을 주의하면 어려울 것이 없습니다.

여기까지 진행한 후, 다시 이 그림을 보도록 하겠습니다.

![](/images/14/02.gif)

|단계|code|설명|
|---|---|---|
|step01|spring-servlet.xml|`spring-servlet.xml`에 지정된 request url과 controller의 matching|
|step02|HomeController|controller코드 실행, `ModelAndView` 반환|
|step03|InternalResourceViewResolver|`ModelAndView`의 ViewName을 이용해서 `/WEB-INF/jsp/hello.jsp`를 반환|
|step04|InternalResourceViewResolver|`View`와 `Model`을 결합시켜 `HTML`을 만들어서 `Response`로 보내줌|


## 테스트 WebApplicaiton

이와같은 web application의 테스트는 어떻게 해야지 될까요? 지금까지 web을 만들고 테스트 하는 것은 web server를 실행시키고, web server의 url을 직접 타이핑을 하던가, 아니면 link를 click-click 해가면서 손으로 테스트를 했습니다. 그런데 이런 테스트 방법은 매우 시간이 많이 걸리고, 개발자들을 불편하게 합니다.
이제 다시 한번 테스트를 어떤것을 해야지 되는지 확인해보도록 합시다. 먼저, 지금 HelloController는 /hello.do url로 접근이 가능한지 확인해야지 됩니다. 그리고 Model에 message가 들어있는지 확인되어야지 됩니다. 마지막으로 message가 원하는 값인 Hello + input 임을 확인할 수 있어야지 됩니다.

정리해보겠습니다.  우리가 Controller에서 테스트할 내용은 다음과 같습니다.

* 원하는 url로 접근이 가능한지.
* return되는 Model에 필요한 값이 존재하는지.
* return된 Model의 값이 원하는 값으로 나오는지 확인

이 3가지가 되면 Controller는 테스트가 가능합니다. 그리고 View는 Controller에 의해서 나온 값을 보여주는 영역이기 때문에 특별히 테스트를 하지 않습니다. 최종적으로 View영역도 HTML의 테스트를 하는 것이 필요하지만, 이 부분은 지금 영역을 넘어가기 때문에 다루지 않도록 하겠습니다.
이러한 테스트 방법으로 spring은 멋진 솔루션을 제공합니다. Controller에 대한 단위 테스트 코드를 작성하도록 하겠습니다.

다음은 전체 테스트 코드입니다.

```java
@SuppressWarnings("unused")
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({
    "file:src/main/webapp/WEB-INF/applicationContext.xml",
    "file:src/main/webapp/WEB-INF/spring-servlet.xml"
})
@WebAppConfiguration
public class HomeControllerTest {
    @Autowired
    private WebApplicationContext webApplicationContext;
    private MockMvc mockMvc;

    @Before
    public void setUp() throws Exception {
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    }

    @Test
    public void helloWithoutNameTest() throws Exception {
        MvcResult mvcResult = mockMvc.perform(get("/hello.do"))
                                     .andExpect(status().isOk())
                                     .andReturn();
        ModelAndView modelAndView = mvcResult.getModelAndView();
        assertThat(modelAndView.getViewName()).isEqualTo("hello");
        assertThat(modelAndView.getModel().containsKey("message")).isTrue();
        assertThat(modelAndView.getModel().get("message")).isEqualTo("이름이 입력되지 않았습니다.");
    }

    @Test
    public void helloWithNameTest() throws Exception {
        MvcResult mvcResult = mockMvc.perform(get("/hello.do").param("name", "윤영권"))
            .andExpect(status().isOk())
            .andReturn();
        ModelAndView modelAndView = mvcResult.getModelAndView();
        assertThat(modelAndView.getViewName()).isEqualTo("hello");
        assertThat(modelAndView.getModel().containsKey("message")).isTrue();
        assertThat(modelAndView.getModel().get("message")).isEqualTo("Hello World!! 윤영권");

    }
}
```

test code 살펴보면 다음과 같습니다.

먼저 MockMvc라는 servlet container를 가상으로 Spring 내부에서 만들어줍니다. 만들어지는 객체는 tomcat이나 jetty와 동일하게 움직입니다.

mvc.perform method를 통해서 url을 기반으로 접근이 됩니다. 그리고, status값이 200이 나오는지 확인을 하는 구조로 만들어집니다.
마지막으로 andReturn을 통해서 Model 과 View값을 얻어내 접근이 가능합니다. 이런 테스트 코드를 이용해서 우리가 만든 Controller가 정상적으로 움직이는지 확인이 가능합니다.

## Hello Application (current)

지금까지 만든 코드는 Spring 2.x 대에서 Controller를 만드는 방법입니다.
지금까지 구현해본 코드의 문제점은 다음과 같습니다.

* 특정 interface를 반드시 상속받는 형태여야지 된다.
* bean name을 이용한 method - url mapping이 된다.
* POST/GET/DELTE/PUT/HEAD 와 같은 http method에 적합하지 않다.
* url 당 1개의 Controller 객체가 생성되기 때문에 Enterprise 개발에 적합하지 않다.

기존에 Controller Interface를 구현해야지 되었으나, 3.0대의 @MVC는 annotation을 이용해서 @Controller로 객체의 meta 정보를 적는 것으로 해결이 가능합니다.

다음과 같은 코드로 기존과 동일한 Controller가 구현 가능합니다.

```java
@Controller
public class HomeController {

    @RequestMapping(value = "/hello.do", method = RequestMethod.GET)
    public ModelAndView getHello(HttpServletRequest request,
                                 HttpServletResponse response) throws Exception {
        String name = request.getParameter("name");
        String message = "이름이 입력되지 않았습니다.";
        if (!Strings.isNullOrEmpty(name)) {
            message = "Hello World!! " + name;
        }
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("message", message);
        modelAndView.setViewName("hello");
        return modelAndView;
    }
}
```

URL을 `spring-servlet.xml`에 정의된 내용들을 변경시켜줘야지 됩니다. `name` attribute를 제거합니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.xyzlast.web.controller.HomeController"/>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
        <property name="order" value="1" />
    </bean>
</beans>
```

이제 테스트를 돌려보면 기존과 동일한 결과를 얻어낼 수 있습니다. `old` 방법의 경우에는 기존 코드의 유지보수에서만 사용되지, 이제는 사용되지 않는 방법입니다.

## URL Mapping

URL Mapping의 기본이 되는 `@RequestMapping`에 대해서 좀 더 알아보겠습니다.

### String[] value

URL을 지정해주는 pattern입니다. 예를 들어 다음과 같은 pattern들을 지정해주는 것이 가능합니다.

* @RequestMapping("/hello")
* @RequestMapping({"/hello", "/hello2"})
* @RequestMapping("/hello1")

Path변수를 지정해주는 것이 가능합니다. "/hello/{name}" 과 같이 method의 input을 url에 같이 실어 주는것이 가능합니다.

* @RequestMapping("/hello/{name}")
* @RequestMapping("/hello/{name}/{number}")

### method

RequestMethod를 지정하는 것 역시 가능합니다. GET, POST, DELETE, PUT, HEAD가 사용 가능합니다. 역시 method도 Path와 동일하게 중복 설정이 가능합니다.

### params

url 요청 파라미터의 유무에 따라 호출되는 method를 결정할 수 있습니다.

```java
@RequestMapping(value="/user/edit", params="type=admin")
public ModelAndView callWithAdmin() {
  // URL: "/user/edit?type=admin"
}

@RequestMapping(value="/user/edit", params="type=member")
public ModelAndView callWithMember() {
  // URL: "/user/edit?type=member"
}
```

method의 input값에 따라 method내에서의 if문을 만들지 않고 처리가 가능한 유용한 팁입니다.

### headers

HTTP header정보에 따른 mapping역시 가능합니다. request type을 text/html, application/json 각각의 type에 따라 다른 method를 처리 가능합니다.

### RequestMapping과 method parameter와 결합

기본적으로 @Controller의 각 method는 다음과 같은 input parameter를 갖을 수 있습니다.

|Type|Annotation|Description|
|---|---|---|
|URL/POST parameter|@RequestParam|GET/POST/DELETE/PUT 등으로 호출될 때, 넘겨지는 Parameter값|
|Path value|@PathVariable|URL 에 포함된 특정 영역 문자열|
|Servlet|	-	|HttpServletRequest, HttpServletResponse를 직접 Handling 하는 기존 Servlet과 같은 코드 역시 사용 가능합니다.|
|Cookie|@CookieValue|Cookie 값을 얻거나 설정할 수 있습니다.|
|Session|@SessionAttributes|Session 값을 얻거나 설정할 수 있습니다.|
|Body|@RequestBody|Request의 Body 부분을 모두 String으로 얻어내는 것이 가능합니다.|

### return value

action method는 주로 4가지 형태의 Return Type을 사용하게 됩니다.

* String : view 이름을 return 합니다. 주로 JSP 또는 Veloctiy, Freemarker, tiles와 Html View를 호출 할 때 사용됩니다. String 형태를 보내줄 때는 일반적으로 ViewResolver라는 객체를 applicationContext.xml에 등록하는 것이 일반적입니다.
* void : 아무것도 return 하지 않습니다. 이는 method의 이름을 String 형태로 return 하는 것과 동일한 결과를 갖습니다.
* Object : REST 형태의 Return값을 사용할 때 이용됩니다. 대체적으로 Object에 대한 Body Response를 덩어내는 것을 주 목적으로 하고 있습니다.
* ModelAndView : Spring 2.X 형태로 Model과 View를 return 하는 것이 가능합니다.

## 확장자가 없는 URL

지금까지의 예시에서는 확장자를 하나씩 붙여서 처리했습니다. 이러한 확장자를 붙여서 처리하는 것은 DispatcherServlet에서 처리할 URL이 어떤 것인지 명확하게 결정해줄 수 있는 장점을 가지고 있습니다. 그렇지만, 이 방법은 조금 애매한 것이 사실입니다. W3C에서 표준한 URL 표준에서, 확장자는 URL을 통해서 얻고 싶은 Response의 protocol을 선택하는 형식입니다. 예를 들어서 다음 URL들의 해석은 다음과 같습니다.

```cmd
/shop/search.json?name=윤영권
/shop/search.html?name=윤영권
/shop/search.rss?name=윤영권
/shop/search.xml?name=윤영권
```

다음 4개의 url format은 json/html/rss/xml 형식으로 데이터를 얻어오는 것을 가정하고 있습니다. url의 extension에 dispatcher servlet 을 종속시키는 것은 이제 옛날 방법입니다. 이제는 모든 URL을 dispatcherServlet에 보내게 됩니다. 그리고 dispatcherServlet에서 해석 불가능한 request들을 servlet container에게 넘기는 방식으로 처리하는 것입니다.

여기서 servlet spec에 대해서 한번 알아보도록 하겠습니다. `SRV.11.2 Specification of Mappings` 에서 servlet container는 다음과 같이 명령을 정의하고 있습니다.

* A string beginning with a ‘/’ character and ending with a ‘/*’ suffix is used for path mapping.
> '/'로 시작하고, '/*'로 끝나는 mapping의 경우에는 path mapping을 따르게 된다. (path mapping : servlet container에서 정의되어 있는 mapping입니다.)
* A string beginning with a ‘*.’ prefix is used as an extension mapping.
> *. 으로 시작하는 mapping의 경우에는 확장 mapping을 따르게 된다. 우리가 기존에 사용하는 것을 확인했던 *.do와 같은 방법입니다.
* A string containing only the ’/’ character indicates the "default" servlet of the application. In this case the servlet path is the request URI minus the context path and the path info is null.
> '/'만 사용하게 되는 경우, 이는 application의 'default' servlet을 의미한다. 이 경우에는 request URI와 context Path가 완전히 일치하게 되고, servlet path는 null이 됩니다.
* All other strings are used for exact matches only.
> 모든 request uri가 완전히 일치하는 경우에는 확장 mapping에 지정된 servlet을 사용한다.

이 4가지 경우중에서 우리가 사용하게 되는 것은 3번으로 모든 request를 동일한 servlet으로 처리하게 됩니다. 이 경우에는 web.xml은 다음과 같이 설정할 수 있습니다.

```xml
<servlet-mapping>
    <servlet-name>spring</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

그리고, `application의 default servlet을 변경`하기 위해서 spring에 다음 설정을 추가합니다.

```xml
<mvc:annotation-driven />
<mvc:default-servlet-handler/>
```

이제 `default-servlet`을 변경시켰기 때문에, 각 `child-container`에 설정을 넣어줄 필요 역시 없어집니다.

`applicationContext.xml`은 다음과 같이 변경될 수 있습니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns:mvc="http://www.springframework.org/schema/mvc"
     xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <mvc:annotation-driven />
    <mvc:default-servlet-handler/>
    <bean class="com.xyzlast.web.controller.HomeController"/>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
        <property name="order" value="1" />
    </bean>
</beans>
```

`spring-servlet.xml`은 다음과 같이 변경될 수 있습니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

## @Configuration을 이용한 DispatcherServlet 설정 (with web.xml)

지금까지 `Service`를 구성할 때, 주로 사용하던 `@Configuration`을 어떻게 사용할 수 있는지 알아보도록 하겠습니다.

먼저, 기존 `applicationContext.xml`에 해당되는 `@RootConfiguration` 코드는 다음과 같습니다.

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = {
    "com.xyzlast.web.controller"
})
public class RootConfiguration {
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
        internalResourceViewResolver.setPrefix("/WEB-INF/jsp/");
        internalResourceViewResolver.setSuffix(".jsp");
        internalResourceViewResolver.setOrder(1);
        return internalResourceViewResolver;
    }
}
```

이를 등록하기 위해서는 기본적으로 사용되는 `contextClass`를 기본값인  `XmlConfigWebApplicationContext`에서 `AnnotationConfigWebApplicationContext`으로 변경해야지 됩니다. 그리고 `contextConfigLocation`을 `@Configuration`으로 바꾸어주면 해결할 수 있습니다.

구성된 `web.xml`은 다음과 같습니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns="http://java.sun.com/xml/ns/javaee"
          xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
          xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
          id="WebApp_ID" version="3.0">
    <display-name>mvc.spring01</display-name>

    <listener>
        <display-name>ContextLoader</display-name>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
    </context-param>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.xyzlast.web.config.RootConfiguration</param-value>
    </context-param>

    <servlet>
        <servlet-name>spring</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>spring</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

## @Configuration을 이용한 DispatcherServlet 설정 (without web.xml)

`web.xml`에 대해서 이야기하던 내용과 같이 `web.xml`이 없는 경우에는 어떻게 해야지 되는지 따라가보도록 하겠습니다. 이 방법은 최근에 주로 사용되고 있기 때문에, 이에 대해서도 잘 알아둘 필요가 있습니다.

이 경우에는 `WebApplicationInitializer`를 통해 설정이 가능합니다. `WebApplicationInitializer`의 경우, `web.xml`의 역활을 모두 다 담당할 수 있습니다. 이 interface를 잘 만들어둔 `AbstractAnnotationConfigDispatcherServletInitializer`를 상속받아서 구현 가능합니다. 구현되는 코드는 다음과 같습니다.

```java
public class WebXml extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfiguration.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[0];
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }

    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
        characterEncodingFilter.setEncoding("UTF-8");
        characterEncodingFilter.setForceEncoding(true);
        return new Filter[] { characterEncodingFilter };
    }
}
```

## Summary

Spring WebMVC의 기본 구조에 대해서 알아봤습니다.

Spring WebMVC에서 사용되는 `applicationContext.xml`의 구조와 `@Configuration`에 대한 설명을 드렸습니다. 또한, `Controller`에 대한 내용들을 같이 이야기했습니다.

너무 많은 내용들이 한 곳에 들어있습니다. Spring WebMVC에서 가장 중요한 부분입니다. 잘 익혀둬야지 됩니다.