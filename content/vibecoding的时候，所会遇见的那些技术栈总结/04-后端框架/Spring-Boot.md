# Spring Boot 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Spring Boot 3.x 新特性、企业级架构、Spring AI 集成及 Java 后端选型分析。

---

## 目录

1. [[#Spring Boot 概述与定位]]
2. [[#Spring Boot 3.x 新特性]]
3. [[#自动配置原理]]
4. [[#Spring Data JPA]]
5. [[#Spring Security]]
6. [[#Spring MVC vs WebFlux]]
7. [[#Spring AI 集成]]
8. [[#企业级架构设计]]
9. [[#Spring Boot vs Django vs FastAPI 对比]]
10. [[#实战场景与选型建议]]
11. [[#参考资料]]

---

## Spring Boot 概述与定位

Spring Boot 是 Pivotal（现 VMware）于 2014 年发布的 Java 企业级框架，旨在简化 Spring 应用的创建和部署。Spring Boot 通过「约定优于配置」的原则和自动配置机制，大幅降低了 Spring 应用的配置复杂度，使开发者能够快速构建生产级别的 Java 应用。

### 核心定位

| 维度 | 定位 |
|------|------|
| **目标用户** | 企业级开发团队、微服务架构师 |
| **核心理念** | 约定优于配置、自动配置、开箱即用 |
| **应用场景** | 企业级应用、微服务、分布式系统、金融系统 |
| **性能等级** | Tiers 2（启动较慢，但运行时性能优秀） |
| **学习曲线** | 中-高（需要 Java 基础 + Spring 框架知识） |
| **生态系统** | 极其庞大（Spring Cloud、Spring Security、Spring Data 等） |

### 核心优势

1. **自动配置**：根据类路径中的依赖自动配置 Spring 应用
2. **嵌入式服务器**：无需部署 WAR 文件，内置 Tomcat/Jetty/Netty
3. **生产就绪**：内置健康检查、指标监控、外部化配置
4. **生态完整**：Spring Cloud（微服务）、Spring Security（安全）、Spring Data（数据访问）
5. **云原生支持**：原生支持 Kubernetes、Docker、12-Factor App

---

## Spring Boot 3.x 新特性

Spring Boot 3.0 于 2022 年发布，是一次重大版本更新，带来了 Java 17+ 基线、Jakarta EE 迁移、AOT 编译支持等重大变化。

### 核心新特性

| 特性 | 说明 | 优势 |
|------|------|------|
| **Java 17+ 基线** | 要求 Java 17 或更高版本 | 现代语言特性、性能提升 |
| **Jakarta EE 10** | 从 javax.* 迁移到 jakarta.* | 现代化的企业级标准 |
| **AOT 编译** | 提前编译支持 | 更快的启动时间 |
| **GraalVM 支持** | 原生镜像构建 | 毫秒级启动 |
| **Observability** | 统一的可观测性 API | 标准化监控 |
| **Pattern Palettes** | 新的配置模式 | 更灵活的配置 |

### 项目结构

```
spring-boot-project/
├── pom.xml 或 build.gradle
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/app/
│   │   │       ├── Application.java
│   │   │       ├── config/
│   │   │       │   ├── SecurityConfig.java
│   │   │       │   └── WebConfig.java
│   │   │       ├── controller/
│   │   │       │   └── UserController.java
│   │   │       ├── service/
│   │   │       │   ├── UserService.java
│   │   │       │   └── impl/
│   │   │       │       └── UserServiceImpl.java
│   │   │       ├── repository/
│   │   │       │   └── UserRepository.java
│   │   │       ├── model/
│   │   │       │   └── User.java
│   │   │       ├── dto/
│   │   │       │   └── UserDTO.java
│   │   │       └── exception/
│   │   │           ├── GlobalExceptionHandler.java
│   │   │           └── ResourceNotFoundException.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       └── db/migration/
│   │           └── V1__init.sql
│   └── test/
│       └── java/
│           └── com/example/app/
│               ├── controller/
│               │   └── UserControllerTest.java
│               └── service/
│                   └── UserServiceTest.java
└── docker-compose.yml
```

### 基础应用

```java
// Application.java
package com.example.app;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * Spring Boot 应用主类
 * @SpringBootApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan
 */
@SpringBootApplication
@EnableConfigurationProperties(AppProperties.class)
public class Application {
    
    public static void main(String[] args) {
        // 启动应用
        SpringApplication.run(Application.class, args);
    }
    
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("https://example.com")
                    .allowedMethods("GET", "POST", "PUT", "DELETE")
                    .allowedHeaders("*")
                    .allowCredentials(true);
            }
        };
    }
}

// application.yml
spring:
  application:
    name: myapp
  profiles:
    active: dev
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: user
    password: pass
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
  flyway:
    enabled: true
    locations: classpath:db/migration

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized

logging:
  level:
    com.example: DEBUG
    org.springframework.web: INFO
```

---

## 自动配置原理

### @SpringBootApplication 核心

```java
package org.springframework.boot.autoconfigure;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class))
public @interface SpringBootApplication {
    
    @AliasFor(annotation = ComponentScan.class)
    String[] exclude() default {};
    
    @AliasFor(annotation = EnableAutoConfiguration.class)
    String[] excludeName() default {};
    
    @AliasFor(annotation = EnableAutoConfiguration.class)
    String[] excludeName() default {};
}
```

### 自动配置流程

```
应用启动
    ↓
@SpringBootApplication 触发
    ↓
@EnableAutoConfiguration 启用
    ↓
扫描 classpath:META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
    ↓
根据条件（@Conditional*）筛选配置类
    ↓
注册 Bean 定义
    ↓
应用启动完成
```

### 自定义自动配置

```java
// 1. 定义配置属性
@ConfigurationProperties(prefix = "app.feature")
public class FeatureProperties {
    private boolean enabled = true;
    private String apiKey;
    private List<String> allowedOrigins;
    
    // Getters and Setters
}

// 2. 创建自动配置类
@AutoConfiguration
@ConditionalOnClass(FeatureClient.class)
@ConditionalOnProperty(prefix = "app.feature", name = "enabled", havingValue = "true")
@EnableConfigurationProperties(FeatureProperties.class)
public class FeatureAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public FeatureClient featureClient(FeatureProperties properties) {
        return new FeatureClient(properties.getApiKey());
    }
}

// 3. 注册自动配置
// src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
// com.example.app.autoconfigure.FeatureAutoConfiguration

// 4. 用户使用
// application.yml
app:
  feature:
    enabled: true
    api-key: your-api-key
```

### 条件注解体系

| 注解 | 条件 |
|------|------|
| `@ConditionalOnClass` | 类路径中存在指定类 |
| `@ConditionalOnMissingClass` | 类路径中不存在指定类 |
| `@ConditionalOnBean` | 容器中存在指定 Bean |
| `@ConditionalOnMissingBean` | 容器中不存在指定 Bean |
| `@ConditionalOnProperty` | 配置属性满足条件 |
| `@ConditionalOnResource` | 存在指定资源文件 |
| `@ConditionalOnWebApplication` | 是 Web 应用 |

---

## Spring Data JPA

### 实体定义

```java
package com.example.app.model;

import jakarta.persistence.*;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import org.hibernate.annotations.*;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;
import java.util.HashSet;
import java.util.Set;

/**
 * 用户实体
 */
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_user_email", columnList = "email"),
    @Index(name = "idx_user_username", columnList = "username")
})
@EntityListeners(AuditingEntityListener.class)
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 50, message = "用户名长度必须在3-50之间")
    @Column(unique = true, nullable = false)
    private String username;
    
    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    @Column(unique = true, nullable = false)
    private String email;
    
    @NotBlank(message = "密码不能为空")
    @Column(name = "password_hash", nullable = false)
    private String passwordHash;
    
    @Column(name = "is_active")
    private boolean active = true;
    
    @Column(name = "is_admin")
    private boolean admin = false;
    
    // 枚举类型
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserStatus status = UserStatus.PENDING;
    
    // 一对多关系
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JsonIgnore
    private Set<Post> posts = new HashSet<>();
    
    // 多对多关系
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
    
    // 审计字段
    @CreatedDate
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // 乐观锁
    @Version
    private Long version;
    
    // Getters and Setters
}

enum UserStatus {
    PENDING, ACTIVE, SUSPENDED, DELETED
}
```

### Repository 定义

```java
package com.example.app.repository;

import com.example.app.model.User;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.Optional;

/**
 * 用户数据访问层
 */
@Repository
public interface UserRepository extends 
    JpaRepository<User, Long>,           // CRUD 操作
    JpaSpecificationExecutor<User>,     // 动态查询
    UserRepositoryCustom {               // 自定义查询接口
    
    // 方法命名查询
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
    
    // 分页查询
    Page<User> findByActiveTrue(Pageable pageable);
    
    // 模糊搜索
    Page<User> findByUsernameContainingIgnoreCase(String keyword, Pageable pageable);
    
    // 自定义 JPQL 查询
    @Query("SELECT u FROM User u WHERE u.email = :email AND u.active = true")
    Optional<User> findActiveUserByEmail(@Param("email") String email);
    
    // 原生 SQL 查询
    @Query(value = "SELECT * FROM users WHERE DATE(created_at) = CURRENT_DATE", nativeQuery = true)
    List<User> findTodayRegisteredUsers();
    
    // 统计查询
    @Query("SELECT COUNT(u) FROM User u WHERE u.admin = false")
    long countNonAdminUsers();
    
    // 删除查询（带验证）
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.id = :id")
    int softDeleteUser(@Param("id") Long id);
}

// 自定义查询接口
public interface UserRepositoryCustom {
    Page<User> searchUsers(String keyword, Set<String> roles, Pageable pageable);
}

// 自定义查询实现
public class UserRepositoryImpl implements UserRepositoryCustom {
    
    @PersistenceContext
    private EntityManager em;
    
    @Override
    public Page<User> searchUsers(String keyword, Set<String> roles, Pageable pageable) {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> root = query.from(User.class);
        
        // 动态构建查询条件
        List<Predicate> predicates = new ArrayList<>();
        
        if (keyword != null && !keyword.isEmpty()) {
            predicates.add(cb.or(
                cb.like(cb.lower(root.get("username")), "%" + keyword.toLowerCase() + "%"),
                cb.like(cb.lower(root.get("email")), "%" + keyword.toLowerCase() + "%")
            ));
        }
        
        if (roles != null && !roles.isEmpty()) {
            predicates.add(root.join("roles").get("name").in(roles));
        }
        
        query.where(predicates.toArray(new Predicate[0]));
        query.orderBy(cb.desc(root.get("createdAt")));
        
        // 执行分页查询
        TypedQuery<User> typedQuery = em.createQuery(query);
        typedQuery.setFirstResult((int) pageable.getOffset());
        typedQuery.setMaxResults(pageable.getPageSize());
        
        // 计数查询
        CriteriaQuery<Long> countQuery = cb.createQuery(Long.class);
        countQuery.select(cb.count(countQuery.from(User.class)));
        countQuery.where(predicates.toArray(new Predicate[0]));
        
        long total = em.createQuery(countQuery).getSingleResult();
        
        return new PageImpl<>(typedQuery.getResultList(), pageable, total);
    }
}
```

### Service 层

```java
package com.example.app.service;

import com.example.app.dto.UserDTO;
import com.example.app.exception.ResourceNotFoundException;
import com.example.app.model.User;
import com.example.app.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.Set;

/**
 * 用户服务层
 */
@Service
@RequiredArgsConstructor
@Transactional
public class UserService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    
    /**
     * 分页获取用户列表
     */
    @Transactional(readOnly = true)
    public Page<UserDTO> getUsers(Pageable pageable) {
        return userRepository.findAll(pageable)
            .map(this::toDTO);
    }
    
    /**
     * 根据 ID 获取用户
     */
    @Transactional(readOnly = true)
    public UserDTO getUserById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
        return toDTO(user);
    }
    
    /**
     * 创建用户
     */
    public UserDTO createUser(UserDTO dto) {
        // 验证用户名唯一性
        if (userRepository.existsByUsername(dto.getUsername())) {
            throw new IllegalArgumentException("Username already exists");
        }
        
        // 验证邮箱唯一性
        if (userRepository.existsByEmail(dto.getEmail())) {
            throw new IllegalArgumentException("Email already exists");
        }
        
        User user = new User();
        user.setUsername(dto.getUsername());
        user.setEmail(dto.getEmail());
        user.setPasswordHash(passwordEncoder.encode(dto.getPassword()));
        user.setActive(true);
        
        User saved = userRepository.save(user);
        return toDTO(saved);
    }
    
    /**
     * 更新用户
     */
    public UserDTO updateUser(Long id, UserDTO dto) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
        
        // 更新字段
        if (dto.getUsername() != null) {
            user.setUsername(dto.getUsername());
        }
        if (dto.getEmail() != null) {
            user.setEmail(dto.getEmail());
        }
        
        User updated = userRepository.save(user);
        return toDTO(updated);
    }
    
    /**
     * 删除用户（软删除）
     */
    public void deleteUser(Long id) {
        int affected = userRepository.softDeleteUser(id);
        if (affected == 0) {
            throw new ResourceNotFoundException("User not found");
        }
    }
    
    /**
     * 实体转 DTO
     */
    private UserDTO toDTO(User user) {
        return UserDTO.builder()
            .id(user.getId())
            .username(user.getUsername())
            .email(user.getEmail())
            .active(user.isActive())
            .roles(user.getRoles().stream()
                .map(Role::getName)
                .collect(Collectors.toSet()))
            .createdAt(user.getCreatedAt())
            .build();
    }
}
```

---

## Spring Security

### 安全配置

```java
package com.example.app.config;

import com.example.app.security.JwtAuthenticationFilter;
import com.example.app.security.JwtTokenProvider;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
@RequiredArgsConstructor
public class SecurityConfig {
    
    private final JwtTokenProvider jwtTokenProvider;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            // CSRF 禁用（使用 JWT）
            .csrf(AbstractHttpConfigurer::disable)
            
            // CORS 配置
            .cors(cors -> cors.configure(http))
            
            // 会话管理：无状态
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            
            // 授权规则
            .authorizeHttpRequests(auth -> auth
                // 公开端点
                .requestMatchers("/api/auth/**", "/api/public/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                
                // 管理端点
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                
                // 其他端点需要认证
                .anyRequest().authenticated()
            )
            
            // JWT 过滤器
            .addFilterBefore(
                new JwtAuthenticationFilter(jwtTokenProvider),
                UsernamePasswordAuthenticationFilter.class
            )
            
            .build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

### JWT 认证

```java
package com.example.app.security;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import jakarta.annotation.PostConstruct;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.*;
import java.util.stream.Collectors;

@Component
@Slf4j
public class JwtTokenProvider {
    
    @Value("${app.jwt.secret}")
    private String jwtSecret;
    
    @Value("${app.jwt.expiration-ms:86400000}")
    private long jwtExpirationMs;
    
    private SecretKey key;
    
    @PostConstruct
    public void init() {
        this.key = Keys.hmacShaKeyFor(jwtSecret.getBytes(StandardCharsets.UTF_8));
    }
    
    /**
     * 生成 Token
     */
    public String generateToken(Authentication authentication) {
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + jwtExpirationMs);
        
        String roles = userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.joining(","));
        
        return Jwts.builder()
            .subject(userDetails.getUsername())
            .claim("roles", roles)
            .issuedAt(now)
            .expiration(expiryDate)
            .signWith(key)
            .compact();
    }
    
    /**
     * 从 Token 获取认证信息
     */
    public Authentication getAuthentication(String token) {
        Claims claims = Jwts.parser()
            .verifyWith(key)
            .build()
            .parseSignedClaims(token)
            .getPayload();
        
        String username = claims.getSubject();
        String roles = claims.get("roles", String.class);
        
        Collection<GrantedAuthority> authorities = Arrays.stream(roles.split(","))
            .filter(role -> !role.isEmpty())
            .map(SimpleGrantedAuthority::new)
            .collect(Collectors.toList());
        
        return new UsernamePasswordAuthenticationToken(username, null, authorities);
    }
    
    /**
     * 验证 Token
     */
    public boolean validateToken(String token) {
        try {
            Jwts.parser().verifyWith(key).build().parseSignedClaims(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            log.error("Invalid JWT token: {}", e.getMessage());
            return false;
        }
    }
}
```

---

## Spring MVC vs WebFlux

### 技术选型对比

| 维度 | Spring MVC | Spring WebFlux |
|------|------------|----------------|
| **编程模型** | 阻塞式 | 响应式 |
| **线程模型** | 每请求一线程 | 事件驱动，少量线程 |
| **吞吐量** | 中等 | 高（适合 IO 密集） |
| **延迟** | 毫秒级 | 微秒级（under load） |
| **学习曲线** | 低 | 高 |
| **生态系统** | 完整 | 有限（需要响应式库） |
| **调试** | 简单 | 复杂 |
| **适用场景** | CRUD、CPU 密集 | 高并发 IO、微服务 |

### Spring MVC 示例

```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    
    private final UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }
    
    @GetMapping
    public ResponseEntity<Page<UserDTO>> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        Pageable pageable = PageRequest.of(page, size);
        return ResponseEntity.ok(userService.getUsers(pageable));
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public UserDTO createUser(@Valid @RequestBody UserDTO dto) {
        return userService.createUser(dto);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<UserDTO> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UserDTO dto) {
        return ResponseEntity.ok(userService.updateUser(id, dto));
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
    }
}
```

### Spring WebFlux 示例

```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserReactiveController {
    
    private final UserReactiveService userService;
    
    @GetMapping("/{id}")
    public Mono<ResponseEntity<UserDTO>> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }
    
    @GetMapping
    public Flux<ResponseEntity<UserDTO>> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return userService.findAll(page, size)
            .map(ResponseEntity::ok);
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<UserDTO> createUser(@Valid @RequestBody Mono<UserDTO> dtoMono) {
        return userService.createUser(dtoMono);
    }
    
    @PutMapping("/{id}")
    public Mono<ResponseEntity<UserDTO>> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody Mono<UserDTO> dtoMono) {
        return userService.updateUser(id, dtoMono)
            .map(ResponseEntity::ok)
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public Mono<Void> deleteUser(@PathVariable Long id) {
        return userService.deleteUser(id);
    }
}

// 响应式服务
@Service
@RequiredArgsConstructor
public class UserReactiveService {
    
    private final ReactiveUserRepository userRepository;
    
    public Mono<User> findById(Long id) {
        return userRepository.findById(id);
    }
    
    public Flux<User> findAll(int page, int size) {
        return userRepository.findAll()
            .skip((long) page * size)
            .take(size);
    }
    
    public Mono<User> createUser(Mono<UserDTO> dtoMono) {
        return dtoMono
            .map(this::toEntity)
            .flatMap(userRepository::save);
    }
}
```

---

## Spring AI 集成

### Spring AI 模块

| 模块 | 功能 |
|------|------|
| **spring-ai-openai** | OpenAI API 集成 |
| **spring-ai-anthropic** | Anthropic Claude 集成 |
| **spring-ai-googleai** | Google Gemini 集成 |
| **spring-ai-vertex-ai** | Google Vertex AI 集成 |
| **spring-ai-huggingface** | Hugging Face 模型 |
| **spring-ai-ollama** | Ollama 本地模型 |
| **spring-ai-deepseek** | DeepSeek API |
| **spring-ai-azure-openai** | Azure OpenAI |
| **spring-ai-neo4j** | Neo4j 图数据库 |
| **spring-ai-pgvector** | PostgreSQL 向量存储 |
| **spring-ai-qdrant** | Qdrant 向量数据库 |
| **spring-ai-chromadb** | Chroma 向量数据库 |

### 基础配置

```java
// application.yml
spring:
  application:
    name: ai-app
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      base-url: https://api.openai.com/v1
      chat:
        options:
          model: gpt-4o
          temperature: 0.7
    deepseek:
      api-key: ${DEEPSEEK_API_KEY}
      base-url: https://api.deepseek.com
      chat:
        options:
          model: deepseek-chat

// 配置类
@Configuration
public class AIConfig {
    
    @Bean
    public OpenAiApi openAiApi(@Value("${spring.ai.openai.api-key}") String apiKey) {
        return OpenAiApi.builder()
            .apiKey(apiKey)
            .build();
    }
    
    @Bean
    public ChatModel chatModel(OpenAiApi openAiApi) {
        return OpenAiChatModel.builder()
            .openAiApi(openAiApi)
            .defaultOptions(OpenAiChatOptions.builder()
                .temperature(0.7f)
                .build())
            .build();
    }
}
```

### ChatClient 使用

```java
package com.example.app.service;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.client.advisor.QuestionAnswerAdvisor;
import org.springframework.ai.chat.model.ChatResponse;
import org.springframework.ai.vectorstore.SearchRequest;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;

@Service
@RequiredArgsConstructor
@Slf4j
public class AIChatService {
    
    private final ChatClient chatClient;
    private final VectorStore vectorStore;
    
    /**
     * 简单对话
     */
    public String chat(String userMessage) {
        return chatClient.prompt()
            .user(userMessage)
            .call()
            .content();
    }
    
    /**
     * 带系统提示的对话
     */
    public String chatWithSystem(String userMessage, String systemPrompt) {
        return chatClient.prompt()
            .system(systemPrompt)
            .user(userMessage)
            .call()
            .content();
    }
    
    /**
     * 多轮对话
     */
    public String multiTurnChat(String userMessage) {
        return chatClient.prompt()
            .user(userMessage)
            .advisors(new LoggerAdvisor())
            .call()
            .content();
    }
    
    /**
     * RAG 问答
     */
    public String ragQuestion(String question) {
        return chatClient.prompt()
            .user(question)
            .advisors(QuestionAnswerAdvisor.builder(vectorStore)
                .searchRequest(SearchRequest.builder()
                    .topK(5)
                    .similarityThreshold(0.7)
                    .build())
                .build())
            .call()
            .content();
    }
    
    /**
     * 流式对话
     */
    public Flux<String> streamChat(String userMessage) {
        return chatClient.prompt()
            .user(userMessage)
            .stream()
            .content();
    }
    
    /**
     * 结构化输出
     */
    public UserProfile extractUserProfile(String text) {
        return chatClient.prompt()
            .user("从以下文本中提取用户信息: " + text)
            .call()
            .entity(UserProfile.class);
    }
}
```

### 向量数据库集成

```java
package com.example.app.config;

import org.springframework.ai.vectorstore.pgvector.PgVectorStore;
import org.springframework.ai.vectorstore.SimpleVectorStore;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jdbc.core.JdbcAggregateTemplate;
import org.springframework.jdbc.core.JdbcTemplate;

@Configuration
public class VectorStoreConfig {
    
    // PostgreSQL + PGvector
    @Bean
    public VectorStore pgVectorStore(JdbcTemplate jdbcTemplate) {
        return PgVectorStore.builder(jdbcTemplate)
            .vectorTableName("vectors")
            .dimension(1536)  // OpenAI embeddings dimension
            .initializeSchema(true)
            .build();
    }
}

// 文档存储服务
@Service
@RequiredArgsConstructor
public class DocumentService {
    
    private final VectorStore vectorStore;
    private final EmbeddingModel embeddingModel;
    private final DocumentReader documentReader;
    
    /**
     * 文档分块并存储
     */
    public void ingestDocument(String filePath) {
        // 读取文档
        List<Document> documents = documentReader.read(filePath);
        
        // 文本分块
        List<Document> chunks = new TextSplitter()
            .splitText(documents, 500, 100);  // 500字，100字重叠
        
        // 存储到向量数据库
        vectorStore.add(chunks);
    }
    
    /**
     * 相似度搜索
     */
    public List<Document> similaritySearch(String query, int topK) {
        return vectorStore.similaritySearch(
            SearchRequest.builder()
                .query(query)
                .topK(topK)
                .build()
        );
    }
}
```

---

## 企业级架构设计

### 分层架构

```
┌─────────────────────────────────────────────┐
│                Presentation                │
│  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Controller │  │   View (Templates)   │ │
│  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│                  Service                    │
│  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Business   │  │   Transaction       │ │
│  │   Logic     │  │   Management        │ │
│  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│              Repository / DAO              │
│  ┌─────────────┐  ┌─────────────────────┐ │
│  │    JPA      │  │    MyBatis         │ │
│  │  Repository │  │    XML Mapper       │ │
│  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│               Data Sources                  │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────────┐  │
│  │ PostgreSQL│ │ MySQL │ │ Redis │ │ VectorDB│ │
│  └──────┘ └──────┘ └──────┘ └──────────┘  │
└─────────────────────────────────────────────┘
```

### 微服务通信

```java
// REST 客户端
@Service
@RequiredArgsConstructor
public class UserServiceClient {
    
    private final RestClient restClient;
    
    public User getUserById(Long id) {
        return restClient.get()
            .uri("http://user-service/api/users/{id}", id)
            .retrieve()
            .body(User.class);
    }
    
    public User createUser(CreateUserRequest request) {
        return restClient.post()
            .uri("http://user-service/api/users")
            .body(request)
            .retrieve()
            .body(User.class);
    }
}

// 配置
@Configuration
public class RestClientConfig {
    
    @Bean
    public RestClient restClient(RoadmapBalancerLoadBalancerFactory factory) {
        return RestClient.builder()
            .baseUrl("http://user-service")
            .defaultHeader("Content-Type", "application/json")
            .requestFactory(new HttpComponentsClientHttpRequestFactory())
            .build();
    }
}

// OpenFeign 客户端
@FeignClient(name = "user-service", url = "${services.user.url}")
public interface UserFeignClient {
    
    @GetMapping("/api/users/{id}")
    User getUserById(@PathVariable("id") Long id);
    
    @PostMapping("/api/users")
    User createUser(@RequestBody CreateUserRequest request);
}
```

---

## Spring Boot vs Django vs FastAPI 对比

### 企业视角对比表

| 维度 | Spring Boot | Django | FastAPI |
|------|-------------|--------|---------|
| **定位** | 企业级应用框架 | 全栈 Web 框架 | 现代 API 框架 |
| **语言** | Java/Kotlin | Python | Python |
| **类型系统** | 静态类型（编译期检查） | 动态类型（可选注解） | 类型注解（运行时验证） |
| **性能** | 中-高（启动慢，运行快） | 中 | 高 |
| **并发模型** | 线程池 | 同步/可选异步 | 原生异步 |
| **ORM** | JPA/Hibernate | Django ORM | SQLAlchemy |
| **管理后台** | 需要开发 | Django Admin（内置） | 需要开发 |
| **认证** | Spring Security | Django Auth | 需自行实现 |
| **生态** | 极其庞大 | 成熟 | 成长中 |
| **学习曲线** | 高 | 中 | 低 |
| **启动时间** | 10-30秒 | 1-2秒 | <1秒 |
| **团队要求** | Java 专家 | Python 全栈 | Python 开发者 |
| **适用场景** | 金融、企业级、长期项目 | 内容网站、快速开发 | API、微服务、AI |

### Spring AI 模块表

| 模块 | 说明 | 使用场景 |
|------|------|---------|
| **spring-ai-openai** | OpenAI GPT 模型 | 通用对话、代码生成 |
| **spring-ai-anthropic** | Claude 模型 | 长文本处理 |
| **spring-ai-googleai** | Gemini 模型 | 多模态、超长上下文 |
| **spring-ai-deepseek** | DeepSeek 模型 | 高性价比 |
| **spring-ai-ollama** | 本地模型 | 数据隐私场景 |
| **spring-ai-pgvector** | PostgreSQL 向量存储 | RAG 知识库 |
| **spring-ai-qdrant** | Qdrant 向量数据库 | 高性能向量检索 |
| **spring-ai-redis** | Redis 向量存储 | 缓存向量 |

### 选型决策树

```
项目类型
├── 企业级核心系统
│   └── → Spring Boot（Java 生态、性能、稳定）
├── 内容管理系统
│   ├── 快速开发 → Django
│   └── Python 团队 → Django
├── AI 应用后端
│   └── → FastAPI / Spring AI
├── 微服务架构
│   ├── Java 团队 → Spring Boot
│   ├── Python 团队 → FastAPI
│   └── 性能优先 → FastAPI
└── 团队技能
    ├── Java 专家 → Spring Boot
    ├── Python 全栈 → Django / FastAPI
    └── 快速原型 → FastAPI
```

> [!TIP]
> **Spring Boot 的核心竞争力**：在需要严格类型安全、高性能运行时、企业级安全、复杂事务管理的场景下，Spring Boot 凭借其成熟的生态和 Java 的静态类型系统，是大型企业项目的首选。

---

## 实战场景与选型建议

### 场景一：企业级 RESTful API

```java
// 统一响应结构
public record ApiResponse<T>(
    boolean success,
    String message,
    T data,
    LocalDateTime timestamp
) {
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, "Success", data, LocalDateTime.now());
    }
    
    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, message, null, LocalDateTime.now());
    }
}

// 控制器
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
@Slf4j
public class UserController {
    
    private final UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<UserDTO>> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(ApiResponse.success(userService.getUserById(id)));
    }
    
    @GetMapping
    public ResponseEntity<ApiResponse<Page<UserDTO>>> getUsers(
            @ParameterObject Pageable pageable) {
        return ResponseEntity.ok(ApiResponse.success(userService.getUsers(pageable)));
    }
}
```

### 场景二：RAG 知识库后端

```java
@Service
@RequiredArgsConstructor
public class KnowledgeBaseService {
    
    private final ChatClient chatClient;
    private final VectorStore vectorStore;
    private final DocumentRepository documentRepository;
    
    public String query(String question) {
        return chatClient.prompt()
            .user(question)
            .advisors(QuestionAnswerAdvisor.builder(vectorStore)
                .searchRequest(SearchRequest.builder()
                    .topK(5)
                    .similarityThreshold(0.7)
                    .build())
                .build())
            .call()
            .content();
    }
    
    public void ingestDocument(MultipartFile file) throws IOException {
        String content = new String(file.getBytes());
        Document document = new Document(content, Map.of(
            "filename", file.getOriginalFilename()
        ));
        
        vectorStore.add(List.of(document));
        
        DocumentEntity entity = new DocumentEntity();
        entity.setFilename(file.getOriginalFilename());
        entity.setContent(content);
        documentRepository.save(entity);
    }
}
```

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| Spring Boot 文档 | https://spring.io/projects/spring-boot |
| Spring AI 文档 | https://spring.io/projects/spring-ai |
| Spring Data JPA | https://spring.io/projects/spring-data-jpa |
| Spring Security | https://spring.io/projects/spring-security |

### 学习资源

| 资源 | 说明 |
|------|------|
| Baeldung | 最好的 Spring 教程网站 |
| Spring Blog | 官方博客 |
| Spring Guides | 官方快速入门 |

---

> [!SUCCESS]
> 本文档全面覆盖了 Spring Boot 3.x 的核心知识，包括自动配置原理、Spring Data JPA、Spring Security、WebFlux 响应式编程及 Spring AI 集成。Spring Boot 凭借其成熟的企业级生态、严格的类型安全和完整的微服务支持，在 2026 年仍然是大型企业项目的首选 Java 框架。

---

## 完整安装与环境配置

### 环境要求

```bash
# Java 版本要求
java --version  # >= 17 (Spring Boot 3.x)
java --version  # >= 11 (Spring Boot 2.x)

# Maven 或 Gradle
mvn --version   # Maven 3.8+
./gradlew --version  # Gradle 8+
```

### 项目创建

```bash
# 使用 Spring Initializr CLI
curl https://start.spring.io/starter.zip \
  -d type=maven-project \
  -d language=java \
  -d bootVersion=3.2.0 \
  -d baseDir=myapp \
  -d groupId=com.example \
  -d artifactId=myapp \
  -d name=myapp \
  -d packageName=com.example.myapp \
  -d javaVersion=17 \
  -d dependencies=web,data-jpa,security,validation \
  -o myapp.zip && unzip myapp.zip && rm myapp.zip
```

### Maven 配置

```xml
<!-- pom.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>myapp</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Data -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- Database -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Security -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
        </dependency>

        <!-- Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- AI -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
        </dependency>

        <!-- Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 项目结构

```
myapp/
├── src/
│   ├── main/
│   │   ├── java/com/example/myapp/
│   │   │   ├── MyAppApplication.java
│   │   │   ├── config/
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── model/
│   │   │   ├── dto/
│   │   │   └── exception/
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/
│   └── test/
├── pom.xml
└── README.md
```

### application.yml 配置

```yaml
# application.yml
spring:
  application:
    name: myapp
  
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}

  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:myapp}
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:password}
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5

  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true

  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}

server:
  port: ${PORT:8080}

logging:
  level:
    root: INFO
    com.example.myapp: DEBUG
```

---

## 数据访问层

### JPA 实体定义

```java
// model/entity/User.java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_users_email", columnList = "email"),
    @Index(name = "idx_users_username", columnList = "username")
})
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 255)
    private String email;

    @Column(nullable = false, unique = true, length = 50)
    private String username;

    @Column(name = "password_hash", nullable = false, length = 255)
    private String passwordHash;

    @Column(name = "first_name", length = 100)
    private String firstName;

    @Column(name = "last_name", length = 100)
    private String lastName;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    @Builder.Default
    private Role role = Role.USER;

    @Column(name = "is_active", nullable = false)
    @Builder.Default
    private Boolean isActive = true;

    @CreationTimestamp
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
}
```

### Repository 定义

```java
// repository/UserRepository.java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);

    Optional<User> findByUsername(String username);

    boolean existsByEmail(String email);

    boolean existsByUsername(String username);

    @Query("SELECT u FROM User u WHERE u.isActive = true AND " +
           "(LOWER(u.email) LIKE LOWER(CONCAT('%', :search, '%')) OR " +
           "LOWER(u.username) LIKE LOWER(CONCAT('%', :search, '%')))")
    Page<User> searchUsers(@Param("search") String search, Pageable pageable);
}
```

---

## 常见陷阱与最佳实践

### 陷阱 1：N+1 查询问题

```java
// ❌ 错误：N+1 查询
@GetMapping
public List<UserDTO> getUsers() {
    return userRepository.findAll().stream()
        .map(user -> {
            List<Post> posts = postRepository.findByAuthor(user.getId());
            return toDTO(user, posts);
        })
        .collect(Collectors.toList());
}

// ✅ 正确：使用 JOIN FETCH
@Query("SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.posts")
List<User> findAllWithPosts();
```

### 陷阱 2：未使用分页

```java
// ❌ 错误：返回所有数据
@GetMapping("/posts")
public List<Post> getAllPosts() {
    return postRepository.findAll();  // 可能返回数百万条记录
}

// ✅ 正确：使用分页
@GetMapping("/posts")
public Page<Post> getAllPosts(Pageable pageable) {
    return postRepository.findPublishedPosts(pageable);
}
```

### 最佳实践清单

1. **分层架构**：Controller → Service → Repository。
2. **DTO 使用**：使用 DTO 隔离实体和 API 响应。
3. **事务管理**：在 Service 层使用 `@Transactional`。
4. **异常处理**：使用全局异常处理器统一处理。
5. **分页**：始终使用分页查询大量数据。
6. **索引**：为常用查询字段添加数据库索引。
7. **缓存**：使用 Spring Cache 减少数据库查询。
8. **异步**：使用 `@Async` 处理耗时任务。
9. **安全**：使用 Spring Security 保护 API。
10. **测试**：编写单元测试和集成测试。

---

## 与其他框架对比

### Spring Boot vs Django

| 特性 | Spring Boot | Django |
|------|-------------|--------|
| **语言** | Java/Kotlin | Python |
| **ORM** | JPA/Hibernate | Django ORM |
| **性能** | 极高 | 中等 |
| **类型** | 强类型 | 动态类型 |
| **生态** | 庞大企业级 | 全栈 |
| **适用场景** | 企业级后端 | 全栈应用 |

### Spring Boot vs FastAPI

| 特性 | Spring Boot | FastAPI |
|------|-------------|---------|
| **语言** | Java | Python |
| **性能** | 极高 | 极高 |
| **类型** | 强类型 | 强类型 |
| **异步** | WebFlux | 原生异步 |
| **学习曲线** | 较陡 | 平缓 |
| **适用场景** | 企业级应用 | 现代 API |

### Spring Boot vs Express

| 特性 | Spring Boot | Express |
|------|-------------|---------|
| **语言** | Java | JavaScript |
| **类型** | 强类型 | 弱类型/TS |
| **性能** | 极高 | 高 |
| **生态** | 庞大 | 增长中 |
| **适用场景** | 企业级应用 | 轻量 API |

---

---

## 核心概念详解

### Spring IoC 容器深入理解

Spring 框架的核心是控制反转（Inversion of Control）容器，也称为依赖注入（Dependency Injection）容器。深入理解容器的生命周期和 Bean 的作用域对于构建高效的企业级应用至关重要。

#### Bean 生命周期

```java
package com.example.app.lifecycle;

import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

/**
 * Bean 生命周期演示
 * Spring Bean 从创建到销毁的完整过程
 */
@Component
public class LifecycleBean implements 
    InitializingBean, 
    DisposableBean,
    BeanNameAware,
    BeanFactoryAware,
    ApplicationContextAware,
    SmartInitializingSingleton {
    
    private String beanName;
    private BeanFactory beanFactory;
    private ApplicationContext applicationContext;
    
    // 1. 设置 Bean 名称（Aware 接口）
    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("[Lifecycle] Bean name set: " + name);
    }
    
    // 2. 设置 Bean 工厂（Aware 接口）
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
        System.out.println("[Lifecycle] BeanFactory set");
    }
    
    // 3. 设置应用上下文（Aware 接口）
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
        System.out.println("[Lifecycle] ApplicationContext set");
    }
    
    // 4. Bean 构造方法执行
    public LifecycleBean() {
        System.out.println("[Lifecycle] Constructor called");
    }
    
    // 5. 依赖注入完成
    // 此阶段可以进行自定义属性设置
    
    // 6. PostConstruct 初始化方法
    @PostConstruct
    public void postConstruct() {
        System.out.println("[Lifecycle] @PostConstruct called");
    }
    
    // 7. InitializingBean.afterPropertiesSet()
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("[Lifecycle] afterPropertiesSet() called");
    }
    
    // 8. 自定义 init-method
    public void customInit() {
        System.out.println("[Lifecycle] customInit() called");
    }
    
    // 9. SmartInitializingSingleton.afterSingletonsInstantiated()
    @Override
    public void afterSingletonsInstantiated() {
        System.out.println("[Lifecycle] afterSingletonsInstantiated() called");
    }
    
    // Bean 使用中...
    public void doSomething() {
        System.out.println("[Lifecycle] Bean is working!");
    }
    
    // 10. PreDestroy 销毁前方法
    @PreDestroy
    public void preDestroy() {
        System.out.println("[Lifecycle] @PreDestroy called");
    }
    
    // 11. DisposableBean.destroy()
    @Override
    public void destroy() throws Exception {
        System.out.println("[Lifecycle] destroy() called");
    }
    
    // 12. 自定义 destroy-method
    public void customDestroy() {
        System.out.println("[Lifecycle] customDestroy() called");
    }
}
```

#### Bean 作用域

```java
package com.example.app.scope;

import org.springframework.context.annotation.Scope;
import org.springframework.context.annotation.ScopedProxyMode;
import org.springframework.stereotype.Component;
import org.springframework.web.context.WebApplicationContext;

import java.util.UUID;

/**
 * Spring Bean 作用域详解
 */
public class BeanScopeExamples {
    
    // 1. Singleton 作用域（默认）
    // 整个容器中只有一个实例
    @Component
    @Scope("singleton")
    public static class SingletonBean {
        private final String id = UUID.randomUUID().toString();
        
        public String getId() {
            return id;
        }
    }
    
    // 2. Prototype 作用域
    // 每次请求都创建新实例
    @Component
    @Scope("prototype")
    public static class PrototypeBean {
        private final String id = UUID.randomUUID().toString();
        
        public String getId() {
            return id;
        }
    }
    
    // 3. Request 作用域（Web 应用）
    // 每个 HTTP 请求创建一个实例
    @Component
    @Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public static class RequestBean {
        private final String id = UUID.randomUUID().toString();
        private final long createdAt = System.currentTimeMillis();
        
        public String getId() {
            return id;
        }
        
        public long getCreatedAt() {
            return createdAt;
        }
    }
    
    // 4. Session 作用域（Web 应用）
    // 每个 HTTP Session 创建一个实例
    @Component
    @Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public static class SessionBean {
        private final String id = UUID.randomUUID().toString();
        private String userName;
        
        public String getId() {
            return id;
        }
        
        public String getUserName() {
            return userName;
        }
        
        public void setUserName(String userName) {
            this.userName = userName;
        }
    }
    
    // 5. Application 作用域
    // 整个 Web 应用只有一个实例
    @Component
    @Scope(value = WebApplicationContext.SCOPE_APPLICATION, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public static class ApplicationBean {
        private final String id = UUID.randomUUID().toString();
        
        public String getId() {
            return id;
        }
    }
    
    // 6. WebSocket 作用域
    // WebSocket 生命周期内只有一个实例
    // @Component
    // @Scope(scopeName = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)
    // public static class WebSocketBean { }
}
```

#### 条件注解与配置

```java
package com.example.app.condition;

import org.springframework.boot.autoconfigure.condition.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;

@Configuration
public class ConditionalConfiguration {
    
    private final Environment environment;
    
    public ConditionalConfiguration(Environment environment) {
        this.environment = environment;
    }
    
    // 条件：根据类是否存在
    @Bean
    @ConditionalOnClass(name = "com.fasterxml.jackson.databind.ObjectMapper")
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
    
    // 条件：根据属性值
    @Bean
    @ConditionalOnProperty(
        name = "app.feature.enabled",
        havingValue = "true",
        matchIfMissing = false
    )
    public FeatureService featureService() {
        return new FeatureService();
    }
    
    // 条件：根据Bean是否存在
    @Bean
    @ConditionalOnBean(ObjectMapper.class)
    public JsonService jsonService(ObjectMapper objectMapper) {
        return new JsonService(objectMapper);
    }
    
    // 条件：缺少特定Bean时创建
    @Bean
    @ConditionalOnMissingBean(CacheService.class)
    public CacheService defaultCacheService() {
        return new DefaultCacheService();
    }
    
    // 条件：根据表达式
    @Bean
    @ConditionalOnExpression("${app.cache.enabled:true} and ${app.cache.local:true}")
    public LocalCacheService localCacheService() {
        return new LocalCacheService();
    }
    
    // 条件：根据 Resource 存在
    @Bean
    @ConditionalOnResource(resources = "classpath:init.sql")
    public DatabaseInitializer databaseInitializer() {
        return new DatabaseInitializer();
    }
    
    // 条件：Web 应用环境
    @Bean
    @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
    public ServletService servletService() {
        return new ServletService();
    }
    
    // 条件：不是 Web 应用环境
    @Bean
    @ConditionalOnNotWebApplication
    public ConsoleService consoleService() {
        return new ConsoleService();
    }
    
    // 条件：根据 Java 版本
    @Bean
    @ConditionalOnJava(JavaVersion.EIGHTEEN)
    public Java18Feature java18Feature() {
        return new Java18Feature();
    }
    
    // 条件：JNDI 存在
    @Bean
    @ConditionalOnJndi("java:comp/env/jdbc/DefaultDB")
    public JndiDataSource dataSource() {
        return new JndiDataSource();
    }
}
```

### AOP 面向切面编程

```java
package com.example.app.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.aspectj.lang.reflect.MethodSignature;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.core.annotation.Order;

import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * Spring AOP 完整示例
 */
@Aspect
@Component
@Order(1)
public class PerformanceMonitorAspect {
    
    private static final Logger log = LoggerFactory.getLogger(PerformanceMonitorAspect.class);
    
    // 使用 ThreadLocal 存储每个线程的执行时间
    private final ThreadLocal<Map<String, Long>> startTimes = ThreadLocal.withInitial(HashMap::new);
    
    // 切入点：所有 Controller 的所有公共方法
    @Pointcut("execution(* com.example.app.controller..*.*(..))")
    public void controllerMethods() {}
    
    // 切入点：所有 Service 的所有公共方法
    @Pointcut("execution(* com.example.app.service..*.*(..))")
    public void serviceMethods() {}
    
    // 切入点：所有 Repository 的所有方法
    @Pointcut("execution(* com.example.app.repository..*.*(..))")
    public void repositoryMethods() {}
    
    // 切入点：带有 @LogExecutionTime 注解的方法
    @Pointcut("@annotation(com.example.app.aop.LogExecutionTime)")
    public void loggedMethods() {}
    
    // 切入点：所有公共方法（排除 Getter/Setter）
    @Pointcut("execution(public * *(..)) && !execution(* get*(..)) && !execution(* set*(..))")
    public void publicMethods() {}
    
    // Around 通知：性能监控
    @Around("controllerMethods() || serviceMethods()")
    public Object monitorPerformance(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        String methodName = joinPoint.getSignature().toShortString();
        
        // 获取方法参数
        Object[] args = joinPoint.getArgs();
        log.debug("Method {} called with args: {}", methodName, Arrays.toString(args));
        
        try {
            Object result = joinPoint.proceed();
            
            long duration = System.currentTimeMillis() - startTime;
            
            // 记录慢查询（超过 500ms）
            if (duration > 500) {
                log.warn("Slow method detected: {} took {}ms", methodName, duration);
            } else {
                log.debug("Method {} completed in {}ms", methodName, duration);
            }
            
            return result;
            
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            log.error("Method {} failed after {}ms: {}", methodName, duration, e.getMessage());
            throw e;
        }
    }
    
    // Before 通知：方法入口日志
    @Before("controllerMethods()")
    public void logBefore(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        log.info("Entering {}.{}()", 
            signature.getDeclaringType().getSimpleName(),
            signature.getName());
    }
    
    // AfterReturning 通知：方法返回后处理
    @AfterReturning(pointcut = "controllerMethods()", returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        if (result != null) {
            log.debug("Method {}.{}() returned: {}", 
                signature.getDeclaringType().getSimpleName(),
                signature.getName(),
                result.getClass().getSimpleName());
        }
    }
    
    // AfterThrowing 通知：异常处理
    @AfterThrowing(pointcut = "serviceMethods()", throwing = "exception")
    public void logAfterThrowing(JoinPoint joinPoint, Throwable exception) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        log.error("Exception in {}.{}(): {}", 
            signature.getDeclaringType().getSimpleName(),
            signature.getName(),
            exception.getMessage(),
            exception);
    }
    
    // 自定义注解拦截
    @Around("@annotation(logExecutionTime)")
    public Object aroundWithAnnotation(ProceedingJoinPoint joinPoint, 
                                       LogExecutionTime logExecutionTime) throws Throwable {
        long startTime = System.nanoTime();
        
        try {
            Object result = joinPoint.proceed();
            long duration = System.nanoTime() - startTime;
            
            MethodSignature signature = (MethodSignature) joinPoint.getSignature();
            log.info("[{}] {}.{}() took {}ms", 
                logExecutionTime.value(),
                signature.getDeclaringType().getSimpleName(),
                signature.getName(),
                duration / 1_000_000);
            
            return result;
        } catch (Exception e) {
            long duration = System.nanoTime() - startTime;
            log.error("[{}] {}.{}() failed after {}ms", 
                logExecutionTime.value(),
                signature.getDeclaringType().getSimpleName(),
                signature.getName(),
                duration / 1_000_000,
                e);
            throw e;
        }
    }
}

// 自定义注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@interface LogExecutionTime {
    String value() default "execution";
}

// 使用示例
@Service
public class UserService {
    
    @LogExecutionTime("user-service")
    public User createUser(CreateUserRequest request) {
        // ...
        return user;
    }
}
```

### 事务管理深入

```java
package com.example.app.transaction;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataIntegrityViolationException;

import java.util.List;

/**
 * Spring 事务管理完整示例
 */
@Service
public class TransactionService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private NotificationService notificationService;
    
    // 1. REQUIRED 传播行为（默认）
    // 如果当前有事务，则加入；否则创建新事务
    @Transactional(propagation = Propagation.REQUIRED)
    public void requiredExample() {
        // 在当前事务中执行
    }
    
    // 2. REQUIRES_NEW 传播行为
    // 总是创建新事务，暂停当前事务
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void requiresNewExample() {
        // 在新事务中执行，不受父事务影响
    }
    
    // 3. NESTED 传播行为
    // 如果当前有事务，则在嵌套事务中执行；否则创建新事务
    @Transactional(propagation = Propagation.NESTED)
    public void nestedExample() {
        // 使用 Savepoint 实现嵌套事务
    }
    
    // 4. MANDATORY 传播行为
    // 要求当前必须有事务，否则抛出异常
    @Transactional(propagation = Propagation.MANDATORY)
    public void mandatoryExample() {
        // 必须有事务才能执行
    }
    
    // 5. NEVER 传播行为
    // 要求当前不能有事务，否则抛出异常
    @Transactional(propagation = Propagation.NEVER)
    public void neverExample() {
        // 必须在无事务环境中执行
    }
    
    // 完整业务事务示例
    @Transactional(rollbackFor = Exception.class)
    public OrderResult createOrderWithPayment(OrderRequest request) {
        // 1. 创建订单
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setTotalAmount(request.getTotalAmount());
        order.setItems(request.getItems());
        order.setStatus(OrderStatus.PENDING);
        order = orderRepository.save(order);
        
        try {
            // 2. 处理支付（可能在另一个服务中）
            PaymentResult paymentResult = processPayment(order);
            
            // 3. 更新订单状态
            if (paymentResult.isSuccess()) {
                order.setStatus(OrderStatus.PAID);
                order.setPaymentId(paymentResult.getPaymentId());
            } else {
                order.setStatus(OrderStatus.PAYMENT_FAILED);
                throw new PaymentFailedException(paymentResult.getErrorMessage());
            }
            orderRepository.save(order);
            
            // 4. 扣减库存
            for (OrderItem item : request.getItems()) {
                inventoryService.decreaseStock(item.getProductId(), item.getQuantity());
            }
            
            // 5. 发送通知（异步，不影响主事务）
            notificationService.sendOrderConfirmation(order.getId());
            
            return new OrderResult(order, paymentResult);
            
        } catch (PaymentFailedException e) {
            // 支付失败，订单已创建但标记为失败
            order.setStatus(OrderStatus.PAYMENT_FAILED);
            order.setFailureReason(e.getMessage());
            orderRepository.save(order);
            throw e;
        }
    }
    
    // 只读事务优化
    @Transactional(readOnly = true)
    public List<User> getActiveUsers() {
        return userRepository.findByActiveTrue();
    }
    
    // 超时控制
    @Transactional(timeout = 30)  // 30 秒超时
    public void longRunningOperation() {
        // 执行可能耗时的操作
    }
    
    // 隔离级别设置
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void readCommittedOperation() {
        // 设置事务隔离级别
    }
    
    // 编程式事务管理
    @Autowired
    private TransactionTemplate transactionTemplate;
    
    public void programmaticTransaction() {
        transactionTemplate.executeWithoutResult(status -> {
            try {
                // 执行操作
                userRepository.save(new User());
                orderRepository.save(new Order());
            } catch (Exception e) {
                status.setRollbackOnly();  // 标记回滚
                throw e;
            }
        });
    }
    
    // 事务监听器
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void handleOrderCreated(OrderCreatedEvent event) {
        // 事务提交后执行（如发送邮件通知）
    }
    
    @TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
    public void handleTransactionRolledBack(OrderCreatedEvent event) {
        // 事务回滚后执行（如清理缓存）
    }
}
```

### Spring Boot 配置属性绑定

```java
package com.example.app.config;

import org.springframework.boot.context.properties.*;
import org.springframework.stereotype.Component;
import org.springframework.validation.annotation.Validated;

import jakarta.validation.constraints.*;
import java.time.Duration;
import java.util.List;
import java.util.Map;

/**
 * 配置属性绑定示例
 */
@ConfigurationProperties(prefix = "app")
@Validated
@Component
public class AppProperties {
    
    // 简单属性
    private String name = "My Application";
    private int maxConnections = 100;
    private boolean enabled = true;
    
    // 嵌套配置
    private Server server = new Server();
    private Database database = new Database();
    private Cache cache = new Cache();
    private List<Feature> features;
    private Map<String, String> customSettings;
    
    // Getters and Setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public int getMaxConnections() { return maxConnections; }
    public void setMaxConnections(int maxConnections) { this.maxConnections = maxConnections; }
    
    public boolean isEnabled() { return enabled; }
    public void setEnabled(boolean enabled) { this.enabled = enabled; }
    
    public Server getServer() { return server; }
    public void setServer(Server server) { this.server = server; }
    
    public Database getDatabase() { return database; }
    public void setDatabase(Database database) { this.database = database; }
    
    public Cache getCache() { return cache; }
    public void setCache(Cache cache) { this.cache = cache; }
    
    public List<Feature> getFeatures() { return features; }
    public void setFeatures(List<Feature> features) { this.features = features; }
    
    public Map<String, String> getCustomSettings() { return customSettings; }
    public void setCustomSettings(Map<String, String> customSettings) { 
        this.customSettings = customSettings; 
    }
    
    // 嵌套类
    public static class Server {
        @NotBlank
        private String host = "localhost";
        
        @Min(1)
        @Max(65535)
        private int port = 8080;
        
        @DurationUnit(java.time.temporal.ChronoUnit.SECONDS)
        private Duration timeout = Duration.ofSeconds(30);
        
        private Ssl ssl = new Ssl();
        
        // Getters and Setters
        public String getHost() { return host; }
        public void setHost(String host) { this.host = host; }
        
        public int getPort() { return port; }
        public void setPort(int port) { this.port = port; }
        
        public Duration getTimeout() { return timeout; }
        public void setTimeout(Duration timeout) { this.timeout = timeout; }
        
        public Ssl getSsl() { return ssl; }
        public void setSsl(Ssl ssl) { this.ssl = ssl; }
        
        public static class Ssl {
            private boolean enabled = false;
            private String keyStore;
            private String keyStorePassword;
            private String keyStoreType = "JKS";
            
            public boolean isEnabled() { return enabled; }
            public void setEnabled(boolean enabled) { this.enabled = enabled; }
            
            public String getKeyStore() { return keyStore; }
            public void setKeyStore(String keyStore) { this.keyStore = keyStore; }
            
            public String getKeyStorePassword() { return keyStorePassword; }
            public void setKeyStorePassword(String keyStorePassword) { 
                this.keyStorePassword = keyStorePassword; 
            }
            
            public String getKeyStoreType() { return keyStoreType; }
            public void setKeyStoreType(String keyStoreType) { 
                this.keyStoreType = keyStoreType; 
            }
        }
    }
    
    public static class Database {
        @NotBlank
        private String url;
        
        private String username;
        private String password;
        
        @Min(1)
        private int maxPoolSize = 10;
        
        @Min(0)
        private int minIdle = 5;
        
        private Pool pool = new Pool();
        
        // Getters and Setters
        public String getUrl() { return url; }
        public void setUrl(String url) { this.url = url; }
        
        public String getUsername() { return username; }
        public void setUsername(String username) { this.username = username; }
        
        public String getPassword() { return password; }
        public void setPassword(String password) { this.password = password; }
        
        public int getMaxPoolSize() { return maxPoolSize; }
        public void setMaxPoolSize(int maxPoolSize) { this.maxPoolSize = maxPoolSize; }
        
        public int getMinIdle() { return minIdle; }
        public void setMinIdle(int minIdle) { this.minIdle = minIdle; }
        
        public Pool getPool() { return pool; }
        public void setPool(Pool pool) { this.pool = pool; }
        
        public static class Pool {
            private boolean autoCommit = true;
            private String connectionTestQuery = "SELECT 1";
            private Duration connectionTimeout = Duration.ofSeconds(30);
            private Duration idleTimeout = Duration.ofMinutes(10);
            private Duration maxLifetime = Duration.ofHours(2);
            
            // Getters and Setters
            public boolean isAutoCommit() { return autoCommit; }
            public void setAutoCommit(boolean autoCommit) { this.autoCommit = autoCommit; }
            
            public String getConnectionTestQuery() { return connectionTestQuery; }
            public void setConnectionTestQuery(String connectionTestQuery) { 
                this.connectionTestQuery = connectionTestQuery; 
            }
            
            public Duration getConnectionTimeout() { return connectionTimeout; }
            public void setConnectionTimeout(Duration connectionTimeout) { 
                this.connectionTimeout = connectionTimeout; 
            }
            
            public Duration getIdleTimeout() { return idleTimeout; }
            public void setIdleTimeout(Duration idleTimeout) { 
                this.idleTimeout = idleTimeout; 
            }
            
            public Duration getMaxLifetime() { return maxLifetime; }
            public void setMaxLifetime(Duration maxLifetime) { 
                this.maxLifetime = maxLifetime; 
            }
        }
    }
    
    public static class Cache {
        private boolean enabled = true;
        private int ttlSeconds = 3600;
        private int maxSize = 1000;
        private String backend = "redis";
        
        // Getters and Setters
        public boolean isEnabled() { return enabled; }
        public void setEnabled(boolean enabled) { this.enabled = enabled; }
        
        public int getTtlSeconds() { return ttlSeconds; }
        public void setTtlSeconds(int ttlSeconds) { this.ttlSeconds = ttlSeconds; }
        
        public int getMaxSize() { return maxSize; }
        public void setMaxSize(int maxSize) { this.maxSize = maxSize; }
        
        public String getBackend() { return backend; }
        public void setBackend(String backend) { this.backend = backend; }
    }
    
    public static class Feature {
        private String name;
        private boolean enabled;
        private Map<String, Object> config;
        
        // Getters and Setters
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        
        public boolean isEnabled() { return enabled; }
        public void setEnabled(boolean enabled) { this.enabled = enabled; }
        
        public Map<String, Object> getConfig() { return config; }
        public void setConfig(Map<String, Object> config) { this.config = config; }
    }
}
```

对应的 YAML 配置：

```yaml
# application.yml
app:
  name: Production App
  max-connections: 200
  enabled: true
  
  server:
    host: 0.0.0.0
    port: 8443
    timeout: 60s
    ssl:
      enabled: true
      key-store: classpath:keystore.p12
      key-store-password: ${KEYSTORE_PASSWORD}
      key-store-type: PKCS12
  
  database:
    url: jdbc:postgresql://localhost:5432/production
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    max-pool-size: 20
    min-idle: 10
    pool:
      auto-commit: false
      connection-timeout: 10s
      idle-timeout: 5m
      max-lifetime: 30m
  
  cache:
    enabled: true
    ttl-seconds: 7200
    max-size: 5000
    backend: redis
  
  features:
    - name: dark-mode
      enabled: true
      config:
        default-theme: dark
    - name: beta-features
      enabled: false
```

---

## 部署配置（Docker）

### 生产环境 Dockerfile

```dockerfile
# Multi-stage build Dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app

# Copy pom.xml first for dependency caching
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source code and build
COPY src ./src
RUN mvn clean package -DskipTests -B

# Stage 2: Runtime
FROM eclipse-temurin:17-jre-alpine

# Security: Create non-root user
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup

WORKDIR /app

# Copy built artifact
COPY --from=builder /app/target/*.jar app.jar

# Set ownership
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# Expose port
EXPOSE 8080

# JVM optimization for containers
ENV JAVA_OPTS="-XX:+UseContainerSupport \
               -XX:MaxRAMPercentage=75.0 \
               -XX:+UseG1GC \
               -XX:+ExitOnOutOfMemoryError \
               -Djava.security.egd=file:/dev/./urandom"

# Entry point
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

```yaml
# docker-compose.yml
version: "3.9"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: myapp:1.0.0
    container_name: myapp
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - JAVA_OPTS=-Xmx512m -Xms256m
      - DATABASE_HOST=postgres
      - DATABASE_PORT=5432
      - DATABASE_NAME=mydb
      - DATABASE_USERNAME=app
      - DATABASE_PASSWORD=${DB_PASSWORD}
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    env_file:
      - .env.production
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:8080/actuator/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 120s
    deploy:
      resources:
        limits:
          cpus: "2"
          memory: 1G
        reservations:
          cpus: "0.5"
          memory: 512M
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "5"
    networks:
      - backend

  postgres:
    image: postgres:16-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=app
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    command: >
      postgres
      -c max_connections=200
      -c shared_buffers=256MB
      -c effective_cache_size=512MB
      -c maintenance_work_mem=64MB
      -c checkpoint_completion_target=0.9
      -c wal_buffers=16MB
      -c default_statistics_target=100
      -c random_page_cost=1.1
      -c effective_io_concurrency=200
      -c work_mem=4MB
      -c min_wal_size=1GB
      -c max_wal_size=4GB
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: "1.5"
          memory: 1G
    networks:
      - backend

  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    command: >
      redis-server
      --maxmemory 512mb
      --maxmemory-policy allkeys-lru
      --appendonly yes
      --appendfsync everysec
      --save 900 1
      --save 300 10
      --save 60 10000
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
    networks:
      - backend

volumes:
  postgres_data:
  redis_data:

networks:
  backend:
    name: myapp-backend
```

### Kubernetes 部署

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - myapp
                topologyKey: kubernetes.io/hostname
      containers:
        - name: myapp
          image: myapp:1.0.0
          ports:
            - containerPort: 8080
              name: http
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "k8s"
            - name: DATABASE_HOST
              valueFrom:
                configMapKeyRef:
                  name: myapp-config
                  key: database.host
            - name: DATABASE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: database.password
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "1000m"
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 30
            timeoutSeconds: 10
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 10"]
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: ScheduleAnyway
          labelSelector:
            matchLabels:
              app: myapp

---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
spec:
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
```

---

## 常见陷阱与最佳实践

### 常见陷阱详解

#### 陷阱 1：循环依赖

```java
// ❌ 错误：循环依赖导致启动失败
@Service
public class AService {
    @Autowired
    private BService bService;
}

@Service
public class BService {
    @Autowired
    private AService aService;
}

// ✅ 正确：使用 @Lazy 延迟加载
@Service
public class AService {
    @Autowired
    @Lazy
    private BService bService;
}

// ✅ 正确：使用 Setter 注入
@Service
public class AService {
    private BService bService;
    
    @Autowired
    public void setBService(BService bService) {
        this.bService = bService;
    }
}

// ✅ 正确：使用 ApplicationContextAware
@Service
public class AService implements ApplicationContextAware {
    private ApplicationContext context;
    
    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        this.context = ctx;
    }
    
    public void doSomething() {
        BService bService = context.getBean(BService.class);
        bService.doSomethingElse();
    }
}
```

#### 陷阱 2：事务中的自调用

```java
// ❌ 错误：事务不生效（自调用）
@Service
public class UserService {
    
    @Transactional
    public void createUserAndSendEmail(User user) {
        createUser(user);  // 自调用，事务不生效
        sendEmail(user);   // 即使 createUser 失败，邮件仍会发送
    }
    
    @Transactional
    public void createUser(User user) {
        userRepository.save(user);
    }
}

// ✅ 正确：使用代理调用
@Service
public class UserService {
    
    @Transactional
    public void createUserAndSendEmail(User user) {
        createUserProxy(user);
        sendEmail(user);
    }
    
    @Transactional
    public void createUser(User user) {
        userRepository.save(user);
    }
    
    // 使用 self() 获取代理
    public void createUserProxy(User user) {
        ((UserService) AopContext.currentProxy()).createUser(user);
    }
}

// ✅ 正确：拆分到不同服务
@Service
public class UserManagementService {
    
    private final UserService userService;
    private final NotificationService notificationService;
    
    @Transactional
    public void createUserAndNotify(User user) {
        userService.createUser(user);
        notificationService.sendWelcomeEmail(user);
    }
}
```

#### 陷阱 3：N+1 查询问题

```java
// ❌ 错误：N+1 查询
@Service
public class UserService {
    
    public List<UserDTO> getAllUsersWithPosts() {
        List<User> users = userRepository.findAll();
        return users.stream()
            .map(user -> {
                List<Post> posts = postRepository.findByAuthorId(user.getId());
                return new UserDTO(user, posts);
            })
            .collect(Collectors.toList());
    }
}

// ✅ 正确：使用 JOIN FETCH
@Service
public class UserService {
    
    @Query("SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.posts")
    List<User> findAllWithPosts();
    
    public List<UserDTO> getAllUsersWithPosts() {
        List<User> users = userRepository.findAllWithPosts();
        return users.stream()
            .map(user -> new UserDTO(user, new ArrayList<>(user.getPosts())))
            .collect(Collectors.toList());
    }
}

// ✅ 正确：使用 EntityGraph
@EntityGraph(attributePaths = {"posts"})
@Query("SELECT u FROM User u")
List<User> findAllWithPosts();

// ✅ 正确：Batch Fetching
@BatchSize(size = 100)
@Entity
public class Post {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id")
    private User author;
}
```

### 最佳实践清单

#### 代码组织

1. **分层清晰**：Controller → Service → Repository
2. **单一职责**：每个类只做一件事
3. **依赖注入**：优先使用构造器注入
4. **接口编程**：面向接口而非实现
5. **不可变对象**：优先使用 final 和不可变类

#### 配置管理

1. **环境分离**：使用 Spring Profiles
2. **外部化配置**：使用 application.yml/yaml
3. **敏感信息**：使用环境变量或密钥管理服务
4. **配置验证**：使用 @Validated

#### 事务处理

1. **事务边界**：在 Service 层使用 @Transactional
2. **只读优化**：查询方法使用 readOnly = true
3. **异常处理**：配置正确的 rollbackFor
4. **避免长事务**：控制事务执行时间

#### 性能优化

1. **懒加载**：合理使用 @Lazy
2. **缓存**：使用 Spring Cache
3. **批量操作**：使用 batch insert/update
4. **索引**：为常用查询添加数据库索引
5. **分页**：始终使用分页查询

#### 安全性

1. **参数验证**：使用 @Valid 和 @Validated
2. **SQL 注入**：使用参数化查询
3. **XSS**：正确转义用户输入
4. **敏感数据**：加密存储敏感信息
5. **日志脱敏**：日志中不记录敏感数据

---

## 与同类框架对比

### Spring Boot vs Django vs FastAPI 详细对比

| 维度 | Spring Boot | Django | FastAPI |
|------|-------------|--------|---------|
| **语言** | Java/Kotlin | Python | Python |
| **类型系统** | 静态（编译期） | 动态（可选类型） | 类型注解（运行时） |
| **性能** | 高（运行时） | 中等 | 高 |
| **并发模型** | 线程池 | 同步/ASGI | 原生异步 |
| **ORM** | JPA/Hibernate | Django ORM | SQLAlchemy |
| **管理后台** | 需开发 | Django Admin | 需开发 |
| **认证** | Spring Security | Django Auth | 需自行实现 |
| **生态** | 极其庞大 | 成熟 | 成长中 |
| **学习曲线** | 高 | 中 | 低 |
| **启动时间** | 10-30秒 | 1-2秒 | <1秒 |
| **团队要求** | Java 专家 | Python 全栈 | Python 开发者 |
| **适用场景** | 金融、企业级 | 内容网站 | API、AI 应用 |

### Spring Boot vs Laravel (PHP)

| 维度 | Spring Boot | Laravel |
|------|-------------|---------|
| **语言** | Java | PHP |
| **生态系统** | 企业级完整 | Web 完整 | 
| **性能** | 高 | 中等 |
| **类型安全** | 强类型 | 弱类型 |
| **学习曲线** | 高 | 低 |
| **适用场景** | 企业级、长期项目 | 快速开发 |

### Spring Boot vs .NET Core

| 维度 | Spring Boot | .NET Core |
|------|-------------|-----------|
| **语言** | Java/Kotlin | C# |
| **跨平台** | 是 | 是 |
| **性能** | 高 | 高 |
| **生态系统** | Java 生态 | .NET 生态 |
| **学习曲线** | 高 | 中 |
| **IDE 支持** | IntelliJ/Eclipse | Visual Studio |

### 选型建议

```
项目选型决策树

├── 企业级核心系统
│   ├── 金融/银行系统 → Spring Boot（强类型、安全、高可靠性）
│   ├── 电商平台 → Spring Boot（高并发、事务可靠）
│   └── 企业内部系统 → Spring Boot/Django
│
├── 快速原型/MVP
│   ├── Python 团队 → FastAPI（开发速度快）
│   ├── PHP 背景 → Laravel
│   └── 需要管理后台 → Django
│
├── AI/ML 应用
│   ├── 模型推理服务 → FastAPI/Spring AI
│   ├── 数据处理 → Python
│   └── AI 应用后端 → Spring Boot + Spring AI
│
├── 微服务架构
│   ├── Java/Kotlin 团队 → Spring Boot + Spring Cloud
│   ├── .NET 团队 → .NET Core
│   └── 多语言 → 各选其长
│
└── 团队因素
    ├── Java 专家 → Spring Boot
    ├── Python 全栈 → Django/FastAPI
    ├── 快速迭代需求 → FastAPI/Django
    └── 长期维护项目 → Spring Boot
```

---

> [!SUCCESS]
> 本文档全面覆盖了 Spring Boot 3.x 的核心知识，包括自动配置原理、Spring Data JPA、Spring Security、WebFlux 响应式编程及 Spring AI 集成。Spring Boot 凭借其成熟的企业级生态、严格的类型安全和完整的微服务支持，在 2026 年仍然是大型企业项目的首选 Java 框架。
