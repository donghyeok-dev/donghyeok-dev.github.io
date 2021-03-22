---
layout: customPost
title:  "Spring Boot에서 Junit5와 Mockito사용하기"
categories: 
  - Spring
  - Test

toc: true
toc_label: "Spring Boot에서 Junit5와 Mockito사용하기"
toc_sticky: true
---


## Unit Test란?

Unit Test는 Class, Method 등에서 발생할 수 있는 오류들을 작은 단위의 Test Case를 만들어 검사하는 방법이다. 이때 나누는 단위는 프로젝트의 크기, 종류 등에 따라 회사, 팀 별로 협의하여 정한다.

Unit Test의 장점

- 작은 단위로 나누어 오류를 정확하고 빠르게 파악할 수 있다.
- 개발시간에서 차지하는 디버깅 시간을 단축 시킨다.
- 테스트 코드를 믿고 리팩토링을 자주 할 수 있기 때문에 코드 품질이 향상 된다.
- 운영 중인 코드에 버그 또는 추가 개선 사항이 있을 때 최소한의 검증은 테스트 코드가 해준다.



## 1. JUnit 5

- JUnit5는 Unit Test를 쉽게할 수 있도록 만들어진 Test Framework의 한 종류인  JUnit의 최신 버전이다.

- JUnit5는 JUnit Platform + JUint Jupiter + JUnit Vintage의 Test Framework이다.

  ![image-20210320235416135](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210320235416135.png)

- JUnit4는 Vintage engine을 사용하며 spring boot 2.2.x 버전부터 기본 engine이 jupiter로 변경 됨.

- JUnit4, JUnit5는 Switch하여 테스트 하기는 쉽다. 테스트 파일을 생성할 때 Testing Libraray를 `JUnit4를 선택하면` gradle이 해당하는 라이브러리를 다운받는다. (gradle에 직접 명시해도 된다.)

  product Class에서 바로 Test Class를 생성하는 방법도 있다. Product Class에서 Alt+Ins, Test...을 선택하면 된다.

  ![image-20210320235004914](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210320235004914.png)

![image-20210320233827378](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210320233827378.png)

이렇게 하면 gradle 파일에 junit 4버전이 추가된다.

![image-20210320233217003](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210320233217003.png)

![image-20210320232943586](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210320232943586.png)

물론 이 상태에서 JUnit5도 테스트가 가능하다.

![image-20210320233429807](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210320233429807.png)

같은 패키지내에 JUnit 4와 JUnit 5를 동시에 구현해서 실행하면 Engine 명도 표시된다.

![image-20210320234255320](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210320234255320.png)

