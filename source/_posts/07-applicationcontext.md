---
title: 07. ApplicationContext
date: 2018-02-21 22:44:36
tags: java, spring
---

# ApplicationContext

Spring에서 ApplicationContext는 IoC, AOP를 설정하는 Container 또는 Configuration이라고 할 수 있습니다.
ApplicationContext는

* bean의 집합
* bean에 대한 Map
* bean의 life cycle 정의
* bean에 대한 정의
* AOP에 의한 bean의 확장에 대한 정의

를 포함하고 있습니다. ApplicationContext에 대해서 좀 더 깊게 들어가보도록 하겠습니다.

## ApplicationContext interface

Application Context는 `org.springframework.context.ApplicationContext` interface를 상속받은 객체입니다. interface의 정의는 다음과 같습니다.

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
        MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    String getId();
    String getApplicationName();
    String getDisplayName();
    long getStartupDate();
    ApplicationContext getParent();
    AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;
}
```

ApplicationContext는 bean에 대한 집합이라고 했습니다. bean은 일반적으로 POJO class로 구성이 되게 됩니다. bean과 POJO의 정의는 다음과 같습니다.

### Java Bean

원 목적은 Servlet에서 Java 객체를 가지고 와서 사용하기 위해서 작성된 객체입니다. **단순한 객체이며, 사용이 편리해야지 된다.** 라는 것을 원칙으로 가지고 있습니다. 특징으로는

* property를 갖는다. (private 변수와 get/set method를 갖는다.)
* serialization이 가능하다.

### POJO (Plan Old Java Object)

POJO 객체는 특정 기술과 Spec에 독립적인 객체로 만들어지는 것을 원칙으로 삼습니다. 자신이 속한 package에 속한 POJO 객체 이외에는 Java의 기본 객체만을 이용해서 작성하는 것이 원칙입니다. 또한, 확장성을 위해 자신이 속한 package의 POJO 객체가 아닌 POJO 객체의 interface를 속성으로 갖는 bean 객체로서 만들어지는 것이 일반적입니다. 지금까지 작성된 Book, User, UserHistory 객체의 경우에 POJO 객체라고 할 수 있습니다.

bean에 대한 집합인 ApplicationContext는 다음과 같은 특징을 갖습니다.

* bean id, name, alias로 구분이 가능한 bean들의 집합입니다.
* life cycle을 갖습니다. (singleton, prototype)
* property는 value 또는 다른 bean을 참조합니다.

ApplicationContext는 bean들의 집합적인 특징 뿐 아니라, bean들의 loading 도 역시 담당하고 있습니다. ApplicationContext의 정보는 일반적으로 `@Configuration`과 `xml`을 이용합니다. 또한 수동으로 ApplicationContext를 구성하는 것 역시 가능합니다.

다음은 `StaticApplicationContext`을 이용해서 수동으로 ApplicationContext를 구성하는 것을 보여줍니다.

```java
@Test
public void registerApplicationContext() {
    StaticApplicationContext ac = new StaticApplicationContext();
    ac.registerSingleton("hello1", Hello.class);
    Hello hello = ac.getBean("hello1", Hello.class);
    assertThat(hello).isNotNull();
    Hello hello2 = ac.getBean("hello1", Hello.class);
    assertThat(hello).isSameAs(hello2);
}

@Test
public void registerApplicationContextWithPrototype() {
    StaticApplicationContext ac = new StaticApplicationContext();
    ac.registerPrototype("hello1", Hello.class);
    Hello hello = ac.getBean("hello1", Hello.class);
    assertThat(hello).isNotNull();
    Hello hello2 = ac.getBean("hello1", Hello.class);
    assertThat(hello).isNotSameAs(hello2);
}
```

registerApplicationContext와 registerApplicationContextWithProtytype은 Hello 객체를 `Singleton` 또는 `Prototype`으로 등록하게 됩니다. Singleton은 ApplicationContext에 등록된 모든 객체들을 재사용하게 되는데, registerPrototype으로 등록된 객체들은 ApplicationContext에서 얻어낼 때마다 객체를 다시 생성해서 얻어내게 됩니다. 이는 `@Configuration`으로는 다음과 같이 구성됩니다.

```java
@Bean(value = "hello1")
@Scope(value = BeanDefinition.SCOPE_SINGLETON)
public Hello hello() {
    return new Hello();
}

