---
layout: customPost
title:  "CentOS 7 Redis 설치와 설정"
categories: 
  - Session
  - Server
---

## CentOS 7 Redis 설치와 설정

---

# Redis 구성



<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210528165507979.png" alt="image-20210528165507979" style="zoom:150%;" />

---

## Redis 사전 작업

---

### 1. 시스템 업데이트

#sudo yum -y update

---

### 2. 패키지 설치

#sudo yum -y install gcc-c++ 

#sudo yum install wget

#sudo yum install tcl tk

---

### 3. 메모리 설정

```
org.springframework.data.redis.RedisSystemException: 
Error in execution; nested exception is io.lettuce.core.RedisCommandExecutionException: MISCONF Redis is configured to save RDB snapshots,
 but it is currently not able to persist on disk. Commands that may modify the data set are disabled, 
 because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option).
  Please check the Redis logs for details about the RDB error.
```

Redis는 프로세스가 장애로 인해 종료되더라도 이전의 상태를 동일하게 복구하기 위해서 자동으로 .rdb 라는 확장자의 파일에 인메모리 데이터를 저장합니다. rdb 파일을 저장하려 할때 Redis는 하위 프로세스를 포킹하여 데이터를 디스크에 저장합니다. 이때 포크를 제대로 하지 못해 발생할 수 있습니다. 이를 해결 하기 위해 아래와 같은 설정을 합니다.

메모리 사용량이 허용량을 초과할 경우, overcommit을 처리하는 방식 결정하는 값을 "항상"으로 변경하고, 기본 값은 "0" 입니다.

0 : 커널 기본값, Heuristic 하게 Overcommit을 허용
(Page Cache + Swap Memory + Slab Reclaimable 값이 요청한 메모리 수 보다 클 경우 허용)
1 : 항상 Overcommit을 허용
2 : 제한적 Overcommit 허용

---

# 적용

#sudo sysctl vm.overcommit_memory=1

#sudo echo "vm.overcommit_memory=1" >> /etc/sysctl.conf 

---

# 적용 확인

#sudo sysctl -a | grep vm.overcommit

sysctl: reading key "net.ipv6.conf.all.stable_secret"
sysctl: reading key "net.ipv6.conf.default.stable_secret"
sysctl: reading key "net.ipv6.conf.ens32.stable_secret"
sysctl: reading key "net.ipv6.conf.lo.stable_secret"
vm.overcommit_kbytes = 0
vm.overcommit_memory = 1
vm.overcommit_ratio = 50

단순히 이 오류 메시지를 무시하려면  /etc/redis.conf을 열어서 아래와 같이 설정합니다.

#vim /etc/sysctl.conf

stop-writes-on-bgsave-error no

---

### 4. TCP BackLog 설정

```
WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
```

기본적인 소켓 accept limit 값입니다. 보통 리눅스 배포판에는 128로 되어있고 이 때문에 tcp backlog 의 셋팅이 511 로 되어있지만 강제로 128로 적용 된다는 뜻입니다.서버 소켓에 Accept를 대기하는 소켓 개수 파라미터를 변경해줍니다. 기본 Accept limit은 128이고 1024 이상으로 변경합니다. (최대 65535)

### 설정

#sudo sysctl -w net.core.somaxconn=1024

#sudo echo "net.core.somaxconn=1024" >> /etc/sysctl.conf

### 확인

#sudo sysctl -a | grep somaxconn

net.core.somaxconn = 1024
sysctl: reading key "net.ipv6.conf.all.stable_secret"
sysctl: reading key "net.ipv6.conf.default.stable_secret"
sysctl: reading key "net.ipv6.conf.ens32.stable_secret"
sysctl: reading key "net.ipv6.conf.lo.stable_secret"

---

#### 5. THP 설정

THP(Transparent Huge Pages) 기능이 Enable 되어 있는 경우 Redis에서는 이를 Disable 시킬 것을 권장합니다. 재부팅 시 설정을 변경하기 위해 /etc/rc.local 에도 명령어를 추가해 줍니다.

```

# echo 'never' >/sys/kernel/mm/transparent_hugepage/enabled
```

재부팅시 적용을 위해 rc.local에도 적용합니다.

```
# vim /etc/rc.local
//맨 아래줄에 추가합니다.
echo 'never' >/sys/kernel/mm/transparent_hugepage/enabled
exit 0

# reboot
```

# Redis 설치

---

# rpm으로 설치한게 있다면 삭제

#sudo yum remove redis

---

# Redis 최신 버전 다운로드

#wget https://download.redis.io/releases/redis-6.2.3.tar.gz 

---

# 압축 풀기

#tar -xvf  tar -xvf redis-6.2.3.tar.gz

---

# 폴더명 변경

#mv redis-6.2.3 ./redis

---

# 컴파일

#cd redis

#ls

00-RELEASENOTES  CONTRIBUTING  MANIFESTO  TLS.md      runtest            runtest-sentinel  tests
BUGS             COPYING       Makefile   deps        runtest-cluster    sentinel.conf     utils
CONDUCT          INSTALL       README.md  redis.conf  runtest-moduleapi  src

#sudo make

cd src && make all
make[1]: Entering directory `/usr/local/src/redis/src'
    CC Makefile.dep
make[1]: Leaving directory `/usr/local/src/redis/src'
make[1]: Entering directory `/usr/local/src/redis/src'

...

jemalloc version   : 5.1.0-0-g0
library revision   : 2

...

Hint: It's a good idea to run 'make test' ;)

---

# Redis 설치 (redis 실행파일이 /usr/local/bin로 복사됨.)

#sudo make install

    CC Makefile.dep

Hint: It's a good idea to run 'make test' ;)

INSTALL redis-server
INSTALL redis-benchmark
INSTALL redis-cli

---

# /usr/local/bin 확인

#ls /usr/local/bin | grep redis

redis-benchmark
redis-check-aof
redis-check-rdb
redis-cli
redis-sentinel
redis-server

---

### Redis Master-Slave 구성

```
# cd /usr/local/src/redis/utils

# ./install_server.sh
Welcome to the redis service installer
This script will help you easily set up a running redis server

This systems seems to use systemd.
Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry

