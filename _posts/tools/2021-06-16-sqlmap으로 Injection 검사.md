---
layout: customPost
title:  "sqlmap으로 Injection 검사"
categories: 
  - Tools
  - sqlmap
---
### 설치하기

sqlmap은 python기반으로 동작하므로 python을 먼저 설치해야되며 지원되는 버전은 아래와 같습니다.

sqlmap works out of the box with [Python](http://www.python.org/download/) version **2.6**, **2.7** and **3.x** on any platform.

https://www.python.org/downloads/windows/



sqlmap 설치:  https://sqlmap.org/



### 테스트 하기

### 실행

```commonlisp
sqlmap -u "http://localhost:8080/testInjection" --data="mbrName=%EA%B8%10" --cookie="JSESSIONID=192A5BAA7DFC45A5F6E42798022849EA" -p mbrName --dbs --dbms=Oracle --level=5 --risk=3 --threads=5
```

-u : 테스트 주소

--data: Post 전송 데이터

--cookie: 로그인이 필요한 경우 로그인된 세션ID 등의 쿠키값.

-p: 테스트할 파라메터

--dbs: 데이터베이스 리스트 나열

--dbms: 테스트할 데이터베이스

--level --risk: 테스트 강도

--threads: 실행 스레드



### 취약점이 존재하는 경우

![image-20210616142535283](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210616142535283.png)

### 취약점이 없는 경우

![image-20210616142640939](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210616142640939.png)



### Mybatis 취약 사례

```sql
SELECT ID, NAME
FROM TEMP
WHERE NAME LIKE '%${name}%'
```

위와 같이 #{...}가 아닌 ${...}로 매개변수를 바인딩하는 경우 sql injection 취약점이 발생합니다. #{...}는 내부적으로 PreparedStatement를 사용하여 특수문자, 쿼리등이 필터링되므로 #{...}를 사용하여 바인딩해야 합니다.

