| name | description |
|------|-------------|
| cloud-infrastructure-security | 在部署到云平台、配置基础设施、管理IAM策略、设置日志记录/监控、实现CI/CD流水线时使用此技能。提供符合最佳实践的云安全检查清单。 |

# 云基础设施安全技能

此技能确保云基础设施、CI/CD流水线、部署配置遵循安全最佳实践并符合行业标准。

## 何时启用

- 将应用部署到云平台（AWS、Vercel、Railway、Cloudflare）
- 设置IAM角色和权限
- 配置CI/CD流水线
- 将基础设施实现为代码（Terraform、CloudFormation）
- 设置日志记录和监控
- 云环境中的机密管理
- 配置CDN和边缘安全
- 实施灾难恢复和备份策略

## 云安全检查清单

### 1. IAM和访问控制

#### 最小权限原则

```yaml
# PASS: 正确：最小权限
iam_role:
  permissions:
    - s3:GetObject  # 仅读取访问
    - s3:ListBucket
  resources:
    - arn:aws:s3:::my-bucket/*  # 仅特定存储桶

# FAIL: 错误：过于广泛的权限
iam_role:
  permissions:
    - s3:*  # 所有S3操作
  resources:
    - "*"  # 所有资源
```

#### 多因素认证（MFA）

```bash
# 始终在root/admin账户启用MFA
aws iam enable-mfa-device \
  --user-name admin \
  --serial-number arn:aws:iam::123456789:mfa/admin \
  --authentication-code1 123456 \
  --authentication-code2 789012
```

#### 验证步骤

- [ ] 不在生产环境使用root账户
- [ ] 在所有特权账户启用MFA
- [ ] 服务账户使用角色而非长期凭证
- [ ] IAM策略遵循最小权限
- [ ] 定期进行访问审查
- [ ] 轮换或删除未使用的凭证

### 2. 机密管理

#### 云机密管理器

```typescript
// PASS: 正确：使用云机密管理器
import { SecretsManager } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManager({ region: 'us-east-1' });
const secret = await client.getSecretValue({ SecretId: 'prod/api-key' });
const apiKey = JSON.parse(secret.SecretString).key;

// FAIL: 错误：硬编码或仅使用环境变量
const apiKey = process.env.API_KEY; // 不轮换、不审计
```

#### 机密轮换

```bash
# 设置数据库凭证自动轮换
aws secretsmanager rotate-secret \
  --secret-id prod/db-password \
  --rotation-lambda-arn arn:aws:lambda:region:account:function:rotate \
  --rotation-rules AutomaticallyAfterDays=30
```

#### 验证步骤

- [ ] 将所有机密保存在云机密管理器（AWS Secrets Manager、Vercel Secrets）
- [ ] 启用数据库凭证自动轮换
- [ ] 至少每季度轮换API密钥
- [ ] 代码、日志、错误消息中无机密
- [ ] 启用机密访问的审计日志

### 3. 网络安全

#### VPC和防火墙设置

```terraform
# PASS: 正确：受限安全组
resource "aws_security_group" "app" {
  name = "app-sg"

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]  # 仅内部VPC
  }

  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # 仅HTTPS出站
  }
}

# FAIL: 错误：暴露到互联网
resource "aws_security_group" "bad" {
  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # 所有端口，所有IP！
  }
}
```

#### 验证步骤

- [ ] 数据库不可公开访问
- [ ] SSH/RDP端口仅限VPN/bastion
- [ ] 安全组遵循最小权限
- [ ] 配置网络ACL
- [ ] 启用VPC流日志

### 4. 日志记录和监控

#### CloudWatch/日志设置

```typescript
// PASS: 正确：全面的日志记录
import { CloudWatchLogsClient, CreateLogStreamCommand } from '@aws-sdk/client-cloudwatch-logs';

const logSecurityEvent = async (event: SecurityEvent) => {
  await cloudwatch.putLogEvents({
    logGroupName: '/aws/security/events',
    logStreamName: 'authentication',
    logEvents: [{
      timestamp: Date.now(),
      message: JSON.stringify({
        type: event.type,
        userId: event.userId,
        ip: event.ip,
        result: event.result,
        // 不记录敏感数据
      })
    }]
  });
};
```

#### 验证步骤

- [ ] 在所有服务启用CloudWatch/日志记录
- [ ] 记录失败的认证尝试
- [ ] 审计管理员操作
- [ ] 设置日志保留（合规性要求90天以上）
- [ ] 设置可疑活动警报
- [ ] 集中日志并防篡改

### 5. CI/CD流水线安全

#### 安全的流水线设置

