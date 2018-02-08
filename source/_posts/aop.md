---
title: 06.AOP
date: 2018-02-08 08:21:07
tags: java, spring, aop
---
# AOP

지금까지 구성된 테스트 코드의 마지막에 코드를 하나 추가해보도록 하겠습니다.

```java
@Test
public void rentTest() {
    UserService userService = applicationContext.getBean(UserService.class);
    UserDao userDao = applicationContext.getBean(UserDao.class);
    BookDao bookDao = applicationContext.getBean(BookDao.class);
    User user = userDao.getAll().get(0);
    Book book = bookDao.getAll().get(0);
    book.setBookStatus(BookStatus.CanRent);
    book.setRentUserId(null);
    bookDao.update(book);

    assertThat(userService.rent(user.getId(), book.getId())).isTrue();
    assertThat(userService).isInstanceOf(UserServiceImpl.class);
}
```

실행해보면 테스트 오류가 발생되는 것을 볼 수 있습니다.

```cmd
java.lang.AssertionError:
Expecting:
  <com.xyzlast.bookstore.service.UserServiceImpl@7bbc8656>
to be an instance of:
  <com.xyzlast.bookstore.service.UserServiceImpl>
but was instance of:
  <com.sun.proxy.$Proxy31>
```

그럼 기존의 `@Transactional`을 제거한 후, 테스트를 돌리면 테스트오류가 발생되지 않습니다. 처음보는 객체이름이 보입니다. 거기에 이제는 최대한 사용하지 말라고 되어 있는 `com.sun` package의 객체입니다. 어떻게 이런 일이 발생하게 되는지 알아보도록 하겠습니다.

이 부분을 이해하기 위해서는 Spring의 이제 2번째 개념인 AOP에 대한 이해가 필요합니다. AOP에 대한 설명을 하기 위해 기존 `PlatformTransactionManager`를 사용한 코드를 먼저 참고합니다.

```java
@Override
public boolean rent(int userId, int bookId) {
    TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
        User user = userDao.get(userId);
        Book book = bookDao.get(bookId);

        book.setRentUserId(user.getId());
        book.setStatus(BookStatus.RentNow);
        bookDao.update(book);

        user.setPoint(user.getPoint() + 10);
        user.setLevel(userLevelLocator.getUserLevel(user.getPoint()));

        UserHistory history = new UserHistory();
        history.setUserId(userId);
        history.setBookId(book.getId());
        history.setAction(HistoryActionType.RENT);

        userDao.update(user);
        userHistoryDao.add(history);
        this.transactionManager.commit(status);
        return true;
    } catch(Exception ex) {
        this.transactionManager.rollback(status);
        throw ex;
    }
}
```

위 코드를 보면 Transaction의 경계설정 코드와 BL 코드가 복잡하고 연결이 되어있는것 같이 보이지만, 명확하게 코드는 분리가 될 수 있습니다. transactionManager를 사용하는 코드와 BL 코드로 분리가 깔끔하게 가능합니다. 여기서 전에 사용한 Template-callback 으로 변경시키는 것 역시 쉽게 가능합니다.

```java
public boolean rent(final int userId, final int bookId) {
    doBLWithTransaction(new BusinessLogic() {
        @Override
        public void doProcess() {
            User user = userDao.get(userId);
            Book book = bookDao.get(bookId);

            book.setRentUserId(user.getId());
            book.setStatus(BookStatus.RentNow);
            bookDao.update(book);

            user.setPoint(user.getPoint() + 10);
            user.setLevel(userLevelLocator.getUserLevel(user.getPoint()));

            UserHistory history = new UserHistory();
            history.setUserId(userId);
            history.setBookId(book.getId());
            history.setAction(HistoryActionType.RENT);

            userDao.update(user);
            userHistoryDao.add(history);
        }
    });
    return true;
}

interface BusinessLogic {
    void doProcess();
}

private void doBLWithTransaction(BusinessLogic loc) {
    TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
        loc.doProcess();
        this.transactionManager.commit(status);
    } catch(Exception ex) {
        this.transactionManager.rollback(status);
        throw ex;
    }
}
```

