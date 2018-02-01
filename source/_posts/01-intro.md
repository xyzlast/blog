---
title: 01.intro
date: 2018-02-01 12:40:57
tags: study
---

# 기술의 발전 & Application의 구조 변화

IT 기술은 다양한 분야를 가지고 있습니다. 기본적으로 정보에 대한 CRUD(create/read/update/delete) 로 구성되는 것이 일반적입니다.
단순 DB Row정보에 대한 CRUD로 이해하지 마시고, 정보의 **생성/변경/삭제/읽기**로 생각하시는 것이 좋습니다.

## 초기 Stand alone

웹 기술이 아닙니다. VB나 사내의 CS Application 형태로 구성되어 있습니다.
예전 윈도우즈에서 보여지고 있는 ODBC 연결이 그 흔적중 하나입니다.

![](/images/01/odbcmodel.gif)

## 2 Tier - 초기 WWW (~ 2006년)

Web 기술이 폭발적으로 발전하던 시기입니다. RDBMS의 발전으로 인하여 DB에 따른 Dynamic Web이 발전되게 됩니다. 이 Dynamic Web 기술에 대한 다양한 변형 기술들을 가지고 오게 됩니다.

![](/images/01/image7.png)

### asp의 폭발적인 발전

기존 개발자들의 70%정도를 차지하고 있던 visual basic 개발자들을 모두 web 개발자로 만들어주는 asp와 iis가 처음 선을 보이게 됩니다. 기존 visual basic 개발자들은 모두 db와 db를 표현하는 방식에 대해서 경험이 풍부한 사람들이였습니다. 그 개발자들을 모두 web 개발자로 만드는 엄청난 일을 MS가 성공하게 됩니다. 지금보면 참 문제가 많은 개발방법으로 개발하게 되지만, 그 당시에는 최고의 기술이였으며 누구나 쉽게 웹 프로그래밍에 접근할 수 있다는 점이 가장 큰 장점이였습니다.

### php의 발전 및 apache httpd의 확장

기존 standalone 서버에서 주로 사용되던 apache httpd를 이용한 dynamic web programming이 대두되게 됩니다. php는 asp의 장점을 흡수하고, 보다 쉬운 표현 방법을 제시합니다. 그리고 LAMP 라는 환경을 제시하기 때문에, 기업은 새로운 x86 windows server를 구매하는 것이 아닌 공짜로 web을 서비스할 수 있다는 점에서 기존 asp 시장을 잠식하기 시작합니다. 또한 linux 서버환경의 폭발적인 증가로 인하여 서버 환경이 기존 UNIX에서 linux 계열로 발전하게 된 계기가 됩니다.

### servlet과 asp .net의 발전

asp와 php로 인하여 dynamic web application이 엄청난 발전을 하게 됩니다. 하지만 asp와 php의 경우 언어의 특성상 큰 약점을 가지고 있습니다. page단위로 동작하게 되기 때문에, 현대적 프로그래밍 기법인 OOP를 사용할 수 없습니다. (방법은 있으나, 매우 괴악한 방법입니다.;) 점점 application이 거대화되어가면서 코드의 재 사용성 및 유지보수의 약점을 극복하기 위해서 asp와 php에 가려서 잠시 존재감을 잊어가고 있던 java와 .net이 다시 등장하게 됩니다.
두 언어는 RDBMS를 표현하는데 있어서 최선(?)의 방식을 가지고 있었으며, vm 기술의 발전으로 memory 및 performance에 강점을 가지고 있었기 때문에, 기존의 web 개발 시장을 빠르게 잠식해나가기 시작합니다.

### fat client의 발전

servlet과 asp .net의 발전은 RDBMS와 dynamic web application간의 여러 문제를 모두 해결해준것같지만 사용자들에게 다양한 경험을 보여주기 위한 View를 제공하는데에 있어서, 기존의 HTML만은 한계를 가지고 있었다. 동영상과 같은 멀티미디어를 포함하기에는 기존의 HTML이 지원하지 않는 부분이 너무나 많은 영역들이 있었기 때문입니다. 멀티미디어에 대한 지원을 강화하기 위해서 나온 첫 기술이 micromedia의 Flash입니다. Flash는 멀티미디어의 지원뿐 아니라, 웹을 보다 더 아름답게 만드는데 큰 공헌을 하게 됩니다. Flash의 대성공으로 MS는 asp로 마련했던 서버 시장에 심각한 타격을 입게 됩니다. 그래서 자사의 IIS + .NET Framework 기술로 동작하는 silverlight를 출시하기에 이릅니다. 이러한 멀티 미디어의 지원과 더불어, RDBMS에 특화된 fat client들이 나오게 되는데, PowerBuilder가 대표적입니다. 우리들에게는 `X-Client`, `X-Window`와 같은 이름들로 알려진것들이 많습니다.

