# 简历筛选（招聘方侧）

> 负责人：待指派　|　截止：2026-10-25　|　本页来源编号 S1–S22，仅本页内有效
>
> **核实状态（2026-10-25 更新）**：原 6 条论文线索已核实 4 条（S1、S2 精确命中；S5 精确命中；S3 找到最接近项）；工具 5 个的 PyPI/GitHub 地址全部验明正身；另新增 4 篇论文、4 个工具。仅 S4、S6 两条线索仍未找到原文。

## 一、核心结论

1. **筛选是招聘技术里研究最密集的方向之一**：传统 NLP+ML、LLM/生成式、效度审计三条路线均有正式论文支撑（IEEE 会议、AAAI IASEAI、arXiv），证据强度高于生成侧。

2. **传统方法仍是多数 ATS 产品的实际内核**：TF-IDF + 余弦相似度 + NER / XGBoost 的两段式方案确定性强、可解释、成本低；2025 年的 IEEE 会议论文（ICIDCA、IICAIET）仍是这条路线的延续。

3. **LLM 筛选的可靠性已被实证打上问号**：普林斯顿团队（Korolova 等，AAAI IASEAI，arXiv:2602.18550）构造了有 ground truth 的简历数据集实测多个 LLM——Claude Sonnet 4 平均 0.96，但 GPT-4o-mini 在两份简历只差一个资历条件时准确率跌到 0.50（等于抛硬币），Llama-3.3-70B 在 0.44–0.88 间波动；模型不会可靠「弃权」（错误中位构成：0.93 无理由弃权 vs 0.07 无理由选择），不同人群选择率存在差异。

4. **LLM 筛选还有安全问题**：arXiv:2512.20164（Mu 等）发现简历里藏「对抗指令」可操纵筛选模型，部分攻击类型成功率超过 80%；提示词防御只降 10.1% 且误拒率上升，训练期防御（FIDS，LoRA）效果更好。**候选人的简历本身就是不可信输入**——这是招聘方产品必须考虑的攻击面。

5. **可解释性正在成为标配**：shortlist-ai 输出证据引用而非总分；多智能体框架（arXiv:2504.02870）输出可解释评分。工程侧与学术侧在这一点上收敛。

---

## 二、研究论文

### 2.1 传统 NLP + 机器学习路线

**S1 · Resume Ranker for Freshers using NLP**（IEEE ICIDCA 2025，pp. 902–907，DOI: 10.1109/ICIDCA66325.2025.11280571）
针对校园招聘场景（候选人学术背景相近、经验少），用 NLP 扫描简历与 JD，抽取技能与学历关键词，**TF-IDF + 余弦相似度**做匹配，spaCy 实现，Streamlit 界面，Random Forest 分类器最终准确率 **90.2%**。原线索中「讨论偏见、透明度与隐私」的表述未在该文摘要中逐字核实，引用时注意。

**S2 · AI-Powered Resume Parser**（IEEE IICAIET 2025，DOI: 10.1109/IICAIET67254.2025.11265526）
即原线索中「XGBoost 77%」那篇，精确命中。**2484 份简历、24 个职位类别**，SMOTE 处理类别不平衡；对比六种算法，XGBoost 职位分类 **77%**（超过 Random Forest 68%、SVM 62%），自定义 NER 模型抽取姓名与技能。定位是「公平、高效、透明」的自动化筛选。

**S3 · Design and development of ML-based resume ranking system**（Tejaswini 等，Global Transitions Proceedings, 2022，ScienceDirect 开放获取）
原线索描述为「综述」，实际找到的最接近项是一篇系统性方法论文：**TF-IDF + 余弦相似度**做内容型推荐匹配、**KNN** 挑选排序，解析准确率 85%、排序准确率 92%（50 份 Kaggle 简历）。可作为传统路线的代表精读；真正的综述待补。

### 2.2 LLM / 生成式路线

**S4 · ChatGPT-4o「公平且可扩展」筛选框架（原线索）——未找到精确原文，仍待补。**
检索中发现两篇高度相关、已核实的论文，可暂时作为该路线的代表：

**S9 · Hybrid Resume Fitment Scoring with LLMs and AHP**（IEEE，文档号 11330300）
GPT-4o / LLaMA-3 / Google Flan 与经典 NLP 并行解析简历，用 **AHP 层次分析法**做标准加权评分，建了 HR 校验的 ground truth，100 份简历实验：GPT+AHP 准确率最高 **71%**。亮点是把「权重」显式化、可调，回应公平性。

