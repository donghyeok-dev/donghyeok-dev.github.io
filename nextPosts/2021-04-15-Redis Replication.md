---
layout: customPost
title:  "Redis Replication"
categories: 
  - Session
  - Tools
---



## Redis Replication



replication 과정에서 fork가 발생하므로 메모리 부족이 발생할 수 있다.

redis-cli-rdb 명령은 현재 상태의 메모리 스냅샷을 가져오므로 메모리 부족이 발생할 수 있음.

AWS나 클라우드의 Redis는 조금 다르게 구현되어 있어서 해당 부분이 안정적으로 동작하지만 속도는 조금 느림.

많은 대수의 Redis 서버가 Replica를 두고 있다면 Network 이슈나 사람의 작업으로 동시에 replication이 재시도 되도록 하면 문제가 발생할 수 있음.

네트워크 안에서 많은 Redis 서버들이 리플리케이션을 동시에 재시작하면 Network band width보다 큰 트래픽이 발생하여 정상 동작하지 않을 수 있기 때문에 

Redis 서버를 순차적으로 작동시켜야함.



Maxclient: 최대 접속 클라이언트 수 50000

Redis 장애 발생 원인 중 가장 큰 이유가 Keys와 Save 설정을 사용해서 발생하기 때문에 보통 Master는 RDB/AOF 설정  가능하면 off하고,

Slave 쪽에만 RDB/AOF 설정을 사용합니다.

AWS의 ElasticCache는 Keys 등을 사용할 수 없음.



Redis 데이터 분산 방법

- Application : Consistent Hasing(실제 균등하게 나누지는 않음), Range Sharding

- Redis Cluster

  ​	장점; 자체적인 primary, secondary FailOver, Slot 단위의 데이터 관리

  ​	단점: 메모리 사용량이 더 많고, Migration 자체는 관리자가 결정해야하며, 언어에서 지원하지 않으면 Library를 직접 구현해야 됨.

![image-20210525172755280](C:\Users\webme\mygit\blog\assets\images\posts\image-20210525172755280.png)





Redis FailOver

VIP, 

DNS, (java의 경우 DNS 캐싱 기능이 있기 때문에 유의)

Cluster



모니터링

Redis Info를 통한 정보 

- RSS (피지컬 메모리를 얼마나 사용하고 있는가?)
- Used Memory (OS 메모리)
- Connection 수 
- 초당 처리 요청 수 (CPU 종속적)

System

- CPU
- Disk
- Network rx/tx



CPU가 100%일 경우

- 처리량이 많은 경우: 좀더 좋은 성능 CPU 서버로 이전.

- O(N) 특정 명령이 많은 경우 : Monitor 명령을 통해 특정 패턴을 파악해야 되고,

  Monitor 명령도 서버에 부하를 줄 수 있기 때문에 짧게 쓰는 것이 좋음.



메모리 기반이므로 빠른 응답속도를 자랑하고, 메모리 사용량의 75%를 넘어 서면 서버 증설을 고려하는 것이 좋음.

Persistent Store의 경우 무조건 Master/Slave 구조로 구성해야되고, 정기적으로 Slave 서버에서만 백업.

RDB보단 AOF가 좀더 안정적임.