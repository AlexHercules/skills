---
name: mtx-loop
description: Generic Chief Engineer workflow for any project — designing round task docs, running the 8-step development loop with code review, testing, UI verification, and documentation delivery.
---

# MTX 开发循环 — 总工程师工作流

> 通用版本，适用于任何项目。项目特定配置见底部「项目配置」section。

## 角色定位

**总工程师（Chief Engineer）**：设计架构、写执行文档、审查结果，不直接写代码。
用户在终端运行 ralph-loop 执行代码，你负责规划和审查。

## 项目配置（渐进式）

配置文件：项目根目录 `mtx.config.md`。

### 启动流程

1. 检查项目根目录是否存在 `mtx.config.md`
2. **不存在** → 用默认值创建，只问用户一个问题：「项目名称是什么？」
3. **已存在** → 直接读取
4. 配置项值为 `?` 表示待定 — 当工作流实际需要该配置时，再问用户并**立即回写到 mtx.config.md**

### 配置项与默认值

| 配置项 | 默认值 | 何时需要 |
|--------|--------|---------|
| `PROJECT_NAME` | （首次必填） | 创建文件时 |
| `ROUNDS_DIR` | `docs/rounds` | Step 3 设计交付物时 |
| `BACKLOG_FILE` | `docs/backlog.md` | Step 8 更新记录时 |
| `DEV_SERVER` | `http://localhost:3000` | Step 6 UI 验证时 |
| `APP_MODE` | `browser` | Step 6 UI 验证时 |
| `TYPECHECK_CMD` | `npm run typecheck` | Step 4/6 验证时 |
| `TEST_CMD` | `npm run test` | Step 6 测试时 |
| `BUILD_CMD` | `npm run build` | 最终构建验证时 |
| `COMMIT_PREFIX` | `feat(RXX-PN):` | Step 8 提交时 |
| `TEST_ENTRY` | `?` | Step 6 UI 验证时（可选） |
| `BASELINE_DOC` | `?` | Step 6 UI 验证时（可选） |

### 回写规则

对话中任何时候确认了某个配置的实际值（用户明确说了、或从代码/package.json 中推断出来），**立即更新 mtx.config.md**，将 `?` 或默认值替换为实际值。不要等到 Round 结束。

### mtx.config.md 格式

```markdown
# MTX Loop 项目配置

| 配置项 | 值 |
|--------|-----|
| PROJECT_NAME | MyProject |
| ROUNDS_DIR | docs/rounds |
| BACKLOG_FILE | docs/backlog.md |
| DEV_SERVER | http://localhost:3000 |
| APP_MODE | browser |
| TYPECHECK_CMD | npm run typecheck |
| TEST_CMD | npm run test |
| BUILD_CMD | npm run build |
| COMMIT_PREFIX | feat(RXX-PN): |
| TEST_ENTRY | ? |
| BASELINE_DOC | ? |
```

此文件应 git commit 到项目中，团队共享。

## 8步开发循环

每个 Round (rXX) 必须完整走完：

```
Step 1 审查  → 读取所有相关文件，理解现有代码（用 Explore agent）
Step 2 思考  → 影响分析、旧方案识别、边界条件
Step 3 设计  → 改动范围、数据流、类型、错误处理
Step 4 迭代  → 最小增量编码，每步 typecheck（在 ralph-loop 内执行）
Step 5 审计  → code-simplifier agent 审视代码质量
Step 6 测试  → typecheck → test → UI 验证（Playwright）
Step 7 清理  → 删废弃代码、合并重复、清 TODO
Step 8 提交  → git add 逐文件（含文档）→ commit → 更新项目文档体系
```

## 交付物结构

每个 Round 必须在 `{ROUNDS_DIR}/rXX/` 创建三个文件：