그런데, 이와 같은 코드는 지금까지 만들어진 `Repository Pattern`과 조금 다른 경계를 갖습니다.  DB에 대한 내용은 Repository측으로, BL측은 Service로 분리가 되어 있던 것이, Transaction에 대한 코드가 BL과 같이 있으면서 OOP의 기본 원칙인 `단일책임원칙`이 어긋나기 시작합니다. 그리고, 이런 코드는 계속되는 반복에 의해서 만들어지기 때문에 코드를 관리하는데 조금 복잡해보이는것이 사실입니다. 이와 같이 분리된 코드를 도식화 하면 다음과 같습니다.

![](/images/aop/01.jpg)

그렇다면, Transaction에 대한 코드를 아애 분리하는 방법을 생각해볼 수 있습니다. BL과 Transaction을 아애 분리시키고, BL을 사용하는 측에서는 Transaction을 사용하는 객체를 사용하는 것입니다. 다음은 분리된 Transaction 코드 패턴을 볼 수 있습니다.

```java
public class UserServiceTx implements UserService {
    @Autowired
    PlatformTransactionManager transactionManager;
    @Autowired
    UserService userServiceImpl;

    interface BusinessLogic {
        void doProcess();
    }

    private void doBLWithTransaction(BusinessLogic loc) {
        TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            loc.doProcess();
            this.transactionManager.commit(status);
        } catch(Exception ex) {
            this.transactionManager.rollback(status);
            throw ex;
        }
    }

    @Override
    public boolean rent(final int userId, final int bookId) {
        doBLWithTransaction(new BusinessLogic() {
            @Override
            public void doProcess() {
                userService.rent(userId, bookId);
            }
        });
        return true;
    }
}
```

위 코드는 실제 구현된 `UserServiceImpl`을 한번 더 감싼 `UserServiceTx`를 또 만들어서 사용하는 것입니다. `UserServiceTx`를 사용하는 측에서 보면 다음과 같이 실행되게 됩니다.

![](/images/aop/02.jpg)

이와 같은 개발 패턴을 proxy pattern이라고 합니다. proxy pattern의 정의는 다음과 같습니다.

**Proxy패턴은 object를 호출할 때, 부가적인 처리를 실행할 수 있도록 하기 위한 패턴입니다. 이와 동시에 그 오브젝트의 이용자(클라이언트)에 대한 변경을 최소화할 수도 있다. Proxy 패턴을 잘 사용한다면, 어플리케이션의 사용편리성을 향상시키는데 이용됩니다.**

말이 어렵습니다. 간단히 말을 풀면, UserService를 실행하는 Test 객체 입장에선 UserServiceImpl을 실행을 하나, UserServiceTx를 실행하나 동일한 interface를 실행하게 됩니다.  동일한 Interface로 접근을 하되, 다른 객체를 통해서 원 객체를 접근하게 만드는 방식을 Proxy pattern 이라고 합니다.

또한, Proxy pattern은 거의 또 다른 pattern을 동시에 실행하게 됩니다. 그건 decorate pattern 입니다. decorate patten의 정의는 다음과 같습니다.

**기능의 '장식'(Decorator)이라는 개념을 클래스로 만들어 분화시킨 후, 이를 기존 객체가 참조하는 형태로 덧붙여나갈 수 있게 한다. Decorator와 꾸미는 대상이 되는 클래스는 같은 기반 클래스를 상속받는다. 그리고 두 클래스 모두 새로운Decorator 객체를 참조할 수 있는 구조로 만든다.**

말이 더 어렵습니다. decorate pattern은 동일한 interface에 접근되는 객체에 새로운 기능이 추가되는 것을 decorate pattern이라고 합니다. proxy pattern과 decorate pattern은 매우 헛갈리면서 같이 사용이 되는 것이 일반적입니다. 접근되는 객체에 대한 관점은 proxy pattern이라고 생각하시면 되고, decorate pattern은 접근된 기능에 추가 기능을 같이 실행시키는 것을 의미합니다.