fat client의 발전은 기존 servlet과 asp .net 기술과 충돌을 일으키게 됩니다. 단순히 servlet과 asp .net을 fat client의 container로 이용하게 되고 모든 로직을 fat client에서 처리하게 되는 개발 방법이 한 때 유행하게 됩니다.

## 3 tier, N tier, Cloud 시기 (2006년 ~ )

2 tier 가 발전한 형태입니다. 다양한 요구사항으로 인하여 기존 2 tier system의 확장을 통한 여러 시스템간의 결합으로 최종적으로는 enterprise system으로의 진화, 발전되었습니다. 기존 2 tier system에서는 해결할 수 없는 기술적 이슈들이 발생하게 됩니다. 다양한 시스템들이 나오게 되고 그 시스템들간의 상호 통신에 의하여 기존 2 tier 에서 n tier로 계속되는 발전이 이루어지게 됩니다.

![](/images/01/3tier.svg)

### script language의 발전

기존 .net과 java와는 다른 언어들이 발전하게 됩니다. 기존의 asp와 php가 OOP적 성격을 갖지 못한 단점을 해결하고 OOP적 장점과 개발의 편의성을 극대화한 script language가 발전하게 됩니다. python, ruby가 대두되기 시작하지요. 이 언어들은 기존의 객체 지향적인 특징과 asp, php와 같은 script 적 성격을 모두 갖게 됩니다. 개발의 속도, 변화 가능성에 대한 열린 대응을 토대로 이와 같은 script language가 계속된 발전을 하게 됩니다. 해외의 많은 시스템들이 하부 tier의 경우에는 ruby, python으로 개발된 것을 지금도 자주 볼 수 있습니다.

최근에는 hot하다고 할 수 있는 node.js가 script 언어중 가장 인기가 좋습니다.

### web service의 개발

n tier system의 발전은 web service가 없었다면 불가능하다고 할 정도로 web service는 n tier system에 깊이 관여되어 있습니다. web service는 기존의 tier를 종적으로나 횡적으로 모두 확장을 시키는 가장 결정적인 역활을 하게 되는 계기가 되었습니다. "Service로서의 Web"에 대한 개념은 수많은 파생적 개념을 만들어 내고, 기술적 발전을 가지고 왔습니다. 기존에는 SOAP을 기반으로 한 web service가 주로 사용되었으나, 지금은 client(javascript) 등에서의 호출 문제로 인하여 xmlrpc의 경량화된 버젼인 REST를 주로 사용하고 있습니다. 이 REST에 대해서는 다시 한번 설명할 기회를 갖도록 하겠습니다.

### MVC의 발전

기존의 웹의 개발은 단순히 DB의 결과값을 웹에 표현하는 방식이였습니다. 대부분의 Business Logic은 DB에서 가지고 있고, 그 Business Logic을 web에 표현하는 방식이 대부분이였지요. 이렇게 된 가장 큰 이유중 하나는 웹으로 개발을 한 내용은 웹에 너무나 밀착되어있는 프로그램이라서, Business Logic만의 테스트가 거의 불가능하다는 것에서 시작되었습니다. 그렇지만, n tier system으로 발전해나가면서 tier만의 중복된 Business Logic을 정리 및 관리하는데 있어서 기존 프로그래밍 언어로 하고자 하는 욕구가 계속해서 발전되어 갑니다. DB Query 중심의 개발은 필연적인 중복코드와 로직의 누수가 발생하기 마련이니까요.

이때, ruby를 기반으로 한 ruby on rails가 발표됩니다. rails framework라고도 불리우는 이 framework는 기존의 web 개발 방법을 완전히 바꿔놓게 됩니다. 사용자에게 보여지는 영역인 View, Http response/request를 처리하는 Controller 마지막으로 Domain의 Business Logic을 처리하는 Model로 완벽한 영역을 분리할 수 있음을 ruby on rails는 보여주게 됩니다. 이러한 장점은 각 영역을 개발자들이 테스트를 해볼 수 있고, 영역에 대한 전문화를 분리시킬 수 있기 때문에 단숨에 웹 개발의 주도적 방법으로 대두되었습니다. java에서는 struct2가, .net에서는 asp .net mvc가 기존 ruby on rails의 사상을 반영한 MVC web framework입니다.

개발자는 지금 MVC를 중요하게 봐야지 됩니다. MVC 개발 방법은 웹뿐 아니라, 모든 application에 적용되어 있는 상황이고 각 영역에 대한 테스트를 통해서 자신의 코드의 완벽성을 스스로 검증할 수 있는 기회를 가지게 되었습니다. 에러에 대한 명확한 정의가 가능하게 되었으며, 에러가 발생했을때의 장애 처리와 같은 새로운 프로세스 정립 역시 MVC의 확립으로 가능하게 되었습니다. 이 부분은 지금까지는 개발자가 아닌, 기획이나 의사결정자들의 손에 있던 부분이였지만, 지금은 개발자들이 제시하는 방법을 선택하는 방향으로 전환이 된 상태입니다.

