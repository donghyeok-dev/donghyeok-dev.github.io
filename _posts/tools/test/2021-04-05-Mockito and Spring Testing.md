---
layout: customPost
title:  "Mockito와 테스트 전략"
categories: 
  - Spring
  - Test
---

##  Mockito가 무엇인가요?

Mock이란 진짜 객체와 비슷한 동작을 하지만 개발자가 직접 행동을 관리하는 객체를 말하며, Mockito는 이러한 Mock 객체를 쉽게 만들고 관리하고 검증할 수 있는 방법을 제공하는 테스트 프레임워크입니다. Mockito 이외에도 JMock, EasyMock 등이 있습니다.

spring boot는 기본적으로  Mockito 및 테스트 관련 dependencies가 포함되어 있는 spring-boot-starter-test를 제공합니다. 단, spring boot 2.2 이후로  JUnit5, AssertJ 등이 포함되었기 때문에 현재 사용하고 계신 버전을 확인 하시고 해당 라이브러리가 없다면 직접 추가해서 사용해야 됩니다.

```
testImplementation 'org.springframework.boot:spring-boot-starter-test'
```

- JUnit5
- Spring Test
- AssertJ
- Hamcrest
- Mockito
- JsonPath
- JSONassert



## Mockito를 왜 쓰죠?

데이터베이스 또는 외부 API 등의 **의존성을 배제한 단위 테스트**를 하고자 할 때 사용합니다. 

예를들어 Product 코드의 Contoller 특정 메서드 내에서 Service나 Repository의 특정 메서드를 포함하고 있을 때, Controller의 특정 메서드를 테스트 하려면 내부에 포함되어 있는 Service나 Repository를 구현한 다음 테스트를 진행 해야 됩니다.  Product에 구현되어 있는 것과 별개로 테스트 코드에서는 Controller가 의존하고 있는 객체들을 Mocking하므로서 의존성을 배제할 수 있습니다. 뿐만 아니라 Mocking된 객체들의 메서드를 호출할 때 특정 조건(예: parameter가 3일 때)과 리턴 값(예: return 10을 해라)을 내 임의로 설정 한다던지, 특정 메소드의 호출 여부 등을 검사할 수도 있습니다. 이처럼 Controller나 Service, Repository 등의 **단위 테스트를 할 때 사용할 수 있는 유용한 Java용 오픈 소스 테스트 프레임워크입니다.**



## Mockito를 어떻게 쓰죠?

- Mockito.mock()로 Mock을 직접 생성하거나 Mocking할 클래스에 @Mock, @Spy 등을 Annotated 합니다.
- Mocking할 클래스에 생성자 등으로 의존성 주입을 하고 있다면 @InjectMocks로 Annotated하고 DI 대상 클래스는 @Mock 또는 @Spy로 Annotated합니다.
- Mock 대상 객체들을 초기화하여 Mocking합니다.
- when, given, doReturn 등으로 Mocking된 객체들의 메서드에 행위를 설정합니다.
- Assertions, verify, Matchers 등으로 값을 검증합니다.



## Mockito.mock() 메서드로 Mock 만들기

- 클래스 또는 인터페이스의 모의 개체를 만들고 Mock 메서드에 대한 반환 값을 스텁하고 호출되었는지 확인할 수 있습니다.

```java
@SpringJUnitWebConfig
class ControllerTest {
    Service service;
 	
    @BeforeEach
    void setUp() {
        this.service = Mockito.mock(Service.class);
    }
    
    @Test
    @DisplayName("테스트")
    void test() throws Exception {
        final String param = "Hi! Nice meet you";

        when(this.service.getMessage(anyString())).thenReturn(Arrays.asList("안녕!", "만나서", "반가워"));

        System.out.println(">>> " + this.service.getMessage(param));
        
        verify(this.service).getMessage(param);
    }
}
    
```



### Annotated된 객체를 Mock으로 초기화하는 3가지 방법