위 객체를 다시 설명을 하면, UserServiceTx라는 Proxy를 만들어서 UserServiceImpl에 접근을 하게 되고, UserServiceTx는 decorate pattern을 이용해서 transaction 기능을 추가하고 있습니다.

최종적으로, Spring의 @Transaction 역시 Proxy Pattern과 Decorate Pattern이 결합된 형태로 구성되어 있습니다. Transaction 기능을 추가하기 위해서 Decorate Pattern으로 기능이 추가 되어 있으며, 실 객체가 아닌 (예제에서는 UserServiceImpl) Decorate가 추가된 객체에 접근하도록 객체의 접근을 제어한 Proxy Pattern이 결합되어 있습니다. 이 부분을 구현하는 Java의 기본 코드 구조를 Dynamic Proxy라고 합니다.

## Dynamic Proxy

이러한 Proxy 객체를 어떻게 만들어주게 되는 걸까요? 지금 저희 코드는 UserServiceTx라는 Proxy를 작성해줬습니다. Spring과 같은 Framework는 범용적이며, 가변적인 Proxy를 만드는 방법들을 가지고 있습니다. 가장 대표적인 것은 reflection입니다. Spring은 기본적으로 reflection을 통해서 이 부분을 처리합니다. 이 부분에 대해서 좀 더 깊게 들어가보도록 하겠습니다. 먼저 간단한 Relection 코드를 확인해보도록 하겠습니다.

```java
@Test
public void doDynamicReflection() throws Exception {
    String name = "ykyoon";
    Method lengthMethhod = String.class.getMethod("length");
    Object invokeResult = lengthMethhod.invoke(name);
    assertThat(invokeResult).isInstanceOf(Integer.class).isEqualTo(6);
}
```

위 코드를 보시면 좀 재미있는 내용들이 보이게 됩니다. String 객체의 length method를 문자열로 mehtod의 이름을 이용해서 호출할 수 있는 것을 확인할 수 있는데요.
java의 Method object는 각 객체의 method에 대한 정의를 갖는 객체입니다. 이를 이용해서 dynamic한 method 호출을 할 수 있지요. 이제 본격적인 Relection 코드를 확인해보도록 하겠습니다.

Dynamic Proxy에 대해서 깊게 알아보기 위해서 가장 먼저 알아봐야지 될 내용은  InvocationHandler interface입니다. InvocationHandler는 Proxy.newInstance에서 새로운 객체를 만들고, 그 객체에 Proxy를 통해서 접근하는 방식을 제공하는 interface입니다.

한번 예시를 사용해보도록 하겠습니다. InvocationHandler를 이용해서 객체의 모든 String output을 대문자로 자동으로 변경시키는 Proxy를 만들어보도록 하겠습니다.

먼저, Proxy의 target이 되는 interface와 객체입니다.

```java
public interface Hello {
    String sayHello(String name);
    String sayHi(String name);
    String sayThankYou(String name);
}

public class HelloImpl implements Hello {
    @Override
    public String sayHello(String name) {
        return "Hello " + name + "!!";
    }
    @Override
    public String sayHi(String name) {
        return "Hi " + name + "!!";
    }
    @Override
    public String sayThankYou(String name) {
        return "Thank you. " + name + "!!";
    }
}
```

그리고, 모든 output을 UpperCast로 변경시키는 InvocationHandler를 구성하도록 하겠습니다.

```java
public class UppercaseHandler implements InvocationHandler {
    private final Object target;

    public UppercaseHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String ret = (String) method.invoke(this.target, args);
        return ret.toUpperCase();
    }
}
```

코드는 매우 단순합니다.

그리고, Hello interface를 통해서 HelloImpl로 접근하는 Proxy 객체를 만들어보도록 하겠습니다.  HelloImpl에 대한 test 를 작성해서 다음 코드를 실행해보면 다음 결과를 얻어낼 수 있습니다.

