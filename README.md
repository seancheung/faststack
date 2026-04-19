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
```

## 使用

**第一次使用**：输入 `/fs-start`，它会引导你选择 **模式**（full / lite）和 **文档目录**（默认 `.faststack/`），并在项目根生成 `.faststack.yml`：

```yaml
version: 1
mode: full          # 或 lite
docs_dir: .faststack
```

## 两种模式

流程本身在两种模式下一致（7 步 skill 链不变），mode 只影响各 skill 的询问深度与产出详略：

| Skill | full（默认） | lite |
| --- | --- | --- |
| `fs-idea` | 含 MVP 规划 | 无 MVP，只要边界 |
| `fs-req` | 功能 + 非功能 + 约束 + 验收标准 | 只要用户角色 + 功能 + 关键流程 |
| `fs-prod` | 含数据与成功指标 | 去掉数据与指标 |
| `fs-tasks` | 每条含验收标准，测试分散到各任务 | 不要验收标准，不拆测试任务 |
| `fs-dev` | 跑 lint + typecheck + test（含 e2e） | 跑 lint + typecheck + 目视验证，**不写不跑测试** |
| `fs-ui` / `fs-tech` / `fs-sync` | — | 无差异 |

选择建议：个人小工具 / 原型 这类走 lite；正式产品、对外发布、团队协作走 full。

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
| `mode` | 流程模式：`full` 或 `lite`（不改变 skill 链，只影响询问深度和产出详略） | `full` |
| `docs_dir` | FastStack 文档根目录（相对项目根） | `.faststack` |

想换目录或切模式，改文件后下次运行任一 skill 即生效。

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