### PM.md 结构（产品经理文档）
```markdown
# RXX: 标题 — 产品说明

## 用户场景
> 用户在什么情况下会遇到/使用这个功能？描述真实使用场景。

## 变更内容

### 页面：[页面名称]
- **位置**：[具体区域]
- **当前状态**：[改动前用户看到什么/能做什么]
- **改动后**：[改动后用户看到什么/能做什么]

### 页面：...（如有多个页面受影响）

## 功能清单
| 功能点 | 用户可感知的变化 |
|--------|----------------|
| ... | ... |

## 边界情况
- [用户可能遇到的特殊情况及处理方式]
```

**写作原则**：
- 零技术术语（不提组件名、函数名、类型名）
- 从用户视角描述：「在 XX 页面，点击 XX 后，现在会 XX」
- 如果是纯后端/测试/基础设施 Round，PM.md 写明「本轮无用户可感知变化」并简述内部改进目的

### task.md 结构
```markdown
# RXX: 标题

> 主线/来源/范围/不含/目标/前置

## 依赖文件（执行前必读）
| 文件 | 作用 |

## Phase 清单
| Phase | 内容 | 改动文件 | 验证 |

## P1: ...（含精确行号、代码片段、验证命令）

## 验收标准（全部满足才算完成）
1. ...
```

### PROMPT.md 结构（Ralph Loop 驱动）
```markdown
# RXX: 标题 — Ralph Loop Prompt

## 任务（1-2句）
**完整规格**: {ROUNDS_DIR}/rXX/task.md（执行前必读）

## 执行规则（6条固定规则）
## 进度检查（Phase 完成标志表）
## Phase 关键提示（每个 Phase 的精确操作指导）

## 最终交付（最后一个 Phase 完成后）
1. 在 `{ROUNDS_DIR}/rXX/PM.md` 写入产品说明文档
2. 更新 `{BACKLOG_FILE}`（对应条目移入已处理）
3. git add + commit 所有文档

## 完成条件
当以下全部满足时，输出：
<promise>RXX COMPLETE</promise>

## 注意事项
```

## UI 验证（Step 6 必做）

每个涉及 UI 的 Round，Ralph 必须在完成 typecheck 后用 Playwright MCP 验证。

**根据 APP_MODE 调整验证方式**：
- `electron`：在 Electron 窗口中验证，注意 localStorage key 需先读代码确认
- `browser`：直接在浏览器中验证
- `mobile`：通过模拟器或真机验证

```bash
# 使用 Playwright MCP 工具：
# 1. browser_navigate → {DEV_SERVER}/[目标路由]
# 2. browser_snapshot → 确认页面结构
# 3. browser_click → 与改动的组件交互
# 4. browser_console_messages → 检查 JS 错误
```

PROMPT.md 中须包含 Playwright 验证步骤，指定：
- 访问路径（具体路由）
- 要验证的改动点（3条以内，精确描述）
- 预期结果
- 运行模式（如「在 Electron 模式下验证」）

## code-simplifier 审计（Step 5）

Ralph 在完成编码（Step 4）后，立即调用：

```
使用 code-simplifier agent 审查本次改动的文件：
- 关注：重复逻辑、不必要的 state、过长的 handler
- 不关注：已有代码风格（只看本次改动）
- 输出：是否有需要简化的地方，如有则修复后再继续
```

PROMPT.md 中须明确指定触发时机：**每个 Phase 编码完成后，typecheck 前**。

## 文档体系更新（Step 8 必做）

每个 Round 完成后，不仅提交代码，还必须更新整个文档体系：

### 8a. Round 交付文档
- 将 PM.md 写入 `{ROUNDS_DIR}/rXX/PM.md`
- PM.md + task.md + PROMPT.md 一起 git commit

### 8b. 待处理记录
- 将 `{BACKLOG_FILE}` 中对应条目从「待处理」移入「已处理」
- 格式：`·问题简述（RXX）\n  处理方式一句话`

### 8c. 项目级文档同步
根据本轮改动，检查并更新以下文档（有变化才改，没变化不动）：
- **MEMORY.md** — Round 进度表、测试覆盖进度、路线图状态
- **CLAUDE.md** — 如果新增了架构模式、约定、关键路径（一般不需要改）
- **产品路线图** — 里程碑完成状态