@Bean(value = "hello2")
@Scope(value = BeanDefinition.SCOPE_PROTOTYPE)
public Hello hello1() {
    return new Hello();
}
```

위 코드는 다음 xml로 표현이 가능합니다.

```xml
<bean id="hello1" class="com.xyzlast.ac.Hello" scope="singleton"/>
<bean id="hello2" class="com.xyzlast.ac.Hello" scope="prototype"/>
```

## ApplicationContext에서 Bean의 등록 방법

### annotation 을 이용하는 경우 - singleton/prototype

주로 사용되는 방법입니다. `@Component`, `@Service`로 선언된 Bean들을 `@ComponentScan`을 통해서 검색해서 등록합니다.
이 때, 다음 두가지를 설정할 수 있습니다.

* 객체의 LifeCycle: `@Scope`를 통해 `singleton/prototype`을 설정 가능합니다.
* 초기 실행 method: `@PostConstruct`를 통해 객체의 생성 이후에 실행할 method를 지정할 수 있습니다.

```java
@Component
public class Hello {
  public Hello() {

  }

  @PostConstruct
  public void doInit() {
    //NOTE: init method.
  }
}
```

### ApplicationContext에서의 FactoryPattern 이용

객체를 생성할 때, 무언가 초기화를 해줘야지 되는 경우가 왕왕 있습니다. `@PostConstruct`와 같이 단순하게 처리되는 경우도 있지만, 값을 설정하거나, TCP connection을 만들어주거나 하는 등의 초기 구성을 해줘야지 되는 경우가 꽤 있지요. 이때 주로 사용되는 것이 `Factory Pattern`입니다. `ApplicationContext`에서는 `FactoryBean<T>`을 통해 Factory Pattern을 지원합니다.

```java
public interface FactoryBean<T> {
  @Nullable
  T getObject() throws Exception;

  @Nullable
  Class<?> getObjectType();

  default boolean isSingleton() {
    return true;
  }
}
```

이와 같은 inteface를 구현한 객체는 `Hello`의 경우 다음과 같이 구성될 수 있습니다.

```java
public class HelloFactoryBean implements FactoryBean<Hello> {
  @Override
  public Hello getObject() throws Exception {
    Hello hello = new Hello();
    return hello;
  }

