---
layout: customPost
title:  "tomcat7_spring3.2_atomikos 분산트랜잭션 적용"
categories: 
  - Spring
  - Test
---

##  tomcat7_spring3.2_atomikos 분산트랜잭션 적용

**자바에서 분산 트랜잭션 처리**
  \- 분산 트랜잭션 서비스를 제공해 주는 트랜잭션 관리자가 필요.
  \- WebLogic이나 JBoss 같은 컨테이너는 자체적으로 분산 트랜잭션 서비스를 지원.
  \- 톰캣과 같은 서블릿 컨테이너는 분산 트랜잭션을 지원하고 있지 않음.

톰캣과 같은 서블릿 컨테이너의 분산 트랜잭션을 하기위한 트랜잭션 관리자가 필요한데,  atomikos, JOTM 등과 같은 오픈 소스 트랜잭션 라이브러리가 있음.



한 개 이상의 DB나 JMS의 작업을 하나의 트랜잭션 안에서 동작하게 하려면 서버가 제공하는 트랜잭션 매니저를 JTA를 통해 사용해야 한다. 스프링에서는 서버에 설정해둔 XA DataSource와 트랜잭션 매니저 그리고 UserTransaction 등을 JNDI를 통해 가져와 모든 데이터 액세스 기술에서 사용할 수 있다. JTA와 분산/글로벌 트랜잭션을 사용하기 위한 설정은 자바 서버마다 다르므로 해당 서버의 매뉴얼을 참고해서 등록하는 방법을 알아둬야 한다.

JtaTransactionManager를 빈으로 등록한다. JtaTransactionManager는 여타 트랜잭션 매니저와는 다르게 프로퍼티로 DataSource나 SessionFactory 등의 빈을 참조하지 않는다. 대신 서버에 등록된 트랜잭션 매니저를 가져와 JTA를 이용해서 트랜잭션을 관리해줄 뿐이다.

JTA는 WAS가 제공하는 서비스를 이용하는 경우가 일반적이지만, 원한다면 서버의 지원 없이도 애플리케이션 안에 JTA 서비스 기능을 내장하는 독립형 JTA 방식으로 이용할 수 있다. 이 방식을 사용하면 JTA를 지원하는 WAS가 아닌 톰캣과 같은 서블릿 컨테이너에서도 JTA 기능을 이용하는 것이 가능하다. 서버에 포함돼서 서비스로 동작하는 것은 아니지만 JTA의 자중 트랜잭션 리소스를 위한 글로벌 트랜잭션 기능을 활용할 수 있다. 스프링 안에서 간단히 설정을 추가하는 것만으로 JTA의 기능을 사용할 수 있다는 점에서 매력적이다.

