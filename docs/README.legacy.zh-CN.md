# Local Agent - 本地知识库智能体

一个可本地运行的单智能体系统，基于 **Ollama 本地大模型** + **Chroma 向量知识库** + **FastAPI 后端** + **Vue3 前端**，支持网页访问与局域网共享，界面风格参考 COZE。

## 核心特性

- **本地大模型**：接入 Ollama，默认对话模型 `gemma4:e4b`，Embedding 模型 `nomic-embed-text`
- **本地知识库**：
  - 优先使用 `backend/data/knowledge_sources/` 目录中的本地文件建立索引
  - 同时保留临时上传入口，支持 PDF、Word、TXT、Markdown、PPT、Excel、HTML 等格式
  - 自动解析、切片、向量化
  - **扫描版 PDF 自动 OCR**：无文本层的页面自动渲染成图片，由内置 RapidOCR（PaddleOCR ONNX 模型）离线识别，无需联网
  - **后台任务处理**：解析/向量化在后台线程执行，前端实时显示页级进度，大文件处理期间可正常使用其他功能
- **高质量检索**：向量相似度 + BM25 混合检索 + 重排序开关
- **多技能 Agent**：通过 YAML 技能文件配置不同的系统提示词、工具与参数，当前内置“通用助手”与“麻醉术后访视助手”
- **患者与随访档案**：创建患者档案、关联随访会话、持久化对话与结构化数据
- **结构化报告**：麻醉术后访视流程完成后自动生成 Word（DOCX）报告并支持下载
- **COZE 风格 UI**：Vue3 + Tailwind CSS，支持对话、患者档案、随访记录、知识库管理、设置五模块
- **模型预热**：启动时后台加载对话模型，前端通过 `/api/health/warmup` 实时显示就绪状态
- **局域网访问**：启动后同一局域网内其他设备可通过 `http://<本机IP>:8000` 访问

## 项目结构

```
local_agent/
├── backend/              # FastAPI 后端
│   ├── app/
│   │   ├── api/          # API 路由
│   │   ├── core/         # 配置、依赖容器、技能管理
│   │   ├── models/       # Pydantic 数据模型
│   │   ├── rag/          # 文档处理、Embedding、向量库、检索器
│   │   ├── services/     # Ollama、患者、会话、报告、搜索服务
│   │   └── skills/       # YAML 技能配置
│   ├── data/             # 本地数据目录
│   │   ├── knowledge_sources/  # 本地知识库文件目录
│   │   ├── vector_db/          # Chroma 向量库
│   │   ├── patients.json       # 患者档案
│   │   ├── sessions/           # 随访会话 JSON
│   │   └── reports/            # 生成的 Word 报告
│   ├── requirements.txt
│   └── venv/             # Python 虚拟环境
├── frontend/             # Vue3 前端
│   ├── src/
│   │   ├── views/        # 页面视图
│   │   ├── components/   # 公共组件
│   │   ├── stores/       # Pinia 状态管理
│   │   └── router.js     # 路由配置
│   ├── dist/             # 构建产物
│   └── package.json
├── start.bat / start.ps1 / start.sh   # Windows / PowerShell / Linux&Mac 启动脚本
├── stop.bat / stop.ps1 / stop.sh      # 停止后端脚本
└── README.md
```

## 环境要求

- Windows 10/11 或 Linux/macOS
- Python 3.10+
- Node.js 18+
- Ollama 已安装并运行
- 至少 16GB 内存（GPU 建议 8GB+ 显存；CPU 也可运行）

## 快速开始

### 1. 进入项目

```bash
cd local_agent
```

### 2. 确保 Ollama 已安装所需模型

```bash
ollama pull gemma4:e4b
ollama pull nomic-embed-text
```

### 3. 安装后端依赖（如未安装）

```bash
cd backend
python -m venv venv
# Windows
venv\Scripts\activate
# Linux/Mac
source venv/bin/activate
pip install -r requirements.txt
cd ..
```

### 4. 安装并构建前端（如未构建）

```bash
cd frontend
npm install
npm run build
cd ..
```

### 5. 启动服务

**Windows：**
```cmd
start.bat
```
或 PowerShell：
```powershell
.\start.ps1
```

**Linux/Mac：**
```bash
bash start.sh
```

启动脚本会自动：
1. 检查并启动 Ollama 服务
2. 检查并拉取 `nomic-embed-text` 模型
3. 释放 8000 端口
4. 启动后端服务
5. 打开浏览器（`start.bat`）

### 6. 访问

- 本机：`http://localhost:8000`
- 局域网：`http://<本机IP>:8000`

### 7. 停止服务

运行对应停止脚本关闭后端进程：

```cmd
stop.bat
```

```powershell
.\stop.ps1
```

```bash
bash stop.sh
```

## 使用说明

### 对话

1. 进入左侧「对话」页面
2. 等待顶部状态栏显示“就绪”（启动时模型需要预热）
3. 勾选/取消「使用知识库」或「联网搜索」
4. 选择或新建患者档案（可选，主要用于术后访视工作流）
5. 输入问题，按 Enter 发送
6. 回答会展示引用的知识库来源或网络来源；访视流程结束后可下载 Word 报告

### 知识库管理