# vim install_server.sh
// 해당 라인 주석처리.
```

<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210526142149545.png" alt="image-20210526142149545" style="zoom:150%;" />

---

# Master Redis 서버 설치

#./install_server.sh

Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 6379 // Redis 실행 포트설정 
Please select the redis config file name [/etc/redis/6379.conf] /usr/local/src/redis/conf/6379.conf // 설정파일 위치 지정
Please select the redis log file name [/var/log/redis_6379.log] /usr/local/src/redis/logs/6379.log // 로그저장 위치 지정
Please select the data directory for this instance [/var/lib/redis/6379] /usr/local/src/redis/6379 // 데이터 디렉터리 위치 지정
Please select the redis executable path [/usr/local/bin/redis-server] //엔터, 실행 위치는 디폴트값 사용
Selected config:
Port           : 6379
Config file    : /usr/local/src/redis/conf/6379.conf
Log file       : /usr/local/src/redis/logs/6379.log
Data dir       : /usr/local/src/redis/6379
Executable     : /usr/local/bin/redis-server
Cli Executable : /usr/local/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort. //엔터
Copied /tmp/6379.conf => /etc/init.d/redis_6379
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!

---

# Slave Redis #1 서버 설치

#./install_server.sh

Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 6380
Please select the redis config file name [/etc/redis/6380.conf] /usr/local/src/redis/conf/6380.conf
Please select the redis log file name [/var/log/redis_6380.log] /usr/local/src/redis/logs/6380.log
Please select the data directory for this instance [/var/lib/redis/6380] /usr/local/src/redis/6380
Please select the redis executable path [/usr/local/bin/redis-server]
Selected config:
Port           : 6380
Config file    : /usr/local/src/redis/conf/6380.conf
Log file       : /usr/local/src/redis/logs/6380.log
Data dir       : /usr/local/src/redis/6380
Executable     : /usr/local/bin/redis-server
Cli Executable : /usr/local/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6380.conf => /etc/init.d/redis_6380
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!

---

# Slave Redis #2 서버 설치

#./install_server.sh

Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 6381
Please select the redis config file name [/etc/redis/6381.conf] /usr/local/src/redis/conf/6381.conf
Please select the redis log file name [/var/log/redis_6381.log] /usr/local/src/redis/logs/6381.log
Please select the data directory for this instance [/var/lib/redis/6381] /usr/local/src/redis/6381
Please select the redis executable path [/usr/local/bin/redis-server]
Selected config:
Port           : 6381
Config file    : /usr/local/src/redis/conf/6381.conf
Log file       : /usr/local/src/redis/logs/6381.log
Data dir       : /usr/local/src/redis/6381
Executable     : /usr/local/bin/redis-server
Cli Executable : /usr/local/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6381.conf => /etc/init.d/redis_6381
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!

---

# Redis 서버 확인

#ps aux | grep redis

root     13717  0.1  0.1 165060  9852 ?        Ssl  17:53   0:00 /usr/local/bin/redis-server 0.0.0.0:6379
root     13727  0.1  0.1 240840  9800 ?        Ssl  17:54   0:00 /usr/local/bin/redis-server 0.0.0.0:6380
root     13737  0.1  0.1 240840  9980 ?        Ssl  17:54   0:00 /usr/local/bin/redis-server 0.0.0.0:6381

---

# Redis 데몬 복사

#/usr/bin/cp -f /tmp/6379.conf /etc/init.d/redis_6379

#/usr/bin/cp -f /tmp/6380.conf /etc/init.d/redis_6380

#/usr/bin/cp -f /tmp/6381.conf /etc/init.d/redis_6381

#ls /etc/init.d

README  functions  netconsole  network  redis_6379  redis_6380  redis_6381

---

# Redis 데몬 중지
#sudo /etc/init.d/redis_6379 stop
#sudo /etc/init.d/redis_6380 stop
#sudo /etc/init.d/redis_6381 stop

---

# 중지 확인

#ps aux | grep redis

root     17163  0.0  0.0 116972  1016 pts/0    S+   15:40   0:00 grep --color=auto redis

---

# Redis 인스턴스별 상태, 실행중이지 않으면 /var/run/redis_6381.pid가 생성되지 않기 때문에 아래와 같이 출력된다.

#sudo /etc/init.d/redis_6381 status

cat: /var/run/redis_6381.pid: 그런 파일이나 디렉터리가 없습니다
Redis is running ()

# 실행 중이면 /var/run/redis_6381.pid가 생성되어 정상 출력된다. 

#sudo /etc/init.d/redis_6381 start

Starting Redis server...

#sudo /etc/init.d/redis_6381 status

Redis is running (17310)

#sudo /etc/init.d/redis_6381 stop

---

# 사용하는 사용자 그룹이 있다면 redis 하위폴더 포함 owner를 사용자계정으로 변경 

#chown -R 사용자:그룹 redis

---

# Master 파일설정 

#vim /usr/local/src/redis/conf/6379.conf


bind 0.0.0.0

daemonize yes

requirepass test1234

masterauth test1234

//로그 레벨

//debug (많은 정보, 개발/개발에 유용)

//verbose(Debug 수준 같은 엉망진창은 아니지만 거의 유용하지 않은 많은 정보)

//notice (자세한 내용, 제작 시 원하는 내용)

//warning(매우 중요한/중요한 메시지만 기록됨)

loglevel notice

logfile /usr/local/src/redis/logs/6379.log

---

# slave #1 파일설정 

#vim /usr/local/src/redis/conf/6380.conf

bind 0.0.0.0

daemonize yes

requirepass test1234

replicaof 211.240.xxx.xxx 6379

masterauth test1234

repl-ping-slave-period 10 //서버 동기화 주기, 주석제거
repl-timeout 60 주석제거

---

# slave #2 파일설정 

#vim /usr/local/src/redis/conf/6381.conf

bind 0.0.0.0

daemonize yes

replicaof 211.240.xxx.xxx 6379

masterauth test1234

repl-ping-slave-period 10 //서버 동기화 주기, 주석제거
repl-timeout 60 주석제거

---

# Redis 데몬 패스워드 설정

설정안하면 데몬 stop시 Waiting for Redis to shutdown ... 발생

#vim /etc/init.d/redis_6379

#vim /etc/init.d/redis_6380

#vim /etc/init.d/redis_6381

#!/bin/sh
#Configurations injected by install_server below....

EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli
PIDFILE=/var/run/redis_6381.pid
CONF="/usr/local/src/redis/conf/6381.conf"
REDISPORT="6381"
REDISPW=test1234 //추가

...

stop)
        if [ ! -f $PIDFILE ]
        then
            echo "$PIDFILE does not exist, process is not running"
        else
            PID=$(cat $PIDFILE)
            echo "Stopping ..."
            $CLIEXEC -p $REDISPORT -a $REDISPW shutdown     //이부분에 -a $REDISPW추가

....

:wq

---

# 실행

#sudo /etc/init.d/redis_6379 start

#sudo /etc/init.d/redis_6380 start

#sudo /etc/init.d/redis_6381 start

---

# 실행 확인

#ps aux | grep redis

root      8288  0.9  0.1 165060  9908 ?        Ssl  11:00   0:05 /usr/local/bin/redis-server 0.0.0.0:6379
root      8532  0.1  0.1 240840 10068 ?        Ssl  11:08   0:00 /usr/local/bin/redis-server 0.0.0.0:6380
root      8562  0.1  0.1 240840 10072 ?        Ssl  11:08   0:00 /usr/local/bin/redis-server 0.0.0.0:6381
root      8598  0.0  0.0 116972  1020 pts/1    S+   11:10   0:00 grep --color=auto redis

---

# 마스터 로그 확인

#vim /usr/local/src/redis/logs/6379.log

13717:M 27 May 2021 17:54:22.972 * Replication backlog created, my new replication IDs are '818b4519e78e443ee4f9bde17b13496e22b352dc' and '0000000000000000000000000000000000000000'
13717:M 27 May 2021 17:54:22.972 * Starting BGSAVE for SYNC with target: disk
13717:M 27 May 2021 17:54:22.973 * Background saving started by pid 13732
13732:C 27 May 2021 17:54:22.976 * DB saved on disk
13732:C 27 May 2021 17:54:22.977 * RDB: 0 MB of memory used by copy-on-write
13717:M 27 May 2021 17:54:23.032 * Background saving terminated with success
13717:M 27 May 2021 17:54:23.032 * Synchronization with replica 211.240.xxx.xxx:6380 succeeded
13717:M 27 May 2021 17:54:42.335 * Replica 211.240.xxx.xxx:6381 asks for synchronization
13717:M 27 May 2021 17:54:42.335 * Partial resynchronization not accepted: Replication ID mismatch (Replica asked for 'c6ba9a5157368f7d2c7b1771b887c8c2dcee9974', my replication IDs are '818b4519e78e443ee4f9bde17b13496e22b352dc' and '0000000000000000000000000000000000000000')
13717:M 27 May 2021 17:54:42.335 * Starting BGSAVE for SYNC with target: disk
13717:M 27 May 2021 17:54:42.335 * Background saving started by pid 13742
13742:C 27 May 2021 17:54:42.338 * DB saved on disk
13742:C 27 May 2021 17:54:42.339 * RDB: 0 MB of memory used by copy-on-write
13717:M 27 May 2021 17:54:42.387 * Background saving terminated with success
13717:M 27 May 2021 17:54:42.387 * Synchronization with replica 211.240.xxx.xxx:6381 succeeded

---

# Slave 로그 확인

#vim /usr/local/src/redis/logs/6380.log
#vim /usr/local/src/redis/logs/6381.log

88532:C 27 May 2021 11:08:13.547 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
8532:C 27 May 2021 11:08:13.547 # Redis version=6.2.3, bits=64, commit=00000000, modified=0, pid=8532, just started
8532:C 27 May 2021 11:08:13.547 # Configuration loaded
8532:S 27 May 2021 11:08:13.548 * Increased maximum number of open files to 10032 (it was originally set to 1024).
8532:S 27 May 2021 11:08:13.548 * monotonic clock: POSIX clock_gettime
8532:S 27 May 2021 11:08:13.549 * Running mode=standalone, port=6380.
8532:S 27 May 2021 11:08:13.549 # Server initialized
8532:S 27 May 2021 11:08:13.549 * Loading RDB produced by version 6.2.3
8532:S 27 May 2021 11:08:13.549 * RDB age 430 seconds
8532:S 27 May 2021 11:08:13.549 * RDB memory usage when created 0.83 Mb
8532:S 27 May 2021 11:08:13.550 * DB loaded from disk: 0.000 seconds
8532:S 27 May 2021 11:08:13.550 * Ready to accept connections
8532:S 27 May 2021 11:08:13.551 * Connecting to MASTER 211.240.xxx.xxx:6379
8532:S 27 May 2021 11:08:13.551 * MASTER <-> REPLICA sync started
8532:S 27 May 2021 11:08:13.551 * Non blocking connect for SYNC fired the event.
8532:S 27 May 2021 11:08:13.551 * Master replied to PING, replication can continue...
8532:S 27 May 2021 11:08:13.552 * Partial resynchronization not possible (no cached master)
8532:S 27 May 2021 11:08:13.553 * Full resync from master: d495eb6da5b58f21f7c3921199363305fe6b7da7:0
8532:S 27 May 2021 11:08:13.558 * MASTER <-> REPLICA sync: receiving 996 bytes from master to disk
8532:S 27 May 2021 11:08:13.558 * MASTER <-> REPLICA sync: Flushing old data
8532:S 27 May 2021 11:08:13.558 * MASTER <-> REPLICA sync: Loading DB in memory
8532:S 27 May 2021 11:08:13.559 * Loading RDB produced by version 6.2.3
8532:S 27 May 2021 11:08:13.559 * RDB age 0 seconds
8532:S 27 May 2021 11:08:13.559 * RDB memory usage when created 1.87 Mb
8532:S 27 May 2021 11:08:13.559 * MASTER <-> REPLICA sync: Finished with success
8532:S 27 May 2021 13:28:48.717 * 1 changes in 3600 seconds. Saving...
8532:S 27 May 2021 13:28:48.720 * Background saving started by pid 11249
11249:C 27 May 2021 13:28:48.724 * DB saved on disk
11249:C 27 May 2021 13:28:48.725 * RDB: 0 MB of memory used by copy-on-write
8532:S 27 May 2021 13:28:48.821 * Background saving terminated with success

---

# Replication 확인

#redis-cli -p 6379

127.0.0.1:6379> AUTH test1234
OK
127.0.0.1:6379> info replication

Replication

role:master
connected_slaves:2
slave0:ip=211.240.xxx.xxx,port=6380,state=online,offset=938,lag=1
slave1:ip=211.240.xxx.xxx,port=6381,state=online,offset=938,lag=1
master_failover_state:no-failover
master_replid:d495eb6da5b58f21f7c3921199363305fe6b7da7
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:938
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:938

127.0.0.1:6379> quit



#redis-cli -p 6380
127.0.0.1:6380> AUTH test1234
OK
127.0.0.1:6380> info replication

Replication

role:slave
master_host:211.240.xxx.xxx
master_port:6379
master_link_status:up
master_last_io_seconds_ago:8
master_sync_in_progress:0
slave_repl_offset:11354
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:d495eb6da5b58f21f7c3921199363305fe6b7da7
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:11354
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:11354

127.0.0.1:6380> quit

---

# Replication 테스트

# 테스트1. Master 데이터가 Slave에 잘 복사 되는지 체크

접속후 AUTH 패스워드를 입력해도 되고, Redis 접속 시 -a 옵션으로 암호를 입력해도 됩니다.

#redis-cli -p 6379 -a test1234

Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> set replica_test helloworld
OK
127.0.0.1:6379> get replica_test
"helloworld"

127.0.0.1:6379> quit

#redis-cli -p 6380 -a test1234
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6380> get replica_test
"helloworld"

127.0.0.1:6380> quit

Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.

command line에서 입력하므로 패스워드가 로그로 남아 유출될 수 있습니다.

-a  대신 --askpass 옵션을 써서 암호를 입력합니다.

#redis-cli -p 6381 --askpass
Please input password: *********
127.0.0.1:6381> info replication

Replication

role:slave
master_host:211.240.xxx.xxx
master_port:6379
master_link_status:up
master_last_io_seconds_ago:5
master_sync_in_progress:0
slave_repl_offset:12630
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:d495eb6da5b58f21f7c3921199363305fe6b7da7
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:12630
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:71
repl_backlog_histlen:12560

---

# 테스트2. 마스터를 shutdown 한다면? 

#sudo /etc/init.d/redis_6379 stop

#vim /usr/local/src/redis/logs/6380.log

8532:S 27 May 2021 14:38:01.558 # Error condition on socket for SYNC: Connection refused
8532:S 27 May 2021 14:38:02.560 * Connecting to MASTER 211.240.xxx.xxx:6379
8532:S 27 May 2021 14:38:02.561 * MASTER <-> REPLICA sync started
8532:S 27 May 2021 14:38:02.561 # Error condition on socket for SYNC: Connection refused
8532:S 27 May 2021 14:38:03.563 * Connecting to MASTER 211.240.xxx.xxx:6379
8532:S 27 May 2021 14:38:03.564 * MASTER <-> REPLICA sync started

.... Master Redis가 정상 동작할 때까지 동일한 메시지가 반복해서 발생합니다. 

이상태로 방치하면 로그파일 크기가 엄청나게 커져버리기 때문에 모니터링이 필수 입니다.

또한 마스터만 read/write가 되고 slave는 read만 되기 때문에 read만 되는 상태가 됩니다.

다시 Master Redis를 시작하고 로그와 정보를 확인합니다.

#sudo /etc/init.d/redis_6379 start

#vim /usr/local/src/redis/logs/6380.log

8532:S 27 May 2021 14:38:03.564 * Non blocking connect for SYNC fired the event.
8532:S 27 May 2021 14:38:03.564 * Master replied to PING, replication can continue...
8532:S 27 May 2021 14:38:03.564 * Trying a partial resynchronization (request d495eb6da5b58f21f7c3921199363305fe6b7da7:41942).
8532:S 27 May 2021 14:38:03.565 * Full resync from master: 2c728a8720bc19ea3bdfceed46f6d41dd78ecb9c:0
8532:S 27 May 2021 14:38:03.565 * Discarding previously cached master state.
8532:S 27 May 2021 14:38:03.604 * MASTER <-> REPLICA sync: receiving 3180 bytes from master to disk
8532:S 27 May 2021 14:38:03.604 * MASTER <-> REPLICA sync: Flushing old data
8532:S 27 May 2021 14:38:03.605 * MASTER <-> REPLICA sync: Loading DB in memory
8532:S 27 May 2021 14:38:03.605 * Loading RDB produced by version 6.2.3
8532:S 27 May 2021 14:38:03.605 * RDB age 0 seconds
8532:S 27 May 2021 14:38:03.605 * RDB memory usage when created 1.84 Mb
8532:S 27 May 2021 14:38:03.606 * MASTER <-> REPLICA sync: Finished with success
8532:S 27 May 2021 14:38:28.033 * 100 changes in 300 seconds. Saving...
8532:S 27 May 2021 14:38:28.034 * Background saving started by pid 12162
12162:C 27 May 2021 14:38:28.037 * DB saved on disk
12162:C 27 May 2021 14:38:28.038 * RDB: 0 MB of memory used by copy-on-write
8532:S 27 May 2021 14:38:28.134 * Background saving terminated with success

#redis-cli -p 6379 --askpass

Please input password: *********
127.0.0.1:6379> keys *
1) "spring:session:expirations:1622095740000"
2) "replica_test"
3) "spring:session:index:org.springframework.session.FindByIndexNameSessionRepository.PRINCIPAL_NAME_INDEX_NAME:kdrco"
4) "menusCache::POSTAL_MENU"
5) "spring:session:sessions:expires:930ed11b-394d-419a-af57-8fab9dae1656"
6) "spring:session:sessions:930ed11b-394d-419a-af57-8fab9dae1656"
7) "spring:session:expirations:1622095440000"

127.0.0.1:6379>quit

#redis-cli -p 6380 --askpass

Please input password: *********
127.0.0.1:6380> keys *
1) "menusCache::POSTAL_MENU"
2) "spring:session:sessions:930ed11b-394d-419a-af57-8fab9dae1656"
3) "spring:session:sessions:expires:930ed11b-394d-419a-af57-8fab9dae1656"
4) "spring:session:expirations:1622095440000"
5) "replica_test"
6) "spring:session:index:org.springframework.session.FindByIndexNameSessionRepository.PRINCIPAL_NAME_INDEX_NAME:kdrco"
7) "spring:session:expirations:1622095740000"



하지만 현재 상태로는 Master 서버에 장애가 발생했을때 Slave를 자동으로 Master로 승격해주지 못합니다.

이러한 장애에 자동으로 failover 해주는 도구가 Sentinel(센티널)입니다.

---

# Sentinel(센티널)

Sentinel는 Master Redis를 감시하고 있다가 비정상적인 중단이 발생하면 자동으로 Slave Redis 중 한 개를 채택해서 Master로 승격 시켜주는 FailOver 역할을 합니다. Sentinel API를 통해 관리자나 프로그램에 피드백을 줄 수 있으며, 보통 Sentinel은 3대로 구성합니다. 가능하면 홀 수개로 구성하는데 그 이유는 Master Redis가 중단되었는지 판단할 때 다수결로 결정하기 때문에 홀수로 한다고 합니다. Master Redis가 정상 동작중인데 Master Redis 서버와 Sentinel 서버간의 네트워크 문제가 발생할 수 도 있기 때문에 3대의 Sentinel 중 Master Redis와 통신이 2대 이상 안될 경우 Master Redis가 중단되었다고 판단합니다. 이를 객관적인 다운상태라고 합니다.

---

# Sentinel 설정하기

#cd /usr/local/src/redis

#ls

00-RELEASENOTES  6381     CONTRIBUTING  MANIFESTO  TLS.md  logs        runtest-cluster    sentinel.conf  utils
6379             BUGS     COPYING       Makefile   conf    redis.conf  runtest-moduleapi  src
6380             CONDUCT  INSTALL       README.md  deps    runtest     runtest-sentinel   tests

원본 파일을 그대로 두고 복제 파일을 만듭니다.

---

# Sentinel #1 설정

#cp sentinel.conf ./conf/sentinel_7000.conf

#vim ./conf/sentinel_7000.conf

//첫번째 Sentinel 서버 포트를 지정합니다.

port 7000     

daemonize yes

pidfile /var/run/redis-sentinel_7000.pid

logfile "/usr/local/src/redis/logs/redis-sentinel_7000.log"

// Master Redis IP, PORT 그리고 제일 뒤에 숫자 2는 Master Redis가 중단되었다고 판단하는 수 이며, quorum(쿼럼)이라고 부릅니다.

// Sentinel 서버를 한대로 사용한다면 1, 3대면 2 이런씩으로 다수결시 결정될 수 있는 값으로 입력해야 됩니다.

sentinel monitor mymaster 211.240.xxx.xxx 6379 2

// 몇초동안 Master Redis로 연결이 실패 했을 때  중단이라고 판단내릴지의 시간입니다. 단위는 밀리세컨드이고 기본값은 30초입니다.

sentinel down-after-milliseconds mymaster 50000

// Master Redis가 중단되었을때 Master에서 Slave로 sync(동기화)해야되는데 동시에 몇개의 Slave를 동기화 시킬지의 개수입니다.

sentinel parallel-syncs mymaster 1

// failOver의 타임아웃으로 기본값은 3분입니다.

sentinel failover-timeout mymaster 180000

// 주석을 풀고 Master Redis 비밀번호를 입력합니다. Master Redis와 Slave Redis 비밀번호를 동일하게 맞춰줘야 합니다.

 sentinel auth-pass mymaster test1234

---

# Sentinel #2 설정

#cp sentinel_7000.conf ./conf/sentinel_7001.conf

#vim ./conf/sentinel_7001.conf

port 7001     

pidfile /var/run/redis-sentinel_7001.pid

logfile "/usr/local/src/redis/logs/redis-sentinel_7001.log"

---

# Sentinel #3 설정

#cp sentinel_7000.conf ./conf/sentinel_7002.conf

#vim ./conf/sentinel_7002.conf

port 7002     

pidfile /var/run/redis-sentinel_7002.pid

logfile "/usr/local/src/redis/logs/redis-sentinel_7002.log"

---

# Sentinel 데몬 작성

#cd /etc/init.d/

#ls
README  functions  netconsole  network  redis_6379  redis_6380  redis_6381

---

# Sentinel #1

#cp redis_6379 ./sentinel_7000

#vim sentinel_7000

EXEC=/usr/local/bin/redis-sentinel
CLIEXEC=/usr/local/bin/redis-cli
PIDFILE=/var/run/redis-sentinel_7000.pid
CONF="/usr/local/src/redis/conf/sentinel_7000.conf"
REDISPORT="7000"
REDISPW=test1234

---

# Sentinel #2

#cp sentinel_7000 ./sentinel_7001

#vim sentinel_7001

EXEC=/usr/local/bin/redis-sentinel
CLIEXEC=/usr/local/bin/redis-cli
PIDFILE=/var/run/redis-sentinel_7001.pid
CONF="/usr/local/src/redis/conf/sentinel_7001.conf"
REDISPORT="7001"
REDISPW=test1234

---

# Sentinel #3

#cp sentinel_7000 ./sentinel_7002

#vim sentinel_7002

EXEC=/usr/local/bin/redis-sentinel
CLIEXEC=/usr/local/bin/redis-cli
PIDFILE=/var/run/redis-sentinel_7002.pid
CONF="/usr/local/src/redis/conf/sentinel_7002.conf"
REDISPORT="7002"
REDISPW=test1234

---

# Sentinel 시작하기

#sudo /etc/init.d/sentinel_7000 start

#sudo /etc/init.d/sentinel_7001 start

#sudo /etc/init.d/sentinel_7002 start

#ps aux | grep redis

root      8532  0.1  0.1 240840 10288 ?        Ssl  11:08   0:22 /usr/local/bin/redis-server 0.0.0.0:6380
root      8562  0.1  0.1 240840 10288 ?        Ssl  11:08   0:22 /usr/local/bin/redis-server 0.0.0.0:6381
root     12154  0.1  0.1 165060  9928 ?        Ssl  14:38   0:08 /usr/local/bin/redis-server 0.0.0.0:6379
root     12906  0.6  0.1 162500 10256 ?        Ssl  16:28   0:01 /usr/local/bin/redis-sentinel *:7000 [sentinel]
root     12918  0.7  0.1 162500 10260 ?        Ssl  16:28   0:01 /usr/local/bin/redis-sentinel *:7001 [sentinel]
root     12927  0.6  0.1 162628 10296 ?        Ssl  16:28   0:01 /usr/local/bin/redis-sentinel *:7002 [sentinel]
root     12948  0.0  0.0 116972  1020 pts/3    S+   16:31   0:00 grep --color=auto redis

#vim /usr/local/src/redis/logs/redis-sentinel_7000.log

#vim /usr/local/src/redis/logs/redis-sentinel_7001.log

#vim /usr/local/src/redis/logs/redis-sentinel_7002.log

28037:X 28 May 2021 10:42:10.216 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
28037:X 28 May 2021 10:42:10.217 # Redis version=6.2.3, bits=64, commit=00000000, modified=0, pid=28037, just started
28037:X 28 May 2021 10:42:10.217 # Configuration loaded
28037:X 28 May 2021 10:42:10.218 * Increased maximum number of open files to 10032 (it was originally set to 1024).
28037:X 28 May 2021 10:42:10.218 * monotonic clock: POSIX clock_gettime
28037:X 28 May 2021 10:42:10.219 * Running mode=sentinel, port=7000.
28037:X 28 May 2021 10:42:10.222 # Sentinel ID is bc3291c7c38dd85f2fd58c4453aa194ac21a8c44
28037:X 28 May 2021 10:42:10.222 # +monitor master mymaster 211.240.xxx.xxx 6379 quorum 2
28037:X 28 May 2021 10:42:10.223 * +slave slave 211.240.xxx.xxx:6380 211.240.xxx.xxx 6380 @ mymaster 211.240.xxx.xxx 6379
28037:X 28 May 2021 10:42:10.224 * +slave slave 211.240.xxx.xxx:6381 211.240.xxx.xxx 6381 @ mymaster 211.240.xxx.xxx 6379
28037:X 28 May 2021 10:42:15.937 * +sentinel sentinel f6ea7414b5e3daf9292fb9497875af7bba37dcee 211.240.xxx.xxx 7001 @ mymaster 211.240.xxx.xxx 6379
28037:X 28 May 2021 10:42:18.667 * +sentinel sentinel 9e50008fedfd4b884704a4ee6ad54584f598fb87 211.240.xxx.xxx 7002 @ mymaster 211.240.xxx.xxx 6379

---

# Sentinel Conf파일 자동 생성되는 설정 정보들

sentinel_7000 ~ 7002 서버를 시작하면 sentinel_7000.conf, sentinel_7001.conf, sentinel_7002.conf 파일 맨 아래에 부분에 자동으로 설정되는 부분이 생깁니다.

Sentinel conf 파일 설정에서는 Master Redis 정보만 입력했지만 Sentinel이 redis ino 명령어를 통해 slave 정보들을 읽어서 slave 정보들을 자동으로 conf 파일에 추가하고 Sentinel 서버간 내부 통신으로 redis의 Pub/Sub 기능을 통해 다른 Sentinel 서버정보를 conf 파일에 자동으로 추가합니다.

#vim sentinel_7000.conf

.....

//맨아래에 자동으로 생성된 설정 정보

#Generated by CONFIG REWRITE

protected-mode no
user default on nopass ~* &* +@all
sentinel myid bc3291c7c38dd85f2fd58c4453aa194ac21a8c44
sentinel config-epoch mymaster 0
sentinel leader-epoch mymaster 0
sentinel current-epoch 0
sentinel known-replica mymaster 211.240.xxx.xxx 6380         // Slave Redis 정보
sentinel known-replica mymaster 211.240.xxx.xxx 6381         // Slave Redis 정보
sentinel known-sentinel mymaster 211.240.xxx.xxx 7001 f6ea7414b5e3daf9292fb9497875af7bba37dcee  // sentinel 서버정보
sentinel known-sentinel mymaster 211.240.xxx.xxx 7002 9e50008fedfd4b884704a4ee6ad54584f598fb87  // sentinel 서버 정보

#vim sentinel_7001.conf

#Generated by CONFIG REWRITE

protected-mode no
user default on nopass ~* &* +@all
sentinel myid f6ea7414b5e3daf9292fb9497875af7bba37dcee
sentinel config-epoch mymaster 0
sentinel leader-epoch mymaster 0
sentinel current-epoch 0
sentinel known-replica mymaster 211.240.xxx.xxx 6381
sentinel known-replica mymaster 211.240.xxx.xxx 6380
sentinel known-sentinel mymaster 211.240.xxx.xxx 7000 bc3291c7c38dd85f2fd58c4453aa194ac21a8c44
sentinel known-sentinel mymaster 211.240.xxx.xxx 7002 9e50008fedfd4b884704a4ee6ad54584f598fb87

---

# Sentinel 검증

## 현재 상태

현재 Master Redis는 6379이고 Slave Redis는 6380과 6381입니다.

#redis-cli -p 6379 --askpass

Please input password: *********
127.0.0.1:6379> info replication

role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online,offset=160771,lag=1
slave1:ip=127.0.0.1,port=6381,state=online,offset=160771,lag=1
master_failover_state:no-failover
master_replid:2c728a8720bc19ea3bdfceed46f6d41dd78ecb9c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:161035
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:161035

127.0.0.1:6379> quit

---

#redis-cli -p 6380 --askpass
Please input password: *********
127.0.0.1:6380> info replication

role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:183101
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:2c728a8720bc19ea3bdfceed46f6d41dd78ecb9c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:183101
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:183101

127.0.0.1:6380> quit

---

## Master Redis를 중단 시키고 상태를 확인 합니다.

#/etc/init.d/redis_6379 stop

Stopping ...
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
Waiting for Redis to shutdown ...
Redis stopped

---

## Master Redis(6379) log - 중단됨

#vim /usr/local/src/redis/logs/6379.log

27921:M 28 May 2021 10:38:51.950 * Replica 211.240.xxx.xxx:6381 asks for synchronization
27921:M 28 May 2021 10:38:51.950 * Partial resynchronization not accepted: Replication ID mismatch (Replica asked for '818b4519e78e443ee4f9bde17b13496e22b352dc', my replication IDs are '9461bcf55a4c832424c37d610b0edb44da16432b' and '0000000000000000000000000000000000000000')
27921:M 28 May 2021 10:38:51.950 * Starting BGSAVE for SYNC with target: disk
27921:M 28 May 2021 10:38:51.951 * Background saving started by pid 27942
27942:C 28 May 2021 10:38:51.955 * DB saved on disk
27942:C 28 May 2021 10:38:51.955 * RDB: 0 MB of memory used by copy-on-write
27921:M 28 May 2021 10:38:52.016 * Background saving terminated with success
27921:M 28 May 2021 10:38:52.017 * Synchronization with replica 211.240.xxx.xxx:6381 succeeded
27921:M 28 May 2021 10:55:17.228 # User requested shutdown...
27921:M 28 May 2021 10:55:17.228 * Saving the final RDB snapshot before exiting.
27921:M 28 May 2021 10:55:17.230 * DB saved on disk
27921:M 28 May 2021 10:55:17.230 * Removing the pid file.
27921:M 28 May 2021 10:55:17.230 # Redis is now ready to exit, bye bye...

---

## Slave Redis(6380) log - Master Redis와 Sync를 시도하다가 Sentinel이 승격시킴

#vim /usr/local/src/redis/logs/6380.log

27928:S 28 May 2021 10:55:45.586 * MASTER <-> REPLICA sync started
27928:S 28 May 2021 10:55:45.587 # Error condition on socket for SYNC: Connection refused
27928:S 28 May 2021 10:55:46.593 * Connecting to MASTER 211.240.xxx.xxx:6379
27928:S 28 May 2021 10:55:46.593 * MASTER <-> REPLICA sync started
27928:S 28 May 2021 10:55:46.593 # Error condition on socket for SYNC: Connection refused
27928:S 28 May 2021 10:55:47.598 * Connecting to MASTER 211.240.xxx.xxx:6379
27928:S 28 May 2021 10:55:47.599 * MASTER <-> REPLICA sync started
27928:S 28 May 2021 10:55:47.599 # Error condition on socket for SYNC: Connection refused

...

sentinel  conf에서 설정한  sentinel down-after-milliseconds mymaster 50000 정보를 토대로 5초동안 master가 끊겼다고 판단할때까지 위 메시지 반복됨.sentinel 이 master가 중단되었다고 판단하면 slave 중 하나를 선택하여 마스터로 승격시킴.

27928:M 28 May 2021 10:55:47.669 * Discarding previously cached master state.
27928:M 28 May 2021 10:55:47.669 # Setting secondary replication ID to 9461bcf55a4c832424c37d610b0edb44da16432b, valid up to offset: 158082. New replication ID is 7cd2df002f40d6f4f5522fefc15e79cd72240a9a
27928:M 28 May 2021 10:55:47.669 * MASTER MODE enabled (user request from 'id=10 addr=211.240.xxx.xxx:41215 laddr=211.240.xxx.xxx:6380 fd=12 name=sentinel-9e50008f-cmd age=811 idle=0 flags=x db=0 sub=0 psub=0 multi=4 qbuf=188 qbuf-free=40766 argv-mem=4 obl=45 oll=0 omem=0 tot-mem=61468 events=r cmd=exec user=default redir=-1')
27928:M 28 May 2021 10:55:47.674 # CONFIG REWRITE executed with success.
27928:M 28 May 2021 10:55:48.108 * Replica 211.240.xxx.xxx:6381 asks for synchronization
27928:M 28 May 2021 10:55:48.108 * Partial resynchronization request from 211.240.xxx.xxx:6381 accepted. Sending 295 bytes of backlog starting from offset 158082.

---

## Sentinel(7000) log - switch-master를 통해 6379에서 6380 Redis로 Master를 변경함.

#vim /usr/local/src/redis/logs/redis-sentinel_7000.log

28037:X 28 May 2021 10:55:47.325 # +sdown master mymaster 211.240.xxx.xxx 6379
28037:X 28 May 2021 10:55:47.429 # +new-epoch 1
28037:X 28 May 2021 10:55:47.430 # +vote-for-leader 9e50008fedfd4b884704a4ee6ad54584f598fb87 1
28037:X 28 May 2021 10:55:48.104 # +config-update-from sentinel 9e50008fedfd4b884704a4ee6ad54584f598fb87 211.240.xxx.xxx 7002 @ mymaster 211.240.xxx.xxx 6379
28037:X 28 May 2021 10:55:48.104 # +switch-master mymaster 211.240.xxx.xxx 6379 211.240.xxx.xxx 6380
28037:X 28 May 2021 10:55:48.104 * +slave slave 211.240.xxx.xxx:6381 211.240.xxx.xxx 6381 @ mymaster 211.240.xxx.xxx 6380
28037:X 28 May 2021 10:55:48.104 * +slave slave 211.240.xxx.xxx:6379 211.240.xxx.xxx 6379 @ mymaster 211.240.xxx.xxx 6380
28037:X 28 May 2021 10:56:18.138 # +sdown slave 211.240.xxx.xxx:6379 211.240.xxx.xxx 6379 @ mymaster 211.240.xxx.xxx 6380

---

## 자동으로 변경된 Conf파일 정보

6379.conf(구 Master Redis) - 현재 중단된 상태이고, 변경된 정보는 없음

#vim /usr/local/src/redis/conf/6380.conf(현 Master Redis) - Sentinel에 의해 Master로 승격되었으며, conf파일 맨 아래 중에 자동으로 설정정보가 추가됨.

...

#Generated by CONFIG REWRITE

save 3600 1
save 300 100
save 60 10000
user default on #49de38df71bfb36054c27bf55182374a3604cae480fc7132be78c2c505ade710 ~* &* +@all

---

#vim /usr/local/src/redis/conf/6380.conf(현 Slave Redis)

...

#Generated by CONFIG REWRITE

save 3600 1
save 300 100
save 60 10000
user default on #49de38df71bfb36054c27bf55182374a3604cae480fc7132be78c2c505ade710 ~* &* +@all

---

#vim /usr/local/src/redis/conf/sentinel_7000.conf (Sentinel #1)

...

#Generated by CONFIG REWRITE

protected-mode no
user default on nopass ~* &* +@all
sentinel myid bc3291c7c38dd85f2fd58c4453aa194ac21a8c44
sentinel config-epoch mymaster 1
sentinel leader-epoch mymaster 1
sentinel current-epoch 1
sentinel known-replica mymaster 211.240.xxx.xxx 6379
sentinel known-replica mymaster 211.240.xxx.xxx 6381
sentinel known-sentinel mymaster 211.240.xxx.xxx 7001 f6ea7414b5e3daf9292fb9497875af7bba37dcee
sentinel known-sentinel mymaster 211.240.xxx.xxx 7002 9e50008fedfd4b884704a4ee6ad54584f598fb87

---

#vim /usr/local/src/redis/conf/sentinel_7001.conf (Sentinel #2)

...

#Generated by CONFIG REWRITE

protected-mode no
user default on nopass ~* &* +@all
sentinel myid f6ea7414b5e3daf9292fb9497875af7bba37dcee
sentinel config-epoch mymaster 1
sentinel leader-epoch mymaster 1
sentinel current-epoch 1
sentinel known-replica mymaster 211.240.xxx.xxx 6379
sentinel known-replica mymaster 211.240.xxx.xxx 6381
sentinel known-sentinel mymaster 211.240.xxx.xxx 7000 bc3291c7c38dd85f2fd58c4453aa194ac21a8c44
sentinel known-sentinel mymaster 211.240.xxx.xxx 7002 9e50008fedfd4b884704a4ee6ad54584f598fb87

---

#vim /usr/local/src/redis/conf/sentinel_7002.conf (Sentinel #3)

...

#Generated by CONFIG REWRITE

protected-mode no
user default on nopass ~* &* +@all
sentinel myid 9e50008fedfd4b884704a4ee6ad54584f598fb87
sentinel config-epoch mymaster 1
sentinel leader-epoch mymaster 1
sentinel current-epoch 1
sentinel known-replica mymaster 211.240.xxx.xxx 6379
sentinel known-replica mymaster 211.240.xxx.xxx 6381
sentinel known-sentinel mymaster 211.240.xxx.xxx 7001 f6ea7414b5e3daf9292fb9497875af7bba37dcee
sentinel known-sentinel mymaster 211.240.xxx.xxx 7000 bc3291c7c38dd85f2fd58c4453aa194ac21a8c44

---

## 만약 정상 동작하지 않는다면 Conf파일에 자동으로 생성된 설정 값들을 지우고 데몬들을 순차적으로 다시 시작 해볼것.

---

## Redis Info Replication 정보 확인

#redis-cli -p 6379 --askpass

Please input password: *********
Could not connect to Redis at 127.0.0.1:6379: Connection refused

not connected> quit

----------------------------------------------------------

#redis-cli -p 6380 --askpass
Please input password: *********
127.0.0.1:6380> info replication

#Replication

role:master
connected_slaves:1
slave0:ip=211.240.xxx.xxx,port=6381,state=online,offset=478420,lag=1
master_failover_state:no-failover
master_replid:7cd2df002f40d6f4f5522fefc15e79cd72240a9a
master_replid2:9461bcf55a4c832424c37d610b0edb44da16432b
master_repl_offset:478692
second_repl_offset:158082
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:478692

----------------------------------------------------------

#redis-cli -p 6381 --askpass

Please input password: *********
127.0.0.1:6381> info replication

#Replication

role:slave
master_host:211.240.xxx.xxx
master_port:6380
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:513746
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:7cd2df002f40d6f4f5522fefc15e79cd72240a9a
master_replid2:9461bcf55a4c832424c37d610b0edb44da16432b
master_repl_offset:513746
second_repl_offset:158082
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:513746

---

## 이전 Master Redis(6379)를 다시 시작하면 Slave로 등록되고, 현 Master Redis(6380)과 Sync합니다.

테스트 key하나를 현 Master Redis(6380)에 추가 합니다.

#redis-cli -p 6380 --askpass

Please input password: *********
127.0.0.1:6380> set sentinel_test 12345
OK
127.0.0.1:6380> get sentinel_test
"12345"

---

#/etc/init.d/redis_6379 start & tail -f /usr/local/src/redis/logs/6379.log ( Master Redis와 Snyc함)

29827:C 28 May 2021 11:33:38.000 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
29827:C 28 May 2021 11:33:38.000 # Redis version=6.2.3, bits=64, commit=00000000, modified=0, pid=29827, just started
29827:C 28 May 2021 11:33:38.000 # Configuration loaded
29827:M 28 May 2021 11:33:38.002 * Increased maximum number of open files to 10032 (it was originally set to 1024).
29827:M 28 May 2021 11:33:38.002 * monotonic clock: POSIX clock_gettime
29827:M 28 May 2021 11:33:38.003 * Running mode=standalone, port=6379.
29827:M 28 May 2021 11:33:38.003 # Server initialized
29827:M 28 May 2021 11:33:38.004 * Loading RDB produced by version 6.2.3
29827:M 28 May 2021 11:33:38.004 * RDB age 2301 seconds
29827:M 28 May 2021 11:33:38.004 * RDB memory usage when created 2.03 Mb
29827:M 28 May 2021 11:33:38.004 * DB loaded from disk: 0.000 seconds
29827:M 28 May 2021 11:33:38.004 * Ready to accept connections
29827:S 28 May 2021 11:33:48.283 * Before turning into a replica, using my own master parameters to synthesize a cached master: I may be able to synchronize with the new master with just a partial transfer.
29827:S 28 May 2021 11:33:48.283 * Connecting to MASTER 211.240.xxx.xxx:6380
29827:S 28 May 2021 11:33:48.283 * MASTER <-> REPLICA sync started
29827:S 28 May 2021 11:33:48.283 * REPLICAOF 211.240.xxx.xxx:6380 enabled (user request from 'id=3 addr=211.240.xxx.xxx:51756 laddr=211.240.xxx.xxx:6379 fd=7 name=sentinel-9e50008f-cmd age=10 idle=0 flags=x db=0 sub=0 psub=0 multi=4 qbuf=199 qbuf-free=40755 argv-mem=4 obl=45 oll=0 omem=0 tot-mem=61468 events=r cmd=exec user=default redir=-1')
29827:S 28 May 2021 11:33:48.288 # CONFIG REWRITE executed with success.
29827:S 28 May 2021 11:33:48.288 * Non blocking connect for SYNC fired the event.
29827:S 28 May 2021 11:33:48.289 * Master replied to PING, replication can continue...
29827:S 28 May 2021 11:33:48.289 * Trying a partial resynchronization (request e77afac4a6263a356e8b8dc3ec002ac7d162515e:1).
29827:S 28 May 2021 11:33:48.291 * Full resync from master: 7cd2df002f40d6f4f5522fefc15e79cd72240a9a:618150
29827:S 28 May 2021 11:33:48.291 * Discarding previously cached master state.
29827:S 28 May 2021 11:33:48.322 * MASTER <-> REPLICA sync: receiving 467 bytes from master to disk
29827:S 28 May 2021 11:33:48.322 * MASTER <-> REPLICA sync: Flushing old data
29827:S 28 May 2021 11:33:48.322 * MASTER <-> REPLICA sync: Loading DB in memory
29827:S 28 May 2021 11:33:48.323 * Loading RDB produced by version 6.2.3
29827:S 28 May 2021 11:33:48.323 * RDB age 0 seconds
29827:S 28 May 2021 11:33:48.323 * RDB memory usage when created 1.97 Mb
29827:S 28 May 2021 11:33:48.323 * MASTER <-> REPLICA sync: Finished with success

---

#vim  /usr/local/src/redis/logs/6380.log ( 다시 시작된 Slave Redis와 Snyc됨)

27928:M 28 May 2021 11:33:48.289 * Replica 211.240.xxx.xxx:6379 asks for synchronization
27928:M 28 May 2021 11:33:48.289 * Partial resynchronization not accepted: Replication ID mismatch (Replica asked for 'e77afac4a6263a356e8b8dc3ec002ac7d162515e', my replication IDs are '7cd2df002f40d6f4f5522fefc15e79cd72240a9a' and '9461bcf55a4c832424c37d610b0edb44da16432b')
27928:M 28 May 2021 11:33:48.289 * Starting BGSAVE for SYNC with target: disk
27928:M 28 May 2021 11:33:48.290 * Background saving started by pid 29836
29836:C 28 May 2021 11:33:48.293 * DB saved on disk
29836:C 28 May 2021 11:33:48.294 * RDB: 0 MB of memory used by copy-on-write
27928:M 28 May 2021 11:33:48.322 * Background saving terminated with success
27928:M 28 May 2021 11:33:48.322 * Synchronization with replica 211.240.xxx.xxx:6379 succeeded

---

#vim /usr/local/src/redis/logs/redis-sentinel_7000.log ( 6379 Slave Redis를 감지하고 6379 Redis Conf 파일 설정을 추가함)

28037:X 28 May 2021 10:55:47.325 # +sdown master mymaster 211.240.xxx.xxx 6379
28037:X 28 May 2021 10:55:47.429 # +new-epoch 1
28037:X 28 May 2021 10:55:47.430 # +vote-for-leader 9e50008fedfd4b884704a4ee6ad54584f598fb87 1
28037:X 28 May 2021 10:55:48.104 # +config-update-from sentinel 9e50008fedfd4b884704a4ee6ad54584f598fb87 211.240.xxx.xxx 7002 @ mymaster 211.240.xxx.xxx 6379
28037:X 28 May 2021 10:55:48.104 # +switch-master mymaster 211.240.xxx.xxx 6379 211.240.xxx.xxx 6380
28037:X 28 May 2021 10:55:48.104 * +slave slave 211.240.xxx.xxx:6381 211.240.xxx.xxx 6381 @ mymaster 211.240.xxx.xxx 6380
28037:X 28 May 2021 10:55:48.104 * +slave slave 211.240.xxx.xxx:6379 211.240.xxx.xxx 6379 @ mymaster 211.240.xxx.xxx 6380
28037:X 28 May 2021 10:56:18.138 # +sdown slave 211.240.xxx.xxx:6379 211.240.xxx.xxx 6379 @ mymaster 211.240.xxx.xxx 6380
28037:X 28 May 2021 11:33:38.491 # -sdown slave 211.240.xxx.xxx:6379 211.240.xxx.xxx 6379 @ mymaster 211.240.xxx.xxx 6380

---

#vim /usr/local/src/redis/conf/6379.conf ( 맨 아래 자동으로 추가됨)

...

#Generated by CONFIG REWRITE

save 3600 1
save 300 100
save 60 10000
user default on #49de38df71bfb36054c27bf55182374a3604cae480fc7132be78c2c505ade710 ~* &* +@all
replicaof 211.240.xxx.xxx 6380

---

#vim /usr/local/src/redis/conf/6380.conf (Sentinel이 Slave Redis를  Master Redis로 승격시킬때 맨 아래 replicaof 부분이 삭제함)

...

#Generated by CONFIG REWRITE

save 3600 1
save 300 100
save 60 10000
user default on #49de38df71bfb36054c27bf55182374a3604cae480fc7132be78c2c505ade710 ~* &* +@all

---

#redis-cli -p 6379 --askpass

Please input password: *********
127.0.0.1:6379> info replication

role:slave
master_host:211.240.xxx.xxx
master_port:6380
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:752105
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:7cd2df002f40d6f4f5522fefc15e79cd72240a9a
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:752105
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:618151
repl_backlog_histlen:133955

// 테스트 key 값 확인

127.0.0.1:6379> get sentinel_test
"12345"

127.0.0.1:6379> quit

---

#redis-cli -p 6380 --askpass
Please input password: *********
127.0.0.1:6380> info replication

role:master
connected_slaves:2
slave0:ip=211.240.xxx.xxx,port=6381,state=online,offset=767027,lag=0
slave1:ip=211.240.xxx.xxx,port=6379,state=online,offset=767027,lag=1
master_failover_state:no-failover
master_replid:7cd2df002f40d6f4f5522fefc15e79cd72240a9a
master_replid2:9461bcf55a4c832424c37d610b0edb44da16432b
master_repl_offset:767027
second_repl_offset:158082
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:767027

---

# Redis 서버 수신 상태 확인

#ss -an | grep 6379
tcp    LISTEN     0      511       *:6379                  *:*
tcp    TIME-WAIT  0      0      127.0.0.1:59480              127.0.0.1:6379
tcp    ESTAB      0      0      211.240.xxx.xxx:6379               211.240.xxx.xxx:42279
tcp    ESTAB      0      0      211.240.xxx.xxx:42279              211.240.xxx.xxx:6379
tcp    ESTAB      0      0      211.240.xxx.xxx:42356              211.240.xxx.xxx:6379
tcp    ESTAB      0      0      211.240.xxx.xxx:6379               211.240.xxx.xxx:51756
tcp    ESTAB      0      0      211.240.xxx.xxx:55217              211.240.xxx.xxx:6379
tcp    ESTAB      0      0      211.240.xxx.xxx:6379               211.240.xxx.xxx:60046
tcp    ESTAB      0      0      211.240.xxx.xxx:6379               211.240.xxx.xxx:47594
tcp    ESTAB      0      0      211.240.xxx.xxx:6379               211.240.xxx.xxx:55217
tcp    ESTAB      0      0      211.240.xxx.xxx:60046              211.240.xxx.xxx:6379
tcp    ESTAB      0      0      211.240.xxx.xxx:6379               222.xxx.xxx.214:50338
tcp    ESTAB      0      0      211.240.xxx.xxx:6379               222.xxx.xxx.214:50339
tcp    ESTAB      0      0      211.240.xxx.xxx:51756              211.240.xxx.xxx:6379
tcp    ESTAB      0      0      211.240.xxx.xxx:47594              211.240.xxx.xxx:6379
tcp    ESTAB      0      0      211.240.xxx.xxx:6379               211.240.xxx.xxx:42356

---

### 외부접속을 위한 방화벽 작업

방화벽 실행 여부 확인

firewall-cmd --state

---

방화벽 작동중인지 확인

firewall-cmd --state

---

현재 적용중인 zone확인
firewall-cmd --get-default-zone

---

서비스 추가 [ 선택사항, 아래에서 특정 port에 대한 ip 제한안하고 서비스로 구분하여 방화벽 열때 ]
firewall-cmd --zone=public --add-service=redis --permanent

---

서비스 삭제 [ 선택사항, 아래에서 특정 port에 대한 ip 제한 할려고 기존 추가한 service 삭제시]
firewall-cmd --permanent --zone=public --remove-service=redis

---

허용ip추가
firewall-cmd --permanent --zone=public --add-source=106.xxx.xxx.0/24

---

특정 port에 대한 ip 제한
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address=106.xxx.xxx.0/24 port port="7000" protocol="tcp" accept'

sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address=222.xxx.xxx.0/24 port port="7000" protocol="tcp" accept'

sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address=211.240.xxx.xxx/24 port port="7000" protocol="tcp" accept'

sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address=106.xxx.xxx.0/24 port port="7001" protocol="tcp" accept'

sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address=222.xxx.xxx.0/24 port port="7001" protocol="tcp" accept'

sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address=211.240.xxx.xxx/24 port port="7001" protocol="tcp" accept'

sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address=106.xxx.xxx.0/24 port port="7002" protocol="tcp" accept'

sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address=222.xxx.xxx.0/24 port port="7002" protocol="tcp" accept'

sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address=211.240.xxx.xxx/24 port port="7002" protocol="tcp" accept'

---

방화벽 재시작
firewall-cmd --reload

---

설정확인
firewall-cmd --list-all

---

# 내컴퓨터에서 원격 Sentinel 서버 접속 여부 체크

Sentinel 체크시

C:\Program Files\Redis>redis-cli -h 211.240.xxx.xxx -p 7003

211.240.xxx.xxx:7000> info sentinel

sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=211.240.xxx.xxx:6379,slaves=2,sentinels=3
211.240.xxx.xxx:7000>

.....

---

# 내컴퓨터에서 원격 Redis 서버 접속 여부 체크

C:\Program Files\Redis>redis-cli -h 211.240.xxx.xxx -p 6379

211.240.xxx.xxx:6379> AUTH test1234

211.240.xxx.xxx:6379> keys *
1) "spring:session:index:org.springframework.session.FindByIndexNameSessionRepository.PRINCIPAL_NAME_INDEX_NAME:kdrco"
2) "spring:session:sessions:expires:4544302f-778c-446c-8ee8-e56470cf1054"
3) "spring:session:expirations:1622183220000"
4) "spring:session:sessions:4544302f-778c-446c-8ee8-e56470cf1054"
5) "replica_test"
6) "menusCache::POSTAL_MENU"
7) "sentinel_test"

---

# logrotate로 로그 파일 일자별로 생성하도록 설정하기

logrotate는 로그 파일의 회전, 압축, 삭제 및 메일링 등 자동으로 수행할 수 있는 기능입니다. 각 로그 파일을 매일, 매주, 매월 또는 size 별 처리할 수 있도록 logrotate를 구성할 수 있습니다.

/etc/logrotate.conf은 logrotate의 기본 구성 파일이고, /etc/logrotate.d/ 는 특정 서비스 별로 설정할 수 있는 디렉토리입니다.

---

주요 설정 옵션 

yearly // 매년

monthly // 매월

weekly // 매주

daily // 매일

rotate 5 //  log 파일이 5개가 되면 첫번째 생성된 log 파일 삭제 후 생성.

dateext // 로그 파일에 날짜 부여

compress    // 로그파일 압축여부 

daily,weekly,monthly // 로그파일분할 시간지정. 

delaycompress     // 최근파일외 전부압축 

errors "emailid"  // 에러발생시 메일발솔여부 지정.

missingok         //  로그파일분실시 에러로 인정안함. 

notifempty       // 로그파일이 제로인경우 분할하지않음. 

olddir "dir"      // 분할된 로그파일을 지정한 디렉토리에 저장함. 

postrotate      // 로그파일분할후 실행할 script지정. 

prerotate       // 로그파일분할전 실행할 script지정. 

rotate 'n'      // 저장할 로그파일개수 지정 

sharedscripts   //  한세트의 로그파일에 한번만 로그분할작업 실행. 

size='logsize'   // 로그파일이 지정된 logsize보다 클떄만 분할

---

Redis 서버 별 로그 설정

#cd /etc/logrotate.d

#vim redis-6379

/usr/local/src/redis/logs/6379.log {
    daily
    rotate 7
    copytruncate
    delaycompress
    compress
    dateext
    notifempty
    missingok
}

#cp redis-6379 ./redis-6380

#vim redis-6380

/usr/local/src/redis/logs/6380.log {
    daily
    rotate 7
    copytruncate
    delaycompress
    compress
    dateext
    notifempty
    missingok
}

#cp redis-6380 ./redis-6381

#vim redis-6381

/usr/local/src/redis/logs/6381.log {
    daily
    rotate 7
    copytruncate
    delaycompress
    compress
    dateext
    notifempty
    missingok
}

위 형식대로 sentinel 로그도 생성해줍니다.

#cp redis-6380 ./redis-sentinel_7000

#cp redis-sentinel_7000 ./redis-sentinel_7001

#cp redis-sentinel_7000 ./redis-sentinel_7002

---

logrotate 실행 : 기본 값은 새벽 4시에 자동으로 실행 되며, 시간대를 변경하려면 /etc/cron.daily/에 있는 logrotate파일을 다른 이동시키고, 름은 logrotate.sh로 변경, etc/cron.d/ 아래 하나의 파일을 하나 생성하여, 그안에 스크립트를 작성해 주면 된다.

0 0 * * * root /usr/logrotate/logrotate.sh 

 지금 강제 실행은 아래 명령으로 하면 됩니다. 

#/usr/sbin/logrotate  -vf /etc/logrotate.conf

reading config file /etc/logrotate.conf
including /etc/logrotate.d
reading config file bootlog
reading config file chrony
reading config file firewalld
reading config file redis-6379
reading config file redis-6380
reading config file redis-6381
reading config file redis-sentinel_7000
reading config file redis-sentinel_7001
reading config file redis-sentinel_7002
reading config file syslog
reading config file wpa_supplicant
reading config file yum
Allocating hash table for state file, size 15360 B

Handling 14 logs

...

rotating pattern: /usr/local/src/redis/logs/6379.log  forced from command line (7 rotations)
empty log files are not rotated, old logs are removed
considering log /usr/local/src/redis/logs/6379.log
  log needs rotating
rotating log /usr/local/src/redis/logs/6379.log, log->rotateCount is 7
dateext suffix '-20210531'
glob pattern '-[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]'
glob finding logs to compress failed
glob finding old rotated logs failed
copying /usr/local/src/redis/logs/6379.log to /usr/local/src/redis/logs/6379.log-20210531
set default create context to unconfined_u:object_r:usr_t:s0
truncating /usr/local/src/redis/logs/6379.log

....

#ls -lah /usr/local/src/redis/logs
합계 4.9M
drwxr-xr-x.  2 root root 4.0K  5월 31 13:55 .
drwxrwxr-x. 12 root root 4.0K  5월 28 11:34 ..
-rw-r--r--.  1 root root    0  5월 31 13:55 6379.log
-rw-r--r--.  1 root root  352  5월 31 13:55 6379.log-20210531
-rw-r--r--.  1 root root    0  5월 31 13:55 6380.log
-rw-r--r--.  1 root root 602K  5월 31 13:55 6380.log-20210531
-rw-r--r--.  1 root root    0  5월 31 13:55 6381.log
-rw-r--r--.  1 root root 598K  5월 31 13:55 6381.log-20210531
-rw-r--r--.  1 root root    0  5월 27 10:12 AUTH
-rw-r--r--.  1 root root    0  5월 27 10:12 info
-rw-r--r--.  1 root root    0  5월 31 13:55 redis-sentinel_7000.log
-rw-r--r--.  1 root root 1.3M  5월 31 13:55 redis-sentinel_7000.log-20210531
-rw-r--r--.  1 root root    0  5월 31 13:55 redis-sentinel_7001.log
-rw-r--r--.  1 root root 1.3M  5월 31 13:55 redis-sentinel_7001.log-20210531
-rw-r--r--.  1 root root    0  5월 31 13:55 redis-sentinel_7002.log
-rw-r--r--.  1 root root 1.3M  5월 31 13:55 redis-sentinel_7002.log-20210531



---

# 모니터링 - Redis 리소스를 모니터링하기 위함.

redis-stat ( [https://github.com/junegunn/redis-stat](https://github.com/junegunn/redis-stat) )



#yum install gcc-c++ patch readline \  readline-devel zlib zlib-devel \  libffi-devel openssl-devel make \   bzip2 autoconf automake libtool \  bison sqlite-devel -y

#gpg2 --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

#curl -sSL https://get.rvm.io | bash -s stable

#source /etc/profile.d/rvm.sh 

#rvm reload

#rvm install 2.5

#rvm list

#rvm use 2.4.10 --default

#rvm info  

#rvm rubygems current (rvm rubygems remove)

#ruby --version

#gem update --system

#bundle update

#ruby -v



#yum install git

#git clone https://github.com/junegunn/redis-stat.git

#cd redis-stat

#gem install redis-stat

#gem -v

#gem list | grep redis-stat

---

# Spring jedis보다 lettuce를 써야되는 이유

여러 쓰레드에서 단일 jedis 인스턴스를 공유하려 할때 jedis는 쓰레드에 안전하지 않다. 멀티쓰레드환경에서 고려해야할 상황이 따른다.

안전한 방법은 pooling(Thread-pool)과 같은 jedis-pool을 사용하는 것이지만 물리적인 비용의(connection할 인스턴스를 미리 만들어놓고 대기하는 연결비용의 증가) 증가가 따른다.

반면 lettuce는 netty 라이브러리 위에서 구축되었고, connection 인스턴스((StatefulRedisConnection))를 여러 쓰레드에서 공유가 가능하다.(Thread-safe)

=> 혼동할 수 있는데, connection 인스턴스의 공유라는 점에서 Thread-safe를 논점을 둔 점이고 Single-Thread의 레디스에 데이터에 접근할 때는 또다르게 고려해야 할 점이다.

```
redis:
  host: 211.240.xxx.xxx
  port: 7000
  connect-timeout: 10s
  password: test1234
  sentinel:
    master: mymaster
    nodes:
      - 211.240.xxx.xxx:7000
      - 211.240.xxx.xxx:7001
      - 211.240.xxx.xxx:7002
  lettuce:
    shutdown-timeout: 500ms
