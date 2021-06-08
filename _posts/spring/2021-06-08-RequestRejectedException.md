---
layout: customPost
title:  "RequestRejectedException"
categories: 
  - Spring
  - Exception
---
## RequestRejectedException

```java
20210608 13:30:19.656 [http-nio-8080-exec-4] ERROR o.a.c.c.C.[.[.[.[dispatcherServlet] - Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception 
org.springframework.security.web.firewall.RequestRejectedException: The request was rejected because the URL contained a potentially malicious String ";"
	at org.springframework.security.web.firewall.StrictHttpFirewall.rejectedBlocklistedUrls(StrictHttpFirewall.java:456)
```

request url에 세미콜론(;) 또는 슬래쉬(/)가 붙으면 security firewall에서 요청을 거부합니다.

org.springframework.security.web.firewall.StrictHttpFirewall 클래스에 블랙리스트로 관리하고 있으며, 이를 허용하고 싶다면 다음과 같이 처리할 수도 있습니다.

```java
@Bean
public StrictHttpFirewall httpFirewall() {
    StrictHttpFirewall firewall = new StrictHttpFirewall();
    firewall.setAllowBackSlash(true);
    firewall.setAllowSemicolon(true);
    return firewall;
}
```



블랙리스트를 유지하면서 모니터링 등이 이유로 로그만 제외하고 싶다면 아래와 같은 클래스를 추가합니다.

```java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class RequestRejectedExceptionFilter extends GenericFilterBean {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        try {
            chain.doFilter(request, response);
        } catch (RequestRejectedException e) {
            // firewall에서 요청이 거부되면 인덱스 페이지로 리다이렉트함.
            ((HttpServletResponse)response).sendRedirect("/");
        }
    }
}
```

