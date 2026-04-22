# FastStack

**从想法到上线的全流程 vibecoding 工具包** · Claude Code 插件

FastStack 提供一组相互衔接的 Skills，帮你用 Claude Code 从一个模糊的点子，一路推进到可工作的代码：

```
fs-idea  →  fs-req  →  fs-prod  →  fs-ui  →  fs-tech  →  fs-tasks  →  fs-dev  →  fs-qa
想法梳理    需求文档    产品设计    UI设计    技术设计    任务拆解    开发执行    测试验收
```

任意阶段都可以作为起点；所有产出都放在 **可配置的文档目录** 下（默认 `.faststack/`），互相引用、可追溯。

## 安装

在 Claude Code 中运行：

```
/plugin marketplace add seancheung/faststack
/plugin install faststack
/reload-plugins
```

## 更新

有新版本发布时，在 Claude Code 中运行：

```
/plugin marketplace update faststack
/reload-plugins
```

## 使用

**第一次使用**：输入 `/fs-start`，它会引导你勾选要启用的 **features**（默认全不选，对应最精简流程）和 **文档目录**（默认 `.faststack/`），并在项目根生成 `.faststack.yml`：

```yaml
version: 1
docs_dir: .faststack
features:
  mvp_planning: false          # fs-idea：是否规划 MVP
  nfr: false                   # fs-req：是否收集非功能需求 / 约束
  business_analysis: false     # fs-prod：是否讨论产品价值 / 商业模式 / 成功指标
```

## Features

流程本身不受 features 影响（7 步 skill 链不变），开关只决定各 skill 的询问深度与产出详略。**默认全关 = 最精简**,适合个人小工具 / 原型验证;正式产品 / 对外发布 / 团队协作建议全开。

| Feature | 作用 | 影响 skill |
| --- | --- | --- |
| `mvp_planning` | 产出 MVP 规划(最小可行形态) | `fs-idea` |
| `nfr` | 收集非功能需求 + 约束与假设 | `fs-req` |
| `business_analysis` | 讨论产品价值 / 商业模式 / 成功指标,做商业视角的功能审查 | `fs-prod` |

`fs-ui` / `fs-tech` / `fs-tasks` / `fs-dev` / `fs-qa` / `fs-sync` 主流程不受 features 影响。测试与验收是 `fs-qa` 的独立职责,按需调用。

**之后**：`fs-start` 会根据 `docs_dir` 下已有文档推荐下一步；或直接调用任一 skill（`$DOCS` 代表你配置的目录）：

| Skill | 作用 | 产出 |
| --- | --- | --- |
| `fs-start` | 初始化配置 / 流程路由 | `.faststack.yml` |
| `fs-idea` | 交互式梳理想法 | `$DOCS/idea.md` |
| `fs-req` | 收集需求 | `$DOCS/requirements.md` |
| `fs-prod` | 产品功能设计 | `$DOCS/product.md` |
| `fs-ui` | 设计规范 + HTML 原型 | `$DOCS/design.md`, `$DOCS/prototype/*.html` |
| `fs-tech` | 技术方案 | `$DOCS/technical.md` |
| `fs-tasks` | 任务拆解 | `$DOCS/tasks.md` |
| `fs-dev` | 按任务清单开发;`fs-dev BUG-xxx` 修复 bug | 代码 + 任务打勾 + 回写 `$DOCS/bugs.md` |
| `fs-qa` | 生成 / 执行测试与验收;记录失败;重测修复闭环 | `$DOCS/qa.md`（测试计划 + 执行记录）+ `$DOCS/bugs.md` |
| `fs-sync` | 下游变更回流到上游 | `$DOCS/changelog.md` + 相关文档更新 |

## 变更管理

开发或评审中想法变了？**不要**直接去改当前文档。运行 `fs-sync`，它会：

1. 分析变更影响的 FR-xxx / P-xxx / T-xxx / TC-xxx / BUG-xxx
2. 从上游到下游逐份更新相关文档
3. 在 `$DOCS/changelog.md` 记录"何时、为何、影响什么"（CR-xxx）

每个非起点 skill（`fs-req` / `fs-prod` / `fs-ui` / `fs-tech` / `fs-tasks` / `fs-dev` / `fs-qa`）在检测到用户描述与上游文档冲突时，都会主动引导去 `fs-sync`，避免文档漂移。

## Bug 回流

`fs-qa` 在执行测试时发现失败,会在 `$DOCS/bugs.md` 记录 `BUG-xxx`（状态 `❌ 待修`），并在 `qa.md` 对应用例的 `关联 Bug` 字段写下引用。

修复闭环:

1. 运行 `fs-dev BUG-xxx`:fs-dev 只读 `bugs.md` 里该条,修完回写状态为 `✅ 已修待重测`（不读 `qa.md` 全文,也不自己标通过）
2. 运行 `fs-qa`:自动拾取 `✅ 已修待重测` 的 BUG,找对应 `TC-xxx` 重测
3. 重测通过 → BUG 整条挪到 `bugs.md` 的 `## 修复存档`;重测失败 → 状态回 `❌ 待修`

两个 skill 通过 `bugs.md` 沟通,用户不用在两边来回搬消息。

## 配置文件 `.faststack.yml`

| 字段 | 说明 | 默认 |
| --- | --- | --- |
| `version` | 配置文件版本，当前为 `1` | — |
| `docs_dir` | FastStack 文档根目录（相对项目根） | `.faststack` |
| `features.mvp_planning` | fs-idea 是否产出 MVP 规划 | `false` |
| `features.nfr` | fs-req 是否收集非功能需求 + 约束 | `false` |
| `features.business_analysis` | fs-prod 是否讨论商业性内容(产品价值 / 商业模式 / 成功指标) | `false` |

想换目录或启停某个 feature,改文件后下次运行任一 skill 即生效。未定义的 feature 字段按 `false` 处理。

## 项目结构

```
faststack/
├── .claude-plugin/
│   └── marketplace.json        # marketplace 定义
└── plugins/
    └── faststack/
        ├── .claude-plugin/
        │   └── plugin.json     # 插件清单
        └── skills/
            ├── fs-start/SKILL.md
            ├── fs-idea/SKILL.md
            ├── fs-req/SKILL.md
            ├── fs-prod/SKILL.md
            ├── fs-ui/SKILL.md
            ├── fs-tech/SKILL.md
            ├── fs-tasks/SKILL.md
            ├── fs-dev/SKILL.md      # 任务执行 + BUG-xxx 修复
            ├── fs-qa/SKILL.md       # 产出 qa.md + bugs.md,闭环重测
            └── fs-sync/SKILL.md
```

## 本地开发与测试

```bash
# 在本地仓库目录
/plugin marketplace add /path/to/faststack
/plugin install faststack@faststack
```

修改 SKILL.md 后，在 Claude Code 内重载即可生效。

## License

MIT
