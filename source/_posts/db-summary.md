---
title: 11. DB Application Summary
date: 2018-03-22 08:14:19
tags: spring, jpa
---

# DB Application Summary

지금까지 우리는 DB에 접속을 하고, DB에 있는 내용을 이용해서 Service를 구성하는 방법에 대해서 알아봤습니다.
기술들은 다들 자신만의 색깔을 가지고, 개발자들을 좀 더 편하게 하기 위해서 발전되어 왔습니다. 그렇기 때문에 개발자들마다 자신이 선호하는 기술들이 따로 있는 것이고요.
그렇지만, 우리가 지금까지 Model을 하는 부분에 대해서는 한가지만은 확실히 나올 수 있습니다.

"각자의 영역으로 분리"

이것은 DB를 다루는 Model 뿐 아니라, 모든 객체와 개발에서의 Layer가 지켜야지 되는 원칙이라고 할 수 있습니다.

일반적으로 우리가 사용한 Model은 다음과 같은 package로 나뉠 수 있습니다.

```cmd
com.xyzlast.bookstore.entity
com.xyzlast.bookstore.vo
com.xyzlast.bookstore.dto
com.xyzlast.bookstore.dao
com.xyzlast.bookstore.repositories
com.xyzlast.bookstore.services
```

먼저, 지금까지 설명하고 있던 내용중에서 용어의 혼란을 가지고 오던 `DAO`와 `Repository`의 차이를 좀 더 깊게 알아보도록 하겠습니다.

## DAO vs Repository

`DAO(Data Access Object)`와 `Repository`는 기술적으로 동일한 기능을 갖습니다.

> Persistence 영역에 대한 접근 수단

그런데, 이 두 객체는 사용되는 영역이 다릅니다. `DAO`의 경우, 용어 그대로 `어디서나 접근 가능한 데이터 영역`으로 된다면, `Repository`는 서비스영역에서 사용되며, `Repository`에서 반환되는 객체들은 우리가 개발하는 `DOMAIN`의 객체들을 사용하게 됩니다. 조금 말이 어렵습니다.

`DAO`는 Service 이외에도 사용이 가능하며, 이는 `Service` 이외에도 사용이 가능하며, 모든 Transaction은 기본적으로 DAO를 기반으로 하게 됩니다. 그렇지만, Repostiory는 `Service`에서만 사용이 되며 Transaction은 제공되지 않고, `Service`에서 제공되는 `Transaction`을 기반으로 하게 됩니다.

## Packages

### entity package

Hibernate와 같은 ORM framework를 사용하는 경우, object와 persistence model간의 관계를 구성한 ORM 객체가 위치하는 영역입니다. 또는 Table에 1:1 mapping을 해서 구성하는 경우도 있습니다. 그렇지만, entities를 사용한다는 것은 기본적으로 DDD를 사용하는 것과 동일합니다. 객체과 그 객체의 관계에 대한 구성을 코드에 녹여내는 방식을 주로 사용하게 됩니다.

### vo package / dto package

vo(value object), dto(data transfer object)의 경우에는 일반적으로 persistence model에서 넘어오는 값들을 단순 parsing할 때 사용됩니다. 여기에 가장 대표적인 기술로 mybatis를 소개하였고, 이는 db에서 얻어오는 값을 view에서 단순 표시하기 위한 방법으로도 자주 사용하게 됩니다.
이 두 package는 myBatis를 Domain Layer의 Framework로 사용하는 경우에는 거의 필수로 사용됩니다. 또는 개발시에 Model 영역에서 Controller/View 로 데이터를 넘길때, DTO 객체를 만들어서 넘길때 사용되기도 합니다. 이때는 vo가 View/Value Object의 의미로 사용되기도 합니다.

### dao

직접 DB에 query를 보내고, vo, dto를 얻어오는 영역입니다. DB를 사용하는 기술에 따라 다르게 구성이 되는 것이 일반적이며, CRUD에 대한 모음으로도 구성될 수 있습니다. 단순 CRUD가 이루어진다고 해도, DB에 대한 기술적 영역이 바뀌게 될 가능성은 항상 열어두고 작업을 하는 것이 좋을 것 같습니다. 그렇기 위해서는 interface로 정의된 dao를 사용하는 것이 보다더 효과적으로 구성이 가능하게 됩니다. dao로 지정하게 되는 것은 controller, domain 모두에서 dao를 사용하겠다는 의미입니다. Hibernate와 같은 ORM을 사용하는 경우에는 repository pattern으로 접근하는 것이 더 좋습니다.

### repositories

Repository는 Dao와 매우 유사한 개념입니다. 이 두 용어의 차이점은 기능이 아닌, 사용되는 코드의 위치입니다. Domain Layer에서만 사용되는 경우에는 repository, Domain뿐 아니라 모든 영역에서 사용된다면 dao로 개발하게 됩니다. 이 둘의 차이는 Model을 풀어가는 pattern의 차이입니다. 보다더 pattern 상위적인 개념이 Repository이고, Dao는 database 적 개념이 강한 Object라고 생각하면 좀 더 이해가 쉬울 것 같습니다.

