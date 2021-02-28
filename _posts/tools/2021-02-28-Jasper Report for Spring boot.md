---
layout: customPost
title:  "JasperReports for Spring Boot"
categories: 
  - Tools
  - Spring
  - JasperReports
 
---
## JasperReports for Spring Boot

- 목표

  - JasperReports  기능을 Spring Boot 프로젝트에 적용.

- JasperReports 선택 이유

  - LGPL 3.0 라이센스로 무료로 상업적으로도 이용 가능.
  
  - Birt Report도 고려하였으나 버전업, 버그 피드백이 느림.
  
  - Birt All-in-One를 설치하였으나 오류발생으로 실행이 안됨. 
  
    (eclipse.ini에 vm으로 jdk 7, 8버전 + add-modules=all-system  설정해 보았지만 실행불가 )

- 개발환경

  - Windows 10 pro
  
  - AdoptOpenJDK 11
  - Eclipse IDE for Enterprise Java Developers Version: 2020-12 (4.18.0)

- Jaspersoft Sutdio

  - JasperReports 용 보고서 템플릿을 설계 및 실행하고, 보고서 쿼리를 작성하고, 복잡한 식을 작성하고, 50개 이상의 차트, 지도, 표, 크로스 탭, 사용자 지정 시각화 등의 레이아웃 구성 요소를 설계 및 실행.
  
  - 이 제품은 커뮤니티와 프로페셔널의 두 가지 버전으로 제공됩니다. Professional 에디션에는 추가 기능, 지도, 고급 HTML5 차트 및 전문가 지원이 포함. (커뮤니티만 무료)
  
  - 동적 HTML로 PDF 인쇄 고품질 PowerPoint, RTF, Word, 스프레드시트 문서 또는 원시 CSV, JSON 또는 XML. 필요에 따라 사용자 지정 내보내기 기능.
  
  - 다양한 유형의 데이터 소스, 빅 데이터, CSV, 최대 절전 모드, Jaspersoft Domain, JavaBeans, JDBC, JSON, NoSQL, XML 또는 사용자 지정 데이터 원본에 액세스.

