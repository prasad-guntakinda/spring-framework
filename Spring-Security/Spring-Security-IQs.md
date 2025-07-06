# Spring Security IQs

## Key Concepts to Learn and Master in Spring Security

- Core Security Architecture (Filters, DelegatingFilterProxy)
- Authentication vs Authorization
- UserDetails, UserDetailsService, AuthenticationProvider
- Password encoding and storage (PasswordEncoder)
- SecurityContext and SecurityContextHolder
- Configuring HTTP security (antMatchers, authorizeRequests, etc.)
- Form-based authentication
- HTTP Basic and Digest authentication
- Custom authentication and authorization
- Method-level security (@PreAuthorize, @Secured, @RolesAllowed)
- Role-based and permission-based access control
- CSRF protection
- CORS (Cross-Origin Resource Sharing)
- Session management and stateless authentication
- Exception handling and custom error pages
- Security for REST APIs (stateless, JWT, OAuth2)
- Integrating with LDAP and external identity providers
- OAuth2 and OpenID Connect (OIDC)
- JWT (JSON Web Token) authentication
- Remember-me authentication
- Security testing and best practices
- Customizing login/logout flows
- Security event listeners and auditing
- Security in microservices (Spring Cloud Security, API Gateway)

Mastering these concepts will help you design, implement, and troubleshoot secure applications using Spring Security.

---


<details>
<summary>Security for REST APIs in Spring Security</summary>

**Explanation:**
Securing REST APIs is crucial because they are often stateless, exposed over the internet, and accessed by various clients. Key concepts include:

**1. Stateless Authentication**
- REST APIs should be stateless; avoid using HTTP sessions.
- Use tokens (like JWT) for authentication instead of session cookies.

**2. HTTP Basic Authentication**
- Simple, built-in mechanism using username and password in the HTTP header.
- Not recommended for production unless used with HTTPS.

