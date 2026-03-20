---
name: codex-generator
description: "Use when the user wants to generate an Obsidian-format code knowledge base from any project. No external indexing tools required — uses Claude Code's native tools (Glob, Grep, Read, Agent) to analyze codebase structure, detect modules, and trace call relationships. Produces multi-layered, interlinked documentation. Examples: \"Generate code docs\", \"Create Obsidian codex\", \"Build code knowledge base\""
---

# Codex Generator — Obsidian 代码知识库生成器

## When to Use

- 用户想对一个项目生成可浏览的代码文档
- 需要用 Obsidian 双向链接梳理代码结构
- 想快速理解一个陌生代码库的全貌

## How It Works — 它到底在做什么？（核心功能通俗解读）

想象你接手了一个别人写的项目，几十上百个文件，不知道从哪里看起。Codex Generator 就是帮你**自动生成一份可浏览的代码地图**，放进 Obsidian 后可以像维基百科一样点击跳转。

### 核心做了三件事

1. **扫描分析** — Claude 像一个新入职的工程师一样，先通读项目结构：哪些文件夹是干什么的、谁引用了谁、哪个文件最核心（被引用最多）。它不需要任何外部工具，纯靠读文件和搜索来理解代码。

2. **搭骨架** — 把分析结果整理成模块划分方案，先给你看一眼：「我打算把代码分成这几个模块，核心流程是这样的」。你觉得不对可以让它调整，确认后才往下走。

3. **填内容** — 对每个源文件生成一份文档，描述它有哪些类和函数、每个函数是干什么的、谁调用了它、它又调用了谁。所有提到的名字都是 Obsidian 双向链接，点击就能跳转过去。

### 实际使用场景

- **接手陌生项目**：快速生成全貌文档，不用一个个文件翻
- **团队知识沉淀**：把代码结构固化成可搜索的知识库
- **代码审查辅助**：在文档里追踪调用链，理解改动影响范围
- **架构梳理**：发现模块间的依赖关系，识别过度耦合

### 为什么分三步走？

直接生成所有文档会有两个问题：一是模块划分可能不合理（比如把不相关的文件归到一起），二是大项目一次性生成太慢且容易出错。所以采用「先扫描 → 确认骨架 → 再填充」的流程——**第一步保证理解正确，第二步保证结构合理，第三步才批量产出**。每一步都可以并行处理多个模块，大项目也能高效完成。

## Prerequisites

- 目标项目为本地可读的代码仓库
- Claude Code 可用（需要 Glob、Grep、Read、Agent 工具）
- 无需任何外部索引工具

## Output Location（输出路径）

输出路径按以下优先级自动决定：

1. **用户显式指定** — 如果用户给了输出路径，直接使用
2. **项目已有 `docs/` 目录** — 在 `docs/` 下创建 `codex/` 子文件夹（`docs/codex/`）
3. **项目没有 `docs/`** — 在项目根目录创建 `{项目名}-codex/`

> Phase 0 扫描时检测 `docs/` 目录是否存在，并在骨架确认阶段向用户展示最终输出路径。

## Output Structure（输出结构）

```
{输出路径}/
├── INDEX.md                        # Index（总概览 + MOC）
├── README.md                       # README（项目原始 README 直接复制，保持原文）
├── {module-a}/                     # Module Folder（模块文件夹，按目录/功能聚类，kebab-case）
│   ├── _overview.md                # Overview（模块概览）
│   ├── {sourceFileName}.md         # File Doc（文件级文档，含函数、类描述）
│   └── ...
├── {module-b}/
│   └── ...
└── ...
```

## Execution — Three Phases

### Phase 0: Scan (代码扫描与自分析)

> 替代外部索引工具，使用 Claude Code 原生能力完成代码分析。

