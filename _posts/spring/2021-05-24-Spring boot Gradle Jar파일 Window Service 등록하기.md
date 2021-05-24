---
layout: customPost
title:  "Spring boot Gradle Jar파일 Window Service 등록하기"
categories: 
  - deploy
  - Spring
---
## Spring boot Gradle Jar파일 Window Service 등록하기

Gradle - build - bootJar을 실행합니다.

![image-20210524101516040](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524101516040.png)



![image-20210524101630018](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524101630018.png)



프로젝트 탭에서 build - libs 아래 jar 파일이 생성됩니다.

![image-20210524101744808](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524101744808.png)



콘솔창에서 java -jar jar파일명으로 실행합니다.

![image-20210524102019852](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524102019852.png)



jar 파일 Window service 등록하기

NSSM 다운로드

[https://nssm.cc/download](https://nssm.cc/download)



jar 파일 폴더 만들기

<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524143641754.png" alt="image-20210524143641754"  />



다운로드 파일을 해당 폴더에 압축을 풀고 

logs 폴더를 생성합니다.

cmd 콘솔을 열고 해당 경로로 이동한 다음 서비스를 등록합니다.

cd C:\Users\Administrator\Desktop\webservice\nssm-2.24\win64

nssm install PostalService

팝업 하나가 열립니다.

![image-20210524143952648](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524143952648.png)



Application탭

Path: java 설치 경로의 java.exe를 선택

Startup directory: jar 파일 경로

Arguments -server -jar 파일명.jar   (추가로 jvm 옵션도 줄 수 있습니다.)

![image-20210524144120479](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524144120479.png)

IO탭

Output :  위에서 만든 logs 폴더 경로를 기입합니다.

Error  : Oupt 경로 입력시 자동으로 입력됩니다.

![image-20210524144216304](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524144216304.png)



Install Service를 하면 서비스가 등록되며 실행시 logs 경로에 로그 정보가 출력됩니다.

![image-20210524144433159](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524144433159.png)

![image-20210524144524187](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210524144524187.png)