---
title:  "Creating Application Using Spring Boot"
date:   2021-03-16
categories: Java
tags: [Java, Spring]
---

*Spring Boot를 이용하여 간단한 웹 어플리케이션을 만들기 위한 포스팅입니다. 계속해서 제작할 예정입니다.*

## Spring Boot의 소개

Spring boot의 매뉴얼을 따라 읽으며 참고할만한 점을 적어봤습니다.

- 독립적이고 (Stand-alone) 제품 수준의 (production-grade), Spring 기반의 애플리케이션을 쉽게 만들 수 있도록 도와줍니다.
- Spring Boot 애플리케이션은 Spring 설정을 거의 필요로 하지 않습니다 (주: 거의는 아니고, 웹, Security, 혹은 기본 설정들이 사용하는 property들을 설정해줘야 한다고 함. 설정의 필요성을 최소화시켜준다는 뜻인 듯)
- java -jar로 쉽게 Java 애플리케이션 제작을 시작할 수 있습니다.
- jar로 배포할 수도 있고, war 포맷으로 배포할 수도 있습니다.
- Command Line Tool로 몇 가지 명령어를 제공합니다 (주: 자주 쓰이지는 않습니다)
- Goals
  1. Spring 개발에 있어서 굉장히 빠르고 간단히 시작할 수 있도록 도와줍니다.
  2. 추가적인 설정 없이 사용하는 것을 지향하지만, 기본 설정과 다른 요구사항이 있으면 얼마든지 변경이 가능하도록 합니다.
  3. 프로젝트에서 큰 영역을 차지하는 비기능적 기능들을 제공합니다 (Embedded server, security, metrics, health checks, 외부 설정 [external configuration] 등)
  4. XML 설정을 요구하거나 더 이상의 코드 생성 (Code generation )이 필요치 않도록 함.

## System Requirements (Spring Boot 2.1.5 Release 기준)

1. Java 8 이상, Java 11 이하
2. Spring Framework 5.1.7 release 이상
3. 빌드 툴 (Maven 3.3 이상, Gradle 4.4 이상)

## Java Version 확인

```
$ java --Version
```

## Java 개발자를 위한 설치 방법

Spring Boot는 그냥 Java 라이브러리이기 때문에 `spring-boot-*.jar` 형식으로 classpath에 include하면 됩니다. 그래서 아무런 IDE나 Text Editor를 사용할 수 있음. (보통 IntelliJ 추천)
Maven이나 Gradle을 이용하여 의존성을 관리하는 것을 추천합니다.

## Maven 설치

```
$ sudo apt-get install maven // for linux
$ brew install maven // for MacOS
```

- Maven은 pom.xml이 주로 필요한데, 상속 관계를 만들 수 있는데, 부모 pom.xml 파일을 명시해두면 부모 pom.xml의 내용을 자식 pom.xml이 상속받는다고 생각하면 됩니다 (Dependency나 플러그인 정보 등)
- 의존성 관리: 부모 pom.xml에 자식들이 필요한 기본적인 정보를 넣으면 자식 pom.xml에 다시 쓸 필요가 없습니다. 예를 들어 Spring Framework 버전을 <parent>에 추가하면 자식에는 정의하지 않아도 됩니다.

*To be added*
