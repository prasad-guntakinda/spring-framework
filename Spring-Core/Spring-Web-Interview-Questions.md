# Spring-Web Interview Questions:

<details>
<summary>1. What is the DispatcherServlet in Spring MVC?</summary>

**Explanation:**
The DispatcherServlet is the front controller in Spring MVC. It receives all incoming HTTP requests, delegates them to appropriate controllers, and manages the overall request processing workflow.

**Example:**
```xml
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
```
</details>

---

<details>
<summary>2. How do you define a controller in Spring MVC?</summary>

**Explanation:**
A controller in Spring MVC is a class annotated with `@Controller` that handles web requests and returns a view or data.

**Example:**
```java
@Controller
public class HomeController {
    @RequestMapping("/home")
    public String home() {
        return "home"; // returns view name
    }
}
```
</details>

---

<details>
<summary>3. What is @RequestMapping and how is it used?</summary>

**Explanation:**
`@RequestMapping` is used to map web requests to specific handler methods or classes in controllers.

**Example:**
```java
@RequestMapping(value = "/greet", method = RequestMethod.GET)
public String greet() {
    return "greeting";
}
```
</details>

---

<details>
<summary>4. How do you handle path variables and request parameters in Spring MVC?</summary>

**Explanation:**
Path variables are handled using `@PathVariable`, and request parameters are handled using `@RequestParam`.

**Example:**
```java
@GetMapping("/user/{id}")
public String getUser(@PathVariable int id) {
    // ...
}

@GetMapping("/search")
public String search(@RequestParam String query) {
    // ...
}
```
</details>

---

<details>
<summary>5. How do you return JSON or XML responses in Spring REST controllers?</summary>

**Explanation:**
Use `@RestController` and return objects directly. Spring automatically serializes the response to JSON or XML based on content negotiation.

**Example:**
```java
@RestController
public class UserController {
    @GetMapping("/api/user")
    public User getUser() {
        return new User("John", "Doe");
    }
}
```
</details>

---

<details>
<summary>6. What is the difference between @Controller and @RestController?</summary>

**Explanation:**
- `@Controller` is used for web pages (views), and methods typically return view names.
- `@RestController` is a convenience annotation that combines `@Controller` and `@ResponseBody`, returning data (like JSON) directly.

**Example:**
```java
@Controller
public class PageController {
    @GetMapping("/page")
    public String page() {
        return "pageView";
    }
}

@RestController
public class ApiController {
    @GetMapping("/api/data")
    public Data getData() {
        return new Data();
    }
}
```
</details>

---

<details>
<summary>7. How do you handle exceptions in Spring MVC/REST?</summary>

**Explanation:**
Use `@ExceptionHandler` in controllers or `@ControllerAdvice` for global exception handling.

**Example:**
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleException(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(ex.getMessage());
    }
}
```
</details>

---

<details>
<summary>8. How do you validate request data in Spring MVC/REST?</summary>

**Explanation:**
Use JSR-303/JSR-380 annotations (like `@Valid`, `@NotNull`, `@Size`) and the `@Valid` annotation in controller methods.

**Example:**
```java
@RestController
public class UserController {
    @PostMapping("/api/user")
    public ResponseEntity<String> createUser(@Valid @RequestBody User user) {
        // ...
        return ResponseEntity.ok("User created");
    }
}

public class User {
    @NotNull
    private String name;
    // ...
}
```
</details>

---

<details>
<summary>9. How do you implement file upload in Spring MVC?</summary>

**Explanation:**
Use `MultipartFile` in controller methods to handle file uploads.

**Example:**
```java
@PostMapping("/upload")
public String uploadFile(@RequestParam("file") MultipartFile file) {
    // process file
    return "success";
}
```
</details>

---

<details>
<summary>10. How do you secure REST endpoints in Spring?</summary>

**Explanation:**
Use Spring Security to secure REST endpoints with authentication and authorization.

**Example:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            .and()
            .httpBasic();
    }
}
```
</details>

---

<details>
<summary>11. How can you handle exceptions in Spring MVC and REST? Explain all ways with examples.</summary>

**Explanation:**
Spring provides multiple ways to handle exceptions in MVC and REST applications:

**1. Using @ExceptionHandler in a Controller**
- Handles specific exceptions for a single controller.

**Example:**
```java
@Controller
public class MyController {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

**2. Using @ControllerAdvice for Global Exception Handling**
- Handles exceptions across all controllers globally.

**Example:**
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleAll(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Error: " + ex.getMessage());
    }
}
```

**3. Using ResponseStatusException and @ResponseStatus**
- Throw `ResponseStatusException` or annotate custom exceptions with `@ResponseStatus` to set HTTP status codes.

**Example:**
```java
// Throwing ResponseStatusException
@GetMapping("/user/{id}")
public User getUser(@PathVariable int id) {
    if (idNotFound) {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "User not found");
    }
    return user;
}

// Custom exception with @ResponseStatus
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class InvalidRequestException extends RuntimeException {
    public InvalidRequestException(String msg) { super(msg); }
}
```

**4. Using HandlerExceptionResolver**
- Implement the `HandlerExceptionResolver` interface for custom exception resolution logic.

