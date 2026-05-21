# AI NovelGenerator — 智能长篇小说生成器

> **An AI-powered long-form novel generation engine with multi-stage pipeline, vector-based context retrieval, and a full-featured desktop GUI. Designed to help authors produce logically coherent, setting-consistent long stories efficiently.**

---

## 🎯 要解决的问题

长篇小说创作面临三大核心痛点：

1. **长程一致性难以维护** — 几十甚至上百章的篇幅下，角色行为、世界观设定、剧情伏笔极易前后矛盾
2. **上下文管理成本极高** — 传统写作需人工翻阅前文、整理摘要、追踪角色状态线
3. **创作流程缺乏工程化支撑** — 从世界观构建到章节生成，缺乏可复用、可迭代的结构化工具

AI_NovelGenerator 通过 **多阶段流水线 + 向量检索 + 状态追踪** 的技术方案，系统化解决上述问题，将长篇小说创作从"纯手工"升级为"人机协同的工程化流程"。

---

## ✨ 核心功能

| 功能模块 | 关键能力 |
|----------|---------|
| 🏗️ 小说架构工坊 | 四阶段流水线生成：核心种子 → 角色动力学 → 世界观 → 三幕式情节架构 |
| 📐 章节蓝图引擎 | 支持分块生成章节目录，自适应 token 预算，支持中断续传 |
| ✍️ 智能章节生成 | 多源上下文聚合（前文摘要 + 角色状态 + 向量检索 + 知识库），支持人工编辑提示词后再生成 |
| 🔍 向量语义检索 | 基于 Chroma + Embedding 的长程上下文检索，自动切分/去重/过滤近章内容 |
| 🧠 角色状态追踪 | 自动生成并维护角色状态表，每章定稿后自动更新 |
| 📚 外部知识库集成 | 导入本地文档（设定集/参考资料），经 LLM 过滤后融入生成上下文 |
| ✅ 一致性审校 | 定稿前自动检测角色逻辑冲突、剧情矛盾、未解决伏笔 |
| 🖥️ 全功能 GUI | CustomTkinter 构建的多标签工作台，配置/生成/审校/角色库管理一体化 |
| ⚡ 批量生成 | 支持指定章节范围批量生成+自动扩写，支持角色库自动注入 |
| 🔧 多模型编排 | 不同阶段可指定不同 LLM（如架构用 Gemini，正文用 GPT），角色库管理多配置预设 |

---

## 🏗 技术架构

### 整体分层

```
┌─────────────────────────────────────────────────┐
│                   Presentation Layer             │
│   CustomTkinter GUI (10-tab workbench + dialogs) │
│  ui/main_window.py  +  ui/*.py  (9 modules)      │
├─────────────────────────────────────────────────┤
│                  Application Layer               │
│  novel_generator/  (6 modules)                   │
│  • architecture.py  — 四阶段小说架构生成          │
│  • blueprint.py     — 分块章节目录生成            │
│  • chapter.py       — 章节草稿生成（多源上下文）  │
│  • finalization.py  — 定稿处理（摘要/状态/向量）  │
│  • knowledge.py     — 外部知识库导入             │
│  • vectorstore_utils.py — Chroma 向量库管理      │
├─────────────────────────────────────────────────┤
│                 Adapter / Infrastructure Layer   │
│  llm_adapters.py     — 10 种 LLM 后端适配器      │
│  embedding_adapters.py — 6 种 Embedding 适配器   │
│  config_manager.py   — JSON 配置持久化 + 多模型编排│
│  consistency_checker.py — LLM 驱动一致性审校     │
├─────────────────────────────────────────────────┤
│                  Data Layer                      │
│  File-based storage (chapters/, vectorstore/)    │
│  Chroma persistent vector store                  │
│  JSON config (multi-profile LLM presets)         │
└─────────────────────────────────────────────────┘
```

### 核心数据流

```
用户输入主题/类型 → Step1 小说架构 → Step2 章节目录 → Step3 章节草稿 → Step4 定稿
                                              ↓
                                    ┌──────────┴──────────┐
                                    │  多源上下文聚合：     │
                                    │  · 前文摘要           │
                                    │  · 角色状态表         │
                                    │  · 向量检索结果       │
                                    │  · 外部知识库(可选)   │
                                    │  · 用户本章指导       │
                                    └──────────┬──────────┘
                                               ↓
                                    Step4 定稿处理：
                                    · 更新全局摘要
                                    · 更新角色状态
                                    · 写入向量库
                                    · 一致性审校(可选)
```

---

## 🛠 技术亮点

