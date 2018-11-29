# How is the configuration managed in geOrchestra?

## Commons

Various modules, like header, extractorapp, mapfishapp, ... use the commons module to manage the properties files.

As seen from maven (`pom.xml` file), they depend on the `georchestra-commons` artifact of the `org.georchestra` project:

```xml
    <!-- geOrchestra commons -->
    <dependency>
      <groupId>org.georchestra</groupId>
      <artifactId>georchestra-commons</artifactId>
      <version>${project.version}</version>
    </dependency>
```

As seen from Java code and the Spring framework, they manage a bean from the class `org.georchestra.commons.configuration.GeorchestraConfiguration`:

```xml
    <bean id="georchestraConfiguration" class="org.georchestra.commons.configuration.GeorchestraConfiguration">
        <constructor-arg value="header" />
    </bean>
```

and use it inside the code:

- importing the class:

  - in Java:

  ```java
  import org.georchestra.commons.configuration.GeorchestraConfiguration;
  ```

  - in JSP files:

  ```jsp
  <%@ page import="org.georchestra.commons.configuration.GeorchestraConfiguration" %>
  ```

- and obtaining access to the bean:

  - in Java / Spring, using an annotation:

  ```
  @Autowired
  private GeorchestraConfiguration georchestraConfiguration;
  ```

  - in JSP:

  ```java
  ApplicationContext ctx = RequestContextUtils.getWebApplicationContext(request);
  GeorchestraConfiguration georchestraConfiguration = ctx.getBean(GeorchestraConfiguration.class);
  ```

- to be able to later access to the properties using the [`getProperty` method](https://github.com/georchestra/georchestra/blob/master/commons/src/main/java/org/georchestra/commons/configuration/GeorchestraConfiguration.java#L122):

  ```java
  georLdapadminPublicContextPath = georchestraConfiguration.getProperty("consolePublicContextPath");
  ```
