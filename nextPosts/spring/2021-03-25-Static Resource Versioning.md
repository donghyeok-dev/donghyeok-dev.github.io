---
layout: customPost
title:  "Static Resource Versioning"
categories: 
  - Spring
---



Static Resource Versioning? css, js, image 등 정적 리소스 파일은 브라우저에 캐싱되는데, 변경된 정적 리소스를 적용하기 위해서는 브라우저에 캐싱된 정보를 삭제하거나 새로고침 등의 작업이 필요하다. 자동으로 캐싱된 정적 리소스를 갱신 해주기 위해서는 정적 리소스의 이름을 변경하는 방법을 많이 사용한다.

예를들어 /css/hello.css 파일에 /css/hello.css?version=1  과 같은 형식으로 사용한다.

하지만 version에 대한 변수 값 처리 작업을 수동으로 해야기 때문에 불편한점이 있다.

이런 작업을 자동으로 할 수 있는 방법이 있는데 spring에서 제공하는 VersionResourceResolver를 사용하는 것이다. VersionResourceResolver는 기본적으로 파일의 이름을 MD5형식으로 view에게 전달한다.

/css/hello-fca09cf9737eb04ff61e6efcb2b89e28.css  실제 파일은 hello.css이지만 VersionResourceResolver가 정적 리소스의 이름을 변경해서 전달한다. 

hello.css의 파일에 변경된 내용이 없다면 동일한 이름으로 전달되기 때문에 브라우저에 캐싱 장점을 그대로 사용할 수 있고, hello.css의 파일 내용이 변경 되면 다른 이름이 전달되기 때문에 브라우저는 새로운 파일을 캐싱하게 된다.



적용방법은 아래와 같다.

첫번째 application.properties에 다음 내용 추가.

```
spring.resources.chain.enabled=true
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
```

두번째 Java Configure 추가.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    //...

	@Bean
	public ResourceUrlEncodingFilter resourceUrlEncodingFilter() {
		return new ResourceUrlEncodingFilter();
	}

	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/static/**")
				.addResourceLocations("classpath:/static/")
				.setCacheControl(CacheControl.maxAge(365, TimeUnit.DAYS))
				.resourceChain(false)
				.addResolver(new VersionResourceResolver().addContentVersionStrategy("/**"))
				.addTransformer(new CssLinkResourceTransformer()); //css 파일 안에도 적용.
	}
}
```

세번째 view 파일의 정적 리소스 정의 부분 수정.

```java
<link href="<c:url value="/css/hello.css" />" rel="stylesheet" >
```

