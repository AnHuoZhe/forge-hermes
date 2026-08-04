# 任务拆解规则

## 流程

1. 读取已通过架构审查的 `docs/architecture.md`，按架构依赖顺序将工作拆成 implementer 一次可独立完成的任务。
2. 在第一个模块的 `tasks` 列表最前自动插入初始化任务：

   ```json
   {
     "id": "init_env",
     "description": "检测项目测试框架是否已安装；未安装时按项目现有包管理方式安装，并验证测试框架可运行",
     "status": "pending",
     "depends_on": [],
     "test_result_path": "docs/test-results/init_env.json",
     "output_files": []
   }
   ```

   `init_env` 必须在所有实现任务之前执行。所有非 `init_env` 任务的依赖图必须直接或间接依赖 `init_env`。安装或验证失败时保持任务未完成，按既有任务失败/重试流程处理，不派发后续实现任务。
3. 任务顺序固定为：初始化环境 → 核心功能模块 → 支撑模块 → 辅助功能 → 优化；模块内使用依赖拓扑排序，禁止循环依赖和遗漏模块。
4. 写入 `docs/tasks.json`，使用标准 schema，不省略字段、不改变字段名。
5. `modules[].name` 必须与架构文档一致；`tasks[].id` 全局唯一；`status` 只能是 `pending`、`in_progress`、`done` 或 `failed`；`depends_on` 只引用已定义任务 ID；`output_files` 列出预期产物。
6. 每个任务的 `description` 必须说明一个可验证的独立交付单元，且明确其输入、输出或验收边界；不要把多个无关功能塞进同一任务。
7. 依赖图按拓扑排序、无循环，每个任务的前置任务在其之前完成。
8. 成功写入后更新 `docs/forge-state.json`：phase → build，current_task_index=0，retry_count=0，locked=false，module_review_passed=false，追加 build 到 completed_phases。
