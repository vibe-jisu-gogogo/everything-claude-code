| name | description |
|------|-------------|
| cloud-infrastructure-security | 在云平台部署、基础设施配置、IAM 策略管理、日志/监控设置、CI/CD 流水线实施时使用此技能。提供符合最佳实践的云安全清单。 |

# 云及基础设施安全技能

此技能确保云基础设施、CI/CD 流水线、部署配置遵循安全最佳实践并遵守行业标准。

## 激活时机

- 在云平台（AWS、Vercel、Railway、Cloudflare）部署应用时
- 配置 IAM 角色及权限时
- 设置 CI/CD 流水线时
- 实施 Infrastructure as Code（Terraform、CloudFormation）时
- 配置日志及监控时
- 在云环境中管理密钥（Secret）时
- 设置 CDN 及边缘安全时
- 实施灾难恢复及备份策略时

## 云安全清单

### 1. IAM 及访问控制

#### 最小权限原则

```yaml
# PASS: CORRECT: Minimal permissions
iam_role:
  permissions:
    - s3:GetObject  # Only read access
    - s3:ListBucket
  resources:
    - arn:aws:s3:::my-bucket/*  # Specific bucket only

# FAIL: WRONG: Overly broad permissions
iam_role:
  permissions:
    - s3:*  # All S3 actions
  resources:
    - "*"  # All resources
```

#### 多因素认证（MFA）

```bash
# ALWAYS enable MFA for root/admin accounts
aws iam enable-mfa-device \
  --user-name admin \
  --serial-number arn:aws:iam::123456789:mfa/admin \
  --authentication-code1 123456 \
  --authentication-code2 789012
```

#### 检查步骤

- [ ] 不使用生产环境的 root 账户
- [ ] 所有特权账户已启用 MFA
- [ ] 服务账户使用角色而非长期凭证
- [ ] IAM 策略遵循最小权限
- [ ] 执行定期访问审查
- [ ] 轮换或移除未使用的凭证

### 2. 密钥管理

#### 云密钥管理器

```typescript
// PASS: CORRECT: Use cloud secrets manager
import { SecretsManager } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManager({ region: 'us-east-1' });
const secret = await client.getSecretValue({ SecretId: 'prod/api-key' });
const apiKey = JSON.parse(secret.SecretString).key;

// FAIL: WRONG: Hardcoded or in environment variables only
const apiKey = process.env.API_KEY; // Not rotated, not audited
```

#### 密钥轮换

```bash
# Set up automatic rotation for database credentials
aws secretsmanager rotate-secret \
  --secret-id prod/db-password \
  --rotation-lambda-arn arn:aws:lambda:region:account:function:rotate \
  --rotation-rules AutomaticallyAfterDays=30
```

#### 检查步骤

- [ ] 所有密钥存储在云密钥管理器中（AWS Secrets Manager、Vercel Secrets）
- [ ] 已启用数据库凭证的自动轮换
- [ ] API 密钥至少每季度轮换一次
- [ ] 代码、日志、错误消息中无密钥
- [ ] 已启用密钥访问的审计日志

### 3. 网络安全

#### VPC 及防火墙配置

```terraform
# PASS: CORRECT: Restricted security group
resource "aws_security_group" "app" {
  name = "app-sg"

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]  # Internal VPC only
  }

  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Only HTTPS outbound
  }
}

# FAIL: WRONG: Open to the internet
resource "aws_security_group" "bad" {
  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # All ports, all IPs!
  }
}
```

#### 检查步骤

- [ ] 数据库不可公开访问
- [ ] SSH/RDP 端口仅限制于 VPN/堡垒机
- [ ] 安全组遵循最小权限
- [ ] 已配置网络 ACL
- [ ] 已启用 VPC 流日志

### 4. 日志及监控

#### CloudWatch/日志配置

```typescript
// PASS: CORRECT: Comprehensive logging
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
        // Never log sensitive data
      })
    }]
  });
};
```

#### 检查步骤

- [ ] 所有服务已启用 CloudWatch/日志
- [ ] 失败的认证尝试已记录日志
- [ ] 管理员操作已审计
- [ ] 已配置日志保留期（为合规性保留 90 天以上）
- [ ] 已配置可疑活动的通知
- [ ] 日志已集中化并防篡改

### 5. CI/CD 流水线安全

#### 安全流水线配置