- **MockitoAnnotations.openMocks(this);** 

  ```java
  public class ControllerTest {
      private AutoCloseable closeable;
      
      @Mock
      Service service;
      
      @BeforeEach
      public void openMocks() {
          closeable = MockitoAnnotations.openMocks(this);
      }
  	
      //...
      
      @AfterEach
      public void releaseMocks() throws Exception {
          closeable.close();
      }
  }
  ```

- **@ExtendWith(MockitoExtension.class)   테스트 클래스에 Annotated 합니다.**

  ```java
  @ExtendWith(MockitoExtension.class)
  class ControllerTest {
      @Mock
      Service service;
      
      //...
  }
  ```

  MockitoExtension의 기본 설정은 Strictness.STRICT_STUBS이며 아래와 같은 기능을 합니다.

  > 생산성 향상: 테스트 대상 코드가 다른 인수를 사용하여 스텁 메서드를 호출하면 테스트가 조기에 실패합니다.

  > 불필요한 스텁이 없는 클리너 테스트: 사용되지 않은 스텁이 있으면 테스트가 실패함.

  > 스텁된 호출을 더 이상 명시적으로 확인할 필요가 없습니다.

  세션을 초기화 하기 위해 새로운 MockitoSession 인스턴스를 만듭니다.  이때 @Mock, @Spy, @InjectMocks등으로 Annotated된 객체(필드)들은 initMocks() 메서드에 의해 (STRICT_STUBS 기반으로) 초기화 됩니다.

  Strictness의 값을 변경하려면 아래와 같이 Annotation을 추가하면 됩니다.

  ```java
  @MockitoSettings(strictness = Strictness.STRICT_STUBS)
  @ExtendWith(MockitoExtension.class)
  class ControllerTest {
  ```

  

- **@Rule public MockitoRule mockito = MockitoJUnit.rule();**

  MockitoRule는 mock을 초기화하고 사용을 검증하며 잘못된 스텁을 감지하며 초기화 방법 중 **제일 추천하는 방법입니다.**
  
  > 스텁 인수 불일치를 자동으로 감지하고 Strictness(엄격성?)으로 규칙을 구성합니다.
  >
  > MockitoJUnitRunner 대신 JUnit Rule을 사용할 수 있고 JUnit 4.7이상이 필요합니다.
  >
  > Mockito Annotation이 달린 mock을 초기화 합니다.
  >
  > MockitoJUnit.rule()의 default Strictness는 Strictness.WARN입니다.
  >
  > MockitoJUnit.rule().strictness(Strictness.STRICT_STUBS); 으로 Strictness를 변경할 수 있습니다.

  ```java
  class ControllerTest {
      @Rule
      public MockitoRule mockito = MockitoJUnit.rule();
      
      @Mock
      Service service;
      
   	//...   
  }
  ```

  참고: MockitoJUnit 내부를 따라 가보면 MockitoExtension 처럼 MockitoSession 인스턴스를 만드는 메소드를 포함하고 있습니다.

  ![image-20210401143124692](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210401143124692.png)



## Mockito Annotation

### @Mock

@Mock을 Annotated하면 Mockito가 초기화할 때 대상을 스캔하여 Mock 객체로 만들어 줍니다.

Mock객체는 실제 객체를 참조하지 않기 때문에 실제 메서드의 리턴값과 관계 없이 지정한 조건에 맞는 할당한 값을 리턴 받을 수 있습니다. when절로 할당하지 않으면 함수의 리턴 값은 default값이 리턴 되는데 타입에 따라 0 또는 null이 나옵니다.

mockitoController.java

```java
private final TestService testService;

public MockitoController(TestService testService) {
    this.testService = testService; //testService: testService
}

@GetMapping("/example3")
public Integer example3(Integer value) {
    //첫번째 호출 resultValue: 0, 두번째 호출 resultValue: 10
    Integer resultValue = testService.calculateValue(value); 
    return resultValue;
}
```

TestService.java

```java
public int calculateValue (int value) { 
    return value * 2; //실행 안됨.
}
```

Test.java

```java
@Mock
TestService testService;

@InjectMocks
MockitoController mockitoController;

//...

final int value = 3;

//첫번째 호출
this.mockMvc.perform(get("/example3").param("value", String.valueOf(value)));

//리턴값 지정
when(this.testService.calculateValue(value)).thenReturn(10);

//두번째 호출
this.mockMvc.perform(get("/example3").param("value", String.valueOf(value)));
```



