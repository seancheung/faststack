---
name: fs-start
description: FastStack 项目初始化与流程引导。首次运行会引导用户选择文档目录并创建 `.faststack.yml` 配置；之后作为流程总控，诊断当前阶段并推荐下一个 fs-* skill。适用于"从头开始"或"不知道从哪步开始"。
---

# fs-start — FastStack 启动向导

兼具两个职责：**首次初始化** 和 **流程路由**。

## 第一步：检查配置

读取项目根目录的 `.faststack.yml`。

### 如果配置不存在 → 走初始化流程

**1. 引导用户选择文档目录**（用 `AskUserQuestion` 或直接问）：

- `.faststack/`（默认推荐 · 隐藏目录，不干扰源码）
- `docs/`（公开目录 · 适合开源项目）
- 自定义路径

**2. 写入 `.faststack.yml`**：

```yaml
# FastStack 配置
# 文档: https://github.com/seancheung/faststack
version: 1

# FastStack 生成的所有文档会放在这个目录下
docs_dir: .faststack
```

**3. 创建 `{docs_dir}/` 目录**，并放一个 `README.md` 说明文件清单：

```markdown
# FastStack 文档

本目录由 FastStack 插件管理，包含从想法到开发的全流程产出：

- `idea.md` — 想法梳理（fs-idea）
- `requirements.md` — 需求文档（fs-req）
- `product.md` — 产品设计（fs-prod）
- `design.md` + `prototype/` — 设计规范与原型（fs-ui）
- `technical.md` — 技术设计（fs-tech）
- `tasks.md` — 任务清单（fs-tasks / fs-dev）
- `changelog.md` — 变更日志（fs-sync）
```

**4. 提示用户** 配置已创建，接下来可以运行哪个 skill。

### 如果配置已存在 → 走路由流程

1. 解析 `docs_dir`
2. 扫描 `{docs_dir}/` 下哪些文档已生成
3. 根据缺失情况给出明确建议，例如：
   - 只有 `idea.md`：建议运行 `fs-req`
   - 有到 `product.md`：建议 `fs-ui` 或（若纯后端）`fs-tech`
   - 有 `tasks.md`：建议 `fs-dev` 开始开发
4. **不要自己去生成文档或写代码**，把控制权交回对应 skill

## 标准流程

```
fs-idea → fs-req → fs-prod → fs-ui → fs-tech → fs-tasks → fs-dev
 想法     需求     产品      UI     技术      任务      开发
```

文档之间互相引用（需求 → 产品 → 设计 → 技术 → 任务），每份文档顶部写明"上游 / 下游"链接。

## 横切 skill

- **`fs-sync`**：任何阶段发生想法变更时，用它把变更回流到上游文档 + 记 `changelog.md`

## 不要做的事

- 不要一次性跑完整个流程，每一步需要在对应 skill 中交互确认
- 不要修改已有的 `.faststack.yml`（除非用户明确要改配置）
- 不要越俎代庖直接生成需求或代码