**Example:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeRequests()
                .antMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated()
            .and()
            .httpBasic();
    }
}
```

**3. JWT (JSON Web Token) Authentication**
- JWTs are self-contained tokens that carry user identity and claims.
- The server validates the token on each request; no session is stored.

**Example:**
- Client logs in and receives a JWT.
- Client sends JWT in the `Authorization: Bearer <token>` header for each request.
- Server validates the JWT signature and claims.

**4. OAuth2**
- Used for delegated authorization (third-party logins, SSO).
- Spring Security provides support for OAuth2 Resource Server and Client.

**5. CSRF Protection**
- CSRF is less relevant for stateless APIs, so CSRF protection is usually disabled for REST endpoints.

**6. CORS (Cross-Origin Resource Sharing)**
- Configure CORS to allow or restrict which domains can access your API.

**Example:**
```java
@Bean
public WebMvcConfigurer corsConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/api/**").allowedOrigins("https://trusted.com");
        }
    };
}
```

**7. Exception Handling**
- Customize error responses for authentication and authorization failures using `@ControllerAdvice` or custom handlers.

**Best Practices:**
- Always use HTTPS.
- Validate and sanitize all inputs.
- Use strong password policies and token expiration.
- Limit API rate and monitor for abuse.

</details>

---

<details>
<summary>How does SecurityConfig work in Spring Security? Explain the flow with examples.</summary>

**Explanation:**
`SecurityConfig` is a configuration class where you define how Spring Security should secure your application. It typically extends `WebSecurityConfigurerAdapter` (Spring Security 5.x and below) or uses `SecurityFilterChain` beans (Spring Security 6+). The configuration controls authentication, authorization, and other security features.

**Spring Security Flow:**
1. **Request Interception:** Every HTTP request passes through a chain of security filters (FilterChainProxy).
2. **Authentication:** The filter chain checks if the request is authenticated. If not, it triggers the authentication process (e.g., form login, HTTP Basic, JWT, etc.).
3. **Authorization:** After authentication, the request is checked for authorization (roles/permissions).
4. **Access Decision:** If authorized, the request proceeds to the controller. If not, an error (401/403) is returned.
5. **SecurityContext:** User details are stored in the `SecurityContextHolder` for the duration of the request.

**Example (Spring Security 5.x):**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeRequests()
                .antMatchers("/public/**").permitAll()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            .and()
            .formLogin()
            .and()
            .httpBasic();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("user").password("{noop}password").roles("USER")
            .and()
            .withUser("admin").password("{noop}admin").roles("ADMIN");
    }
}
```

**Example (Spring Security 6+):**
```java
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults())
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.withUsername("user")
            .password("{noop}password")
            .roles("USER")
            .build();
        UserDetails admin = User.withUsername("admin")
            .password("{noop}admin")
            .roles("ADMIN")
            .build();
        return new InMemoryUserDetailsManager(user, admin);
    }
}
```

**Key Points:**
- `@EnableWebSecurity` enables Spring Security configuration.
- `configure(HttpSecurity http)` defines which URLs are secured and how.
- `configure(AuthenticationManagerBuilder auth)` or `UserDetailsService` defines user details and roles.
- The filter chain intercepts requests, checks authentication/authorization, and grants or denies access.

**Summary:**
Spring Security uses a filter chain to intercept requests, authenticate users, and authorize access based on your `SecurityConfig`. You can customize login, error handling, and integrate with databases, LDAP, JWT, OAuth2, etc.
</details>

---

<details>
<summary>How does JWT authentication workflow work in Spring Security? Explain with an example.</summary>

**Explanation:**
JWT (JSON Web Token) is a stateless authentication mechanism commonly used in REST APIs. The workflow involves issuing a signed token to the client after successful authentication, which the client then includes in subsequent requests.

**JWT Workflow Steps:**
1. **User Login:**
   - The client sends a POST request with username and password to the authentication endpoint (e.g., `/login`).
2. **Token Generation:**
   - If credentials are valid, the server generates a JWT containing user information and signs it with a secret key.
   - The JWT is sent back to the client in the response.
3. **Token Usage:**
   - The client stores the JWT (usually in localStorage or sessionStorage).
   - For each subsequent API request, the client includes the JWT in the `Authorization` header as `Bearer <token>`.
4. **Token Validation:**
   - The server intercepts incoming requests, extracts the JWT, and validates its signature and claims.
   - If valid, the request is processed; otherwise, a 401 Unauthorized error is returned.

**Example:**

*Authentication Controller:*
```java
@RestController
public class AuthController {
    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody AuthRequest request) {
        // Authenticate user (pseudo-code)
        if (isValidUser(request)) {
            String jwt = jwtUtil.generateToken(request.getUsername());
            return ResponseEntity.ok(new AuthResponse(jwt));
        } else {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
        }
    }
}
```

*JWT Filter:*
```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            if (jwtUtil.validateToken(token)) {
                // Set authentication in SecurityContext
            }
        }
        filterChain.doFilter(request, response);
    }
}
```

**Key Points:**
- JWTs are stateless; no session is stored on the server.
- The server only needs the secret key to validate tokens.
- JWTs can carry user roles and claims for authorization.
- Always use HTTPS to protect tokens in transit.

</details>

---

<details>
<summary>How does OAuth2 authentication workflow work? Explain with an example.</summary>

**Explanation:**
OAuth2 is an authorization framework that allows third-party applications to access a user's resources without sharing their credentials. It is widely used for SSO (Single Sign-On) and delegated access (e.g., "Login with Google").

**OAuth2 Workflow Steps (Authorization Code Grant):**
1. **User Requests Login:**
   - The client (e.g., web app) redirects the user to the OAuth2 provider's authorization endpoint (e.g., Google, GitHub).
2. **User Grants Permission:**
   - The user logs in and grants permission to the client application.
3. **Authorization Code Issued:**
   - The OAuth2 provider redirects the user back to the client with an authorization code.
4. **Token Exchange:**
   - The client exchanges the authorization code for an access token (and optionally a refresh token) by calling the provider's token endpoint.
5. **Access Protected Resources:**
   - The client uses the access token to call protected APIs on behalf of the user.

**Example:**

*Spring Boot OAuth2 Client Configuration (application.yml):*
```yaml
oauth2:
  client:
    registration:
      google:
        client-id: your-client-id
        client-secret: your-client-secret
        scope: openid,profile,email
    provider:
      google:
        authorization-uri: https://accounts.google.com/o/oauth2/v2/auth
        token-uri: https://oauth2.googleapis.com/token
        user-info-uri: https://openidconnect.googleapis.com/v1/userinfo
```

*Controller Example:*
```java
@RestController
public class UserController {
    @GetMapping("/user")
    public String user(@AuthenticationPrincipal OidcUser principal) {
        return "Hello, " + principal.getFullName();
    }
}
```

**Key Points:**
- OAuth2 separates roles: Resource Owner (user), Client (app), Authorization Server, Resource Server.
- Access tokens are used to access protected resources; refresh tokens can be used to obtain new access tokens.
- Spring Security provides auto-configuration for OAuth2 login and resource server.
- Always use HTTPS to protect tokens and credentials.

</details>

---

<details>
<summary>What is OAuth2 and OIDC? How are they used and what is their workflow?</summary>

**Explanation:**
- **OAuth2 (Open Authorization 2.0):**
  - An authorization framework that allows third-party applications to obtain limited access to a user's resources without exposing their credentials.
  - Used for delegated access (e.g., "Login with Google"), API security, and SSO (Single Sign-On).

- **OIDC (OpenID Connect):**
  - An identity layer built on top of OAuth2.
  - Adds authentication (verifying user identity) to OAuth2's authorization.
  - Returns an ID token (JWT) containing user identity information, in addition to OAuth2's access and refresh tokens.

**OAuth2 Workflow (Authorization Code Grant):**
1. User is redirected to the OAuth2 provider's authorization endpoint.
2. User authenticates and grants permission to the client application.
3. Provider redirects user back to the client with an authorization code.
4. Client exchanges the code for an access token (and optionally a refresh token).
5. Client uses the access token to access protected resources.

**OIDC Workflow:**
- Follows the OAuth2 flow, but the token response also includes an ID token (JWT) with user identity claims (e.g., name, email, sub).
- The client can use the ID token to authenticate the user and the access token to access APIs.

**Example (Spring Boot OIDC Login):**
```yaml
oauth2:
  client:
    registration:
      google:
        client-id: your-client-id
        client-secret: your-client-secret
        scope: openid,profile,email
```

```java
@RestController
public class UserController {
    @GetMapping("/user")
    public String user(@AuthenticationPrincipal OidcUser principal) {
        return "Hello, " + principal.getFullName();
    }
}
```

**Key Points:**
- OAuth2 is for authorization (access control), OIDC adds authentication (user identity).
- OIDC is widely used for SSO and federated identity (e.g., "Sign in with Google").
- Always use HTTPS to protect tokens and user data.

</details>

---

<details>
<summary>What is the structure of a JWT token with and without OIDC?</summary>

**Explanation:**
A JWT (JSON Web Token) consists of three parts: Header, Payload, and Signature, separated by dots (`.`):

```
<base64url-encoded header>.<base64url-encoded payload>.<base64url-encoded signature>
```

**1. JWT Token Structure (Without OIDC):**
- Used for authorization (e.g., API access)
- Contains claims like `sub` (subject), `exp` (expiration), `roles`, etc.

**Example:**
```json
// Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload
{
  "sub": "user123",
  "roles": ["USER"],
  "exp": 1719999999
}
```

**2. JWT Token Structure (With OIDC):**
- OIDC issues an ID token (JWT) with additional identity claims.
- Contains standard OIDC claims like `iss` (issuer), `aud` (audience), `email`, `name`, `iat` (issued at), `exp`, and `sub`.

**Example:**
```json
// Header
{
  "alg": "RS256",
  "typ": "JWT"
}

// Payload
{
  "iss": "https://accounts.google.com",
  "aud": "your-client-id",
  "sub": "1234567890",
  "email": "user@example.com",
  "name": "John Doe",
  "iat": 1719999000,
  "exp": 1719999999
}
```

**Key Points:**
- JWTs for authorization may only have user ID and roles.
- OIDC ID tokens include identity information (name, email, etc.) for authentication.
- Both are signed to ensure integrity and authenticity.

</details>

---

<details>
<summary>What are the different OAuth2 flows (grant types)? Explain each with examples.</summary>

**Explanation:**
OAuth2 defines several grant types (flows) for different use cases:

**1. Authorization Code Grant**
- Used by web and mobile apps for user login via a browser.
- Most secure; supports refresh tokens.
- **Example:**
  1. User is redirected to the provider's authorization endpoint.
  2. User logs in and grants access.
  3. Provider redirects back with an authorization code.
  4. App exchanges code for access token (and refresh token).

**2. Client Credentials Grant**
- Used for machine-to-machine (server-to-server) communication.
- No user context; only client credentials are used.
- **Example:**
  - App sends client_id and client_secret to the token endpoint.
  - Receives access token for API access.

**3. Resource Owner Password Credentials Grant**
- Used when the app is highly trusted (legacy or first-party apps).
- User provides username and password directly to the app.
- **Example:**
  - App sends username, password, client_id, and client_secret to the token endpoint.
  - Receives access token (and optionally refresh token).

**4. Implicit Grant**
- Used by single-page applications (SPAs) running in browsers.
- Access token is returned directly in the redirect URI (no refresh token).
- **Note:** Now considered less secure and generally discouraged.

**5. Device Authorization Grant**
- Used for devices with limited input (e.g., smart TVs).
- Device displays a code; user enters it on another device to authorize.
- **Example:**
  1. Device requests device/user code from the provider.
  2. User visits a URL on another device and enters the code.
  3. Device polls the token endpoint until access is granted.

**Example (Client Credentials Grant in Spring):**
```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          my-client:
            client-id: my-client-id
            client-secret: my-client-secret
            authorization-grant-type: client_credentials
            scope: api.read
```

**Key Points:**
- Choose the grant type based on your application's needs and security requirements.
- Authorization Code Grant is recommended for most user-facing apps.
- Client Credentials Grant is for backend services.
- Device Grant is for devices without browsers.

</details>

---