### **@Spy**

테스트할 객체 중 특정 객체만 선택적으로 실제 객체로 테스트할 수 있게 해줍니다. 그럼 실제 객체를 그냥 사용하면 되지 왜 @Spy로 Mocking하나요? 실제 객체의 Product 코드를 변경하며 테스트를 해야되는 번거움도 있고, 실제 객체를 직접 new로 인스턴스화 하여 컨트롤러에 주입 시켜야되며 그 컨트롤러 또한 Mock객체가 아닌 실제 객체를 주입 받아야되는 상황이므로 @injectMocks를 사용 못하고 직접 생성해야 됩니다. 그리고 verify 메서드 와 같은 Mock 객체만 사용할 수 있는 메서드들도 사용할 수 없습니다.

```java
//@Spy사용 
@Mock
MockitoService mockitoService;

@Spy
TestService testService;

@InjectMocks
MockitoController mockitoController;

@BeforeEach
void setUp() {
	this.mockMvc = MockMvcBuilders.standaloneSetup(mockitoController).build();
}

//직접 생성
@Mock
MockitoService mockitoService;

TestService testService;

MockitoController mockitoController;

@BeforeEach
void setUp() {
    this.testService = new testService();
	this.mockMvc = MockMvcBuilders.standaloneSetup(new MockitoController(mockitoService,  testService)).build();
}
```



**@Spy 테스트 예제**

mockitoController.java

```java
private final TestService testService;

public MockitoController(TestService testService) {
    this.testService = testService; 
    //testService: com.example.tdd1.TestService$MockitoMock$1712051045@6f5d90f7
}

@GetMapping("/example3")
public Integer example3(Integer value) {
    //첫번째 호출 resultValue: 6, 두번째 호출 resultValue: 10
    Integer resultValue = testService.calculateValue(value); 
    return resultValue;
}
```

TestService.java

```java
public int calculateValue (int value) { 
    return value * 2; //첫번째, 두번째 모두 호출됨.
}
```

Test.java

```java
@Spy
TestService testService;

@InjectMocks
MockitoController mockitoController;

//...

final int value = 3;

//첫번째 호출
this.mockMvc.perform(get("/example3").param("value", String.valueOf(value)));

//리턴값 지정
when(this.testService.calculateValue(value)).thenReturn(10);

//두번째 호출
this.mockMvc.perform(get("/example3").param("value", String.valueOf(value)));
```

위 예제를 간략히 설명하자면 객체의 메서드가 두번 모두 호출되었는데, 첫번째 resultValue는 3*2가되어 6으로 나오지만, 두번째는 호출전 테스트코드에서 작성한 when(...) thenReturn(10);의 정한 값대로 10이 나오는걸 확인할 수 있습니다. 이것으로 알 수 있는 것은 실제 객체의 메서드가 호출은 되지만 Mockito의 예상값을 지정하면 그 값으로 오버라이드 된 값이 나온다는 것을 알 수 있었습니다.

위의 Test.java에서 when 메서드 이외에도 비슷한 기능을 하는 doReturn 메서드가 있습니다.

```java
doReturn(리턴값).when(Mock객체).Mock객체의 메서드(...);
```

when과 doReturn 의 차이는 @Spy 객체를 사용할 때 차이가 나는데요,  위에서 @Spy로 Mocking된 TestService의 calculateValue 메서드는 Controller에서 호출 시에 실제 메서드가 호출이 되었습니다.  만약 Spy 클래스에서 특정 메서드는 @Mock 처럼 실제 메서드를 호출하지 않고 지정된 값으로 대체하고 싶다면 어떻게 해야될까요?

doReturn(...) 메서드는  @Mock처럼 실제 호출을 하지 않고 개발자가 정의한 리턴을 줄 수 있도록 합니다. 예를들어 