### mobile 기기의 발전 및 확산

기존 n tier system의 발전은 획적 확장에 해당된다면 mobile 기기의 발전 및 확산은 종적 확장에 해당됩니다.기존까지 있던 web application의 대상은 모두 PC의 browser를 대상으로 하고 있었습니다. PC를 기반으로 하고 있기 때문에 PC의 특정 browser만을 target으로 하고 개발이 가능해졌지요. 그렇지만, iPhone을 시작으로 한 mobile 기기의 발전은 이러한 생각을 모두 바꾸어놓게됩니다. 누구나 가지고 다니는 mobile 기기는 언제 어디에서나 접근이 가능한 특징을 가지고 있습니다. 이는 기존보다 많은 접속을 만들어 내고, 모든 device에서 동일하게 보여야지 된다는 문제를 가지고 오게 됩니다. 그리고, mobile device의 빈약한 시스템 자원과 기존 windows system과 다른 OS 환경으로 인하여 HTML의 표준화에 대한 요구 사항이 높아지게 됩니다.

### fat client의 쇠퇴와 HTML5의 대두

기존 flash와 silverlight와 같은 fat client가 mobile device에서 정상적으로 동작하지 않는 문제가 발생하게 됩니다. 이는 기존 PC에서도 windows-IE 환경 이외에서도 계속 지적되던 문제였지만, 본격적인 문제로서 국내에서는 대두되기 시작한 것은 바로 mobile 기기의 확산때문입니다. 기존 HTML에서 지원되지 않던 multimedia에 대한 지원을 비롯하여 animation과 websocket등의 표준화로 인하여 기존 flash와 silverlight가 설 자리가 없어지기 시작합니다. 지금 fat client는 flash만을 제외하고 거의 사장되어가는 분위기입니다. Flash의 경우에도 HTML5의 확산 전까지 잠시의 대체제로서의 의미 이외에는 퇴색해가는 것이 현실입니다. 기존 Flex 개발자들이 설 자리가 많이 없어지고 있지요.

### Big data와 Cloud 의 대두

끊임없이 이야기가 나오고 있는 Big data와 Cloud는 mobile 기기의 발전 및 확산과 기존 web system의 오랜 발전으로 인하여 나온 기술이라고 할 수 있습니다. 기존의 RDBMS에서는 처리를 할 수 없을 정도의 데이터가 이제는 수집이 된 상태이고, 이 데이터들을 어떻게 분석을 해야지 될지. 이 데이터들을 어떻게 활용을 해야지 될지. 그리고 많은 mobile 기기에서 동시 다발적으로 들어오는 데이터를 어떻게 해야지 될지에 대한 물음에서 Big Data와 Cloud를 활용하는 방법을 찾아보는것이 방법일것 같습니다. 이러한 Big Data를 저장하기 위한 방법으로 cassandra, mongoDB, 등이 있으며 Big Data를 처리하기 위한 방법으로 Hadoop이 대두되게 됩니다. 또한 Cloud 시장은 아직 춘추전국의 시대와 같이 복잡한 상황이며, 시장 1위인 amazon의 AWS, MS의 Azure, Google의 Google Cloud Platform등이 경쟁을 하고 있습니다.

## 개발자가 익혀야지 될 기술

### language - Java, Javascript

웹쪽 개발에 있어서 backend는 java, frontend는 javascript로 구성되는 것은 아주 일반적입니다. 근간에는 backend쪽도 node.js를 이용해서 frontend와 backend의 언어쪽 기술을 동일하게 들고가는 경우도 많습니다. 요즘은 Javascript의 시대라는 말이 나올정도로 javascript의 인기가 좋은 편입니다. 개발을 이제 시작하는 분들이 있다면 javascript도 꽤나 추천하고 싶은 언어입니다. 웹을 하기 위해서는 어차피 javascript를 해야지만 하니까요.

### Backend Framework - Spring(Java), Express(node.js), Ko(node.js), python(django)

언어가 정해지면, 그 언어를 이용해서 처음부터 열심히 만드는 것이 아닙니다. 많은 천재들이 만들어둔 Framework를 이용하는 것이 일반적입니다. 웹 개발에 있어서 Java측에서는 Spring Framework, node.js에서는 Express와 Ko가 주로 사용됩니다. Framework의 사용법을 익힌다는 것은 Framework의 사상을 이해하는 것이 필요합니다.

### Frontend Framework - ReactJs, AngularJs, Angular2, Vue.js

