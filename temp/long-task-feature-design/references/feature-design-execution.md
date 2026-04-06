# 特性级详细设计 — SubAgent 执行参考

您是特性设计执行 SubAgent。严格遵循这些规则。完成后，使用此文档底部的**结构化返回契约**返回您的结果。

---

# 特性级详细设计

为单个特性产出详细设计，桥接系统级设计（§4.N）和 TDD 实现。

系统设计回答"存在哪些类以及它们如何交互。"
此技能回答"每个方法内部做什么，什么可能出错，以及如何测试它。"

## 输入

在编写任何设计内容之前，阅读**所有**这些：

1. **feature-list.json 中的特性对象** — ID、标题、描述、srs_trace、ui 标志、依赖、优先级（verification_steps 如果存在）
2. **系统设计节** — 来自设计文档的完整 §4.N（阅读整个小节，不是 grep）
3. **SRS 需求** — 来自 SRS 文档的完整 FR-xxx
4. **UCD 节**（如果 `"ui": true`）— 来自 UCD 文档的组件/页面提示
5. **feature-list.json 根的约束和假设**
6. **相关 NFR** — 可追溯到此特性的 SRS 中的 NFR-xxx
7. **现有代码** — 如果依赖特性正在通过，读取它们的公共接口（导入、类/函数签名）
8. **内部 API 契约**（如果 §6.2 存在）— 来自设计第 6.2 节，读取此特性作为 Provider 或 Consumer 出现的行。这些定义此特性的接口契约（§3）必须对齐的跨特性模式。

## 模板

使用 `skills/long-task-feature-design/references/feature-design-template.md` 作为结构模板。复制模板，为目标特性填写每个节。

## 检查清单

您**必须**按顺序完成每个步骤：

### 1. 加载上下文

阅读上面输入中列出的所有输入产物。

### 1b. 歧义扫描

在阅读所有输入之后，**在编写任何设计内容之前**，扫描可能影响设计正确性的规范歧义。此扫描使用以下分类：

| 代码 | 检查什么 |
|------|---------------|
| `SRS-VAGUE` | 验收标准包含模糊语言（"快速"、"用户友好"、"适当"、"应该处理"）而没有可衡量阈值或具体行为 |
| `SRS-DESIGN-CONFLICT` | SRS 需求和设计 §4.N 在接口类型、数据格式、行为或错误处理上矛盾 |
| `SRS-MISSING` | 验收标准没有 Given/When/Then 或未指定预期结果 |
| `ATS-MISMATCH` | ATS 需要测试类别（例如 SEC）但特性的可观察行为没有该类别的表面 |
| `UCD-VAGUE` | 视觉要求不够具体，无法得出 DOM 选择器或可测试断言（仅 ui:true） |
| `DEP-AMBIGUOUS` | 跨特性接口不清晰 — 依赖的 §6.2 条目缺失或不完整 |
| `NFR-GAP` | 引用 NFR 没有可衡量阈值（例如，"应该可扩展"没有数字） |

**扫描程序：**

1. 对于每个 SRS 验收标准（来自 srs_trace 需求）：检查它是否包含可衡量、特定、可测试的条件。标记没有数字阈值或具体行为的模糊语言 → `SRS-VAGUE`
2. 对于映射到此特性的每个 SRS 需求：交叉引用设计 §4.N。标记接口类型、数据格式、行为或错误处理中的矛盾 → `SRS-DESIGN-CONFLICT`
3. 对于每个 SRS 验收标准：验证存在带明确预期结果的 Given/When/Then → `SRS-MISSING`
4. 对于每个 ATS 必需类别（如果提供了 ATS 文档）：检查特性的可观察行为是否有该类别的可测试表面 → `ATS-MISMATCH`
5. 对于 UCD 节（如果 ui:true）：检查视觉要求是否指定具体颜色、排版、间距或选择器 → `UCD-VAGUE`
6. 对于此特性作为 Provider 或 Consumer 的 §6.2 契约：检查模式是否完整（没有缺失字段、没有模糊类型）→ `DEP-AMBIGUOUS`
7. 对于引用的 NFR：验证存在可衡量阈值 → `NFR-GAP`

