---
name: long-task-feature-design
description: "在 TDD 之前在 long-task 项目中使用 — 产出带接口契约、算法伪代码、图表和测试清单的特性级详细设计"
---

# 特性级详细设计 — SubAgent 派遣

将特性详细设计生产委托给具有新鲜上下文的 SubAgent。主 Agent 仅派遣和解析结构化结果 — 它永远不会直接读取设计/SRS/UCD 文档节或编写设计文档。

**在开始时宣布：** "我正在使用 long-task-feature-design 技能通过 SubAgent 产出详细设计。"

## 何时运行

- Worker 步骤 4，在 TDD 之前（步骤 5-7）
- 对于每个特性（对于 `category: "bugfix"` 特性为精简版本）
- 由 `long-task-work` 作为子技能调用（不是直接由路由器调用）

> **对于 `category: "bugfix"` 特性**：SubAgent 应专注于：(1) 根因文档（来自 `root_cause` 字段），(2) 有针对性的修复方法，(3) 来自 SRS 验收标准的回归测试清单（通过 `srs_trace`）。除非 bug 直接触及这些表面，否则跳过完整接口契约、数据流图和状态图。

## 步骤 1：收集路径参数

从当前会话状态收集这些。**不要**自己读取文档内容：

- `feature_json` — 来自 feature-list.json 的当前特性对象（紧凑 JSON）
- `quality_gates_json` — 来自 feature-list.json 的 quality_gates（紧凑 JSON）
- `tech_stack_json` — 来自 feature-list.json 的 tech_stack（紧凑 JSON）
- `design_doc_path` — 设计文档路径（`docs/plans/*-design.md`）
- `design_start` / `design_end` — §4.N 小节的行范围（来自 Orient Document Lookup）
- `srs_doc_path` — SRS 文档路径（`docs/plans/*-srs.md`）
- `srs_start` / `srs_end` — FR-xxx 小节的行范围（来自 Orient Document Lookup）
- `ucd_doc_path` — UCD 文档路径（仅当 `"ui": true`；否则省略）
- `ucd_start` / `ucd_end` — 相关 UCD 节的行范围（如适用）
- `ats_doc_path` — ATS 文档路径（`docs/plans/*-ats.md`），如果存在；否则省略
- `constraints` — feature-list.json 根的 constraints[]
- `assumptions` — feature-list.json 根的 assumptions[]
- `output_path` — 目标文件：`docs/features/YYYY-MM-DD-<feature-name>.md`
- `working_dir` — 项目工作目录

## 步骤 2：构建 SubAgent 提示

```
您是特性设计执行 SubAgent。

## 您的任务
1. 阅读执行规则：Read {skills_root}/long-task-feature-design/references/feature-design-execution.md
2. 阅读模板：Read {skills_root}/long-task-feature-design/references/feature-design-template.md
3. 阅读设计节：Read {design_doc_path} lines {design_start} to {design_end}
4. 阅读 SRS 节：Read {srs_doc_path} lines {srs_start} to {srs_end}
5. 阅读 UCD 节：Read {ucd_doc_path} lines {ucd_start} to {ucd_end}（仅当 ui:true）
5b. 阅读 ATS 映射表：Read {ats_doc_path}（仅当 ATS 文档存在）— 找到特性需求 ID 的映射行（来自 srs_trace）；提取所需类别
5c. 阅读内部 API 契约：Read {design_doc_path} 第 6.2 节 — 找到此特性作为 Provider 或 Consumer 出现的行。这些定义此特性必须产生或消费的精确模式。
6. 遵循执行规则产出详细设计文档
7. 将文档写入：{output_path}
8. 使用执行规则中的结构化返回契约返回您的结果

## 输入参数
- Feature: {feature_json}
- quality_gates: {quality_gates_json}
- tech_stack: {tech_stack_json}
- Constraints: {constraints}
- Assumptions: {assumptions}
- ATS doc path: {ats_doc_path}（或"none"如果没有 ATS 文档）
- Working directory: {working_dir}

## 关键约束
- 将完整设计文档写入 {output_path}
- 每个节（§2-§6）必须是完整的或有"N/A — [原因]"
- 测试清单负率必须 >= 40%
- 测试清单主要类别（FUNC/BNDRY/SEC/UI/PERF/INTG）必须覆盖此特性需求的所有 ATS 必需类别
- 具有外部依赖的特性必须有 ≥1 INTG 行 per 依赖类型；纯计算特性："INTG: N/A"
- 具有 `"ui": true` 的特性必须有完整的视觉渲染契约（§Visual Rendering Contract）：列出所有视觉元素、指定渲染技术、定义正向渲染断言。N/A 仅对 `"ui": false` 有效。对于每个正向渲染断言，必须至少存在一个 `UI/render` 测试清单行。缺失行 → FAIL。
- 不要开始 TDD — 只产出设计文档
```

## 步骤 3：派遣 SubAgent

