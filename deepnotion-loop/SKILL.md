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
Step 8 提交  → git add 逐文件 → commit → 更新待处理文档
```

## 交付物结构

每个 Round 必须在 `docs/v2/rounds/rXX/` 创建两个文件：

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
**完整规格**: docs/v2/rounds/rXX/task.md（执行前必读）

## 执行规则（6条固定规则）
## 进度检查（Phase 完成标志表）
## Phase 关键提示（每个 Phase 的精确操作指导）

## 完成条件
当以下全部满足时，输出：
<promise>RXX COMPLETE</promise>

## 注意事项
```

## Playwright UI 验证（Step 6 必做）

每个涉及 UI 的 Round，Ralph 必须在完成 typecheck 后：

```bash
# dev server 已在 http://localhost:3001 运行
# 使用 Playwright MCP 工具：
# 1. browser_navigate → http://localhost:3001/notebook/[test-id]
# 2. browser_snapshot → 确认页面结构
# 3. browser_click → 与改动的组件交互
# 4. browser_console_messages → 检查 JS 错误
```

PROMPT.md 中须包含 Playwright 验证步骤，指定：
- 访问路径（具体到笔记本 ID，参考 playwright-baseline.md）
- 要验证的改动点（3条以内，精确描述）
- 预期结果

## code-simplifier 审计（Step 5）

Ralph 在完成编码（Step 4）后，立即调用：

```
使用 code-simplifier:code-simplifier agent 审查本次改动的文件：
- 关注：重复逻辑、不必要的 state、过长的 handler
- 不关注：已有代码风格（只看本次改动）
- 输出：是否有需要简化的地方，如有则修复后再继续
```

PROMPT.md 中须明确指定触发时机：**每个 Phase 编码完成后，typecheck 前**。

## 待处理记录更新规则

每个 Round 完成后，将 `docs/v2/待处理记录-v2.md` 中对应条目：
- 从「待处理问题」移入「已经处理问题」
- 格式：`·问题简述（RXX）\n  处理方式一句话`

## Ralph Loop 启动命令

```bash
cd /Users/cutealexander/Desktop/deving/deepnotion/old
/ralph-loop "$(cat docs/v2/rounds/rXX/PROMPT.md)" --max-iterations 12 --completion-promise "RXX COMPLETE"
```

## 进入下一 Round 的触发

Round 完成（PROGRESS.md 写入 + 待处理记录更新）后，询问用户是否规划下一个 Round，然后：
1. 读取 `待处理记录-v2.md` 确认下一优先问题
2. 读取相关代码文件
3. 创建 `docs/v2/rounds/rXX/task.md` + `PROMPT.md`
4. 提供启动命令

## 项目快速参考

| 条目 | 值 |
|------|-----|
| Dev Server | `http://localhost:3001` |
| 验证顺序 | `typecheck → test → Playwright → build` |
| Commit 格式 | `feat(RXX-PN): 简短描述` |
| 测试笔记本 | `collection-ddc35190-3da5-44ac-b69b-7249e8cbf20f` |
| 待处理记录 | `docs/v2/待处理记录-v2.md` |
| Playwright 基线 | `docs/v2/playwright-baseline.md` |
