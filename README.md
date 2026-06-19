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

#### 🔗 社交 u2u 召回引擎 · 双路线互补 + 三模型路由
*109K nodes / 603K edges · 阿里电商社交图 · CAAI-BDSC 2023 · [Repo](https://github.com/nanzhijin/GNN)*

**LightGBM 路线 — 五版本系统消融 (A→E)**

| 模型 | 特征组 | AUC | MRR@5 | 关键发现 |
|------|--------|-----|-------|----------|
| **LGB-A** | BASE (113d) | **0.8957** | 0.5523 | 不含朋友圈，全局 MRR 最优 |
| LGB-B | +EDGE (119d) | 0.8606 | 0.4291 | 朋友圈特征反降 AUC |
| LGB-D | +TEMPORAL+CATEGORY | 0.9479 | 0.3650 | ⚠️ rank 特征注水 AUC |
| **LGB-E** ★ | -rank (119d) | 0.8840 | 0.4482 | 去注水后 HITS@5=0.76 |

**GNN 路线 — GraphSAGE + DySAT 对比**

| 模型 | 架构 | AUC | MRR@5 | 定位 |
|------|------|-----|-------|------|
| **GNN-A** | SAGE + ItemEncoder | 0.9853 | 0.3517 | 陌生人专家 |
| **GNN-B** ★ | +Extra 6d (时序+品类) | **0.9889** | **0.3824** | 全局门卫 |
| DySAT + Item | +Temporal Attention | 0.9790 | 0.3111 | 时序信号弱，负优化 |

**MRR@5 分场景拆解 — 朋友 74.6% / 陌生人 25.4%**

| 场景 | LGB-A | LGB-E | GNN-A | GNN-B |
|------|:-----:|:-----:|:-----:|:-----:|
| 朋友组 (Seen) | **0.74** | 0.52 | 0.28 | 0.34 |
| 陌生人组 (Unseen) | 0.00 | 0.23 | **0.56** | 0.52 |

> **最优方案：** 朋友用 LGB-A (MRR 0.74) + 陌生人用 GNN-A (MRR 0.56)，理论组合 MRR ≈ 0.69。GNN-B 下沉为全链路粗筛门卫 (AUC 0.989)，过滤低概率候选再送精排。

**全链路闭环：** 行为日志清洗 → 时间切分构图 → 四组特征工程 (BASE/EDGE/TEMPORAL/CATEGORY) → 负采样 1:3 (240万训练 / 48万验证) → 模块化实验框架 (FeatureSelector × ExperimentRunner 交叉组合) → 三模型可加载 artifact → AutoDL RTX 5090 云部署验证

**A/B 实验设计：** 从"调参工具"三轮推翻重建为完整实验设计书 — 正交分流 / 三层指标树 (曝光→click→follow) / 最小样本量估算 / 置换检验 + Bootstrap 离线层 + 真实施部署线上层

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