**对于检测到的每个歧义，产出结构化记录：**
```
- 类别：[来自分类的代码]
- 来源：[文档路径 + 节/行引用]
- 描述：[什么是歧义的]
- 影响：[哪些设计节无法完成无法解决 — 例如，"§3 接口契约后置条件"、"§7 测试清单预期结果"]
- 建议解释：[SubAgent 基于上下文的最佳猜测（如果存在）；如果没有合理解释则"none"]
- 给用户的问题：[具体、可操作的问题，将解决歧义]
```

**对于 `category: "bugfix"` 特性**：仅扫描 bug 验收标准的 `SRS-VAGUE` 和 `SRS-DESIGN-CONFLICT`。跳过 `UCD-VAGUE`、`ATS-MISMATCH` 和 `NFR-GAP`（bugfix 特性专注于根因，不是完整规范覆盖）。

**决策门控：**
- **检测到零个歧义** → 正常继续到步骤 2。不添加摩擦。
- **所有歧义都有合理的建议解释 AND 影响仅限于非关键节**（不影响接口契约签名、测试清单预期结果或跨特性 §6.2 契约）→ 用假设继续。在设计文档的 `## Clarification Addendum` 节中记录每个假设，Authority = "assumed"。将 Verdict 设置为 `PASS`。在 `### Next Step Inputs` 中包含假设计数。
- **任何歧义有高影响**（影响接口契约签名、测试清单预期结果或跨特性契约）**或者没有合理的建议解释** → 将 Verdict 设置为 `CLARIFY`。在结构化返回契约中包含完整歧义表。**不要**继续到步骤 2 — 编排器将收集用户答案并重新派遣。

> **关于带澄清附录的重新派遣**：如果 SubAgent 提示包含 `## Clarification Addendum (user-approved resolutions)` 节，将这些解决方案作为权威约束对待。**不要**将它们重新标记为歧义。将它们纳入设计，就像它们在原始 SRS/设计文档中一样。

### 2. 组件数据流图

显示此特性内部的组件以及数据在运行时如何在它们之间流动。这**不是**系统设计类图的副本 — 它是**运行时数据流视图**，显示什么数据进入、如何转换以及什么流出。

要求：
- Mermaid `graph` 或 `flowchart` 格式
- 用数据类型标记边（组件之间流动的内容）
- 包括外部依赖为虚线边框框
- 每个组件映射到要实现的类或模块

> **跳过规则**：如果特性是单个类，单个方法，没有内部组件协作，写"N/A — single-class feature, see Interface Contract below"

### 3. 接口契约

对于此特性暴露或修改的每个**公共**方法：

| 方法 | 签名 | 前置条件 | 后置条件 | 抛出 |
|--------|-----------|---------------|----------------|--------|
| name   | full typed signature | 呼叫前必须为真的 | 呼叫后保证的 | 异常 + 条件 |

规则：
- 前置条件使用 SRS 验收标准的 Given/When 风格
- 后置条件具体且可测试（不是"返回正确结果"）
- 每个 SRS 验收标准（来自 srs_trace 需求）必须可追溯到至少一个方法的后置条件
- 仅当包含非平凡逻辑时才包含内部方法
- **§6.2 对齐规则**：对于产生或消费跨特性数据的方法，方法签名（参数、返回类型）必须与设计第 6.2 节定义的模式兼容。如果特性是 **Provider**，后置条件必须保证响应模式。如果 **Consumer**，前置条件必须假设请求模式格式。任何偏差需要在设计理由中明确说明，并触发以下契约偏差协议。

### 契约偏差协议

如果在特性设计期间，发现 §6.2 契约不正确、不充分或技术不可行：

1. **不要沉默偏差** — 不匹配的契约将导致集成失败
2. **在设计文档的设计理由节中记录偏差**：
   - 契约 ID（例如，IAPI-001）
   - 原始模式 vs 提议更改
   - 更改的技术原因
   - 对 Consumer 特性的影响（列出受影响的特性 ID）
3. **将 Verdict 设置为 BLOCKED**，Issue："Contract deviation requires design update"
4. 编排器（long-task-work）将通过 AskUserQuestion 升级给用户
5. 如果批准：用户更新设计文档中的 §6.2；编排器重新派遣 SubAgent
6. 如果拒绝：SubAgent 必须遵守原始契约

### 3b. 视觉渲染契约（对 `"ui": true` 强制）

对于具有 `"ui": true` 的特性，指定用户必须看到的所有视觉元素。此契约是 TDD 规则 7（正向渲染测试）和 Feature-ST（渲染验证）的真实来源。