**Example:**
```java
@Component
public class MyExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        ModelAndView mav = new ModelAndView("error");
        mav.addObject("message", ex.getMessage());
        return mav;
    }
}
```

These approaches allow you to handle exceptions at different levels (method, controller, global) and customize the response for web and REST APIs.
</details>

---


## RESTful Web Services + HTTP Protocol:

<details>
<summary>1. What is REST and what are its key principles?</summary>

**Explanation:**
REST (Representational State Transfer) is an architectural style for designing networked applications. Key principles include:
- Statelessness
- Client-Server architecture
- Uniform Interface
- Resource-based (identified by URIs)
- Cacheability
- Layered System
</details>

---

<details>
<summary>2. What are HTTP methods and how are they used in RESTful services?</summary>

**Explanation:**
HTTP methods define the action to be performed on a resource:
- **GET:** Retrieve data
- **POST:** Create new resource
- **PUT:** Update/replace resource
- **PATCH:** Partially update resource
- **DELETE:** Remove resource

**Example:**
```http
GET /api/users/1
POST /api/users
PUT /api/users/1
PATCH /api/users/1
DELETE /api/users/1
```
</details>

---

<details>
<summary>3. What are HTTP status codes? Give examples relevant to REST APIs.</summary>

**Explanation:**
HTTP status codes indicate the result of an HTTP request. Common codes in REST:
- **200 OK:** Success
- **201 Created:** Resource created
- **204 No Content:** Success, no response body
- **400 Bad Request:** Invalid input
- **401 Unauthorized:** Authentication required
- **403 Forbidden:** Not allowed
- **404 Not Found:** Resource not found
- **500 Internal Server Error:** Server error
</details>

---

<details>
<summary>4. How do you design resource URIs in RESTful APIs?</summary>

**Explanation:**
- Use nouns for resources (e.g., `/users`, `/orders`)
- Use plural names
- Use sub-resources for relationships (e.g., `/users/1/orders`)
- Avoid verbs in URIs

**Example:**
```http
GET /api/products/123
GET /api/users/1/orders
```
</details>

---

<details>
<summary>5. What is content negotiation in REST?</summary>

**Explanation:**
Content negotiation allows clients and servers to agree on the format of the response (e.g., JSON, XML) using the `Accept` header.

**Example:**
```http
GET /api/user/1
Accept: application/json
```
</details>

---

<details>
<summary>6. How do you secure RESTful web services?</summary>

**Explanation:**
Common ways to secure REST APIs:
- HTTP Basic Authentication
- OAuth2/JWT tokens
- HTTPS for transport security
- Role-based access control (Spring Security)

**Example:**
```http
GET /api/secure-data
Authorization: Bearer &lt;token&gt;
```
</details>

---

<details>
<summary>7. What are idempotent and safe HTTP methods?</summary>

**Explanation:**
- **Safe methods** (like GET, HEAD): Do not modify resources.
- **Idempotent methods** (like GET, PUT, DELETE): Multiple identical requests have the same effect as a single request.

**Example:**
- GET /api/user/1 (safe, idempotent)
- PUT /api/user/1 (idempotent)
- DELETE /api/user/1 (idempotent)
</details>

---

<details>
<summary>8. How do you handle versioning in REST APIs?</summary>

**Explanation:**
Common versioning strategies:
- URI versioning: `/api/v1/users`
- Request header versioning: `Accept: application/vnd.company.v1+json`
- Query parameter versioning: `/api/users?version=1`
</details>

---

<details>
<summary>9. What are common ways to document RESTful APIs?</summary>

**Explanation:**
- OpenAPI/Swagger
- Spring REST Docs
- RAML
- API Blueprint

**Example:**
Swagger UI provides interactive documentation for REST APIs.
</details>

---

<details>
<summary>10. What is HATEOAS in REST?</summary>

**Explanation:**
HATEOAS (Hypermedia as the Engine of Application State) is a constraint of REST that says clients interact with a REST API entirely through hypermedia provided dynamically by server responses.

**Example:**
A REST response includes links to related resources:
```json
{
  "id": 1,
  "name": "John",
  "links": [
    { "rel": "self", "href": "/api/users/1" },
    { "rel": "orders", "href": "/api/users/1/orders" }
  ]
}
```
</details>

---

<details>
<summary>11. Can you elaborate on API versioning strategies in RESTful web services? Provide clear examples.</summary>

**Explanation:**
API versioning is important to ensure backward compatibility as APIs evolve. Common strategies include:

**1. URI Versioning**
- Version is included in the URL path.
- **Example:**
```http
GET /api/v1/users
GET /api/v2/users
```

**2. Request Header Versioning**
- Version is specified in a custom or Accept header.
- **Example:**
```http
GET /api/users
Accept: application/vnd.company.v1+json
```

**3. Query Parameter Versioning**
- Version is passed as a query parameter.
- **Example:**
```http
GET /api/users?version=1
```

**4. Content Negotiation Versioning**
- Version is part of the media type in the Accept header.
- **Example:**
```http
GET /api/users
Accept: application/json;version=1
```

**Best Practices:**
- URI versioning is the most common and easiest to implement.
- Header and content negotiation versioning keep URLs clean but may be harder to test/debug.
- Choose a strategy that fits your team's needs and stick to it consistently.
</details>

---



