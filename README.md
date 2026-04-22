# FastStack

**从想法到上线的全流程 vibecoding 工具包** · Claude Code 插件

FastStack 提供一组相互衔接的 Skills，帮你用 Claude Code 从一个模糊的点子，一路推进到可工作的代码：

```
fs-idea  →  fs-req  →  fs-prod  →  fs-ui  →  fs-tech  →  fs-tasks  →  fs-dev
想法梳理    需求文档    产品设计    UI设计    技术设计    任务拆解    开发执行
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
  acceptance_criteria: false   # fs-req & fs-tasks：是否写验收标准
  business_analysis: false       # fs-prod：是否讨论产品价值 / 商业模式 / 成功指标
  tests: false                 # fs-tasks & fs-dev：是否拆测试任务、写跑自动化测试
```

## Features

流程本身不受 features 影响（7 步 skill 链不变），开关只决定各 skill 的询问深度与产出详略。**默认全关 = 最精简**,适合个人小工具 / 原型验证;正式产品 / 对外发布 / 团队协作建议全开。

| Feature | 作用 | 影响 skill |
| --- | --- | --- |
| `mvp_planning` | 产出 MVP 规划(最小可行形态) | `fs-idea` |
| `nfr` | 收集非功能需求 + 约束与假设 | `fs-req` |
| `acceptance_criteria` | 每条 P0 需求 / 每个任务写验收标准 | `fs-req`, `fs-tasks` |
| `business_analysis` | 讨论产品价值 / 商业模式 / 成功指标,做商业视角的功能审查 | `fs-prod` |
| `tests` | 任务清单拆测试任务;开发时写 / 跑自动化测试 | `fs-tasks`, `fs-dev`, `fs-sync` |

`fs-ui` / `fs-tech` / `fs-sync` 主流程不受 features 影响(`fs-sync` 仅在更新任务清单时参考 `tests`)。

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
| `fs-dev` | 按任务清单开发 | 代码 + 任务打勾 |
| `fs-sync` | 下游变更回流到上游 | `$DOCS/changelog.md` + 相关文档更新 |

## 变更管理

开发或评审中想法变了？**不要**直接去改当前文档。运行 `fs-sync`，它会：

1. 分析变更影响的 FR-xxx / P-xxx / T-xxx
2. 从上游到下游逐份更新相关文档
3. 在 `$DOCS/changelog.md` 记录"何时、为何、影响什么"（CR-xxx）

每个非起点 skill（`fs-req` / `fs-prod` / `fs-ui` / `fs-tech` / `fs-tasks` / `fs-dev`）在检测到用户描述与上游文档冲突时，都会主动引导去 `fs-sync`，避免文档漂移。

## 配置文件 `.faststack.yml`

| 字段 | 说明 | 默认 |
| --- | --- | --- |
| `version` | 配置文件版本，当前为 `1` | — |
| `docs_dir` | FastStack 文档根目录（相对项目根） | `.faststack` |
| `features.mvp_planning` | fs-idea 是否产出 MVP 规划 | `false` |
| `features.nfr` | fs-req 是否收集非功能需求 + 约束 | `false` |
| `features.acceptance_criteria` | fs-req / fs-tasks 是否写验收标准 | `false` |
| `features.business_analysis` | fs-prod 是否讨论商业性内容(产品价值 / 商业模式 / 成功指标) | `false` |
| `features.tests` | fs-tasks / fs-dev 是否拆测试任务、写跑自动化测试 | `false` |

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
            └── fs-dev/SKILL.md
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
