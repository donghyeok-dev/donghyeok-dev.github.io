---
layout: customPost
title:  "Spring Boot, Sentry, Slack"
categories: 
  - Sentry
  - Slack
  - SpringBoot
---
# Sentry 시작하기

https://sentry.io/

https://docs.sentry.io/platforms/java/guides/spring-boot/?_ga=2.13694775.1073304577.1623644962-772191642.1623118635

Setry는 애플리케이션에서 발생하는 로그를 수집하고 특정 조건에 따라 slack 등으로 알람을 보낼 수 있는 솔루션입니다. 또한 로그에서 오류 발생 위치나 요청 URL, 요청 brower 정보 등 다양한 정보를 수집할 수 있어 오류 분석에 도움이 됩니다.

Sentry는 클라우드 버전과 설치형(On-Premise)버전이 있습니다. 클라우드 버전은 인프라와 솔루션을 제공하기 때문에 편리하게 사용가능하지만 버전에 따라 금액이 책정될 수 있습니다. 무료 버전은 개발자 혼자 사용하고 특정 기능의 제약이 있을 수 있습니다. 설치형은 인프라 구축, 운영을 직접해야 되지만 민감한 정보가 외부로 유출되는 것을 막을 수 있습니다.  여기서는 테스트를 위해 무료 클라우드 버전을 사용 해보겠습니다.

![image-20210614161101009](C:\Users\webme\mygit\blog\assets\images\posts\image-20210614161101009.png)

https://sentry.io/signup/ 에서 회원가입을 하여 로그인 합니다.

로그인 후 프로젝트를 생성합니다. 여기서는 SpringBoot 프로젝트로 생성하겠습니다.

![image-20210614150327879](C:\Users\webme\mygit\blog\assets\images\posts\image-20210614150327879.png)

<img src="C:\Users\webme\mygit\blog\assets\images\posts\image-20210614145847366.png" alt="image-20210614145847366" style="zoom:150%;" />



Sentry의 SpringBoot 연동은 버전 2.1.0 이상 지원이 됩니다.

Gradle

```groovy
implementation 'io.sentry:sentry-spring-boot-starter:5.0.1'
implementation 'io.sentry:sentry-logback:5.0.1'
```

Maven

```xml
<dependency>
    <groupId>io.sentry</groupId>
    <artifactId>sentry-spring-boot-starter</artifactId>
    <version>5.0.1</version>
</dependency>
<dependency>
    <groupId>io.sentry</groupId>
    <artifactId>sentry-logback</artifactId>
    <version>5.0.1</version>
</dependency>
```



### yml 파일에 DSN을 추가합니다.  

dsn 값은 프로젝트 최초 생성시 가이드에 나옵니다. 또한 Settings > Project > Client Keys에서도 확인할 수 있습니다.

```yaml
sentry:
  dsn: https://e32f3a121224ce8b7cdbebcdfdfd2b1@o846429.ingest.sentry.io/12344232
  logging:
    minimum-event-level: error
```



### logback-spring.xml 

https://docs.sentry.io/platforms/java/guides/spring-boot/logging-frameworks/

기존에 log4j에 logback을 설정해둔 상태였는데 SentryAppender를 지정하지 않아도 정상 동작함...

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <!--<appender name="Sentry" class="io.sentry.logback.SentryAppender" />-->

  <logger name="jdbc" level="OFF"/>
  <logger name="jdbc.sqlonly" level="OFF"/>
  <logger name="jdbc.sqltiming" level="DEBUG"/>
  <logger name="jdbc.audit" level="OFF"/>
  <logger name="jdbc.resultset" level="OFF"/>
  <logger name="jdbc.resultsettable" level="DEBUG"/>
  <logger name="jdbc.connection" level="OFF"/>

  <root level="INFO">
    <appender-ref ref="STDOUT" />
    <!--<appender-ref ref="Sentry" />-->
  </root>
</configuration>
```



### 테스트 해보기

```java
@Controller
public class PostalController {
    @GetMapping(value="/sentryTest")
    public void sentryTest() {
        throw new FailResponseApiException("Error!! Sentry Test!!");
    }
```



```java
@ControllerAdvice
public class CommonExceptionHandler {
    private static final String ERROR_PAGE = "error/dispatcher-internal-error";

    @ExceptionHandler(FailResponseApiException.class)
    public String handleFailResponseApiException(FailResponseApiException e, Model model) {
        Sentry.captureException(e);
        model.addAttribute(CommonUtil.ERROR_MSSAGE_CODE, ExceptionType.FAIL_RESPONSE_API);
        return ERROR_PAGE;
    }

    @ExceptionHandler(NotFoundDataException.class)
    public String handleDataNotFoundException(NotFoundDataException e, Model model) {
        Sentry.captureException(e);
        model.addAttribute(CommonUtil.ERROR_MSSAGE_CODE, ExceptionType.NOT_FOUND_DATA);
        return ERROR_PAGE;
    }
//    ...
```



브라우저에서 http://localhost:8080/sentryTest를 호출합니다.

센트리 프로젝트 홈에서 Issues를 확인합니다.

![image-20210614160431600](C:\Users\webme\mygit\blog\assets\images\posts\image-20210614160431600.png)

상세 페이지에서 오류메시지, 오류 코드 위치, 요청 URL, 요청 클라이언트 정보 등을 볼 수 있습니다.

![image-20210614160555355](C:\Users\webme\mygit\blog\assets\images\posts\image-20210614160555355.png)





# Sentry 알람 Slack으로 받기

sentry에서 발생하는 오류 알람을 slack으로 받는 설정을 합니다.



## Slack 시작하기

https://slack.com/intl/ko-kr/help/categories/360000049043

협업 메시징 도구이며 무료와 유료 라이센스가 있습니다.

https://slack.com/intl/ko-kr/pricing

![image-20210609092626154](C:\Users\webme\mygit\blog\assets\images\posts\image-20210609092626154.png)



https://slack.com/intl/ko-kr/help/categories/360000049043 에서 환경에 맞게 다운로드 받고 설치합니다.

![image-20210614163313699](C:\Users\webme\mygit\blog\assets\images\posts\image-20210614163313699.png)

워크스페이스를 생성합니다.

![image-20210614163416914](C:\Users\webme\mygit\blog\assets\images\posts\image-20210614163416914.png)

![image-20210614163545155](C:\Users\webme\mygit\blog\assets\images\posts\image-20210614163545155.png)

채널을 개설하고 팀원을 초대합니다. 

채널 우측 상단에 아이콘을 클릭 > 통합 > 앱추가 > webhook을 검색하여 설치합니다.

![image-20210614165412960](C:\Users\webme\mygit\blog\assets\images\posts\image-20210614165412960.png)

![image-20210614165558448](C:\Users\webme\mygit\blog\assets\images\posts\image-20210614165558448.png)



https://hooks.slack.com/services/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



![image-20210614165642389](C:\Users\webme\mygit\blog\assets\images\posts\image-20210614165642389.png)



slack webhook 테스트

```
curl -X POST --data-urlencode "payload={\"channel\": \"#developer\", \"username\": \"webhookbot\", \"text\": \"test message\"}" https://hooks.slack.com/services/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
```



Sentry Settings > Integrations > WebHooks를 설치하고 Slack의 WebHooks URL을 붙여 넣습니다.

![image-20210614170044675](C:\Users\webme\mygit\blog\assets\images\posts\image-20210614170044675.png)





https://slack.com/intl/ko-kr/help/categories/200111606

