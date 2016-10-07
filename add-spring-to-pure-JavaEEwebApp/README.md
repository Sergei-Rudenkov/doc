### Adding Spring Support to existing Java EE application 

Primary reason of doing this was Jersey(JAX-RS) doesn't support EJB injection into classes proxied by Jersey. I substetuded it with Spring DI. 

##### 1) Add dependencies to pom.xml
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.3.3.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>4.3.3.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.3.3.RELEASE</version>
</dependency>
```

##### 2) Make your Context listener Spring aware by adding extension:

```java
public class YourContextListener extends org.springframework.web.context.ContextLoaderListener
```


##### 3) Add to web.xml

```xml
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>WEB-INF/servlet-context.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/spring/*</url-pattern>
</servlet-mapping>
```
##### 4) Add to WEB-INF `servlet-context.xml` in order to create Spring beans from EJB by jndi names 

<beans>

    <context:component-scan base-package="com..." />

    <mvc:annotation-driven />

    <jee:remote-slsb id="Local"
                     jndi-name="java:app/Local"
                     business-interface="..."
                     home-interface="..."
                     cache-home="false" lookup-home-on-startup="false"
                     refresh-home-on-connect-failure="true" />

</beans>

