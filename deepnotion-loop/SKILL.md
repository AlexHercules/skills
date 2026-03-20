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

## GitNexus 深度适配（Step 1 审查 + PROMPT.md 编写 必做）

在写 task.md / PROMPT.md **之前**，必须用 GitNexus 对涉及的代码区域做深度挖掘，确保提示词中的文件路径、函数名、调用链、数据流全部基于**代码实况**而非记忆或文档。

### 必做流程

```
1. READ gitnexus://repo/deepnotion/context          → 确认索引新鲜度
   ⚠️ 如果 stale → 先跑 `npx gitnexus analyze`，否则后续结果不可信

2. gitnexus_query({query: "本轮涉及的功能关键词"})    → 定位相关执行流 + 符号
   例：gitnexus_query({query: "notebook source material list"})

3. gitnexus_context({name: "关键符号"})              → 查看调用者/被调用者/所属流程
   对每个要改动的核心函数/组件，至少查一次 context

4. gitnexus_impact({target: "要改的符号", direction: "upstream"})
   → 确认 blast radius，d=1 的依赖必须写入 task.md「依赖文件」表

5. 读取源文件验证                                     → Read 实际文件，确认行号和签名
   GitNexus 给的是索引快照，PROMPT.md 中的行号必须来自 Read 工具
```

### 输出要求

GitNexus 挖掘的结果必须体现在 task.md 中：
- **依赖文件表**：补充 GitNexus 发现的上下游文件（不仅是你已知的）
- **Phase 改动文件列**：用 impact 分析确认的实际波及范围
- **代码片段**：从 Read 工具获取的当前代码（不是凭记忆写的）

### 禁止行为

| 禁止 | 原因 |
|------|------|
| ❌ 跳过 GitNexus 直接凭记忆写 PROMPT.md | 记忆中的路径/函数名/行号可能已过时，导致 Ralph 执行时找不到目标或改错位置 |
| ❌ 只用 Explore agent 不用 GitNexus | Explore 是全文搜索，缺少调用链和执行流信息；GitNexus 提供结构化的符号关系图 |
| ❌ GitNexus 索引 stale 时继续使用其结果 | 过期索引的符号/行号不可信，必须先 `npx gitnexus analyze` 刷新 |
| ❌ 把 GitNexus 的行号直接写入 PROMPT.md 而不经 Read 验证 | GitNexus 索引是快照，代码可能在索引后被改动，行号必须用 Read 工具二次确认 |
| ❌ 忽略 impact 分析中 d=1 的依赖 | d=1 意味着直接调用者/导入者，改动后**必定受影响**，不列入 task.md 会导致 Ralph 漏改 |
| ❌ 对不涉及代码改动的纯文档 Round 跑 GitNexus | 浪费时间；GitNexus 只在有代码改动的 Round 才有价值 |

### 8步循环中的嵌入点

```
Step 1 审查  → 用 GitNexus 做结构化代码挖掘（替代或补充 Explore agent）
Step 2 思考  → 基于 impact 分析评估改动风险
Step 3 设计  → 将 GitNexus 发现的调用链纳入改动范围设计
写 PROMPT.md → 所有文件路径/函数名/行号必须经过 GitNexus + Read 双重验证
```

---

## 交付物结构

每个 Round 必须在 `docs-main/rounds/rXX/` 创建三个文件：

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
**完整规格**: docs-main/rounds/rXX/task.md（执行前必读）

## 执行规则（6条固定规则）
## 进度检查（Phase 完成标志表）
## Phase 关键提示（每个 Phase 的精确操作指导）

