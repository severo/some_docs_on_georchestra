# How is the configuration managed in geOrchestra?

## Using properties files inside Java and JSP files

Various modules, like header, extractorapp, mapfishapp, ... use the commons
module to use the properties, from the properties files, inside Java and JSP
code.

As seen from maven (`pom.xml` file), they depend on the `georchestra-commons`
artifact of the `org.georchestra` project:

```xml
    <!-- geOrchestra commons -->
    <dependency>
      <groupId>org.georchestra</groupId>
      <artifactId>georchestra-commons</artifactId>
      <version>${project.version}</version>
    </dependency>
```

As seen from Java code and the Spring framework, they manage a bean from the
class `org.georchestra.commons.configuration.GeorchestraConfiguration`:

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

- to be able to later access to the properties using the
  [`getProperty` method](https://github.com/georchestra/georchestra/blob/master/commons/src/main/java/org/georchestra/commons/configuration/GeorchestraConfiguration.java#L122):

  ```java
  georLdapadminPublicContextPath = georchestraConfiguration.getProperty("consolePublicContextPath");
  ```

GeoNetwork and GeoServer do not use the GeorchestraConfiguration class, but
implement almost the same code, to access to some properties in geOrchestra
properties files. In GeoNetwork, the properties can be overriden by
`${georchestra.datadir}/default.properties` and
`${georchestra.datadir}/geonetwork/geonetwork.properties` files, and the
priority order is the same. In particular, it allows to pass the `headerUrl` and
`headerHeight` to GeoNetwork. The same occurs to GeoServer, with the
`${georchestra.datadir}/default.properties` and
`${georchestra.datadir}/geoserver/geoserver.properties` files, and only for the
two `headerUrl` and `headerHeight` variables.

## Using properties files as placeholders inside Spring

Various modules use the properties files inside the XML Spring files (web.xml,
ws-servlet.xml, ...):

- they use the `context:property-placeholder` tag to define where to load the
  properties from (see
  [#2123](https://github.com/georchestra/georchestra/issues/2123) for a
  discussion about the priority order between the files):

  ```xml
  <context:property-placeholder
    location="file:${georchestra.datadir}/analytics/analytics.properties"
    ignore-resource-not-found="true" ignore-unresolvable="true" order="1" />
  ```

- they use the `${...}` placeholders to pass properties or constructor arguments
  to the beans:

  - properties:

    ```xml
    <bean id="jpaDataSource" class="org.apache.commons.dbcp.BasicDataSource" depends-on="waitForDb">
        <property name="url" value="${dlJdbcUrlOGC}"/>
        <property name="driverClassName" value="org.postgresql.Driver"/>
        <property name="testOnBorrow" value="true"/>
        <property name="validationQuery" value="select 1 as dbcp_connection_test"/>
        <property name="poolPreparedStatements" value="true"/>
        <property name="maxOpenPreparedStatements" value="-1"/>
        <property name="defaultReadOnly" value="false"/>
        <property name="defaultAutoCommit" value="true"/>
    </bean>
    ```

  - constructor arguments:

        ```xml
          <bean id="statisticsController" class="org.georchestra.analytics.StatisticsController">
          <constructor-arg name="localTimezone" value="${localTimezone}"/>
        </bean>
        ```

    Note that there could be other uses, managing the Spring Java classes
    PropertyPlaceholderConfigurer, PropertySourcesPlaceholderConfigurer and
    PropertyOverrideConfigurer, but they are used only in upstream GeoServer and
    GeoNetwork (but not for geOrchestra properties files), not in the other
    geOrchestra modules.
