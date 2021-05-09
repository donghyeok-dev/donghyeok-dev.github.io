---
layout: customPost
title:  "JRE, JDK, SE, EE, OpenJDK"
categories: 
  - Java
  
---
<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210508233051567.png" alt="image-20210508233051567" style="zoom:150%;" />

### JRE

JRE는 JVM, 자바 클래스 라이브러리, 자바 명령 등을 포함하며 자바 프로그램을 실행하는데 필요한 패키지입니다. keytool, policytool 등의 유틸리티도 포함되어 있습니다. 

###  JDK

JDK는 JRE의 기능 뿐만 아니라 javac, javadoc과 같은 도구 등 java를 사용하기 위해 필요한 모든 기능을 간춘 SDK(Software Development Kit)입니다.

### Java SE(Standard Edition)

가장 대중적으로 많이 사용되는 에디션으로 네트워킹, 보안, 데이터베이스 등 Java 개발에 필요한 대부분의 기능이 포함되어 있습니다. SE의 API에는 java.lang.*, java.io.*, java.util.* 등의 specification을 포함하고 있고, JDK는 SE의 specification을 구현한 구현체이기 때문에 SE버전과 JDK버전은 종속적입니다.

- API
- RMI
- JDBC
- Swing : AWT를 확장한 GUI도구로 OS에 종속적이지 않으며 많은 컴포넌트를 제공.
- Collections
- Xml binding
- JavaFX(java8)
- Collections Streaming API(java8)
- Reactive Streams API(java9)
- HTTP/2 API(java9)

### Java EE(Enterprise Edition)

SE 플랫폼에 주로 서버에서 동작하는 기능들을 추가한 서버 개발 위한 플랫폼입니다.

- Servlet
- Websocket
- JNDI
- EJB(Enterprise JavaBeans): 기업환경의 시스템을 구현하기 위한 서버 애플리케이션.
- Persistence
- Transaction
- JMS
- Batch API

### Java ME(Micro Edition)

휴대전화, PDA, 세톱박스 등과 같은 임베디드 개발환경을 위한 플랫폼입니다.



### LTS(Long Time Support) 버전

오래동안 지원할 버전을 뜻함.



### OpenJDK

2018년 오라클 라이선스 체계가 변경되면서 2019년 1월 이후 무료라이센스를 사용할 수 없게 되었습니다. 따라서 무료로 사용할 수 있는 OpenJDK 사용을 검토해야만 했습니다. OpenJDK는 대표적으로 Oracle의 OpenJDK, AZUL의 Zulu, RedHat의 OpenJDK, AdoptOpenJDK가 있습니다. 이 중에서 AdoptOpenJDK는 TCK(Technology Compatibility Kit) 인증을 통과하진 못했지만 IBM에서 지원하고 있고, Spring 공식 사이트에서도 추천하고 있으며, 네이버 라인에서 OpenJDK 관련한 테스트에서도 평이 괜찮다고하여 선택하였습니다.

https://spring.io/quickstart

https://engineering.linecorp.com/ko/blog/line-open-jdk/