- JUnit5의  life Cycle는 [상세 이미지](https://junit.org/junit5/docs/current/user-guide/images/extensions_DatabaseTestsDemo.png){:target="_blank"}를 참고하고, 간단하게 설명 하자면 아래와 같다.

  ```
  exectue -> hooks( beforeXXX ) -> Test methods -> hooks( afterXXX ) -> result
  ```

- Annotation:

  `@Test` 테스트 메서드

  ```java
  @Test
  @DisplayName("기본적인 테스트 메소드")
  void test1() {
  	Assertions.assertEquals(1, 1);
  }
  ```

  `@ParameterizedTest` 매개 변수가 있는 테스트. 

  ```java
  @ParameterizedTest
  @DisplayName("매개변수 테스트")
  @ValueSource(strings={"Java", "Spring", "JUnit5"})
  public void test(String param) {
  	Assertions.assertTrue(() -> {
  		return Objects.requireNonNull(param).length() > 3; 
      });
  }
  ```

  ![image-20210321174123909](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210321174123909.png)

  `@RepeatedTest` 반복 테스트를 위한 테스트 .

  ```java
  @RepeatedTest(3)
  @DisplayName("반복 테스트")
  void testMethod1() {
      Assertions.assertEquals(1, 1);
  }
  
  @RepeatedTest(3)
  void testMethod2(RepetitionInfo repetitionInfo) {
      Assertions.assertTrue(repetitionInfo.getCurrentRepetition() <= repetitionInfo.getTotalRepetitions());
  }
  
  @RepeatedTest(value = 3, name="{displayName} {currentRepetition} of {totalRepetitions}")
  @DisplayName("JUnit5 반복 테스트")
  void testMethod3(TestInfo testinfo) {
      Assertions.assertTrue(Objects.nonNull(testinfo.getDisplayName()));
  }
  ```

  `@TestFactory` 동적 테스트.

  ```java
  @SpringBootTest
  public class SampleDynamicTest {
  
      @TestFactory
      Collection<DynamicTest> dynamicTests() {
          return Arrays.asList(
              dynamicTest("java", () -> assertTrue(true)),
              dynamicTest("spring", new CustomExecutable()),
              dynamicTest("Exception", () -> {throw new RuntimeException("Runtime Exception Test.");})
          );
      }
  }
  
  class CustomExecutable implements Executable {
      @Override
      public void execute() throws Throwable {
          System.out.println("~~~ ok ~~~~");
      }
  }
  ```

  `@TestTemplate`

  메서드가 등록 된 공급자가 반환 한 호출 컨텍스트 수에 따라 여러 번 호출되도록 설계된 테스트 케이스 의 템플릿.

  `@TestMethodOrder` (JUnit4 @FixMethodOrder)

  테스트 메서드에 @Order를 사용하려면 해당 클래스에 이 어노테이션을 정의해야 함.

  ```java
  @SpringBootTest(classes = SampleTestMethodOrder.class)
  @TestMethodOrder(MethodOrderer.OrderAnnotation.class)
  public class SampleTestMethodOrder {
      @Test
      @Order(3)
      @DisplayName("첫번째 메소드")
      public void firstMethod() {
          Assertions.assertEquals(1, 1);
      }
  
      @Test
      @Order(1)
      @DisplayName("두번째 메소드")
      public void secondMethod() {
          Assertions.assertEquals(1, 1);
      }
  }
  ```

  `@BeforeEach` (JUnit4 @Before)

  각 메서드 실행 전 실행되는 메서드

  `@AfterEach`  (JUnit4 @After)

  각 메서드 실행 후 실행되는 메서드

  `@BeforeAll`  (JUnit4 @BeforeClass)

  클래스 내에서 가장 먼저 실행되는 메서드.

  `@AfterAll`

  클래스내에서 마지막으로 실행되는 메서드

  `@TestInstance`

  클래스에 대한 테스트 인스턴스 수명주기 를 구성하는 데 사용됩니다.

  PER_CLASS

  - Test 클래스 당 하나의 인스턴스가 생성.
  - @BeforeAll, @AfterAll 함수를 `static 없이 사용 가능함`.

  PER_CLASS 

  - Test 메서드 당 하나의 인스턴스가 생성.

  ```java
  @SpringBootTest(classes = SampleTestInstance.class)
  @TestInstance(TestInstance.Lifecycle.PER_CLASS)
  public class SampleTestInstance {
      @BeforeAll
      public void beforeAllTest() {
          System.out.println(">>> beforeAllTest called");
      }
  
      @BeforeEach
      public void beforeEachTest() {
          System.out.println(">>> beforeEachTest called");
      }
  
      @Test
      void test1() {
          System.out.println(">>> test1 called");
          Assertions.assertEquals(1, 1);
      }
  
      @Test
      void test2() {
          System.out.println(">>> test2 called");
          Assertions.assertEquals(1, 1);
      }
  
      @AfterEach
      public void afterEachTest() {
          System.out.println(">>> afterEachTest called");
      }
  
      @AfterAll
      public void afterAllTest() {
          System.out.println(">>> afterAllTest called");
      }
  }
  ```

  `@DisplayName`

  테스트 클래스 또는 메서드에 대한 사용자 지정 표시 이름을 선언합니다.

  ```java
  @Test
  @DisplayName("테스트 이름")
  public void testMethod() {
  	Assertions.assertEquals(1, 1);
  }
  ```

  `@DisplayNameGeneration`

  클래스에 대한 사용자 지정 표시 이름 생성기 를 선언합니다.

  Class에 Annotation을 작성.

   @DisplayName보다 우선순위가 낮음.

  | Standard            | Matches the standard display name generation behavior in place since JUnit Jupiter 5.0 was released. |
  | ------------------- | ------------------------------------------------------------ |
  | Simple              | Removes trailing parentheses for methods with no parameters. |
  | ReplaceUnderscores  | Replaces underscores with spaces.                            |
  | IndicativeSentences | Generates complete sentences by concatenating the names of the test and the enclosing classes. |

  ```java
  @SpringBootTest(classes=SampleDisplayGeneration.class)
  @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
  public class SampleDisplayGeneration {
  
      @Test
      void test_if_equals() {
          Assertions.assertEquals(1,1);
      }
  
      @Test
      @DisplayName("DisplayName 우선 적용")
      void test_if_true() {
          Assertions.assertTrue(true);
      }
  }
  ```

  ![image-20210322165405274](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210322165405274.png)

  `@Nested`

  주석이 달린 클래스가 정적이 아닌 중첩 테스트 클래스 임을 나타냅니다 . @BeforeAll및 @AfterAll방법은 직접 사용할 수 없습니다 @Nested은 "당 클래스"를 제외 테스트 클래스 테스트 인스턴스 라이프 사이클이 사용.

  

  `@Tag` (JUnit @Categories)

  클래스 또는 메서드 수준에서 테스트 필터링을위한 태그 를 선언하는 데 사용.

  ```java
  import org.junit.jupiter.api.Tag;
  import org.junit.jupiter.api.Test;
  
  import java.lang.annotation.ElementType;
  import java.lang.annotation.Retention;
  import java.lang.annotation.RetentionPolicy;
  import java.lang.annotation.Target;
  
  @Target({ ElementType.TYPE, ElementType.METHOD })
  @Retention(RetentionPolicy.RUNTIME)
  @Tag("moduleA")
  @Test
  public @interface ModuleA {
  }
  ```

  ModuleA 처럼 ModuleB, ModuleC Custom Annotation를 만든 후 아래와 같은 코드를 작성한다.

  ```java
  package com.example.tdd1;
  
  import org.junit.jupiter.api.*;
  import org.springframework.boot.test.context.SpringBootTest;
  
  @SpringBootTest
  public class SampleDefault {
      UserDto user;
      
      @BeforeEach
      public void beforeEachTest() {
          //각 @Test Method가 실행되기 전 실행 된다.
          user = UserDto.builder().userId("Java").userName("World").build();
      }
  
      @ModuleA
      @DisplayName("assertTrue 테스트")
      public void assertTrueTest() {
          Assertions.assertTrue(user.getUserId().equalsIgnoreCase("java"), "userId가 java이다.");
  
      }
      
      @ModuleB
      @DisplayName("assertFalse 테스트")
      void assertFalseTest() {
          Assertions.assertFalse(user.getUserName().length() < 5, "userName이 5자리 보다 작다.");
      }
  
      @ModuleC
      @DisplayName("assertEquals 테스트")
      void assertEqualsTest() {
          Assertions.assertEquals(user.getUserId(), "Java");
          Assertions.assertEquals(user.getUserName(), "World", "userName이 World가 아니다.");
      }
  }
  ```

  `Tags 필터링 방법`

  Test를 IntelliJ IDEA로 할 경우 Settings-Build...-Build Tools-Gradle에서 Run tests using을 IntelliJ IDEA로 선택하고 Run/Debug Configuration에서 Tags를 선택하고 [Expression](https://junit.org/junit5/docs/current/user-guide/#running-tests-tag-expressions)을 입력한다.

  ![image-20210321164718225](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210321164718225.png)

  ![image-20210321165223878](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210321165223878.png)

  ![image-20210321165908669](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210321165908669.png)

  한 가지 주의할 점은 IntelliJ 단축키 `Ctrl`+`Shift`+`F10`으로 실행할 경우 설정한 Tags가 실행되는게 아니라 해당 클래스가 Test Run되기 때문에 우측 상단에 설정한 Tags인지 확인 해야 한다. 그래서 설정한 Tags로 계속 테스트할 때는 `Shift`+`F10` 단축키로 실행해야 한다.

  ![image-20210321170152274](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210321170152274.png)

  ![image-20210321170232396](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210321170232396.png)

  

  Gradle로 Tags를 설정하여 Test하는 방법은 Settings-Build...-Build Tools-Gradle에서 Run tests using을 Gradle로 하고 build.gradle에 아래 내용을 추가한다. 

  ```gradle
  test {
      useJUnitPlatform {
          includeTags 'moduleA', 'moduleB'
          // excludeTags 'moduleC'
          includeEngines 'junit-jupiter'
          // excludeEngines 'junit-vintage'
      }
  }
  ```

  그리고 Run Configurations에서 Gradle을 추가하고 설정 정보를 넣은 다음 실행하면 된다.

  ![image-20210321170916528](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210321170916528.png)

  ![image-20210321171028397](https://cdn.jsdelivr.net/gh/donghyeok-dev/donghyeok-dev.github.io@master/assets/images/posts/image-20210321171028397.png)

  개인적으로 IntelliJ IDEA를 선호하는데 그 이유는 결과 창에 상세 항목 정보가 안나오고 실행된 수만 나오기 때문이다. (대신 위 그림에서 보라색 Gradle Icon을 클릭하면 웹에서 상세 정보를 볼 수는 있다.)

  

  `@Disabled` (JUnit @Ignore)

  테스트 클래스 또는 메서드 를 비활성화.

  `@Timeout`

  실행이 주어진 기간을 초과하는 경우 테스트, 테스트 팩토리, 테스트 템플릿 또는 수명주기 메서드를 실패하는 데 사용.

  `@ExtendWith`

  확장을 선언적 으로 등록하는 데 사용.

  `@RegisterExtension`

  필드를 통해 프로그래밍 방식으로 확장 을 등록하는 데 사용 .

  `@TempDir`

  라이프 사이클 방법 또는 테스트 방법에서 필드 주입 또는 매개 변수 주입을 통해 임시 디렉토리 를 제공하는 데 사용. 

  

- JUnit5 User Guide : [https://junit.org/junit5/docs/current/user-guide](https://junit.org/junit5/docs/current/user-guide){:target="_blank"}

  

## 2. Mockito

- Mock: 진짜 객체와 비슷한 동작을 하지만 개발자가 직접 행동을 관리하는 객체.
- Mockito : Mock 객체를 쉽게 만들고 관리하고 검증할 수 있는 방법을 제공함.