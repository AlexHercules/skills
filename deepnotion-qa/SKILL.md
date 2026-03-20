---
name: deepnotion-qa
description: DeepNotion QA — 完整质量验证技能。在完成一批 round 后、发布前、或手动触发时使用。支持 /qa、/qa m2、/qa --full、/qa --no-fix 调用。
---

# DeepNotion QA — 完整验证流

## 调用方式

```
/qa              ← diff-aware，自动推断受影响模块
/qa m1           ← 只测输入中心
/qa m2           ← 只测笔记本
/qa m3           ← 只测知识库
/qa m4           ← 只测工作区
/qa --full       ← 全模块测试
/qa --no-fix     ← 只报告，不修复
```

## 前置检查

1. 确认 dev server 在线：`curl -s http://localhost:3001 > /dev/null` — 如果未启动，提示用户先运行 `npm run dev`
2. 确认 git working tree 状态（记录当前 branch 和最新 commit）
3. 读取 `git diff HEAD~1` 确定改动范围（用于 diff-aware 模式的模块推断）

## Phase 1 — 初始化 & 模块推断

**diff-aware 模式下的模块推断规则**：

| 改动文件路径 | 推断受影响模块 |
|---|---|
| `components/input*/`、`M1*/`、`import*/` | M1 |
| `components/notebook*/`、`editor*/`、`CodeMirror*/`、`ThreeColumn*/`、`Studio*/` | M2 |
| `components/knowledge*/`、`graph*/`、`vault*/`、`force-graph*/` | M3 |
| `components/workspace*/`、`agent*/`、`M4*/` | M4 |
| `lib/db*/`、`ipc*/`、`*.sqlite*/`、`backend*/`、`main/` | M2 + M3（存储层变更影响渲染） |
| `components/shared*/`、`hooks*/`、`store*/` | 全模块（共享层，跑 --full） |

推断完成后，列出将要测试的模块，**等待用户确认或修改**（不要直接开始）。

## Phase 2 — 模块测试

每个模块按以下通用流程执行，再加模块专属步骤：

**通用流程**：
```
1. browser_navigate → 模块入口 URL
2. browser_snapshot → 记录页面结构（检查是否白屏/布局崩溃）
3. 执行模块专属交互步骤
4. browser_console_messages → 收集所有 JS 错误和警告
5. 记录发现的问题（severity: critical/high/medium/low）
```

### M1 输入中心测试脚本

```
URL: http://localhost:3001/
入口: 导入/输入中心面板

测试步骤:
- browser_snapshot → 确认各输入端按钮渲染（公众号/网页/PDF/学术论文/文件/Obsidian）
- browser_click → 点击任意一个输入端按钮，确认面板响应
- browser_snapshot → 确认导入状态区域存在
- browser_console_messages → 收集错误
```

### M2 笔记本测试脚本

```
URL: http://localhost:3001/notebook/collection-ddc35190-3da5-44ac-b69b-7249e8cbf20f

测试步骤:
- browser_snapshot → 确认三栏布局渲染（左/中/右栏均可见）
- browser_click → 左栏点击一个源材料条目
- browser_snapshot → 确认中栏内容切换
- browser_click → 点击编辑器区域
- browser_key_press → 输入几个字符
- browser_snapshot → 确认 CodeMirror 编辑器响应输入
- browser_click → 右栏 Studio 面板开关按钮（如存在）
- browser_console_messages → 重点检查 CodeMirror extension 报错
```

### M3 知识库测试脚本

```
URL: http://localhost:3001/knowledge（或对应路径）

测试步骤:
- browser_snapshot → 确认图谱视图加载（react-force-graph 节点/边可见）
- browser_snapshot → 确认文件列表区域加载
- browser_console_messages → 重点检查 WebGL/canvas/force-graph 报错
```

### M4 工作区测试脚本

```
URL: http://localhost:3001/workspace（或对应路径）

测试步骤:
- browser_snapshot → 确认 Agent 入口渲染
- browser_click → 对话输入框
- browser_key_press → 输入测试文字
- browser_snapshot → 确认输入框响应
- browser_console_messages → 收集错误
```

## Phase 3 — Health Score 计算

测试完成后，按以下维度打分（满分 100）：

| 维度 | 权重 | 评判标准 |
|---|---|---|
| Console 错误 | 30% | 0个error=30分，每个error -5分，warning -1分 |
| 核心功能可用 | 30% | 编辑器可输入/图谱渲染/导入入口可点击 |
| UI 渲染完整 | 20% | 无白屏、无布局崩溃、无缺失组件 |
| 交互响应 | 20% | 点击/输入有反应，状态变化符合预期 |

**分数解读**：
- 90-100：可发布
- 70-89：有问题需关注，建议修复后再合并
- 50-69：存在明显 bug，必须修复
- <50：有 critical 问题，不可合并

## Phase 4 — 修复循环（`--no-fix` 时跳过）

针对 **critical** 和 **high** severity 问题执行修复：

```
1. 读源码定位根因
2. 最小改动修复
3. npm run typecheck → 确认类型安全
4. git add <改动文件> && git commit -m "fix(qa): [问题描述]"
5. 重新执行受影响模块的测试步骤，验证修复
6. 如有回归风险，在 Vitest 中补充回归测试
```

**medium/low** 问题：记录在报告中，不自动修复，由下一个 round 处理。

## Phase 5 — 报告输出

将报告写入 `docs-main/qa-reports/YYYY-MM-DD-RXX.md`（RXX 从当前 branch 或最新 commit 推断）：

```markdown
# QA 报告 — YYYY-MM-DD

**Branch**: xxx  **Commit**: xxxxxxx
**测试范围**: M2（diff-aware）/ 全模块（--full）

## Health Score: XX/100

| 维度 | 得分 | 说明 |
|---|---|---|
| Console 错误 | XX/30 | ... |
| 核心功能可用 | XX/30 | ... |
| UI 渲染完整 | XX/20 | ... |
| 交互响应 | XX/20 | ... |

## 发现的问题

### Critical
- [ ] [问题描述] — [模块] — [已修复/待处理]

### High
- [ ] ...

### Medium / Low
- [ ] ...（不自动修复）

## 修复记录
- fix(qa): [描述] — commit xxxxxxx

## 结论
[可发布 / 建议修复后合并 / 必须修复]
```

## 注意事项

- Electron 模式：所有验证在 `http://localhost:3001` 进行，不是纯浏览器环境
- 测试笔记本 ID：`collection-ddc35190-3da5-44ac-b69b-7249e8cbf20f`
- M3/M4 的入口路径若不确定，先用 `browser_navigate` + `browser_snapshot` 探索，不要假设
- 修复时只动最小范围，不做顺手重构
