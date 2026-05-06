---
name: springboot-security
description: Spring Security best practices for authn/authz, validation, CSRF, secrets, headers, rate limiting, and dependency security in Java Spring Boot services.
origin: ECC
---

# Spring Boot 安全审查

在添加认证、处理登录、创建 endpoint 或处理敏感信息时使用。

## 何时激活

- 添加身份验证 (JWT, OAuth2, session-based)
- 实现授权 (@PreAuthorize, role-based 访问)
- 验证用户输入 (Bean Validation, custom validator)
- 配置 CORS, CSRF 或 security headers
- 管理机密信息 (Vault, 环境变量)
- 添加 rate limiting 或 brute-force 保护
- 扫描依赖项的 CVE

## 身份认证

- 优先使用带有撤销列表的 stateless JWT 或 opaque token
- Session 使用 `httpOnly`, `Secure`, `SameSite=Strict` cookies
- 通过 `OncePerRequestFilter` 或 resource server 验证 token

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
  private final JwtService jwtService;

  public JwtAuthFilter(JwtService jwtService) {
    this.jwtService = jwtService;
  }

  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain chain) throws ServletException, IOException {
    String header = request.getHeader(HttpHeaders.AUTHORIZATION);
    if (header != null && header.startsWith("Bearer ")) {
      String token = header.substring(7);
      Authentication auth = jwtService.authenticate(token);
      SecurityContextHolder.getContext().setAuthentication(auth);
    }
    chain.doFilter(request, response);
  }
}
```

## 授权

- 启用方法级安全: `@EnableMethodSecurity`
- 使用 `@PreAuthorize("hasRole('ADMIN')")` 或 `@PreAuthorize("@authz.canEdit(#id)")`
- 默认拒绝；仅暴露必要的 scope

```java
@RestController
@RequestMapping("/api/admin")
public class AdminController {

  @PreAuthorize("hasRole('ADMIN')")
  @GetMapping("/users")
  public List<UserDto> listUsers() {
    return userService.findAll();
  }

  @PreAuthorize("@authz.isOwner(#id, authentication)")
  @DeleteMapping("/users/{id}")
  public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    userService.delete(id);
    return ResponseEntity.noContent().build();
  }
}
```

## 输入验证

- 在 Controller 中使用 `@Valid` 进行 Bean Validation
- 在 DTO 上应用约束: `@NotBlank`, `@Email`, `@Size`, custom validator
- 渲染前使用白名单清理任何 HTML

```java
// 错误: 没有 Validation
@PostMapping("/users")
public User createUser(@RequestBody UserDto dto) {
  return userService.create(dto);
}

// 正确: 经过验证的 DTO
public record CreateUserDto(
    @NotBlank @Size(max = 100) String name,
    @NotBlank @Email String email,
    @NotNull @Min(0) @Max(150) Integer age
) {}

@PostMapping("/users")
public ResponseEntity<UserDto> createUser(@Valid @RequestBody CreateUserDto dto) {
  return ResponseEntity.status(HttpStatus.CREATED)
      .body(userService.create(dto));
}
```

## SQL 注入防护

- 使用 Spring Data repositories 或参数化查询
- 原生查询使用 `:param` binding；永远不要拼接字符串

```java
// 错误: 原生查询中的字符串拼接
@Query(value = "SELECT * FROM users WHERE name = '" + name + "'", nativeQuery = true)

// 正确: 参数化原生查询
@Query(value = "SELECT * FROM users WHERE name = :name", nativeQuery = true)
List<User> findByName(@Param("name") String name);

// 正确: Spring Data 派生查询 (自动参数化)
List<User> findByEmailAndActiveTrue(String email);
```

## 密码编码

- 始终使用 BCrypt 或 Argon2 对密码进行 hash — 永远不要明文存储
- 使用 `PasswordEncoder` bean 而不是手动 hash

```java
@Bean
public PasswordEncoder passwordEncoder() {
  return new BCryptPasswordEncoder(12); // cost factor 12
}

