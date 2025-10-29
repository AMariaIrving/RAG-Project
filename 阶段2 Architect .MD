

### **阶段2: Architect (架构阶段)**

**目标:** 将`ALIGNMENT_stage1.md`中描述的“做什么”，转化为“怎么组织代码来做”。我们要设计出`run_stage_1.py`的内部骨架和蓝图，定义好各个功能模块（函数）以及它们之间的数据流。这份架构设计将直接指导后续的原子化任务拆分和编码工作。

**关键产出:** `DESIGN_stage1.md` 文件。

**您的工作：创建 `DESIGN_stage1.md`**

这份设计文档的核心，是**将宏大的任务分解为若干个高内聚、低耦合的函数**，并清晰地描述它们各自的职责和交互方式。这正是优秀软件工程的体现。

同样地，我们使用一个为您定制的模板来完成这项工作。

---

#### **`DESIGN_stage1.md` 模板**

```markdown
# 架构设计文档 - [任务名: run_stage_1.py - 需求发现与情境化]

## 1. 设计概述 (Design Overview)

本脚本将采用模块化的函数式编程风格。整个执行流程将由一个主函数 `main()` 来协调和驱动。核心逻辑将被分解为三个独立的、职责清晰的子模块（函数），分别对应需求分析的三个主要步骤：信息摘要、情境检索和综合分析。所有配置信息（如文件路径、API密钥）将集中管理，便于维护。

## 2. 系统架构图 (Architecture Diagram)

我们可以使用Mermaid语法来可视化脚本的结构和数据流。

```mermaid
graph TD
    subgraph User Action
        A[执行: python scripts/run_stage_1.py]
    end

    subgraph main() Function
        B[1. 初始化与路径设置] --> C{run_stage_1_workflow}
    end

    subgraph run_stage_1_workflow()
        D[2. ingest_raw_data] --> E[3. summarize_raw_info]
        E --> F[4. retrieve_macro_context]
        F --> G[5. synthesize_final_report]
    end

    subgraph Data & Services
        H[/00_input/ 文件夹] --> D
        E --> I[MOE/LLM API]
        F --> J[RAG API]
        G --> I
        D -- "原始数据" --> E
        E -- "摘要文本" --> F
        F -- "宏观情境" --> G
        E -- "写入" --> K[a_raw_info_summary.txt]
        F -- "写入" --> L[b_macro_context_report.txt]
        G -- "写入" --> M[c_contextualized_req_report.txt]
    end

    A --> B
```

## 3. 模块/函数定义 (Module/Function Definitions)

### 3.1 `main()`
- **职责:** 脚本的入口点。负责调用主工作流函数，并处理全局的异常捕获。
- **输入:** 无
- **输出:** 无 (在控制台打印成功或失败信息)

### 3.2 `ingest_raw_data(input_path: str) -> (dict)`
- **职责:** 实现`ALIGNMENT`文档中的【功能点1】和【功能点2】。负责遍历输入目录，根据文件类型智能提取内容（文本、图片路径、音视频转录）。
- **输入:** `input_path` (字符串, `/00_input/`的路径)
- **输出:** 一个Python字典，结构为 `{'texts': '聚合后的文本内容', 'image_paths': ['路径1', '路径2']}`。

### 3.3 `summarize_raw_info(raw_data: dict, output_path: str) -> str`
- **职责:** 实现`ALIGNMENT`文档中的【功能点3】。负责构建Prompt，调用多模态LLM进行初次摘要，并将结果写入文件。
- **输入:** `raw_data` (字典, `ingest_raw_data`的输出), `output_path` (字符串, `/01_stage_discovery/`的路径)
- **输出:** `summary_text` (字符串, 生成的摘要文本，用于传递给下一个函数)

### 3.4 `retrieve_macro_context(summary_text: str, output_path: str) -> str`
- **职责:** 实现`ALIGNMENT`文档中的【功能点4】。负责从摘要中提取关键词，多次调用RAG，汇总结果并写入文件。
- **输入:** `summary_text` (字符串, `summarize_raw_info`的输出), `output_path` (字符串, `/01_stage_discovery/`的路径)
- **输出:** `context_text` (字符串, 汇总后的宏观情境文本，用于传递给下一个函数)

### 3.5 `synthesize_final_report(summary_text: str, context_text: str, output_path: str) -> None`
- **职责:** 实现`ALIGNMENT`文档中的【功能点5】。负责结合摘要和情境，构建最终的Prompt，调用LLM进行深度分析，并将最终报告写入文件。
- **输入:** `summary_text` (字符串), `context_text` (字符串), `output_path` (字符串, `/01_stage_discovery/`的路径)
- **输出:** 无 (直接写入文件)

## 4. 数据流 (Data Flow)

1. `main()` 启动，定义好输入输出路径。
2. `ingest_raw_data` 读取物理文件，返回一个包含文本和图片路径的内存中的字典 `raw_data`。
3. `summarize_raw_info` 接收 `raw_data`，调用LLM，将结果写入`a_...txt`，并返回摘要文本 `summary_text`。
4. `retrieve_macro_context` 接收 `summary_text`，调用RAG，将结果写入`b_...txt`，并返回情境文本 `context_text`。
5. `synthesize_final_report` 接收 `summary_text` 和 `context_text`，调用LLM，将最终结果写入`c_...txt`。

```

