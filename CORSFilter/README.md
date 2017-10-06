CORSFilter [ juggling pebbles ]
--------------------------
Whenever the request are from different domain we face CORS Exceptions, Cross Origin Resource Sharing is a mechanism supported by W3C to enable cross origin requests in web-browsers. CORS requires support from both browser and server to work.

Error
```javascript
Failed to load http://serverNAme/webserver/method: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://serverNAme:8080' is therefore not allowed access.
```

1. Adding <filter> in Tomcat
--------
The easiest way is to add the filter provided by Apache on tomcat server's web.xml. The path to file is located on apache-tomcat-8.0.30/conf/web.xml. Make sure to add the <filter> before ending </web-app> tag.

```xml
<filter>
  <filter-name>CorsFilter</filter-name>
  <filter-class>org.apache.catalina.filters.CorsFilter</filter-class>
  <init-param>
    <param-name>cors.allowed.origins</param-name>
    <param-value>*</param-value>
  </init-param>
  <init-param>
    <param-name>cors.allowed.methods</param-name>
    <param-value>GET,POST,HEAD,OPTIONS,PUT</param-value>
  </init-param>
  <init-param>
    <param-name>cors.allowed.headers</param-name>
    <param-value>Content-Type,X-Requested-With,accept,Origin,Access-Control-Request-Method,Access-Control-Request-Headers</param-value>
  </init-param>
  <init-param>
    <param-name>cors.exposed.headers</param-name>
    <param-value>Access-Control-Allow-Origin,Access-Control-Allow-Credentials</param-value>
  </init-param>
  <init-param>
    <param-name>cors.support.credentials</param-name>
    <param-value>true</param-value>
  </init-param>
  <init-param>
    <param-name>cors.preflight.maxage</param-name>
    <param-value>10</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>CorsFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```
This might work if your server is not hosting other services which are developed using sever side CORSfilter handling ( we will discuss below ). If we services are developed implementing server side CORS filter handling then adding the above filter in tomcat will throw `Duplicate Header Exception`.

2. Adding CORSFilter for Jersey services
------
```java
package com.company.application.filters;

import com.sun.jersey.spi.container.ContainerRequest;
import com.sun.jersey.spi.container.ContainerResponse;
import com.sun.jersey.spi.container.ContainerResponseFilter;

public class CORSFilter implements ContainerResponseFilter {

	@Override
	public ContainerResponse filter(ContainerRequest request,
			ContainerResponse response) {

		response.getHttpHeaders().add("Access-Control-Allow-Origin", "*");
		response.getHttpHeaders().add("Access-Control-Allow-Headers",
				"origin, content-type, accept, authorization, X-Request-With");
		response.getHttpHeaders().add("Access-Control-Allow-Credentials",
				"true");
		response.getHttpHeaders().add("Access-Control-Allow-Methods",
				"GET, POST, PUT, DELETE, OPTIONS, HEAD");

		return response;
	}
}
```
We don't need to make any changes in the web.xml

3. Adding CORSFilter using Servlet  
--------
```java
package com.company.application.filters;

import javax.servlet.*;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class CORSFilter implements Filter {

	@Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse httpResponse = (HttpServletResponse) servletResponse;
        httpResponse.addHeader("Access-Control-Allow-Origin", "*");
        httpResponse.addHeader("Access-Control-Allow-Headers",
				"origin, content-type, accept, authorization, X-Request-With");
        httpResponse.addHeader("Access-Control-Allow-Credentials",
				"true");
        httpResponse.addHeader("Access-Control-Allow-Methods",
				"GET, POST, PUT, DELETE, OPTIONS, HEAD");
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {

    }
}
```
and now we need to add <filter> in web.xml as mentioned below, Please note this will still cause `Duplicate Header` exception as we have added the custom filter in web.xml.

```xml
<filter>
    <filter-name>CorsFilter</filter-name>
    <filter-class>com.company.application.filters.CORSFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>CorsFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

4. Adding CORSFilter for Glassfish services
------------
```java
package com.company.application.filters;

import javax.ws.rs.ext.Provider;
import javax.ws.rs.container.ContainerRequestContext;
import javax.ws.rs.container.ContainerResponseContext;
import javax.ws.rs.container.ContainerResponseFilter;

/**
 * Allow the system to serve xhr level 2 from all cross domain site
 */
@Provider
public class CORSFilter implements ContainerResponseFilter {

	@Override
    public void filter(ContainerRequestContext creq, ContainerResponseContext cres) {
        cres.getHeaders().add("Access-Control-Allow-Origin", "");
        cres.getHeaders().add("Access-Control-Allow-Headers", "");
        cres.getHeaders().add("Access-Control-Allow-Credentials", "");
        cres.getHeaders().add("Access-Control-Allow-Methods", "");
        cres.getHeaders().add("Access-Control-Max-Age", "");
    }
}
```
web.xml
```xml
<filter>
  <param-name>org.glassfish.jersey.container.ContainerResponseFilters</param-name>
<param-value>
    com.company.application.filters.CORSFilter
</param-value>
</filter>
```
