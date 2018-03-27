---
title: 13.Java Web Application의 구조
date: 2018-03-28 08:17:31
tags: java, web.xml
---
# Java Web Application의 구조

java로 개발한 application을 배포를 할때는 주로 jar, war 형태로 배포를 하게 됩니다.
이 둘은 완전히 동일한 형식입니다. 다만, war는 web application을 배포하는 형식이고 jar는 library나 일반 application을 배포하는 형식입니다.

## Jar Application

build된 jar 파일의 압축을 풀어보면 다음과 같은 폴더 구조를 볼 수 있습니다.

```cmd
.
├── META-INF
│   └── MANIFEST.MF
└── com
```

`com`으로 시작되는 폴더는 만들어진 class가 들어가는 위치입니다. 그런데 지금 생성하지 않은 `META-INF` 폴더가 있는 것을 볼 수 있습니다. 폴더 안에는 build환경에 따라 다른 다양한 파일들이 들어있는데, 주목할 내용은 `MANIFEST.MF` 입니다.

이 파일은 다음과 같은 정보를 포함합니다.

* Manifest-Version: Manifest file version
* Build-By: Build user
* Build-Jdk: jdk version
* Created-By: Build tool
* Class-Path: 외부 추가 lib를 지정할 수 있습니다. jar내부에 외부 추가 jar를 같이 넣어서 배포가 가능합니다.
* Main-Class: 실행 파일 경로 및 파일명을 지정합니다.

이 내용은 jar파일에 대한 manual이나 스팩을 기록하는 폴더입니다. 그리고 우리가 web-resource와 같은 정적인 resource를 배포할 때도 역시 META-INF 폴더를 이용하게 됩니다. jar 파일은 자신이 개발한 class를 가질수도 있고, 심지어 jar에 jre까지 포함하는 것까지 가능합니다. 다른 jar 역시 추가하는 것도 가능합니다.

## War Application

```cmd
├── META-INF
│   └── MANIFEST.MF
└── WEB-INF
    ├── classes
    └── lib
```

war의 기본정보와 static-resource를 저장한 META-INF 폴더의 경우에는 jar와 동일하게 만들어집니다. 그리고 또다른 폴더가 하나더 생깁니다. WEB-INF 폴더가 바로 그것인데요. 이 폴더는 다음 2개의 폴더를 같이 포함합니다.

* lib - 외부 및 추가 jar가 위치하는 폴더
* classes - 개발된 application의 compile된 class 파일이 위치

war의 경우에는 ClassLoader에 의해서 로드되는 객체들은 classes 폴더를 기준으로 로드가 이루어지게 됩니다.

그리고, 이 안에는 매우 중요한 파일이 하나 들어가게 됩니다. 그것은 web.xml인데요. java web application의 구성은 기본적으로 WEB-INF/web.xml 에서 결정됩니다. web.xml은 필터, 서블릿, DB소스 등  web container가 구동하는데 이용되는 환경설정파일입니다. 서버가 처음 로딩될때 web.xml파일을 읽어들여 해당 환경설정에 대해 어플리케이션 배치를 하게 됩니다. 다른 말로 web.xml은 배치 서술자(deployment de-scriptor)라고도 합니다.

## web.xml

web.xml의 기본 환경설정 정보는 다음과 같습니다.

* 필터정보 (매핑 포함)
* 서블릿정보 (매핑 포함)
* 웹 애플리케이션 정보
* 세션정보
* 세션정보가 소멸, 생성, 수정되는 것을 알려주는 리스너 정보
* MIME매핑
* welcomefile정보
* errorPage 정보
* url보호 정보
* 인터프라이즈 빈의 홈레퍼런스 정보 (로컬 레퍼런스 정보 포함)

### 필터정보

필터는 요청이 들어오면 URL 패턴을 분석해서 해당 패턴의 요청이 들어오면 해당 요청에 담긴 정보들을 프로그래머가 원하는 형태의 데이터로 가공해서 서버로 요청을 넘길 수 있습니다. servlet과 비슷한 개념으로 계속 요청을 감시하고있습니다가 url패턴에 해당하는 요청이 들어오게되면 해당 요청을 가로채서 원하는 데이터 형태로 가공한뒤 다시 요청을 서버로 보내는 역할을 하게 됩니다.. 따라서 서버내에서 사용될 데이터 또는 화면으로 다시 보낼 데이터의 형태를 프로그래머가 원하는 형태로 단일화 시킬 수 있습니다.

