# Spring Boot + Keycloak Role-Based Authentication

This project demonstrates how to integrate Spring Boot with Keycloak to implement secure authentication and role-based authorization (RBAC). Users authenticate using Keycloak, and access to REST endpoints is controlled based on assigned roles.

---

## Tech Stack

- Java 25+
- Spring Boot 4.0+
- Spring Security
- Keycloak (Docker)
- Maven
- REST API

---

## Features

- Keycloak authentication
- JWT token validation
- Role-based authorization
- Protected REST endpoints
- Admin and User roles

---

## Project Structure

src/main/java/com/example/demo  
├── config  
│    └── SecurityConfig.java  
├── controller  
│    └── TestController.java  
└── DemoApplication.java

---

## Run Keycloak Using Docker

Run the following command:
```bash
docker run -d --name keycloak -p 9090:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:latest start-dev
```
Access Keycloak Admin Console:

http://localhost:9090

Login Credentials:

Username: admin  
Password: admin

---

## Keycloak Configuration

1. Create a Realm:
    - Name: spring-realm

2. Create a Client:
    - Client ID: spring-client
    - Client Type: OpenID Connect
    - Access Type: Confidential
    - Valid Redirect URI:
      http://localhost:8080/*

3. Create Roles:
    - ROLE_USER
    - ROLE_ADMIN

4. Create Users:
    - Assign roles to users accordingly.

---

## Spring Boot Configuration

application.yml

server:
port: 8080

spring:
security:
oauth2:
resourceserver:
jwt:
issuer-uri: http://localhost:9090/realms/spring-realm

---

## Security Configuration

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public").permitAll()
                .requestMatchers("/user").hasRole("USER")
                .requestMatchers("/admin").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt());

        return http.build();
    }
}

---

## Sample Controller

@RestController
public class TestController {

    @GetMapping("/public")
    public String publicEndpoint() {
        return "Public Access";
    }

    @GetMapping("/user")
    public String userEndpoint() {
        return "User Access";
    }

    @GetMapping("/admin")
    public String adminEndpoint() {
        return "Admin Access";
    }
}

---

## Get Access Token

Use curl or Postman:

curl -X POST \
http://localhost:9090/realms/spring-realm/protocol/openid-connect/token \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=password" \
-d "client_id=spring-client" \
-d "username=your_username" \
-d "password=your_password"

Use the returned access_token in request headers:

Authorization: Bearer <access_token>

---

## Role-Based Access Table

Endpoint    | Required Role
------------|--------------
/public     | None
/user       | ROLE_USER
/admin      | ROLE_ADMIN

---

## Run Spring Boot Application

Using Maven:

mvn spring-boot:run

Or run the main class from your IDE.

---

## How It Works

1. User authenticates through Keycloak.
2. Keycloak issues a JWT access token.
3. Spring Boot validates the token using issuer URI.
4. Spring Security checks the user's role before granting access to endpoints.

---

## License

This project is open-source and available under the MIT License.