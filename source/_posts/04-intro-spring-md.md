---
title: 04.Spring Framework 소개
date: 2018-02-01 12:45:14
tags: study
---

# Spring Framework 소개

Spring Framework에 대한 소개와 왜 Spring을 써야지 되는지에 대한 당위성을 간단한 application을 작성하면서 알아보도록 하겠습니다.
application을 작성하면서 놀랍게 줄어드는 코드 양과 Spring의 강력함을 느끼실 수 있으실겁니다.

## Framework란?

먼저 Framework의 정의를 해야지 됩니다. 각자 개인마다 Framework에 대한 정의가 다를거라고 생각이 되지만, 가장 혼란스러운 개념이 Library와 Framework라고 생각이 됩니다.
Libray와 Framework는 매우 유사한 점을 가지고 있습니다. 우선적으로 많은 기능들을 별도의 file, class에서 제공하고 있습니다. 그리고 이를 묶어놓은 것들을 외부 Library 또는 외부 Framework라고 합니다.

그럼 이 둘의 차이는 무엇일까요?

Library는 특정 목적을 위한 function, class들을 모아놓은 꾸러미입니다. 우리는 Library를 특정 목적을 위해서 가져다가 사용할 수 있습니다. 예를 들어 JSON을 객체로 변경시키기 위해서 Jackson과 같은 Json Library를 가져다가 사용할 수 있습니다.
Framework를 application의 실행 flow를 변경시킵니다. Framework를 이용하게 된다면, 우리의 Application의 실행은 모두 Framework에 의해서 동작하게 되며 우리의 코드는 Framework가 가져다가 사용하는 일부의 part가 되게 됩니다.
한마디로 Framework란 자동차의 뼈대와 같이 Application의 기본 구조를 제공하고 있으며, 개발자들은 구현하고자하는 기능구현에 집중할 수 있도록 해주는 소프트웨어라고 할 수 있습니다.

## Spring Framework 란?

Spring Framework 란 엔터프라이즈급 자바 어플리케이션 개발에서 필요로 하는 경량형 어플리케이션 프레임워크입니다.
스프링 프레임워크는 J2EE[Java 2 Enterprise Edition] 에서 제공하는 대부분의 기능을 지원하기 때문에, J2EE를 대체하는 프레임워크로 자리잡고 있습니다. Spring이라는 이름의 기원은 기존 EJB로 대표되는 Enterprise Framework의 시대를 겨울(winter)로 정의하고, 이젠 봄(Spring)이 왔다 라는 의미로 지어졌습니다. 시작은 한권의 책의 예제에서부터 시작이 되었습니다.

Spring Framework는 다음과 같은 특징을 가지고 있습니다.

* 경량 컨테이너입니다. (light container) 스프링은 객체를 담고 있는 컨테이너로써 자바 객체의 생성과 소멸과 같은 라이프사이클을 관리하고, 언제든 필요한 객체를 가져다 사용할 수 있도록 도와주는 기능을 가지고 있습니다.
* DI[Dependency Injection] 패턴 지원을 지원합니다. (DI : 의존성 주입) 객체들간의 의존 관계를 쉽게 설정할 수 있습니다. 그로인해 객체들간의 느슨한 결합을 유지하고 직접 의존하고 있는 객체를 굳이 생성하거나 검색할 필요성이 없이 구성이 가능합니다. 이는 IoC(Inversion of Controller)로 이야기되기도 합니다. 정확히는 DI로 인한 IoC를 가능하게 하는 Framework라고 할 수 있습니다.
* AOP[Aspect Oriented Programming] 지원 (AOP : 측면 지향 프로그래밍 )합니다. AOP는 문제를 바라보는 관점을 기준으로 프로그래밍하는 기법이다. 이는 문제를 해결하기 위한 핵심 관심 사항과 전체에 적용되는 공통관심 사항을 기준으로 프로그래밍 함으로써 공통 모듈을 여러 코드에 쉽게 적용할 수 있도록 한다. 스프링은 자체적으로 프록시 기반의 AOP를 지원하므로 트랜잭션이나 로깅, 보안등과 같이 여러 모듈에서 공통적으로 필요하지만 실제모듈핵심은 아닌 기능들을 분리하여 각 모듈에 적용할 수 있도록 한다.

Spring Framework는 위의 3가지의 특징을 가진 Framework입니다. 또한, 부가적인 기능으로서 ruby on rails에서 표방한 non shared status web 개발을 지원하는 @MVC 역시 지원하고 있습니다.

![](/images/04/spring.png)

Spring Framework의 기본구조입니다. 위에서 말한 3가지의 특징은 Spring Core와 Spring AOP, Spring Context에 의하여 구성이 되어 있습니다. 나머지 ORM, WEB, DAO, WEB MVC의 경우에는 부가적 기능이라고 볼 수 있습니다.

이런식으로만 적어두면, Spring이 과연 무엇을 하는 녀석인지를 알 수가 없습니다. 그래서 간단한 예제를 통해서 Spring을 통해서 점점 진화가 되어가는 코드의 변화를 보면서 Spring을 익혀보도록 하겠습니다.

