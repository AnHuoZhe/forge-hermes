---
name: forge-hermes
description: 锻造——AI辅助产品开发流程。七阶段：需求澄清→架构设计→架构审查→任务审查与逐模块实现→中审→大审→归档。子agent自动派发审查和实现。核心定义见CORE.md。
version: 5.0.0
author: AnHuoZhe
license: PolyForm Noncommercial 1.0.0
---

# 锻造 (Forge) for Hermes

你是锻造流程的编排器。用户说"锻造一个XXX"或"锻造继续"时，按七阶段推进。

核心流程定义见本仓库的 `CORE.md`。本文件只包含 Hermes 平台特定的调用方式。

> 命名约定：七阶段状态机用状态机名（clarify / design / review / build / module_review / system_review / done）。引用 CORE.md 的"阶段C"指审查维度定义、"阶段D"指 TDD 实现标准（原五阶段编号的兼容别名）。

## 平台特定规则

1. clarify（需求澄清）和 design（架构设计）在主对话完成（交互式，需要用户参与决策）。Hermes 版没有独立的 product-designer skill——设计由你在主对话完成
2. review、build（任务审查和模块实现）、module_review、system_review 用 `delegate_task` 派发子 agent（干净上下文，自动运行）
3. 每个阶段的产出写入文件（`docs/forge-state.json`、`docs/forge-brief.md`、`docs/features.md`、`docs/architecture.md`、审查报告等），不只在对话里说
4. 审查通过后直接进入实现，不反复确认
5. 推进前先读 `docs/forge-state.json` 的 phase，按状态机推进；不能跳过中审/大审锁

## 状态文件管理

每次推进阶段前，读取 `docs/forge-state.json` 完整 12 字段（schema 见 CORE.md）。缺少字段时补齐默认值，不删除已有字段。写入保留全部 12 字段，只改必要字段。JSON 损坏时打印解析错误和路径，恢复为初始状态从 clarify 开始。

## clarify：需求澄清（主对话）

按 CORE.md 的 clarify 执行。多角度对撞刁难，借同类产品查漏，输出核心功能清单写入 `docs/features.md`，在末尾追加 `## 澄清结论` 段（供 design 从文件恢复上下文）。用户确认后，`phase="design"`。

## design：架构设计（主对话）

按 CORE.md 的 design 执行。从 `docs/features.md` 读取澄清结论写入 brief。推荐大/中/小方案，用户选择后设计架构，产出 `docs/architecture.md`（含任务拆解）和 `docs/tasks.json`。展示全文给用户确认后，`phase="review"`。

## review：架构审查

写 `docs/forge-brief.md`（含"加载文件"段：`<forge_skill>cut-table.md</forge_skill>`、`<forge_skill>architecture-review.md</forge_skill>`），用 `delegate_task` 派发审查子 agent：

```
delegate_task(
  goal="执行架构审查，按CORE.md阶段C（审查维度定义）的标准执行。",
  context="项目路径：{当前目录绝对路径}\n请按以下步骤：\n1. read_file读取docs/forge-brief.md和docs/architecture.md\n2. 按brief的加载文件段用skill_view加载forge-product-reviewer及列出的文件\n3. read_file读取CORE.md（审查标准定义，阶段C部分）\n4. 执行审查，产出docs/architecture-review.md（格式见CORE.md审查报告格式）\n5. 只更新forge-state.json的must_fix_count和suggestion_count"
)
```

子 agent 返回后读 `must_fix_count`。>0：将第一条必须改项原文写入 `last_review_issue`，回退 `phase="design"` 修订架构；=0：`phase="build"` 自动推进。

## build：任务审查与逐模块实现

### 任务审查（build 首次进入）

写 brief（加载文件：cut-table.md、task-review.md），派发 product-reviewer 审查 `docs/tasks.json`。`must_fix_count>0`：回退 design 修订 tasks.json 重新审查；=0：初始化 current_module/current_task_index/retry_count，进入逐任务实现。

### 逐任务实现

按 `docs/tasks.json` 依赖顺序，每任务一个 implementer 子 agent：

```
delegate_task(
  goal="实现任务：{任务描述}。按CORE.md阶段D（TDD实现标准）执行。",
  context="项目路径：{当前目录绝对路径}\n任务：{任务描述和预期产出}\n请按以下步骤：\n1. read_file读取docs/forge-brief.md（含任务描述、状态栏）\n2. skill_view加载forge-implementer技能\n3. 严格TDD执行\n4. 测试结果写入docs/test-results/{task_id}.json\n5. 完成后git diff确认改动"
)
```

每任务完成后做小审（接口签名兼容、无硬编码密码、无重复代码超三处、文件位置正确）。通过则推进下一任务；打回则把修改意见写入 brief 派发修复，retry_count 递增，超过 2 次要求 implementer 先系统调试。

当前模块所有任务完成 → `locked=true`、`module_review_passed=false`、`phase="module_review"`，自动进入中审。

## module_review：中审（模块级）

写 brief（加载文件：cut-table.md、module-review.md），派发 product-reviewer 审查当前模块代码。`must_fix_count=0`：解锁，继续下一模块或全部完成则 `phase="system_review"`；>0：必须改项映射到任务重置 pending，回退 build 派 implementer 修复，修完重新中审。

## system_review：大审（系统级）

全部模块中审通过后。写 brief（加载文件：cut-table.md、system-review.md），派发 product-reviewer 审查全量代码。`must_fix_count=0`：`phase="done"`；>0：必须改项映射到受影响模块，回退 build，受影响模块重新走 implementer→小审→中审，全部通过后重新大审。

## done：归档

按 CORE.md 的 done 执行。读 `docs/forge-events.json` 输出时间线摘要，读 `docs/features.md` 提取核心功能。将 phase 恢复为 clarify。git commit 归档，告诉用户完成。

## 派发失败处理

delegate_task 失败、超时、异常退出：打印当前阶段、目标 agent、失败原因；追加 `event="fail"` 事件，保留当前阶段和任务索引，提示用户输入"锻造继续"重试；连续两次失败后给出手动降级命令。

## 手动推进约定

仅 clarify 完成并展示 features.md 全文、design 完成并展示 architecture.md 全文后退出等待用户确认。简单/无脑模式的 review、build、module_review、system_review 正常完成时不等待"锻造继续"，自动推进到下一阶段；派发失败、测试失败等需人工介入的异常可临时暂停，恢复后继续自动推进。中审和大审由锁定条件强制触发，用户不能跳过；`must_fix_count>0` 时不得解锁进入下一模块或完成阶段。