**源数据**：阅读 SRS 需求的 `Visual output` 字段（用户看到什么变化）+ UCD 组件/页面提示（应该看起来如何）+ 系统设计 §4.N UI/UX 方法。

**如何填充每列**：
- **视觉元素**：命名用户看到的每个不同视觉事物（例如，"蛇身体段"、"游戏板网格"、"分数计数器"、"食物项"）。**不是**抽象概念如"UI"或"页面"。
- **DOM/Canvas 选择器**：具体的 CSS 选择器（`canvas#game-board`、`div.snake-segment`、`#score-display`）或 canvas 元素 ID。必须足够具体，`document.querySelector()` 可以找到它。
- **渲染时机**：导致此元素出现的触发器（页面加载、游戏开始、状态更改、用户动作）
- **视觉状态变体**：基于状态的不同视觉外观（alive=green，dead=red；selected=蓝色边框，unselected=灰色）
- **最小尺寸**：预期大小（每格 20x20px、视口全宽等）
- **数据源**：驱动渲染的内容（GameState.segments[]、API 响应、表单输入）

**正向渲染断言**：对于每个元素，编写关于触发后**必须**在视觉上存在的可测试陈述。不是"元素可见"而是"画布在游戏板区域有非透明像素"或"div.snake-segment 计数等于 GameState.segments.length"。

**交互深度断言**：对于每个交互元素，编写它响应什么交互以及产生什么视觉变化。渲染但不响应其设计交互的元素是"仅显示"缺陷。

> **跳过规则**：如果 `"ui": false`，写"N/A — backend-only feature"。如果 `"ui": true`，此节**强制**不能跳过 — 即使对于看起来"主要是后端"但有 `"ui": true"` 的特性。

### 4. 内部序列图

显示特性**自己**的类/函数之间的方法到方法调用。与系统设计的序列图（系统范围流）不同，这显示特性的类/函数自己的协作。

要求：
- Mermaid `sequenceDiagram` 格式
- 必须覆盖主要成功路径
- 必须覆盖接口契约 §3 中每个 Raises 条目至少一个错误路径
- 参与者是特性**自己的**类/函数

> **跳过规则**：如果特性只有一个类，没有值得图解的内部跨方法委托，写"N/A — single-class implementation, error paths documented in Algorithm §5 error handling table"

### 5. 算法 / 核心逻辑

对于每个非平凡方法（超越简单委托或 CRUD 的任何内容）：

**a) 流程图**（Mermaid `flowchart TD`）：
- 每个分支条件的决策节点
- 转换的处理节点
- 返回/抛出的终端节点

**b) 伪代码**：
```
FUNCTION name(param1: Type, param2: Type) -> ReturnType
  // Step 1: [major step]
  // Step 2: [formula or key decision]
  //         e.g., score = Σ 1/(k + rank_i) for each list
  // Step 3: [edge case handling]
  //         IF input_list is empty THEN return []
  RETURN result
END
```

**c) 边界决策表**：

| 参数 | 最小 | 最大 | 空/空值 | 在边界 |
|-----------|-----|-----|------------|-------------|
| [param]   | [val] | [val] | [behavior] | [behavior] |

**d) 错误处理表**：

| 条件 | 检测 | 响应 | 恢复 |
|-----------|-----------|----------|----------|
| [condition] | [how detected] | [exception or default] | [caller action] |

> **跳过规则**：如果方法是纯委托（调用另一个服务、返回结果），写"Delegates to [X] — see Feature #N"，而不是完整算法节。没有显式跳过的空节是缺陷。

### 6. 状态图（如适用）

对于管理有状态对象的特性（具有生命周期的实体）：

- Mermaid `stateDiagram-v2` 格式
- 所有有效状态和转换
- 转换触发器（事件/方法调用）
- 转换上的守卫条件

> **跳过规则**：如果不存在对象生命周期，写"N/A — stateless feature"。大多数查询/转换特性是无状态的。

### 7. 测试清单

作为**最终**设计步骤构建此表 — 它综合上面所有节为具体测试场景。

