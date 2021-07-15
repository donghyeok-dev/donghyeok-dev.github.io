---
layout: customPost
title:  "Nginx 설치(Window)"
categories: 
  - Nginx
  - Server
---
## Nginx WebServer Window 설치 및 설정

### 다운로드

 http://nginx.org/en/download.html

![image-20210524145840346](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524145840346.png)



### 서비스 등록

콘솔창에서 nssm 명령어로 서비스를 등록합니다.

```
//등록
nssm install 서비스명

//수정
nssm edit 서비스명

//삭제 
sc delete 서비스명
```

![image-20210524152041119](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524152041119.png)

![image-20210524152118452](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524152118452.png)

### 오류 발생 ?

nginx 실행 시 아래와 같은 오류 발생 시 80포트 사용 중인 서비스를 찾아서 중지 시켜줘야 합니다.

```
2021/05/24 15:09:58 [emerg] 3192#2868: bind() to 0.0.0.0:80 failed (10013: An attempt was made to access a socket in a way forbidden by its access permissions)
```



![image-20210524151545323](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524151545323.png)