```java
@Test
public void helloWorldReflection() throws Exception {
    Hello proxiedHello = (Hello) Proxy.newProxyInstance(getClass().getClassLoader(),
        new Class[] { Hello.class},
        new UppercaseHandler(new HelloImpl()));

    String output = proxiedHello.sayHello("ykyoon");
    System.out.println("Output은 다음과 같습니다. : " + output);
    System.out.println("Proxy의 이름은 다음과 같습니다. : " + proxiedHello.getClass().getName());
}
```

```cmd
com.intellij.rt.execution.junit.JUnitStarter -ideVersion5 -junit4 com.xyzlast.bookstore.aop.ReflectionTest,helloWorldReflection
Output은 다음과 같습니다. : HELLO YKYOON!!
Proxy의 이름은 다음과 같습니다. : com.sun.proxy.$Proxy4
```

Spring에서 @Transaction에서 보시던 결과가 나왔습니다. Spring에서 @Transaction은 다음과 같은 코드로 구성되어 있습니다.  (Sudo code입니다.)

```java
public class SpringTransactionProxy implements InvocationHandler {

    private final Object targetService;

    @Autowired
    private PlatformTransactionManager transactionManager;

    public SpringTransactionProxy(Object targetService) {
        this.targetService = targetService;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        TransactionStatus status = transactionManager.beginTransaction(new DefaultTransaction());
        try {
            method.invoke(this.targetService, args);
            transactionManager.commit(status);
        } catch(Exception ex) {
            transactionManager.rollback(status);
            throw ex;
        }
    }
}
```

잘 보시면, 기존의 template-callback pattern과 유사한 코드가 만들어집니다. 단, 구현 방법이 InvocationHandler를 이용한 Proxy객체를 만들어서, 그 객체를 통해 method의 전/후에 특정한 action을 집어 넣고 있는 방식을 사용하고 있는 것입니다. 이렇게 method의 전/후에 특정 action을 삽입하는 개발 방법을 AOP(aspect oriented programming)라고 합니다.

## Spring에서의 AOP 이용

AOP는 Transaction 뿐 아니라, 다른 method들에서도 이와 같은 Decorate와 Proxy patten들을 사용할 수 있는 멋진 방법들을 제공합니다. 지금까지 보던 Transaction에 대한 코드를 보시면 Transaction의 대상이 되는 method의 전/후에 transaction.begin() / commit()이 실행되고 있습니다. 좀더 이것을 발전시킨다면 특정 method가 시작되기 전에, 또는 method가 실행 된 후에,아니면 method에서 exception이 발생한 후에 수행되는 특정 InvocationHandler를 지정해주는 것도 가능하지 않을까? 라는 의문이 생깁니다. 이러한 개발 방법을 이용하면, 지금까지 공부해왔던 OOP에 대한 다른 개념으로 발전하게 됩니다.

OOP를 공부하면 가장 많이 나오는 말은 상속 그리고 구현입니다. extends, implements에 대한 이야기들이 대부분이지요. 이는 object의 종적인 연결관계를 나타냅니다. 그렇지만, 이와 같이 method의 횡적인 연결관계에 대한 논의입니다. OOP와는 조금 다른 부분의 이야기라고 생각합니다.

![](/images/aop/03.jpg)

이러한 개념은 여러 곳에서 사용될 수 있습니다. 예를 들어...

* 사용자의 권한 체크
* In/Out에 대한 로그 기능
* Return값에 특정한 값이 있는 경우에 공통되는 Action을 해야지 되는 경우
* Transaction과 같은 method의 실행에 있어서 전/후 처리를 해줘야지 될때
* Return값에 특정한 데이터 양식을 추가해야지 될 때
* Exception의 처리

제가 생각하는 기능들은 이정도인데, 다른 분들은 어떤 기능을 추가할 수 있을까요?

AOP의 확장은 거의 무한대에 가깝습니다. 기존의 OOP 적 사고방식을 크게 확장시킬 수 있는 개념이기도 하면서, 기존의 OOP의 설계를 무너트릴수도 있는 개념입니다. OOP는 종적, AOP는 횡적 객체의 확장이다. 라고 이해를 하시고 다음 용어들을 이해하시면 좀 더 이해가 편할 것 같습니다.