```xml
<filter>
     <filter-name>myRequest</filter-name>
     <filter-class>myFilter.util.myReqFilter</filter-class>
     <init-param>
          <param-name>myRequestFilter</param-name>
          <param-value>filterTest</param-value>
     </init-param>
</filter>
```

필터 이름과 클래스, 초기화 파라미터를 지정하게 됩니다.. myRequest라는 필터는 myFilter.util 페키지 안에 있는 myReqFilter를 호출하며 초기화 파라미터로 myRequestFilter의 이름을 가진 filterTest값을 가지고 filter를 호출할것입니다. filter를 사용할때 사용할 클래스는 반드시 Filter인터페이스를 implenebts받아 doFilter메소드를 반드시 오버라이드 해주어야 하며 메소드 실행순서는 init() -> doFilter() -> destroy() 순으로 구성됩니다. 이는 Servlet의 생명 주기와 동일하게 동작합니다.

```xml
<filter-mapping>
     <filter-name>myRequest</filter-name>
     <url-pattern>*.doFilter</url-pattern>
</filter-mapping>
```

위에서 선언한 filter정보를 url패턴과 매핑하는 부분입니다.
어떠한 URL의 확장자로 .doFilter가 함께 넘어오면 myRequest필터를 호출하게 되고 해당 필터는 myFilter.util 페키지안에 있는 myReqFilter 를 호출하게 됩니다. url-pattern 대신 servlet-name이 올 수 있습닌다.

servlet-name이 들어오게 되면 해당 패턴에 맞는 모든 filter가 실행되고 나서 servlet을 호출하게 됩니다..
filter는 여러겹으로 사용할 수 있는데 필터 체인이라는 용어로 쓰입니다.

![](/images/13/01.png)

### 서블릿(Servlet) 정보

필터와 비슷한 맥락의 기능입니다. 그렇지만 큰 차이를 가지고 옵니다. Filter의 경우에는 Response와 Request에 대한 stream의 처리를 담당하고, 이에 대한 실질적인 Action을 담당하는 것이 Servlet입니다. web.xml에 매핑되어져 있는 서블릿들은 해당 요청을 포함하여 url패턴을 분석하고 거기에 매핑되어져 있는 클래스를 호출하여 처리를 하게 됩니다.
지정 방법은 다음과 같습니다.

```xml
<servlet>
     <servlet-name>loginServlet</servlet-name>
     <servlet-class>myServlet.util.loginServlet</servlet-class>
</servlet>
```

위처럼 하나의 서블릿 클래스를 생성해두고 해당 클래스를 servlet-class라는 element를 지정하게 됩니다..
나중에 loginServlet을 호출하면 myServlet.util 페키지 안에있는 loginServlet클래스가 호출될것입니다.

```xml
<servlet-mapping>
     <servlet-name>loginServlet</servlet-name>
     <url-pattern>*.login</url-pattern>
</servlet-mapping>
```

url패턴과 해당 해당 패턴에 해당하는 url로 접근시 호출될 servlet을 설정하는 부분입니다.
메타문자로 *가 올 수 있고 확장자를 설정하게 됩니다..
나중에 *.login이라는 URL을 서버로 요청하게되면 loginServlet의 이름을 가진 서블릿을 호출하게 됩니다..
해당 서블릿은 위에서 myServlet.util.loginServlet 클래스를 매핑해두었으므로 해당 클래스가 호출되는 형태가 될것입니다.

Spring은 이곳에 `DispatcherServlet`이라는 Spring에서 제공하는 Servlet을 이용해서 Spring MVC에서 Request를 처리합니다. 예를 들어 Naver의 경우에는 `*.nhn` 이라는 url-pattern에 `DispatcherServlet`을 연결시켜두는 것이 일반적입니다.
인터넷 서핑을 주로 하시다보면 `*.do` url을 간간히 보게 되는데요. 이것은 Spring의 2.5대 버젼에서 Spring에서 추천하던 url-pattern입니다. web.xml의 설정의 경우에는 개발자들이 copy-paste 신공을 펼치는 경우가 많기 때문에 이것을 계속해서 이용하게 되는 경우도 많지요.

### 웹 어플리케이션 정보 (Web Application )

웹 어플리케이션 정보를 설정하는 부분으로

