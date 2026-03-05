---
name: deepnotion-loop
description: Use when acting as Chief Engineer for DeepNotion project — designing round task docs, running the 8-step development loop with Playwright UI verification and code-simplifier review, or writing ralph-loop PROMPT.md files.
---

# DeepNotion 开发循环 — 总工程师工作流

## 角色定位

**总工程师（Chief Engineer）**：设计架构、写执行文档、审查结果，不直接写代码。
用户在终端运行 ralph-loop 执行代码，你负责规划和审查。

## 8步开发循环

每个 Round (rXX) 必须完整走完：

```
Step 1 审查  → 读取所有相关文件，理解现有代码（用 Explore agent）
Step 2 思考  → 影响分析、旧方案识别、边界条件
Step 3 设计  → 改动范围、数据流、类型、错误处理
Step 4 迭代  → 最小增量编码，每步 typecheck（在 ralph-loop 内执行）
Step 5 审计  → code-simplifier agent 审视代码质量
Step 6 测试  → typecheck → test → Playwright UI 验证
Step 7 清理  → 删废弃代码、合并重复、清 TODO
Step 8 提交  → git add 逐文件（含文档）→ commit → 更新项目文档体系
```

## 交付物结构

每个 Round 必须在 `docs/v3/rounds/rXX/` 创建三个文件：

### PM.md 结构（产品经理文档）
```markdown
# RXX: 标题 — 产品说明

## 用户场景
> 用户在什么情况下会遇到/使用这个功能？描述真实使用场景。

## 变更内容

### 页面：[页面名称]（如 笔记本详情页）
- **位置**：[具体区域，如 左侧栏源材料列表]
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
1. ✅ ...
```

### PROMPT.md 结构（Ralph Loop 驱动）
```markdown
# RXX: 标题 — Ralph Loop Prompt

## 任务（1-2句）
**完整规格**: docs/v3/rounds/rXX/task.md（执行前必读）

## 执行规则（6条固定规则）
## 进度检查（Phase 完成标志表）
## Phase 关键提示（每个 Phase 的精确操作指导）

## 最终交付（最后一个 Phase 完成后）
1. 在 `docs/v3/rounds/rXX/PM.md` 写入产品说明文档（参考 task.md 同目录下的 PM.md 模板）
2. 更新 `docs/v3/待处理记录.md`（对应条目移入已处理）
3. git add + commit 所有文档

## 完成条件
当以下全部满足时，输出：
<promise>RXX COMPLETE</promise>

## 注意事项
```

## Playwright Electron 验证（Step 6 必做）

**重要：本项目运行在 Electron 模式，不是纯浏览器。** Playwright 验证必须在 Electron 环境下进行。

每个涉及 UI 的 Round，Ralph 必须在完成 typecheck 后：

```bash
# Electron dev 模式已在 http://localhost:3001 运行
# 使用 Playwright MCP 工具：
# 1. browser_navigate → http://localhost:3001/notebook/[test-id]
# 2. browser_snapshot → 确认页面结构
# 3. browser_click → 与改动的组件交互
# 4. browser_console_messages → 检查 JS 错误
# 注意：检查 localStorage key 时用实际代码中的 key 名（先读代码确认）
```

PROMPT.md 中须包含 Playwright 验证步骤，指定：
- 访问路径（具体到笔记本 ID，参考 playwright-baseline.md）
- 要验证的改动点（3条以内，精确描述）
- 预期结果
- 明确标注「在 Electron 模式下验证」

## code-simplifier 审计（Step 5）

Ralph 在完成编码（Step 4）后，立即调用：

```
使用 code-simplifier:code-simplifier agent 审查本次改动的文件：
- 关注：重复逻辑、不必要的 state、过长的 handler
- 不关注：已有代码风格（只看本次改动）
- 输出：是否有需要简化的地方，如有则修复后再继续
```

PROMPT.md 中须明确指定触发时机：**每个 Phase 编码完成后，typecheck 前**。

## 文档体系更新（Step 8 必做）

每个 Round 完成后，不仅提交代码，还必须更新整个文档体系：

### 8a. Round 交付文档
- 将 PM.md 写入 `docs/v3/rounds/rXX/PM.md`（Ralph 编码完成后由总工程师撰写或由 PROMPT.md 指示 Ralph 生成）
- PM.md + task.md + PROMPT.md 一起 git commit

### 8b. 待处理记录
- 将 `docs/v3/待处理记录.md` 中对应条目从「待处理问题」移入「已经处理问题」
- 格式：`·问题简述（RXX）\n  处理方式一句话`

### 8c. 项目级文档同步
根据本轮改动，检查并更新以下文档（有变化才改，没变化不动）：
- **MEMORY.md** — Round 进度表、测试覆盖进度、产品路线图状态
- **CLAUDE.md** — 如果新增了架构模式、约定、关键路径（一般不需要改）
- **产品路线图**（MEMORY.md 中）— 里程碑完成状态

### 8d. git commit 范围
```bash
# 代码 + 测试
git add <改动的源码文件> <测试文件>
git commit -m "feat/test(RXX-PN): 功能描述"

# 文档（单独或合并提交均可）
git add docs/v3/rounds/rXX/ docs/v3/待处理记录.md
git commit -m "docs(RXX): 交付文档 + 记录更新"
```

## Ralph Loop 启动命令（单轮，在 Claude Code 内）

```bash
/ralph-loop "$(cat docs/v3/rounds/rXX/PROMPT.md)" --max-iterations 12 --completion-promise "RXX COMPLETE"
```

## 多 Round 批量执行（系统终端，无人值守）

**场景**：连续跑多个 Round，从系统终端（Terminal.app）调用 `claude -p`，每轮独立 Claude 会话。

### 关键：Ralph Loop 状态文件必须手动预写

直接调用 `claude -p` 时不经过 `/ralph-loop` skill，stop hook 拿不到 completion promise → 无限循环（34次惨案）。

**解法**：每轮前手动写入 `.claude/ralph-loop.local.md`：

```
---
iteration: 1
max_iterations: 8
completion_promise: R12 COMPLETE
session_id:
---

<PROMPT.md 内容>
```

stop hook 检测到 `<promise>R12 COMPLETE</promise>` → 自动删除文件 → 退出循环。`session_id:` 留空（空值跳过 session 隔离检查，允许外部脚本使用）。

### 项目脚本

```bash
# 项目已有：run-all-rounds.sh
bash run-all-rounds.sh r12 r28   # 跑 R12-R28
bash run-all-rounds.sh r15 r15   # 只跑单轮
```

脚本已包含状态文件预写逻辑。如需在其他项目复用，参考下方通用模板（见 mtx-loop skill）。

## 进入下一 Round 的触发

Round 完成（PROGRESS.md 写入 + 待处理记录更新）后，询问用户是否规划下一个 Round，然后：
1. 读取 `待处理记录.md` 确认下一优先问题
2. 读取相关代码文件
3. 创建 `docs/v3/rounds/rXX/task.md` + `PROMPT.md` + `PM.md`
4. 提供启动命令

## 项目快速参考

| 条目 | 值 |
|------|-----|
| Dev Server | `http://localhost:3001` |
| 验证顺序 | `typecheck → test → Playwright → build` |
| Commit 格式 | `feat(RXX-PN): 简短描述` |
| 测试笔记本 | `collection-ddc35190-3da5-44ac-b69b-7249e8cbf20f` |
| 待处理记录 | `docs/v3/待处理记录.md` |
| Playwright 基线 | `docs/v3/playwright-baseline.md` |
