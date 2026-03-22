---
name: claude-code-anatomy
description: 基于"4要素框架"快速搭建 Claude Code 项目结构。提供 CLAUDE.md 模板、Skills 设计规范、Hooks 配置和渐进式文档组织方式。
---

# Claude Code 项目骨架

基于 Vaidehi 的「4要素框架」，快速搭建适合 AI 协作的项目结构。

## 核心原则

> Prompting 是临时的，Structure 是永久的。

混乱的仓库 → Claude 像聊天机器人
结构化的仓库 → Claude 像住在代码库里的工程师

## 4要素框架

```
项目/
├── CLAUDE.md              # 仓库记忆（简短）
├── .claude/
│   ├── skills/           # 可复用专家模式
│   └── hooks/            # 自动护栏
├── docs/                 # 渐进式上下文
└── src/
    └── [模块]/
        └── CLAUDE.md     # 局部上下文（危险区域）
```

## 要素1：CLAUDE.md = 仓库记忆

**保持简短，只放三件事：**

```markdown
# Project Name

## Purpose
这个系统为什么存在，解决什么问题。

## Repo Map
```
src/
├── api/          # HTTP handlers
├── domain/       # 业务逻辑
├── persistence/  # 数据访问
└── contracts/    # 共享类型
```

## Rules & Commands

### 必须遵守
- 永远不要直接修改 .env 文件
- 提交前必须跑通测试
- 敏感目录（auth/ billing/）修改需额外确认

### 常用命令
```bash
pnpm install    # 安装依赖
pnpm dev        # 开发模式
pnpm test       # 运行测试
pnpm lint       # 代码检查
```
```

**警告：** 超过 200 行会导致模型丢失关键信号。清晰 > 篇幅。

## 要素2：.claude/skills/ = 可复用专家模式

**把重复指令变成可复用技能：**

```
.claude/skills/
├── code-review/
│   └── SKILL.md
├── refactor/
│   └── SKILL.md
├── debug/
│   └── SKILL.md
└── release/
    └── SKILL.md
```

**code-review SKILL.md 示例：**

```markdown
---
name: code-review
description: Code review checklist for PRs. Use when reviewing changes.
---

## Checklist
- [ ] 代码逻辑正确性
- [ ] 错误处理完备
- [ ] 测试覆盖新增逻辑
- [ ] 无安全问题（SQL注入、XSS等）
- [ ] 性能无明显退化

## Output
每项 Pass / Fail，Fail 必须给出具体修复建议。
```

**效果：** 跨会话、跨团队成员输出一致。

## 要素3：.claude/hooks/ = 自动护栏

**模型会忘，Hooks 不会。**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "pattern": "*.ts",
        "hooks": [
          {
            "type": "command",
            "command": "pnpm lint --fix $FILE",
            "statusMessage": "Running linter..."
          }
        ]
      },
      {
        "matcher": "Edit",
        "pattern": "src/auth/**/*",
        "hooks": [
          {
            "type": "command", 
            "command": "echo '⚠️  Auth module modified - review required'",
            "statusMessage": "Auth zone warning"
          }
        ]
      }
    ]
  }
}
```

**典型场景：**
- 编辑后自动格式化
- 核心文件变更后触发测试
- 敏感目录（auth/ billing/ migrations/）阻断或警告

## 要素4：docs/ = 渐进式上下文

**不要让 Claude 记住所有信息，让它知道去哪里找。**

```
docs/
├── architecture/         # 系统架构
│   ├── overview.md
│   └── data-flow.md
├── decisions/            # ADRs (架构决策记录)
│   └── 001-why-postgres.md
└── runbooks/            # 运维手册
    ├── incident-response.md
    └── rollback.md
```

**原则：** Claude 不需要所有信息在内存里，只需要知道 truth 在哪里。

## 要素5：局部 CLAUDE.md（危险区域）

**复杂模块需要额外上下文：**

```
src/
├── auth/
│   ├── ...
│   └── CLAUDE.md        # 认证流程、安全约束
├── persistence/
│   ├── ...
│   └── CLAUDE.md        # 事务边界、迁移规则
└── billing/
    ├── ...
    └── CLAUDE.md        # 支付流程、金额计算精度
```

**效果：** Claude 在危险区域工作时精确理解约束，大幅减少错误。

## 快速启动模板

### 最小可用结构

```bash
# 1. 创建根 CLAUDE.md
cat > CLAUDE.md << 'EOF'
# [项目名]

## Purpose
[一句话描述系统存在的理由]

## Repo Map
[目录结构说明]

## Rules
- [约束1]
- [约束2]

## Commands
- `cmd1` - 描述
- `cmd2` - 描述
EOF

# 2. 创建 .claude 目录
mkdir -p .claude/{skills,hooks}

# 3. 创建第一个 Skill
cat > .claude/skills/review/SKILL.md << 'EOF'
---
name: review
description: Code review checklist. Use for PR reviews.
---

## Checklist
1. 逻辑正确性
2. 错误处理
3. 测试覆盖
4. 安全问题

## Output
Pass/Fail per item with specific fixes.
EOF

# 4. 添加 Hooks 配置
cat > .claude/hooks.json << 'EOF'
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "pattern": "*.[js|ts|py]",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'File modified: $FILE'",
            "statusMessage": "Checking..."
          }
        ]
      }
    ]
  }
}
EOF

# 5. 创建 docs 目录
mkdir -p docs/{architecture,decisions,runbooks}
```

## 质量检查清单

- [ ] CLAUDE.md < 200 行
- [ ] 有清晰的 Purpose、Repo Map、Rules
- [ ] Skills 有明确触发条件和输出格式
- [ ] Hooks 有合理范围，不过度触发
- [ ] 复杂模块有局部 CLAUDE.md
- [ ] docs/ 结构清晰，不是信息堆积

## 常见错误

| 错误 | 正确做法 |
|------|----------|
| CLAUDE.md 写成长篇文档 | 保持简短，链接到详细文档 |
| Skills 描述模糊 | 明确"何时使用"而非"这是什么" |
| Hooks 过度触发 | 限定文件类型和目录范围 |
| 所有信息塞给 Claude | 渐进式披露，按需加载 |
| 一次配置永久不管 | 随项目演进迭代更新 |

## 关键洞察

1. **结构 > 提示** — 好的项目结构比完美的提示词更重要
2. **清晰 > 完整** — CLAUDE.md 要简短有力，不是百科全书
3. **自动 > 提醒** — Hooks 强制执行，不依赖记忆
4. **分层 > 集中** — 局部上下文处理复杂模块
5. **演进 > 完美** — 先跑起来，再迭代优化