### Target

AOP의 Target이 되는 객체입니다. 지금까지 만들었던 Service객체 또는 HelloImpl과 같이 직접 AOP 당하는 객체를 Target이라고 합니다.

### Advice

위에 구성된 InvocationHandler에 의해서 사용될 interface가 구현된 객체입니다. 다른 객체에 추가적인 action을 확장시키기 위한 객체입니다. Spring은 총 3개의 Advice interface를 제공하고 있습니다. 아래의 interface를 상속받아 Advice를 구성합니다.

|name|description|
|---|---|
|MethodBeforeAdvice|method가 실행되기 전에 실행되는 Advicer를 구성합니다.|
|AfterReturningAdvice|method가 실행 후, return 값을 보내기 직전에 실행되는 Advicer를 구성합니다.|
|MethodInterceptor|method의 전/후 모두에 실행 가능한 Advicer를 구성합니다.|

### Pointcut

Advice를 적용할 Point를 선별하는 작업을 하는 객체를 말합니다. Spring에서는 @Transactional이 적용된 모든 method에 대한 Transaction Advice의 Pointcut은

```xml
<tx:annotation-driven transaction-manager="transactionManager" />
```

으로 선언되고 있습니다.

### Advisor

```
Advisor = Advice + PointCut
```

이라고 생각하시면 됩니다. 부가기능 + 적용대상이 반영된 interface라고 하면 설명이 좀더 쉽습니다.
이제 Spring에서 AOP를 다른 method에 적용해보도록 하겠습니다.

간단히 `AfterReturningAdvice`를 `UserService`에 적용하는 설정은 다음과 같습니다.

```java
public class ServiceMethodAdvice implements AfterReturningAdvice {
    @Override
    public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
        System.out.println(method.getName() + " is called");
    }
}
```

`Advice`를 선언한 후, 이제 `PointCut`과 `Target`을 섞어서 선언해줍니다. 이는 `@Configuration`에서 설정합니다.

```java
@Bean
public BeanNameAutoProxyCreator beanNameAutoProxyCreator() {
    BeanNameAutoProxyCreator beanNameAutoProxyCreator = new BeanNameAutoProxyCreator();
    beanNameAutoProxyCreator.setBeanNames("userServiceImpl");
    beanNameAutoProxyCreator.setInterceptorNames("methodInterceptor");
    return beanNameAutoProxyCreator;
}

@Bean(value = "methodInterceptor")
public ServiceMethodAdvice methodInterceptor() {
    return new ServiceMethodAdvice();
}
```

구성되어 있는 `AOP`는 객체중심으로 선언이 됩니다. 그런데, 요즘은 이와 같은 방법을 통해서 `AOP`를 선언하는 것을 권장하지 않습니다.

## AspectJ + Spring을 이용한 AOP

`AspectJ`는 AspectJ는 PARC에서 개발한 자바 프로그래밍 언어용 관점 지향 프로그래밍 (AOP) 확장 기능입니다. 이클립스 재단 오픈 소스 프로젝트에서 독립형 또는 이클립스로 통합하여 이용 가능합니다. AspectJ는 최종 사용자를 위한 단순함과 이용성을 강조함으로써 폭넓게 사용되는 AOP에 대한 데 팍토 표준입니다.

기본 Spring Proxy를 이용하는 것보다 performance + 확장성이 더 좋습니다. 먼저 `spring-aop`를 추가합니다.

```groovy
compile group: 'org.springframework', name: 'spring-aop', version: '5.0.2.RELEASE'
compile 'org.aspectj:aspectjrt:1.8.1'
compile 'org.aspectj:aspectjtools:1.8.1'
compile 'org.aspectj:aspectjweaver:1.8.1'
```

먼저 Target이 될 method를 결정합니다. 이 method를 결정하는 방법은 여러가지가 있습니다. 다음 6가지가 주로 사용됩니다.

