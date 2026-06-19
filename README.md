## Hi there 👋 我是 南志锦

[![NVIDIA SkillSpector](https://img.shields.io/badge/NVIDIA-SkillSpector-76B900?style=flat&logo=nvidia)](https://github.com/NVIDIA/SkillSpector/pull/100)
[![GitHub stars](https://img.shields.io/github/stars/nanzhijin?style=flat&logo=github)](https://github.com/nanzhijin)

**推荐系统算法工程师 · 主攻多路召回架构与模型落地**

- 🔭 **正在做** — 社交图召回系统的工程化落地 & [NVIDIA SkillSpector](https://github.com/NVIDIA/SkillSpector/pull/100) 并发批处理引擎向上游提交
- 🌱 **正在学** — 推荐系统全链路（王喆《深度学习推荐系统》）· ML 基础冲刺（回归→树模型→聚类→神经网络）· Spark 实战
- 💬 **可以聊** — 图神经网络召回 · 并发系统设计 · LightGBM 特征工程 · FAISS 向量检索 · AI Agent 安全
- 📫 **联系我** — n19931465818@outlook.com
- ⚡ **Fun fact** — 会写七言绝句，"些许风霜些许愁，无足之鸟不回头。青霄有壁盘云上，鹭海鲸涛几个丘。"

---

### 🛠 Tech Stack

**模型 / 召回** — PyTorch · PyG (GraphSAGE/GAT) · LightGBM · sklearn · FAISS (ANN)

**数据 / 特征** — Spark SQL / DataFrame · SQL (窗口函数 / 复杂查询) · Pandas

**工程 / 部署** — Git · Linux · FastAPI / gunicorn · 模型序列化→inference 校验闭环

---

### 📦 Featured Projects

#### 🔗 社交 u2u 召回引擎 · 双通道互补架构
*109K nodes / 603K edges · 阿里电商社交图 · CAAI-BDSC 2023*

> **热连接：** 共有邻居 → LightGBM → MRR@5=0.74, HitRate@5=0.89  
> **冷探索：** GNN GraphSAGE → MRR@5=0.56，补齐零交互用户盲区

全链路闭环：行为日志清洗 → 时间切分构图 → 负采样(1:3) → 240万训练pair → 离线评测 → 模型序列化。产出 A/B 上线方案 (正交分流 / 指标树 / 最小样本量估算)

#### 🎵 音频内容召回 · 双模态表征 + FAISS
*面向播客 / 音乐冷启动 item*

- **声学通道：** Mel-spectrogram CNN → 128d embedding
- **语义通道：** BERT → 384d text embedding  
- Audio Recall@5 = **95.85%** · Joint Recall@5 = **96.14%**
- 数据驱动决策：Audio 单路已够用、Joint 增量不显著故不上

#### 🚀 nanagi.cn · 多路召回 Serving 原型

统一接口 `recall(user_id, top_k) → List[Candidate]`，router 按入口路由到对应召回通道。FastAPI + gunicorn 部署，含模型热加载、embedding 版本对齐、/health 探活。

---

### 🔧 Engineering

独立走通 清洗→构图→训练→评测→序列化→inference 全链路。

**并发系统设计 —** 为 [NVIDIA SkillSpector](https://github.com/NVIDIA/SkillSpector/pull/100) 独立开发 contrib 批处理引擎 (PR #100)，三层并发架构 (ThreadPoolExecutor × asyncio.Semaphore × 20-analyzer fan-out)，K8s 风格 API Key 池 (指数退避自动故障转移)，修复多线程 instance attribute 竞态条件，零行改动上游代码，跨平台兼容 Windows/macOS。

**典型踩坑 —** label-leakage 防护 · O(N²)→O(1) 候选过滤重构 · 图采样 ID mapping 一致性 · 跨模型 embedding 版本对齐

---

*能做 u2u 社交召回 + 内容冷启动召回 + 独立把模型跑通到 serving。下一步在更大规模音频内容场景上验证这套双路召回体系。*
