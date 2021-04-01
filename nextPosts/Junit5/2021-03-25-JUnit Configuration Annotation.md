---
layout: customPost
title:  "Mockito"
categories: 
  - Spring
  - Test
---



@SpringJUnitWebConfig는 메타 Annotation입니다.

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration
@WebAppConfiguration
@Documented
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface SpringJUnitWebConfig {
...
```

단위 테스트에서는 @SpringJUnitWebConfig를 Annotated해서 사용해야 합니다. 그 이유는 @SpringBootTest나 @WebMvcTest는 실제 Spring이 기동되기 때문에 테스트 속도가 느립니다.

@SpringBootTest의 경우 일반적인 테스트로 slicing을 전혀 사용하지 않기 때문에 전체 응용 프로그램 컨텍스트를 시작하기 때문에 전체 빈 로드&주입 시간이 걸려 테스트 속도가 느립니다.

@WebMvcTest의 경우는 Controller를 테스트하고 모의 객체를 사용하기 때문에 나머지 필요한 bean을 직접 세팅해줘야 합니다.



 mockMvc.perform() 메소도로 테스트코드에서 Product Controller의 메소드를 호출 했을때 Mock객체 때문에 통합 테스트할 때 사용합니다.







