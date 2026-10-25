# 简历自动生成：产品调查记录

本仓库记录 LLM 驱动的简历自动生成领域的研究工作与开源工具调查，为产品决策提供参考。

- 调查范围：学术（arXiv、SIGIR 等）+ 工程（GitHub、PyPI、Docker Hub）
- 截止日期：2026-10-25
- 证据强度：区分「已读原文/摘要」「仅看仓库文档」「二手转述」，逐条登记在文末来源台账

---

## 一、核心结论

1. **学术上有两条清晰的技术路线**：一条是「即时定制生成」——以 ResumeFlow（SIGIR 2024）为代表，拿单份简历 + JD 直接让 LLM 生成定制简历；另一条是「长期职业库检索增强」——以 Career-Aware Resume Tailoring（arXiv 2026 预印本）为代表，维护历史职业记录，用多源 RAG 组装简历。

2. **RAG 路线在低经验匹配场景下会退化**：Career-Aware 的试点评估显示，有相关过往经历的岗位 ATS 风格匹配分平均 +7.8，缺乏相关经验的岗位反而 −8.0。作者自述需要「置信度门控的检索」来解决。

3. **工程实践比学术研究更成熟**：GitHub / PyPI 上已有多个可部署的开源工具，覆盖 AI 生成、ATS 检查、LaTeX 输出、求职信生成。ATS 可解析性检查已成为较新工具的标配（如 Resume-Libre 的 31 项确定性检查）。

4. **幻觉控制是共同关注点**：ResumeFlow 专门提出了评估对齐与幻觉的指标；Career-Aware 设置了 anti-hallucination guardrails；工程侧则多靠「结构化输入 + 确定性检查」来压制编造。

---

## 二、研究工作

### 2.1 ResumeFlow（SIGIR 2024 Demo，arXiv:2402.06221）

代表「即时定制生成」路线。给定一份写得很详细的简历和目标岗位 JD，用现成的 GPT-4 或 Gemini 直接生成针对该岗位定制的简历，**不需要任何微调**，秒级出结果——这是它区别于同期微调方案的核心卖点。

除了生成流程，论文的主要贡献还包括一套针对该任务的评估指标：同时衡量生成内容与岗位的**对齐程度**，以及是否存在**幻觉**（编造候选人没有的经历或技能）。这套指标后来被同类工作引用为评估基线。

局限也很明确：它处理的是「单份简历 → 单个岗位」的一次性映射，不涉及跨岗位的长期职业库，也不做多源检索；简历的版本沉淀、历史管理不在讨论范围内。

### 2.2 Career-Aware Resume Tailoring（arXiv:2605.05257，预印本）

走的是另一条路：**长期职业库 + 多源检索增强**。它要求用户维护结构化的历史职业记录（存进向量数据库），面对新岗位时不只是改写当前简历，而是从整个职业库里检索、组装最相关的内容。

工程实现是一条 **12 节点的 LangGraph 流水线**，输入是历史简历 + 结构化职业记录 + JD，输出是针对该岗位重新组装的简历内容。

评估采用 ATS 风格的匹配分，在 **9 个 JD 上做单候选人试点**：有相关过往经历的岗位平均 **+7.8 分**，缺乏相关经验的岗位反而 **−8.0 分**，部分重叠的岗位约 **+2 分**。作者自己把低匹配场景的退化列为待解问题，提出需要「置信度门控的检索」（confidence-gated retrieval），并设置了 anti-hallucination guardrails 防止从职业库里拼出不存在的经历。

硬伤同样明显：单一候选人、9 个 JD，样本量小，结论只能算方向性证据。

### 2.3 对比表

| 名称 | 来源/编号 | 技术路线 | 输入 | 输出 | 评估方式 | 关键发现 | 局限 | 证据强度 |
|---|---|---|---|---|---|---|---|---|
| ResumeFlow | SIGIR 2024 Demo；arXiv:2402.06221（S1/S2） | LLM 即时定制生成（GPT-4 / Gemini，无需微调） | 详细简历 + 目标职位描述 | 针对该职位定制化的简历 | 任务特定的对齐度与幻觉评估指标 | 秒级生成定制简历；完全 off-the-shelf 使用 | 未涉及长期职业库或多源检索 | 已读摘要 |
| Career-Aware Resume Tailoring | arXiv:2605.05257（预印本，2026-05；S3/S4） | 多源 RAG + 长期职业库（向量数据库）；12 节点 LangGraph 流水线 | 历史简历 + 结构化职业记录 + JD | 从职业库组装针对职位的简历内容 | ATS 风格匹配分（9 个 JD 试点） | 有相关经历 +7.8；无相关经历 −8.0；部分重叠 +2 | 低经验匹配场景退化；样本量小（单一候选人，9 个 JD） | 已读摘要 |

