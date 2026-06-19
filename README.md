## Hi there 👋 我是 南志锦

[![NVIDIA SkillSpector](https://img.shields.io/badge/NVIDIA-SkillSpector-76B900?style=flat&logo=nvidia)](https://github.com/NVIDIA/SkillSpector/pull/100)

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

#### ⚡ NVIDIA SkillSpector · 多语言并发批处理引擎
*[PR #100](https://github.com/NVIDIA/SkillSpector/pull/100) 已提交 · 2,900+ 行 Python · 8 模块 · 零改动上游*

为 NVIDIA 开源的 AI Agent 安全扫描器独立开发 contrib 批处理引擎，将单线程英文工具升级为多语言并发生产级系统。

**三层并发架构 — 每层互不感知**

```
Layer 3 [CONTRIB]  ThreadPoolExecutor(max_workers=N)    跨技能并行
Layer 2 [UPSTREAM] asyncio.Semaphore(10)                分析器内并发
Layer 1 [UPSTREAM] 20 analyzers fan-out (15 static + 5 LLM)  单技能内并行
```

核心洞察：`graph.invoke(state)` 是纯函数（每个 skill state dict 独立，graph 只读编译蓝图）→ "scan N skills" = `ThreadPoolExecutor.map(graph.invoke, states)`。在多线程安全证明之上叠加语言检测、API 池和对比标注，不碰函数本身。

**7 个 import-time 猴子补丁 — 无侵入兼容 DeepSeek**

| # | 目标 | 效果 |
|---|------|------|
| 1 | `LLMAnalyzerBase.__init__` | `self.response_schema=None` 禁用 structured output (instance attr，零竞态) |
| 2-3 | `parse_response` × 2 | 手动 `json.loads` → Pydantic 校验，替代上游的 `with_structured_output()` |
| 4-5 | `build_prompt` × 2 | 追加 JSON 格式指令 (无 `response_format` 时引导模型输出) |
| 6 | `ChatOpenAI.__init__` | 注入 `httpx.Timeout(connect=8s, read=30s)` 防 hung connection |
| 7 | `asyncio.run` | 静默 `Event loop is closed` (httpx 清理阶段无害噪音) |

**竞态条件修复 (Patch 1 的关键洞察)：** 类属性 `response_schema` 被多线程同时读写 → Thread A 恢复原值时 Thread B 的 meta-analyzer 正在创建实例 → `with_structured_output()` 触发 → HTTP 400。修复：`self.response_schema = None` 写入实例 `__dict__` → Python MRO 先找实例属性 → 每个 analyzer 实例独立 → 零共享状态。

**K8s 风格 API Key 池 — 多 Key 调度 + 故障转移**

```
acquire → 选 least-loaded idle key
release(success=True)  → 标记 idle, consecutive_429 清零
release(success=False) → 标记 rate_limited, 指数退避 30s×2ⁿ (上限 300s)
下次 acquire → 自动选不同 key
PooledChatModel: 透明重试 ≤5 次, 兼容 LangChain BaseChatModel 接口
```

**跨平台清理韧性：** `shutil.rmtree` (macOS 上遇 dangling fd 卡死) → 子进程 fallback (`cmd /c rmdir /s /q` on Windows / `rm -rf` on Unix, 10s 超时)，子进程不受 Python fd 影响。

**拒绝方案记录：** ProcessPoolExecutor (macOS spawn 30s 启动超时) · 全 asyncio (graph.invoke 同步阻塞，LangGraph 不暴露 async 入口) · fork 上游 (永久分叉，每次 release 都需 rebase) · 完整二次图扫描 (仅 8/25 规则无语义等价覆盖)
    
**实验验证：** 23 fixtures × 8 workers，零超时零崩溃。LLM 语义分析器将 `ssd1_semantic_injection` 从 0/100 → 100/100, `ssd4_narrative_deception` 从 10/100 → 100/100。Clean skill 保持 clean — 零误判。跨平台验证 Windows/macOS 通过。

**额外功能：** Unicode 脚本比例语言检测 (CJK/Hiragana/Hangul, 零依赖 `unicodedata`) · 8 规则 LLM gap-fill (P5/P6-P8/MP1-MP3/RA1-RA2) · 三种输出格式 (Terminal Rich / JSON / Markdown) · 三级退出码 CI 友好

#### 🦊 NaNaGi · 关系型 AI Agent 平台
*[nanagi.cn](https://nanagi.cn) 已上线 · Fork 即用 · DeepSeek V4 Pro · [Repo](https://github.com/nanzhijin/NaNaGi)*

**定位：** 市面 Agent 解决"怎么让 LLM 做事"，NaNaGi 解决"怎么让 LLM 有持续的关系记忆和社交情境感知"。工具型 Agent → 关系型 Agent。

**ETCLOVG 七层架构 (Harness Engineering 关系型特化)**

基于 CMU·Yale·JHU·Amazon 联合发布的 Agent Harness Engineering 综述（不改模型权重，纯 Harness 编码基准 10 倍提升），将七层框架重新定义为关系型 Agent 的底层基础设施：

```
L6 入口层   — bcrypt+JWT 鉴权, PersonaSwitch(双人格切换), 三身份通道(admin/guest-iv/guest)
L5 人格引擎  — Context Governor(中央调度器, Token~8K), CPM 情绪引擎(5维评价→双通路),
               Self-Node(6 trait) + IWM Nodes(Bowlby 依恋), SIP 社交规划(6步决策+5策略池)
L4 Agent执行 — ReAct 循环(10步), while round<5, 三层容灾(30s超时→重试1次→降级回复)
L3 工具层    — 8 工具注册表 (get-time/weather/search-web/generate-image/search-memory/
               save-memory/paper-search/gnn-recommend), 按人格自适应筛选
L2 存储层    — admin 文件系统(可审计) + guest LevelDB(物理隔离), 每人独立 IWM+memories+cells
L1 质量保障  — O(Observability): LLMTrace/ToolTrace/EmotionTrace/Cost per session
               V(Verification): 社交一致性/人格边界/幻觉防御/Supervisor×3(Drift/Critic/Trajectory)
               G(Governance): 关系边界/H1-H4指令层级/速率限制/审计追踪
L0 外部依赖  — DeepSeek V4 Pro, 腾讯混元(生图), 和风天气, QQ邮箱SMTP, LanceDB, Sentence-BERT
```

**记忆系统 — L1/L2/L3 三级 + 双人格双路检索**

| 层级 | 存储 | 检索 | 延迟 | 容量 |
|------|------|------|------|------|
| L1 热 | System Prompt 内嵌 (IWM摘要+滚动摘要+Session) | 自动注入 | 0ms | ~4K tokens |
| L2 温 | .md frontmatter + MEMORY.md 索引 + [[关联]] 图遍历 | SIM-RAG(companion) / IterResearch(worker) + RRF 融合 | ~500ms | ~150条 |
| L3 冷 | LanceDB 向量索引 (Sentence-BERT embedding) | 余弦 top-k + Ebbinghaus 14天衰减 | <200ms | 无上限 |

🛡 Supervisor 三层监督 (作用 L2 [[关联]] 遍历): Semantic Drift(cosine<0.3→拒绝) → Lightweight Critic(YES/NO) → Trajectory(3步单调降→拒绝)

**心理学模型现代化 (2026-06 修订)**

| 模块 | 原理论 | 升级后 | 改动 |
|------|--------|--------|------|
| 情绪引擎 | OCC (1988) 3维 | **CPM** (Scherer 1993-2009) 5维 + appraisalSequence | ~100行 |
| IWM 表征 | Bowlby (1969) | Bowlby + Wu三阶段模型 (2025) | +1字段 |
| 图传播 | Heider (1958) | Heider + RAF 认知断路器 (2025) | +40行 guard |
| 心理理论 | Premack (1978) | Flavell元认知 + Wellman信念-欲望 | +50行 |
| 人格面具 | Jung 原型 | Self-Determination Theory 动机连续性 + Big Five 锚定 | +60行 |

**实施进度 — v5.3 路线图** | P1-P4 ✅ 已完成 | P5-A 身份系统 (进行中) | P5-B 认知增强 + V3 记忆分级 | P5-C 质量保障 + Supervisor | P5-D 闭环 + 双路检索 | 44 任务 × 4,165 行 × 19 篇论文引用

**Fork 即用 —** `npm install && cp .env.example .env.local` 填入 DeepSeek Key 即可启动。SMTP(QQ邮箱)/管理员密码/访客密码均预配。Docker 一键部署。

---

### 🔧 Engineering

独立走通 清洗→构图→训练→评测→序列化→inference 全链路。

**典型踩坑 —** label-leakage 防护 (不过滤负样本 MRR 被拖死 10 倍) · O(N²)→O(1) 候选过滤重构 · 图采样 ID mapping 一致性 · 跨模型 embedding 版本对齐

---

*从社交图召回引擎到关系型 Agent 平台，从并发批处理引擎到开源社区贡献，独立设计、独立实现、独立部署。下一步：把 GNN 召回 + 音频内容召回接入 NaNaGi，成为 Agent 可调用的 recall 插件。*