```yaml
# PASS: 正确：安全的GitHub Actions工作流
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read  # 最小权限

    steps:
      - uses: actions/checkout@v4

      # 扫描机密
      - name: Secret scanning
        uses: trufflesecurity/trufflehog@main

      # 依赖审计
      - name: Audit dependencies
        run: npm audit --audit-level=high

      # 使用OIDC而非长期令牌
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/GitHubActionsRole
          aws-region: us-east-1
```

#### 供应链安全

```json
// package.json - 使用锁文件和完整性检查
{
  "scripts": {
    "install": "npm ci",  // 使用ci进行可重现构建
    "audit": "npm audit --audit-level=moderate",
    "check": "npm outdated"
  }
}
```

#### 验证步骤

- [ ] 使用OIDC而非长期凭证
- [ ] 在流水线中进行机密扫描
- [ ] 依赖漏洞扫描
- [ ] 容器镜像扫描（如适用）
- [ ] 强制执行分支保护规则
- [ ] 合并前需要代码审查
- [ ] 强制签名提交

### 6. Cloudflare和CDN安全

#### Cloudflare安全设置

```typescript
// PASS: 正确：带安全头的Cloudflare Workers
export default {
  async fetch(request: Request): Promise<Response> {
    const response = await fetch(request);

    // 添加安全头
    const headers = new Headers(response.headers);
    headers.set('X-Frame-Options', 'DENY');
    headers.set('X-Content-Type-Options', 'nosniff');
    headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
    headers.set('Permissions-Policy', 'geolocation=(), microphone=()');

    return new Response(response.body, {
      status: response.status,
      headers
    });
  }
};
```

#### WAF规则

```bash
# 启用Cloudflare WAF管理规则
# - OWASP Core Ruleset
# - Cloudflare Managed Ruleset
# - 速率限制规则
# - 机器人保护
```

#### 验证步骤

- [ ] 启用带OWASP规则的WAF
- [ ] 设置速率限制
- [ ] 启用机器人保护
- [ ] 启用DDoS保护
- [ ] 设置安全头
- [ ] 启用SSL/TLS严格模式

### 7. 备份和灾难恢复

#### 自动备份

```terraform
# PASS: 正确：自动RDS备份
resource "aws_db_instance" "main" {
  allocated_storage     = 20
  engine               = "postgres"

  backup_retention_period = 30  # 保留30天
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"

  enabled_cloudwatch_logs_exports = ["postgresql"]

  deletion_protection = true  # 防止意外删除
}
```

#### 验证步骤

- [ ] 设置自动每日备份
- [ ] 备份保留满足合规要求
- [ ] 启用时间点恢复
- [ ] 每季度进行备份测试
- [ ] 记录灾难恢复计划
- [ ] 定义并测试RPO和RTO

## 部署前云安全检查清单

所有生产云部署前：

- [ ] **IAM**：不使用root账户、启用MFA、最小权限策略
- [ ] **机密**：将所有机密放入带轮换的云机密管理器
- [ ] **网络**：限制安全组、无公开数据库
- [ ] **日志**：启用带保留的CloudWatch/日志记录
- [ ] **监控**：设置异常警报
- [ ] **CI/CD**：OIDC认证、机密扫描、依赖审计
- [ ] **CDN/WAF**：启用带OWASP规则的Cloudflare WAF
- [ ] **加密**：加密静态和传输中的数据
- [ ] **备份**：带已测试恢复的自动备份
- [ ] **合规**：满足GDPR/HIPAA要求（如适用）
- [ ] **文档**：记录基础设施、创建运行手册
- [ ] **事件响应**：部署安全事件计划

## 常见云安全配置错误

### S3存储桶暴露

```bash
# FAIL: 错误：公开存储桶
aws s3api put-bucket-acl --bucket my-bucket --acl public-read

# PASS: 正确：带特定访问的私有存储桶
aws s3api put-bucket-acl --bucket my-bucket --acl private
aws s3api put-bucket-policy --bucket my-bucket --policy file://policy.json
```

### RDS公开访问

```terraform
# FAIL: 错误
resource "aws_db_instance" "bad" {
  publicly_accessible = true  # 绝对不要这样做！
}

# PASS: 正确
resource "aws_db_instance" "good" {
  publicly_accessible = false
  vpc_security_group_ids = [aws_security_group.db.id]
}
```

## 资源

- [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)
- [Cloudflare Security Documentation](https://developers.cloudflare.com/security/)
- [OWASP Cloud Security](https://owasp.org/www-project-cloud-security/)
- [Terraform Security Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/)

**请记住**：云配置错误是数据泄露的主要原因。一个暴露的S3存储桶或过于宽松的IAM策略可能会危及整个基础设施。始终遵循最小权限原则和深度防御。
