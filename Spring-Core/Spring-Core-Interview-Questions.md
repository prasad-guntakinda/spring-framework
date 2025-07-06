# Spring Core Interview Questions:

<details>
<summary>1. What is Inversion of Control (IoC) in Spring?</summary>

**Explanation:**
Inversion of Control is a design principle in which the control of object creation and dependency management is transferred from the application code to the Spring IoC container. This allows for loose coupling and easier testing.

**Example:**
```java
// Service interface
public interface MessageService {
    String getMessage();
}

// Implementation
public class EmailService implements MessageService {
    public String getMessage() {
        return "Email Message";
    }
}

// Consumer
public class Notification {
    private MessageService messageService;
    // Dependency injected via constructor
    public Notification(MessageService messageService) {
        this.messageService = messageService;
    }
}
```
</details>

---

<details>
<summary>2. What is Dependency Injection (DI)?</summary>

**Explanation:**
Dependency Injection is a technique where the Spring container injects dependencies (objects) into a class, rather than the class creating them itself. DI can be done via constructor, setter, or field injection.

**Example:**
```java
@Component
public class UserService {
    private final UserRepository userRepository;
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```
</details>

---

<details>
<summary>3. What are the different types of bean scopes in Spring?</summary>

**Explanation:**
Spring supports several bean scopes:
- **Singleton** (default): One instance per Spring container
- **Prototype**: New instance each time requested
- **Request**: One instance per HTTP request (web only)
- **Session**: One instance per HTTP session (web only)
- **Application**: One instance per ServletContext (web only)
- **Websocket**: One instance per WebSocket

**Example:**
```java
@Component
@Scope("prototype")
public class MyPrototypeBean {
    // ...
}
```
</details>

---

<details>
<summary>4. What is the difference between BeanFactory and ApplicationContext?</summary>

**Explanation:**
- **BeanFactory** is the basic container, provides basic DI support.
- **ApplicationContext** is a superset of BeanFactory, providing additional features like event propagation, internationalization, and application-layer specific contexts.

**Example:**
```java
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
MyBean bean = context.getBean(MyBean.class);
```
</details>

---

<details>
<summary>5. How does Spring manage the lifecycle of a bean?</summary>

**Explanation:**
Spring manages bean lifecycle through several phases: instantiation, property population, initialization (custom init methods), and destruction (custom destroy methods). You can use `@PostConstruct` and `@PreDestroy` annotations or specify init/destroy methods in configuration.

**Example:**
```java
@Component
public class MyBean {
    @PostConstruct
    public void init() {
        // Initialization logic
    }
    @PreDestroy
    public void destroy() {
        // Cleanup logic
    }
}
```
</details>

---

<details>
<summary>6. What is the use of @Component, @Service, @Repository, and @Controller annotations?</summary>

**Explanation:**
These are stereotype annotations for auto-detecting and registering beans in the Spring context:
- `@Component`: Generic stereotype for any bean
- `@Service`: Service layer
- `@Repository`: Data access layer, adds exception translation
- `@Controller`: Web controller

**Example:**
```java
@Service
public class OrderService { }

@Repository
public class OrderRepository { }

@Controller
public class OrderController { }
```
</details>

---

<details>
<summary>7. What is Spring's @Autowired annotation?</summary>

**Explanation:**
`@Autowired` is used for automatic dependency injection by type. It can be applied to constructors, fields, or setter methods.

**Example:**
```java
@Component
public class ProductService {
    @Autowired
    private ProductRepository productRepository;
}
```
</details>

---

<details>
<summary>8. What is the purpose of the @Qualifier annotation?</summary>

**Explanation:**
`@Qualifier` is used along with `@Autowired` to resolve ambiguity when multiple beans of the same type exist.

**Example:**
```java
@Autowired
@Qualifier("emailService")
private MessageService messageService;
```
</details>

---

<details>
<summary>9. How does Spring handle events?</summary>

**Explanation:**
Spring provides an event publishing mechanism. You can create custom events and listeners using `ApplicationEventPublisher` and `@EventListener`.

**Example:**
```java
public class CustomEvent extends ApplicationEvent {
    public CustomEvent(Object source) {
        super(source);
    }
}

@Component
public class CustomEventListener {
    @EventListener
    public void handleCustomEvent(CustomEvent event) {
        // Handle event
    }
}
```
</details>

---

<details>
<summary>10. What is the Spring Expression Language (SpEL)?</summary>

**Explanation:**
SpEL is a powerful expression language for querying and manipulating objects at runtime. It is used in annotations like `@Value`, in XML, and in bean definitions.

**Example:**
```java
@Value("#{2 * 10}")
private int result; // result = 20
```
</details>

---

<details>
<summary>11. Why is constructor injection recommended over other types of Dependency Injection in Spring?</summary>

**Explanation:**
Constructor injection is generally preferred over field or setter injection for several reasons:

- **Immutability:** Dependencies are provided at object creation, making the object immutable and thread-safe.
- **Mandatory Dependencies:** Ensures all required dependencies are provided, preventing the object from being in an invalid state.
- **Testability:** Makes it easier to write unit tests, as dependencies can be passed directly in test code.
- **Promotes Loose Coupling:** The class does not depend on the Spring container for injection, making it more decoupled and portable.
- **Easier for Dependency Management Tools:** Tools like Lombok and frameworks like Spring Boot work better with constructor injection, especially for autowiring.

**Example:**
```java
@Component
public class OrderService {
    private final PaymentService paymentService;
    // Constructor injection
    @Autowired
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

In summary, constructor injection leads to safer, more maintainable, and testable code.
</details>

---

<details>
<summary>12. How does Spring Boot work better with constructor injection? Explain with an example.</summary>

**Explanation:**
Spring Boot works better with constructor injection because:
- If a class has only one constructor, Spring Boot automatically injects dependencies without requiring the `@Autowired` annotation, making the code cleaner.
- It ensures all required dependencies are provided at object creation, supporting immutability and easier testing.
- Constructor injection is less error-prone and more explicit about required dependencies.

**Example:**
```java
@Component
public class EmailService {
    public void send(String message) {
        // send email
    }
}

@Service
public class NotificationService {
    private final EmailService emailService;

    // No need for @Autowired if only one constructor
    public NotificationService(EmailService emailService) {
        this.emailService = emailService;
    }

    public void notifyUser(String msg) {
        emailService.send(msg);
    }
}
```

In this example, Spring Boot automatically injects `EmailService` into `NotificationService` via the constructor, ensuring all dependencies are satisfied at creation time and making the code concise and maintainable.
</details>

---

