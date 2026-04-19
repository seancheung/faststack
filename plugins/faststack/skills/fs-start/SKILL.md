---
name: fs-start
description: FastStack 项目初始化与流程引导。首次运行引导用户选择模式（full / lite）与文档目录并创建 `.faststack.yml` 配置；之后作为流程总控，诊断当前阶段并推荐下一个 fs-* skill。流程本身在两种模式下一致，mode 只影响各 skill 的询问深度与产出详略。
---

# fs-start — FastStack 启动向导

兼具两个职责：**首次初始化** 和 **流程路由**。

## 第一步：检查配置

读取项目根目录的 `.faststack.yml`。

### 如果配置不存在 → 走初始化流程

**1. 引导用户选择模式**（`AskUserQuestion` 或直接问）：

- **`full`**（默认 · 正式项目 / 团队协作 / 公开发布）
  每阶段完整覆盖 MVP 规划、非功能需求、约束假设、验收标准、测试、数据指标
- **`lite`**（个人项目 / 小工具 / 原型验证）
  跳过 MVP / 非功能 / 约束 / 验收 / 测试 / 数据指标，只保留边界与核心功能

判断启发式：描述"我想做个小工具给自己用" 之类 → 推荐 lite；"公司项目" / "要上线给用户" / 团队协作 → 推荐 full。

**2. 引导用户选择文档目录**：

- `.faststack/`（默认推荐 · 隐藏目录，不干扰源码）
- `docs/`（公开目录 · 适合开源项目）
- 自定义路径

**3. 写入 `.faststack.yml`**：

```yaml
# FastStack 配置
# 文档: https://github.com/seancheung/faststack
version: 1

# 模式：full（完整流程）或 lite（简化询问与产出，流程不变）
mode: full

# FastStack 生成的所有文档会放在这个目录下
docs_dir: .faststack
```

**4. 创建 `{docs_dir}/` 目录**，放一个 `README.md` 说明文件清单（fs-idea/req/prod/ui/tech/tasks 对应的产物）。

**5. 提示用户** 接下来运行哪个 skill。

### 如果配置已存在 → 走路由流程

1. 解析 `docs_dir` 和 `mode`
2. 扫描 `{docs_dir}/` 下哪些文档已生成
3. 根据缺失情况给出建议：
   - 只有 `idea.md`：建议 `fs-req`
   - 有到 `product.md`：建议 `fs-ui` 或（纯后端）`fs-tech`
   - 有 `tasks.md`：建议 `fs-dev`
4. **不要自己去生成文档或写代码**，把控制权交回对应 skill

## 标准流程（两种模式共用）

```
fs-idea → fs-req → fs-prod → fs-ui → fs-tech → fs-tasks → fs-dev
```

## 两种模式的差异速查

| Skill | full | lite |
| --- | --- | --- |
| `fs-idea` | 含 MVP 规划 | 无 MVP，只要边界 |
| `fs-req` | 功能 + 非功能 + 约束 + 验收标准 | 只要用户角色 + 功能清单 + 关键流程 |
| `fs-prod` | 含数据与成功指标 | 去掉数据与指标 |
| `fs-ui` | 不变 | 不变 |
| `fs-tech` | 不变 | 不变 |
| `fs-tasks` | 每条含验收标准，测试分散到各任务 | 不要验收标准，不拆测试任务 |
| `fs-dev` | 跑 lint + typecheck + test（含 e2e） | 跑 lint + typecheck + 目视验证，**不写不跑测试** |

## 横切 skill

- **`fs-sync`**：任何阶段变更时回流到上游文档 + 记 `changelog.md`

## 不要做的事

- 不要一次性跑完整个流程，每步在对应 skill 交互确认
- 不要修改已有的 `.faststack.yml`（除非用户明确要切模式 / 换目录）
- 不要越俎代庖直接生成需求或代码