```
- [ ] 确认目标项目路径（用户指定或当前工作目录）
- [ ] 确定输出路径：
      a. 检查项目根目录是否存在 `docs/` 文件夹
      b. 若存在 → 输出到 `docs/codex/`
      c. 若不存在 → 输出到 `{项目名}-codex/`
      d. 用户显式指定路径时覆盖以上逻辑
- [ ] Glob 扫描项目结构：
      a. `**/*.{ts,js,tsx,jsx,py,go,rs,java,rb,swift,kt,c,cpp,h,hpp,cs,vue,svelte}`
         （根据项目实际语言调整）
      b. 排除：node_modules, dist, build, .git, vendor, __pycache__, .next, coverage
      c. 记录文件总数和目录结构
- [ ] 读取项目配置文件识别技术栈：
      a. package.json / go.mod / Cargo.toml / pyproject.toml / pom.xml 等
      b. tsconfig.json / .eslintrc / Dockerfile 等
      c. 提取项目名称、依赖、构建命令
- [ ] 分析目录结构，按顶层目录 + 功能聚类划分模块：
      a. 优先使用 src/ 下的一级子目录作为模块边界
      b. 如果没有 src/，使用项目根下的功能目录
      c. 单文件或杂散文件归入 "core" 或 "utils" 模块
- [ ] 对每个模块，Grep 扫描 import/require/use 语句：
      a. 建立文件间依赖图（谁导入了谁）
      b. 识别核心文件（被多处引用的文件）
      c. 识别入口文件（main, index, app 等）
- [ ] 对关键文件（入口、核心）快速 Read 提取：
      a. export 的类名、函数名
      b. 主要接口/类型定义
      c. 关键常量和配置
- [ ] 输出扫描摘要：
      - 项目名 + 技术栈
      - 文件总数 / 代码文件数
      - 模块划分方案（模块名 → 包含的文件列表）
      - 核心文件排名（按被引用次数）
```

> **性能提示：** 使用并行 Agent 同时扫描不同目录。对于大型项目（>200 文件），优先分析 src/ 目录，测试文件仅记录存在不深入分析。

### Phase 1: Skeleton (骨架生成)

```
- [ ] READ 项目根目录 README.md → 复制为 codex/README.md（原文不改动）
- [ ] 基于 Phase 0 的模块划分生成骨架：
      a. 生成 INDEX.md（项目概览 + 模块表格 + 核心流程）
      b. 为每个模块生成 _overview.md（骨架内容：职责、文件清单）
- [ ] 识别核心流程：
      a. 从入口文件追踪调用链（Read 入口文件 → 找到调用的模块 → 串联流程）
      b. 典型流程：请求处理流程、数据流、初始化流程等
      c. 记录在 INDEX.md 的「核心流程」一节
- [ ] 呈现给用户确认：
      - 模块划分是否合理？需要合并/拆分/重命名？
      - 有遗漏的文件？
      - 核心流程是否准确？
- [ ] 用户确认后进入 Phase 2
```

### Phase 2: Fill (填充生成)

```
- [ ] 逐模块生成文件级文档
- [ ] 对每个源文件：
      a. Read 源文件 → 提取类、函数、接口、类型定义
      b. Grep 查找该文件的导出符号在其他文件中的引用 → 构建调用关系
      c. 生成 .md 文档（描述 + 调用链接 + 必要代码片段）
- [ ] 使用并行 Agent 加速（各模块可独立生成）
- [ ] 配置文件、脚本等非代码文件直接读取生成简要描述
- [ ] 调用关系追踪策略：
      a. 对每个 export 的函数/类名，Grep 搜索项目中的 import 语句
      b. 记录「被谁调用」和「调用了谁」
      c. 如果调用链过深（>3 层），只记录直接调用关系
- [ ] 生成完毕输出统计摘要：
      - 总文件数 / 生成文档数
      - 模块数 / 链接数
      - 未覆盖文件列表（如有）
```

## Obsidian Format Rules（格式规范）

生成的所有 `.md` 文件必须严格遵循 Obsidian 兼容格式：

### Wikilinks（双向链接）

- 使用 Obsidian wikilink 语法 `[[]]`，**不要**使用标准 Markdown 链接 `[]()`
- 文件引用：`[[authService]]`
- 带显示文本：`[[auth/_overview|auth 模块]]`
- 锚点跳转到 heading：`[[authService#validateToken（校验令牌）]]`
- 跨文件夹引用需要包含相对路径：`[[auth/authService]]`

### Headings（标题层级）

- `#` 仅用于文档标题（每个文件只有一个）
- `##` 用于大分类（概述、类、函数、核心流程等）
- `###` 用于具体的类名/函数名
- `####` 用于属性、参数等子级
- 标题中包含英文名 + 中文注释：`### AuthService（认证服务）`

### Code Blocks（代码块）

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

### Callouts（提示框）

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

### Tags（标签）

- 在 frontmatter 的 `tags` 字段中使用数组格式
- 标签使用中文，与功能语义对齐
- 不要在正文中使用 `#标签` 语法（避免与 heading 冲突）

### Tables（表格）

- 使用标准 Markdown 表格语法（Obsidian 完全支持）
- 表格中的引用同样使用 wikilink：`| [[authService]] | 认证核心逻辑 |`

### Embeds（嵌入）

- 不使用 `![[]]` 嵌入语法，避免文档间内容重复
- 保持每个文档的独立完整性，通过链接而非嵌入关联

### Special Characters（特殊字符）

- 文件名避免使用 `/ \ : * ? " < > |`
- 如果源文件名包含特殊字符，替换为 `-`
- 中文和英文之间不需要手动加空格（Obsidian 渲染时自动处理）

## Document Conventions

### Language（语言规范）

- 结构和描述使用中文
- 代码名称（函数名、类名、模块名、属性名）保持英文原样
- 所有英文命名旁边加括号中文注释：`### validateToken（校验令牌）`
- **模块名同样适用**：INDEX.md 模块表格的显示文本、`_overview.md` 标题，均须带中文，如 `[[auth/_overview|auth（认证）]]`

### Frontmatter（元数据规范）

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

### Linking（链接规范）

- 引用文件文档：`[[authService]]`
- 引用具体函数/类：`[[authService#validateToken（校验令牌）]]`
- 引用模块概览：`[[auth/_overview|auth 模块]]`
- 所有出现的类名、函数名如果有对应文档，必须包裹为链接

### Code Snippets（代码片段）

- 仅在必要时补充关键代码片段（核心算法、复杂逻辑、重要配置）
- 代码片段要精简，不超过 20 行
- 用代码块标注语言类型

## Templates

### INDEX.md（总索引）

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

### _overview.md（模块概览）

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
<!-- 从入口文件追踪调用链，简述步骤 -->

## 文件清单
| 文件 | 职责 |
|------|------|
| [[{fileName}]] | {职责描述} |
```

### {fileName}.md（文件级文档）

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

## File Naming Rules（文件命名规则）

- 模块文件夹 Module Folders：按源码目录结构或功能聚类命名，转 kebab-case
- 文件级文档 File Docs：源文件名去扩展名（`authService.ts` → `authService.md`）
- 同名文件消歧 Disambiguation：加路径前缀（`api-authService.md`）
- **文件名中文标注**：文档标题（`#` 行）必须带英文原名 + 中文注释，如 `# authService（认证服务）`；文件名本身保持英文，中文仅体现在文档内标题和链接显示文本中

## Scan Strategy by Language（各语言扫描策略）

### TypeScript / JavaScript
- 入口检测：`package.json` 的 `main`/`module` 字段，或 `src/index.ts`
- 模块边界：`src/` 下一级子目录
- 导入追踪：`import ... from` / `require()`
- 导出检测：`export function` / `export class` / `export default` / `module.exports`

### Python
- 入口检测：`__main__.py` / `main.py` / `app.py`
- 模块边界：含 `__init__.py` 的目录
- 导入追踪：`from ... import` / `import ...`
- 导出检测：`__all__` 列表，或模块顶层定义

### Go
- 入口检测：`func main()` 所在文件
- 模块边界：Go package（按目录）
- 导入追踪：`import` 块
- 导出检测：大写开头的标识符

### Rust
- 入口检测：`fn main()` / `lib.rs`
- 模块边界：`mod` 声明 + 目录
- 导入追踪：`use` 语句
- 导出检测：`pub fn` / `pub struct` / `pub enum`

### Java / Kotlin
- 入口检测：含 `public static void main` 的类
- 模块边界：Java package（按目录层级）
- 导入追踪：`import` 语句
- 导出检测：`public` 修饰的类/方法

### Other Languages
- 通用策略：按目录结构划分模块
- 读取每个文件提取函数/类定义
- 通过 Grep 搜索文件名/函数名追踪引用关系

## Key Principles

1. **零依赖** — 不需要任何外部索引工具，仅使用 Claude Code 原生能力
2. **骨架先行** — 先出结构让用户确认，再批量生成细节
3. **链接优先** — 每一个提及的类/函数/文件都要是可点击的 Obsidian 链接
4. **精准描述** — 用人类可读的语言描述功能，不是复制代码
5. **中文注释** — 所有英文命名都附带括号中文翻译
6. **必要代码** — 仅在核心逻辑处补充精简代码片段
7. **Dataview 友好** — frontmatter 规范统一，支持后续查询扩展
8. **目录结构可见** — 当一个文件夹内的多个源文件被合并进同一个 MD（无论是 `_overview.md` 还是合并文件文档），必须包含 `## 目录结构` 一节，用目录树展示该文件夹的原始文件布局
9. **并行加速** — 模块间无依赖的分析和生成任务使用并行 Agent 执行