| ID | 类别 | 可追溯到 | 输入 / 设置 | 预期 | 消灭哪个 Bug？ |
|----|----------|-----------|---------------|----------|-----------------|
| A  | FUNC/happy | FR-xxx AC-1 | [specific values] | [exact result] | [wrong impl] |
| B  | FUNC/error | §3 Raises row | [trigger] | [exception type + msg] | [missing branch] |
| C  | BNDRY/edge | §5c boundary table | [edge value] | [behavior] | [off-by-one] |
| D  | FUNC/state | §6 transition | [pre-state + event] | [post-state] | [missing guard] |
| E  | INTG/db    | §3 method + required_configs | [real DB setup] | [data persisted + queryable] | [connection not established / wrong table] |
| F  | INTG/api   | §4.N cross-service call | [real HTTP endpoint] | [correct response schema] | [wrong endpoint / timeout not handled] |

类别格式：`MAIN/subtag`，其中 MAIN 是 `FUNC、BNDRY、SEC、UI、PERF、INTG` 之一，subtag 是自由形式标签。

规则：
- 每个 SRS 验收标准（来自 srs_trace 需求）至少一行
- 负测试（FUNC/error + BNDRY/*）>= 总行数的 40%
- "可追溯到"引用测试派生的设计节
- "消灭哪个 Bug？"命名此测试捕获的特定错误实现

**ATS 类别对齐**（如果提供了 ATS 文档）：ATS 映射表中列出的此特性需求的主要类别**必须**作为至少一行 Category 前缀出现在此测试清单中。例如，如果 ATS 要求 FR-005 的 SEC，至少一个测试清单行必须有 Category = `SEC/*`。缺失 ATS 类别 → 在继续到 §8 之前添加行。

**视觉渲染覆盖**（对 `"ui": true` 强制）：对于 §3b（视觉渲染契约）中的每个正向渲染断言，添加至少一个 `UI/render` 测试清单行。"可追溯到" = §3b 视觉渲染契约，特定元素行。"消灭哪个 Bug？" = 此测试捕获的渲染失败（例如，"render function never called"、"canvas blank"、"DOM element not appended"）。如果视觉渲染契约列出 N 个视觉元素，必须至少有 N `UI/render` 行。

**集成测试行（INTG 类别）：**
- 对于有外部依赖的特性（DB、HTTP 服务、文件系统、第三方 SDK）：每个依赖类型添加 ≥1 `INTG/*` 行
- 派生自：与外部系统交互的接口契约（§3）方法 + 具有 connection-string 键的 `required_configs[]` 条目
- "可追溯到" = §3 方法 + 特定外部依赖
- "消灭哪个 Bug？" = 单元 mock 会错过的连接/集成失败
- 如果特性是纯计算没有外部依赖：写"INTG: N/A — pure function, no external I/O"（镜像 TDD 规则 5 豁免）

**与 TDD 的关系**：此表是 TDD Red（long-task-tdd 步骤 1）的主要输入。TDD Red 使用此表作为起点，并根据其自己的规则 1-5（类别覆盖、断言质量、真实测试要求）添加测试。测试清单提供设计驱动的场景；TDD 添加编码期间发现的实现驱动场景。

**设计接口覆盖门控（强制 — 在继续到 §8 之前执行）：**

1. 重新阅读系统设计文档的 §4.N
2. 提取所有命名函数、方法、端点、中间件、验证器和授权检查（例如 `check_repo_access`、`validate_input`）
3. 对于**每个**命名项：确认至少一个测试清单行测试它（匹配"可追溯到"或"输入/设置"列）
4. 如果**任何**设计指定的函数零测试清单覆盖：
   - 添加行 — 通常是错误/安全类别
   - 设置"可追溯到" = 特定设计节（例如，"§4.5.3 ACL check"）
5. 添加后重新验证负测试比率 >= 40%

这是防止规范漂移的**主要防御**。如果设计说"check_repo_access enforces ACL"但没有测试行覆盖它，TDD 阶段将静默跳过它 — 导致延迟发现，触发级联 mock 设置成本。

### 8. TDD 任务分解

设计完成后，分解为 TDD 任务。

**任务粒度**：每个任务应该是 2-5 分钟的工作。如果任务需要更长时间，则拆分。

**任务结构**：

#### 任务 1：编写失败测试
**文件**：[确切路径]
**步骤**：
1. 创建带导入的测试文件
2. 为测试清单（§7）中的每行编写测试代码：
   - 包括 mock 设置、特定输入值、具体断言
   - 测试 A：[匹配表行 A]
   - 测试 B：[匹配表行 B]
3. 运行：`[test command]`
4. **预期**：所有测试因正确原因 FAIL

#### 任务 2：实现最小代码
**文件**：[确切路径]
**步骤**：
1. [引用算法 §5 伪代码的精确更改]
2. [引用接口契约 §3 的精确更改]
3. 运行：`[test command]`
4. **预期**：所有测试 PASS

#### 任务 3：覆盖率门控
1. 运行：`[coverage command]`
2. 检查阈值。如果低于：返回任务 1。
3. 记录覆盖率输出作为证据。

#### 任务 4：重构
1. [特定重构操作]
2. 运行完整测试套件。所有测试 PASS。

#### 任务 5：突变门控
1. 运行：`[mutation command] --paths-to-mutate=<changed-files>`
2. 检查阈值。如果低于：改进断言。
3. 记录突变输出作为证据。

### 验证检查清单
- [ ] 所有 SRS 验收标准（来自 srs_trace）可追溯到接口契约后置条件
- [ ] 所有 SRS 验收标准（来自 srs_trace）可追溯到测试清单行
- [ ] 算法伪代码覆盖所有非平凡方法
- [ ] 边界表覆盖所有算法参数
- [ ] 错误处理表覆盖所有 Raises 条目
- [ ] 测试清单负率 >= 40%
- [ ] ui:true 特性的视觉渲染契约完整（列出所有视觉元素、定义正向渲染断言、定义交互深度断言）
- [ ] 每个视觉渲染契约元素有 ≥1 UI/render 测试清单行
- [ ] 每个跳过的节有明确的"N/A — [原因]"
- [ ] §4.N 中命名的每个函数/方法至少有一个测试清单行

## 图表质量规则

具体、可验证的规则：

- **组件/流图**：每条边用数据类型标记；每个节点映射到类/模块
- **序列图**：为所有分支包含 alt/opt/loop 块；显示返回类型；参与者名称匹配 §2 中的类名
- **流程图**：每个决策节点恰好有 2 个出口；没有带标记条件的转换
- **状态图**：每个状态可从初始状态到达；每个终端可到达；没有孤立状态；模糊转换上有守卫条件

## 显式跳过规则

每个节（§2-§6）必须：
- 包含符合上述要求的**完整**内容，或
- 陈述"N/A — [此节不适用的特定原因]"

空或半填充节是阻止 TDD 的设计缺陷。说"N/A"但没有理由的节也是缺陷。

---

## 结构化返回契约

设计文档完成后，**完全按照此格式**返回结果：

```markdown
## SubAgent Result: Feature Design
### Verdict: PASS | FAIL | BLOCKED | CLARIFY
### Summary
[1-3 句话 — 设计了什么、关键架构决策、文档完整性]
### Artifacts
- [docs/features/YYYY-MM-DD-<feature-name>.md]：特性详细设计文档
### Metrics
| Metric | Value | Threshold | Status |
|--------|-------|-----------|--------|
| Sections Complete | N/8 | 8/8 (or N/A justified) | PASS/FAIL |
| Test Inventory Rows | N | ≥ SRS acceptance criteria count (from srs_trace) | PASS/FAIL |
| Negative Test Ratio | N% | ≥ 40% | PASS/FAIL |
| Verification Checklist | N/10 | 10/10 | PASS/FAIL |
| Design Interface Coverage | N/M | M/M | PASS/FAIL |
| Visual Rendering Assertions | N | ≥ Visual Rendering Contract element count (ui:true) | PASS/FAIL/N/A |
### Issues (only if FAIL or BLOCKED)
| # | Severity | Description |
|---|----------|-------------|
### Ambiguities (only if CLARIFY)
| # | Category | Source | Description | Impact | Suggested Interpretation | Question |
|---|----------|--------|-------------|--------|--------------------------|----------|
| 1 | [code] | [doc § section] | [what is ambiguous] | [affected design sections] | [best guess or "none"] | [specific question for user] |
### Assumptions Made (only if PASS with assumptions)
| # | Category | Source | Assumption | Rationale |
|---|----------|--------|------------|-----------|
| 1 | [code] | [doc § section] | [what was assumed] | [why this is reasonable] |
### Next Step Inputs
- feature_design_doc: [path to the design document]
- test_inventory_count: [number of test inventory rows]
- tdd_task_count: [number of TDD tasks]
- ambiguity_count: [number of unresolved ambiguities, 0 if PASS]
- assumption_count: [number of assumptions made, 0 if none]
```

**重要**：将设计文档写入指定输出路径。编排器期望此 SubAgent 完成后文件存在。