### 1. 多模型编排架构（Multi-Model Orchestration）

不同生成阶段可独立配置 LLM，充分利用各模型优势：

```json
"choose_configs": {
  "architecture_llm": "Gemini 2.5 Pro",    // 架构设计 → 强推理
  "chapter_outline_llm": "DeepSeek V3",    // 目录规划 → 高性价比
  "prompt_draft_llm": "DeepSeek V3",       // 正文草稿 → 高吞吐
  "final_chapter_llm": "GPT 5",            // 定稿润色 → 高质量
  "consistency_review_llm": "DeepSeek V3"  // 审校检查 → 低成本
}
```

支持 **10 种 LLM 后端**（OpenAI / DeepSeek / Gemini / Azure OpenAI / Azure AI / Ollama / LM Studio / 火山引擎 / 硅基流动 / Grok），通过策略模式的 `BaseLLMAdapter` 统一接口，切换零成本。

### 2. 向量语义检索 + 智能上下文过滤

长程一致性是长篇小说的核心挑战。项目引入基于 **Chroma + Embedding** 的向量检索系统，并设计了多层过滤策略：

- **近章内容自动跳过** — 3 章内内容自动过滤，避免重复
- **中期内容强制改写** — 3-5 章内容要求 ≥40% 改写
- **远章内容优先引用** — 5 章前内容可引用核心设定
- **知识库标签分类** — 自动识别 `[TECHNIQUE]` / `[SETTING]` / `[GENERAL]` 上下文
- **LLM 二次过滤** — 经知识过滤器提炼后注入提示词

### 3. 四阶段小说架构生成（带断点续传）

小说架构生成采用串行四阶段流水线，每阶段结果独立持久化：

```
Step 1: 核心种子 (core_seed) ──→ partial_architecture.json
Step 2: 角色动力学 (character_dynamics) ──→ partial_architecture.json
Step 2.5: 初始角色状态 (character_state) ──→ character_state.txt
Step 3: 世界观 (world_building) ──→ partial_architecture.json
Step 4: 三幕式情节 (plot_architecture) ──→ Novel_architecture.txt
```

任一阶段失败可自动保存中间状态，下次启动从断点继续。

### 4. 分块章节目录生成（Adaptive Chunking）

章节目录生成根据 `max_tokens` 自动计算分块大小：

```python
chunk_size = (floor(max_tokens / 200) 向下取整到10的倍数) - 10
```

- 章节数 ≤ chunk_size：一次性生成
- 章节数 > chunk_size：分块生成，每块增量追加到文件
- 支持中断续传：解析已有章节号，从下一章继续
- 已有目录仅保留最近 100 章，避免 prompt 超长

### 5. 角色库系统

支持按分类管理角色档案（`.txt` 文件），生成章节时自动注入角色详情到提示词，替代占位符，提升生成质量。

### 6. GUI 交互增强

- **可编辑提示词预览** — 生成草稿前弹出提示词对话框，用户可编辑/查看字数统计后确认
- **角色导入窗口** — 分类复选框选择角色，一键注入本章参与角色
- **批量生成** — 指定起止章节 + 期望字数 + 自动扩写阈值
- **异步任务处理** — 所有 LLM 调用均在独立线程执行，UI 不阻塞
- **配置多预设管理** — 支持保存/加载多组 LLM 配置

---

## 📊 成果与进展

### 已实现能力

- **完整的多阶段生成流水线**：从主题输入到章节定稿，全流程覆盖
- **10 种 LLM 后端 + 6 种 Embedding 后端**：涵盖主流云服务与本地部署方案
- **向量检索长程上下文管理**：基于 Chroma 的语义检索 + 多层过滤策略
- **角色状态自动维护**：每章定稿后自动更新角色状态表
- **一致性审校机制**：LLM 驱动的剧情冲突检测
- **全功能桌面 GUI**：10 个标签页 + 角色库 + 批量生成 + 配置管理
- **外部知识库集成**：支持导入本地文档作为生成参考
- **可编辑提示词**：用户对 AI 生成的提示词有最终编辑权
- **PyInstaller 打包支持**：可编译为独立可执行文件

### 技术规模

| 指标 | 数值 |
|------|------|
| Python 源文件 | 20+ |
| 代码总量 | 约 6,000+ 行 |
| UI 模块 | 10 个独立标签页 |
| LLM 适配器 | 10 种后端 |
| Embedding 适配器 | 6 种后端 |
| 提示词模板 | 20+ 个精心设计的 prompt |
| 外部依赖 | 140+ Python 包 |

