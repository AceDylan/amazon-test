# CCG 高级协作工作流配置 (Enhanced Hybrid Workflow)

## 1. 核心交互原则 (Interaction Guidelines)

**最高优先级规则**：

1. **计划先行 (Plan & Validate)**：禁止在未经过 Codex/Gemini 技术校验且未获用户批准的情况下直接编写代码。
2. **代码主权 (Code Sovereignty)**：
* **Codex/Gemini** (通过 CLI)：仅负责分析、架构设计、生成 Diff/原型、Review。**禁止**直接操作文件系统进行写操作。
* **Claude** (当前模型)：负责最终代码的合成与写入 (`Edit` 工具)。你必须参考后端模型的建议，编写生产级代码。


3. **上下文极简 (Minimal Context)**：严格遵守文件读取策略，避免 Token 浪费。

---

## 2. 工作流阶段 (Phased Workflow)

在处理复杂任务时，必须严格遵循以下阶段：

### Phase 1: 需求增强与风暴 (Discovery)

* **触发条件**：用户发送模糊需求。
* **执行动作**：
1. 调用 `mcp__ace-tool__enhance_prompt` 将需求转化为结构化任务。
2. 若需求仍有缺漏，向用户输出引导性问题列表，直至边界清晰。



### Phase 2: 上下文全量检索 (Context Retrieval)

* **执行原则**：禁止基于假设 (Assumption) 回答。
* **工具调用**：调用 `mcp__ace-tool__search_context`。
* **文件读取策略 (File Reading Strategy)**：
* **侦察**：先用 Grep/Tree 了解结构。
* **精准打击**：使用 `read_file` 时**必须指定** `offset` 和 `limit` (建议单次 < 300 行)。
* **禁止**：全量读取无关的大文件。



### Phase 3: 计划与校验 (Planning & Validation)

* **动作**：
1. 起草 Draft Plan。
2. **双模校验**：使用 CLI 调用后端模型（Codex 查逻辑，Gemini 查前端/交互）对计划进行“可行性校验”。
3. **用户审批**：向用户汇报最终实施方案（含风险评估），**获得“同意”后方可进入下一阶段**。



### Phase 4: 原型获取 (Prototyping)

* **动作**：调用后端模型获取实现原型。
* **指令要求**：要求后端模型**仅给出 unified diff patch** 或 **伪代码**，严禁其尝试直接修改。

### Phase 5: 实施与审查 (Implementation & Review)

* **实施**：Claude 参考 Phase 4 的原型，重写为高质量代码并执行 `Edit`。
* **审查**：代码变更后，立即再次调用 CLI 让后端模型进行 Code Review。

---

## 3. 多模型协作调用规范 (CLI Specs)

所有后端能力通过 `codeagent-wrapper` 调用。

### 通用调用策略

1. **会话复用**：
* 首次调用获取 `SESSION_ID`。
* 后续相关任务必须使用 `resume <SESSION_ID>` 保持上下文连贯。


2. **并行加速**：
* 对于独立的分析任务（如同时分析前端和后端），必须使用 `run_in_background: true`。
* 使用 `TaskOutput` 阻塞等待所有结果返回。



### 后端任务 (Codex)

* **适用场景**：逻辑实现、算法、数据库、API、Plan 校验、Code Review。
* **调用模板**：
```bash
# 新会话
codeagent-wrapper --backend codex - [工作目录] <<'EOF'
[Context]: <传入当前已知信息>
[Task]: <具体的分析/校验/原型生成任务>
[Constraint]: Output unified diff only. Do NOT write files.
EOF

# 复用会话
codeagent-wrapper --backend codex resume <SESSION_ID> - [工作目录] <<'EOF' ... EOF

```



### 前端任务 (Gemini)

* **适用场景**：UI 组件、CSS、响应式布局、前端交互逻辑。
* **调用模板**：
```bash
codeagent-wrapper --backend gemini - [工作目录] <<'EOF' ... EOF

```



---

## 4. 工具使用清单

### 提示词增强

`mcp__ace-tool__enhance_prompt`

* **作用**：将自然语言转化为结构化、无歧义的执行清单。

### 上下文检索

`mcp__ace-tool__search_context`

* **策略**：语义查询 (Where/What/How)，确保获取相关类、函数的完整签名。

### 核心执行

`Bash` (用于执行 codeagent-wrapper)
`Edit` (仅限 Claude 使用，用于最终代码写入)

---

## 5. 异常处理

* **上下文不足**：若检索后信息仍不全，触发递归检索。
* **模型冲突**：若 Codex 和 Gemini 给出的建议冲突，以 Codex (逻辑优先) 或 Gemini (视觉优先) 为准，或向用户呈现冲突点供决策。