### 8d. git commit 范围
```bash
# 代码 + 测试
git add <改动的源码文件> <测试文件>
git commit -m "{COMMIT_PREFIX} 功能描述"

# 文档（单独或合并提交均可）
git add {ROUNDS_DIR}/rXX/ {BACKLOG_FILE}
git commit -m "docs(RXX): 交付文档 + 记录更新"
```

## Ralph Loop 启动命令（单轮，在 Claude Code 内）

```bash
/ralph-loop "$(cat {ROUNDS_DIR}/rXX/PROMPT.md)" --max-iterations 12 --completion-promise "RXX COMPLETE"
```

## 多 Round 批量执行（系统终端，无人值守）

**场景**：连续跑多个 Round，从系统终端（Terminal.app）调用 `claude -p`，每轮独立 Claude 会话。

### 关键：Ralph Loop 状态文件必须手动预写

直接调用 `claude -p` 时不经过 `/ralph-loop` skill，stop hook 拿不到 completion promise。
**不预写状态文件 = stop hook 无限循环**（completion promise 不匹配，不停重试）。

状态文件路径：`.claude/ralph-loop.local.md`（项目根目录）

```
---
iteration: 1
max_iterations: 8
completion_promise: RXX COMPLETE
session_id:
---

<PROMPT.md 内容>
```

说明：
- `session_id:` 留空 → 跳过 session 隔离检查，允许外部脚本调用
- `max_iterations: 8` → 最多重试 8 次（正常 1 次完成，出错才重试）
- stop hook 检测到 `<promise>RXX COMPLETE</promise>` → 自动删除文件 → 退出

### 通用批量执行脚本模板

```bash
#!/bin/bash
# 用法: bash run-all-rounds.sh [起始轮次] [结束轮次]
set -euo pipefail
cd "$(dirname "$0")"

START="${1:-r01}"
END="${2:-r99}"
ROUNDS_DIR="docs/rounds"   # 改为你的 ROUNDS_DIR

ALL_ROUNDS=(r01 r02 r03)   # 填入完整轮次列表

ROUNDS=()
in_range=false
for r in "${ALL_ROUNDS[@]}"; do
  [ "$r" = "$START" ] && in_range=true
  $in_range && ROUNDS+=("$r")
  [ "$r" = "$END" ] && break
done

FAILED=()

for r in "${ROUNDS[@]}"; do
  PROMPT_FILE="$ROUNDS_DIR/$r/PROMPT.md"
  [ ! -f "$PROMPT_FILE" ] && { FAILED+=("$r-missing"); continue; }

  # 关键：预写 ralph-loop 状态文件，配置 completion promise
  PROMISE="$(echo "$r" | tr '[:lower:]' '[:upper:]') COMPLETE"
  mkdir -p .claude
  cat > .claude/ralph-loop.local.md << STATEFILE
---
iteration: 1
max_iterations: 8
completion_promise: $PROMISE
session_id:
---

$(cat "$PROMPT_FILE")
STATEFILE

  echo "🚀 $r 开始  $(date +%H:%M:%S)"
  set +e
  claude --dangerously-skip-permissions -p "$(cat "$PROMPT_FILE")"
  EXIT_CODE=$?
  set -e
  rm -f .claude/ralph-loop.local.md  # 异常时清理

  [ $EXIT_CODE -eq 0 ] && echo "✅ $r 完成" || { echo "❌ $r 失败"; FAILED+=("$r"); }
done

[ ${#FAILED[@]} -eq 0 ] && echo "🎉 全部完成" || echo "⚠️ 失败: ${FAILED[*]}"
```

## 进入下一 Round 的触发

Round 完成后，询问用户是否规划下一个 Round，然后：
1. 读取 `{BACKLOG_FILE}` 确认下一优先问题
2. 读取相关代码文件
3. 创建 `{ROUNDS_DIR}/rXX/task.md` + `PROMPT.md` + `PM.md`
4. 提供启动命令