  @Override
  public Class<?> getObjectType() {
    return Hello.class;
  }
}
```

이제 이 코드를 `@Configuration`에 등록시켜줍니다.

```java
@Configuration
public class DomainConfiguration {
  @Bean
  public HelloFactoryBean helloFactoryBean() {
    return new HelloFactoryBean();
  }
}
```

이렇게 등록된 `ApplicationContext`의 경우에는 다음 2개의 객체를 얻어낼 수 있습니다.

* HelloFactoryBean
* Hello

객체의 이름 그대로 `Factory pattern`을 사용해서 `Bean`을 얻어낼 수 있도록 지원합니다.

## ApplicationContext의 등록방법

applicationContext 의 등록방법은 spring에서 계속해서 발전되어가는 분야중 하나입니다. 총 3가지의 방법으로 나눌 수 있으며, 이 방법들은 같이 사용되는 것도 가능합니다.

### applicationContext.xml만을 이용, 전체 Bean을 모두 각각 등록하는 방법

bookDao를 이용할 때, 처음 사용한 방법입니다. bean을 선언하고, id값을 이용해서 사용하는 방법으로 이 방법은 가장 오랫동안 사용해왔기 때문에 많은 reference들이 존재합니다. 그렇지만, 객체가 많아질수록 파일의 길이가 너무나 길어지고 관리가 힘들어지는 단점 때문에 요즘은 잘 사용되지 않습니다.

### @annotation과 aplicationContext.xml을 이용하는 방법

@Component, @Repository, @Service, @Controller, @Value 등을 사용하고, component-scan 을 이용해서 applicationContext.xml에서 등록하는 방법입니다. 이 방법은 예전에 가장 많이 사용되었던 방법입니다. applicationContext.xml의 길이가 적당히 길고, 구성하기 편하다는 장점을 가지고 있습니다.

### @Configuration을 이용한 applicationContext 객체를 이용하는 방법

Spring에서 근간에 밀고 있는 방법입니다. 이 방법은 다음과 같은 장점을 가지고 있습니다.

* 이해하기 쉽습니다. - xml에 비해서 사용하기 쉽습니다.
* 복잡한 Bean 설정이나 초기화 작업을 손쉽게 적용할 수 있습니다. - 프로그래밍 적으로 만들기 때문에, 개발자가 다룰수 있는 영역이 늘어납니다.
* 작성 속도가 빠릅니다. - eclipse에서 java coding 하는것과 동일하게 작성하기 때문에 작성이 용의합니다.

개인적으로는 2, 3번 방법을 모두 알아둬야지 된다고 생각합니다. 이유는 2번 방법의 경우, 기존에 가장 많이 사용되었던 방법입니다. legacy 코드에 대한 지원을 하기 위해서는 이 방법을 다룰줄 알아야지 됩니다. 최근은 당연히 3번 방법입니다. 최근에는 `xml`을 사용하는 예시조차 찾기 힘들정도로 3번 방법으로 집중되어가는 중입니다.

그럼, 이 3가지 방법을 모두 이용해서 기존의 BookStore를 등록하는 applicationContext를 한번 살펴보도록 하겠습니다.

### applicationContext.xml만을 이용하는 방법

초기 Spring 2.5 이하 버젼에서 지원하던 방법입니다. 일명 xml 지옥이라고 불리우는 어마어마한 xml을 자랑했습니다. 최종적으로 만들어져 있는 applicationContext.xml의 구성은 다음과 같습니다. Transaction annotation이 적용되지 않기 때문에, Transaction에 대한 AOP code 까지 추가되는 엄청나게 긴 xml을 보여주고 있습니다.

```xml
<?xml version="1.0" encoding="UTF-8"?><beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd">
  <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource" />
  </bean>
  <bean id="dataSource" class="com.jolbox.bonecp.BoneCPDataSource"
    destroy-method="close">
    <property name="driverClass" value="${connect.driver}" />
    <property name="jdbcUrl" value="${connect.url}" />
    <property name="username" value="${connect.username}" />
    <property name="password" value="${connect.password}" />
    <property name="idleConnectionTestPeriodInMinutes" value="60" />
    <property name="idleMaxAgeInMinutes" value="240" />
    <property name="maxConnectionsPerPartition" value="30" />
    <property name="minConnectionsPerPartition" value="10" />
    <property name="partitionCount" value="3" />
    <property name="acquireIncrement" value="5" />
    <property name="statementsCacheSize" value="100" />
    <property name="releaseHelperThreads" value="3" />
  </bean>
  <context:property-placeholder location="classpath:spring.properties" />
  <bean id="bookDao" class="com.xyzlast.bookstore02.dao.BookDaoImpl">
    <property name="jdbcTemplate" ref="jdbcTemplate"/>
  </bean>
  <bean id="userDao" class="com.xyzlast.bookstore02.dao.UserDaoImpl">
    <property name="jdbcTemplate" ref="jdbcTemplate"/>
  </bean>
  <bean id="historyDao" class="com.xyzlast.bookstore02.dao.HistoryDaoImpl">
    <property name="jdbcTemplate" ref="jdbcTemplate"/>
  </bean>
  <bean id="userService" class="com.xyzlast.bookstore02.services.UserServiceImpl">
    <property name="bookDao" ref="bookDao"/>
    <property name="userDao" ref="userDao"/>
    <property name="historyDao" ref="historyDao"/>
  </bean>
  <bean id="bookService" class="com.xyzlast.bookstore02.services.HistoryServiceImpl">
    <property name="bookDao" ref="bookDao"/>
    <property name="userDao" ref="userDao"/>
    <property name="historyDao" ref="historyDao"/>
  </bean>

  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
  </bean>
  <bean id="transactionManagerAdvisor" class="com.xyzlast.bookstore02.utils.TransactionAdvisor">
    <property name="transactionManager" ref="transactionManager"/>
  </bean>

  <bean
    class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <property name="beanNames">
      <list>
        <value>userService</value>
        <value>bookService</value>
      </list>
    </property>
    <property name="interceptorNames">
      <list>
        <value>transactionManagerAdvisor</value>
      </list>
    </property>
  </bean>
