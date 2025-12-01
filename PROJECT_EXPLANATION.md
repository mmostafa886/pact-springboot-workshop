# Comprehensive Project Explanation

## Table of Contents
- [1. What is Spring Boot?](#1-what-is-spring-boot)
- [2. Why Do We Need Annotations Like @Autowired?](#2-why-do-we-need-annotations-like-autowired)
- [3. All Annotations Used in This Project](#3-all-annotations-used-in-this-project)
- [4. The Concept of Pact Contract Testing](#4-the-concept-of-pact-contract-testing)
- [5. Project Structure](#5-project-structure)
- [6. Summary](#6-summary)

---

## 1. What is Spring Boot?

**Spring Boot** is an opinionated framework built on top of the Spring Framework that simplifies the creation of production-ready applications. It was created by Pivotal (now part of VMware) to address the complexity and configuration overhead of traditional Spring applications.

### The Problem Before Spring Boot

Traditional Spring applications required:

**1. Extensive XML Configuration**
```xml
<!-- applicationContext.xml - Traditional Spring -->
<beans xmlns="http://www.springframework.org/schema/beans">
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
        <property name="driverClassName" value="org.h2.Driver"/>
        <property name="url" value="jdbc:h2:mem:testdb"/>
        <property name="username" value="sa"/>
        <property name="password" value=""/>
    </bean>

    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="packagesToScan" value="com.example.model"/>
        <!-- ... dozens more properties ... -->
    </bean>

    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
    </bean>

    <!-- ... hundreds of lines of XML ... -->
</beans>
```

**2. Manual Dependency Management**
```xml
<!-- pom.xml - Traditional Spring -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.3.10</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.10</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.3.10</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.10</version>
    </dependency>
    <!-- ... manually managing 20+ dependencies ... -->
</dependencies>
```

**3. Complex Server Deployment**
- Package application as WAR file
- Install and configure Tomcat/JBoss/WebLogic separately
- Deploy WAR to external server
- Manage server configuration files
- Handle classpath conflicts

**4. Boilerplate Configuration Code**
```java
// Traditional Spring - WebConfig.java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        registry.viewResolver(resolver);
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/resources/");
    }

    @Bean
    public DataSource dataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:mem:testdb");
        dataSource.setUsername("sa");
        dataSource.setPassword("");
        return dataSource;
    }

    // ... 100+ lines of configuration code ...
}
```

### How Spring Boot Solves These Problems

**Spring Boot** applies the **"Convention over Configuration"** principle:

#### 1. Auto-Configuration Magic

**Before (Traditional Spring):**
```java
// Manually configure dozens of beans
@Configuration
public class AppConfig {
    @Bean
    public DataSource dataSource() { /* ... */ }

    @Bean
    public EntityManagerFactory entityManagerFactory() { /* ... */ }

    @Bean
    public PlatformTransactionManager transactionManager() { /* ... */ }

    @Bean
    public RestTemplate restTemplate() {
        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
        factory.setConnectTimeout(5000);
        factory.setReadTimeout(5000);
        return new RestTemplate(factory);
    }
    // ... dozens more beans ...
}
```

**After (Spring Boot):**
```java
// Spring Boot auto-configures everything
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    // Optional: Override only what you need
    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder.build(); // That's it!
    }
}
```

Spring Boot detects:
- H2 on classpath → Auto-configures in-memory database
- `spring-boot-starter-web` → Auto-configures Tomcat, Spring MVC, JSON serialization
- JPA on classpath → Auto-configures EntityManager, TransactionManager
- Thymeleaf on classpath → Auto-configures template engine

#### 2. Starter Dependencies

**Before (Traditional Spring):**
```xml
<!-- Managing 15+ dependencies for a web app -->
<dependencies>
    <dependency><groupId>org.springframework</groupId><artifactId>spring-core</artifactId><version>5.3.10</version></dependency>
    <dependency><groupId>org.springframework</groupId><artifactId>spring-web</artifactId><version>5.3.10</version></dependency>
    <dependency><groupId>org.springframework</groupId><artifactId>spring-webmvc</artifactId><version>5.3.10</version></dependency>
    <dependency><groupId>com.fasterxml.jackson.core</groupId><artifactId>jackson-databind</artifactId><version>2.12.3</version></dependency>
    <dependency><groupId>javax.servlet</groupId><artifactId>javax.servlet-api</artifactId><version>4.0.1</version></dependency>
    <!-- ... 10 more dependencies ... -->
</dependencies>
```

**After (Spring Boot):**
```xml
<!-- One starter includes everything needed for web apps -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

The `spring-boot-starter-web` includes:
- Spring MVC (web framework)
- Tomcat (embedded server)
- Jackson (JSON processing)
- Validation API
- All compatible versions

**Common Starters:**
- `spring-boot-starter-web`: Web applications
- `spring-boot-starter-data-jpa`: Database access with JPA
- `spring-boot-starter-security`: Authentication and authorization
- `spring-boot-starter-test`: Testing with JUnit, Mockito, AssertJ
- `spring-boot-starter-thymeleaf`: Template engine

#### 3. Embedded Servers

**Before (Traditional Spring):**
```bash
# Complex deployment process
1. Package application:
   mvn clean package
   # Produces: myapp.war

2. Install Tomcat:
   - Download Tomcat 9.0.x
   - Extract to /opt/tomcat
   - Configure server.xml, context.xml, tomcat-users.xml

3. Deploy:
   cp myapp.war /opt/tomcat/webapps/

4. Start server:
   /opt/tomcat/bin/startup.sh

5. Check logs:
   tail -f /opt/tomcat/logs/catalina.out
```

**After (Spring Boot):**
```bash
# Simple one-step execution
mvn spring-boot:run
# Application runs immediately with embedded Tomcat!

# Or as a JAR:
mvn package
java -jar target/myapp.jar
```

Spring Boot embeds Tomcat **inside the JAR**:
```
myapp.jar
├── BOOT-INF/
│   ├── classes/        (Your application code)
│   └── lib/            (Dependencies + embedded Tomcat)
├── META-INF/
└── org/springframework/boot/loader/  (Boot launcher)
```

#### 4. Production-Ready Features

Spring Boot includes **Spring Boot Actuator** for monitoring:

```java
// Just add dependency - no configuration needed
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**Instant endpoints:**
- `GET /actuator/health` - Application health status
- `GET /actuator/metrics` - Performance metrics
- `GET /actuator/info` - Application information
- `GET /actuator/env` - Environment properties
- `GET /actuator/loggers` - Log configuration

**Example response:**
```json
// GET /actuator/health
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "H2",
        "validationQuery": "SELECT 1"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 499963174912,
        "free": 91897569280,
        "threshold": 10485760
      }
    }
  }
}
```

#### 5. Externalized Configuration

**Before (Traditional Spring):**
```java
// Hardcoded or complex property loading
public class AppConfig {
    private static final String DB_URL = "jdbc:mysql://localhost:3306/mydb"; // Hardcoded!

    // Or complex PropertyPlaceholderConfigurer setup
}
```

**After (Spring Boot):**
```properties
# application.properties - automatically loaded
server.port=8080
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
serviceClients.products.baseUrl=http://localhost:9000
```

```java
// Automatically injected
@Value("${serviceClients.products.baseUrl}")
private String baseUrl;
```

**Profile-based configuration:**
```
application.properties          # Default
application-dev.properties      # Development
application-prod.properties     # Production
```

```bash
# Run with specific profile
java -jar app.jar --spring.profiles.active=prod
```

### Key Features Summary

| Feature | Traditional Spring | Spring Boot |
|---------|-------------------|-------------|
| **Configuration** | 100s of lines of XML/Java config | `@SpringBootApplication` |
| **Dependencies** | Manage 20+ individual dependencies | One starter dependency |
| **Server** | External Tomcat installation | Embedded in JAR |
| **Setup Time** | Hours to days | Minutes |
| **Deployment** | WAR to external server | Executable JAR |
| **Monitoring** | Manual setup | Built-in actuator |

### In This Project

Spring Boot manages:
- **Web application infrastructure**: Embedded Tomcat server, Spring MVC framework
- **Dependency injection container**: IoC container managing all beans
- **REST endpoints**: `@RestController` components with JSON serialization
- **Testing infrastructure**: JUnit 5, MockMvc, test slices
- **Database connections**: H2 DataSource, JPA EntityManager (for the provider)
- **Security**: Bearer token authentication filter
- **Configuration**: Properties injection via `@Value`

---

## 2. Why Do We Need Annotations Like @Autowired?

### The Evolution of Object Creation in Java

To understand why annotations are crucial, let's see how Java applications evolved:

#### Era 1: Manual Object Creation (The Dark Ages)

**The Traditional Approach:**

```java
// UserController.java
public class UserController {
    private UserService userService;

    public UserController() {
        // Manually create all dependencies
        DatabaseConnection dbConnection = new DatabaseConnection("localhost", 5432, "mydb", "user", "pass");
        UserRepository userRepository = new UserRepositoryImpl(dbConnection);
        EmailService emailService = new SmtpEmailService("smtp.gmail.com", 587);
        Logger logger = new FileLogger("/var/log/app.log");
        this.userService = new UserService(userRepository, emailService, logger);
    }

    public User getUser(Long id) {
        return userService.findById(id);
    }
}
```

**Real-World Example:**

```java
// ProductServiceClient.java - Without Dependency Injection
public class ProductServiceClient {
    private RestTemplate restTemplate;
    private String baseUrl;
    private int connectTimeout;
    private int readTimeout;

    public ProductServiceClient() {
        // Hardcoded configuration
        this.baseUrl = "http://localhost:9000";
        this.connectTimeout = 5000;
        this.readTimeout = 5000;

        // Manual object creation
        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
        factory.setConnectTimeout(this.connectTimeout);
        factory.setReadTimeout(this.readTimeout);
        this.restTemplate = new RestTemplate(factory);
    }

    public List<Product> fetchProducts() {
        String url = baseUrl + "/products";
        ProductServiceResponse response = restTemplate.getForObject(url, ProductServiceResponse.class);
        return response.getProducts();
    }
}
```

**Problems with Manual Creation:**

**1. Tight Coupling**
```java
// ProductServiceClient is TIGHTLY COUPLED to RestTemplate implementation
public class ProductServiceClient {
    private RestTemplate restTemplate = new RestTemplate(); // Cannot change!

    // What if you want to use WebClient instead?
    // You must modify THIS class - violates Open/Closed Principle
}
```

**2. Testing Nightmare**
```java
// ProductServiceClientTest.java
public class ProductServiceClientTest {
    @Test
    public void testFetchProducts() {
        ProductServiceClient client = new ProductServiceClient();

        // How do you test this without making REAL HTTP calls?
        // You can't! The RestTemplate is hardcoded inside.
        List<Product> products = client.fetchProducts();

        // This test will:
        // ❌ Make real network calls (slow, unreliable)
        // ❌ Require the provider service running (complex setup)
        // ❌ Fail if network is down (flaky)
    }
}
```

**3. Configuration Nightmare**
```java
// Different environments need different configs
public class ProductServiceClient {
    public ProductServiceClient() {
        // Which URL to use?
        // Development: http://localhost:9000
        // Staging: http://staging-api.company.com
        // Production: http://api.company.com

        // Hardcoded = disaster!
        this.baseUrl = "http://localhost:9000"; // Won't work in production!
    }
}
```

**4. Code Duplication**
```java
// Multiple classes need RestTemplate
public class ProductServiceClient {
    private RestTemplate restTemplate = new RestTemplate(); // Duplicate!
}

public class OrderServiceClient {
    private RestTemplate restTemplate = new RestTemplate(); // Duplicate!
}

public class PaymentServiceClient {
    private RestTemplate restTemplate = new RestTemplate(); // Duplicate!
}

// Creating 3 RestTemplate instances when 1 shared instance would work!
```

**5. Complex Dependency Graphs**
```java
// Controller depends on Service
public class ProductCatalogueController {
    private ProductServiceClient client;

    public ProductCatalogueController() {
        // Service depends on RestTemplate
        RestTemplate restTemplate = new RestTemplate();
        this.client = new ProductServiceClient(restTemplate);
        // Now what if ProductServiceClient needs MORE dependencies?
        // We have to change the controller!
    }
}

// Real-world example with deep dependency tree:
public class OrderController {
    public OrderController() {
        // Create 10+ objects in correct order
        DatabaseConnection db = new DatabaseConnection(...);
        UserRepository userRepo = new UserRepositoryImpl(db);
        ProductRepository productRepo = new ProductRepositoryImpl(db);
        InventoryService inventoryService = new InventoryService(productRepo);
        PaymentGateway paymentGateway = new StripeGateway(...);
        EmailService emailService = new SmtpEmailService(...);
        SmsService smsService = new TwilioSmsService(...);
        NotificationService notificationService = new NotificationService(emailService, smsService);
        OrderService orderService = new OrderService(userRepo, productRepo, inventoryService, paymentGateway, notificationService);
        // Exhausting! And fragile!
    }
}
```

**6. Lifecycle Management**
```java
public class DatabaseConnection {
    private Connection connection;

    public DatabaseConnection() {
        this.connection = DriverManager.getConnection(...);
    }

    // Who closes the connection?
    // When?
    // What if multiple objects use the same connection?
    // Memory leaks everywhere!
}
```

---

### Era 2: Factory Pattern (Better, But Still Tedious)

```java
// ServiceFactory.java
public class ServiceFactory {
    private static RestTemplate restTemplate;

    public static synchronized RestTemplate getRestTemplate() {
        if (restTemplate == null) {
            restTemplate = new RestTemplate();
        }
        return restTemplate;
    }

    public static ProductServiceClient getProductServiceClient() {
        return new ProductServiceClient(getRestTemplate());
    }
}

// Usage
public class ProductCatalogueController {
    private ProductServiceClient client = ServiceFactory.getProductServiceClient();
}
```

**Still problematic:**
- Factories need manual maintenance
- Testing still difficult (static methods)
- Configuration still hardcoded
- Boilerplate code everywhere

---

### Era 3: Dependency Injection with Annotations (Modern Solution)

**How Annotations Solve All These Problems:**

#### Problem 1: Tight Coupling → SOLVED

**Before:**
```java
public class ProductServiceClient {
    // Tightly coupled to RestTemplate
    private RestTemplate restTemplate = new RestTemplate();
}
```

**After:**
```java
@Service
public class ProductServiceClient {
    // Loosely coupled - Spring injects ANY RestTemplate bean
    @Autowired
    private RestTemplate restTemplate;
}

// Configuration
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        // Want to change implementation? Just change this bean!
        return new RestTemplate(); // Or WebClient, or MockRestTemplate, etc.
    }
}
```

**Want to switch implementations? Change only ONE place:**
```java
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        // Changed from RestTemplate to custom implementation
        return new CustomRestTemplateWithRetry(); // Changed once, affects everywhere!
    }
}
```

#### Problem 2: Testing Difficulties → SOLVED

**Before (Impossible to Test):**
```java
public class ProductServiceClient {
    private RestTemplate restTemplate = new RestTemplate(); // Can't mock!

    public List<Product> fetchProducts() {
        return restTemplate.getForObject(...); // Makes REAL HTTP calls
    }
}
```

**After (Easy to Test):**
```java
@Service
public class ProductServiceClient {
    @Autowired
    private RestTemplate restTemplate; // Can inject mock!

    public List<Product> fetchProducts() {
        return restTemplate.getForObject(...);
    }
}

// Test
@SpringBootTest
class ProductServiceClientTest {
    @MockBean  // Spring annotation - replaces real bean with mock
    private RestTemplate restTemplate;

    @Autowired
    private ProductServiceClient client;

    @Test
    void testFetchProducts() {
        // Mock the HTTP call - no real network needed!
        when(restTemplate.getForObject(anyString(), eq(ProductServiceResponse.class)))
            .thenReturn(new ProductServiceResponse(Arrays.asList(
                new Product(1L, "Test Product", "CREDIT_CARD")
            )));

        List<Product> products = client.fetchProducts();

        assertThat(products, hasSize(1));
        // ✅ Fast (milliseconds)
        // ✅ Reliable (no network)
        // ✅ Isolated (no external dependencies)
    }
}
```

#### Problem 3: Configuration Management → SOLVED

**Before (Hardcoded):**
```java
public class ProductServiceClient {
    private String baseUrl = "http://localhost:9000"; // Hardcoded!
}
```

**After (Externalized):**
```java
@Service
public class ProductServiceClient {
    @Value("${serviceClients.products.baseUrl}") // Injected from config!
    private String baseUrl;
}
```

**Configuration files:**
```properties
# application.properties (default)
serviceClients.products.baseUrl=http://localhost:9000

# application-dev.properties
serviceClients.products.baseUrl=http://dev-api.company.com

# application-prod.properties
serviceClients.products.baseUrl=http://api.company.com
```

**Run with different profiles:**
```bash
# Development
java -jar app.jar --spring.profiles.active=dev

# Production
java -jar app.jar --spring.profiles.active=prod
```

#### Problem 4: Code Duplication → SOLVED

**Before (Duplicate Objects):**
```java
public class ProductServiceClient {
    private RestTemplate restTemplate = new RestTemplate(); // Instance 1
}

public class OrderServiceClient {
    private RestTemplate restTemplate = new RestTemplate(); // Instance 2
}

public class PaymentServiceClient {
    private RestTemplate restTemplate = new RestTemplate(); // Instance 3
}
```

**After (Shared Singleton):**
```java
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate(); // Created ONCE by Spring
    }
}

@Service
public class ProductServiceClient {
    @Autowired
    private RestTemplate restTemplate; // Shared instance
}

@Service
public class OrderServiceClient {
    @Autowired
    private RestTemplate restTemplate; // Same instance!
}

@Service
public class PaymentServiceClient {
    @Autowired
    private RestTemplate restTemplate; // Same instance!
}
```

**Spring creates ONE RestTemplate and injects it everywhere:**
```
┌─────────────────────────────────────┐
│     Spring IoC Container            │
│                                     │
│  ┌──────────────┐                  │
│  │ RestTemplate │ (Single instance)│
│  │    Bean      │                  │
│  └──────┬───────┘                  │
│         │                           │
│         │ Injected into:            │
│         ├──> ProductServiceClient  │
│         ├──> OrderServiceClient    │
│         └──> PaymentServiceClient  │
└─────────────────────────────────────┘
```

#### Problem 5: Complex Dependency Graphs → SOLVED

**Before (Constructor Hell):**
```java
public class OrderController {
    public OrderController() {
        DatabaseConnection db = new DatabaseConnection(...);
        UserRepository userRepo = new UserRepositoryImpl(db);
        ProductRepository productRepo = new ProductRepositoryImpl(db);
        EmailService emailService = new SmtpEmailService(...);
        OrderService orderService = new OrderService(userRepo, productRepo, emailService);
        // 50 lines of constructor code!
    }
}
```

**After (Automatic Wiring):**
```java
@RestController
public class OrderController {
    @Autowired
    private OrderService orderService; // Spring figures out the dependency graph!

    // Spring automatically:
    // 1. Creates DatabaseConnection
    // 2. Creates UserRepository (injecting DatabaseConnection)
    // 3. Creates ProductRepository (injecting DatabaseConnection)
    // 4. Creates EmailService
    // 5. Creates OrderService (injecting all of the above)
    // 6. Injects OrderService into this controller
    // All automatic!
}
```

**Dependency Graph (Spring manages automatically):**
```
OrderController
    └── OrderService
            ├── UserRepository
            │       └── DatabaseConnection
            ├── ProductRepository
            │       └── DatabaseConnection (same instance!)
            └── EmailService
```

#### Problem 6: Lifecycle Management → SOLVED

**Before (Manual Management):**
```java
public class Application {
    public static void main(String[] args) {
        DatabaseConnection db = new DatabaseConnection();
        ProductRepository repo = new ProductRepositoryImpl(db);

        // Application runs...

        // Who closes the database connection?
        // When?
        // Memory leak!
    }
}
```

**After (Spring Manages Lifecycle):**
```java
@Service
public class DatabaseService {
    private Connection connection;

    @PostConstruct // Spring calls AFTER construction
    public void init() {
        this.connection = DriverManager.getConnection(...);
        System.out.println("Database connection opened");
    }

    @PreDestroy // Spring calls BEFORE destruction
    public void cleanup() {
        if (connection != null) {
            connection.close();
            System.out.println("Database connection closed");
        }
    }
}
```

**Spring automatically:**
1. Creates beans in correct order
2. Calls `@PostConstruct` after injection
3. Manages bean scope (singleton, prototype, request, session)
4. Calls `@PreDestroy` before shutdown
5. Handles circular dependencies

---

### What Are Annotations?

**Annotations are metadata** that provide information about code to:
- The compiler (compile-time)
- Build tools (build-time)
- Frameworks like Spring (runtime)

**Syntax:**
```java
@AnnotationName
@AnnotationWithValue("value")
@AnnotationWithMultipleValues(name = "John", age = 30)
```

**Built-in Java Annotations:**
```java
@Override          // Compiler checks method overrides parent
@Deprecated        // Marks code as obsolete
@SuppressWarnings  // Tells compiler to ignore warnings
```

**Spring Annotations (Runtime Metadata):**
```java
@Service           // "This class is a service bean"
@Autowired         // "Inject a dependency here"
@Value             // "Inject a property value here"
```

**How Spring Uses Annotations:**
```
┌────────────────────────────────────────┐
│  1. Application Startup                │
│                                        │
│  Spring scans all classes              │
│  Looking for annotations:              │
│  - @Service                            │
│  - @Controller                         │
│  - @Repository                         │
│  - @Component                          │
└────────────┬───────────────────────────┘
             │
             v
┌────────────────────────────────────────┐
│  2. Bean Creation                      │
│                                        │
│  For each annotated class:             │
│  - Create instance                     │
│  - Register in IoC container           │
│  - Assign bean name                    │
└────────────┬───────────────────────────┘
             │
             v
┌────────────────────────────────────────┐
│  3. Dependency Injection               │
│                                        │
│  For each @Autowired field:            │
│  - Find matching bean                  │
│  - Inject into field                   │
└────────────┬───────────────────────────┘
             │
             v
┌────────────────────────────────────────┐
│  4. Application Ready                  │
│                                        │
│  All beans created and wired           │
│  Application can handle requests       │
└────────────────────────────────────────┘
```

---

### Real-World Comparison

**Scenario: E-commerce application with 50 classes**

| Aspect | Manual Creation | With Annotations |
|--------|----------------|------------------|
| **Lines of boilerplate** | ~500 lines of object creation | ~50 annotations |
| **Configuration files** | 10+ XML files | 1-2 properties files |
| **Time to add new dependency** | Modify 5-10 classes | Add 1 annotation |
| **Test setup complexity** | 20+ lines per test | 3-5 lines per test |
| **Refactoring difficulty** | High (ripple effects) | Low (change once) |
| **Onboarding time** | Days (understand object graph) | Hours (understand annotations) |

---

### Dependency Injection Visualized

**Without DI (Manual):**
```
┌─────────────────────────────────┐
│     ProductServiceClient        │
│                                 │
│  new RestTemplate()   ─────┐   │
│  new Logger()        ──────┼───┼─> ProductServiceClient CONTROLS
│  new Config()       ───────┘   │   everything (tight coupling)
│                                 │
└─────────────────────────────────┘
```

**With DI (Spring):**
```
┌─────────────────────────────────────┐
│     Spring IoC Container            │
│                                     │
│  ┌──────────────┐                  │
│  │ RestTemplate │                  │
│  │    Bean      │                  │
│  └──────────────┘                  │
│         │                           │
│         │ Injects (loose coupling) │
│         ↓                           │
│  ┌──────────────────────┐          │
│  │ ProductServiceClient │          │
│  │   @Autowired         │          │
│  │   RestTemplate       │          │
│  └──────────────────────┘          │
└─────────────────────────────────────┘

ProductServiceClient RECEIVES dependencies
(doesn't control creation)
```

---

### Summary: Why Annotations Are Essential

| Problem | Solution |
|---------|----------|
| **Tight coupling** | `@Autowired` - Spring injects implementations |
| **Hard to test** | `@MockBean` - Easy to inject mocks |
| **Hardcoded config** | `@Value` - Externalized configuration |
| **Code duplication** | `@Bean` - Shared singletons |
| **Complex dependencies** | `@Service` + `@Autowired` - Auto-wiring |
| **Lifecycle management** | `@PostConstruct` + `@PreDestroy` - Spring manages |
| **Boilerplate code** | `@SpringBootApplication` - Auto-configuration |

**Annotations transform Java from verbose, coupled, hard-to-test code into clean, modular, testable applications.**

---

## 3. All Annotations Used in This Project

### A. Spring Core Annotations

#### `@SpringBootApplication`

**Location**: `consumer/src/main/java/io/pact/workshop/product_catalogue/Application.java:10`

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

- **Purpose**: Main entry point for Spring Boot application
- **Combines three annotations**:
  - `@Configuration`: Defines configuration beans
  - `@EnableAutoConfiguration`: Enables Spring Boot's auto-configuration
  - `@ComponentScan`: Scans for Spring components in the package and sub-packages
- **Effect**: Bootstraps the entire Spring application

---

#### `@Service`

**Location**: `consumer/src/main/java/io/pact/workshop/product_catalogue/clients/ProductServiceClient.java:15`

```java
@Service
public class ProductServiceClient {
    // Business logic here
}
```

- **Purpose**: Marks class as a service layer component
- **Effect**: Spring automatically detects and registers it as a bean during component scanning
- **Semantic meaning**: Contains business logic (service layer in MVC architecture)
- **Alternative annotations**: `@Component`, `@Repository`, `@Controller`
- **When to use**: For classes that contain business logic or orchestration

---

#### `@RestController`

**Location**: `provider/src/main/java/io/pact/workshop/product_service/controllers/ProductsController.java:14`

```java
@RestController
public class ProductsController {
    // REST endpoints here
}
```

- **Purpose**: Marks class as a REST API controller
- **Combines**: `@Controller` + `@ResponseBody`
- **Effect**: All methods return data directly as JSON/XML (not view names)
- **Use case**: Building RESTful web services

---

#### `@Autowired`

**Locations**: Multiple files

```java
@Autowired
private RestTemplate restTemplate;
```

**In ProductServiceClient.java:17**
```java
@Service
public class ProductServiceClient {
    @Autowired
    private RestTemplate restTemplate;  // Field injection

    @Value("${serviceClients.products.baseUrl}")
    private String baseUrl;
}
```

**In ProductsController.java:19**
```java
@RestController
public class ProductsController {
    @Autowired
    private ProductRepository productRepository;  // Field injection
}
```

- **Purpose**: Tells Spring to inject a dependency
- **How it works**: Spring finds a matching bean in the container and injects it
- **Injection types**:
  - **Field injection** (shown above): Direct field injection
  - **Constructor injection** (recommended): `public ProductServiceClient(RestTemplate restTemplate) { ... }`
  - **Setter injection**: `@Autowired public void setRestTemplate(RestTemplate rt) { ... }`
- **Note**: Modern practice favors constructor injection for better testability

---

#### `@Bean`

**Location**: `consumer/src/main/java/io/pact/workshop/product_catalogue/Application.java:16`

```java
@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {
    return builder
        .requestFactory(OkHttp3ClientHttpRequestFactory.class)
        .build();
}
```

- **Purpose**: Declares a method that produces a bean managed by Spring
- **Effect**: Return value is registered in Spring container and can be injected elsewhere
- **Use case**: When you need to configure third-party classes or complex object creation
- **Must be in**: A `@Configuration` class (or `@SpringBootApplication` which includes it)

---

#### `@Value`

**Location**: `consumer/src/main/java/io/pact/workshop/product_catalogue/clients/ProductServiceClient.java:20`

```java
@Value("${serviceClients.products.baseUrl}")
private String baseUrl;
```

- **Purpose**: Injects values from properties files or environment variables
- **Source**: `application.properties`, `application.yml`, or system properties
- **Syntax**:
  - `${property.name}`: From properties file
  - `${property.name:defaultValue}`: With default value
  - `#{expression}`: SpEL (Spring Expression Language)
- **Example properties file**:
  ```properties
  serviceClients.products.baseUrl=http://localhost:9000
  ```

---

### B. Spring MVC Annotations

#### `@GetMapping`

**Location**: `provider/src/main/java/io/pact/workshop/product_service/controllers/ProductsController.java:22, 27`

```java
@GetMapping("/products")
public ProductsResponse allProducts() {
    return new ProductsResponse((List<Product>) productRepository.findAll());
}

@GetMapping("/product/{id}")
public Product productById(@PathVariable("id") Long id) {
    return productRepository.findById(id).orElseThrow(ProductNotFoundException::new);
}
```

- **Purpose**: Maps HTTP GET requests to handler methods
- **Shortcut for**: `@RequestMapping(method = RequestMethod.GET)`
- **Path**: Can be at class level or method level
- **Related annotations**: `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`

---

#### `@PathVariable`

**Location**: `provider/src/main/java/io/pact/workshop/product_service/controllers/ProductsController.java:28`

```java
@GetMapping("/product/{id}")
public Product productById(@PathVariable("id") Long id) {
    // id is extracted from the URL path
}
```

- **Purpose**: Extracts values from URI path templates
- **Example**:
  - Request: `GET /product/10`
  - Result: `id = 10`
- **Type conversion**: Automatic conversion to method parameter type
- **Optional syntax**: `@PathVariable Long id` (name inferred from parameter name in Java 8+)

---

#### `@ResponseStatus`

**Location**: `provider/src/main/java/io/pact/workshop/product_service/controllers/ProductsController.java:16`

```java
@ResponseStatus(code = HttpStatus.NOT_FOUND, reason = "product not found")
public static class ProductNotFoundException extends RuntimeException { }
```

- **Purpose**: Marks exception with HTTP status code and reason
- **Effect**: When this exception is thrown, Spring returns the specified HTTP status
- **Use case**: Custom exception handling without writing `@ExceptionHandler`
- **Common status codes**:
  - `404 NOT_FOUND`
  - `400 BAD_REQUEST`
  - `401 UNAUTHORIZED`
  - `500 INTERNAL_SERVER_ERROR`

---

### C. Testing Annotations (JUnit 5)

#### `@SpringBootTest`

**Location**: `consumer/src/test/java/io/pact/workshop/product_catalogue/clients/ProductServiceClientTest.java:26`

```java
@SpringBootTest
@ExtendWith({ WiremockResolver.class, WiremockUriResolver.class })
class ProductServiceClientTest {
    @Autowired
    private ProductServiceClient productServiceClient;
    // Test methods
}
```

- **Purpose**: Loads full Spring application context for integration testing
- **Effect**: All beans are available in test (same as production)
- **Performance**: Slower than unit tests (full context loading)
- **Configuration options**:
  - `webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT`: Start server on random port
  - `webEnvironment = SpringBootTest.WebEnvironment.MOCK`: Mock web environment
- **When to use**: Integration tests that need the full application context

---

#### `@ExtendWith`

**Location**: `consumer/src/test/java/io/pact/workshop/product_catalogue/clients/ProductServiceClientTest.java:27`

```java
@ExtendWith({ WiremockResolver.class, WiremockUriResolver.class })
class ProductServiceClientTest {
    // ...
}
```

- **Purpose**: Registers JUnit 5 extensions to add custom behavior to test lifecycle
- **Effect**: Extensions can inject parameters, modify test execution, add setup/teardown
- **Use cases**:
  - Mock servers (WireMock)
  - Parameter injection
  - Custom setup/teardown
  - Conditional test execution
- **JUnit 5**: Replaces JUnit 4's `@RunWith`

---

#### `@Test`

**Location**: Multiple test files

```java
@Test
void fetchProducts(@Wiremock WireMockServer server, @WiremockUri String uri) {
    productServiceClient.setBaseUrl(uri);
    server.stubFor(
        get(urlPathEqualTo("/products"))
            .willReturn(aResponse()
                .withStatus(200)
                .withBody("...")
                .withHeader("Content-Type", "application/json"))
    );

    ProductServiceResponse response = productServiceClient.fetchProducts();
    assertThat(response.getProducts(), hasSize(2));
}
```

- **Purpose**: Marks method as a test case
- **JUnit 5**: Replaces JUnit 4's `@Test`
- **Visibility**: Can be package-private (no need for public)
- **Parameters**: Can accept parameters injected by extensions

---

### D. Pact Annotations

#### `@PactTestFor`

**Location**: `consumer/src/test/java/io/pact/workshop/product_catalogue/clients/ProductServiceClientPactTest.java:29`

**Class-level usage:**
```java
@SpringBootTest
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "ProductService")
class ProductServiceClientPactTest {
    // All tests in this class test interactions with ProductService
}
```

**Method-level usage:**
```java
@Test
@PactTestFor(pactMethod = "allProducts", pactVersion = PactSpecVersion.V3)
void testAllProducts(MockServer mockServer) {
    // This test uses the "allProducts" pact
}
```

- **Purpose**: Configures Pact consumer test
- **Parameters**:
  - `providerName`: Name of the provider service being tested
  - `pactMethod`: Which `@Pact` method defines the contract for this test
  - `pactVersion`: Pact specification version (V3, V4)
- **Effect**: Starts a mock server that implements the pact contract

---

#### `@Pact`

**Location**: `consumer/src/test/java/io/pact/workshop/product_catalogue/clients/ProductServiceClientPactTest.java:34`

```java
@Pact(consumer = "ProductCatalogue")
public RequestResponsePact allProducts(PactDslWithProvider builder) {
    return builder
        .given("products exists")
        .uponReceiving("get all products")
        .path("/products")
        .matchHeader("Authorization", "Bearer [a-zA-Z0-9=\\+/]+", "Bearer AAABd9yHUjI=")
        .willRespondWith()
        .status(200)
        .body(
            new PactDslJsonBody()
                .minArrayLike("products", 1, 2)
                .integerType("id", 9L)
                .stringType("name", "Gem Visa")
                .stringType("type", "CREDIT_CARD")
                .closeObject()
                .closeArray()
        )
        .toPact();
}
```

- **Purpose**: Defines a pact (contract) between consumer and provider
- **Consumer**: Name of the consumer service
- **Returns**: `RequestResponsePact` object describing the interaction
- **DSL Components**:
  - `.given()`: Provider state (setup condition)
  - `.uponReceiving()`: Description of the interaction
  - `.path()`, `.method()`: HTTP request details
  - `.matchHeader()`: Header matching (regex or exact)
  - `.willRespondWith()`: Expected response
  - `.status()`: HTTP status code
  - `.body()`: Response body with type matching
- **Output**: Generates a JSON pact file

---

### E. Pact Provider Verification Annotations

#### `@Provider`

**Location**: Provider test files

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Provider("ProductService")
@PactBroker
public class PactVerificationTest {
    // Provider verification tests
}
```

- **Purpose**: Identifies which provider this test is verifying
- **Name**: Must match the provider name in consumer pacts
- **Effect**: Pact framework uses this to find relevant pacts to verify

---

#### `@PactBroker`

**Location**: Provider test files

```java
@Provider("ProductService")
@PactBroker
public class PactVerificationTest {
    // Loads pacts from broker
}
```

- **Purpose**: Tells Pact to load contracts from a Pact Broker
- **Alternative**: `@PactFolder("pacts")` - loads from local directory
- **Configuration**: Requires `application.yml` with broker details:
  ```yaml
  pactbroker:
    host: localhost
    port: "9292"
    auth:
      username: pact_workshop
      password: pact_workshop
  ```

---

#### `@State`

**Location**: Provider test files

```java
@State(value = "products exists", action = StateChangeAction.SETUP)
void productsExists() {
    productRepository.deleteAll();
    productRepository.saveAll(Arrays.asList(
        new Product(100L, "Test Product 1", "CREDIT_CARD", "v1", "CC_001"),
        new Product(200L, "Test Product 2", "CREDIT_CARD", "v1", "CC_002")
    ));
}

@State(value = "product with ID 10 exists", action = StateChangeAction.SETUP)
void productExists(Map<String, Object> params) {
    long productId = ((Number) params.get("id")).longValue();
    productRepository.save(new Product(productId, "Product", "TYPE", "v1", "001"));
}
```

- **Purpose**: Sets up provider in specific state before verifying interaction
- **Value**: Matches the `.given()` state from consumer pact
- **Action**:
  - `StateChangeAction.SETUP`: Before interaction
  - `StateChangeAction.TEARDOWN`: After interaction (cleanup)
- **Parameters**: Can receive parameters from pact state
- **Use case**: Database setup, mock configuration, feature flags

---

#### `@TestTemplate`

**Location**: Provider test files

```java
@TestTemplate
@ExtendWith(PactVerificationSpringProvider.class)
void pactVerificationTestTemplate(PactVerificationContext context, HttpRequest request) {
    if (request.containsHeader("Authorization")) {
        request.setHeader("Authorization", "Bearer " + generateToken());
    }
    context.verifyInteraction();
}
```

- **Purpose**: JUnit 5 template method that runs multiple times (once per interaction)
- **Effect**: Pact creates a test instance for each interaction in the pact file
- **Parameters**:
  - `PactVerificationContext`: Context for current verification
  - `HttpRequest`: Can modify request before sending (use carefully!)
- **Use case**: Request filtering, dynamic header injection

---

## 4. The Concept of Pact Contract Testing

### What is Pact?

**Pact** is a code-first consumer-driven contract testing framework. It ensures that services can communicate correctly without requiring end-to-end integration tests.

### The Problem It Solves

**Traditional Integration Testing Problems:**

```
┌──────────┐           ┌──────────┐
│ Consumer │  ───────> │ Provider │
│  Tests   │           │  Tests   │
└──────────┘           └──────────┘

❌ Requires both services running simultaneously
❌ Slow and brittle (network, data dependencies)
❌ Hard to maintain test data consistency
❌ Environment dependencies (databases, message queues)
❌ Difficult to parallelize
❌ Late feedback (only in integration environment)
```

**Example Scenario:**
```
Consumer expects:     Provider returns:
GET /product/10       GET /products/10

❌ Mismatch discovered only during integration testing!
```

---

### How Pact Works

```
┌─────────────────────────────────────────────────────────────┐
│                   1. Consumer Tests                         │
│  ┌──────────┐              ┌──────────────┐                │
│  │ Consumer │  ─────────>  │    Pact      │                │
│  │   Code   │              │ Mock Server  │                │
│  └──────────┘              └──────────────┘                │
│       │                            │                        │
│       │ Makes request              │ Validates & Records   │
│       v                            v                        │
│  ┌────────────────────────────────────────┐                │
│  │  "When I request GET /products         │                │
│  │   I expect status 200 with JSON        │                │
│  │   containing an array of products"     │                │
│  └────────────────────────────────────────┘                │
│                            │                                │
│                            v                                │
│                   ┌─────────────────┐                       │
│                   │  Pact File      │                       │
│                   │  (Contract)     │                       │
│                   │  JSON           │                       │
│                   └─────────────────┘                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ Publish
                              v
                    ┌──────────────────┐
                    │   Pact Broker    │
                    │  (Central Repo)  │
                    └──────────────────┘
                              │
                              │ Fetch
                              v
┌─────────────────────────────────────────────────────────────┐
│                2. Provider Verification                     │
│  ┌─────────────────┐                                       │
│  │  Pact File      │                                       │
│  │  (Contract)     │                                       │
│  └─────────────────┘                                       │
│          │                                                  │
│          v                                                  │
│  ┌──────────────┐        ┌──────────────┐                 │
│  │    Pact      │ ────>  │   Provider   │                 │
│  │  Verifier    │ Replay │     Code     │                 │
│  │              │ <────  │  (Real API)  │                 │
│  └──────────────┘        └──────────────┘                 │
│          │                                                  │
│          v                                                  │
│  ✅ Pass: Provider meets consumer expectations             │
│  ❌ Fail: Provider doesn't match contract                  │
└─────────────────────────────────────────────────────────────┘
```

---

### Key Concepts

#### 1. Consumer-Driven Contracts

**Philosophy**: The consumer (API client) defines what it needs from the provider (API server).

```
Traditional: Provider builds API → Consumer adapts
Pact:        Consumer defines needs → Provider implements
```

**Benefits:**
- Consumer gets exactly what it needs
- No over-engineering of provider APIs
- Clear communication between teams
- Living documentation

---

#### 2. Provider States

**Location**: `consumer/src/test/java/io/pact/workshop/product_catalogue/clients/ProductServiceClientPactTest.java:37`

```java
.given("product with ID 10 exists", "id", 10)
```

**Purpose**: Set up provider in specific state for testing

**Examples in this project:**
- `"products exists"` - Database has products
- `"no products exists"` - Empty database
- `"product with ID 10 exists"` - Specific product exists
- `"product with ID 10 does not exist"` - Product doesn't exist

**Provider Implementation:**
```java
@State(value = "product with ID 10 exists", action = StateChangeAction.SETUP)
void productExists(Map<String, Object> params) {
    long productId = ((Number) params.get("id")).longValue();
    productRepository.save(new Product(productId, "Product", "TYPE", "v1", "001"));
}
```

**Why needed?**
- Tests must be repeatable and isolated
- Database state varies between test runs
- Allows testing edge cases (empty results, missing data)

---

#### 3. Interactions

Each pact defines one or more **interactions** (request/response pairs):

```java
@Pact(consumer = "ProductCatalogue")
public RequestResponsePact allProducts(PactDslWithProvider builder) {
    return builder
        .given("products exists")                    // Provider state
        .uponReceiving("get all products")           // Description
        .path("/products")                           // Request path
        .method("GET")                               // HTTP method
        .matchHeader("Authorization", "...")         // Request headers
        .willRespondWith()                           // Response expectations
        .status(200)                                 // Expected status
        .body(                                       // Expected body
            new PactDslJsonBody()
                .minArrayLike("products", 1, 2)     // Array with 1-2 items
                .integerType("id", 9L)              // Type matching
                .stringType("name", "Gem Visa")
                .stringType("type", "CREDIT_CARD")
                .closeObject()
                .closeArray()
        )
        .toPact();
}
```

**Components:**
- **Given**: Provider state setup
- **Upon receiving**: Human-readable description
- **Request**: Path, method, headers, query params, body
- **Will respond with**: Expected response (status, headers, body)

---

#### 4. Matching vs Equality

**Pact uses flexible matching** instead of exact equality:

```java
// ❌ Exact matching (brittle)
.body("{\"id\": 10, \"name\": \"Product\"}")

// ✅ Type matching (flexible)
.body(
    new PactDslJsonBody()
        .integerType("id", 10L)      // Any integer, example: 10
        .stringType("name", "Product") // Any string, example: "Product"
)
```

**Matching types:**
- `integerType()`: Matches any integer
- `stringType()`: Matches any string
- `booleanType()`: Matches any boolean
- `minArrayLike(name, min, examples)`: Array with minimum size
- `eachLike()`: Array where each item matches pattern
- `regex()`: Matches regular expression

**Why?**
- Don't care about exact IDs (generated by database)
- Don't care about exact timestamps
- Care about data types and structure

---

### Workflow in This Project

#### Phase 1: Consumer Side (ProductCatalogue)

**Step 1: Define Expectations**

File: `consumer/src/test/java/io/pact/workshop/product_catalogue/clients/ProductServiceClientPactTest.java:34-52`

```java
@Pact(consumer = "ProductCatalogue")
public RequestResponsePact allProducts(PactDslWithProvider builder) {
    return builder
        .given("products exists")
        .uponReceiving("get all products")
        .path("/products")
        .matchHeader("Authorization", "Bearer [a-zA-Z0-9=\\+/]+", "Bearer AAABd9yHUjI=")
        .willRespondWith()
        .status(200)
        .body(new PactDslJsonBody()
            .minArrayLike("products", 1, 2)
            .integerType("id", 9L)
            .stringType("name", "Gem Visa")
            .stringType("type", "CREDIT_CARD")
        )
        .toPact();
}
```

**Step 2: Test Against Mock Server**

File: `ProductServiceClientPactTest.java:55-62`

```java
@Test
@PactTestFor(pactMethod = "allProducts", pactVersion = PactSpecVersion.V3)
void testAllProducts(MockServer mockServer) {
    // Point client to Pact mock server
    productServiceClient.setBaseUrl(mockServer.getUrl());

    // Make actual call
    List<Product> products = productServiceClient.fetchProducts().getProducts();

    // Verify response
    assertThat(products, hasSize(2));
    assertThat(products.get(0), is(equalTo(new Product(9L, "Gem Visa", "CREDIT_CARD", null, null))));
}
```

**What happens:**
1. Pact starts mock server on random port
2. Mock server knows expectations from `@Pact` method
3. Test makes real HTTP call to mock server
4. Mock server validates request matches expectations
5. Mock server returns expected response
6. Test validates response handling
7. Pact records interaction to file

**Step 3: Generate Pact File**

Output: `consumer/target/pacts/ProductCatalogue-ProductService.json`

```json
{
  "consumer": {
    "name": "ProductCatalogue"
  },
  "provider": {
    "name": "ProductService"
  },
  "interactions": [
    {
      "description": "get all products",
      "providerStates": [
        {
          "name": "products exists"
        }
      ],
      "request": {
        "method": "GET",
        "path": "/products",
        "headers": {
          "Authorization": ["Bearer AAABd9yHUjI="]
        }
      },
      "response": {
        "status": 200,
        "headers": {
          "Content-Type": "application/json"
        },
        "body": {
          "products": [
            {
              "id": 9,
              "name": "Gem Visa",
              "type": "CREDIT_CARD"
            }
          ]
        },
        "matchingRules": {
          "body": {
            "$.products": {
              "matchers": [{"match": "type", "min": 1}]
            }
          }
        }
      }
    }
  ]
}
```

---

#### Phase 2: Provider Side (ProductService)

**Step 1: Load Pact File**

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Provider("ProductService")
@PactBroker  // Or @PactFolder("pacts") for local files
public class PactVerificationTest {
    // ...
}
```

**Step 2: Set Up Provider States**

File: `provider/src/test/java/io/pact/workshop/product_service/PactVerificationTest.java`

```java
@State(value = "products exists", action = StateChangeAction.SETUP)
void productsExists() {
    // Clear database
    productRepository.deleteAll();

    // Insert test data
    productRepository.saveAll(Arrays.asList(
        new Product(100L, "Test Product 1", "CREDIT_CARD", "v1", "CC_001"),
        new Product(200L, "Test Product 2", "CREDIT_CARD", "v1", "CC_002")
    ));
}

@State(value = "no products exists", action = StateChangeAction.SETUP)
void noProductsExist() {
    productRepository.deleteAll();
}

@State(value = "product with ID 10 exists", action = StateChangeAction.SETUP)
void productExists(Map<String, Object> params) {
    long productId = ((Number) params.get("id")).longValue();
    productRepository.save(new Product(productId, "Product", "TYPE", "v1", "001"));
}
```

**Step 3: Verify Against Real Provider**

```java
@TestTemplate
@ExtendWith(PactVerificationSpringProvider.class)
void pactVerificationTestTemplate(PactVerificationContext context, HttpRequest request) {
    // Optional: Modify request (e.g., refresh auth token)
    if (request.containsHeader("Authorization")) {
        request.setHeader("Authorization", "Bearer " + generateToken());
    }

    // Run verification
    context.verifyInteraction();
}
```

**What happens:**
1. Pact loads contract file
2. For each interaction:
   - Calls `@State` setup method
   - Sends real HTTP request to running provider
   - Receives actual response
   - Validates response against expectations
   - Calls `@State` teardown method (if defined)
3. Reports pass/fail for each interaction

**Step 4: Publish Results**

```bash
./mvnw verify \
  -Dpact.verifier.publishResults=true \
  -Dpact.provider.version=$(git rev-parse HEAD) \
  -Dpact.provider.branch=$(git rev-parse --abbrev-ref HEAD)
```

Results published to Pact Broker with:
- ✅ or ❌ for each interaction
- Git commit SHA
- Git branch
- Timestamp

---

### Benefits of Pact

| Benefit | Description |
|---------|-------------|
| **Fast** | No need to deploy services, tests run in milliseconds |
| **Reliable** | Isolated tests, no network flakiness or data race conditions |
| **Early Feedback** | Catch breaking changes in development, not production |
| **Documentation** | Pact files serve as executable, always up-to-date API docs |
| **Confidence** | "Can I deploy?" checks prevent breaking changes |
| **Independent Development** | Consumer and provider teams work in parallel |
| **Versioning** | Track contract changes over time |

---

### Pact vs Other Testing Approaches

| Approach | Pact Contract Testing | End-to-End Testing | Unit Testing with Mocks |
|----------|----------------------|-------------------|------------------------|
| **Speed** | ⚡ Fast (seconds) | 🐌 Slow (minutes) | ⚡ Fast (milliseconds) |
| **Reliability** | ✅ Reliable | ❌ Flaky | ✅ Reliable |
| **Setup** | 🟡 Moderate | 🔴 Complex | 🟢 Simple |
| **Real Integration** | ✅ Yes | ✅ Yes | ❌ No |
| **Catches Breaking Changes** | ✅ Yes | ✅ Yes | ❌ No |
| **Tests in Isolation** | ✅ Yes | ❌ No | ✅ Yes |
| **Requires Infrastructure** | ❌ No | ✅ Yes | ❌ No |

---

### Example: Catching Breaking Changes

**Scenario: Provider changes endpoint**

```java
// Provider changes endpoint from /product/{id} to /products/{id}
@GetMapping("/products/{id}")  // Changed!
public Product productById(@PathVariable("id") Long id) {
    return productRepository.findById(id).orElseThrow(ProductNotFoundException::new);
}
```

**Without Pact:**
```
Consumer deployed → Makes request to /product/10 → 404 Error
❌ Discovered in production!
```

**With Pact:**
```
1. Provider runs verification tests
2. Pact replays consumer requests
3. Request to /product/10 → 404 (expected 200)
❌ Test fails immediately
4. Provider team fixes endpoint or negotiates with consumer
5. Deploy safely
```

---

### Can I Deploy?

**The Problem:**
- Consumer verified against old provider
- Provider verified against old consumer
- Both look good individually
- But will they work together in production?

**The Solution: Pact Broker + Can-I-Deploy**

```bash
# Consumer checks if it can deploy
./mvnw pact:can-i-deploy \
  -Dpacticipant='ProductCatalogue' \
  -DpacticipantVersion=$(git rev-parse HEAD) \
  -DtoEnvironment=production

# Broker checks:
# ✅ Does a verified provider exist in production?
# ✅ Is the pact verified by that provider?
# ✅ Are there any pending changes?
```

**Workflow:**
```
┌─────────────────────────────────────────┐
│  1. Consumer publishes pact             │
│     Version: abc123                     │
└─────────────────────────────────────────┘
              │
              v
┌─────────────────────────────────────────┐
│  2. Provider verifies pact              │
│     Version: def456                     │
│     Result: ✅ PASS                     │
└─────────────────────────────────────────┘
              │
              v
┌─────────────────────────────────────────┐
│  3. Provider records deployment         │
│     Version: def456                     │
│     Environment: production             │
└─────────────────────────────────────────┘
              │
              v
┌─────────────────────────────────────────┐
│  4. Consumer can-i-deploy?              │
│     ✅ Yes! Verified provider in prod   │
└─────────────────────────────────────────┘
```

---

## 5. Project Structure

```
pact-workshop-Maven-Springboot-JUnit5/
│
├── consumer/                           # ProductCatalogue (API consumer)
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   └── io/pact/workshop/product_catalogue/
│   │   │   │       ├── Application.java
│   │   │   │       │   @SpringBootApplication
│   │   │   │       │   @Bean RestTemplate
│   │   │   │       │
│   │   │   │       ├── clients/
│   │   │   │       │   ├── ProductServiceClient.java
│   │   │   │       │   │   @Service
│   │   │   │       │   │   @Autowired RestTemplate
│   │   │   │       │   │   @Value baseUrl
│   │   │   │       │   │   Methods: fetchProducts(), getProductById()
│   │   │   │       │   │
│   │   │   │       │   └── ProductServiceResponse.java
│   │   │   │       │       (DTO for products list)
│   │   │   │       │
│   │   │   │       ├── controllers/
│   │   │   │       │   └── ProductCatalogueController.java
│   │   │   │       │       @RestController
│   │   │   │       │       Web UI endpoints
│   │   │   │       │
│   │   │   │       └── models/
│   │   │   │           ├── Product.java
│   │   │   │           │   (Domain model)
│   │   │   │           └── ProductCatalogue.java
│   │   │   │
│   │   │   └── resources/
│   │   │       ├── application.properties
│   │   │       │   serviceClients.products.baseUrl=http://localhost:9000
│   │   │       └── templates/
│   │   │           (Thymeleaf templates)
│   │   │
│   │   └── test/
│   │       └── java/
│   │           └── io/pact/workshop/product_catalogue/clients/
│   │               │
│   │               ├── ProductServiceClientTest.java
│   │               │   @SpringBootTest
│   │               │   @ExtendWith(WiremockResolver)
│   │               │   Unit tests with WireMock
│   │               │   Tests: fetchProducts(), getProductById()
│   │               │
│   │               └── ProductServiceClientPactTest.java
│   │                   @SpringBootTest
│   │                   @ExtendWith(PactConsumerTestExt)
│   │                   @PactTestFor(providerName = "ProductService")
│   │
│   │                   Pact definitions:
│   │                   - allProducts() - products exist
│   │                   - noProducts() - no products exist
│   │                   - singleProduct() - product exists
│   │                   - singleProductNotExists() - 404 case
│   │                   - noAuthToken() - 401 case
│   │                   - noAuthToken2() - 401 case for single product
│   │
│   ├── target/
│   │   └── pacts/
│   │       └── ProductCatalogue-ProductService.json
│   │           (Generated pact contract file)
│   │
│   └── pom.xml
│       Dependencies:
│       - spring-boot-starter-web
│       - spring-boot-starter-thymeleaf
│       - au.com.dius.pact.consumer (Pact consumer library)
│       - wiremock (for unit tests)
│
├── provider/                           # ProductService (API provider)
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   └── io/pact/workshop/product_service/
│   │   │   │       ├── Application.java
│   │   │   │       │   @SpringBootApplication
│   │   │   │       │
│   │   │   │       ├── controllers/
│   │   │   │       │   ├── ProductsController.java
│   │   │   │       │   │   @RestController
│   │   │   │       │   │   @Autowired ProductRepository
│   │   │   │       │   │   @GetMapping("/products")
│   │   │   │       │   │   @GetMapping("/product/{id}")
│   │   │   │       │   │
│   │   │   │       │   └── ProductsResponse.java
│   │   │   │       │       (DTO)
│   │   │   │       │
│   │   │   │       ├── products/
│   │   │   │       │   ├── Product.java
│   │   │   │       │   │   @Entity (JPA entity)
│   │   │   │       │   │
│   │   │   │       │   └── ProductRepository.java
│   │   │   │       │       @Repository
│   │   │   │       │       extends JpaRepository
│   │   │   │       │
│   │   │   │       └── config/
│   │   │   │           ├── BearerAuthorizationFilter.java
│   │   │   │           │   extends OncePerRequestFilter
│   │   │   │           │   Validates bearer tokens
│   │   │   │           │
│   │   │   │           └── WebSecurityConfig.java
│   │   │   │               @Configuration
│   │   │   │               Security configuration
│   │   │   │
│   │   │   └── resources/
│   │   │       ├── application.properties
│   │   │       │   Database configuration (H2)
│   │   │       └── data.sql
│   │   │           Sample data (products)
│   │   │
│   │   └── test/
│   │       ├── java/
│   │       │   └── io/pact/workshop/product_service/
│   │       │       └── PactVerificationTest.java
│   │       │           @SpringBootTest(webEnvironment = RANDOM_PORT)
│   │       │           @Provider("ProductService")
│   │       │           @PactBroker
│   │       │           @Autowired ProductRepository
│   │       │
│   │       │           State handlers:
│   │       │           @State("products exists")
│   │       │           @State("no products exists")
│   │       │           @State("product with ID 10 exists")
│   │       │           @State("product with ID 10 does not exist")
│   │       │
│   │       │           @TestTemplate - verification method
│   │       │
│   │       └── resources/
│   │           └── application.yml
│   │               Pact Broker configuration
│   │
│   ├── pacts/
│   │   └── ProductCatalogue-ProductService.json
│   │       (Copied from consumer for local verification)
│   │
│   └── pom.xml
│       Dependencies:
│       - spring-boot-starter-web
│       - spring-boot-starter-data-jpa
│       - spring-boot-starter-security
│       - h2database (in-memory database)
│       - au.com.dius.pact.provider (Pact provider library)
│
├── docker-compose.yml
│   Pact Broker setup:
│   - PostgreSQL database
│   - Pact Broker (UI + API)
│   - Accessible at http://localhost:9292
│
└── README.md
    Comprehensive workshop guide
```

---

### Component Interactions

```
┌─────────────────────────────────────────────────┐
│              Consumer Application               │
│                                                 │
│  ┌──────────────────────────────────┐          │
│  │  ProductCatalogueController      │          │
│  │  @RestController                 │          │
│  │  Web endpoints for users         │          │
│  └────────────────┬─────────────────┘          │
│                   │                             │
│                   v                             │
│  ┌──────────────────────────────────┐          │
│  │  ProductServiceClient            │          │
│  │  @Service                        │          │
│  │  @Autowired RestTemplate         │          │
│  │                                  │          │
│  │  Methods:                        │          │
│  │  - fetchProducts()               │          │
│  │  - getProductById(id)            │          │
│  └────────────────┬─────────────────┘          │
│                   │                             │
│                   │ HTTP requests               │
└───────────────────┼─────────────────────────────┘
                    │
                    │ Bearer token authentication
                    │
                    v
┌─────────────────────────────────────────────────┐
│              Provider Application               │
│                                                 │
│  ┌──────────────────────────────────┐          │
│  │  BearerAuthorizationFilter       │          │
│  │  Validates auth token            │          │
│  └────────────────┬─────────────────┘          │
│                   │                             │
│                   v                             │
│  ┌──────────────────────────────────┐          │
│  │  ProductsController              │          │
│  │  @RestController                 │          │
│  │  @Autowired ProductRepository    │          │
│  │                                  │          │
│  │  Endpoints:                      │          │
│  │  - GET /products                 │          │
│  │  - GET /product/{id}             │          │
│  └────────────────┬─────────────────┘          │
│                   │                             │
│                   v                             │
│  ┌──────────────────────────────────┐          │
│  │  ProductRepository               │          │
│  │  @Repository                     │          │
│  │  JpaRepository<Product, Long>    │          │
│  └────────────────┬─────────────────┘          │
│                   │                             │
│                   v                             │
│  ┌──────────────────────────────────┐          │
│  │  H2 Database                     │          │
│  │  In-memory SQL database          │          │
│  └──────────────────────────────────┘          │
└─────────────────────────────────────────────────┘
```

---

## 6. Summary

### Spring Boot Fundamentals

| Concept | Explanation | Example |
|---------|-------------|---------|
| **Inversion of Control (IoC)** | Framework controls object creation, not developer | Spring creates `RestTemplate`, injects it |
| **Dependency Injection** | Objects receive dependencies from outside | `@Autowired` injects beans |
| **Component Scanning** | Automatically finds and registers beans | `@Service`, `@RestController` |
| **Auto-configuration** | Configures app based on classpath | Sees H2 → configures DataSource |

### Annotations Quick Reference

| Annotation | Purpose | Layer |
|------------|---------|-------|
| `@SpringBootApplication` | Main entry point | Application |
| `@Service` | Business logic component | Service |
| `@RestController` | REST API controller | Controller |
| `@Repository` | Data access component | Data |
| `@Autowired` | Inject dependency | Any |
| `@Value` | Inject property value | Any |
| `@Bean` | Define bean factory method | Configuration |
| `@GetMapping` | Map GET request | Controller |
| `@PathVariable` | Extract from URL path | Controller |
| `@SpringBootTest` | Integration test | Test |
| `@Pact` | Define consumer contract | Test (Consumer) |
| `@PactTestFor` | Configure pact test | Test (Consumer) |
| `@Provider` | Identify provider | Test (Provider) |
| `@State` | Provider state handler | Test (Provider) |

### Pact Contract Testing

| Aspect | Description |
|--------|-------------|
| **What** | Code-first contract testing between services |
| **Why** | Catch integration issues early without E2E tests |
| **When** | Every commit - consumer and provider independently |
| **How** | Consumer defines expectations → Provider verifies |
| **Output** | JSON contract files + verification results |
| **Tools** | Pact framework + Pact Broker |

### Testing Pyramid with Pact

```
        ╱╲
       ╱  ╲      E2E Tests
      ╱────╲     (Few, Slow, Brittle)
     ╱      ╲
    ╱────────╲   Contract Tests (Pact)
   ╱          ╲  (Moderate, Fast, Reliable)
  ╱────────────╲
 ╱              ╲ Unit Tests
╱────────────────╲ (Many, Very Fast, Isolated)
```

### Key Takeaways

1. **Spring Boot** simplifies Java web development through auto-configuration and convention over configuration

2. **Annotations** enable declarative programming - you declare **what** you want, Spring handles **how**

3. **Dependency Injection** promotes loose coupling, testability, and maintainability

4. **Pact** provides fast, reliable integration testing without deploying services

5. **Consumer-Driven Contracts** ensure services can communicate before integration

6. **Provider States** make contract tests repeatable and isolated

7. **Pact Broker** enables safe continuous deployment with "can-i-deploy" checks

---

### Additional Resources

- **Spring Boot Documentation**: https://spring.io/projects/spring-boot
- **Pact Documentation**: https://docs.pact.io/
- **Pact Workshop**: https://github.com/pact-foundation/pact-workshop-Maven-Springboot-JUnit5
- **Martin Fowler on Contract Testing**: https://martinfowler.com/bliki/ContractTest.html
- **Consumer-Driven Contracts**: https://martinfowler.com/articles/consumerDrivenContracts.html

---

### Workshop Steps Summary

This project follows an incremental workshop approach:

1. **Step 1**: Create consumer without provider
2. **Step 2**: Write unit tests with WireMock
3. **Step 3**: Write Pact consumer tests
4. **Step 4**: Verify provider with pacts
5. **Step 5**: Fix consumer endpoint mismatch
6. **Step 6**: Add tests for missing data (404)
7. **Step 7**: Add provider states
8. **Step 8**: Add authentication to consumer
9. **Step 9**: Implement authentication on provider
10. **Step 10**: Use request filters for dynamic auth tokens
11. **Step 11**: Integrate with Pact Broker for CI/CD

Each step demonstrates a key concept in contract testing and Spring Boot development.