```java
//이렇게 @Spy 객체를 when으로 사용하면 calculateValue가 연속으로 5번 실제로 호출이 됩니다.
when(this.testService.calculateValue(3)).thenReturn(10);
when(this.testService.calculateValue(4)).thenReturn(11);
when(this.testService.calculateValue(5)).thenReturn(12);
when(this.testService.calculateValue(6)).thenReturn(13);
when(this.testService.calculateValue(7)).thenReturn(14);

//doReturn을 사용하면 calculateValue가 한번도 호출되지 않으면서 위와 같은 결과값이 나옵니다.
doReturn(10).when(this.testService).calculateValue(3);
doReturn(11).when(this.testService).calculateValue(4);
doReturn(12).when(this.testService).calculateValue(5);
doReturn(13).when(this.testService).calculateValue(6);
doReturn(14).when(this.testService).calculateValue(7);
```



추가로 연속된 when, doReturn을 사용하고 싶다면 아래와 같이 작성하면 됩니다.

```java
when(this.testService.calculateValue(3)).thenReturn(10, 11, 12, 13, 14);
//또는
doReturn(10, 11, 12, 13, 14).when(this.testService).multiply(3);

//10
this.mockMvc.perform(get("/example3").param("value", String.valueOf(value)));
//11
this.mockMvc.perform(get("/example3").param("value", String.valueOf(value)));
//12
this.mockMvc.perform(get("/example3").param("value", String.valueOf(value)));
```



### @InjectMocks

@InjectMocks로 Annotated된 클래스는 생성자 Injection 등이 필요하면, ExtensionContext.getRequiredTestInstances().getAllInstances()에서  @Mock, @Spy 등으로 Mocking된 객체를 찾아서 DI 해줍니다.  

아래 예제는 MockitoController에 Injection 대상 mockitoService, testService를 자동으로 Mock객체를 DI 해주는 예제 입니다.

MockitoController.java

```java
private final MockitoService mockitoService;
private final TestService testService;

public MockitoController(MockitoService mockitoService, TestService testService) {
    this.mockitoService = mockitoService;
    this.testService = testService;
}
```

Test.java

```java
@Mock
MockitoService mockitoService;

@Spy
TestService testService;

//직접 객체를 생성하고 Dependency Injection을 수동으로 하는 방법 
this.mockMvc = MockMvcBuilders.standaloneSetup(new MockitoController(mockitoService, testService)).build();

//@InjectMocks을 이용하는 방법
@InjectMocks
MockitoController mockitoController;

this.mockMvc = MockMvcBuilders.standaloneSetup(mockitoController).build();

```



### @MockBean

```java
import org.springframework.boot.test.mock.mockito.MockBean;
```

Spring boot는 @MockBean Annotation으로 ApplicationContext에 등록된 Bean을  Mock 객체로 오버라이드할 수 있게 해줍니다. 이는 테스트 클래스에서 선언된 Mock이 응용프로그램 내의 빈으로 주입된다는 의미입니다.

여기서 이런 생각이 들 수도 있을 것 같습니다.

@Mock과 상세하게 무슨 차이가 나는지 이미지로 안그려지고 글자가 휘날리는 느낌이다. 

이런 생각... 맞나요? (저만 그럴지도.. 쿨럭) 



ApplicationContext를 사용하지 않고  독립적인 단위 테스트 시에 @MockBean을 사용하는 예

Contoller.java

```java
@RestController
public class MockitoController {
    private final MockBeanService mockBeanService;
    
    public MockitoController(MockBeanService mockBeanService) {
        //(1)
        this.mockBeanService = mockBeanService;
    }
    
    @GetMapping("/mockbeanTest")
    public List<String> mockbeanTest(String param) {
        //(2)
        return this.mockBeanService.getSamples(param);
    }
}
```

Test.java

