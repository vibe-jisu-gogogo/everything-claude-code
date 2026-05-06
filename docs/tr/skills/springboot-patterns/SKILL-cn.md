---
name: springboot-patterns
description: Spring Boot architecture patterns, REST API design, layered services, data access, caching, async processing, and logging. Use for Java Spring Boot backend work.
origin: ECC
---

# Spring Boot 开发模式

用于可扩展、生产级别服务的 Spring Boot 架构和 API 模式。

## 何时激活

- 使用 Spring MVC 或 WebFlux 创建 REST API
- 配置 Controller → service → repository 层级结构
- 配置 Spring Data JPA、caching 或 async processing
- 添加 Validation、exception handling 或分页
- 为开发/测试/生产环境设置 profiles
- 使用 Spring Events 或 Kafka 实现 event-driven 模式

## REST API 结构

```java
@RestController
@RequestMapping("/api/markets")
@Validated
class MarketController {
  private final MarketService marketService;

  MarketController(MarketService marketService) {
    this.marketService = marketService;
  }

  @GetMapping
  ResponseEntity<Page<MarketResponse>> list(
      @RequestParam(defaultValue = "0") int page,
      @RequestParam(defaultValue = "20") int size) {
    Page<Market> markets = marketService.list(PageRequest.of(page, size));
    return ResponseEntity.ok(markets.map(MarketResponse::from));
  }

  @PostMapping
  ResponseEntity<MarketResponse> create(@Valid @RequestBody CreateMarketRequest request) {
    Market market = marketService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(MarketResponse::from(market));
  }
}
```

## Repository 模式 (Spring Data JPA)

```java
public interface MarketRepository extends JpaRepository<MarketEntity, Long> {
  @Query("select m from MarketEntity m where m.status = :status order by m.volume desc")
  List<MarketEntity> findActive(@Param("status") MarketStatus status, Pageable pageable);
}
```

## 带 Transaction 的 Service 层

```java
@Service
public class MarketService {
  private final MarketRepository repo;

  public MarketService(MarketRepository repo) {
    this.repo = repo;
  }

  @Transactional
  public Market create(CreateMarketRequest request) {
    MarketEntity entity = MarketEntity.from(request);
    MarketEntity saved = repo.save(entity);
    return Market.from(saved);
  }
}
```

## DTO 与 Validation

```java
public record CreateMarketRequest(
    @NotBlank @Size(max = 200) String name,
    @NotBlank @Size(max = 2000) String description,
    @NotNull @FutureOrPresent Instant endDate,
    @NotEmpty List<@NotBlank String> categories) {}

public record MarketResponse(Long id, String name, MarketStatus status) {
  static MarketResponse from(Market market) {
    return new MarketResponse(market.id(), market.name(), market.status());
  }
}
```

## Exception Handling

```java
@ControllerAdvice
class GlobalExceptionHandler {
  @ExceptionHandler(MethodArgumentNotValidException.class)
  ResponseEntity<ApiError> handleValidation(MethodArgumentNotValidException ex) {
    String message = ex.getBindingResult().getFieldErrors().stream()
        .map(e -> e.getField() + ": " + e.getDefaultMessage())
        .collect(Collectors.joining(", "));
    return ResponseEntity.badRequest().body(ApiError.validation(message));
  }

  @ExceptionHandler(AccessDeniedException.class)
  ResponseEntity<ApiError> handleAccessDenied() {
    return ResponseEntity.status(HttpStatus.FORBIDDEN).body(ApiError.of("Forbidden"));
  }

  @ExceptionHandler(Exception.class)
  ResponseEntity<ApiError> handleGeneric(Exception ex) {
    // 使用 stack trace 记录意外错误
    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body(ApiError.of("Internal server error"));
  }
}
```

## Caching

需要在 configuration 类中启用 `@EnableCaching`。

```java
@Service
public class MarketCacheService {
  private final MarketRepository repo;

  public MarketCacheService(MarketRepository repo) {
    this.repo = repo;
  }

  @Cacheable(value = "market", key = "#id")
  public Market getById(Long id) {
    return repo.findById(id)
        .map(Market::from)
        .orElseThrow(() -> new EntityNotFoundException("Market not found"));
  }

  @CacheEvict(value = "market", key = "#id")
  public void evict(Long id) {}
}
```

## Async Processing

需要在 configuration 类中启用 `@EnableAsync`。

```java
@Service
public class NotificationService {
  @Async
  public CompletableFuture<Void> sendAsync(Notification notification) {
    // 发送 email/SMS
    return CompletableFuture.completedFuture(null);
  }
}
```

## 日志记录 (SLF4J)

```java
@Service
public class ReportService {
  private static final Logger log = LoggerFactory.getLogger(ReportService.class);

  public Report generate(Long marketId) {
    log.info("generate_report marketId={}", marketId);
    try {
      // 业务逻辑
    } catch (Exception ex) {
      log.error("generate_report_failed marketId={}", marketId, ex);
      throw ex;
    }
    return new Report();
  }
}
```

## Middleware / Filter