## 最终交付（最后一个 Phase 完成后）
1. 在 `docs-main/rounds/rXX/PM.md` 核实产品说明文档与实际改动一致
2. 在 `docs-main/log/RXX.md` 写入迭代日志（日期、改动文件、解决的问题、验收结果）
3. 在 `docs-main/log/README.md` 末尾追加本轮条目（格式：`- [RXX](RXX.md) — 日期 — 标题`）
4. 更新 `docs-main/problems/PX.md` 末尾添加「## 修复记录」区块（仅涉及的 Problem）
5. 更新 `old/docs/architecture/MX-xxx/MX.X-xxx.md` 中的「当前状态」（仅涉及的模块，有架构变化才改）
6. git add + commit 所有文档
7. `rm -f .claude/ralph-loop.local.md`（清除 ralph-loop 状态文件，防止 stop hook 残留触发）

## 完成条件
当以下全部满足时，输出：
<promise>RXX COMPLETE</promise>

## 注意事项
```

## Step 6 验证（三段式，必做）

**重要：本项目运行在 Electron 模式，dev server 在 `http://localhost:3001`。**

### 6a. typecheck
```bash
npm run typecheck
```
失败则回 Step 4 修复，不得跳过。

### 6b. unit test
```bash
npm run test
```
失败则回 Step 4 修复，不得跳过。

### 6c. QA Lite（涉及 UI 的 Round 必做，纯后端/基础设施 Round 可跳过）

Ralph 执行以下结构化验证：

**Step 1 — 推断受影响组件**
读取本 Round 改动的文件路径，确定要验证的组件和入口 URL（参考 PROMPT.md 中的「验证目标」）。

**Step 2 — 页面加载确认**
```
browser_navigate → http://localhost:3001/[入口路径]
browser_snapshot → 确认无白屏、布局正常
```

**Step 3 — 改动点交互验证**
执行 PROMPT.md 中指定的 1-3 个交互动作，每步 `browser_snapshot` 确认状态变化。

**Step 4 — Console 扫描**
```
browser_console_messages → 收集所有 JS 错误和警告
```

**Step 5 — 输出 QA Lite 结果**
写入迭代日志「验收结果」区块：
```markdown
- ✅ QA lite：[组件名] 无 console 错误
# 或
- ⚠️ QA lite：发现 [N] 个 console 错误（已记录，待完整 /qa 处理）
  - [错误摘要1]
  - [错误摘要2]
```

发现 console 错误：**记录但不阻断 round**，标 ⚠️ 留给完整 `/qa` 处理。
发现白屏/布局崩溃/交互无响应：**阻断 round**，回 Step 4 修复。

---

**PROMPT.md 中 Step 6 验证目标写法**（替代原来的「Playwright 验证步骤」）：

```markdown
## Step 6 验证目标
- **入口 URL**: http://localhost:3001/notebook/collection-ddc35190-3da5-44ac-b69b-7249e8cbf20f
- **交互动作**:
  1. [具体动作，如「点击左栏第一个源材料」]
  2. [具体动作，如「在编辑器中输入文字」]
- **预期结果**: [每个动作的预期状态变化]
```

注意：检查 localStorage key 时先读代码确认实际 key 名。

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
- 将 PM.md 写入 `docs-main/rounds/rXX/PM.md`（Ralph 编码完成后由总工程师撰写或由 PROMPT.md 指示 Ralph 生成）
- PM.md + task.md + PROMPT.md 一起 git commit

### 8b. 项目级文档同步
根据本轮改动，检查并更新以下文档（有变化才改，没变化不动）：
- **MEMORY.md** — Round 进度表、测试覆盖进度、产品路线图状态
- **CLAUDE.md** — 如果新增了架构模式、约定、关键路径（一般不需要改）
- **产品路线图**（MEMORY.md 中）— 里程碑完成状态

### 8c. 迭代日志（必做）
在 `docs-main/log/RXX.md` 写入本轮迭代日志，并在 `docs-main/log/README.md` 末尾追加该轮条目：

```markdown
- [RXX](RXX.md) — YYYY-MM-DD — 一句话标题（关联 Problem）
```

迭代日志 RXX.md 格式：

