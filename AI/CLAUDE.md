# CCG 工作流增强配置

## 1. Prompt 增强 (每次对话自动触发)
**执行时机**：用户发送消息时
**工具调用**：调用 mcp__ace-tool__enhance_prompt
**作用**：将模糊需求转化为结构化、可执行的任务描述，返回中文

## 2. 上下文全量检索 (生成代码前必须执行)
**执行时机**：在生成任何建议或代码之前
**工具调用**：调用 mcp__ace-tool__search_context

**检索策略**：
- 禁止基于假设 (Assumption) 回答
- 使用自然语言构建语义查询 (Where/What/How)
- 完整性检查：必须获取相关类、函数、变量的完整定义与签名
- 若上下文不足，触发递归检索直至信息完整

**需求对齐**：若检索后需求仍有模糊空间，必须向用户输出引导性问题列表，直至需求边界清晰（无遗漏、无冗余）。

## 3. 工作流执行原则

1. 先检索，后生成：永远不要在没有调用 search_context 的情况下编写代码
2. 增强需求：对于复杂任务，先调用 enhance_prompt 明确需求边界
3. 智能路由：根据任务类型自动选择 Codex/Gemini/Claude
4. 交叉验证：关键决策使用双模型并行分析
5. 代码主权：Codex/Gemini 仅负责分析、规划、审查，禁止直接修改文件；所有代码实现由 Claude 完成


## 多模型协作调用规范

### Codex CLI 调用 (后端任务)
调用方式：codeagent-wrapper --backend codex - [工作目录] <<'EOF' ... EOF
适用场景：后端逻辑、算法实现、数据库操作、API 开发、性能优化、调试分析

### Gemini CLI 调用 (前端任务)
调用方式：codeagent-wrapper --backend gemini - [工作目录] <<'EOF' ... EOF
适用场景：UI/UX 组件开发、CSS 样式、响应式布局、前端交互逻辑

### 会话复用
每次调用返回 SESSION_ID: xxx，后续阶段用 resume xxx 复用上下文，保持对话连贯性。

新会话调用：codeagent-wrapper --backend <codex|gemini> - [工作目录] <<'EOF' ... EOF
复用会话调用：codeagent-wrapper --backend <codex|gemini> resume <SESSION_ID> - [工作目录] <<'EOF' ... EOF

### 并行调用
使用 run_in_background: true 启动后台任务，用 TaskOutput 等待结果。必须等所有模型返回后才能进入下一阶段。

并行启动示例：
Bash({ command: "codeagent-wrapper --backend codex ...", run_in_background: true, timeout: 3600000 })
Bash({ command: "codeagent-wrapper --backend gemini ...", run_in_background: true, timeout: 3600000 })

等待后台任务：TaskOutput({ task_id: <TASK_ID>, block: true, timeout: 600000 })
