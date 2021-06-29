---
layout: customPost
title:  "Spring Boot, Sentry, Slack, Logback"
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

![image-20210614161101009](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210614161101009.png)

https://sentry.io/signup/ 에서 회원가입을 하여 로그인 합니다.

계정관리에서 Timezone을 서울로 변경합니다.

![image-20210623154022691](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210623154022691.png)



프로젝트를 생성합니다. 여기서는 SpringBoot 프로젝트로 생성합니다.

![image-20210614150327879](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210614150327879.png)

<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210614145847366.png" alt="image-20210614145847366" style="zoom:150%;" />



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
  dsn: https://d625fcfcf8e5492196f237d6267c34f9@o846771.ingest.sentry.io/....
  logging:
    minimum-event-level: error
  send-default-pii: true
```

minimum-event-level : sentry issue로 발생시킬 최소 log level

send-default-pii: sentry issue 상세페이지에서 사용자 정보 표시여부(현재 로그인 id, ip정보)



### logback-spring.xml 설정

https://docs.sentry.io/platforms/java/guides/spring-boot/logging-frameworks/

기존에 log4j에 logback을 설정해둔 상태였는데 SentryAppender를 지정하지 않아도 정상 동작함...

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <logger name="jdbc" level="OFF"/>
  <logger name="jdbc.sqlonly" level="OFF"/>
  <logger name="jdbc.sqltiming" level="DEBUG"/>
  <logger name="jdbc.audit" level="OFF"/>
  <logger name="jdbc.resultset" level="OFF"/>
  <logger name="jdbc.resultsettable" level="DEBUG"/>
  <logger name="jdbc.connection" level="OFF"/>

  <root level="INFO">
    <appender-ref ref="STDOUT" />
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
        log.error(e.getMessage(), e);
        model.addAttribute(CommonUtil.ERROR_MSSAGE_CODE, ExceptionType.FAIL_RESPONSE_API);
        return ERROR_PAGE;
    }

    @ExceptionHandler(NotFoundDataException.class)
    public String handleDataNotFoundException(NotFoundDataException e, Model model) {
        log.error(e.getMessage(), e);
        model.addAttribute(CommonUtil.ERROR_MSSAGE_CODE, ExceptionType.NOT_FOUND_DATA);
        return ERROR_PAGE;
    }
//    ...
```



브라우저에서 http://localhost:8080/sentryTest를 호출합니다.

센트리 프로젝트 홈에서 Issues를 확인합니다.

![image-20210614160431600](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210614160431600.png)

상세 페이지에서 오류메시지, 오류 코드 위치, 요청 URL, 요청 클라이언트 정보 등을 볼 수 있습니다.

![image-20210614160555355](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210614160555355.png)





## Slack 시작하기

https://slack.com/intl/ko-kr/help/categories/360000049043

협업 메시징 도구이며 무료와 유료 라이센스가 있습니다.

https://slack.com/intl/ko-kr/pricing

![image-20210609092626154](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210609092626154.png)



https://slack.com/intl/ko-kr/help/categories/360000049043 에서 환경에 맞게 다운로드 받고 설치합니다.

![image-20210614163313699](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210614163313699.png)

워크스페이스를 생성합니다.

![image-20210614163416914](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210614163416914.png)

![image-20210614163545155](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210614163545155.png)

Slack 좌측 메뉴바에서 채널추가를 눌러 Sentry에서 알람을 받을 채널 하나를 만듭니다.

![image-20210622170232313](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210622170232313.png)





# Sentry 알람 Slack으로 받기

settings - Integrations - Slack을 추가합니다. 설치를 위해서는 Team 라이센스가 필요하기 때문에 15일 try버전으로 테스트 해보겠습니다.

![image-20210622165720825](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210622165720825.png)

프로젝트 상세 페이지에서 Create Alert를 클릭합니다.

![image-20210622165851267](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210622165851267.png)

Set Conditions를 클릭합니다.

![image-20210622165921520](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210622165921520.png)

기본정보 Alert 조건을 설정하고 알람을 받을 Slack정보를 기입합니다.

![image-20210622170044224](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210622170044224.png)