```java
@SpringJUnitWebConfig
class MockitoControllerTest {
@Rule
public MockitoRule mockito = MockitoJUnit.rule();

MockMvc mockMvc;

@MockBean
MockBeanService mockBeanService;

@InjectMocks
MockitoController mockitoController;
    
@BeforeEach
void setUp() {
    this.mockMvc = MockMvcBuilders.standaloneSetup(mockitoController)
            .build();
}
    
@Test
@DisplayName("MockBean 테스트")
void testMockbean() throws Exception {
    final String param = "Hi, Nice Meet you";

    when(this.mockBeanService.getSamples(anyString())).thenReturn(Arrays.asList("안녕", "만나서", "반가워"));

    this.mockBeanService.getSamples(param));   //(3)
    
    this.mockMvc.perform(get("/mockbeanTest")
                .param("param", param))
                .andExpect(status().isOk()); //(4)
}
```

**(4)**에서 오류가 빵~하고 터집니다.

<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210403235311959.png" alt="image-20210403235311959" style="zoom:150%;" />

위와 같이 Controller를 @InjectMocks으로 Test Scope에 Bean을 생성시키고, Service를 @MockBean으로   Application Context에 Bean을 생성 시키면,  **(1)**  Controller는 Injection 대상인 Service를 Test Scope에서 찾기 때문에 DI가 실패 되고  null로 생성됩니다.  

**(3)** test파일 내에서는 Application Context의 Service Bean을 Access 할 수 있기 때문에 정상적으로 호출이 됩니다.

**(4)** test파일에서 controller로 request요청을 보내고 **(2)** controller에서 참조하고 있는 service가 null이므로 500에러가 response 됩니다.

<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210403234445245.png" alt="image-20210403234445245" style="zoom:150%;" />

> 
>
> **만약 @MockBean 대신 @Mock으로 변경하면 Service가 Test Scope에 생성되기 때문에 모두 성공하게 됩니다.**
>
> 

<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210403235337081.png" alt="image-20210403235337081" style="zoom:150%;" />

<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210403235511674.png" alt="image-20210403235511674" style="zoom:150%;" />



그럼 그림에 보이는 Application Context와 Web Server는 언제 사용하지? 라는 생각이 들 수도 있고 이미 알고 계신분도 있으리라 생각됩니다.

먼저 Application Context만 사용하는 경우에 대해서 말씀드리면 위에서 예제 첫번째 줄에@SpringJUnitWebConfig 대신에 많이들 사용하고 계신? @WebMvcTest으로 변경하면 Application Context를 사용하여 테스트를 하게 되는 것입니다. 그럼 위 예제를 @WebMvcTes과  @MockBean을 사용하여 테스트 해보면?

<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210403235311959.png" alt="image-20210403235311959" style="zoom:150%;" />

네! 실패 합니다. 이유는 첫번째 실패할때 사유와 같습니다. @WebMvc는 Application Context를 사용하게만 해준다는 말이지 Test Scope에 있는 Controller와 Application Context에 있는 Service를 연결해주는 기능까지 해준다는 말은 아닙니다.

<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210404002613046.png" alt="image-20210404002613046" style="zoom:150%;" />

그럼 어떻게 해야 하나요?

controller를 Application Context로 옮겨야 되나? 원래 @Controller나 @RestController는 Ioc Container에 의해 자동으로 Application Context에 등록되는게 아니였나? 갑자기 머리가 복잡해지기 시작합니다...

원인은 @InjectMocks로 controller를 Test Scope에 생성시켜서 같은 타입으로 Application Context에 생성되어 있는 controller bean이 Ignore되었기 때문입니다. 

간단하게 해결하려면  @InjectMock 대신 @Autowired을 사용하면 됩니다. 그러면 Application Context에 있는 controller의 bean을 사용할테니 말이죠. 

```java
@WebMvcTest
class MockitoControllerTest {
@Rule
public MockitoRule mockito = MockitoJUnit.rule();

MockMvc mockMvc;

@MockBean
MockBeanService mockBeanService;

@Autowired
MockitoController mockitoController;
    
//..
    
}
```

여기에서 한가지 궁금한게 있습니다. 만약 Application Context의 Controller이 Test Scope에 있는 또 다른 Mocking된 객체를 DI가 가능한가? **Test Scope과 Application Context의 DI는 불가합니다.** 

<img src="https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210404011008175.png" alt="image-20210404011008175" style="zoom:150%;" />