* execution: 메소드 실행 결합점(join points)과 일치시키는데 사용된다.
* within: 특정 타입에 속하는 결합점을 정의한다.
* this: 빈 참조가 주어진 타입의 인스턴스를 갖는 결합점을 정의한다.
* target: 대상 객체가 주어진 타입을 갖는 결합점을 정의한다.
* args: 인자가 주어진 타입의 인스턴스인 결합점을 정의한다.
* annotation: method에 선언된 @Annotation을 기준으로 결합점을 정의한다.

다음은 Pointcut의 예시입니다.

|Pointcut|선택된 Joinpoints|
|---|---|
|execution(public * *(..))|public 메소드 실행|
|execution(* set*(..))|이름이 set으로 시작하는 모든 메소드명 실행|
|execution(* set*(..))|이름이 set으로 시작하는 모든 메소드명 실행|
|execution(* com.xyz.service.AccountService.*(..))|AccountService 인터페이스의 모든 메소드 실행|
|execution(* com.xyz.service.*.*(..))|service 패키지의 모든 메소드 실행|
|execution(* com.xyz.service..*.*(..))|service 패키지와 하위 패키지의 모든 메소드 실행|
|within(com.xyz.service.*)|service 패키지 내의 모든 결합점|
|within(com.xyz.service..*)|service 패키지 및 하위 패키지의 모든 결합점|
|this(com.xyz.service.AccountService)|AccountService 인터페이스를 구현하는 프록시 개체의 모든 결합점|
|target(com.xyz.service.AccountService)|AccountService 인터페이스를 구현하는 대상 객체의 모든 결합점|
|args(java.io.Serializable)|하나의 파라미터를 갖고 전달된 인자가 Serializable인 모든 결합점|
|@target(org.springframework.transaction.annotation.Transactional)|대상 객체가 @Transactional 어노테이션을 갖는 모든 결합점|
|@within(org.springframework.transaction.annotation.Transactional)|대상 객체의 선언 타입이 @Transactional 어노테이션을 갖는 모든 결합점|
|@annotation(org.springframework.transaction.annotation.Transactional)|실행 메소드가 @Transactional 어노테이션을 갖는 모든 결합점|
|@args(com.xyz.security.Classified)|단일 파라미터를 받고, 전달된 인자 타입이 @Classified 어노테이션을 갖는 모든 결합점|
|bean(accountRepository)|“accountRepository” 빈|
|!bean(accountRepository)|“accountRepository” 빈을 제외한 모든 빈|
|bean(*)|모든 빈|
|bean(account*)|이름이 'account'로 시작되는 모든 빈|
|bean(*Repository)|이름이 “Repository”로 끝나는 모든 빈|
|bean(accounting/*)|이름이 “accounting/“로 시작하는 모든 빈|

개인적으로는 Spring에서의 `@Transaction`의 처리와 동일하게 AOP선언에 맞게 신규 `@Annotation`을 만들어주고, 이를 Target으로 명확하게 지정하는 것입니다. 다음과 같이 구성하는 것입니다.

```java
@Target(value = ElementType.METHOD)
@Retention(value = RetentionPolicy.RUNTIME)
public @interface ServiceAopMethod {

}
```

그리고, 이를 `Advisor`에 다음과 같이 등록합니다.

```java
@Aspect
public class ServiceAdvisor {
    @Pointcut(value = "@annotation(com.xyzlast.bookstore.aop.ServiceAopMethod)")
    public void serviceMethodPointCut() {

    }
}
```

`AspectJ`는 다음 5개의 annotation을 제공합니다. 이는 다음과 같습니다.

|Annotation|Description|
|---|---|
|@Before|method가 call되기 전 입니다.|
|@Around|method를 wrap 시켜서 사용되는 형태입니다.|
|@After|method가 call이 모두 완료된 이후에 실행됩니다. finally와 동일한 형태라고 생각하시면 됩니다.|
|@AfterReturning|method가 정상적으로 실행된 이후, 실행됩니다.|
|@AfterThrowing|method에서 exception이 발생될 때 사용됩니다.|

이제 `Pointcut`과 `Advice`가 모두 추가된 `Advisor`는 다음과 같은 형태로 구성됩니다.

```java
@Aspect
public class ServiceAdvisor {

    @Pointcut(value = "@annotation(com.xyzlast.bookstore.aop.ServiceAopMethod)")
    public void serviceMethodPointCut() {

    }

    @Before("serviceMethodPointCut()")
    public void beforeTargetMethod(JoinPoint thisJoinPoint) {
        System.out.println("beforeTargetMethod");
    }

    @AfterReturning(pointcut = "serviceMethodPointCut()", returning = "retVal")
    public void afterReturningTargetMethod(JoinPoint thisJoinPoint, Object retVal) {
        System.out.println("AspectUsingAnnotation.afterReturningTargetMethod executed." +
            " return value is [" + retVal + "]");
    }

    @AfterThrowing(pointcut = "serviceMethodPointCut()", throwing = "exception")
    public void afterThrowingTargetMethod(JoinPoint thisJoinPoint,
                                          Exception exception) throws Exception {
        System.out.println("AspectUsingAnnotation.afterThrowingTargetMethod executed.");
        System.out.println("에러가 발생했습니다." + exception.getMessage());
        throw new RuntimeException("에러가 발생했습니다.", exception);
    }

    @After(value = "serviceMethodPointCut()")
    public void after(JoinPoint thisJoinPoint) {
        System.out.println("AspectUsing After");
    }

    @Around("serviceMethodPointCut()")
    public Object wrapResponseObject(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("before: ");
        Object ret = pjp.proceed();
        System.out.println("after: ");
        return ret;
    }
}
```

`@Aspect`로 `Advisor`임을 지정하는 일이 필요합니다. 이제 이 `Advisor`를 `@Configuration`에 등록하는 코드는 다음과 같습니다.

```java
@Configuration
@ComponentScan(basePackages = {
    "com.xyzlast.bookstore.dao",
    "com.xyzlast.bookstore.service"
})
@EnableTransactionManagement
@EnableAspectJAutoProxy
public class BookStoreConfig {
    @Bean
    public DataSource dataSource() {
    }

    @Bean
    public JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dataSource());
    }

    @Bean
    public PlatformTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }

    @Bean
    public ServiceAdvisor serviceAdvisor() {
        return new ServiceAdvisor();
    }
}
```

`@Bean`의 등록이 필요하고, `@EnableAspectJAutoProxy`를 *꼭* 선언하는 것을 기억해주세요.

## Summary

AOP는 spring에서 매우 중요한 개념입니다. 무엇보다 OOP의 한계인 횡적인 객체의 확장이 가능하게 하는 놀라운 기술중 하나입니다. 그렇지만, 알아야지 될 내용들도 무척 많습니다. 테스트 코드에서는 NameMatchMethodPointcut 만을 사용했지만, Pointcut의 종류가 매우 많습니다. Pointcut에 따라, 특정 객체, method, 그리고 advice에 따라 method의 실행 전/후를 결정을 해서 많은 일들을 할 수 있습니다.

개발을 하다보면 좀더 많은 활용법을 같이 생각해볼 수 있는 구조입니다. 꼭 깊게 공부를 해보시길 바랍니다. 그리고 추가로 이야기드릴것이 Spring은 AspectJ라는 AOP 개발 기법을 지원합니다. 지금까지 proxy를 이용한 AOP 방법이였다면, AspectJ는 compile 시에 기존 class를 변경시켜서 새로운 class를 만들어줍니다. 아애 변형된 객체를 구성시켜주는 것이지요. 이는 성능상에서 이익을 가지고 오고, AOP의 pointcut과 acdvice에 대한 테스팅이 매우 쉬운 장점을 가지고 있습니다. AspectJ에 대해서는 개인적인 공부를 좀 더 해주시길 바랍니다.

이제 JDBC를 직접 이용하는 Simple application의 제작이 모두 마쳐졌습니다. 모두들 수고하셨습니다. 기존의 버릇과는 다른 개발 방법에 대해 고민을 많이 하셨을 것 같은데, 이쪽까지 잘 해주셔서 정말 감사드립니다. 지금까지 하신 내용을 잊어먹지는 말아주세요.