**S10 · AI Hiring with LLMs: Context-Aware and Explainable Multi-Agent Framework**（arXiv:2504.02870）
CrewAI 多智能体 + RAG（ChromaDB）：抽取 agent、多维评分 agent（经验/技能/教育等）、分数格式化 agent 分工协作，105 份 HR 标注简历上 DeepSeek-V3 表现最好（PC₁₀=0.84，MAE 0.90），显著优于单模型。支持 Ollama 本地部署保隐私。**这是「多智能体做筛选」的代表论文，与我们自己的多智能体流水线思路同构。**

### 2.3 对 LLM 筛选的效度审计（已核实，本页最重要的一篇）

**S5 · Measuring Validity in LLM-based Resume Screening**（Korolova, Metevier, Castleman, Springer, Shen；AAAI IASEAI；arXiv:2602.18550）

普林斯顿团队解决「没有带 ground truth 的简历语料」这一根本难题：**系统性构造可比较、优劣已知的合成简历数据集**，用变形测试（metamorphic）与变异测试（mutation）审计多个 LLM 的排序效度、弃权行为与人群公平性。

结果（据公开转述与论文页）：Claude Sonnet 4 平均 **0.96**；GPT-4o-mini 在两份简历只差一个资历条件时 **0.50**（抛硬币）；Llama-3.3-70B 波动于 **0.44–0.88**。失败模式以**过度弃权**为主（无理由弃权与无理由选择之比中位数 0.93 : 0.07）；不同人群选择率不同，部分模型会优先选择历史上被边缘化群体，部分偏好白人男性——**偏见方向因模型而异，不能假设「中立」**。

价值：给出了一套可在无 ground truth 场景下复用的审计框架，是「LLM 筛选不可轻信」的最强实证。

### 2.4 LLM 筛选的安全与人类对比（新增）

**S7 · AI Security Beyond Core Domains**（Mu 等，arXiv:2512.20164）
简历中藏「对抗指令」可操纵筛选模型：部分攻击类型**成功率超 80%**；提示词防御仅降 10.1% 且误拒 +12.5%；训练期防御 FIDS（LoRA）降 15.4%、误拒 +10.4%，组合 26.3%。**候选人的简历是不可信输入**。

**S8 · Signal or Noise?**（Varshney & Ganuthula，arXiv:2507.08019）
同一批简历在四种情境（无公司信息 / 跨国公司 / 初创 / 简化）下测 Claude、GPT、Gemini，与三位人类 HR 专家对比：八个纯 LLM 条件中四个存在显著均值差异，**所有 LLM 与人类评估显著不同**（p<0.1）。LLM 筛选是「信号 + 噪音」混合体。

### 2.5 对比表

| 论文 | 路线 | 方法 | 关键结果 | 证据强度 |
|---|---|---|---|---|
| Resume Ranker for Freshers（ICIDCA 2025，S1） | 传统 NLP | TF-IDF + 余弦 + NER + RF | 准确率 90.2% | 已核实元数据 |
| AI-Powered Resume Parser（IICAIET 2025，S2） | 传统 NLP | XGBoost 分类 + 自定义 NER | 24 类 77%，六算法对比 | 已核实元数据 |
| ML resume ranking（2022，S3） | 传统 NLP | 内容推荐 + KNN | 解析 85%、排序 92% | 已核实摘要 |
| Hybrid Fitment（IEEE，S9） | LLM 混合 | GPT-4o 等 + AHP 加权 | GPT+AHP 71% | 已核实摘要 |
| Multi-agent hiring（arXiv:2504.02870，S10） | LLM 多智能体 | CrewAI + RAG，分工 agent | PC₁₀=0.84，优于单模型 | 已核实摘要 |
| **Measuring Validity（arXiv:2602.18550，S5）** | **效度审计** | 合成 ground truth + 变形/变异测试 | 0.96～0.44；不弃权；人群差异 | 已核实摘要 |
| AI Security（arXiv:2512.20164，S7） | 安全 | 简历内嵌对抗指令基准 | 攻击成功率 >80% | 已核实摘要 |
| Signal or Noise?（arXiv:2507.08019，S8） | 人机对比 | 4 情境 × 3 LLM vs 3 位 HR | LLM 与人类显著不同 | 已核实摘要 |
| ChatGPT-4o 公平框架（S4） | LLM | 技能加权 + 匿名化 + 可解释 | — | **未找到原文，待补** |
| Sentence-BERT 多语言短列表（S6） | 脏数据 | SBERT + OCR + GitHub 评分 | — | **未找到原文，待补** |

---

## 三、开源工具

> 2026-10-25 更新：原 5 个工具的 PyPI / GitHub 地址全部验明正身（PyPI JSON API + GitHub Search API），另新增 4 个高相关工具。

