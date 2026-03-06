---
name: codex-generator
description: "Use when the user wants to generate an Obsidian-format code knowledge base from a GitNexus-indexed project. Produces multi-layered, interlinked documentation with module overviews, file docs, function/class descriptions, and call relationships. Examples: \"Generate code docs\", \"Create Obsidian codex\", \"Build code knowledge base\""
---

# Codex Generator — Obsidian 代码知识库生成器

## When to Use

- 用户想对一个项目生成可浏览的代码文档
- 需要用 Obsidian 双向链接梳理代码结构
- 想快速理解一个陌生代码库的全貌

## Prerequisites

- 目标项目已通过 `npx gitnexus analyze` 建立索引
- GitNexus MCP 服务可用
- 确认索引未过期（`gitnexus://repo/{name}/context`）

## Output Structure

```
{项目名}-codex/
├── INDEX.md                        # 总概览 + MOC
├── README.md                       # 项目原始 README 直接复制（保持原文）
├── {module-a}/                     # 模块文件夹（cluster 名，kebab-case）
│   ├── _overview.md                # 模块概览
│   ├── {sourceFileName}.md         # 文件级文档（含函数、类描述）
│   └── ...
├── {module-b}/
│   └── ...
└── ...
```

## Execution — Two Phases

### Phase 1: Skeleton (骨架生成)

```
- [ ] READ gitnexus://repo/{name}/context           → 确认索引可用
- [ ] READ 项目根目录 README.md                      → 复制为 codex/README.md（原文不改动）
- [ ] READ gitnexus://repo/{name}/clusters           → 获取模块划分
- [ ] READ gitnexus://repo/{name}/processes           → 获取执行流程
- [ ] 对每个 cluster 读取成员列表                       → 确定文件归属
- [ ] 生成 INDEX.md + 各模块 _overview.md（骨架内容）
- [ ] 呈现给用户确认：
      - 模块划分是否合理？需要合并/拆分/重命名？
      - 有遗漏的文件？
- [ ] 用户确认后进入 Phase 2
```

> Phase 1 只用 GitNexus 资源读取，不读源文件，保持轻量快速。

### Phase 2: Fill (填充生成)

```
- [ ] 逐模块生成文件级文档
- [ ] 对每个源文件：
      a. gitnexus_context({name: 关键符号}) → 获取调用关系
      b. Read 源文件 → 提取类、函数结构
      c. 生成 .md 文档（描述 + 调用链接 + 必要代码片段）
- [ ] 使用并行 Agent 加速（各模块可独立生成）
- [ ] GitNexus 未覆盖的文件（配置文件等）直接读源文件生成描述
- [ ] 生成完毕输出统计摘要
```

## Obsidian Format Rules (格式规范)

生成的所有 `.md` 文件必须严格遵循 Obsidian 兼容格式：

### Wikilinks (双向链接)

- 使用 Obsidian wikilink 语法 `[[]]`，**不要**使用标准 Markdown 链接 `[]()`
- 文件引用：`[[authService]]`
- 带显示文本：`[[auth/_overview|auth 模块]]`
- 锚点跳转到 heading：`[[authService#validateToken（校验令牌）]]`
- 跨文件夹引用需要包含相对路径：`[[auth/authService]]`

### Headings (标题层级)

- `#` 仅用于文档标题（每个文件只有一个）
- `##` 用于大分类（概述、类、函数、核心流程等）
- `###` 用于具体的类名/函数名
- `####` 用于属性、参数等子级
- 标题中包含英文名 + 中文注释：`### AuthService（认证服务）`

### Code Blocks (代码块)

- 必须使用三反引号围栏式代码块，并标注语言：
  ````
  ```typescript
  function validate(token: string): boolean {
    return jwt.verify(token, secret);
  }
  ```
  ````
- **不要**使用缩进式代码块（Obsidian 中渲染不一致）
- 行内代码用单反引号：`` `validateToken` ``

### Callouts (提示框)

在需要强调的地方使用 Obsidian callout 语法：

```markdown
> [!info] 技术要点
> 该模块使用 JWT 进行无状态认证

> [!warning] 注意
> 此函数不做参数校验，调用方需自行确保入参合法

> [!tip] 设计模式
> 采用策略模式实现多种支付方式的切换
```

可用类型：`info`、`warning`、`tip`、`note`、`important`

### Tags (标签)

- 在 frontmatter 的 `tags` 字段中使用数组格式
- 标签使用中文，与功能语义对齐
- 不要在正文中使用 `#标签` 语法（避免与 heading 冲突）

### Tables (表格)

- 使用标准 Markdown 表格语法（Obsidian 完全支持）
- 表格中的引用同样使用 wikilink：`| [[authService]] | 认证核心逻辑 |`

### Embeds (嵌入)