</beans>
```

### @annotation + applicationContext.xml을 이용한 방법

@Repository, @Component, @Service를 이용해서 편한 방법을 제공합니다. 특히 component-scan과 @Autowired를 이용하면 위 applicationContext.xml을 효과적으로 줄일 수 있습니다.

```xml
<?xml version="1.0" encoding="UTF-8"?><beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd">
  <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource" />
  </bean>
  <bean id="dataSource" class="com.jolbox.bonecp.BoneCPDataSource"
    destroy-method="close">
    <property name="driverClass" value="${connect.driver}" />
    <property name="jdbcUrl" value="${connect.url}" />
    <property name="username" value="${connect.username}" />
    <property name="password" value="${connect.password}" />
    <property name="idleConnectionTestPeriodInMinutes" value="60" />
    <property name="idleMaxAgeInMinutes" value="240" />
    <property name="maxConnectionsPerPartition" value="30" />
    <property name="minConnectionsPerPartition" value="10" />
    <property name="partitionCount" value="3" />
    <property name="acquireIncrement" value="5" />
    <property name="statementsCacheSize" value="100" />
    <property name="releaseHelperThreads" value="3" />
  </bean>
  <context:property-placeholder location="classpath:spring.properties" />
  <context:component-scan base-package="com.xyzlast.bookstore02.dao" />
  <context:component-scan base-package="com.xyzlast.bookstore02.services" />
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
  </bean>
  <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

### @annotation + @Configuration을 이용한 방법

이 방법은 xml을 아애 없애버릴 수 있습니다. xml이 제거된 ApplicationContext의 내용은 다음과 같습니다.

```java
@Configuration
@PropertySource("classpath:spring.properties")
@ComponentScan(basePackages = {"com.xyzlast.bookstore02.dao", "com.xyzlast.bookstore02.services"})
@EnableTransactionManagement
public class BookStoreConfiguration {
    @Autowired
    private Environment env;

    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        PropertySourcesPlaceholderConfigurer configHolder = new PropertySourcesPlaceholderConfigurer();
        return configHolder;
    }

    @Bean
    public DataSource dataSource() {
        BoneCPDataSource dataSource = new BoneCPDataSource();
        dataSource.setUsername(env.getProperty("connect.username"));
        dataSource.setPassword(env.getProperty("connect.password"));
        dataSource.setDriverClass(env.getProperty("connect.driver"));
        dataSource.setJdbcUrl(env.getProperty("connect.url"));
        dataSource.setMaxConnectionsPerPartition(20);
        dataSource.setMinConnectionsPerPartition(3);
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate() {
        JdbcTemplate template = new JdbcTemplate();
        template.setDataSource(dataSource());
        return template;
    }

    @Bean
    public PlatformTransactionManager transactionManager() {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager(dataSource());
        return transactionManager;
    }
}
```

지금까지 작성하던 xml과는 완전히 다른 모습의 ApplicationContext입니다. ApplicationContext Config 객체는 다음과 같은 특성을 갖습니다.

* @Configuration annotation을 갖는다.
* bean으로 등록되는 객체는 method로 관리되고, @Bean annotation을 갖습니다.
* method의 이름이 `<bean id="">`값과 매칭됩니다.
* component-scan, property-place-holder의 경우, class의 annotation으로 갖습니다.
* @EnableTransactionManagement와 같이 @Enable** 로 시작되는 annotation을 이용해서 전역 annotation을 구성합니다.
* Properties 파일을 사용하기 위해서는 반드시 static method로 PropertySourcesPlaceholderConfigurer를 return 시켜줘야지 됩니다.

### xml + @annotation + @Configuration을 이용한 방법

위에서는 3가지 방법이라고 했지만, 한가지 방법이 더 있습니다. 모두를 다 섞어서 사용하는 방법입니다. 기본은 `@Configuration`에서 시작합니다.

```java
@Configuration
@ImportResource("classpath:/config/spring.xml")
public class AppConfig {

}
```

용도에 따라, 다른 `@Configuration`을 결합해서 사용할 필요도 있습니다. 다음과 같이 확장이 가능합니다.

```java
@Configuration
@Import({ AppConfigWeb.class })
@ImportResource("classpath:/config/spring.xml")
public class AppConfig {

}
```

## Summay

ApplicationContext에 대한 기본 개념을 정리해봤습니다. ApplicationContext는 Spring의 핵심 기능입니다. DI를 통한 IoC를 가능하게 하고, AOP에 대한 설정 등 모든 Spring에서 하는 일이 설정되어 있는 것이 ApplicaionContext라고 할 수 있습니다. 이에 대한 설정 및 구성을 명확하게 알아놓을 필요가 있습니다. 그리고, 내부에서 어떤 일을 해서 Spring에서 하는 이런 일들이 가능하게 되는지에 대한 이해가 필요합니다. 마지막으로 ApplicationContext를 구성하는 방법에 대해서 알아봤습니다. 제공되는 3가지 방법에 대해 자유자재로 사용할 수 있는 능력이 필요합니다.