```xml
<display-name>webApplication</display-name>
<description>desc for webApplication</description>
```

으로, Web Application의 정보와 표시 이름을 적을 수 있습니다.

### 세션정보 ( Session )

Session의 timeout 및 공유 방법을 결정할 수 있습니다.

```xml
<session-config>
    <timeout>20</timeout>
    <shared>true</shared>
</session-config>
```

### 세션 리스너 정보 ( Session Listner )

Session이 연결되고, 제거되는 각각의 event에 연결되는 Listner에 대한 정보를 기술할 수 있습니다. 아래는 servlet에서 기본으로 제공하는 listener입니다.

#### javax.servlet.ServletContextAttributeListener

이 리스너는 컨텍스트에 저장된 애트리뷰의 변화가 있었을 때 설정된 이벤트를 엔진이 자동으로 발생시키도록 합니다. 이벤트 감지를 얻어내기 위해서는 여러분들이 직접 작성한 리스너의 implemention클래스를 통해야 하는것은 물론이며, 아래의 리스너 모두 동일하게 설정됩니다.

#### javax.servlet.ServletContextListener

당신이 작성한 웹애플리케이션에서 컨텍스트를 이용하여 무언가를 저장하고 이용할 수 있는데 이 리스너의 작동은 그러한 컨텍스트에 대한 변경이 이루어졌을 때 자동으로 엔진이 감지하고 작성한 클래스의 이벤트를 발생시킵니다.

#### javax.servlet.http.HttpSessionActivationListener

request.getSession(true)같은 경우처럼 Session에 대한 내용이 새로 생성되어 activate되어졌을 때 발생하는 이벤트를 감지하는 리스너입니다.

#### javax.servlet.http.HttpSessionAttributeListener

HttpSession에 대한 애트리뷰트의 변경시 연결되어지는 리스너입니다.

#### javax.servlet.http.HttpSessionBindingListener

HttpSession에 대하여 클라이언트 세션정보에 대한 바인딩이 이루어졌을 경우 감지되는 리스너입니다.

이 SessionListener들을 상속받아 구현된 class에 대한 정보를 web.xml에 다음과 같이 등록하여 사용가능합니다.

```xml
<listener>
    <listener-class>SessionListener</listener-class>
</listener>
```

### MIME 매핑 (MIME Mapping)

web server에서 보내지는 각각의 파일 리소스에 대한 정보를 mapping하는 영역입니다. 예를 들어 JSON의 경우 application/json 형태임을 web browser에게 알리는 등의 일을 수행합니다. 이것이 정상적으로 설정되어 있지 않으면 hwp와 같은 파일 포멧을 다운받지 못하고, browser에서 직접 열게 되는 식의 오동작을 발생시키게 됩니다. 아래와 같은 형식으로 설정이 가능합니다.

```xml
<mime-mapping>
    <extension>zip</extension>
    <mime-type>application/zip</mime-type>
</mime-mapping>
<mime-mapping>
    <extension>hwp</extension>
    <mime-type>application/unknown</mime-type>
</mime-mapping>
```

### welcomefile 정보 ( welcomeFile )

welcomeFile은 서버에 접속하면 가장 최초로 보여줄 파일을 지정하는 부분입니다.

```xml
<welcome-file-list>
  <welcome-file>index.html</welcome-file>
  <welcome-file>index.jsp</welcome-file>
</welcome-file-list>
```

서버를 구동한 후 http://localhost:8080/myWeb/ 로 접속을 하게되면 index.html이 실행될것입니다. html파일이 없다면 index.jsp가 실행됩니다.

### errorPage 정보( ErrorPage )

특정 요청에 대해 error가 반환될때 해당 페이지를 보여주도록 설정이 가능합니다.

```xml
<error-page>
  <error-code>404</error-code>
  <location>/errors/404.jsp</location>
</error-page>
<error-page>
  <error-code>java.lang.NullPointerException</error-code>
  <location>/errors/badcode.jsp</location>
</error-page>
```

### url보호 정보 ( URL Security )

로그인 및 각 URL에 대한 접근 정책을 결정할 수 있습니다.  이 때 사용되는 Security 정책은 servlet-container의 정책에 따르게 되기 때문에 Web Application에서 사용하기에는 조금 힘듭니다.
tomcat에서는 conf/tomcat-user.xml 에 사용자가 정의되어 있습니다.  tomcat manager에 등록되어 있는 default security 정책은 다음과 같습니다.

