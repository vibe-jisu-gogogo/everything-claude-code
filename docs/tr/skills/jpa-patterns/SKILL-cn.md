---
name: jpa-patterns
description: Spring Boot 中用于 entity 设计、关系、查询优化、transaction、auditing、索引、分页和连接池的 JPA/Hibernate 模式。
origin: ECC
---

# JPA/Hibernate 模式

用于 Spring Boot 中的数据建模、repository 和性能调优。

## 何时激活

- 设计 JPA entity 和表映射时
- 定义关系时 (@OneToMany, @ManyToOne, @ManyToMany)
- 优化查询时 (防止 N+1, fetch 策略, projections)
- 配置 transaction、auditing 或 soft delete 时
- 设置分页、排序或自定义 repository 方法时
- 配置连接池 (HikariCP) 或二级缓存时

## Entity 设计

```java
@Entity
@Table(name = "markets", indexes = {
  @Index(name = "idx_markets_slug", columnList = "slug", unique = true)
})
@EntityListeners(AuditingEntityListener.class)
public class MarketEntity {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(nullable = false, length = 200)
  private String name;

  @Column(nullable = false, unique = true, length = 120)
  private String slug;

  @Enumerated(EnumType.STRING)
  private MarketStatus status = MarketStatus.ACTIVE;

  @CreatedDate private Instant createdAt;
  @LastModifiedDate private Instant updatedAt;
}
```

启用 Auditing:
```java
@Configuration
@EnableJpaAuditing
class JpaConfig {}
```

## 关系和 N+1 预防

```java
@OneToMany(mappedBy = "market", cascade = CascadeType.ALL, orphanRemoval = true)
private List<PositionEntity> positions = new ArrayList<>();
```

- 默认 lazy loading；需要时在查询中使用 `JOIN FETCH`
- 避免在集合上使用 `EAGER`；对于只读路径使用 DTO projections

```java
@Query("select m from MarketEntity m left join fetch m.positions where m.id = :id")
Optional<MarketEntity> findWithPositions(@Param("id") Long id);
```

## Repository 模式

```java
public interface MarketRepository extends JpaRepository<MarketEntity, Long> {
  Optional<MarketEntity> findBySlug(String slug);

  @Query("select m from MarketEntity m where m.status = :status")
  Page<MarketEntity> findByStatus(@Param("status") MarketStatus status, Pageable pageable);
}
```

- 对于轻量级查询使用 projections:
```java
public interface MarketSummary {
  Long getId();
  String getName();
  MarketStatus getStatus();
}
Page<MarketSummary> findAllBy(Pageable pageable);
```

## Transaction

- 用 `@Transactional` 标记服务方法
- 使用 `@Transactional(readOnly = true)` 优化只读路径
- 谨慎选择 propagation；避免长时间运行的 transaction

```java
@Transactional
public Market updateStatus(Long id, MarketStatus status) {
  MarketEntity entity = repo.findById(id)
      .orElseThrow(() -> new EntityNotFoundException("Market"));
  entity.setStatus(status);
  return Market.from(entity);
}
```

## 分页

```java
PageRequest page = PageRequest.of(pageNumber, pageSize, Sort.by("createdAt").descending());
Page<MarketEntity> markets = repo.findByStatus(MarketStatus.ACTIVE, page);
```

对于类似 cursor 的分页，在 JPQL 中添加 `id > :lastId` 并配合排序。

## 索引和性能

- 为常用过滤器添加索引 (`status`, `slug`, 外键)
- 使用匹配查询模式的复合索引 (`status, created_at`)
- 避免使用 `select *`；只投影必要的列
- 使用 `saveAll` 和 `hibernate.jdbc.batch_size` 批量写入

## 连接池 (HikariCP)

推荐配置:
```
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.validation-timeout=5000
```

对于 PostgreSQL LOB 处理添加:
```
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
```

## 缓存

- 一级缓存按 EntityManager 划分；避免在 transaction 之间保留 entity
- 对于读密集的 entity，谨慎考虑二级缓存；验证 eviction 策略

## 迁移

- 使用 Flyway 或 Liquibase；永远不要在生产中依赖 Hibernate 自动 DDL
- 保持迁移幂等且仅做增量操作；避免在没有计划的情况下删除列

## 数据访问测试

- 优先使用带有 Testcontainers 的 `@DataJpaTest` 来反映生产环境
- 使用日志验证 SQL 效率：设置 `logging.level.org.hibernate.SQL=DEBUG` 和 `logging.level.org.hibernate.orm.jdbc.bind=TRACE` 以查看参数值

**记住**: 保持 entity 精简，查询有目的性，transaction 简短。通过 fetch 策略和 projections 防止 N+1，并为读写路径建立索引。