```



```java
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 1800)
public class SessionConfig extends AbstractHttpSessionApplicationInitializer {
    private final RedisProperties redisProperties;

    public SessionConfig(RedisProperties redisProperties) {
        super(SessionConfig.class);
        this.redisProperties = redisProperties;
    }

    @Bean
    public RedisConnectionFactory connectionFactory() {

        RedisSentinelConfiguration sentinelConfiguration = new RedisSentinelConfiguration()
                .master(redisProperties.getSentinel().getMaster());
        redisProperties.getSentinel().getNodes().forEach(s -> {
            String[] ipSet = s.split(":");
            sentinelConfiguration.sentinel(ipSet[0], Integer.parseInt(ipSet[1]));
        });
        sentinelConfiguration.setPassword(redisProperties.getPassword().isEmpty() ? RedisPassword.none() : RedisPassword.of(redisProperties.getPassword()));

        return new LettuceConnectionFactory(sentinelConfiguration, LettuceClientConfiguration.defaultConfiguration());

    }
}

```

참고:

https://dltlabs.medium.com/how-to-setup-redis-replication-c9cc89ba6c03

https://medium.com/trendyol-tech/high-availability-with-redis-sentinel-and-spring-lettuce-client-9da40525fc82

https://crystalcube.co.kr/176

https://jojoldu.tistory.com/418