- 不使用 `![[]]` 嵌入语法，避免文档间内容重复
- 保持每个文档的独立完整性，通过链接而非嵌入关联

### Special Characters (特殊字符)

- 文件名避免使用 `/ \ : * ? " < > |`
- 如果源文件名包含特殊字符，替换为 `-`
- 中文和英文之间不需要手动加空格（Obsidian 渲染时自动处理）

## Document Conventions

### Language (语言规范)

- 结构和描述使用中文
- 代码名称（函数名、类名、模块名、属性名）保持英文原样
- 所有英文命名旁边加括号中文注释：`### validateToken（校验令牌）`
- **模块名同样适用**：INDEX.md 模块表格的显示文本、`_overview.md` 标题，均须带中文，如 `[[auth/_overview|auth（认证）]]`

### Frontmatter (元数据规范)

每个 .md 文件必须包含 YAML frontmatter，支持 Dataview 查询：

```yaml
---
type: index | module | file          # 文档类型
source: src/auth/authService.ts       # 源文件路径（file 类型必填）
module: auth                          # 所属模块（module/file 类型必填）
tags: [认证, middleware]              # 功能标签
generated: 2026-03-05                 # 生成日期
---
```

### Linking (链接规范)

- 引用文件文档：`[[authService]]`
- 引用具体函数/类：`[[authService#validateToken（校验令牌）]]`
- 引用模块概览：`[[auth/_overview|auth 模块]]`
- 所有出现的类名、函数名如果有对应文档，必须包裹为链接

### Code Snippets (代码片段)

- 仅在必要时补充关键代码片段（核心算法、复杂逻辑、重要配置）
- 代码片段要精简，不超过 20 行
- 用代码块标注语言类型

## Templates

### INDEX.md

```markdown
---
type: index
project: {项目名}
generated: {日期}
---

# {项目名} 代码知识库

## 项目概述
<!-- 2-3 句话概括项目功能和技术栈 -->

## 模块一览
| 模块 | 职责 | 关键文件数 |
|------|------|-----------|
| [[{module}/_overview\|{module}（{中文模块名}）]] | {职责描述} | {数量} |

## 核心流程
- [[{module}/_overview#{流程名}|{流程名}]]
```

### _overview.md (模块概览)

```markdown
---
type: module
module: {模块名}
source: {源目录路径}
tags: [{标签}]
generated: {日期}
---

# {模块名}（{中文注释}）

## 职责
<!-- 模块的核心功能，3-5 句 -->

## 关键技术
<!-- 用到的框架、库、设计模式 -->

## 目录结构

<!-- 当本文件夹内有多个源文件合并写入同一 MD 时，必须包含此节，展示原始目录树 -->

```
{源目录路径}/
├── {fileA.ts}      # {一句话说明}
├── {fileB.ts}      # {一句话说明}
└── {subDir}/
    └── {fileC.ts}  # {一句话说明}
```

## 核心流程
<!-- 从 GitNexus processes 提取，简述步骤 -->

## 文件清单
| 文件 | 职责 |
|------|------|
| [[{fileName}]] | {职责描述} |
```

### {fileName}.md (文件级文档)

```markdown
---
type: file
source: {完整源文件路径}
module: {模块名}
tags: [{标签}]
generated: {日期}
---

# {fileName}（{中文注释}）

## 概述
<!-- 该文件整体职责，2-3 句 -->

## 类

### {ClassName}（{中文注释}）
<!-- 类的职责描述 -->

**关键属性：**
- `{propName}`（{中文注释}）— {描述}，参见 [[{相关文件}]]

## 函数

### {functionName}（{中文注释}）
<!-- 功能描述，入参出参的语义说明 -->

调用关系：被 [[{caller文件}#{caller函数}]] 调用，内部调用 [[{callee文件}#{callee函数}]]

<!-- 必要时补充关键代码 -->
```

## File Naming Rules

- 模块文件夹：GitNexus cluster 名称，转 kebab-case
- 文件级文档：源文件名去扩展名（`authService.ts` → `authService.md`）
- 同名文件消歧：加路径前缀（`api-authService.md`）

## Key Principles

1. **骨架先行** — 先出结构让用户确认，再批量生成细节
2. **链接优先** — 每一个提及的类/函数/文件都要是可点击的 Obsidian 链接
3. **精准描述** — 用人类可读的语言描述功能，不是复制代码
4. **中文注释** — 所有英文命名都附带括号中文翻译
5. **必要代码** — 仅在核心逻辑处补充精简代码片段
6. **Dataview 友好** — frontmatter 规范统一，支持后续查询扩展
7. **目录结构可见** — 当一个文件夹内的多个源文件被合并进同一个 MD（无论是 `_overview.md` 还是合并文件文档），必须包含 `## 目录结构` 一节，用目录树展示该文件夹的原始文件布局，让读者无需离开文档即可了解源码组织
