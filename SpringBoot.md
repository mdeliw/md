[toc]

# SpringBoot

## Creating the Application

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class DemoApp {...}
```

- `@Configuration` makes this class as Java configuration class. In some cases `@SpringBootConfiguration` is used to identify this as a Spring Boot based application. 
- `@ComponentScan` pick up other components
- `@EnableAutoConfiguration` allows Spring Boot to do its auto-configurations.

>  The three above can be replaced with `@SpringBootApplication`

## Configure a Bean

- Use `@SpringBootApplication` 
- You can also use an additional `@ComponentScan` in case packages not in the default scan have components. 
- `@Bean`  is used when we need more control over the construction of the bean. Typically a method returning the bean is annotated. The method may have arguments that are injected / resolved by other beans in the application context.

## Externalize Properties

- Command line args
- `application.properties` inside the application. 
- `application-{profile}.properties` inside the application.
- Inside the application is `/src/main/resources`.

```bash
java -jar <file.jar> --spring.profiles.active=<profile>
java -jar <file.jar> --prop_name1=<value> --prop_name2=<value>
```

- Use a different configuration file in code or command line

```java
@PropertySource("classpath:some-other.properties")
```

```bash
java -jar <file.jar> --spring.config.name=<some-other.properties>,<yet-another.properties>
```

## Testing

- Integrates with Mockito, Spring MockMvc, and WebDriver.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = MainClass.Class)
public class MainClassTests {..}

@MockBean
```

