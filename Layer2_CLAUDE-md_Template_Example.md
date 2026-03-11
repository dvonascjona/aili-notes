# CLAUDE.md 生成模板参考 (Layer 2)
# 用途：展示 Layer 1 元提示词最终输出的 CLAUDE.md 应该长什么样
# 版本：v1.0 | 2025-03-11 | 初始版本

---

以下是一个**示例输出**——假设用户需求是「n8n + 飞书集成的直播复盘系统」，
Layer 1 元提示词走完 Phase 1-4 后应输出类似如下的 CLAUDE.md：

---

```markdown
# Live-Review-System

n8n-based live-stream retrospective workflow integrating Lark/Feishu APIs,
multimodal AI analysis, and structured data output.

## Tech Stack

- n8n (self-hosted, Docker, v1.x)
- Node.js 18+ (for n8n custom functions)
- Lark/Feishu Open Platform API
- MySQL 8.0 (data persistence)
- 1Panel (server management)

## Commands

```bash
# 启动 n8n（开发）
docker compose -f docker-compose.dev.yml up -d

# 查看 n8n 日志
docker logs -f n8n --tail 100

# 导出 n8n workflow 为 JSON
docker exec n8n n8n export:workflow --all --output=/data/backups/workflows.json

# 数据库迁移
mysql -u root -p live_review < migrations/latest.sql

# 运行测试 webhook（模拟飞书回调）
curl -X POST http://localhost:5678/webhook/lark-callback \
  -H "Content-Type: application/json" \
  -d @tests/fixtures/lark_event_sample.json
```

## Architecture

```
/
├── n8n-workflows/        # Exported workflow JSON files (version controlled)
├── scripts/              # Helper scripts (backup, migration, health-check)
├── tests/
│   └── fixtures/         # Sample payloads for testing
├── docs/
│   ├── api-patterns.md   # Lark API auth flow & error handling patterns
│   └── data-schema.md    # MySQL table schemas & relationships
├── migrations/           # SQL migration files
└── docker-compose.yml
```

- `n8n-workflows/`: 每个 workflow 单独一个 JSON 文件，文件名格式 `{用途}_{版本}.json`
- `scripts/`: 所有脚本必须有 `--dry-run` 参数
- `docs/`: 详细规范文档，通过 @引用加载

## Workflows

### 新功能开发流程

1. 在 n8n UI 中创建/修改 workflow
2. 用测试 fixture 验证（见 Commands 中的 curl 命令）
3. 导出 workflow JSON 到 `n8n-workflows/`
4. `git add` + commit（commit message 格式：`feat(workflow): 简述变更`）
5. 在 docs/ 中更新相关文档（如果涉及新 API 或新数据表）

### 故障排查流程

1. 检查 n8n execution log（n8n UI → Executions）
2. 检查 Docker 日志
3. 检查飞书 API 调用日志（open.feishu.cn → 开发者后台 → 日志）
4. 检查 MySQL 连接与查询性能

## Code Conventions

- n8n Code 节点中的 JS：使用 `try/catch` 包裹所有外部 API 调用
- 错误处理模式：每个 HTTP Request 节点后必须接 IF 节点检查 `statusCode !== 200`
- 变量命名：n8n 中使用 `snake_case`，JSON payload 中保持与上游 API 一致
- 所有 hardcoded 值必须通过 n8n Credentials 或 Environment Variables 管理

## Key References

@docs/api-patterns.md  — Lark API 鉴权流程、token 刷新、错误码速查
@docs/data-schema.md   — 数据库表结构、索引策略、查询模板

External:
- n8n 官方节点文档: https://docs.n8n.io/integrations/builtin/
- n8n 社区 workflow 参考: https://n8n.io/workflows/
- 飞书开放平台 API: https://open.feishu.cn/document/server-docs/overview
- 飞书事件订阅: https://open.feishu.cn/document/uAjLw4CM/ukTMukTMukTM/event-subscription-guide/overview

## Boundaries

- **禁止直接修改**: `docker-compose.yml`（变更需走 PR review）
- **禁止 hardcode**: 任何 API key、token、密码——必须用环境变量
- **禁止**: 在 n8n Code 节点中引入 npm 包（使用 n8n 内置函数或 HTTP Request 节点替代）
- **数据安全**: 日志输出中禁止打印完整的 access_token，最多显示前 8 位

## Terminology

- **场控 (broadcast operator)**: 直播间的运营操作人员
- **复盘 (retrospective/review)**: 对单场直播的数据与操作进行回顾分析
- **话术 (script/talking points)**: 主播在直播中使用的标准化话术模板
```

---

## 配套文件建议

### `.claude/commands/new-workflow.md`

```markdown
创建一个新的 n8n workflow：

1. 确认 workflow 用途和触发方式（Webhook / Cron / Manual）
2. 设计节点链路：Trigger → Process → Action
3. 为每个节点指定：节点类型、关键参数、错误处理
4. 生成测试用的 fixture JSON（保存到 tests/fixtures/）
5. 输出 workflow 的 n8n JSON 导入格式
6. 更新 docs/ 中的相关文档
```

### `.claude/commands/debug.md`

```markdown
排查当前问题：

1. 读取用户描述的错误现象
2. 检查 n8n-workflows/ 中相关的 workflow JSON
3. 检查 docs/api-patterns.md 中的已知错误模式
4. 给出排查步骤（按可能性从高到低排列）
5. 如果需要修改 workflow，生成修改后的 JSON 并说明变更点
```

### Hooks 建议

```bash
# .claude/hooks/post-edit.sh
# 每次文件编辑后自动格式化 JSON
if [[ "$EDITED_FILE" == *.json ]]; then
  python3 -m json.tool "$EDITED_FILE" > /tmp/formatted.json && mv /tmp/formatted.json "$EDITED_FILE"
fi
```
