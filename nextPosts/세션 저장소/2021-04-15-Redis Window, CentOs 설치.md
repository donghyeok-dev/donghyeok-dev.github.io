---
layout: customPost
title:  "Redis Window, CentOs 설치"
categories: 
  - Session
  - Tools
---



## Redis 사용 해보기



## Window Redis 설치

Redis는 보통 Linux에서 사용하지만 Window에서도 사용할 수 있습니다. Window 10에서 Redis를 설치하고 테스트 해보기로 했습니다.

https://github.com/microsoftarchive/redis/releases/tag/win-3.0.504에서 msi 확장자로 된 파일을 다운로드 받습니다.

다운로드 받은 파일 설치

![image-20210416094851598](C:\Users\webme\mygit\blog\assets\images\posts\image-20210416094851598.png)

![image-20210416094920943](C:\Users\webme\mygit\blog\assets\images\posts\image-20210416094920943.png)

![image-20210416094940423](C:\Users\webme\mygit\blog\assets\images\posts\image-20210416094940423.png)

![image-20210416095051275](C:\Users\webme\mygit\blog\assets\images\posts\image-20210416095051275.png)

![image-20210416095659911](C:\Users\webme\mygit\blog\assets\images\posts\image-20210416095659911.png)







## CentOS 7 Redis 설치

시스템 업데이트

```
sudo yum -y update
```



REMI repisitory 추가

```
sudo yum -y install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```



Redis 설치

```
sudo yum --enablerepo=remi install redis
```

<img src="C:\Users\webme\mygit\blog\assets\images\posts\image-20210521165419105.png" alt="image-20210521165419105" style="zoom:150%;" />



Redis 실행

```
sudo systemctl enable --now redis
```



Redis 설정파일

```
sudo vim /etc/redis.conf

#Redis에 접속할 ip 설정 0.0.0.0은 모든 ip접근 가능
bind 0.0.0.0 ::1
#bind 222.xxx.xxx.xxx 211.xxx.xxx.xxx 127.0.0.1
#bind 222.xxx.xxx.xxx
#bind 211.xxx.xxx.xxx

#데몬설정
daemonize yes

#패스워드 설정
# requirepass foobared

```



Redis 재시작

```
sudo systemctl restart redis
```



Redis 접속 

```
Redis-cli
```



Redis 서비스 상태 확인

```
sudo systemctl status redis

```

<img src="C:\Users\webme\mygit\blog\assets\images\posts\image-20210521170123368.png" alt="image-20210521170123368" style="zoom:150%;" />



### 외부접속을 위한 방화벽 오픈

방화벽 실행 여부 확인

```
firewall-cmd --state
```



redis 방화벽 설정

```
$ sudo firewall-cmd --zone=public --add-service=redis --permanent
$ sudo firewall-cmd --reload
```



## Redis 오류 관련

```
org.springframework.data.redis.RedisSystemException: 
Error in execution; nested exception is io.lettuce.core.RedisCommandExecutionException: MISCONF Redis is configured to save RDB snapshots,
 but it is currently not able to persist on disk. Commands that may modify the data set are disabled, 
 because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option).
  Please check the Redis logs for details about the RDB error.
```

Redis는 프로세스가 장애로 인해 종료되더라도 이전의 상태를 동일하게 복구하기 위해서 자동으로 .rdb 라는 확장자의 파일에 인메모리 데이터를 저장합니다. rdb 파일을 저장하려 할때 Redis는 하위 프로세스를 포킹하여 데이터를 디스크에 저장합니다. 이때 포크를 제대로 하지 못해 발생할 수 있습니다. 이를 해결 하기 위해 아래와 같은 설정을 합니다.

/etc/sysctl.conf

```
vm.overcommit_memory=1
```

```
# sudo sysctl -p /etc/sysctl.conf
```

단순히 이 오류 메시지를 무시하려면  /etc/redis.conf을 열어서 아래와 같이 설정합니다.

```
stop-writes-on-bgsave-error no
```



redis rdb 파일이나 로그 파일의 경로를 확인하려면 /etc/redis.conf에서 dir 설정부분을 보면 됩니다. 이 디렉토리에는 log, rdb, appendonly, nodes.conf 파일이 위치합니다

```
dir /var/lib/redis
```

<img src="C:\Users\webme\mygit\blog\assets\images\posts\image-20210524112835453.png" alt="image-20210524112835453" style="zoom:150%;" />