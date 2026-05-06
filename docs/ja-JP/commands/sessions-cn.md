# Sessions 命令

管理 Claude Code 会话历史 - 列出、加载、设置别名和编辑保存在 `~/.claude/session-data/` 中的会话。为了向后兼容，也会读取旧的 `~/.claude/sessions/` 中的文件。

## 使用方法

`/sessions [list|load|alias|info|help] [选项]`

## 操作

### 列出会话

显示所有会话，包含元数据、过滤和分页功能。

```bash
/sessions                              # 列出所有会话（默认）
/sessions list                         # 同上
/sessions list --limit 10              # 显示 10 个会话
/sessions list --date 2026-02-01       # 按日期过滤
/sessions list --search abc            # 按会话 ID 搜索
```

**脚本:**
```bash
node -e "
const sm = require('./scripts/lib/session-manager');
const aa = require('./scripts/lib/session-aliases');

const result = sm.getAllSessions({ limit: 20 });
const aliases = aa.listAliases();
const aliasMap = {};
for (const a of aliases) aliasMap[a.sessionPath] = a.name;

console.log('Sessions (showing ' + result.sessions.length + ' of ' + result.total + '):');
console.log('');
console.log('ID        Date        Time     Size     Lines  Alias');
console.log('────────────────────────────────────────────────────');

for (const s of result.sessions) {
  const alias = aliasMap[s.filename] || '';
  const size = sm.getSessionSize(s.sessionPath);
  const stats = sm.getSessionStats(s.sessionPath);
  const id = s.shortId === 'no-id' ? '(none)' : s.shortId.slice(0, 8);
  const time = s.modifiedTime.toTimeString().slice(0, 5);

  console.log(id.padEnd(8) + ' ' + s.date + '  ' + time + '   ' + size.padEnd(7) + '  ' + String(stats.lineCount).padEnd(5) + '  ' + alias);
}
"
```

### 加载会话

加载并显示会话内容（通过 ID 或别名指定）。

```bash
/sessions load <id|alias>             # 加载会话
/sessions load 2026-02-01             # 按日期指定（无 ID 的会话）
/sessions load a1b2c3d4               # 按短 ID 指定
/sessions load my-alias               # 按别名指定
```

**脚本:**
```bash
node -e "
const sm = require('./scripts/lib/session-manager');
const aa = require('./scripts/lib/session-aliases');
const id = process.argv[1];

// First try to resolve as alias
const resolved = aa.resolveAlias(id);
const sessionId = resolved ? resolved.sessionPath : id;

const session = sm.getSessionById(sessionId, true);
if (!session) {
  console.log('Session not found: ' + id);
  process.exit(1);
}

const stats = sm.getSessionStats(session.sessionPath);
const size = sm.getSessionSize(session.sessionPath);
const aliases = aa.getAliasesForSession(session.filename);

console.log('Session: ' + session.filename);
console.log('Path: ' + session.sessionPath);
console.log('');
console.log('Statistics:');
console.log('  Lines: ' + stats.lineCount);
console.log('  Total items: ' + stats.totalItems);
console.log('  Completed: ' + stats.completedItems);
console.log('  In progress: ' + stats.inProgressItems);
console.log('  Size: ' + size);
console.log('');

if (aliases.length > 0) {
  console.log('Aliases: ' + aliases.map(a => a.name).join(', '));
  console.log('');
}

if (session.metadata.title) {
  console.log('Title: ' + session.metadata.title);
  console.log('');
}

if (session.metadata.started) {
  console.log('Started: ' + session.metadata.started);
}

if (session.metadata.lastUpdated) {
  console.log('Last Updated: ' + session.metadata.lastUpdated);
}
" "$ARGUMENTS"
```

### 创建别名

为会话创建易于记忆的别名。

```bash
/sessions alias <id> <name>           # 创建别名
/sessions alias 2026-02-01 today-work # 创建名为 "today-work" 的别名
```

**脚本:**
```bash
node -e "
const sm = require('./scripts/lib/session-manager');
const aa = require('./scripts/lib/session-aliases');

const sessionId = process.argv[1];
const aliasName = process.argv[2];

if (!sessionId || !aliasName) {
  console.log('Usage: /sessions alias <id> <name>');
  process.exit(1);
}

// Get session filename
const session = sm.getSessionById(sessionId);
if (!session) {
  console.log('Session not found: ' + sessionId);
  process.exit(1);
}

const result = aa.setAlias(aliasName, session.filename);
if (result.success) {
  console.log('✓ Alias created: ' + aliasName + ' → ' + session.filename);
} else {
  console.log('✗ Error: ' + result.error);
  process.exit(1);
}
" "$ARGUMENTS"
```

