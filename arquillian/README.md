#### Basic arquillian instaliation

##### 1) Create arquillian.xml in the resource root

```xml
<?xml version="1.0" encoding="UTF-8"?>
<arquillian xmlns="http://jboss.org/schema/arquillian"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://jboss.org/schema/arquillian http://jboss.org/schema/arquillian/arquillian_1_0.xsd">

    <defaultProtocol type="Servlet 3.0" />
    <container qualifier="jbossas7" default="true">
        <configuration>
            <property name="jbossHome">/usr/local/share/jboss-eap-6.1.1</property>
            <!--<property name="javaVmArguments">-Xmx512m -XX:MaxPermSize=512m -Xdebug -Xnoagent -Djava.compiler=NONE -agentlib:jdwp=transport=dt_socket,address=8787,server=y,suspend=y</property>-->
            <property name="serverConfig">standalone-full.xml</property>
            <property name="javaVmArguments">-Xmx512m -XX:MaxPermSize=512m -Dcom.sun.jersey.server.impl.cdi.lookupExtensionInBeanManager=true</property>
            <property name="managementAddress">127.0.0.1</property>
            <property name="managementPort">56999</property>
        </configuration>
    </container>

</arquillian>
```

##### 2) Add maven dependencies 
```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.jboss.shrinkwrap.resolver</groupId>
                <artifactId>shrinkwrap-resolver-bom</artifactId>
                <version>3.0.0-alpha-1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.jboss.arquillian</groupId>
                <artifactId>arquillian-bom</artifactId>
                <version>${version.arquillian}</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.jboss.as</groupId>
            <artifactId>jboss-as-arquillian-container-managed</artifactId>
            <version>7.1.1.Final</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.jboss.arquillian.protocol</groupId>
            <artifactId>arquillian-protocol-servlet</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.jboss.arquillian.junit</groupId>
            <artifactId>arquillian-junit-container</artifactId>
            <scope>test</scope>
        </dependency>
```

##### 3) Add test to `test` package 

```java
@RunWith(Arquillian.class)
public class AccountTest {

    @Deployment
    public static JavaArchive createTestArchive() {
        return ShrinkWrap.create(JavaArchive.class).addClasses(DateServiceImpl.class, DateService.class, Date.class);
    }

    @EJB
    private DateService dateService;

    @Test
    public void shouldBeAbleToInjectEJB() throws Exception {
        Assert.assertNotNull(dateService);
    }
}
```

##### 4) Run with Junit in test phase of maven lifecycle
```sh
mvn test
```