---

## 📝 反思与成长

### 做得好的地方

1. **适配器模式的成功应用** — `BaseLLMAdapter` / `BaseEmbeddingAdapter` 使多后端切换零成本，新增提供商只需实现一个子类
2. **状态持久化设计** — `partial_architecture.json` 的断点续传机制有效应对长耗时任务的稳定性问题
3. **向量检索的多层过滤** — 不只是简单的 similarity_search，而是设计了时间衰减 + 标签分类 + LLM 二次过滤的三层策略
4. **用户对 AI 的控制权** — 可编辑提示词设计让用户不是被动接受 AI 输出，而是有最终编辑权

### 待改进的方向

1. **配置管理耦合度过高** — `main_window.py` 中状态变量直接散落在 GUI 类中，应抽象为独立的状态管理层
2. **硬编码配置引用** — `generation_handlers.py` 中大量 `self.loaded_config["llm_configs"][xxx]` 嵌套访问，应封装为配置解析器
3. **缺少流式输出** — 当前 LLM 调用为阻塞式等待完整响应，用户体验上缺少实时反馈，后续应接入 SSE 流式渲染
4. **向量库持久化策略简单** — 当前使用 Chroma 本地文件存储，缺乏版本管理和回滚能力
5. **错误处理不够精细** — 多处 `try/except` 仅打印 traceback，缺少分级错误码和用户友好的恢复建议
6. **测试覆盖缺失** — 除 `test_all_features.py` 外缺少系统化单元测试，核心逻辑（如 prompt 构造、向量检索）应补充测试
7. **计划重构** — 原计划采用更先进的技术栈（如 Web 界面 + 流式输出 + 更精细的上下文管理）进行架构升级

---

## 🚀 快速开始

### 环境要求

- Python 3.9+（推荐 3.10–3.12）
- pip 包管理工具
- 有效 API 密钥（云服务或本地 Ollama）

### 安装与运行

```bash
# 克隆项目
git clone https://github.com/YILING0013/AI_NovelGenerator
cd AI_NovelGenerator

# 安装依赖
pip install -r requirements.txt

# 启动 GUI
python main.py
```

### 打包为可执行文件

```bash
pip install pyinstaller
pyinstaller main.spec
# 输出位于 dist/main.exe
```

---

## 📁 项目结构

```
AI_NovelGenerator/
├── main.py                      # 应用入口，启动 CustomTkinter GUI
├── llm_adapters.py              # 10 种 LLM 后端适配器（策略模式）
├── embedding_adapters.py        # 6 种 Embedding 后端适配器
├── config_manager.py            # JSON 配置加载/保存/多模型编排
├── consistency_checker.py       # LLM 驱动的一致性审校
├── prompt_definitions.py        # 20+ 个 AI 提示词模板
├── chapter_directory_parser.py  # 章节目录解析器
├── utils.py                     # 文件操作工具函数
├── tooltips.py                  # GUI 参数说明
├── novel_generator/             # 核心生成逻辑
│   ├── architecture.py          # 四阶段小说架构生成（断点续传）
│   ├── blueprint.py             # 分块章节目录生成（自适应 chunking）
│   ├── chapter.py               # 章节草稿生成（多源上下文聚合）
│   ├── finalization.py          # 定稿处理（摘要/状态/向量更新）
│   ├── knowledge.py             # 外部知识库导入
│   └── vectorstore_utils.py     # Chroma 向量库管理
├── ui/                          # GUI 界面模块（10 个标签页）
│   ├── main_window.py           # 主窗口布局与事件处理
│   ├── config_tab.py            # LLM/Embedding 配置面板
│   ├── novel_params_tab.py      # 小说参数配置
│   ├── generation_handlers.py   # 生成/审校/批量操作事件处理
│   ├── role_library.py          # 角色库管理（分类/搜索/注入）
│   ├── setting_tab.py           # 小说设定展示/编辑
│   ├── directory_tab.py         # 章节目录展示/编辑
│   ├── character_tab.py         # 角色状态展示/编辑
│   ├── summary_tab.py           # 全局摘要展示/编辑
│   ├── chapters_tab.py          # 章节列表/编辑器
│   └── other_settings.py        # 其他设置
├── requirements.txt             # 140+ Python 依赖
└── main.spec                    # PyInstaller 打包配置
```

---

## 📄 License

This project is licensed under the [Apache 2.0 License](LICENSE).

---

<div align="center">
  <em>Built with Python, LangChain, Chroma, CustomTkinter, and a passion for AI-powered creative writing.</em>
</div>
