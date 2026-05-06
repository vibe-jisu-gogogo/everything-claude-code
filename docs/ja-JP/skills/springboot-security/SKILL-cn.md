---
name: springboot-security
description: Spring Security best practices for authn/authz, validation, CSRF, secrets, headers, rate limiting, and dependency security in Java Spring Boot services.
---

# Spring Boot 安全审查

在添加认证、处理输入、创建端点或处理密钥时使用。

## 认证

- 优先使用无状态JWT或带有撤销列表的不透明令牌
- 会话使用 `httpOnly`、`Secure`、`SameSite=Strict` Cookie
- 使用 `OncePerRequestFilter` 或资源服务器验证令牌

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

- 启用方法安全: `@EnableMethodSecurity`
- 使用 `@PreAuthorize("hasRole('ADMIN')")` 或 `@PreAuthorize("@authz.canEdit(#id)")`
- 默认拒绝，仅公开必要范围

## 输入验证

- 在控制器中使用 `@Valid` 进行Bean Validation
- 对DTO应用约束: `@NotBlank`、`@Email`、`@Size`、自定义验证器
- 渲染前使用白名单对HTML进行清理

## SQL注入防护

- 使用Spring Data仓库或参数化查询
- 原生查询使用 `:param` 绑定，不要拼接字符串

## CSRF防护

- 浏览器会话应用启用CSRF，在表单/头中包含令牌
- 纯API使用Bearer令牌时禁用CSRF，依赖无状态认证

```java
http
  .csrf(csrf -> csrf.disable())
  .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
```

## 密钥管理

- 源代码不要包含密钥。从环境变量或vault读取
- 从 `application.yml` 中移除凭证，使用占位符
- 定期轮换令牌和数据库凭证

## 安全头

```java
http
  .headers(headers -> headers
    .contentSecurityPolicy(csp -> csp
      .policyDirectives("default-src 'self'"))
    .frameOptions(HeadersConfigurer.FrameOptionsConfig::sameOrigin)
    .xssProtection(Customizer.withDefaults())
    .referrerPolicy(rp -> rp.policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.NO_REFERRER)));
```

## 速率限制

- 对高成本端点应用Bucket4j或网关级别的限制
- 记录突发流量并发送警报，返回带重试提示的429

## 依赖安全

- 在CI中运行OWASP Dependency Check / Snyk
- 保持Spring Boot和Spring Security在受支持版本
- 发现已知CVE时使构建失败

## 日志和PII

- 不要记录密钥、令牌、密码、完整PAN数据
- 编辑敏感字段，使用结构化JSON日志

## 文件上传

- 验证大小、内容类型、扩展名
- 保存在Web根目录之外，必要时进行扫描

## 发布前检查清单

- [ ] 认证令牌验证正确并已过期
- [ ] 所有敏感路径都有授权防护
- [ ] 所有输入都经过验证和清理
- [ ] 没有字符串拼接的SQL
- [ ] 针对应用类型的CSRF措施正确
- [ ] 密钥已外部化，未提交到代码库
- [ ] 安全头已配置
- [ ] API有限速措施
- [ ] 依赖已扫描并保持最新
- [ ] 日志中没有敏感数据

**注意**: 默认拒绝、验证输入、应用最小权限、优先通过配置实现安全。