- Jaspersoft Sutdio 설치 및 jrxml만들기

  jaspersoft 홈페이지에서 다운로드 받아서 설치하거나 이클립스 경우 플러그인으로 설치 가능합니다. 

  [jaspersoft 홈페이지에서 studio를 다운로드 (회원가입이 필요함)](https://community.jaspersoft.com/project/jaspersoft-studio/releases){:target="_blank"}

  

  ![image-20210227002430657](/assets/images/posts/image-20210227002430657.png)

  

  windows_x86_64.exe를 다운 받아서 설치 합니다.

  ![image-20210227004228839](/assets/images/posts/image-20210227004228839.png)

  Agree -> next -> finish하면 설치가 완료되면서 Jaspersoft studio가 실행 됩니다.

  ![image-20210227004439054](/assets/images/posts/image-20210227004439054.png)

  

  실행 첫 화면에 Get Stated를 클릭합니다. (Learn More을 눌러서 내용을 살펴 보는것도 좋습니다.)

  ![image-20210227004555686](/assets/images/posts/image-20210227004555686.png)

  

  메인 화면이 보입니다.

  ![image-20210227004624453](/assets/images/posts/image-20210227004624453.png)

  

  일단 뭐라도 만들어보죠 New -Jasper Report를 선택하고 Next.

  ![image-20210227010134330](/assets/images/posts/image-20210227010134330.png)

  기본적으로 템플릿을 제공해주는데요. 원하는 템플릿을 선택하고 Next 합니다. (여기서는 Blank A4를 선택)

![image-20210227100435836](/assets/images/posts/image-20210227100435836.png)

​		Jrxml 파일명을 적고 Next

​		![image-20210227130331204](/assets/images/posts/image-20210227130331204.png)

​		Report에서 사용할 데이터를 가져올 Data Adater를 선택합니다. 

​		Sample DB로 간단하게 테스트 해보실 수도 있습니다.

​		![image-20210227101412019](/assets/images/posts/image-20210227101412019.png)

![image-20210227095246096](/assets/images/posts/image-20210227095246096.png)

​		![image-20210227103839590](/assets/images/posts/image-20210227103839590.png)

​		그외에 New를 눌러보면 다양한 타입의 Data Adater를 선택할 수 있습니다.

​		![image-20210227095407329](/assets/images/posts/image-20210227095407329.png)

 		저는 Empty Record로 진행합니다.

​		![image-20210227220354100](/assets/images/posts/image-20210227220354100.png)

​		간단한 리포트가 생성되었습니다.

![image-20210227221346614](/assets/images/posts/image-20210227221346614.png)

제목을 아래의 순서로 만듭니다.

① 좌측 상단 Palette에서 Static Text Element를 Title 영역으로 드래그&드롭.

② Static Text Element를  해당 영역의 크기로 늘인다.

③ 원하는 스타일을 지정한다.

![image-20210227222735623](/assets/images/posts/image-20210227222735623.png)

Page Header 영역에 StaticText,  Text Field 각각 한개씩 올려 놓습니다.

(StaticText는 Label개념으로 Text Field는 Input으로 생각합니다.)

![image-20210227223147030](/assets/images/posts/image-20210227223147030.png)

StaticText는  총 사용자 수로, Text Field의 Expression은 $P{userTotalCount} 로 입력합니다.

[Expression 상세보기](https://community.jaspersoft.com/documentation/tibco-jaspersoft-studio-user-guide/v60/expressions){:target="_blank"}

![image-20210227225918045](/assets/images/posts/image-20210227225918045.png)

Palette에서 Table을 Detail 영역으로 드래그&드롭 하면 wizard 창이 열립니다.

![image-20210228014158677](/assets/images/posts/image-20210228014158677.png)

Next하면 상세한 옵션들을 선택할 수 있지만 저는 xml로 Setting할 예정이라 여기서 finish합니다.

(사실 화면에 그릴 필요도 없지만,  wizard로 table 생성하는 것이 편리하기 때문에 그런 방법이 있다라고 알려 드리기 위해 작성했습니다.)

![image-20210228014459290](/assets/images/posts/image-20210228014459290.png)

그럼 Detail 영역에 테이블 하나가 생성됩니다. 

![image-20210228014651897](/assets/images/posts/image-20210228014651897.png)

화면 아래 Source탭을 클릭하여 아래 코드를 참고하여 edit하거나 Copy&Paste합니다.

![image-20210228015528306](/assets/images/posts/image-20210228015528306.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Created with Jaspersoft Studio version 6.16.0.final using JasperReports Library version 6.16.0-48579d909b7943b64690c65c71e07e0b80981928  -->
<jasperReport xmlns="http://jasperreports.sourceforge.net/jasperreports" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://jasperreports.sourceforge.net/jasperreports http://jasperreports.sourceforge.net/xsd/jasperreport.xsd" name="users" pageWidth="595" pageHeight="842" columnWidth="555" leftMargin="20" rightMargin="20" topMargin="20" bottomMargin="20" uuid="fa3d03d8-b228-427a-8ae8-46795765233c">
	
	<style name="defaultStyle" isDefault="true" fontName="nanum-gothic" fontSize="12" />
	
	
	<style name="Table_TH" mode="Opaque" backcolor="#808080">
		<box>
			<pen lineWidth="0.5" lineColor="#000000"/>
			<topPen lineWidth="0.5" lineColor="#000000"/>
			<leftPen lineWidth="0.5" lineColor="#000000"/>
			<bottomPen lineWidth="0.5" lineColor="#000000"/>
			<rightPen lineWidth="0.5" lineColor="#000000"/>
		</box>
	</style>
	<style name="Table_CH" mode="Opaque" backcolor="#EDEDED">
		<box>
			<pen lineWidth="0.5" lineColor="#000000"/>
			<topPen lineWidth="0.5" lineColor="#000000"/>
			<leftPen lineWidth="0.5" lineColor="#000000"/>
			<bottomPen lineWidth="0.5" lineColor="#000000"/>
			<rightPen lineWidth="0.5" lineColor="#000000"/>
		</box>
	</style>
	<style name="Table_TD" mode="Opaque" backcolor="#FFFFFF">
		<box>
			<pen lineWidth="0.5" lineColor="#000000"/>
			<topPen lineWidth="0.5" lineColor="#000000"/>
			<leftPen lineWidth="0.5" lineColor="#000000"/>
			<bottomPen lineWidth="0.5" lineColor="#000000"/>
			<rightPen lineWidth="0.5" lineColor="#000000"/>
		</box>
	</style>
	
	<subDataset name="Dataset1" uuid="0658c9f6-3c7a-493a-87af-a35d392d3de3">
		<field name="userId" class="java.lang.String"/>
		<field name="userName" class="java.lang.String"/>
		<field name="address" class="java.lang.String"/>
	</subDataset>
	
	<parameter name="userTotalCount" class="java.lang.String"/>
	<parameter name="datasource1" class="java.util.List"/>
	
	<background>
		<band splitType="Stretch"/>
	</background>
	<title>
		<band height="79" splitType="Stretch">
			<staticText>
				<reportElement x="0" y="0" width="555" height="79" uuid="d1aa9148-99a3-4d31-88ee-e3dd44eb07f4"/>
				<textElement textAlignment="Center" verticalAlignment="Middle" rotation="None">
					<font size="20"/>
				</textElement>
				<text><![CDATA[사용자 리스트]]></text>
			</staticText>
		</band>
	</title>
	<pageHeader>
		<band height="82" splitType="Stretch">
			<staticText>
				<reportElement x="0" y="60" width="100" height="20" uuid="35ee7030-345b-4ca6-8518-d90b3937c092">
					<property name="com.jaspersoft.studio.unit.height" value="px"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
				</reportElement>
				<text><![CDATA[총 사용자 수:]]></text>
			</staticText>
			<textField>
				<reportElement x="100" y="60" width="100" height="20" uuid="6d62235d-83d5-4aae-be74-8124c9b4b644">
					<property name="com.jaspersoft.studio.unit.height" value="px"/>
				</reportElement>
				<textFieldExpression><![CDATA[$P{userTotalCount}]]></textFieldExpression>
			</textField>
		</band>
	</pageHeader>
	<columnHeader>
		<band height="61" splitType="Stretch"/>
	</columnHeader>
	<detail>
		<band height="237" splitType="Stretch">
			<componentElement>
				<reportElement x="0" y="0" width="555" height="237" uuid="49542a0b-21da-4a89-9088-cc770269b049">
					<property name="com.jaspersoft.studio.layout" value="com.jaspersoft.studio.editor.layout.VerticalRowLayout"/>
					<property name="com.jaspersoft.studio.table.style.table_header" value="Table_TH"/>
					<property name="com.jaspersoft.studio.table.style.column_header" value="Table_CH"/>
					<property name="com.jaspersoft.studio.table.style.detail" value="Table_TD"/>
				</reportElement>
				<jr:table xmlns:jr="http://jasperreports.sourceforge.net/jasperreports/components" xsi:schemaLocation="http://jasperreports.sourceforge.net/jasperreports/components http://jasperreports.sourceforge.net/xsd/components.xsd">
					<datasetRun subDataset="Dataset1" uuid="167b1609-9892-4308-9000-a1ef1468ba6c">
						<dataSourceExpression><![CDATA[new net.sf.jasperreports.engine.data.JRBeanCollectionDataSource($P{datasource1})]]></dataSourceExpression>
					</datasetRun>
					<jr:column width="100" uuid="63be25a7-6792-4a02-b92c-86c51ca490eb">
						<jr:columnHeader style="Table_CH" height="30">
							<staticText>
								<reportElement x="0" y="0" width="100" height="30" uuid="681c6c43-d707-4f7b-9d76-27aa630f6e73"/>
								<textElement textAlignment="Center" verticalAlignment="Middle" rotation="None" />
								<text><![CDATA[사용자ID]]></text>
							</staticText>
						</jr:columnHeader>
						<jr:detailCell style="Table_TD" height="30">
							<textField>
								<reportElement x="0" y="0" width="100" height="30" uuid="fdee2ed1-5554-432e-80e9-2eb1a2bb42cc"/>
								<textElement textAlignment="Center" verticalAlignment="Middle" rotation="None" />
								<textFieldExpression><![CDATA[$F{userId}]]></textFieldExpression>
							</textField>
						</jr:detailCell>
					</jr:column>
					<jr:column width="100" uuid="63be25a7-6792-4a02-b92c-86c51ca490eb">
						<jr:columnHeader style="Table_CH" height="30">
							<staticText>
								<reportElement x="0" y="0" width="100" height="30" uuid="681c6c43-d707-4f7b-9d76-27aa630f6e73"/>
								<textElement textAlignment="Center" verticalAlignment="Middle" rotation="None" />
								<text><![CDATA[사용자명]]></text>
							</staticText>
						</jr:columnHeader>
						<jr:detailCell style="Table_TD" height="30">
							<textField>
								<reportElement x="0" y="0" width="100" height="30" uuid="fdee2ed1-5554-432e-80e9-2eb1a2bb42cc"/>
								<textElement textAlignment="Center" verticalAlignment="Middle" rotation="None" />
								<textFieldExpression><![CDATA[$F{userName}]]></textFieldExpression>
							</textField>
						</jr:detailCell>
					</jr:column>
					<jr:column width="355" uuid="63be25a7-6792-4a02-b92c-86c51ca490eb">
						<jr:columnHeader style="Table_CH" height="30">
							<staticText>
								<reportElement x="0" y="0" width="355" height="30" uuid="681c6c43-d707-4f7b-9d76-27aa630f6e73"/>
								<textElement textAlignment="Center" verticalAlignment="Middle" rotation="None" />
								<text><![CDATA[주소]]></text>
							</staticText>
						</jr:columnHeader>
						<jr:detailCell style="Table_TD" height="30">
							<textField>
								<reportElement x="0" y="0" width="355" height="30" uuid="fdee2ed1-5554-432e-80e9-2eb1a2bb42cc"/>
								<textElement textAlignment="Center" verticalAlignment="Middle" rotation="None" />
								<textFieldExpression><![CDATA[$F{address}]]></textFieldExpression>
							</textField>
						</jr:detailCell>
					</jr:column>
				</jr:table>
			</componentElement>
		</band>
	</detail>
	<columnFooter>
		<band height="45" splitType="Stretch"/>
	</columnFooter>
	<pageFooter>
		<band height="54" splitType="Stretch"/>
	</pageFooter>
	<summary>
		<band height="176" splitType="Stretch"/>
	</summary>
</jasperReport>
```

디자인 파일이 완성되었습니다. 만약 그림처럼  parameter오류가 나오더라도 무시 하시면 됩니다.

![image-20210228015657188](/assets/images/posts/image-20210228015657188.png)

이제 폰트 설정 정보만 export 한다음 spring project로 가져가면 됩니다.  프로젝트 Properties로 들어갑니다.

![image-20210228015915879](/assets/images/posts/image-20210228015915879.png)

report에서 사용할 폰트를 지정합니다.

![image-20210228020156406](/assets/images/posts/image-20210228020156406.png)

![image-20210228020213013](/assets/images/posts/image-20210228020213013.png)

apply 한 다음 Export버튼을 누르고 jar파일을 저장합니다.

![image-20210228020542110](/assets/images/posts/image-20210228020542110.png)

압푹을 풀면 fonts 디렉토리와 jasperreports_extension.properties 파일이 있는데 spring 프로젝트의 resource 경로 아래로 옮겨주면 됩니다. fonts 폴더는 이름을 변경해도 되고  jasperreports_extension.properties 와 fonts/fontsfamilyxxxxx.xml 파일 내용 중  fonts경로만 변경 해주면 됩니다.

![image-20210228020634296](/assets/images/posts/image-20210228020634296.png)

Spring project는 Spring boot로 간단하게 구현하여 테스트 했습니다.

![image-20210228021257386](/assets/images/posts/image-20210228021257386.png)

Controller

```java
import java.io.FileNotFoundException;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import net.sf.jasperreports.engine.JRException;

@Controller
public class DemoController {
	private final DemoService demoService;
	
	public DemoController(DemoService demoService) {
		this.demoService = demoService;
	}
	
	@GetMapping("/report")
	public String report() throws FileNotFoundException, JRException {
		demoService.exportReport();
		return "main";
	}
}
```

Service

```java
import java.io.File;
import java.io.FileNotFoundException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.IntStream;

import org.springframework.stereotype.Service;
import org.springframework.util.ResourceUtils;

import net.sf.jasperreports.engine.JREmptyDataSource;
import net.sf.jasperreports.engine.JRException;
import net.sf.jasperreports.engine.JasperCompileManager;
import net.sf.jasperreports.engine.JasperExportManager;
import net.sf.jasperreports.engine.JasperFillManager;
import net.sf.jasperreports.engine.JasperPrint;
import net.sf.jasperreports.engine.JasperReport;

@Service
public class DemoService {

	public void exportReport() throws FileNotFoundException, JRException {
		File file = ResourceUtils.getFile("classpath:jrxml/users.jrxml");
		JasperReport jasperReport = JasperCompileManager.compileReport(file.getAbsolutePath());
		
		Map<String, Object> parameters = new HashMap<>();
		
		List<USER> users = new ArrayList<>();
		
		IntStream.rangeClosed(1, 100).forEach( idx -> {
			users.add(USER.builder().userId("test"+idx)
									.userName("사용자"+idx)
									.address("주소"+idx)
									.build());
		});
		
		parameters.put("userTotalCount", "123456");
		parameters.put("datasource1", users);
		
		//JRBeanCollectionDataSource dataSource = new JRBeanCollectionDataSource(users);
		//JasperPrint jasperPrint = JasperFillManager.fillReport(jasperReport, parameters, dataSource);
		
		JasperPrint jasperPrint = JasperFillManager.fillReport(jasperReport, parameters, new JREmptyDataSource());
		String path = "C:\\Users\\mosic\\Downloads\\users.pdf";
		
		//JasperExportManager.exportReportToPdf(jasperPrint);
		//JasperExportManager.exportReportToPdfStream(jasperPrint, null);
		JasperExportManager.exportReportToPdfFile(jasperPrint, path);
	}
}
```

build.gradle (jasperreports의 pdf다운로드 기능은 itext 2.1.7 버전과 동작합니다.)

```properties
plugins {
	id 'org.springframework.boot' version '2.4.3'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
	id 'war'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
	mavenCentral()
}

dependencies {
	//lombok
	compileOnly         'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	
	implementation group: 'net.sf.jasperreports', name: 'jasperreports', version: '6.1.0'
	implementation group: 'com.lowagie', name: 'itext', version: '2.1.7'
	
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}

```

![image-20210228022114924](/assets/images/posts/image-20210228022114924.png)