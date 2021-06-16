---
layout: customPost
title:  "로컬 Tomcat SSL 테스트하기"
categories: 
  - Tomcat
  - SSL
---
tomcat 7, tomcat 8.5

**openssl genrsa -aes256 -out localhost_private.key 2048**

**openssl req -new -key localhost_private.key -out localhost.csr** 

**openssl x509 -req -days 1825 -extensions v3_user -in localhost.csr -CA rootca.crt -CAcreateserial -CAkey rootca_private.key -out localhost.crt**



**openssl pkcs12 -export -in localhost.crt -inkey localhost_private.key -out keystore -name "localhost cert"**

keytool -importkeystore -srckeystore keystore -srcstoretype pkcs12 -destkeystore keystore.jks -deststoretype jks



<Connector SSLEnabled="true" clientAuth="false"
    connectionTimeout="20000"
    keystoreFile="C:\Users\webme\eclipse-workspace\Servers\onetouchpay-home-config\keystore.jks"
    keystorePass="test1234"
    port="443"
    protocol="org.apache.coyote.http11.Http11Protocol"
    scheme="https" secure="true" sslProtocol="TLS"/>