![image-20210622170959144](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210622170959144.png)

위에서 작성한 오류 테스트용 /sentryTest를 호출하여 slack에 추가한 채널을 확인합니다.

(주의: 위에서 Set action interval 이내 동일한 오류는 alerting되지 않습니다.)

![image-20210622170542594](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210622170542594.png)



여기까지 Spring에서 발생한 오류를 Sentry로 전송하고 Alert 기능으로 Slack에게 메시지를 전송하는 것 까지 해보았습니다. 하지만 Sentry에서 Slack 전송은 Team 라이센스가 필요하기 때문에 무료로 하는 방법이 있나 찾아보다가 Spring에서 WebHook을 통해 Slack으로 전송하는 방법을 찾았습니다.



### Spring boot에서 Slack으로 error 메시지 보내기

먼저 Slack에서 채널 우측 상단에 아이콘을 클릭 > 통합 > 앱추가 > webhook을 검색하여 설치합니다.

![image-20210622171741755](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210622171741755.png)

![image-20210622171807390](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210622171807390.png)

![image-20210614165558448](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210614165558448.png)



https://hooks.slack.com/services/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



![image-20210614165642389](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210614165642389.png)



커맨드창에서 slack webhook 테스트

```
curl -X POST --data-urlencode "payload={\"channel\": \"#error\", \"username\": \"webhookbot\", \"text\": \"test message\"}" https://hooks.slack.com/services/T025A3C7J73/....
```



### Spring  Slack 설정