### 删除别名

删除现有的别名。

```bash
/sessions alias --remove <name>        # 删除别名
/sessions unalias <name>               # 同上
```

**脚本:**
```bash
node -e "
const aa = require('./scripts/lib/session-aliases');

const aliasName = process.argv[1];
if (!aliasName) {
  console.log('Usage: /sessions alias --remove <name>');
  process.exit(1);
}

const result = aa.deleteAlias(aliasName);
if (result.success) {
  console.log('✓ Alias removed: ' + aliasName);
} else {
  console.log('✗ Error: ' + result.error);
  process.exit(1);
}
" "$ARGUMENTS"
```

### 会话信息

显示会话的详细信息。

```bash
/sessions info <id|alias>              # 显示会话详情
```

**脚本:**
```bash
node -e "
const sm = require('./scripts/lib/session-manager');
const aa = require('./scripts/lib/session-aliases');

const id = process.argv[1];
const resolved = aa.resolveAlias(id);
const sessionId = resolved ? resolved.sessionPath : id;

const session = sm.getSessionById(sessionId, true);
if (!session) {
  console.log('Session not found: ' + id);
  process.exit(1);
}

const stats = sm.getSessionStats(session.sessionPath);
const size = sm.getSessionSize(session.sessionPath);
const aliases = aa.getAliasesForSession(session.filename);

console.log('Session Information');
console.log('════════════════════');
console.log('ID:          ' + (session.shortId === 'no-id' ? '(none)' : session.shortId));
console.log('Filename:    ' + session.filename);
console.log('Date:        ' + session.date);
console.log('Modified:    ' + session.modifiedTime.toISOString().slice(0, 19).replace('T', ' '));
console.log('');
console.log('Content:');
console.log('  Lines:         ' + stats.lineCount);
console.log('  Total items:   ' + stats.totalItems);
console.log('  Completed:     ' + stats.completedItems);
console.log('  In progress:   ' + stats.inProgressItems);
console.log('  Size:          ' + size);
if (aliases.length > 0) {
  console.log('Aliases:     ' + aliases.map(a => a.name).join(', '));
}
" "$ARGUMENTS"
```

### 列出别名

显示所有会话别名。

```bash
/sessions aliases                      # 列出所有别名
```

**脚本:**
```bash
node -e "
const aa = require('./scripts/lib/session-aliases');

const aliases = aa.listAliases();
console.log('Session Aliases (' + aliases.length + '):');
console.log('');

if (aliases.length === 0) {
  console.log('No aliases found.');
} else {
  console.log('Name          Session File                    Title');
  console.log('─────────────────────────────────────────────────────────────');
  for (const a of aliases) {
    const name = a.name.padEnd(12);
    const file = (a.sessionPath.length > 30 ? a.sessionPath.slice(0, 27) + '...' : a.sessionPath).padEnd(30);
    const title = a.title || '';
    console.log(name + ' ' + file + ' ' + title);
  }
}
"
```

## 参数

$ARGUMENTS:
- `list [选项]` - 列出会话
  - `--limit <n>` - 显示的最大会话数（默认：50）
  - `--date <YYYY-MM-DD>` - 按日期过滤
  - `--search <模式>` - 按会话 ID 搜索
- `load <id|alias>` - 加载会话内容
- `alias <id> <name>` - 创建会话别名
- `alias --remove <name>` - 删除别名
- `unalias <name>` - 同 `--remove`
- `info <id|alias>` - 显示会话统计
- `aliases` - 列出所有别名
- `help` - 显示此帮助

## 示例

```bash
# 列出所有会话
/sessions list

# 为今天的会话创建别名
/sessions alias 2026-02-01 today

# 通过别名加载会话
/sessions load today

# 显示会话信息
/sessions info today

# 删除别名
/sessions alias --remove today

# 列出所有别名
/sessions aliases
```

## 备注

- 会话以 Markdown 文件形式保存在 `~/.claude/session-data/` 中，旧的 `~/.claude/sessions/` 中的文件仍会继续读取
- 别名保存在 `~/.claude/session-aliases.json` 中
- 会话 ID 可以缩短（通常前 4-8 个字符即可唯一标识）
- 建议为频繁引用的会话使用别名
