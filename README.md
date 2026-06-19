## Hi there 👋 我是 南志锦

[![NVIDIA SkillSpector](https://img.shields.io/badge/NVIDIA-SkillSpector-76B900?style=flat&logo=nvidia)](https://github.com/NVIDIA/SkillSpector/pull/100)
[![GitHub stars](https://img.shields.io/github/stars/nanzhijin?style=flat&logo=github)](https://github.com/nanzhijin)

**推荐系统 & AI Agent 工程师 · 主攻召回架构与 Agent 系统设计**

- 🔭 **正在做** — 社交图召回系统工程化落地 & [NVIDIA SkillSpector](https://github.com/NVIDIA/SkillSpector/pull/100) 并发批处理引擎向上游提交
- 🌱 **正在学** — 推荐系统全链路（王喆《深度学习推荐系统》）· ML 基础冲刺（回归→树模型→聚类→神经网络）· Spark 实战
- 💬 **可以聊** — 图神经网络召回 · AI Agent 架构 · 并发系统设计 · FAISS 向量检索 · LightGBM 特征工程
- 📫 **联系我** — n19931465818@outlook.com
- ⚡ **Fun fact** — 会写七言绝句，"些许风霜些许愁，无足之鸟不回头。青霄有壁盘云上，鹭海鲸涛几个丘。"

---

### 🛠 Tech Stack

**模型 / 召回** — PyTorch · PyG (GraphSAGE/GAT) · LightGBM · sklearn · FAISS (ANN)

**Agent / LLM** — LangChain · LangGraph · 多模型实时推理 (DeepSeek) · Tool Calling · RAG 向量检索

**数据 / 特征** — Spark SQL / DataFrame · SQL (窗口函数 / 复杂查询) · Pandas

**工程 / 部署** — Git · Linux · FastAPI / gunicorn · 自购服务器运维 · 模型热加载 & 版本对齐

---

### 📦 Featured Projects

#### 🔗 社交 u2u 召回引擎 · 双通道互补架构
*109K nodes / 603K edges · 阿里电商社交图 · CAAI-BDSC 2023*

> **热连接：** 共有邻居 → LightGBM → MRR@5=0.74, HitRate@5=0.89  
> **冷探索：** GNN GraphSAGE → MRR@5=0.56，补齐零交互用户盲区

全链路闭环：行为日志清洗 → 时间切分构图 → 负采样(1:3) → 240万训练pair → 离线评测 → 模型序列化。产出 A/B 上线方案 (正交分流 / 指标树 / 最小样本量估算)

#### 🦊 NaNaGi · AI Agent 实时推理平台
*[nanagi.cn](https://nanagi.cn) 已上线 · Fork 即用 · DeepSeek V4 Pro 驱动*

- **三层人格架构 —** 情绪参与推理 (六维情绪模型) + 关系锚定价值 (Bowlby 依恋理论 IWM 图结构 v5.1) + 安全依恋降低反驳，自研关系型 Agent 设计 (ETCLOVG 七层审计)
- **双路径记忆系统 —** 同伴路径 (SIM-RAG 向量检索，情绪驱动) + 工作路径 (IterResearch 迭代搜索，任务驱动)，读写分离 (Letta 式文件写入 / Mem0 式向量检索)
- **多模型实时推理 —** LLM 对话 + 混元生图 + 唱片机音频，含 Tool Calling 6 工具链、ReAct 推理循环
- FastAPI + gunicorn 自购服务器部署，多账户记忆隔离，SMTP 邮件预配，跨平台 (Windows/macOS)

#### 🎵 音频内容召回 · 双模态表征 + FAISS
*面向播客 / 音乐冷启动 item*

- **声学通道：** Mel-spectrogram CNN → 128d embedding
- **语义通道：** BERT → 384d text embedding  
- Audio Recall@5 = **95.85%** · Joint Recall@5 = **96.14%**
- 数据驱动决策：Audio 单路已够用、Joint 增量不显著故不上

---

### 🔧 Engineering

独立走通 清洗→构图→训练→评测→序列化→inference 全链路。

**并发系统设计 —** 为 [NVIDIA SkillSpector](https://github.com/NVIDIA/SkillSpector/pull/100) 独立开发 contrib 批处理引擎 (PR #100)，三层并发架构 (ThreadPoolExecutor × asyncio.Semaphore × 20-analyzer fan-out)，K8s 风格 API Key 池 (指数退避自动故障转移)，修复多线程 instance attribute 竞态条件，零行改动上游代码，跨平台兼容 Windows/macOS。

**典型踩坑 —** label-leakage 防护 · O(N²)→O(1) 候选过滤重构 · 图采样 ID mapping 一致性 · 跨模型 embedding 版本对齐

---

*从 GNN 召回到 Agent 推理，从模型训练到自购服务器部署，独立走通全链路。下一步：把社交召回 + 音频召回接入 NaNaGi 成为 Agent 可调用的 recall 插件。*
