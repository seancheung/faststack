---
name: fs-start
description: FastStack 项目初始化与流程引导。首次运行引导用户勾选 features（多选）与文档目录并创建 `.faststack.yml` 配置；之后作为流程总控，诊断当前阶段并推荐下一个 fs-* skill。流程本身不受 features 影响，开关只决定各 skill 的询问深度与产出详略。
---

# fs-start — FastStack 启动向导

兼具两个职责：**首次初始化** 和 **流程路由**。

## 第一步：检查配置

读取项目根目录的 `.faststack.yml`。

### 如果配置不存在 → 走初始化流程

**1. 引导用户勾选 features**（`AskUserQuestion` 的多选模式，**默认全部不选**）：

- **`mvp_planning`** · fs-idea 产出 MVP 规划（最小可行形态）
- **`nfr`** · fs-req 收集非功能需求（性能 / 安全 / 合规）+ 约束与假设
- **`business_analysis`** · fs-prod 讨论产品价值 / 商业模式 / 成功指标,并启动商业视角的功能审查

判断启发式：
- "个人小工具 / 给自己用 / 原型验证" → 建议全关（保持默认）
- "公司项目 / 正式产品 / 对外发布 / 团队协作" → 建议全开

告诉用户：全部省略也 OK，之后随时改 `.faststack.yml` 即可。测试与验收不在 features 中——始终由 `fs-qa` 负责，按需调用。

**2. 引导用户选择文档目录**：

- `.faststack/`（默认推荐 · 隐藏目录，不干扰源码）
- `docs/`（公开目录 · 适合开源项目）
- 自定义路径

**3. 写入 `.faststack.yml`**（未勾选的 feature 也显式写 `false`，便于用户之后看到有哪些开关）：

```yaml
# FastStack 配置
# 文档: https://github.com/seancheung/faststack
version: 1

# FastStack 生成的所有文档会放在这个目录下
docs_dir: .faststack

# 各 feature 开关；默认全关 = 最精简流程
features:
  mvp_planning: false          # fs-idea：MVP 规划
  nfr: false                   # fs-req：非功能需求 + 约束
  business_analysis: false     # fs-prod：产品价值 / 商业模式 / 成功指标
```

**4. 创建 `{docs_dir}/` 目录**，放一个 `README.md` 说明文件清单（fs-idea/req/prod/ui/tech/tasks 对应的产物）。

**5. 提示用户** 接下来运行哪个 skill。

### 如果配置已存在 → 走路由流程

1. 解析 `docs_dir` 和 `features`（未定义的 feature 字段按 `false` 处理）
2. 扫描 `{docs_dir}/` 下哪些文档已生成
3. 根据缺失情况给出建议：
   - 只有 `idea.md`：建议 `fs-req`
   - 有到 `product.md`：建议 `fs-ui` 或（纯后端）`fs-tech`
   - 有 `tasks.md`：建议 `fs-dev`
   - `tasks.md` 大部分 / 全部完成且无 `qa.md`：建议 `fs-qa` 生成测试计划
   - 有 `qa.md` 且含未执行用例：建议 `fs-qa` 继续执行
   - `bugs.md` 中有 `❌ 待修`：建议 `fs-dev BUG-xxx` 处理
   - `bugs.md` 中有 `✅ 已修待重测`：建议 `fs-qa` 重测
4. **不要自己去生成文档或写代码**，把控制权交回对应 skill

## 标准流程

```
fs-idea → fs-req → fs-prod → fs-ui → fs-tech → fs-tasks → fs-dev → fs-qa
```

## Feature 差异速查

| Feature | 开启时 | 关闭时（默认） | 影响 skill |
| --- | --- | --- | --- |
| `mvp_planning` | fs-idea 追问并产出 MVP 规划 | fs-idea 只要边界，不聊 MVP | `fs-idea` |
| `nfr` | fs-req 收集非功能需求 + 约束假设 | fs-req 只收功能需求 | `fs-req` |
| `business_analysis` | fs-prod 讨论商业内容 + 商业视角审查 | fs-prod 不讨论任何商业内容(产品价值 / 商业模式 / 指标),审查去掉商业视角 | `fs-prod` |

`fs-ui` / `fs-tech` / `fs-tasks` / `fs-dev` / `fs-qa` / `fs-sync` 不受任何 feature 影响。测试与验收是 `fs-qa` 的独立职责。

## 横切 skill

- **`fs-sync`**：任何阶段变更时回流到上游文档 + 记 `changelog.md`

## 不要做的事

- 不要一次性跑完整个流程，每步在对应 skill 交互确认
- 不要修改已有的 `.faststack.yml`（除非用户明确要换目录 / 启停某个 feature）
- 不要越俎代庖直接生成需求或代码