오픈소스인 [logback-slack-appender](https://github.com/maricn/logback-slack-appender) 를 이용하여 설정합니다.

gradle에 의존성 추가

```groovy
implementation 'com.github.maricn:logback-slack-appender:1.4.0'
```



### iconEmoji 설정

iconEmoji는 slack 메시지를 표시할때 아이콘을 지정할 수 있으며, slack 내 아이콘을 지정하거나 사용자가 직접 아이콘을 업로드하여 이름을 지정하여 사용할 수 있습니다. 단 아이콘 추가 작업은 데스크톱 Slack에서만 추가할 수 있습니다. slack 대화창에서 아이콘을 클릭하면 아이콘 리스트에 마우스를 올리면 아이콘명이 나옵니다. 여기서는 직접 등록하여 사용했습니다.

무료아이콘 사이트: [https://slackmojis.com/](https://slackmojis.com/)

![image-20210624145104387](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210624145104387.png)

![image-20210624145351111](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210624145351111.png)

### application.yml 파일 설정

```yml
logging:
  slack:
      webhook-uri: https://hooks.slack.com/services/T025A3C7J73/B0253LZJD7W/....
      channel: "#error"
      username: potal-web-service-app
      iconEmoji: ":spring:"
  info-pattern: "%d{HH:mm:ss.SSS} %-5level %logger{36} - %msg%n"
  error-pattern: "%d{HH:mm:ss.SSS} %-5level %logger{36} url=[%X{mdcUrl}] params=[%X{mdcParams}] - %msg%n"
  file:
    path: C:\Users\webme\logs
    info-name: application
    error-name: application_error
```

webhook-uri: slack webhook url

channel: 에러 메시지를 받을 채널명

username: 메시지를 표시할때 메시지의 대화명

iconEmoji: 메시지를 표시할 때 메시지의 아이콘



### AOP 설정

기본적으로 Post로 전송 시에 Body에 포함된 Parameter가 sentry issue 상세 정보에 포함되지 않기 때문에 AOP를 이용해 Post Parameter를 MDC에 추가하고 logback에 출력되도록 합니다.

```java
@Configuration
@EnableAspectJAutoProxy
@RequiredArgsConstructor
@Aspect
@Slf4j
public class AspectConfig {
    private final MenuService menuService;
    private final HttpServletRequest request;
    private final Gson gson;
    final String MDC_PARAM_KEY = "mdcParams";
    final String MDC_URL_KEY = "mdcUrl";
    
    //...
    
    private String paramMapToString(Map<String, String[]> paramMap) {
        return paramMap.entrySet().stream()
            .map(entry -> String.format("%s=%s",
                                        entry.getKey(), entry.getValue()[0]))
            .collect(Collectors.joining(", "));
    }

    @Around("within(com.kdr.postal.controller..*)")
    public Object putMDCParameters(ProceedingJoinPoint joinPoint) throws Throwable {
        MDC.put(MDC_URL_KEY, request.getServletPath());

        if(request.getContentType() != null && request.getContentType().equals(MediaType.APPLICATION_JSON_VALUE)) {
            MDC.put(MDC_PARAM_KEY, gson.toJson(joinPoint.getArgs()[0]));
        }else  {
            MDC.put(MDC_PARAM_KEY, paramMapToString(request.getParameterMap()));
        }

        return joinPoint.proceed(joinPoint.getArgs());
    }
```



### logback-spring.xml 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>
  <springProperty  name="SLACK_WEBHOOK_URI" source="logging.slack.webhook-uri" />
  <springProperty  name="SLACK_CHANNEL" source="logging.slack.channel" />
  <springProperty  name="SLACK_USERNAME" source="logging.slack.username" />
  <springProperty  name="SLACK_ICONEMOJI" source="logging.slack.iconEmoji" />

  <springProperty  name="LOG_INFO_PATTERN" source="logging.info-pattern" />
  <springProperty  name="LOG_ERROR_PATTERN" source="logging.error-pattern" />
  <springProperty  name="LOG_PATH" source="logging.file.path" />
  <springProperty  name="LOG_INFO_FILE_NAME" source="logging.file.info-name" />
  <springProperty  name="LOG_ERROR_FILE_NAME" source="logging.file.error-name" />

  <!--INFO 로그 콘솔 출력 설정-->
   <appender name="INFO_SOUT" class="ch.qos.logback.core.ConsoleAppender">
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>INFO</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>
    <encoder>
      <pattern>${LOG_INFO_PATTERN}</pattern>
    </encoder>
  </appender>

  <!--오류 로그 콘솔 출력 설정-->
  <appender name="ERROR_SOUT" class="ch.qos.logback.core.ConsoleAppender">
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>ERROR</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
    </filter>
    <encoder>
      <pattern>${LOG_ERROR_PATTERN}</pattern>
    </encoder>
  </appender>

  <!--전체 로그 파일 설정-->
  <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <!--<filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>INFO</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
    </filter>-->
    <!-- 파일경로 설정 -->
    <file>${LOG_PATH}/${LOG_INFO_FILE_NAME}.log</file>
    <!-- 출력패턴 설정-->
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
      <pattern>${LOG_INFO_PATTERN}</pattern>
    </encoder>
    <!-- Rolling 정책 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- .gz,.zip 등을 넣으면 자동 일자별 로그파일 압축 -->
      <fileNamePattern>${LOG_PATH}/${LOG_INFO_FILE_NAME}.%d{yyyy-MM-dd}_%i.log</fileNamePattern>
      <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <!-- 파일당 최고 용량 kb, mb, gb -->
        <maxFileSize>10MB</maxFileSize>
      </timeBasedFileNamingAndTriggeringPolicy>
      <!-- 일자별 로그파일 최대 보관주기(~일), 해당 설정일 이상된 파일은 자동으로 제거-->
      <maxHistory>30</maxHistory>
      <!--<MinIndex>1</MinIndex> <MaxIndex>10</MaxIndex>-->
    </rollingPolicy>
  </appender>

  <!--ERROR 로그 파일 설정-->
  <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>ERROR</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
    </filter>
    <!-- 파일경로 설정 -->
    <file>${LOG_PATH}/${LOG_ERROR_FILE_NAME}.log</file>
    <!-- 출력패턴 설정-->
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
      <pattern>${LOG_ERROR_PATTERN}</pattern>
    </encoder>
    <!-- Rolling 정책 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- .gz,.zip 등을 넣으면 자동 일자별 로그파일 압축 -->
      <fileNamePattern>${LOG_PATH}/${LOG_ERROR_FILE_NAME}.%d{yyyy-MM-dd}_%i.log</fileNamePattern>
      <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <!-- 파일당 최고 용량 kb, mb, gb -->
        <maxFileSize>10MB</maxFileSize>
      </timeBasedFileNamingAndTriggeringPolicy>
      <!-- 일자별 로그파일 최대 보관주기(~일), 해당 설정일 이상된 파일은 자동으로 제거-->
      <maxHistory>30</maxHistory>
      <!--<MinIndex>1</MinIndex> <MaxIndex>10</MaxIndex>-->
    </rollingPolicy>
  </appender>

  <appender name="SLACK" class="com.github.maricn.logback.SlackAppender">
    <!-- Slack API token -->
    <!--<token>1111111111-1111111-11111111-111111111</token>-->
    <!-- Slack incoming webhook uri. Uncomment the lines below to use incoming webhook uri instead of API token. -->
    <webhookUri>${SLACK_WEBHOOK_URI}</webhookUri>
    <!-- Channel that you want to post - default is #general -->
    <channel>#${SLACK_CHANNEL}</channel>
    <!-- Formatting (you can use Slack formatting - URL links, code formatting, etc.) -->
    <layout class="ch.qos.logback.classic.PatternLayout">
      <pattern>${LOG_ERROR_PATTERN}</pattern>
    </layout>
    <!-- Username of the messages sender -->
    <username>${SLACK_USERNAME}</username>
    <!-- Emoji to be used for messages -->
    <iconEmoji>${SLACK_ICONEMOJI}</iconEmoji>
    <!-- If color coding of log levels should be used -->
    <colorCoding>true</colorCoding>
  </appender>

  <!-- Currently recommended way of using Slack appender -->
  <appender name="ASYNC_SLACK" class="ch.qos.logback.classic.AsyncAppender">
    <appender-ref ref="SLACK" />
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
      <level>ERROR</level>
    </filter>
  </appender>

  <!-- 로컬 환경 -->
  <springProfile name="local">
    <logger name="jdbc" level="OFF"/>
    <logger name="jdbc.sqlonly" level="OFF"/>
    <logger name="jdbc.sqltiming" level="DEBUG"/>
    <logger name="jdbc.audit" level="OFF"/>
    <logger name="jdbc.resultset" level="OFF"/>
    <logger name="jdbc.resultsettable" level="DEBUG"/>
    <logger name="jdbc.connection" level="OFF"/>

    <root level="INFO">
      <appender-ref ref="INFO_SOUT" />
      <appender-ref ref="ERROR_SOUT" />
    </root>
  </springProfile>

  <!-- 운영 환경 -->
  <springProfile name="prod">
    <logger name="jdbc" level="OFF"/>
    <logger name="jdbc.sqlonly" level="OFF"/>
    <logger name="jdbc.sqltiming" level="INFO"/>
    <logger name="jdbc.audit" level="OFF"/>
    <logger name="jdbc.resultset" level="OFF"/>
    <logger name="jdbc.resultsettable" level="INFO"/>
    <logger name="jdbc.connection" level="OFF"/>

    <root level="INFO">
      <appender-ref ref="ASYNC_SLACK" />
      <appender-ref ref="INFO_SOUT" />
      <appender-ref ref="ERROR_SOUT" />
      <appender-ref ref="ERROR_FILE" />
      <appender-ref ref="INFO_FILE" />
    </root>
  </springProfile>
</configuration>
```



### 테스트

임의로 오류를 발생 시키면 slack에서 webhook 메시지와 sentry에서 발생하는 메시지 2건이 보입니다. 추후에 sentry-slack연동을 하지 않더라도 webhook 메시지를 확인하고 sentry에 접속하여 상세 로그를 확인할 수 있습니다. 금전적 여유가 된다면 sentry-slack이 편합니다. 그 이유는 webhook 메시지가 sentry issue 페이지로 바로 가는 링크 기능이 없기 때문입니다. 아래 보이는 sentry 메시지를 클릭하면 issue 상세 페이지로 바로 연결되기 때문입니다.

<slack 메시지>

![image-20210629173257939](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210629173257939.png)

<sentry issue 상세화면>

![image-20210629173609561](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210629173609561.png)

![image-20210629173826895](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210629173826895.png)

![image-20210629173924749](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210629173924749.png)
