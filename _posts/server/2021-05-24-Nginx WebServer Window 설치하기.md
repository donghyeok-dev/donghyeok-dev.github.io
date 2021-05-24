---
layout: customPost
title:  "Nginx WebServer Window 설치 및 설정"
categories: 
  - Nginx
  - Server
---
## Nginx WebServer Window 설치 및 설정

### 다운로드

 http://nginx.org/en/download.html

![image-20210524145840346](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524145840346.png)



### nginx.conf 수정

nginx 압축폴더를 적당한 경로로 옮기고 압축을 풉니다.

nginx-1.20.0\conf\nginx.conf 파일을 열어 서버에 맞게 설정합니다.

```

#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
	client_max_body_size 100M;

    sendfile        on;
    keepalive_timeout  65;

    upstream service1{
          server test.co.kr:8081;
    }
    
    upstream service2{
          server test2.co.kr:8080;
    }
    
    server {
        listen       80;
        server_name  test.co.kr;

        return 301 https://$host$request_uri;
    }
    
    server {
        listen       80;
        server_name  test2.co.kr;

        return 301 https://$host$request_uri;
    }
    
    server {
        listen       443 ssl http2;
        listen       [::]:443 ssl http2;
        server_name  test.co.kr;

        ssl_certificate      ssl/server.pem;
        ssl_certificate_key  ssl/key.pem;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_protocols  TLSv1.2;
        ssl_ciphers  ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384;
        ssl_prefer_server_ciphers  on;
    

        location / {
            proxy_set_header    Host $http_host;
            proxy_set_header    X-Real-IP $remote_addr;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header    X-Forwarded-Proto $scheme;
            proxy_set_header    X-NginX-Proxy true;
            
            proxy_pass http://service1;
            proxy_redirect off;
            charset utf-8;
        }
    }
    
    server {
        listen       443 ssl http2;
        listen       [::]:443 ssl http2;
        server_name  test2.co.kr;

        ssl_certificate      ssl/server.pem;
        ssl_certificate_key  ssl/key.pem;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_protocols  TLSv1.2;
        ssl_ciphers  ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384;
        ssl_prefer_server_ciphers  on;

        location / {
            proxy_set_header    Host $http_host;
            proxy_set_header    X-Real-IP $remote_addr;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header    X-Forwarded-Proto $scheme;
            proxy_set_header    X-NginX-Proxy true;
            
            proxy_pass http://service2;
            proxy_redirect off;
            charset utf-8;
        }
    }
}

```

### SSL 설정

ssl 설정 파일에서 통합인증서(서버+체인+루트)와 key파일을 준비합니다.

crt 또는 pem으로 구성된 서버, 체인, 루트 파일이 분리 되어 있다면 

cmd 콘솔에서 아래와 같은 형식으로 통합인증서로 만듭니다. 순서는 서버-체인-루트 순으로 합치고,

만약 통합인증서가 존재한다면 아래 작업은 skip합니다.

```
type [인증서경로]/서버.crt.pem [인증서경로]/체인.pem [인증서경로]/루트.pem > server.pem
```

통합 인증서 파일과 key파일을 conf/ssl 폴더를 만들고 넣어줍니다.



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