독립형 JTA 트랜잭션 매니저는 ObjectWeb의 JTA 엔진인 JOTM(http://jotm.objectweb.org) 와 Atomikos의 TransactionalEssentials(http://www.atomikos.com) 가 대표적이다. 두 가지 모두 오픈소스 제품이므로 자유롭게 가져다 쓸 수 있다. Atomikos에서는 오픈소스외에도 고급 기능을 가진 상용 제품인 ExtremeTransactions 를 판매하기도 한다.

이 두 가지 모두 스프링의 JtaTransactionManager와 결합해서 JTA 트랜잭션 서비스로 사용할 수 있다.



또한 각 DBMS서버에 XA설정이 필요한데 예를들어 SQL Server의 경우에는 

1. 로컬 DTC설정

```
 a. 시작 > 제어판 > 관리도구 > 구성 요소 서비스

  네비게이션에서 콘솔 루트 > 구성 요소 서비스 > 컴퓨터 > 내 컴퓨터 > MSDTC > Distributed Transaction Coordinator > 로컬 DTC

  b. 로컬 DTC 오른쪽 마우스 클릭 후 속성 > 보안 탭

  c. 보안 설정 섹션 내에 네트워크 DTC 액세스 체크, XA 트랜잭션 상용 체크, SNA LU 6.2 트랜잭션 사용 체크

  d. 클라이언트 및 관리 섹션 내에 원격 클라이언트 허용 체크, 원격 관리 허용 체크

  e. 트랜잭션 관리자 통신 섹션 내에 인바운드 허용 체크, 아웃바운드 허용 체크, 인증 필요 없음 선택

  f. OK 클릭
```

![image-20210331115804142](C:\Users\webme\mygit\blog\assets\images\posts\image-20210331115804142.png)



https://docs.microsoft.com/ko-kr/sql/connect/jdbc/understanding-xa-transactions?view=sql-server-ver15

https://docs.microsoft.com/ko-kr/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-ver15

1. sqljdbc를 다운받아서 압축을 풀고, 폴더 내에

   sqljdbc_9.2\kor\xa\x64\sqljdbc_xa.dll을 mssql 설치 경로 Binn 폴더에 넣어주고

![image-20210331120357882](C:\Users\webme\mygit\blog\assets\images\posts\image-20210331120357882.png)

sqljdbc_9.2\kor\xa\xa_install.sql 스크립트를 master 데이터베이스에서 실행해야 함.

![image-20210331115659232](C:\Users\webme\mygit\blog\assets\images\posts\image-20210331115659232.png)



================================ spring 설정 ======================================

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
...
...
	<!-- https://mvnrepository.com/artifact/com.atomikos/transactions-jta -->
    <dependency>
        <groupId>com.atomikos</groupId>
        <artifactId>transactions-jta</artifactId>
        <version>3.9.3</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/com.atomikos/transactions-jdbc -->
    <dependency>
        <groupId>com.atomikos</groupId>
        <artifactId>transactions-jdbc</artifactId>
        <version>3.9.3</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/javax.transaction/jta -->
    <dependency>
        <groupId>javax.transaction</groupId>
        <artifactId>jta</artifactId>
        <version>1.1</version>
    </dependency>
```

dataSource.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
<context:property-placeholder location="/WEB-INF/config/jdbc.properties" />
    
<bean id="dataSourceSpied" class="oracle.jdbc.xa.client.OracleXADataSource">
        <property name="URL" value="${jdbc.url}" />
        <property name="user" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
    </bean>

    <bean id="dataSourceSpied2" class="com.microsoft.sqlserver.jdbc.SQLServerXADataSource">
        <property name="URL" value="${jdbc.urlMSSql}" />
        <property name="user" value="${jdbc.usernameMSSql}" />
        <property name="password" value="${jdbc.passwordMSSql}" />
    </bean>

    <bean id="dataSourceSpiedDRVPS" class="com.microsoft.sqlserver.jdbc.SQLServerXADataSource">
        <property name="URL" value="${jdbc.urlMSSqlDRVPS}" />
        <property name="user" value="${jdbc.usernameMSSqlDRVPS}" />
        <property name="password" value="${jdbc.passwordMSSqlDRVPS}" />
    </bean>

<!-- 분산 트랜잭션을 위한 Atomikos 설정 -->
	 <bean id="dataSourceXA-oracle" class="com.atomikos.jdbc.AtomikosDataSourceBean" init-method="init" destroy-method="close">
        <property name="uniqueResourceName" value="XAOracle" />
        <property name="xaDataSource" ref="dataSourceSpied"/>
    </bean>

    <bean id="dataSourceXA-sqlserver-receipt" class="com.atomikos.jdbc.AtomikosDataSourceBean" init-method="init" destroy-method="close">
        <property name="uniqueResourceName" value="XASqlserverReceipt" />
        <property name="xaDataSource" ref="dataSourceSpied2"/>
    </bean>

    <bean id="dataSourceXA-sqlserver-DRVPS" class="com.atomikos.jdbc.AtomikosDataSourceBean" init-method="init" destroy-method="close">
        <property name="uniqueResourceName" value="XASqlserverDRVPS" />
        <property name="xaDataSource" ref="dataSourceSpiedDRVPS"/>
    </bean>

<!-- SqlSessionFactoryBean 설정 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSourceXA-oracle" />
        <property name="mapperLocations" value="classpath*:org/dhmng/**/dao/sql/*Ora.xml" />
    </bean>

	<bean id="sqlSessionFactory2" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSourceXA-sqlserver-receipt" />
        <property name="mapperLocations" value="classpath*:org/dhmng/**/dao/sql/*Ora.xml" />
    </bean>

	<bean id="sqlSessionFactoryDRVPS" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSourceXA-sqlserver-DRVPS" />
        <property name="mapperLocations" value="classpath*:org/dhmng/**/dao/sql/*Ora.xml" />
    </bean>

<!-- SqlSessionTemplate 설정 -->
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory" />
    </bean>
<bean id="sqlSessionMSSql" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory2" />
    </bean>
<bean id="sqlSessionDRVPS" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactoryDRVPS" />
    </bean>
    
</beans>
```



jdbc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

 	<!--  분산 트랜잭션 설정 -->
 	<bean id="userTransactionService" class="com.atomikos.icatch.config.UserTransactionServiceImp" init-method="init" destroy-method="shutdownForce">
		<constructor-arg>
			<props>
				<prop key="com.atomikos.icatch.service">com.atomikos.icatch.standalone.UserTransactionServiceFactory</prop>
				<prop key="com.atomikos.icatch.log_base_name">your_project_name_log</prop>
		        <prop key="com.atomikos.icatch.output_dir">/MyDhMng/webapp/WEB-INF/spring/atomikos_log/</prop>
		        <prop key="com.atomikos.icatch.log_base_dir">/MyDhMng/webapp/WEB-INF/spring/atomikos_log/</prop>
			</props>
		</constructor-arg>
	</bean>

	<bean id="atomikosTransactionManager" class="com.atomikos.icatch.jta.UserTransactionManager" init-method="init" destroy-method="close" depends-on="userTransactionService">
		<property name="startupTransactionService" value="false" />
		<property name="forceShutdown" value="false" />
	</bean>

	<bean id="atomikosUserTransaction" class="com.atomikos.icatch.jta.UserTransactionImp" depends-on="userTransactionService">
		<property name="transactionTimeout" value="300" />
	</bean>

	<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager">
		<property name="transactionManager" ref="atomikosTransactionManager"/>
		<property name="userTransaction" ref="atomikosUserTransaction"/>
	</bean>

	<tx:annotation-driven transaction-manager="transactionManager" />

    <context:annotation-config />
</beans>
```