1. 进入「知识库」页面
2. 将文件放入页面显示的本地目录 `backend/data/knowledge_sources/`，刷新后点击「处理全部」或单个「处理」
3. 处理在后台进行，文件下方实时显示进度（解析页数、OCR 页数、向量入库），期间可切换页面
4. 已索引的文件会被「处理全部」自动跳过，需要重建时点击该文件的「重新处理」
5. 扫描版 PDF（无文本层）会自动逐页 OCR 识别，速度约 3~10 秒/页，数百页的扫描书需要较长时间，请耐心等待
6. 也支持拖拽临时上传单个文件进行测试
7. 支持 PDF、DOCX、TXT、MD、PPTX、XLSX、HTML 等格式
8. 可删除单个文档或清空全部知识库

### 患者档案与会话

1. 进入「患者档案」页面新建或编辑患者
2. 在「对话」页面选择患者后点击「新建随访」开始一次随访
3. 进入「随访记录」可查看历史会话、继续对话或下载已生成的报告

### 设置

1. 进入「设置」页面
2. 调整系统提示词、Temperature、最大 Token、检索条数、切片参数、Embedding 提供方等
3. 保存后立即生效

## 配置说明

可通过后端 `backend/.env` 文件或环境变量修改：

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `OLLAMA_BASE_URL` | `http://127.0.0.1:11434` | Ollama 服务地址 |
| `OLLAMA_MODEL` | `gemma4:e4b` | 对话大模型 |
| `OLLAMA_EMBEDDING_MODEL` | `nomic-embed-text` | Embedding 模型 |
| `EMBEDDING_PROVIDER` | `ollama` | `ollama` 或 `huggingface` |
| `HUGGINGFACE_EMBEDDING_MODEL` | `BAAI/bge-small-zh-v1.5` | HuggingFace Embedding 模型 |
| `CHUNK_SIZE` | `512` | 文档切片大小（字符） |
| `CHUNK_OVERLAP` | `128` | 切片重叠（字符） |
| `TOP_K` | `5` | 检索条数 |
| `RERANK_TOP_K` | `3` | 重排序后返回条数 |
| `SCORE_THRESHOLD` | `0.5` | 相似度阈值 |
| `ENABLE_RERANK` | `true` | 是否启用重排序 |
| `TEMPERATURE` | `0.3` | 模型温度 |
| `MAX_TOKENS` | `2048` | 最大生成长度 |
| `SYSTEM_PROMPT` | — | 默认系统提示词 |

### Embedding 调优环境变量

在部分 Windows/CUDA 环境下，`nomic-embed-text` 使用 GPU 可能出现内部服务崩溃。后端已默认对 Embedding 请求使用 CPU 并分批发送，可通过环境变量覆盖：

| 环境变量 | 默认值 | 说明 |
|----------|--------|------|
| `OLLAMA_EMBEDDING_NUM_GPU` | `0` | Embedding 使用的 GPU 层数，`0` 表示 CPU |
| `OLLAMA_EMBEDDING_BATCH_SIZE` | `64` | 每批 Embedding 请求的最大文本数 |

## 技能（Skills）

技能配置位于 `backend/app/skills/*.yaml`，包含以下字段：

```yaml
id: anesthesia_followup
name: 麻醉术后访视助手
description: ...
icon: stethoscope
is_default: true
welcome_message: 您好！...
tools:
  - knowledge_base
  - web_search
system_prompt: |
  你是一名...
parameters:
  temperature: 0.2
  max_tokens: 2048
  top_k: 8
  chunk_size: 512
  chunk_overlap: 128
  enable_rerank: true
```

新增技能只需在该目录新增 YAML 文件并重启后端即可。

## 麻醉术后访视工作流

内置 `anesthesia_followup` 技能实现了一个严格分阶段的状态机：

1. **结构化信息采集 + MEWS 评分**
2. **QoR-15 第一轮（项目 1-5）**
3. **QoR-15 第二轮（项目 6-10）**
4. **QoR-15 第三轮（项目 11-15，反向计分）**
5. **麻醉并发症筛查**
6. **伤口快速评估**
7. **结构化报告生成**

每轮仅询问当前阶段问题，系统会自动提取分数、计算总分，并在最后生成包含患者信息、MEWS、QoR-15、并发症、伤口评估与临床建议的 Word 报告。

## 技术栈

- **后端**：Python 3.11 + FastAPI + Uvicorn + Pydantic + LangChain + Chroma
- **前端**：Vue 3 + Vite + Vue Router + Pinia + Tailwind CSS + Axios + marked + lucide-vue-next
- **模型服务**：Ollama
- **文档处理**：PyMuPDF、python-docx、openpyxl、python-pptx
- **报告生成**：python-docx

## 注意事项

1. **首次启动较慢**：加载 `gemma4:e4b` 需要一定时间，前端会显示预热状态。
2. **Ollama 地址**：若 Ollama 不在本机，请修改 `.env` 中的 `OLLAMA_BASE_URL`。
3. **IPv6 问题**：部分环境下 `localhost` 解析到 IPv6 会导致 Ollama 502 错误，默认已改为 `127.0.0.1`。
4. **Embedding 模型**：默认使用 Ollama 的 `nomic-embed-text`；如遇到 GPU 相关问题，已默认切换为 CPU 执行。
5. **知识库目录**：正式使用建议将文档放入 `backend/data/knowledge_sources/`，避免临时上传后文件重复。

## License

MIT