---

## 三、开源工具

### 3.1 Resume-Libre

功能最全的一个，基本是「把简历当产品做」的思路。技术栈 React 19 + FastAPI + Supabase，PDF 用 LaTeX（Tectonic）渲染，输出质量接近手工排版。功能覆盖 AI 生成、GitHub / LinkedIn 导入、按 JD 定制、版本控制、公开发布个人主页。

它最有参考价值的设计是 **ATS 检查走确定性路径**：31 项检查分 6 大类，不用 LLM 判断，而是规则化的硬检查；检查结果再注入提示词，触发一次「修复式重生成」。这个「先确定性检查、再让 LLM 修」的闭环，比单纯让模型自我评估靠谱得多。

许可证 AGPL-3.0，Docker Compose 自托管，提供无需 API Key 的 DEMO_MODE，适合先跑通再决定是否投入。

### 3.2 resume-builder-llm

一个结构清晰的 Next.js 全栈实现。技术栈 Next.js 15 + TypeScript + Vercel AI SDK，模型可选 OpenAI GPT-4o 或部署在 AWS Bedrock 上的 Llama 3 70B（后者适合有数据合规要求的场景）。

功能包括简历定制、求职信生成、内置 ATS 评分、实时 PDF 预览，核心概念是 **Master Profile（主档案）**：用户维护一份完整档案，AI 每次从中挑选与目标岗位相关的内容，而不是从零生成。这个思路和学术侧的「长期职业库」本质相同，但实现更轻。

本地 npm 运行，需要自备 API Key，许可证未在搜索结果中明确标注。

### 3.3 jobjob

一个 Python 命令行工具，定位是「一条命令搞定投递材料」。Python ≥3.12，底层用 Claude 模型，从 JD 生成定制简历、求职信和技能差距分析三件套。

工程上有个值得注意的细节：整个流程 **5 次 Claude 调用共享同一段缓存前缀**，把简历等固定上下文缓存在前面，显著降低 token 成本——这是生产化部署时绕不开的优化点。

MIT 许可证，pipx / pip 安装，附带一个本地浏览器面板，不用记命令行参数；可选把结果上传 Google Drive。注意命令已变更：旧文档写 `jobjob apply`，新版实际是 `jobjob build`（版本 3.0.1）。

### 3.4 resi-builder

严格说不是一个产品，而是一个 **Python 库 / 构建块**。它提供 `build_resume_preview` 和 `build_resume_pdf` 两个接口，输入 JD 和用户历史数据，输出简历预览与 PDF。

抽象程度低、文档示例简单，适合作为自己系统的渲染层复用，不适合直接拿来给用户用。维护状态未确认。

### 3.5 StackResume

面向软件开发者的自托管方案，卖点是**数据不出本机**。核心是一条多代理 LangGraph 流水线，模型可接 OpenAI / Anthropic / Gemini，也可以完全走 Ollama 本地模型离线运行。

功能包括一句话生成简历、按 JD 定制（提供 0–100 的定制强度滑块，这个交互设计值得借鉴）、输出 PDF / Word / ODT、投递进度跟踪，以及对话式 UI。

GPL-3.0，Docker 单命令启动。自我描述为 "ATS-optimised"，但没有明确说明是否内置独立的 ATS 检查器——这点需要实测确认。

### 3.6 对比表

| 名称 | 链接 | 技术栈 | 核心功能 | ATS 支持 | 许可证 | 部署方式 | 成熟度 |
|---|---|---|---|---|---|---|---|
| Resume-Libre | github.com/EmaadAkhter/Resume-Libre（S5/S6） | React 19 + FastAPI + Supabase + LaTeX(Tectonic) | AI 生成、GitHub/LinkedIn 导入、JD 定制、LaTeX PDF、版本控制、公开发布 | 31 项确定性检查（6 大类），非 LLM 路径；结果注入提示词触发修复重生成 | AGPL-3.0 | Docker Compose 自托管；DEMO_MODE 免 Key | 可部署 |
| resume-builder-llm | github.com/jaganraajan/resume-builder-llm（S7） | Next.js 15 + TS + Vercel AI SDK；GPT-4o 或 Bedrock 上的 Llama 3 70B | 简历定制、求职信、ATS 评分、实时 PDF 预览、Master Profile | 内置 ATS Score Calculator | 未明确标注 | 本地 npm 运行 | 可部署（需 API Key） |
| jobjob | pypi.org/project/jobjob/（S8） | Python ≥3.12；Claude | JD → 简历 + 求职信 + 技能差距分析；5 次调用共享缓存前缀 | 未提及 | MIT | pipx/pip + 本地浏览器面板 | 可部署 |
| resi-builder | pypi.org/project/resi-builder/（S9） | Python 库 | `build_resume_preview` / `build_resume_pdf` 两个接口 | 未提及 | 未明确标注 | Python 包安装 | 早期库/构建块 |
| StackResume | github.com/Sathvik-Rao/StackResume（S10） | 多代理 LangGraph；OpenAI / Anthropic / Gemini / Ollama | 一句话生成、JD 定制（0–100 强度）、PDF/Word/ODT、投递跟踪、对话式 UI | 自称 "ATS-optimised"，未见独立检查器 | GPL-3.0 | Docker 单命令；支持完全离线 | 可部署 |

