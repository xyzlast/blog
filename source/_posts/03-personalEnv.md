---
title: 03.개인 개발 환경 구축
date: 2018-02-01 12:44:11
tags: study
---

# 개인개발환경 구성

이 장에서는 개인적으로 추천하는 개발 툴들을 소개하고자 합니다.
java 개발환경은 모두가 설정할 줄 안다고 가정을 하고, IDE는 IntelliJ community version을 기반으로 설명을 하도록 하겠습니다.

## gradle

gradle은 안드로이드 개발을 할때 많이 보신분들이 있겠지만, build tool입니다.
java, c#, c++ 등 모든 언어를 compile 할 수 있는 build tool을 목표로 하고 있습니다. java에서는 maven이 가장 많이 사용되고 있는 build tool이지만,
개인적으로는 gradle이 더 좋은 tool이라고 확신하는 편입니다.

Spring Source 역시 gradle을 기본 build tool로 사용하고 있으며, multi-project build에 강점을 갖습니다. 다양한 plugin 역시 장점입니다.

### build란?

*build라는 것이 무엇인가*에 대한 정의에 따라서 build tool이 해야지 되는 일들은 달라집니다.
기본적인 build의 정의는 다음과 같이 말할 수 있습니다.

*소스 코드를 동작하는 독립적인 소프트웨어 산출물로 만드는 과정*

매우 단순하게 정의가 가능한 이 과정은 막상 해보게 되면 다음과 같은 문제를 같이 생각해봐야지 됩니다.

* 의존관계를 갖는 외부 library와의 버젼 관리 문제
* 개발자 1명이 아닌, 개발팀이 동시에 환경을 구축이 가능한 수 있는지

그리고, 작은 의미의 build가 아닌 좀 더 확대해보면 다음과 같이 말할 수 있습니다.

*소스 코드를 동작하는 독립적인 소프트웨어 산출물로 만들어 배포하는 과정*

이렇게 생각해보면 build는 compile, test, publish를 모두 포함하는 범위입니다. 이러한 build를 도와주는 tool을 build tool이라고 합니다.
java에서는 maven, gradle이 사실상의 정의(de-factor)로 사용되고 있습니다.

java에서의 주요 build tool은 다음것들이 있습니다.

#### Apache Ant

* java로 구현된 project의 build에 적합함
* file의 copy, command의 실행에 중점
* java project의 경우, classpath와 javac 명령어의    조합으로 compile 하는    것을 목적
* build.xml 또는 ant.properties 파일을 통해서 build를 정의

#### Apache Maven

* ant와 비슷
* goal, phase를 통해 project의 life cycle을 정형화 하고 관리
* pom.xml 을 통해 build를 정의
* dependency library들의 버젼별 관리 가능
* documentation 및 relase note에 대한 정의

#### Gradle

* ant + maven + groovy
* plug in의 조합으로 project의 life cycle을 정형화하고 관리
* maven의 goal로 지정되어 있는 정적인 결합이 아닌, groovy언어로 programing이 가능한 build의 정의가 가능

## git flow

*git*을 이용한 개발시에 있어서 개발 process를 강제하는 command tool입니다.

```cmd
git flow [command]
```

를 통해서 실행을 하며 branch -> commit -> develop merge -> master merge를 강제하게 됩니다. git을 *잘* 사용하게 돕는 tool로 생각하면 좋습니다.
다음은 git flow의 예시입니다.

### 신규 Issue의 진행

* redmine에서 신규 개발 process issue 80번을 확인
* `git flow feature start is80`을 통해 신규 branch 에서 작업 진행
* `git flow feature finish is80`을 통해 branch 작업 완료
* `git checkout master`를 통해 신규 master branch 이동
* `git pull`을 통해 최신 master update
* `git merge develop`를 통해 개발한 내용 merge
* `git push`를 통한 최신 소스 push
* `git checkout develop`를 통해 다시 개발 branch로 이동

### 신규 Release 진행

* master, develop간의 sync 확인
* `git flow release start [[version-name]]`을 통해 신규 release 시작
* `git flow release finish [[version-name]]`을 통해 release tag 를 붙여서 release를 작성
* `git push origin [[version-name]]`을 통해 tag된 release를 remote origin으로 push

자세한 설명은 https://danielkummer.github.io/git-flow-cheatsheet/index.ko_KR.html 를 통해 볼 수 있습니다. 많은 분들이 사용하셨으면 하는 tool입니다.

## git kraken

git의 개발 흐름을 한눈에 볼 수 있도록 도와주는 tool입니다. commit 등에도 유용하게 사용이 가능하지만, 개인적으론 git은 terminal 환경에서 사용하는 것이 더 좋고, 개발흐름을 보는데 많은 도움을 줄 수 있습니다.

![](/images/03/gitkraken.png)

## PMD, codestyle, FindBug

PMD, codestyle, FindBug은 자바를 위한 정적 분석 도구입니다. 개발자들은 표준 코드에 부합하고 좋은 코드를 제공하기 위해 사용하는 것이 좋습니다. 팀 리더와 품질 보증 직원들은 코드 검토(Code Review)의 특성을 바꾸기 위해 사용하는데, 이 도구들은 코드 검토로부터 발견된 기계적이고 문법 적인 점검을 취약점으로 변경하게 됩니다.

자신들의 코드를 끊임없이 평가를 받고 그에 대한 feedback을 받을 수 있다면, 개발자들의 실력은 좀 더 나아질 수 있을 것입니다.

## sonarQube

PMD, codestyle, FindBug에서 평가된 내용에 대한 통계 및 처리 방법에 대한 Report를 제공합니다. 이러한 Report를 볼 수 있습니다. (http://192.168.94.18:9000/dashboard?id=fds-v2)

![](/images/04/sonarqube.png)

## Docker

교육중에 DB는 Docker image를 통해 제공하도록 하겠습니다. 제 IP를 기준으로 image를 다운 받을 수 있도록 하겠습니다. docker에 대해서 사용을 해보시고, 어떤 식으로 사용이 가능할지를 한번 고민해보는 자리가 되었으면 좋겠습니다.

## Summary

앞으로는 실제적인 개발코드 및 과제가 나갈 예정입니다. 아래 SW의 설치를 해둔 환경을 준비해주시길 바랍니다.

* IntelliJ Community Version
* Gradle
* JAVA 8
* DOCKER