Frontend분야의 발전은 놀라울 정도입니다. 지금 인기가 좋은 Frontend Framework의 경우에는 다음 것들 ReactJs, AngularJs, Angular2, Vue.js가 있습니다. 이 중 하나이상은 익혀두는 것이 좋습니다. 개인적으로는 Vue.js/React.js를 추천합니다.

### Data Storage - Database, Big-Data, No-SQL, Memory-Db

Database는 고전적인 이야기이긴 하지만, 여전히 중요한 분야입니다. 기존의 RDBMS뿐 아니라 다른 DB역시 필요합니다. memory db의 일종인 Redis는 이제 서비스를 운영하는 곳이라면 사용하지 않는 곳이 없을정도로 대중적인 인프라중 하나입니다. 또한 NO-SQL 진영에서의 MongoDb나 Big Data 부분에서의 Hadoop 역시 중요합니다. 어떤 분야에 대해서 공부를 할 것인지에 대해서는 많은 세미나를 가보는 것도 좋고, 다른 분야에 대한 관심을 계속해서 갖는 것이 필요합니다.

### HTTP, HTTP2

HTTP는 계속해서 발전되어가면서 지금은 HTTP2의 시대가 오고 있습니다. 웹의 기초가 되는 HTTP가 어떻게 동작하고 있는지에 대한 공부는 계속해서 필요합니다. 그리고 HTTP2가 이제 상용화가 되고 있는 시점이기 때문에, 이에 대한 개발 방법의 변화등도 따라가야지 됩니다. 언제까지 옛날 기술에 계속해서 매몰되어갈 필요는 없습니다.

### Modeling

Modeling은 소위 말하는 '업무분석' 과정입니다. 업무를 분석하고 분석한 업무를 programming language로 구현 가능한 형태로 추상화를 시키는 과정을 의미합니다. 개발자의 생각의 방향은 학교에서 자주 들은 Divide & conquer 입니다. 작게 나누고, 하나하나 해결해나가고. 그 해결한 조그마한 것들을 다시 붙이는 작업들이 필요합니다.

Machine Learning에 있어서도 역시 Modeling은 가장 중요한 일중 하나가 되어 있습니다. **실 상황을 우리가 생각하는 공학적인 분야로 이동시키는 것** 이것이 Modeling의 본질이 됩니다.

### 개발 방법론

개발 방법론은 Agile을 기반으로 한 scrum, kanban 등이 인기를 끌고 있습니다. 개발만을 중심으로 생각한다면 TDD(Test Driven Development)가 있습니다. TOP-DOWN 방식으로 진행이 된다면 그에 따르면 좋겠지만, 그것이 안된다면 자신이 자기만의 개발 패턴을 만들기 위해서라도 간단한 이론등을 아는 것이 좋습니다.

### Design pattern

개발 방법론과 매우 유사한 분야입니다. 개발을 할때, 코드를 최적화 하는 패턴들이 존재합니다. 패턴을 익히는 것보다는 패턴을 이용해서 개발자들끼리의 의사소통이 가능해져야지 됩니다. 개발자들끼리 사용하는 약어가 되는 경우가 꽤나 많습니다. 그리고 책을 통해서 학습을 할때도 유용합니다.

### 단위 테스트 (Unit Test)

개발자는 자신이 만든 모든 s/w를 테스트할 수 있어야지 되고, 그 테스트한 결과로서 자신의 s/w의 품질을 증명할 수 있어야지 됩니다. "내가 하면 되었는데.", "어제는 되었는데." 식의 이야기는 곤란합니다. in/out이 결정나면 그 in/out에 대한 명확한 테스트를 하고 그 테스트 결과를 보여줄 수 있어야지 됩니다. 또한 s/w의 규모가 크면 클수록 그 코드를 검증할 수 있어야지 됩니다. 이제는 테스트를 돌리기 위해서 개발을 한다. 라는 말이 나올 정도로 테스트는 일반화된 기술입니다. 자동화된 테스트를 구성하는 능력은 반드시 필요합니다.

### Infrastructure (Cloud, Docker env)

이제는 환경을 제공하는 Cloud를 기반으로 하여 개발 및 서비스가 진행되어가는 시대입니다. Cloud, Docker를 기반으로 하는 환경에서는 역시 이 환경에 대한 이해가 반드시 필요합니다.

## Summary

웹어플리케이션은 지난 20여년동안 눈부시게 발전한 분야입니다. 여러 패러다임들의 변화가 끊임없이 있었습니다. 지금 한국에서 가장 주가 되어 있는 `Java`를 기준으로 Application 개발 방법에 대해서 한번 같이 알아보도록 하겠습니다.

마지막으로... 제가 Java를 해야지 된다고 이야기를 했지만, 현 2018년의 server-side programing language의 현 사용률은 다음과 같습니다. 국내는 자료 못찾았습니다.

![](/images/01/flow.png)