```xml
<security-constraint>
  <display-name>Name String</display-name>
  <web-resource-collection>
    <web-resource-name>GETServlet</web-resource-name>
    <description>some description</description>
    <url-pattern>/servlet/*</url-pattern>
    <http-method>GET</http-method>
  </web-resource-collection>

  <auth-constraint>
    <description>some description</description>
    <role-name>*</role-name>
  </auth-constraint>

  <user-data-constraint>
     <description>some description</description>
     <transport-guarantee>INTEGRAL</transport-guarantee>
  </user-data-constraint>
</security-constraint>

<login-config>
  <auth-method>FORM</auth-method>
  <realm-name>MemoryRealm</realm-name>
  <form-login-config>
    <form-login-page>login.jsp</form-login-page>
    <form-error-page>notAuthenticated.jsp</form-error-page>
  </form-login-config>
</login-config>

<security-role>
  <description>some description</description>
  <role-name>administrator</role-name>
</security-role>
```

### 인터프라이즈 빈의 홈레퍼런스 정보 (로컬 레퍼런스 정보 포함) ( Interprize Bean's Home Reference [Local Reference])

이제는 과거의 유산이 되어버린 ejb에 대한 설정 정보가 위치하는 곳입니다. ejb에 대한 정보 설정은 다음과 같이 설정됩니다.

```xml
<env-entry>
  <description>some description</description>
  <env-entry-name>MinimumValue</env-entry-name>
  <env-entry-value>5</env-entry-value>
  <env-entry-type>java.lang.Integer</env-entry-value>
</env-entry>
<ejb-ref>
  <description>some description</description>
  <ejb-ref-name>Employee Bean</ejb-ref-name>
  <ejb-ref-type>EmployeeBean</ejb-ref-type>
  <home>com.foobar.employee.EmployeeHome</home>
  <remote>com.foobar.employee.Employee</remote>
</ejb-ref>
```

## web.xml 등록법

servlet의 변화에 따라 web.xml은 많은 확장이 있었습니다. web.xml의 정보는 다음 3가지 방법으로 표현될 수 있습니다.

* WEB-INF/web.xml 을 이용해서 등록하는 방법

> 가장 많이 사용되고 있으며, servlet 1.0부터 지금까지 구현된 방법입니다. 가장 대중적입니다.

* META-INF/web-fragment.xml 을 이용하는 방법

> servlet 3.0 부터 지원되는 방법입니다. web.xml은 web application에 종속되는 방법입니다. web application 뿐 아니라, jar로 개발된 project에서 가지고 있는 META-INF 폴더 안의 web-fragment.xml을 이용해서 web을 구동하는 방법을 지원하고 있습니다. web-fragment.xml은 web.xml과 내용이 동일합니다. 이 방법이 나오게 된 이유는 Servlet 3.0에서부터는 web container를 포함한 설치형 war가 배포가 가능합니다. 지금까지는 web application을 tomcat과 같은 web container에 배포하는 것이 일반적이였지만, web container를 포함한 한개의 web application으로 구동하는 방법을 servlet 3.0에서 이제는 지원하고 있습니다.

* javax.servlet.ServletContainerInitializer interface를 구현한 객체를 이용하는 방법

> servlet 3.0 부터 지원되는 방법입니다. web.xml이 없이, application에서 작성한 ServletContainerInitializer를 구현하는 것으로 web.xml 대신 페이지를 작성할 수 있도록 지원하고 있습니다.

3가지 방법중 3번의 방법이 조금 재미있습니다. ServletContainerInitializer를 구현한 객체를 이용하게 되면, 다음과 같은 설정들만 사용 가능합니다.

* Listener 구성
* filter 구성
* Session Listener 구성
* ejb 구성

web.xml의 핵심이 구현되긴하지만, 자주 사용되는 다음 항목들이 제공되지 않습니다.

* session-config
* error-page

이 두항목을 사용하기 위해서는 web.xml과 ServletContainerInitializer를 구현한 객체를 같이 사용해주면 됩니다. 그럼 ServletContainer는 web.xml을 먼저 로딩 후, 객체를 로드해서 사용하게 됩니다.

## Gradle을 이용한 Hello World Web App

먼저 기본적으로 기존에 사용하던 `java-application`으로부터 시작합니다.