```yaml
# PASS: CORRECT: Secure GitHub Actions workflow
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read  # Minimal permissions

    steps:
      - uses: actions/checkout@v4

        # Scan for secrets
      - name: Secret scanning
        uses: trufflesecurity/trufflehog@6c05c4a00b91aa542267d8e32a8254774799d68d

        # Dependency audit
      - name: Audit dependencies
        run: npm audit --audit-level=high

        # Use OIDC, not long-lived tokens
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/GitHubActionsRole
          aws-region: us-east-1
```

#### 供应链安全

```json
// package.json - Use lock files and integrity checks
{
  "scripts": {
    "deps:install": "npm ci",  // Use ci for reproducible builds
    "audit": "npm audit --audit-level=moderate",
    "check": "npm outdated"
  }
}
```

#### 检查步骤

- [ ] 使用 OIDC 替代长期凭证
- [ ] 流水线中的密钥扫描
- [ ] 依赖漏洞扫描
- [ ] 容器镜像扫描（如适用）
- [ ] 已应用分支保护规则
- [ ] 合并前必须进行代码审查
- [ ] 已应用签名提交

### 6. Cloudflare 及 CDN 安全

#### Cloudflare 安全配置

```typescript
// PASS: CORRECT: Cloudflare Workers with security headers
export default {
  async fetch(request: Request): Promise<Response> {
    const response = await fetch(request);

    // Add security headers
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

#### WAF 规则

```bash
# Enable Cloudflare WAF managed rules
# - OWASP Core Ruleset
# - Cloudflare Managed Ruleset
# - Rate limiting rules
# - Bot protection
```

#### 检查步骤

- [ ] 已启用带 OWASP 规则的 WAF
- [ ] 已配置速率限制
- [ ] 已启用机器人保护
- [ ] 已启用 DDoS 保护
- [ ] 已配置安全头
- [ ] 已启用 SSL/TLS 严格模式

### 7. 备份及灾难恢复

#### 自动备份

```terraform
# PASS: CORRECT: Automated RDS backups
resource "aws_db_instance" "main" {
  allocated_storage     = 20
  engine               = "postgres"

  backup_retention_period = 30  # 30 days retention
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"

  enabled_cloudwatch_logs_exports = ["postgresql"]

  deletion_protection = true  # Prevent accidental deletion
}
```

#### 检查步骤

- [ ] 已配置自动每日备份
- [ ] 备份保留期满足合规性要求
- [ ] 已启用时间点恢复
- [ ] 执行季度备份测试
- [ ] 灾难恢复计划已文档化
- [ ] RPO 及 RTO 已定义并测试

## 部署前云安全清单

所有生产云部署前：

- [ ] **IAM**: 不使用 root 账户、启用 MFA、最小权限策略
- [ ] **密钥**: 所有密钥与轮换一起存储在云密钥管理器中
- [ ] **网络**: 安全组受限、无公开数据库
- [ ] **日志**: CloudWatch/日志与保留期一起启用
- [ ] **监控**: 异常迹象的通知已配置
- [ ] **CI/CD**: OIDC 认证、密钥扫描、依赖审计
- [ ] **CDN/WAF**: 已启用带 OWASP 规则的 Cloudflare WAF
- [ ] **加密**: 存储及传输中数据加密
- [ ] **备份**: 与已测试恢复一起的自动备份
- [ ] **合规性**: 满足 GDPR/HIPAA 要求（如适用）
- [ ] **文档化**: 基础设施文档化、运行手册已编写
- [ ] **事件响应**: 安全事件计划已准备

## 常见云安全错误配置

### S3 存储桶暴露

```bash
# FAIL: WRONG: Public bucket
aws s3api put-bucket-acl --bucket my-bucket --acl public-read

# PASS: CORRECT: Private bucket with specific access
aws s3api put-bucket-acl --bucket my-bucket --acl private
aws s3api put-bucket-policy --bucket my-bucket --policy file://policy.json
```

### RDS 公开访问

```terraform
# FAIL: WRONG
resource "aws_db_instance" "bad" {
  publicly_accessible = true  # NEVER do this!
}

# PASS: CORRECT
resource "aws_db_instance" "good" {
  publicly_accessible = false
  vpc_security_group_ids = [aws_security_group.db.id]
}
```

## 参考资料

- [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)
- [Cloudflare Security Documentation](https://developers.cloudflare.com/security/)
- [OWASP Cloud Security](https://owasp.org/www-project-cloud-security/)
- [Terraform Security Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/)

**请记住**: 云错误配置是数据泄露的主要原因。一个暴露的 S3 存储桶或过度宽松的 IAM 策略可能会破坏整个基础设施。始终遵循最小权限原则和深度防御。