**@WebMvcTest를 Annotated하면 자동으로 MockMvc 인스턴스가 Application Context에 구성됩니다.** 따라서  @Autowired로 Application Context에 있는 인스턴스를 사용할 수 있고 생략 가능한 부분도 생깁니다.

```java
@Autowired
MockMvc mockMvc;

@MockBean
MockBeanService mockBeanService;

/* 생략
@Autowired
MockitoController mockitoController;
*/

@BeforeEach
void setUp(WebApplicationContext wac) {
    //생략
    //this.mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    //this.mockMvc = MockMvcBuilders.standaloneSetup(mockitoController).build();
}
//...

this.mockMvc.perform(get("/mockbeanTest")...
```

mockMvc가 @Autowired로 Application Context에 있는 Bean을 사용하면 Controller Bean과 동일한 영역에 있으므로  mockMvc.perform()으로 Controller에게 요청을 보낼 수 있습니다.

참고로 말씀드리면 @Autowired로  Application Context의 mockMvc을 사용하는 것과  setUp 메서드에서 mockMvc = MockMvcBuilders.webAppContextSetup(wac).build()로 생성된 mockMvc와는  다른 영역이므로 이름 때문에 혼동하지 마시길 바랍니다.

@WebMvcTest는 단위 테스트보단 통합 테스트엔 가까운 친구인 것 같습니다. 그리고 Application Context를 사용하지만 요청과 응답은 여전히 가짜라는 것을 명심해야 합니다.  

다음은 통합 테스트에서 사용하는 @SpringBootTest Annotation에 대한 설명입니다.

- @ContextConfiguration(loader=...)가 정의되지 않은 경우 SpringBootContextLoader를 사용하고 @Configuration을 사용하지 않고 명시적인 클래스가 지정(@SpringBootTest(classes=....)) 되지 않은 경우@SpringBootConfiguration을 자동으로 검색합니다.
- 사용자 지정 환경 속성을 정의하면 웹서버의 실행 포트를 지정하거나 랜덤포트로 정의할 수 있습니다. 

- 표준 RestTemplate를 사용하여 REST 컨트롤러에 대한 외부 서버 테스트를 작성할 수 있습니다.
- 통합 테스트에 유용한 기능(인증 헤더  등)이 포함된 테스트별 TestRestTemplate를 사용할 수 있습니다.
- 기존 방식처럼 서비스 계층들을 Mocking하고 컨트롤러만 테스트도 가능합니다.
- 실행 속도는 가장 느립니다.   SpringBootTest  <  WebMvcTest  <<< standaloneSetup



@WebMvcTest 대신에 @SpringBootTest를 적용 해봅니다. 이때 @SpringBootTest만 사용하면 MockMvc를 Autowired를 할 수 없다는 오류가 나옵니다. 오류를 간단히 해결하려면 MockMvc의 자동 구성을 활성화 및 구성하기 위해 테스트 클래스에 적용할 수 있는 @AutoConfigureMockMvc Annotation을 같이 사용하는 것입니다.

Controller.java

```java
//...

@GetMapping("/mockbeanTest")
    public List<String> mockbeanTest(HttpServletRequest request, String param) {
        //(1) request : org.springframework.mock.web.MockHttpServletRequest@546ed2a0
        return this.mockBeanService.getSamples(param);
    }
```

Test.java

```java
@SpringBootTest
@AutoConfigureMockMvc
class MyIntegratedTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    MockBeanService mockBeanService;

    @Test
    @DisplayName("통합테스트 MockMvc")
    void testMockMvc() throws Exception {
        //@SpringBootTest와 @AutoConfigureMockMvc가 Annotated되어야 한다.
        String param = "Hello";

        when(this.mockBeanService.getSamples(anyString())).thenReturn(Arrays.asList("안녕", "만나서", "반가워"));

        this.mockMvc.perform(get("/mockbeanTest")
                .param("param", param))
                .andExpect(status().isOk());
    }
}
```

