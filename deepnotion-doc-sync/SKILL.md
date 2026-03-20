---
name: deepnotion-doc-sync
description: Use when syncing DeepNotion documentation after one or more rounds complete — updates log/, old/docs/architecture/, and problems/ based on code changes. Standalone doc-sync session, NOT per-round automation.
---

# DeepNotion 文档同步 — 独立专项工作流

## 定位

**独立触发**：完成若干轮迭代后，专门将代码变化同步回文档体系。不是轮次内的子任务，是独立的文档维护专项。

典型触发场景：
- "跑完 R139-R142，帮我更新文档"
- "做一次文档同步"
- "把最近几轮的改动同步到 architecture"

---

## 文档两层架构

| 层级 | 位置 | 描述什么 | 本 skill 覆盖？ |
|------|------|---------|----------------|
| 系统层 | `old/docs/architecture/` | 模块职责、数据流、接口契约、存储结构 | ✅ |
| 问题层 | `docs-main/problems/` | 已知问题状态与修复记录 | ✅ |
| 日志层 | `docs-main/log/` | 每轮迭代记录 | ✅ |
| 代码层 | `old/docs/codebase-guide/` | 文件/函数级说明 | ❌ 单独专项 |

> `codebase-guide/` 的更新是**单独专项任务**，本 skill 结束时标注待同步项，不执行。

---

## Step 1 — 确定同步范围

```bash
# 查看最近完成的轮次
ls docs-main/rounds/phase0/   # 或 v4/ v5/ 等当前版本目录

# 查看日志已更新到哪一轮
ls docs-main/log/

# 确认哪些轮次代码已完成
git log --oneline -20
```

明确：哪些轮次（RXX）已完成代码但文档尚未同步？

---

## Step 2 — 逐轮读取改动

对每个待同步轮次：
1. 读取 `docs-main/rounds/rXX/task.md` — 了解技术改动范围
2. 读取 `docs-main/rounds/rXX/PM.md` — 确认功能层面影响
3. 必要时用 Explore agent 验证代码实际状态

---

## Step 3 — 更新 log/（必做）

如果该轮没有 `docs-main/log/RXX.md`，写入：

```markdown
# RXX 迭代日志

**日期**: YYYY-MM-DD
**关联 Problem**: PX（问题标题，无则写"无"）

## 做了什么
一句话摘要。

## 改动文件
- `path/to/file.ts` — 改动原因

## 解决的问题
- [断裂点名称]：修复描述（无则写"无"）

## 验收结果
- ✅ typecheck 通过
- ✅ 构建通过
```

同时在 `docs-main/log/README.md` 末尾追加：
```
- [RXX](RXX.md) — YYYY-MM-DD — 一句话标题
```

---

## Step 4 — 更新 architecture/（有架构变化才做）

**判断标准**：本轮是否改变了模块的职责边界、数据流、接口契约、存储结构？

| 本轮类型 | 是否更新 architecture/ |
|---------|----------------------|
| 存储结构变化、新接口、模块重组 | ✅ 必须更新 |
| Bug fix、代码清理、性能优化 | ❌ 不动 |
| 新增 UI 但数据流不变 | ❌ 不动 |

找到对应文件 `old/docs/architecture/MX-xxx/MX.X-xxx.md`，更新：
- "当前状态"描述
- 数据流（如有变化）
- 接口/IPC 清单（如有变化）

> 有疑问时**以代码为准**，不以旧文档为准。

---

## Step 5 — 更新 problems/（有修复才做）

如果本轮修复了某 Problem 的断裂点，在对应 `docs-main/problems/PX.md` 末尾追加：

```markdown
## 修复记录

### RXX（YYYY-MM-DD）
- **修复内容**：[具体修复了哪个断裂点]
- **状态变化**：⚠️ 部分修复 → ✅ 已解决（或保持 ⚠️ 并说明残余问题）
```

如果 Problem 已完全解决，将文件移动到 `docs-main/problems/already/`。

---

## Step 6 — git commit

```bash
git add docs-main/log/
git add old/docs/architecture/   # 有变化才加
git add docs-main/problems/      # 有变化才加
git commit -m "docs: 同步 R{起}-R{止} 文档 — log + architecture + problems"
```

---

## Step 7 — 标注 codebase-guide 待同步项

如果 architecture/ 有更新，在会话末尾列出哪些 codebase-guide 文件需要跟进：

```
⚠️ codebase-guide/ 待同步（需单独专项任务）：
- old/docs/codebase-guide/01-DATA-LAYER.md — 受 RXX 影响（存储层变化）
- old/docs/codebase-guide/04-KNOWLEDGE-GRAPH.md — 受 RXX 影响（graph_nodes 改动）
```

---

## 注意事项

- architecture/ 描述**系统设计意图**，不逐行描述代码实现
- 不修改 `rounds/rXX/` 内已有的 task.md / PROMPT.md（历史记录，只读）
- 多轮批量同步时，按轮次顺序逐轮处理，不跳跃
