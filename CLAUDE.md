# CLAUDE.md

## 1. Interaction Guidelines (用户交互准则)
这些规则优先级最高，用于控制交互流程和代码变更的安全性。

- **0. 需求澄清优先 (Requirement Clarification First)**：在调用任何工具（包括 Codex）或制定计划之前，**必须先使用 `AskUserQuestion` 向用户提问**。确保你完全理解了“要做什么”以及“用户的具体期望”，避免盲目开始分析。
- **1. 制定并确认计划 (Plan & Confirm)**：在完成需求澄清并获得 Codex 的技术分析后，你必须使用 `AskUserQuestion` 将最终实施方案展示给用户。
- **2. 等待批准 (Wait for Approval)**：在用户明确批准你的计划之前，**严禁**执行 `Edit` 或 `Bash` 命令（只读命令除外）。
- **3. 保持质疑 (Be Critical)**：Codex 只是顾问，你才是执行者。必须对 Codex 的输出进行逻辑校验。

---

## 2. Core Instruction for CodeX MCP (CodeX 协作核心指令)
在任何时刻，你必须思考当前过程可以如何与 Codex 进行协作，将其作为你客观全面分析的保障。

**务必严格执行以下工作流：**

**步骤 1：需求获取与澄清 (Phase 1: Discovery)**
不要直接开始编码或分析。首先通过与用户对话，彻底明确需求细节。只有当你觉得“我已经完全懂了用户想要什么”之后，才进入下一步。

**步骤 2：Codex 辅助分析 (Phase 2: Codex Analysis)**
将经过澄清的**用户需求**和你初步的**上下文理解**告知 Codex（使用 `codex` 工具），要求 Codex：
1. 完善需求分析；
2. 检查潜在风险；
3. 生成详细的实施计划。

**步骤 3：方案汇报与审批 (Phase 3: Proposal)**
获得 Codex 的分析后，结合你自己的理解，通过 `AskUserQuestion` 向用户汇报最终的**实施计划**。
*只有在用户回复“同意”或类似确认后，才能继续。*

**步骤 4：获取原型 (Phase 4: Prototyping)**
在实施具体编码任务前，**必须向 Codex 索要代码实现原型**。
* 参数要求：`sandbox="read-only"`
* 指令要求：要求 Codex **仅给出 unified diff patch**，严禁对代码做任何真实修改。

**步骤 5：代码重写与实施 (Phase 5: Implementation)**
获取 Codex 的 patch 后，你**只能以此为逻辑参考**。你必须运用你的编程能力，**重新编写**代码，确保形成企业生产级别、可读性极高、可维护性极高的代码，然后使用 `Edit` 工具实施修改。

**步骤 6：代码审查 (Phase 6: Review)**
无论何时，只要完成切实编码行为后，**必须立即使用 Codex review 代码改动**，检查需求完成程度和潜在 Bug。

---

## 3. Codex Tool Invocation Specification

### 1. 工具概述
codex MCP 提供了一个工具 `codex`，用于执行 AI 辅助的编码任务。该工具**通过 MCP 协议调用**，无需使用命令行。

### 2. 工具参数

**必选参数：**
- `PROMPT` (string): 发送给 codex 的任务指令
- `cd` (Path): codex 执行任务的工作目录根路径

**可选参数：**
- `sandbox` (string): 沙箱策略，可选值：
    - `"read-only"` (默认): 只读模式，最安全 **(分析和原型阶段务必使用此模式)**
    - `"workspace-write"`: 允许在工作区写入
    - `"danger-full-access"`: 完全访问权限
- `SESSION_ID` (UUID | null): 用于继续之前的会话以与 codex 进行多轮交互，默认为 None（开启新会话）
- `skip_git_repo_check` (boolean): 是否允许在非 Git 仓库中运行，默认 False
- `return_all_messages` (boolean): 是否返回所有消息（包括推理、工具调用等），默认 False
- `image` (List[Path] | null): 附加一个或多个图片文件到初始提示词，默认为 None
- `model` (string | null): 指定使用的模型，默认为 None（使用用户默认配置）
- `yolo` (boolean | null): 无需审批运行所有命令（跳过沙箱），默认 False
- `profile` (string | null): 从 `~/.codex/config.toml` 加载的配置文件名称，默认为 None

**返回值：**
```json
{
  "success": true,
  "SESSION_ID": "uuid-string",
  "agent_messages": "agent回复的文本内容",
  "all_messages": []
}

```

### 3. 调用规范 (最佳实践)

1. **会话保持**：每次调用 codex 工具时，必须保存返回的 `SESSION_ID`，以便在后续步骤（如分析->原型->Review）中保持上下文连贯。
2. **路径准确**：`cd` 参数必须指向存在的目录。
3. **安全隔离**：在获取代码思路时，始终使用 `sandbox="read-only"`，避免 Codex 意外覆盖文件。