@SpringBootTest로 구성은 했지만 아직 @WebMvcTest로 구성했을 때와 별 차이가 없습니다. 그 이유는 **@SpringBootTest 기본값인 WebEnvironment.MOCK로 구성**되는데 이는 servlet API가 클래스 경로에 있는 경우 모의 서블릿 환경으로 웹 애플리케이션 컨텍스트 만들어 주기 때문에 실제 서버를 로드하진 않습니다.  그래서 **(1)** **Controller의 request가 MockHttpServletRequest로** **들어오기 때문에 표준 RestTemplate를 사용할 수 없고, MockMvc를 사용해야 합니다.**

- MOCK

  WebApplicationContext를 로드하며 내장된 서블릿 컨테이너가 아닌 Mock 서블릿을 제공합니다. 

- RANDOM_PORT

  EmbeddedWebApplicationContext를 로드하며 실제 서블릿 환경을 구성합니다. 생성된 서블릿 컨테이너는 임의의 포트는 listen합니다.

- DEFINED_PORT

  RAMDOM_PORT와 동일하게 실제 서블릿 환경을 구성하지만  application.properties 또는 application.yml에서 지정한 포트를 listen합니다.

- NONE

  일반적인 ApplicationContext를 로드하며 아무런 서블릿 환경을 구성하지 않습니다.

이렇게 @SpringBootTest와 @AutoConfigureMockMvc 조합으로 MockMvc를 사용할 거라면 차라리 특정 컨트롤러에 대해 로드된 컨텍스트를 사용하는 @WebMvcTest를 사용하는 것이 더 좋습니다.

그럼 실제 서버 환경에서 테스트 하기 위해서 @SpringBootTest에 서버 포트는 WebEnvironment.DEFINED_PORT 또는 WebEnvironment.RANDOM_PORT로 지정할 수 있는데, RANDOM_PORT를 주로 사용하게 됩니다. 그 이유는 임의로 할당된 포트 번호를 사용하게 되면 병렬 테스트를 실행할 때 포트 충돌을 방지할 때 유용하기 때문입니다.

Contoller.java

```java
//...

@GetMapping("/mockbeanTest")
public List<String> mockbeanTest(HttpServletRequest request, String param) {
    //(1) request : org.apache.catalina.connector.RequestFacade@4b645a70
    return this.mockBeanService.getSamples(param);
}
```

Test.java

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class MockitoControllerTest {
	@Autowired
    private TestRestTemplate testRestTemplate;

    @MockBean
    MockBeanService mockBeanService;

    @Test
    @DisplayName("통합테스트 testRestTemplate")
    void testRestTemplate() {
        String param = "Hello";

        when(mockBeanService.getSamples(param)).thenReturn(Arrays.asList("안녕"));

         ResponseEntity<String> responseEntity = testRestTemplate.getForEntity("/mockbeanTest?param={param}", String.class, param);

         assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
    }
}
```



@SpringBootTest를 이용한 통합 테스트는 이전 방법들과 다르게 웹 서버와 함께 Spring Boot Context를 로드하면서 Filter 나 ControllerAdvice 등과 같은 부가적인 요소들도 함께 로드 합니다. 그래서 테스트 방식 중 가장 무겁고 로드가 오래 걸립니다. 하지만 클래스간 상호 작용도 확인할 수 있기 때문에 통합 테스트에서는 이 방식을 사용해야 합니다. 단 유닛 테스트와는 별개로 생각 해보아야 할 부분이기도 합니다.

정리해보면 다른 동작에 초점을 맞추지 않고 컨트롤러의 논리에 대한 단위테스트를 작성할때는 standaloneSetup을 사용해야 하고 필터링, 인터셉터, 인증 등을 테스트 해야할 경우에는 @SpringBootTest를 이용해서 테스트를 해야 합니다.



## @SpyBean

@Mock과 @Spy의 차이점을 @MockBean과 @SpyBean에 대입해서 생각해 보시면 됩니다.

ApplicationContext에 있는 Bean을 실제 객체를 사용하는 것 입니다.



## InOrder 예제

메서드 호출 순서를 검증합니다.

```java
@Test
@DisplayName("InOrder 테스트")
void testInOrder() {
    this.testService.multiply(3);
    this.testService.multiply(4);
    this.testService.multiply(5);

    //순서대로 호출 되는지 검증
    InOrder inOrder =  inOrder(this.testService);
    inOrder.verify(this.testService).multiply(3);
    inOrder.verify(this.testService).multiply(4);
    inOrder.verify(this.testService).multiply(5);
}
```



## doThrow 예제

```java
@Spy
TestService testService;