```markdown
# RXX 迭代日志

**日期**: YYYY-MM-DD
**关联 Problem**: PX（问题标题）

## 做了什么
一句话摘要。

## 改动文件
- `path/to/file.tsx` — 改动原因
- ...

## 解决的问题
- [断裂点 / 功能点名称]：修复描述

## 验收结果
- ✅ typecheck 通过
- ✅ Playwright 验证：[简述验证结果]
```

### 8d. Problems 和 Architecture 文档同步（必做，有变化才改）
- **problems/**: 如果本轮修复了某 Problem 中的某个断裂点或功能项，在该 `docs-main/problems/PX.md` 末尾追加「## 修复记录」区块，注明轮次和状态
- **architecture/**: 如果本轮改动了某模块的架构（职责边界/数据流/接口契约/存储结构），更新 `old/docs/architecture/MX-xxx/MX.X-xxx.md` 中的「当前状态」描述。纯 bug fix 或代码清理不动。

### 8e. git commit 范围
```bash
# 代码 + 测试
git add <改动的源码文件> <测试文件>
git commit -m "feat/test(RXX-PN): 功能描述"

# 文档（单独或合并提交均可）
git add docs-main/rounds/rXX/ docs-main/log/RXX.md
git add docs-main/problems/PX.md old/docs/architecture/MX-xxx/  # 有变化才加
git commit -m "docs(RXX): 交付文档 + 迭代日志 + 记录更新"
```

## Ralph Loop 启动命令（单轮，在 Claude Code 内）

```bash
/ralph-loop "$(cat docs-main/rounds/rXX/PROMPT.md)" --max-iterations 12 --completion-promise "RXX COMPLETE"
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

Round 完成后，询问用户是否规划下一个 Round，然后：
1. 读取 `docs-main/problems/` 确认下一优先问题
2. 读取相关代码文件
3. 创建 `docs-main/rounds/rXX/task.md` + `PROMPT.md` + `PM.md`
4. 提供启动命令

## 项目快速参考

| 条目 | 值 |
|------|-----|
| Dev Server | `http://localhost:3001` |
| 验证顺序 | `typecheck → test → Playwright → build` |
| Commit 格式 | `feat(RXX-PN): 简短描述` |
| 测试笔记本 | `collection-ddc35190-3da5-44ac-b69b-7249e8cbf20f` |

### 文档体系结构

```
old/docs/                           ← 描述当前状态（只读参考 + 轮次后更新）
├── codebase-guide/                 ← 代码层：文件/函数级说明（单独专项任务更新）
│   ├── 00-PROJECT-OVERVIEW.md
│   ├── 01-DATA-LAYER.md
│   └── ...
├── architecture/                   ← 系统层：模块职责/数据流/接口契约（轮次后更新）
│   ├── core/                       ← 核心架构（存储层/IPC/布局/统一节点）
│   ├── M1-input-center/            ← M1.1~M1.7 + ui-spec
│   ├── M2-notebook/                ← M2.1~M2.6 + ui-spec
│   ├── M3-knowledge-base/          ← M3.1~M3.3
│   ├── M4-workspace/               ← M4.1~M4.3
│   ├── M5-shared/
│   └── M6-settings/
└── references/                     ← 竞品/工具参考材料

docs-main/                          ← 规划/过程/问题（活跃更新）
├── INDEX.md                        ← 总索引
├── plans/                          ← 技术路线图（V5 等）
├── design/                         ← UI/设计规格（给设计师）
├── prompt/                         ← 大型非轮次任务的提示词存储
├── audit/                          ← 模块审计文档（M1~M6）
├── problems/
│   ├── P1~P11.md                   ← 问题文档，修复后追加修复记录
│   └── already/                    ← 已解决的问题归档
├── rounds/
│   └── rXX/
│       ├── PM.md                   ← 产品说明（用户视角）
│       ├── task.md                 ← 技术规格（Phase 清单 + 代码指导）
│       └── PROMPT.md               ← Ralph Loop 执行 prompt
└── log/
    └── RXX.md                      ← 迭代日志（每轮完成后写入）
```