### 3.1 shortlist-ai（已核实）

PyPI `shortlist-ai` 0.2.0，仓库 [vinayvobbili/shortlist-ai](https://github.com/vinayvobbili/shortlist-ai)。官方描述：「Evidence-backed, blind resume shortlisting with LLMs — with the evals and bias tests to check it」，与原线索一致：证据引用、盲评、自带评估与偏见测试。

### 3.2 talent-lens（慧眼）（已核实）

PyPI `talent-lens` 0.1.0，官方描述即「慧眼 - 候选人简历质量评估系统，从高校背景、大厂经历、职业稳定性、资深程度、学历五个维度进行量化评分」；另有 GitHub 桌面应用 [1186258278/TalentLens](https://github.com/1186258278/TalentLens)（15 星，AI 驱动 HR 筛选工具）。两者关系待实测确认（可能是同一项目的不同发布形态）。

### 3.3 cvz（已核实）

PyPI `cvz` 0.1.7「ATS Resume Analyzer CLI」，仓库 [anikchand461/cvz](https://github.com/anikchand461/cvz)（描述：装一次、离线用的终端 ATS 检查器）。原线索中「关键词 50% + 语义 30% + 结构 20%」与 `compare`/`gap` 功能的具体实现需读源码确认。

### 3.4 ai-recruitment-agent（已核实）

PyPI `ai-recruitment-agent` 0.3.2「CV screening CLI — ranks candidates against a JD using Google Gemini and pushes（到 Notion）」，仓库 [kneeraazon404/ai-recruitment-cli-agent](https://github.com/kneeraazon404/ai-recruitment-cli-agent)。与原线索（Gemini + Notion + 去重）一致。

### 3.5 resume-screening-mcp（已核实，另发现同类）

仓库 [htStreak/resume-screening-mcp](https://github.com/htStreak/resume-screening-mcp)：「MCP service for resume screening, JD matching, trust checking, scoring and candidate ranking」，与原线索六项能力吻合（⭐0，需实测可用性）。**另值得并列**：LlamaIndex 官方出的 [run-llama/mcp_resume_screening](https://github.com/run-llama/mcp_resume_screening)（14 星，「codebase to top candidates」的远程 MCP server）——大厂背书的同类形态。

### 3.6 补充扫描（GitHub Search API，2026-10-25）

| 名称 | 星数 | 定位 | 链接 |
|---|---|---|---|
| Automated-Resume-Screening-System | 482 | 经典 ML 筛选系统（带数据集） | [JAIJANYANI/Automated-Resume-Screening-System](https://github.com/JAIJANYANI/Automated-Resume-Screening-System) |
| candisift | 6 | 开源 AI ATS：LLM 筛选 + CV 解析 + JD↔候选人匹配 + **bias-audited ranking** | [confused-ai/candisift](https://github.com/confused-ai/candisift) |
| AI-Recruitment-Agent | 52 | 微软 AutoGen 多智能体招聘助手 | [Ancastal/AI-Recruitment-Agent](https://github.com/Ancastal/AI-Recruitment-Agent) |
| RSART | 16 | 简历短列表与排序工具 | [PawanRamaMali/Resume-Shortlisting-and-Ranking-Tool](https://github.com/PawanRamaMali/Resume-Shortlisting-and-Ranking-Tool) |
| resume-job-matcher | 273 | TS CLI：**硬约束先淘汰，再打分**，每条判断带证据 | [sliday/resume-job-matcher](https://github.com/sliday/resume-job-matcher) |
| Resume-Screening-RAG-Pipeline | 190 | RAG 聊天机器人动态检索简历做筛选 | [Hungreeee/Resume-Screening-RAG-Pipeline](https://github.com/Hungreeee/Resume-Screening-RAG-Pipeline) |
| mcp_resume_screening（LlamaIndex 官方） | 14 | 远程 MCP：从代码库找最合格候选人 | [run-llama/mcp_resume_screening](https://github.com/run-llama/mcp_resume_screening) |

**读法**：resume-job-matcher 的「硬约束前置」值得借鉴——先用确定性规则淘汰明显不合格者，再让 LLM 精评，成本与幻觉风险双降；RAG 管线说明筛选也在走「检索增强」路线。

### 3.7 对比表

| 名称 | 链接 | 技术栈 | 核心功能 | 特色 | 核实状态 |
|---|---|---|---|---|---|
| shortlist-ai | [GitHub](https://github.com/vinayvobbili/shortlist-ai) / [PyPI 0.2.0](https://pypi.org/project/shortlist-ai/)（S11） | LLM | 盲评短列表，证据引用，自带 evals 与偏见测试 | 把审计做成产品功能 | PyPI/GitHub 已核实 |
| talent-lens（慧眼） | [PyPI 0.1.0](https://pypi.org/project/talent-lens/) / [GitHub](https://github.com/1186258278/TalentLens)（S12） | Python + LLM | 五维度评分（高校/大厂/稳定性/资深/学历） | 偏好显式化 | PyPI/GitHub 已核实 |
| cvz | [PyPI 0.1.7](https://pypi.org/project/cvz/) / [GitHub](https://github.com/anikchand461/cvz)（S13） | spaCy + sklearn | 终端 ATS 分析器，离线可用 | 离线确定性 | PyPI/GitHub 已核实 |
| ai-recruitment-agent | [PyPI 0.3.2](https://pypi.org/project/ai-recruitment-agent/) / [GitHub](https://github.com/kneeraazon404/ai-recruitment-cli-agent)（S14） | Gemini | JD → 排序 → 推送 Notion | 嵌入现有工作流 | PyPI/GitHub 已核实 |
| resume-screening-mcp | [GitHub](https://github.com/htStreak/resume-screening-mcp)（S15） | Node.js（MCP） | JD 匹配、可信度、评分、排序 | agent 编排形态 | GitHub 已核实 |
| mcp_resume_screening | [GitHub](https://github.com/run-llama/mcp_resume_screening)（S16） | MCP（LlamaIndex） | codebase → top candidates | 大厂官方 | GitHub 已核实 |
| resume-job-matcher | [GitHub](https://github.com/sliday/resume-job-matcher)（S17） | TypeScript CLI | 硬约束前置淘汰 + 证据打分 | 两段式降本 | GitHub 已核实 |
| Resume-Screening-RAG-Pipeline | [GitHub](https://github.com/Hungreeee/Resume-Screening-RAG-Pipeline)（S18） | RAG | 聊天式动态检索筛选 | 检索增强 | GitHub 已核实 |
| Automated-Resume-Screening-System | [GitHub](https://github.com/JAIJANYANI/Automated-Resume-Screening-System)（S19） | 经典 ML | 传统 ML 筛选，带数据集 | 教学参考 | GitHub 已核实 |
| candisift | [GitHub](https://github.com/confused-ai/candisift)（S20） | Python | LLM 筛选 + 解析 + 匹配 + 偏见审计排序 | **唯一自带 bias-audit 的** | GitHub 已核实 |
| AI-Recruitment-Agent | [GitHub](https://github.com/Ancastal/AI-Recruitment-Agent)（S21） | AutoGen 多智能体 | 招聘全流程助手 | 多智能体形态 | GitHub 已核实 |
| RSART | [GitHub](https://github.com/PawanRamaMali/Resume-Shortlisting-and-Ranking-Tool)（S22） | Python | 短列表与排序 | 轻量 | GitHub 已核实 |

---

## 四、机会与空白

- **稳定性 / 效度审计几乎没有产品化**：学术侧已有 ground truth 审计框架（S5），工程侧只有 shortlist-ai 的 evals/bias tests 沾边——把「审计」做成产品的空白最大。
- **对抗攻击防御在产品里完全缺位**：论文证明 >80% 攻击成功率（S7），9 个工具没有一个提到防注入。
- **偏见审计刚有雏形**：candisift 自称「bias-audited ranking」，是 13 个工具里唯一提偏见审计的（实现深度待读源码）；talent-lens 的院校/大厂维度反而天然带偏见。对照 S5 的审计框架，这块仍是最大空白。
- **与生成侧共用内核**：两侧都需要「JD 解析 → 结构化匹配」，目前没有工具同时服务两侧（cvz 最接近）。

---

## 五、来源台账

| 编号 | 信息点 | 来源链接 | 类型 | 核实状态 | 备注 |
|---|---|---|---|---|---|
| S1 | Resume Ranker for Freshers：TF-IDF+余弦+NER+RF 90.2% | https://doi.org/10.1109/ICIDCA66325.2025.11280571 | 论文（IEEE ICIDCA 2025） | 已核实元数据 | 原线索的「偏见/隐私讨论」未逐字核实 |
| S2 | AI-Powered Resume Parser：XGBoost 77% + 自定义 NER | https://doi.org/10.1109/IICAIET67254.2025.11265526 | 论文（IEEE IICAIET 2025） | 已核实元数据 | 精确命中原线索；2484 简历、24 类、SMOTE |
| S3 | ML-based resume ranking：TF-IDF+余弦+KNN，解析 85% / 排序 92% | https://www.sciencedirect.com/science/article/pii/S2666285X21001011 | 论文（2022，开放获取） | 已核实摘要 | 原线索称「综述」，实际是方法论文；真正综述待补 |
| S4 | ChatGPT-4o 公平筛选框架（技能加权+匿名化+可解释） | **待补** | 论文（IEEE） | 未找到 | 暂以 S9、S10 为该路线代表 |
| S5 | Measuring Validity in LLM-based Resume Screening | https://arxiv.org/abs/2602.18550 | 论文（AAAI IASEAI） | 已核实摘要 | Korolova 等；0.96/0.50/0.44–0.88；过度弃权 0.93:0.07 |
| S6 | Sentence-BERT + OCR + GitHub 评分多语言短列表 | **待补** | 论文 | 未找到 | — |
| S7 | AI Security Beyond Core Domains：简历对抗指令 >80% | https://arxiv.org/abs/2512.20164 | 论文（arXiv） | 已核实摘要 | FIDS 防御；组合降 26.3% |
| S8 | Signal or Noise?：LLM vs 人类专家 | https://arxiv.org/abs/2507.08019 | 论文（arXiv） | 已核实摘要 | 4 情境 × 3 LLM vs 3 HR |
| S9 | Hybrid Resume Fitment Scoring with LLMs and AHP | https://ieeexplore.ieee.org/document/11330300 | 论文（IEEE） | 已核实摘要 | GPT+AHP 71%，100 简历 |
| S10 | AI Hiring with LLMs：多智能体 RAG 框架 | https://arxiv.org/abs/2504.02870 | 论文（arXiv） | 已核实摘要 | CrewAI + ChromaDB；105 份 HR 标注简历 |
| S11 | shortlist-ai：盲评 + 证据 + evals/bias tests | https://pypi.org/project/shortlist-ai/ · https://github.com/vinayvobbili/shortlist-ai | PyPI + 仓库 | 已核实 | 版本 0.2.0 |
| S12 | talent-lens 慧眼：五维度评分 | https://pypi.org/project/talent-lens/ · https://github.com/1186258278/TalentLens | PyPI + 仓库 | 已核实 | 版本 0.1.0；桌面应用 15 星 |
| S13 | cvz：ATS Resume Analyzer CLI | https://pypi.org/project/cvz/ · https://github.com/anikchand461/cvz | PyPI + 仓库 | 已核实 | 版本 0.1.7；50/30/20 权重待读源码确认 |
| S14 | ai-recruitment-agent：Gemini 排序 → Notion | https://pypi.org/project/ai-recruitment-agent/ · https://github.com/kneeraazon404/ai-recruitment-cli-agent | PyPI + 仓库 | 已核实 | 版本 0.3.2 |
| S15 | resume-screening-mcp：MCP 六项能力 | https://github.com/htStreak/resume-screening-mcp | 仓库 | 已核实 | ⭐0，可用性待实测 |
| S16 | mcp_resume_screening：LlamaIndex 官方 MCP | https://github.com/run-llama/mcp_resume_screening | 仓库 | 已核实 | 14 星 |
| S17 | resume-job-matcher：硬约束前置 + 证据打分 | https://github.com/sliday/resume-job-matcher | 仓库 | 已核实 | 273 星 |
| S18 | Resume-Screening-RAG-Pipeline | https://github.com/Hungreeee/Resume-Screening-RAG-Pipeline | 仓库 | 已核实 | 190 星 |
| S19 | Automated-Resume-Screening-System | https://github.com/JAIJANYANI/Automated-Resume-Screening-System | 仓库 | 已核实 | 482 星，带数据集 |
| S20 | candisift：开源 AI ATS，自称 bias-audited ranking | https://github.com/confused-ai/candisift | 仓库 | 已核实 | 6 星；「偏见审计」具体实现待读源码 |
| S21 | AI-Recruitment-Agent：AutoGen 多智能体招聘助手 | https://github.com/Ancastal/AI-Recruitment-Agent | 仓库 | 已核实 | 52 星 |
| S22 | RSART：Resume Shortlisting And Ranking Tool | https://github.com/PawanRamaMali/Resume-Shortlisting-and-Ranking-Tool | 仓库 | 已核实 | 16 星 |

---

## 六、维护约定

- S4、S6 仍是「待补」：正文中不得引用其具体说法；补齐并核对原文后升级状态。
- 论文核实分两档：**已核实元数据**（标题/作者/DOI 对上）与**已核实摘要**（读过摘要）——引用数字前必须至少达到后者。
- 星数为 2026-10-25 快照，写进正文时注明日期。
- 预印本标注「arXiv 预印本」；正式发表后更新状态并在 commit message 注明。