> 注：Reactive Resume 属于传统编辑器，不具备 AI 自动生成能力，未列入。如需要作为对照组，可另加一行并备注「无 AI 生成」。

---

## 四、机会与空白

- **跨职位长期职业库没有成熟产品**：学术侧有原型（Career-Aware），工程侧最近的只是「主档案」（resume-builder-llm）这种轻量近似实现。
- **低经验匹配场景的退化没有工程解法**：学术侧已暴露问题（−8.0），工程侧尚无对应机制。
- **版本管理与「简历历史」普遍薄弱**：多数工具聚焦单次生成，只有 Resume-Libre 做出了完整的版本控制。
- **确定性检查 + LLM 修复的闭环值得复用**：Resume-Libre 的 31 项规则检查是当前最可借鉴的工程质量设计。

---

## 五、来源台账

| 编号 | 信息点 | 来源链接 | 类型 | 核实状态 | 备注 |
|---|---|---|---|---|---|
| S1 | ResumeFlow：SIGIR 2024 Demo 论文，LLM 即时定制简历 | https://arxiv.org/abs/2402.06221 | 论文 | 已读摘要 | 正式发表于 SIGIR 2024 Proceedings, pp. 2781-2785；DOI: 10.1145/3626772.3657680 |
| S2 | ResumeFlow 使用 GPT-4/Gemini，off-the-shelf，提出对齐/幻觉指标 | 同 S1 | 论文 | 已读摘要 | — |
| S3 | Career-Aware Resume Tailoring：多源 RAG + 长期职业库 | https://arxiv.org/abs/2605.05257 | 论文（预印本） | 已读摘要 | **预印本，未确认正式发表**；2026-05 提交；6 页，5 表 |
| S4 | 评估数据：匹配职位 +7.8，低匹配 −8.0，部分重叠 +2 | 同 S3 | 论文（预印本） | 已读摘要 | 单一候选人，9 个 JD 的试点评估；作者自述需 confidence-gated retrieval |
| S5 | Resume-Libre：功能列表、ATS 31 项检查、AGPL-3.0、Demo 模式 | https://github.com/EmaadAkhter/Resume-Libre | 仓库/文档 | 已读仓库 README | 2025-11 创建；功能描述来自 README |
| S6 | Resume-Libre 使用 React 19 + FastAPI + Supabase + Tectonic | 同 S5 | 仓库/文档 | 已读仓库 README | — |
| S7 | resume-builder-llm：Next.js 15 + Llama 3 70B/OpenAI，Master Profile，ATS 评分 | https://github.com/jaganraajan/resume-builder-llm | 仓库/文档 | 已读仓库 README | 2026-01 更新 |
| S8 | jobjob：Claude 5 次调用、pipx 安装、MIT、Python ≥3.12 | https://pypi.org/project/jobjob/ | 包文档 | 已读 PyPI 页面 | 版本 3.0.1；命令为 `jobjob build`（旧版 `apply` 已变更） |
| S9 | resi-builder：Python 库，`build_resume_preview` / `build_resume_pdf` | https://pypi.org/project/resi-builder/1.1.0/ | 包文档 | 已读 PyPI 页面 | 版本 1.1.0；抽象程度低，维护状态未确认 |
| S10 | StackResume：LangGraph 多代理、Docker、Ollama 离线、GPL-3.0 | https://hub.docker.com/r/sathvikrao/stackresume | 仓库/文档 | 已读 Docker Hub 描述 | 源仓库 github.com/Sathvik-Rao/StackResume |

---

## 六、维护约定

- 新增论文或工具：先在来源台账登记编号和核实状态，再在对应对比表加行，两处编号必须一致。
- 简介段落与对比表只写有来源的事实；未确认的信息标「未明确标注」或「待实测」，不靠推测补齐。
- 预印本必须标注「预印本，未确认正式发表」。
- 证据强度升级（如预印本正式发表）：更新台账状态，并在 commit message 中注明。