**Claude Code：** 使用 `Agent` 工具：
```
Agent(
  description = "Feature Design for feature #{feature_id}",
  prompt = [上面构建的提示]
)
```

**OpenCode：** 使用 `@mention` 语法或平台的原生子代理机制，包含相同的提示内容。

## 步骤 4：解析结果

读取 SubAgent 返回的文本并找到 `### Verdict:` 行：

- **`### Verdict: PASS`**
  1. 验证设计文档文件存在于 `output_path`
  2. **视觉渲染契约抽查（仅 ui:true）：** 主 Agent（不是 SubAgent）从生成的文档读取 `## Visual Rendering Contract` 节并验证：
     - 至少列出一个具有具体 DOM/Canvas 选择器的视觉元素（不是"页面"或"UI"这样的通用词）
     - 指定了渲染技术（Canvas 2D / WebGL / DOM / SVG / CSS）
     - 至少一个正向渲染断言引用具体视觉结果（不仅仅是"元素可见"）
     - 测试清单中 `UI/render` 行数等于或超过视觉渲染契约元素数
     - **如果任何检查失败**：重新派遣 SubAgent，反馈："视觉渲染契约不完整 — [具体差距]。通过第 1 层错误检测的空白页面不可接受。用户应该看到的每个视觉元素必须列出，带可测试选择器和断言。"
  3. 提取下一步输入：`feature_design_doc`、`test_inventory_count`、`tdd_task_count`
  4. 记录在 `task-progress.md`："特性设计：PASS（{N} 个测试场景，{M} 个 TDD 任务）"
  5. 如果 `assumption_count > 0`：追加到 `task-progress.md`："（{K} 个假设记录在澄清附录中）"
  6. 继续到 TDD（步骤 5-7）

- **`### Verdict: CLARIFY`**
  1. 读取歧义表 — 提取所有分类的问题
  2. 通过 `AskUserQuestion` 以结构化格式呈现给用户：
     ```
     需要特性设计澄清：特性 #{id}（{title}）

     在分析需求和设计文档时，发现 {N} 个影响设计的歧义。
     对于每个，提供建议解释 — 您可以接受它、提供不同答案，或说"skip"使用建议作为假设。

     歧义 1 [{category}]：{description}
       来源：{source}
       影响：{impact}
       建议：{suggested_interpretation}
       → 您的答案（或"accept"使用建议，或"skip"作为假设）：

     歧义 2 [{category}]：...
     ```
  3. 解析用户响应 — 对于每个歧义，记录：
     - "accept" 或特定答案 → 解决方案，Authority = "user-approved"
     - "skip" → 解决方案 = 建议解释，Authority = "assumed"
  4. **批准门控**：收集所有答案后，通过 `AskUserQuestion` 呈现摘要：
     ```
     特性 #{id} 澄清摘要：
     1. [{category}] {description} → 解决方案：{answer}（Authority：{authority}）
     2. ...

     继续这些解决方案？（yes / revise #N）
     ```
     - 如果批准：继续步骤 5
     - 如果用户想要修订：重新询问特定项目，然后重新呈现摘要
  5. 构建**澄清附录**并用原始提示**加上**重新派遣 SubAgent：
     ```
     ## 澄清附录（用户批准解决方案）
     | # | 类别 | 原始歧义 | 解决方案 | Authority |
     |---|----------|--------------------|------------|-----------|
     | 1 | {category} | {description} | {resolution} | user-approved / assumed |

     将这些解决方案作为权威约束应用。不要将这些重新标记为歧义。
     将它们纳入设计，就像它们在原始 SRS/设计文档中一样。
     ```
  6. 记录在 `task-progress.md`："特性设计：CLARIFY（{N} 个歧义已解决）→ 重新派遣"
  7. **最大 2 轮澄清**：如果 SubAgent 在收到澄清后第二次返回 `CLARIFY`，升级剩余歧义给用户：
     "2 轮澄清后仍发现持续规范差距。考虑使用 `long-task-increment` 更新 SRS/设计文档。"
     - 如果用户说"需要更新 SRS"：在 `task-progress.md` 中记录差距，建议 `long-task-increment`，跳到下一个符合条件的特性
     - 如果用户提供最终答案：纳入并重新派遣一次
     - 如果仍然无法解决：设置为 BLOCKED

- **`### Verdict: FAIL`**
  1. 读取问题表 — 识别哪些节不完整
  2. 如有需要，使用额外上下文重新派遣 SubAgent（最多 2 次重试）
  3. 如果仍然失败，通过 `AskUserQuestion` 升级给用户

- **`### Verdict: BLOCKED`**
  1. 读取问题表 — 识别阻塞者
  2. 通过 `AskUserQuestion` 升级给用户

## 集成

**被调用：** long-task-work（步骤 4）
**需要：** 系统设计文档、SRS、feature-list.json
**产出：** `docs/features/YYYY-MM-DD-<feature-name>.md`（由 SubAgent 写入）
**链到：** long-task-tdd（通过 Work 步骤 5-7）
