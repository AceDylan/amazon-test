# CLAUDE.md

## 1. Interaction Guidelines (用户交互准则)

这些规则优先级最高，用于控制交互流程和代码变更的安全性。

* **0. 需求澄清与风暴 (Brainstorm First)**：在深入技术细节前，结合用户输入和 `/superpowers:brainstorm` 彻底理清需求边界。
* **1. 计划先行 (Planning & Validation)**：利用 `/superpowers:write-plan` 起草计划，但**必须**经过 Codex 的技术校验后，才能向用户汇报。
* **2. 严禁自动执行 (No Auto-Execution)**：**严禁使用 `/superpowers:execute-plan**`。我们只使用 Superpowers 的思考能力，不使用其行动能力。
* **3. 等待批准 (Wait for Approval)**：在用户明确批准最终计划之前，**严禁**执行 `Edit` 或 `Bash` 命令（只读命令除外）。
* **4. 保持质疑 (Be Critical)**：Codex 是技术顾问，Superpowers 是流程辅助，你才是最终责任人。必须对所有输出进行逻辑校验。

---

## 2. Core Instruction for Hybrid Workflow (Codex + Superpowers 协作流)

在任何时刻，你必须严格执行以下混合工作流，将 Superpowers 的规划能力与 Codex 的深度技术分析相结合。

**步骤 1：需求获取与头脑风暴 (Phase 1: Discovery & Brainstorming)**

1. 初步阅读用户请求。
2. **执行 `/superpowers:brainstorm**`：利用此工具挖掘被忽略的边缘情况、架构选择和潜在风险。
3. 根据风暴结果，向用户提出关键问题，直到需求完全清晰。

**步骤 2：计划起草与 Codex 校验 (Phase 2: Planning & Validation)**

1. **执行 `/superpowers:write-plan**`：基于澄清后的需求，生成一份详细的任务清单（Draft Plan）。
2. **调用 `codex` 工具**：
* **Input**：将“用户原始需求”、“Brainstorm 的结论”以及“Draft Plan”全部发送给 Codex。
* **Instruction**：要求 Codex 对该计划进行**技术可行性校验**，补充遗漏的技术细节，并评估潜在风险。
* *参数提示：使用 `sandbox="read-only"*`。



**步骤 3：方案汇报与审批 (Phase 3: Proposal)**
结合 Codex 的校验反馈，修正计划，并通过 `AskUserQuestion` 向用户汇报最终的**实施方案**。
*只有在用户回复“同意”后，才能进入编码阶段。*

**步骤 4：获取原型 (Phase 4: Prototyping)**
*从这里开始，禁止使用 Superpowers，完全切换回 Codex 模式。*
在实施具体编码任务前，**必须向 Codex 索要代码实现原型**。

* 参数要求：`sandbox="read-only"`
* 指令要求：要求 Codex **仅给出 unified diff patch**，严禁对代码做任何真实修改。

**步骤 5：代码重写与实施 (Phase 5: Implementation)**
获取 Codex 的 patch 后，你**只能以此为逻辑参考**。你必须运用你的编程能力，**重新编写**代码，确保形成企业生产级别、可读性极高、可维护性极高的代码，然后使用 `Edit` 工具实施修改。

**步骤 6：代码审查 (Phase 6: Review)**
完成编码后，**必须立即使用 Codex review 代码改动**，检查需求完成程度和潜在 Bug。

---

## 3. Codex Tool Invocation Specification

### 1. 工具概述

codex MCP 提供了一个工具 `codex`，用于执行 AI 辅助的编码任务。

### 2. 工具参数

**必选参数：**

* `PROMPT` (string): 发送给 codex 的任务指令
* `cd` (Path): codex 执行任务的工作目录根路径

**可选参数：**

* `sandbox` (string): 沙箱策略，可选值：
* `"read-only"` (默认): **分析、校验计划、获取原型阶段务必使用此模式**
* `"workspace-write"`: 允许在工作区写入


* `SESSION_ID` (UUID | null): 用于保持上下文连贯，默认为 None
* `skip_git_repo_check` (boolean): 默认 False
* `return_all_messages` (boolean): 默认 False

### 3. 调用规范 (最佳实践)

1. **会话保持**：在 Phase 2 (校验) 到 Phase 6 (审查) 的过程中，尽量复用 `SESSION_ID` 以保持上下文。
2. **安全隔离**：在获取代码思路时，始终使用 `sandbox="read-only"`。


## 4. File Reading Strategy (文件读取策略)

**强制规则**：每次调用 Read 工具时 **必须** 指定 `offset` 和 `limit` 参数，禁止使用默认值。

### 参数要求

| 参数 | 要求 | 说明 |
| --- | --- | --- |
| `offset` | **必须指定** | 起始行号（从 0 开始） |
| `limit` | **必须指定** | 读取行数，单次不超过 500 行 |

### 读取流程

1. **侦察 (Recon)**：先用 Grep 了解文件结构，或定位目标关键词行号。
2. **精准打击 (Precision Strike)**：使用 offset + limit 精确读取目标区域。
3. **扩展 (Expansion)**：如果需要更多上下文，再调整 offset 继续读取。

> **目标**：保持上下文精准、最小化。如果不遵守，工具调用将被 Hook 拦截。
