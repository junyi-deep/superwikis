# 模块设计生成器插件

一个通过代码分析来生成和增强模块设计文档的 Claude Code 插件。

## 功能特点

本插件提供 5 个主要技能及支持技能：

### 主要技能

| 技能 | 命令 | 说明 |
|-----|------|------|
| **增强模块设计** | `/enhance-module-design` | 通过分析代码来增强现有设计文档 |
| **生成项目总览** | `/generate-project-overview` | 生成项目总览和模块索引 |
| **反向生成设计** | `/reverse-generate-design` | 仅通过代码生成设计文档 |
| **批量生成文档** | `/batch-generate-docs` | 批量生成完整的文档集 |
| **根据变更更新设计** | `/update-design-with-changes` | 根据需求变更更新文档 |

### 支持技能

| 技能 | 说明 |
|-----|------|
| **pdf** | PDF 处理 - 提取文本、表格、表单 |
| **docx** | Word 文档处理 - 创建、编辑、分析 |
| **mermaid-diagrams** | 生成 Mermaid 图表（流程图、时序图、类图、ERD、C4） |

## 模板说明

所有设计生成技能支持 4 种内置模板：

1. **普通功能类** - 基础功能、业务逻辑、接口定义
2. **管理系统类** - CRUD 操作、数据流、权限控制、状态管理
3. **算法类** - 算法流程、时间/空间复杂度、边界情况处理
4. **中间件类** - 底层通信、协议处理、性能优化、容错机制

## 安装方式

### 方式一：本地安装

将此插件复制到 Claude Code 插件目录：

```bash
cp -r /path/to/module-design-generator ~/.claude/plugins/module-design-generator
```

### 方式二：从仓库克隆

```bash
git clone https://github.com/your-repo/module-design-generator.git
ln -s /path/to/module-design-generator ~/.claude/plugins/module-design-generator
```

## 使用方法

### 技能一：增强模块设计

```bash
/enhance-module-design
```

流程：
1. 提供原始设计文档（PDF/DOCX/Markdown）
2. 选择代码仓库
3. 选择模板（或自定义）
4. 获取带 Mermaid 图表的增强 Markdown 文档

### 技能二：生成项目总览

```bash
/generate-project-overview
```

流程：
1. 提供多个模块设计文档
2. 选择代码仓库
3. 获取项目总览 + 模块索引

### 技能三：反向生成设计

```bash
/reverse-generate-design
```

流程：
1. 选择代码仓库
2. 选择模板
3. 获取从代码生成的设计文档

### 技能四：批量生成文档

```bash
/batch-generate-docs
```

流程：
1. 提供包含模块设计文档的目录
2. 选择代码仓库
3. 获取完整文档集 + 项目总览

### 技能五：根据变更更新设计

```bash
/update-design-with-changes
```

流程：
1. 提供现有的模块设计文档
2. 提供新的需求文档
3. 配置 git 提交筛选条件
4. 获取更新后的文档及版本历史

## 模板自我增强

所有技能都支持模板自我增强功能：
- 执行后询问用户是否启用自我增强
- 用户可以审阅和修改生成的内容
- 模板会根据用户反馈进行改进

## 输出格式

所有文档均以 Markdown 格式输出，包含：
- Mermaid 图表（流程图、时序图、类图、ERD）
- 清晰的章节结构
- 版本历史表格

默认输出目录：项目根目录下的 `docs/` 文件夹

## 依赖项

- Python 3.x 用于 PDF/DOCX 处理
- 所需包：pypdf、pdfplumber、python-docx

## 许可证

MIT