### services

business logic 영역입니다. 여러개의 dao object들과 entity 또는 vo/dto object들을 얻어내고, 그 객체들간의 로직이 구성되는 영역입니다. 일반적으로 transaction의 단위 영역이 된다는 것을 명심해주세요.
위의 구성에서 결국은 model은 3개 이상의 package로 구성이 됨을 알 수 있습니다. 남의 코드를 보더라도 대부분 위와 비슷한 구성으로 package가 구성된 경우가 많으니 참고바랍니다.

## Persistence Framework Layer의 비교

다음은 지금까지 알아본 Model을 접근하는 Framework의 조합에 대해서 한번 정리해보도록 하겠습니다.

|방법|특징|장점|단점|
|---|---|---|---|
|No Framework - <br>PreparedStatement<br>이용하는 방법|1. Native Query 구성|1. native query를 이용한 학습 시간의 단축|1.오타가 발생하는 경우, query를 실행하기 전까지 에러를 확인할 수 없다.<br>2.중복 코드가 많이 발생된다.<br>3.connection의 관리를 비롯한 Transaction의 처리를 모두 수동으로 해줘야지 된다.<br>4.java 코드에 sql 코드가 들어가기 때문에 관리 및 debug가 힘들다.|
|JdbcTemplate|1.Spring JdbcTemplate 이용<br>2.DataSource 이용<br>3.Native Query|1.native query를 이용한 학습 시간의 단축<br>2.connection 관리<br>3.Transaction 관리|1.오타가 발생한 경우, query를 실행하기 전까지 에러를 확인할 수 없다.<br>2.중복 코드가 많이 발생된다.<br>3.java 코드에 sql 코드가 들어가기 때문에 관리 및 debug가 힘들다.|
|myBatis (iBatis)|1.Native Query<br>2.DataSource<br>3.ibatis-spring 이용|1.국내의 많은 사용자<br>2.sql query의 관리를 하는 것이 가능하다.|1.객체가 아닌 VO type의 이용으로 인한, 프로그램 architecture의 발전 가능성이 낮아짐<br>2.java 언어 뿐 아니라, sql 로의 확장으로 인하여 관리 코드가 늘어남|
|Hibernate|1.HQL, Criteria<br>2.DataSource<br>3.Spring Transaction|1.다양한 reference<br>2.객체를 이용한 query의 처리로 인하여, 코드양을 줄일 수 있다.<br>3.프로그램 설계 및 구성 부분에 대해서 장점을 갖는다.<br>(DDD와 같은 객체 지향적 구성 가능)<br>4.Table의 변경 또는 DB의 변경에 유연한 장점을 갖고 있다.<br>5.동적 query가 자유스럽다.|1.학습 시간의 소요<br>2.criteria, HQL 모두 오타에 취약한 구조|
|Hibernate + queryDSL|1.JQL, Criteria<br>2.DataSource<br>3.type-safe<br>4.code generate|1.다양한 reference<br>2.객체를 이용한 query의 처리로 인하여, 코드양을 줄일 수 있다.<br>3.프로그램 설계 및 구성 부분에 대해서 장점을 갖는다.<br>(DDD와 같은 객체 지향적 구성 가능)<br>4.Table의 변경 또는 DB의 변경에 유연한 장점을 갖고 있다.<br>5.오타가 발생할 수 없는 type-safe 한 query를 작성할 수 있다.<br>6.sql query와 비슷한 문법으로, 학습에 도움을 줄 수 있다.|1.학습시간의 소요<br>2.단순 CRUD에 대한 코드양 증가<br>3.설정이 까다롭다.|
|Hibernate + JPA +<br>queryDSL + Spring Data JPA|1.JQL, Criteria<br>2.DataSource<br>3.type-safe<br>4.code generate|1.JPA - java 표준<br>2.객체를 이용한 query의 처리로 인하여, 코드양을 줄일 수 있다.<br>3.프로그램 설계 및 구성 부분에 대해서 장점을 갖는다. (DDD와 같은 객체 지향적 구성 가능)<br>4.Table의 변경 또는 DB의 변경에 유연한 장점을 갖고 있다.<br>5.오타가 발생할 수 없는 type-safe 한 query를 작성할 수 있다.<br>6.sql query와 비슷한 문법으로, 학습에 도움을 줄 수 있다.<br>7.CUD, select 문의 코딩양이 현격하게 줄어든다.<br>8.Hibernate 코드 역시 사용 가능하고 유연한 방법으로 대처가 가능하다.<br>9.Repository Interface code에 의한 가독성 향상|1.학습 시간의 소요<br>2.설정이 까다롭다.|

지금 전 이정도로 정리를 해봤는데, 다른 분들은 어떻게 정리를 할 수 있을까요? 각 방법에 대한 장/단점을 파악하는 것이 중요합니다. 적어도 이곳에 있는 분들만이라도요.
그리고, 만약에 외부 Project에 참여하게 되는 경우에는, 그 Project에 맞는 개발 방법을 이용해서 개발을 하는 것이 중요합니다.

