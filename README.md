# 智扫通机器人智能客服

基于 LangChain ReAct Agent + RAG 的扫地机器人智能客服系统，支持知识库问答与个性化使用报告生成。

## 项目简介

本项目构建了一个具备自主思考与工具调用能力的 AI 智能客服，专为扫地/扫拖一体机器人场景设计。系统采用 ReAct（Reasoning + Acting）范式，结合 RAG 检索增强生成技术，能够从本地知识库中检索专业资料回答用户问题，并根据外部数据为指定用户生成个性化的机器人使用情况报告。

## 技术栈

| 模块 | 技术选型 |
|------|----------|
| **UI 框架** | Streamlit |
| **Agent 框架** | LangChain (create_agent) |
| **大语言模型** | 通义千问 Qwen3-Max (DashScope) |
| **嵌入模型** | DashScope text-embedding-v4 |
| **向量数据库** | Chroma |
| **文本分割** | LangChain RecursiveCharacterTextSplitter |
| **日志** | Python logging（文件 + 控制台） |

## 项目结构

```
first_agent/
├── app.py                      # Streamlit 前端入口
├── agent/
│   ├── react_agent.py          # ReAct Agent 定义
│   └── tools/
│       ├── agent_tools.py      # 工具定义（RAG检索、天气、用户信息等）
│       └── middleware.py       # 中间件（工具监控、日志、动态提示词切换）
├── rag/
│   ├── rag_service.py          # RAG 总结服务
│   └── vector_store.py         # Chroma 向量库管理
├── model/
│   └── factory.py              # 模型工厂（ChatModel + Embeddings）
├── config/
│   ├── agent.yml               # Agent 配置
│   ├── chroma.yml              # Chroma 向量库配置
│   ├── prompts.yml             # 提示词路径配置
│   └── rag.yml                 # RAG 模型配置
├── prompts/
│   ├── main_prompt.txt         # 主系统提示词
│   ├── rag_summarize.txt       # RAG 总结提示词
│   └── report_prompt.txt       # 报告生成提示词
├── utils/
│   ├── config_handler.py       # 配置加载
│   ├── file_handler.py         # 文件处理（PDF/TXT 加载）
│   ├── logger_handler.py       # 日志处理
│   ├── path_tool.py            # 路径工具
│   └── prompt_loader.py        # 提示词加载
├── data/
│   ├── 扫地机器人100问.pdf     # 知识库文件
│   ├── 扫拖一体机器人100问.txt
│   ├── 扫地机器人100问2.txt
│   ├── 故障排除.txt
│   ├── 维护保养.txt
│   ├── 选购指南.txt
│   └── external/
│       └── records.csv         # 用户使用记录数据
└── chroma_db/                  # Chroma 向量持久化目录
```

## 核心功能

### 1. RAG 知识库问答
- 支持 PDF / TXT 格式知识文档自动加载与向量化
- 基于 MD5 去重，避免重复加载
- 自动文本分割（可配置 chunk_size / chunk_overlap）
- 从向量库检索相关参考资料后由 LLM 总结回答

### 2. ReAct Agent 智能工具调用
Agent 可自主调用以下工具：

| 工具 | 功能 |
|------|------|
| `rag_summarize` | 从向量库检索扫地机器人专业知识 |
| `get_weather` | 获取指定城市天气信息 |
| `get_user_location` | 获取当前用户所在城市 |
| `get_user_id` | 获取当前用户 ID |
| `get_current_month` | 获取当前月份 |
| `fetch_external_data` | 获取用户使用记录 |
| `fill_context_for_report` | 触发报告生成上下文 |

### 3. 个性化使用报告生成
- 自动识别报告生成意图
- 按固定流程：获取用户 ID → 获取月份 → 注入报告上下文 → 检索使用数据
- 中间件动态切换提示词，从客服模式无缝过渡到报告撰写模式
- 输出结构化 Markdown 报告，包含使用情况分析与保养建议

### 4. 中间件系统
- **工具调用监控**：记录每次工具调用的名称、参数与结果
- **模型调用日志**：记录模型输入消息数量与内容
- **动态提示词切换**：根据上下文自动在客服/报告提示词间切换

## 快速开始

### 环境要求
- Python 3.10+
- 阿里云 DashScope API Key

### 安装

```bash
# 克隆仓库
git clone https://github.com/your-username/SmartSweep-RAG-Agent.git
cd SmartSweep-RAG-Agent

# 安装依赖
pip install -r requirements.txt
```

### 配置

1. 设置 DashScope API Key 环境变量：
```bash
export DASHSCOPE_API_KEY="your-api-key"
```

2. （可选）调整 `config/` 目录下的配置文件，修改模型名称、向量库参数、文档分块策略等。

### 初始化知识库

首次运行前需加载知识文档到向量库：

```python
from rag.vector_store import VectorStoreService

vs = VectorStoreService()
vs.load_document()
```

### 启动应用

```bash
streamlit run app.py
```

浏览器访问 `http://localhost:8501` 即可使用。

## 配置说明

### `config/rag.yml`
- `chat_model_name`: 对话模型名称（默认 `qwen3-max`）
- `embedding_model_name`: 嵌入模型名称（默认 `text-embedding-v4`）

### `config/chroma.yml`
- `collection_name`: 向量集合名称
- `persist_directory`: 向量持久化目录
- `k`: 检索返回文档数
- `chunk_size` / `chunk_overlap`: 文本分块大小与重叠
- `data_path`: 知识文档存放路径
- `allow_knowledge_file_type`: 允许加载的文件类型

## 知识库数据

项目内置了以下扫地机器人领域知识文档：
- 扫地机器人 100 问（PDF + TXT）
- 扫拖一体机器人 100 问
- 故障排除指南
- 维护保养指南
- 选购指南

可自行在 `data/` 目录下添加更多 TXT/PDF 文档扩展知识库。

## GitHub 仓库命名建议

以下是一些适合本项目的 GitHub 仓库名称：

| 名称 | 风格 |
|------|------|
| **SmartSweep-RAG-Agent** | 英文，突出技术栈 |
| **ZhiSaoTong-Chatbot** | 保留品牌名 "智扫通" |
| **RAG-ReAct-SweepingRobot** | 技术向，描述准确 |
| **SweepingRobot-Intelligent-CS** | 领域 + 应用场景 |
| **Tongyi-RAG-CustomerBot** | 模型 + 技术 + 场景 |

> 推荐使用 **SmartSweep-RAG-Agent**，简洁且清晰传达了项目定位。
