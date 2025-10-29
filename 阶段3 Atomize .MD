

### **阶段3: Atomize (原子化阶段)**

**目标:** 将`DESIGN_stage1.md`中定义的每一个高层函数，进一步拆解成**AI可以直接理解并编码的、最小的、可执行的、可验证的原子任务**。这份“任务清单”将是我们下一阶段（Automate）直接喂给Claude的指令。

**关键产出:** `TASK_stage1.md` 文件。

**您的工作：创建 `TASK_stage1.md`**

这份文档的核心是**极度的具体化**。我们要把所有模糊的地方都消除掉，为每一个编程步骤提供清晰的上下文和明确的输入输出。这能最大限度地避免AI在编码时自由发挥，确保它严格按照我们的意图行事。

---

#### **`TASK_stage1.md` 模板**

```markdown
# 原子化任务清单 - [任务名: run_stage_1.py - 需求发现与情境化]

## 1. 任务依赖关系 (Task Dependencies)

```mermaid
graph TD
    T1[任务1: 环境设置与主流程搭建] --> T2[任务2: 实现 ingest_raw_data 函数]
    T2 --> T3[任务3: 实现 summarize_raw_info 函数]
    T3 --> T4[任务4: 实现 retrieve_macro_context 函数]
    T4 --> T5[任务5: 实现 synthesize_final_report 函数]```
*说明: 我们将按照从1到5的顺序，逐个完成这些任务的编码。*

---

## 2. 原子任务详细定义 (Atomic Task Definitions)

### **任务1: 环境设置与主流程搭建**
- **目标:** 创建脚本文件，搭建好基本的代码框架和主函数流程，确保整个脚本的骨架可以运行。
- **输入契约:** `DESIGN_stage1.md` 中的架构图和函数定义。
- **输出契约:**
    - 一个名为 `run_stage_1.py` 的文件。
    - 文件中包含 `main` 函数和所有其他函数的空架子（带类型提示和文档字符串）。
    - 主函数 `main` 中已经写好了调用这些空函数的逻辑流程。
- **执行步骤:**
    1. 创建 `scripts/run_stage_1.py` 文件。
    2. 导入必要的标准库（如 `os`, `pathlib`）。
    3. 根据 `DESIGN_stage1.md`，定义出 `main`, `ingest_raw_data`, `summarize_raw_info`, `retrieve_macro_context`, `synthesize_final_report` 这几个函数的签名（函数名、参数、返回值类型提示），函数体暂时用 `pass` 占位。
    4. 在 `main` 函数中，编写调用这些函数的代码，并用 `print` 语句打印每一步的开始和结束，例如 `print("开始执行数据摄入...")`。
    5. 在文件末尾添加 `if __name__ == "__main__":` 来调用 `main` 函数。

### **任务2: 实现 `ingest_raw_data` 函数**
- **目标:** 编写该函数的完整逻辑，使其能够处理多种文件格式。
- **输入契约:** `input_path` (字符串)。
- **输出契约:** 一个字典 `{'texts': str, 'image_paths': list[str]}`。
- **执行步骤:**
    1. 初始化一个空字符串变量 `aggregated_text` 和一个空列表 `image_paths`。
    2. 使用 `os.walk` 或 `pathlib.rglob` 遍历 `input_path` 下的所有文件。
    3. **对于每个文件:**
        - 获取文件扩展名。
        - **如果**是 `.txt` 或 `.md`，直接读取内容追加到 `aggregated_text`。
        - **如果**是 `.docx`，使用 `python-docx` 库读取内容追加。
        - **如果**是 `.pdf`，使用 `PyMuPDF` 库读取内容追加。
        - **如果**是 `.jpg` 或 `.png`，将其完整路径追加到 `image_paths` 列表。
        - **(高级)** 如果是 `.mp3`，调用 `whisper` API，将转录结果追加到 `aggregated_text`。 (MVP阶段可先用占位符)
    4. 在每个追加的文本前后，添加文档名标记，如 `--- START: filename.txt ---`。
    5. **返回** 最终的字典。

### **任务3: 实现 `summarize_raw_info` 函数**
- **目标:** 实现调用多模态LLM进行初次摘要的逻辑。
- **输入契约:** `raw_data` (字典), `output_path` (字符串)。
- **输出契约:** `summary_text` (字符串)。
- **执行步骤:**
    1. 从 `raw_data` 字典中解包出 `texts` 和 `image_paths`。
    2. **构建Prompt:** 根据我们在`run_stage_1.py`超详细设计中的【步骤2】的Prompt模板，将`texts`和`image_paths`（可能需要将图片转为Base64）组装成一个完整的、符合多模态模型API要求的数据结构。
    3. **调用AI服务:** 编写一个调用MOE/LLM API的辅助函数（可以放在另一个`utils.py`文件中），传入构建好的Prompt。
    4. **处理返回:** 获取LLM返回的摘要文本。
    5. **写入文件:** 将摘要文本写入到 `output_path` 下的 `a_raw_info_summary.txt` 文件中。
    6. **返回** 摘要文本。

### **任务4: 实现 `retrieve_macro_context` 函数**
- **目标:** 实现提取关键词并调用RAG系统的逻辑。
- **输入契约:** `summary_text` (字符串), `output_path` (字符串)。
- **输出契约:** `context_text` (字符串)。
- **执行步骤:**
    1. **提取关键词:**
        - **(MVP方案)** 简单实现：编写一个基础的关键词提取逻辑（例如，基于文本分割或常用词）。
        - **(高级方案)** 调用一个轻量级LLM，传入`summary_text`，让其返回关键词列表。
    2. 初始化一个空列表 `rag_results`。
    3. **循环调用RAG:** 遍历关键词列表，对每个关键词调用RAG系统的API，并将返回的片段追加到 `rag_results`。
    4. **汇总与去重:** 将 `rag_results` 列表中的所有文本片段连接成一个大的字符串，并处理可能的重复内容。
    5. **写入文件:** 将汇总后的文本写入 `output_path` 下的 `b_macro_context_report.txt` 文件中。
    6. **返回** 汇总后的文本。

### **任务5: 实现 `synthesize_final_report` 函数**
- **目标:** 实现最终的综合分析报告生成逻辑。
- **输入契约:** `summary_text` (字符串), `context_text` (字符串), `output_path` (字符串)。
- **输出契约:** 无。
- **执行步骤:**
    1. **构建Prompt:** 根据我们在`run_stage_1.py`超详细设计中的【步骤4】的终极Prompt模板，将`summary_text`和`context_text`作为上下文注入。
    2. **调用AI服务:** 调用强大的MOE/LLM API。
    3. **写入文件:** 将LLM返回的最终报告，直接写入 `output_path` 下的 `c_contextualized_req_report.txt` 文件中。

```