//...

@SneakyThrows
    @Test
    @DisplayName("Mockito doThrow Method 테스트")
    void testException()  {

        //value가 0으로 넘어가면 실제 메서드에서 ArithmeticException: / by zero 발생.
        this.mockMvc.perform(get("/example4")
                .param("value", String.valueOf(0)))
                .andExpect(result -> Assertions.assertTrue(result.getResolvedException() instanceof ArithmeticException))
                .andReturn();

        //파라메터 10이 넘어오면 RuntimeException 발생하도록 설정함.
        doThrow(new RuntimeException("아메리카노")).when(this.testService).division(10);

        //value 10을 넘겨서 RuntimeException 발생 확인.
        this.mockMvc.perform(get("/example4")
                .param("value", String.valueOf(10)))
                .andExpect(result -> Assertions.assertTrue(result.getResolvedException() instanceof RuntimeException)) //ArithmeticException
                .andReturn();

        verify(this.testService, times(2)).division(isA(Integer.class));
    }
```



## doNothing 예제

```java
@Spy
TestService testService;

//...

@Test
@DisplayName("doNothing 테스트")
void testDoNothing() throws Exception {
    //testService의 getMessage 메서드 호출 무시.
    ArgumentCaptor<Integer> valueCapture = ArgumentCaptor.forClass(Integer.class);

    //getMessage 메서드는 호출 하진 않지만  넘어가는 파라메터를 캡처함.
    doNothing().when(this.testService).getMessage(valueCapture.capture());

    this.mockMvc.perform(get("/example5")
            .param("value", "10"))
            .andReturn();

    //캡처한 파라메터값 검증.
    assertEquals(10, valueCapture.getValue());

    verify(this.testService).division(anyInt());
    verify(this.testService).getMessage(anyInt());
}
```



## doAnswer 예제

```java
@Spy
TestService testService;

//...

@Test
@DisplayName("Answer 테스트")
void testAnswer() {
    Answer<Integer> answer = new Answer<Integer>() {
        @Override
        public Integer answer(InvocationOnMock invocation) throws Throwable {
            return Arrays.stream(invocation.getArguments())
                .map(o -> Integer.parseInt(o.toString()))
                .reduce(0, Integer::sum);
        }
    };

    when(this.testService.dynamicSum(10, 20, 30)).thenAnswer(answer);
    System.out.println(">>> result: " + this.testService.dynamicSum(10, 20, 30));

    doAnswer(answer).when(this.testService).dynamicSum(10, 20, 30);
    System.out.println(">>> result: " + this.testService.dynamicSum(10, 20, 30));

    //dynamicSum 메서드는 두번 호출되었다고 검증되지만 Spy 객체인 testService.dynamicSum 메서드는 when절 이후에만 실제 호출되었음.
    verify(this.testService, times(2)).dynamicSum(any());
}
```



## doCallRealMethod

Mock객체의 메서드를 실제로 호출해준다. @Spy는 객체 전체 적용이라고 한다면 doCallRealMethod는 @Mock으로 Mocking된 객체의 특정 메서드만 실제 호출하여 테스트가 가능합니다.

```java
@Mock
TestService testService;

//...

@Test
@DisplayName("doCallRealMethod 테스트")
void testDoCallRealMethod() {
    doCallRealMethod().when(this.testService).getTeamName(anyString());

    this.testService.getTeamName("Spring");

    verify(this.testService, times(1)).getTeamName("Spring");
}
```

한가지 주의할 점은  Mocking된 객체가 추상적이면 안되고 java object class만 가능함.  예를들어 service를 인터페이스로 선언하여 @Mock하면 사용할 수 없습니다.

```properties
org.mockito.exceptions.base.MockitoException: 
Cannot call abstract real method on java object!
Calling real methods is only possible when mocking non abstract method.
```