```cmd
gradle init --type=java-application
```

구성된 build.gradle에서 다음을 추가합니다.

```groovy
apply plugin: 'war'
```

web application을 개발할 예정이기 때문에, `war`를 추가하는 것이 필요합니다.
이제 IntelliJ에서 Project를 open 시킵니다.

먼저 web에서 사용될 resource를 추가하도록 합니다. `/src/main/webapp` 폴더를 추가합니다. 그리고 내부에 `META-INF`, `WEB-INF` 폴더를 생성하여 추가합니다.
만들어진 전체 folder구조는 다음과 같습니다.

```cmd
.
├── build.gradle
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── xyzlast
    │   ├── resources
    │   └── webapp
    │       ├── META-INF
    │       ├── WEB-INF
    │       │   ├── jsp
    │       │   └── web.xml
    │       └── index.jsp
    └── test
        ├── java
        └── resources
```

신규 Servlet을 생성합니다. package는 자신이 원하는대로 넣어주시면 됩니다.

```java
public class HelloWorldServlet extends HttpServlet {
    private static final long serialVersionUID = -4589997486063553833L;

    private String message;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setAttribute("date", new Date());
        req.setAttribute("message", this.message);
        req.getRequestDispatcher("/WEB-INF/jsp/hello.jsp").forward(req, resp);
    }

    @Override
    public void destroy() {
        super.destroy();
    }

    @Override
    public void init() throws ServletException {
        super.init();
        message = "Hello World from Servlet";
    }
}
```

꼭 `HttpServlet` 객체를 상속받아 처리해야지 됩니다. 이제 `Servlet`에서 사용할 `jsp`파일을 만들어줍니다. 위치는 `src/main/webapp/WEB-INF/jsp/hello.jsp`입니다.
이 jsp 파일은 다음과 같이 구성가능합니다.

```jsp
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Hello World</title>
    </head>
    <body>
        <p>${message}, ${date}</p>
    </body>
</html>
```

이제 `src/main/webapp/WEB-INF`안에 `web.xml`을 넣어줍니다. `web.xml`은 다음과 같습니다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
         version="2.4">
    <display-name>HelloWorld Application</display-name>
    <servlet>
        <servlet-name>HelloServlet</servlet-name>
        <servlet-class>com.xyzlast.servlet.HelloWorldServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>HelloServlet</servlet-name>
        <url-pattern>/hello.do</url-pattern>
    </servlet-mapping>
</web-app>
```

`web.xml`에서는 `/hello.do`에 `HelloWorldServlet`을 연결시키는 코드가 있는 것을 볼 수 있습니다.

이제 구성된 web application을 실행시킬 수 있어야지 됩니다. 개발중인 web application을 실행하기 위해서는 여러가지 방법들이 있습니다.
다들 가장 많이 알고 계신 방법들이, 지금 실행중인 tomcat의 webapp folder에 war 파일을 배포하여 실행하면 됩니다.

war 파일을 만드는 방법은 다음과 같습니다.

```cmd
gradle war
```

`BUILD SUCCESS`가 나온 후, `/build/libs`에 보시면 war파일을 확인 할 수 있습니다.

그런데, 이 방법은 너무나 비효율적입니다. Servlet을 하나 추가할때마다 war 파일을 만들고, war 파일을 카피하는 일들을 계속해서 반복해야지 됩니다. 이를 편하게 해주는 plugin을 gradle에서 제공합니다.

```groovy
apply from: 'https://raw.github.com/akhikhl/gretty/master/pluginScripts/gretty.plugin'
gretty {
    contextPath = '/'
}
```

추가 후, `grdle appRun`을 통해 실행하면 다음과 같은 메세지를 볼 수 있습니다.

```cmd
16:55:04 INFO  Jetty 9.2.22.v20170606 started and listening on port 8080
16:55:04 INFO  HelloWorld Application runs at:
16:55:04 INFO    http://localhost:8080/

> Task :appRun
Press any key to stop the server.
```

이제 web browser에서 `http://localhost:8080/hello.do`로 접근하면 만든 web page를 보실 수 있습니다.

![](/images/14/03.png)

## Summary

Java에서 Web을 표시하는 기본적인 방법에 대해서 알아봤습니다. `web.xml`의 내용 구성과 개발시에 필요한 `build.gradle`구성은 익혀두는 것이 필요합니다.