```java
@Component
public class RequestLoggingFilter extends OncePerRequestFilter {
  private static final Logger log = LoggerFactory.getLogger(RequestLoggingFilter.class);

  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain filterChain) throws ServletException, IOException {
    long start = System.currentTimeMillis();
    try {
      filterChain.doFilter(request, response);
    } finally {
      long duration = System.currentTimeMillis() - start;
      log.info("req method={} uri={} status={} durationMs={}",
          request.getMethod(), request.getRequestURI(), response.getStatus(), duration);
    }
  }
}
```

## 分页与排序

```java
PageRequest page = PageRequest.of(pageNumber, pageSize, Sort.by("createdAt").descending());
Page<Market> results = marketService.list(page);
```

## 容错的外部调用

```java
public <T> T withRetry(Supplier<T> supplier, int maxRetries) {
  int attempts = 0;
  while (true) {
    try {
      return supplier.get();
    } catch (Exception ex) {
      attempts++;
      if (attempts >= maxRetries) {
        throw ex;
      }
      try {
        Thread.sleep((long) Math.pow(2, attempts) * 100L);
      } catch (InterruptedException ie) {
        Thread.currentThread().interrupt();
        throw ex;
      }
    }
  }
}
```

## Rate Limiting (Filter + Bucket4j)

**安全注意**：`X-Forwarded-For` 头默认是不可信的，因为客户端可以伪造它。
仅在以下情况下使用 Forwarded 头：
1. 您的应用程序在可信的 reverse proxy 之后（nginx、AWS ALB 等）
2. 您已将 `ForwardedHeaderFilter` 注册为 bean
3. 在 application properties 中配置了 `server.forward-headers-strategy=NATIVE` 或 `FRAMEWORK`
4. 您的 proxy 配置为覆盖（而不是追加）`X-Forwarded-For` 头

当 `ForwardedHeaderFilter` 正确配置后，`request.getRemoteAddr()` 会自动
从 forwarded 头中返回正确的客户端 IP。没有此配置时，直接使用
`request.getRemoteAddr()` — 它返回即时连接 IP，这是唯一可信的值。

```java
@Component
public class RateLimitFilter extends OncePerRequestFilter {
  private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();

  /*
   * 安全：此过滤器使用 request.getRemoteAddr() 识别客户端以进行 rate limiting。
   *
   * 如果您的应用程序在 reverse proxy 之后（nginx、AWS ALB 等），您必须配置 Spring
   * 以正确处理 forwarded 头以获得准确的客户端 IP 检测：
   *
   * 1. 在 application.properties/yaml 中设置 server.forward-headers-strategy=NATIVE（适用于云平台）
   *    或 FRAMEWORK
   * 2. 如果使用 FRAMEWORK 策略，注册 ForwardedHeaderFilter：
   *
   *    @Bean
   *    ForwardedHeaderFilter forwardedHeaderFilter() {
   *        return new ForwardedHeaderFilter();
   *    }
   *
   * 3. 确保您的 proxy 配置为覆盖 X-Forwarded-For 头以防止欺骗（而不是追加）
   * 4. 为您的容器配置 server.tomcat.remoteip.trusted-proxies 或等效项
   *
   * 没有此配置，request.getRemoteAddr() 将返回 proxy IP 而不是客户端 IP。
   * 不要直接读取 X-Forwarded-For — 在没有可靠 proxy 处理的情况下很容易被伪造。
   */
  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain filterChain) throws ServletException, IOException {
    // 使用 getRemoteAddr() — 在配置 ForwardedHeaderFilter 时返回正确的客户端 IP，
    // 否则返回直接连接 IP。在没有正确 proxy 配置的情况下，不要信任
    // X-Forwarded-For 头。
    String clientIp = request.getRemoteAddr();

    Bucket bucket = buckets.computeIfAbsent(clientIp,
        k -> Bucket.builder()
            .addLimit(Bandwidth.classic(100, Refill.greedy(100, Duration.ofMinutes(1))))
            .build());

    if (bucket.tryConsume(1)) {
      filterChain.doFilter(request, response);
    } else {
      response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
    }
  }
}
```

## 后台 Job

使用 Spring 的 `@Scheduled` 或与队列集成（如 Kafka、SQS、RabbitMQ）。
保持 handler 为幂等且可观察。

## 可观测性

- 用 Logback encoder 配置的结构化日志（JSON）
- Metrics：Micrometer + Prometheus/OTel
- Tracing：OpenTelemetry 或带有 Brave backend 的 Micrometer Tracing

## Production 默认值

- 优先使用 Constructor injection，避免 field injection
- 启用 `spring.mvc.problemdetails.enabled=true` 以支持 RFC 7807 错误（Spring Boot 3+）
- 为工作负载配置 HikariCP 池大小，设置超时
- 对于查询使用 `@Transactional(readOnly = true)`
- 使用 `@NonNull` 和适当位置的 `Optional` 来强制 null-safety

**记住**：保持 Controller 精简、service 专注、repository 简单，以及错误处理集中化。
为可维护性和可测试性进行优化。