## Web Application에서의 데이터 흐름

Domain Model을 이용해서 Web Application을 구성하는 방법은 두가지가 있습니다. 원칙을 지키고자 하는 `Pure Domain Model`과 Domain Model의 자유로운 사용이 특징인 `Domain Model Everywhere`입니다.
먼저 Domain Model Everywhere에 대해서 알아보도록 하겠습니다.

### Domain Model Everywhere

![](/images/11/01.png)

Repository, Service, Controller, View가 모두 Entity Model을 가지고 동작하는 방식입니다. 이 방식은 Domain의 Entity 객체가 Repository 뿐 아니라, Controller, View에 표현되는 방법까지 가지게 되게 됩니다. 이렇게 되면 Entity Model이 매우 커지게 됩니다. 그리고 객체의 "단일책임원칙"을 위반하게 되는 단점을 가지고 있습니다. 이 방법의 장단은 다음과 같습니다.

장점 :
* 개발하기에 빠르고 편리함

단점 :
* 복잡한 View의 표현이 매우 힘듭니다.
* Domain Model에서 View Logic이 포함되기 때문에 Domain Model의 순수성이 떨어지고 객체의 단일 책임원칙을 위반하게 됩니다.
* REST 서버와 같이 외부와의 통신 API에 사용하게 되는 경우에는 Domain Model이 변경되면 API가 바뀌게 되기 때문에 큰 문제를 야기할 수 있습니다.

### Pure Domain Model

![](/images/11/02.png)

Domain Model은 서비스와 Repository에서만 사용하고, Controller와 View에서는 DTO를 새로 만들어서 사용하는 방법입니다. 이는 Domain의 순수성을 지키게 되는 큰 장점을 가지고 있습니다. View에서만 DTO를 사용하는 것이 BL의 변화에 따른 View의 변경을 격리할 수 있는 좋은 방법이 됩니다.

장점 :
* 순수한 Domain Model - 객체지향적인 OOP 모델

단점
* DTO를 따로 만드는 것이 귀찮고, 성가시고, 괴롭다.
* DTO는 또 다른 중복 코드를 만들어낸다.
* DTO와 Domain Model간의 Convert를 따로 만들어줘야지 된다.

Domain Model Everywhere pattern의 경우에는 anti-pattern이라고도 말하는 사람이 있을정도로 호불호가 매우 강하게 갈리는 pattern중 하나입니다. 개인적으로도 최대한 Pure Domain Model을 이용해서 개발하는 것이 좋다고 생각합니다.

............................................................................ 현실은 ? OTL

지금까지 개발을 해본 결과. 다음과 같은 상황에서는 반드시 DTO를 사용해야지 됩니다.

* REST 서버와 같은 API의 response.
> Domain Model은 자주 바뀔 수 있습니다. 그렇지만, client와 통신을 하게 되는 API의 response는 바뀌면 문제가 생기게 됩니다. 따라서 이 둘은 반드시 분리를 해야지 됩니다.

* 여러 Domain Model이 결합되어서 만들어지는 새로운 View에 대한 DTO
> View가 너무너무나 복잡해서 서로간에 연관성이 없는 Domain Model을 response로 보내줘야지 되는 경우에는 DTO를 만드는 것이 좋습니다.

이 두가지 경우를 제외하고..... DTO를 모두 만드는 것은 다음과 같은 문제가 발생합니다.

* 객체가 너무 많이 만들어집니다. 지금 DataWindow의 package와 같이 각 SP의 숫자만큼의 package같이 객체들이 구성되게 됩니다. 이는 객체의 naming rule 및 관리가 매우 힘들어지는 결과를 가지고 옵니다.
* 많이 만들어진 객체 숫자 만큼의 Converter가 필요합니다. Domain Model을 DTO로 바꿔주고 DTO를 Domain Model로 바꿔주는 Convert 숫자가 필요하게 됩니다.
* 위 이유로 관리 포인트가 3개로 늘어나게 됩니다. - Domain Model, DTO, Converter

**배보다 배꼽이 더 큰 사태가 발생할 수 있습니다. Domain Model Everywhere가 절대로 좋은 것은 아닙니다. 그렇지만 개발 시간과 관리 포인트를 생각해서 Domain Model Everywhere로 만들고, 그 객체들을 차츰차츰 DTO로 변환시켜가는 과정이 Application을 더욱더 깔끔하게 만드는 과정이 되게 된다고 생각합니다.**


결론입니다.

* 처음에는 Domain Model Everywhere로 Project를 시작합니다.
* Domain Model로 처리하기 힘든 경우에는 DTO를 사용해서 객체의 변환을 합니다.
* 최대한 Pure Domain Model을 유지하기 위해서 계속 노력합니다.

> Domain Model Everywhere를 지원하기 위해서 Spring은 JPA Model Object를 위한 OpenEntityManagerInViewFilter와 Hibernate를  위한 OpenSessionInViewFilter를 지원하고 있습니다.