// 在 Service 中
public User register(CreateUserDto dto) {
  String hashedPassword = passwordEncoder.encode(dto.password());
  return userRepository.save(new User(dto.email(), hashedPassword));
}
```

## CSRF 保护

- 对于浏览器 session 应用保持 CSRF 启用；在表单/headers 中添加 token
- 对于使用 Bearer token 的纯 API 禁用 CSRF，依赖 stateless auth

```java
http
  .csrf(csrf -> csrf.disable())
  .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
```

## 机密信息管理

- 源代码中不存储机密；从 env 或 vault 加载
- 保持 `application.yml` 不含凭据；使用占位符
- 定期轮换 token 和数据库凭据

```yaml
# 错误: 在 application.yml 中硬编码
spring:
  datasource:
    password: mySecretPassword123

# 正确: 环境变量占位符
spring:
  datasource:
    password: ${DB_PASSWORD}

# 正确: Spring Cloud Vault 集成
spring:
  cloud:
    vault:
      uri: https://vault.example.com
      token: ${VAULT_TOKEN}
```

## 安全 Headers

```java
http
  .headers(headers -> headers
    .contentSecurityPolicy(csp -> csp
      .policyDirectives("default-src 'self'"))
    .frameOptions(HeadersConfigurer.FrameOptionsConfig::sameOrigin)
    .xssProtection(Customizer.withDefaults())
    .referrerPolicy(rp -> rp.policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.NO_REFERRER)));
```

## CORS 配置

- 在安全过滤器级别配置 CORS，而不是每个 controller
- 限制允许的 origins — 生产环境永远不要使用 `*`

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
  CorsConfiguration config = new CorsConfiguration();
  config.setAllowedOrigins(List.of("https://app.example.com"));
  config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
  config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
  config.setAllowCredentials(true);
  config.setMaxAge(3600L);

  UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
  source.registerCorsConfiguration("/api/**", config);
  return source;
}

// 在 SecurityFilterChain 中:
http.cors(cors -> cors.configurationSource(corsConfigurationSource()));
```

## Rate Limiting

- 在昂贵的 endpoint 上应用 Bucket4j 或 gateway 级别的限制
- 爆发时记录日志并告警；返回带重试提示的 429

```java
// 使用 Bucket4j 实现每个 endpoint 的 rate limiting
@Component
public class RateLimitFilter extends OncePerRequestFilter {
  private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();

  private Bucket createBucket() {
    return Bucket.builder()
        .addLimit(Bandwidth.classic(100, Refill.intervally(100, Duration.ofMinutes(1))))
        .build();
  }

  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain chain) throws ServletException, IOException {
    String clientIp = request.getRemoteAddr();
    Bucket bucket = buckets.computeIfAbsent(clientIp, k -> createBucket());

    if (bucket.tryConsume(1)) {
      chain.doFilter(request, response);
    } else {
      response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
      response.getWriter().write("{\"error\": \"Rate limit exceeded\"}");
    }
  }
}
```

## 依赖安全

- 在 CI 中运行 OWASP Dependency Check / Snyk
- 保持 Spring Boot 和 Spring Security 在支持的版本
- 在已知 CVE 时使构建失败

## 日志和 PII

- 永远不要记录机密、token、密码或完整的 PAN 数据
- 编辑敏感字段；使用结构化 JSON 日志

## 文件上传

- 验证大小、content type 和扩展名
- 存储在 web root 之外；必要时进行扫描

## 发布前检查清单

- [ ] Auth token 已正确验证且过期时间设置正确
- [ ] 每个敏感路径都有授权保护
- [ ] 所有输入都已验证和清理
- [ ] 没有字符串拼接的 SQL
- [ ] 应用类型使用正确的 CSRF 姿势
- [ ] 机密信息外部存储；没有提交到代码库
- [ ] 安全 headers 已配置
- [ ] API 有 rate limiting
- [ ] 依赖已扫描且为最新版本
- [ ] 日志不含敏感数据

**记住**: 默认拒绝、验证输入、最小权限和优先配置安全。