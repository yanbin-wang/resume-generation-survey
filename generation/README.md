# 简历生成（候选人侧）

> 负责人：待指派　|　截止：2026-10-25　|　本页来源编号 S1–S10，仅本页内有效

## 一、核心结论

1. **两条技术路线**：一是「即时定制生成」，以 ResumeFlow（SIGIR 2024）为代表，单份简历 + JD 直接让 LLM 生成；二是「长期职业库检索增强」，以 Career-Aware Resume Tailoring（arXiv 2026 预印本）为代表，维护历史职业记录，用多源 RAG 组装简历。

2. **RAG 路线在低经验匹配场景会退化**：Career-Aware 试点显示，有相关经历的岗位 ATS 风格匹配分平均 +7.8，缺乏相关经验的岗位反而 −8.0；作者自述需要「置信度门控的检索」。

3. **工程实践比学术研究更成熟**：GitHub / PyPI 上已有多个可部署工具，覆盖 AI 生成、ATS 检查、LaTeX 输出、求职信生成；ATS 可解析性检查已成为较新工具的标配（Resume-Libre 的 31 项确定性检查）。

4. **幻觉控制是共同关注点**：ResumeFlow 提出对齐与幻觉评估指标；Career-Aware 设 anti-hallucination guardrails；工程侧多靠「结构化输入 + 确定性检查」压制编造。

---

## 二、研究工作

### 2.1 ResumeFlow（SIGIR 2024 Demo，arXiv:2402.06221）

代表「即时定制生成」路线。给定一份写得很详细的简历和目标岗位 JD，用现成的 GPT-4 或 Gemini 直接生成针对该岗位定制的简历，**不需要任何微调**，秒级出结果——这是它区别于同期微调方案的核心卖点。

除生成流程外，主要贡献还包括一套针对该任务的评估指标：同时衡量生成内容与岗位的**对齐程度**，以及是否存在**幻觉**（编造候选人没有的经历或技能）。这套指标后来被同类工作引用为评估基线。

局限很明确：只处理「单份简历 → 单个岗位」的一次性映射，不涉及长期职业库，也不做多源检索；简历的版本沉淀不在讨论范围内。

### 2.2 Career-Aware Resume Tailoring（arXiv:2605.05257，预印本）

走的是**长期职业库 + 多源检索增强**。要求用户维护结构化的历史职业记录（存进向量数据库），面对新岗位时不只改写当前简历，而是从整个职业库里检索、组装最相关内容。

工程实现是一条 **12 节点的 LangGraph 流水线**，输入为历史简历 + 结构化职业记录 + JD，输出为针对该岗位重新组装的简历内容。

评估用 ATS 风格匹配分，在 **9 个 JD 上做单候选人试点**：有相关经历平均 **+7.8 分**，缺乏相关经验 **−8.0 分**，部分重叠约 **+2 分**。作者把低匹配场景的退化列为待解问题，提出需要「置信度门控的检索」，并设了 anti-hallucination guardrails。

硬伤：单一候选人、9 个 JD，样本量小，只能算方向性证据。

### 2.3 对比表

| 名称 | 来源/编号 | 技术路线 | 输入 | 输出 | 评估方式 | 关键发现 | 局限 | 证据强度 |
|---|---|---|---|---|---|---|---|---|
| ResumeFlow | SIGIR 2024 Demo；arXiv:2402.06221（S1/S2） | LLM 即时定制生成（GPT-4 / Gemini，无需微调） | 详细简历 + JD | 定制化简历 | 任务特定的对齐度与幻觉指标 | 秒级生成；完全 off-the-shelf | 不涉及长期职业库或多源检索 | 已读摘要 |
| Career-Aware | arXiv:2605.05257（预印本，2026-05；S3/S4） | 多源 RAG + 长期职业库；12 节点 LangGraph | 历史简历 + 结构化职业记录 + JD | 从职业库组装的简历内容 | ATS 风格匹配分（9 个 JD） | +7.8 / −8.0 / +2 | 低匹配场景退化；样本量小 | 已读摘要 |

---

## 三、开源工具

### 3.1 Resume-Libre

功能最全的一个，思路是「把简历当产品做」。React 19 + FastAPI + Supabase，PDF 用 LaTeX（Tectonic）渲染，输出接近手工排版。覆盖 AI 生成、GitHub / LinkedIn 导入、按 JD 定制、版本控制、公开发布主页。

最有参考价值的设计是 **ATS 检查走确定性路径**：31 项检查分 6 大类，不用 LLM 判断而是规则硬检查；结果再注入提示词，触发一次「修复式重生成」。这个闭环比让模型自我评估靠谱得多。

AGPL-3.0，Docker Compose 自托管，提供免 Key 的 DEMO_MODE。

### 3.2 resume-builder-llm

结构清晰的 Next.js 全栈实现。Next.js 15 + TypeScript + Vercel AI SDK，模型可选 GPT-4o 或 AWS Bedrock 上的 Llama 3 70B（后者适合数据合规场景）。

功能：简历定制、求职信、内置 ATS 评分、实时 PDF 预览。核心概念是 **Master Profile（主档案）**：用户维护完整档案，AI 每次从中挑选与岗位相关的内容，而不是从零生成——与学术侧「长期职业库」本质相同，实现更轻。

本地 npm 运行，需自备 Key，许可证未标注。

### 3.3 jobjob

Python 命令行工具，定位「一条命令搞定投递材料」。Python ≥3.12，底层 Claude，从 JD 生成简历、求职信、技能差距分析三件套。

工程细节值得注意：全流程 **5 次 Claude 调用共享同一段缓存前缀**，把简历等固定上下文缓存在前，显著降低 token 成本。

MIT，pipx / pip 安装，附带本地浏览器面板；可选上传 Google Drive。命令已变更：旧文档 `jobjob apply`，新版实际 `jobjob build`（3.0.1）。

### 3.4 resi-builder

不是一个产品，而是 **Python 库 / 构建块**。提供 `build_resume_preview` 与 `build_resume_pdf`，输入 JD 与用户历史数据，输出预览和 PDF。

抽象程度低、示例简单，适合做自己系统的渲染层，不适合直接给用户用。维护状态未确认。

### 3.5 StackResume

面向开发者的自托管方案，卖点是**数据不出本机**。多代理 LangGraph 流水线，模型可接 OpenAI / Anthropic / Gemini，也可全走 Ollama 离线。

功能：一句话生成、JD 定制（0–100 强度滑块，交互设计值得借鉴）、输出 PDF / Word / ODT、投递跟踪、对话式 UI。

GPL-3.0，Docker 单命令启动。自称 "ATS-optimised"，但未见独立检查器，需实测确认。

### 3.6 对比表

| 名称 | 链接 | 技术栈 | 核心功能 | ATS 支持 | 许可证 | 部署 | 成熟度 |
|---|---|---|---|---|---|---|---|
| Resume-Libre | github.com/EmaadAkhter/Resume-Libre（S5/S6） | React 19 + FastAPI + Supabase + LaTeX(Tectonic) | AI 生成、导入、JD 定制、LaTeX PDF、版本控制、公开发布 | 31 项确定性检查（6 类） | AGPL-3.0 | Docker Compose；DEMO_MODE 免 Key | 可部署 |
| resume-builder-llm | github.com/jaganraajan/resume-builder-llm（S7） | Next.js 15 + TS + Vercel AI SDK | 简历定制、求职信、ATS 评分、实时预览、Master Profile | 内置 ATS Score Calculator | 未标注 | 本地 npm | 可部署（需 Key） |
| jobjob | pypi.org/project/jobjob/（S8） | Python ≥3.12；Claude | JD → 简历 + 求职信 + 技能差距 | 未提及 | MIT | pipx + 本地面板 | 可部署 |
| resi-builder | pypi.org/project/resi-builder/（S9） | Python 库 | 预览 / PDF 两个接口 | 未提及 | 未标注 | Python 包 | 早期库 |
| StackResume | github.com/Sathvik-Rao/StackResume（S10） | LangGraph 多代理；多模型 / Ollama | 一句话生成、JD 定制、多格式输出、投递跟踪 | 自称 "ATS-optimised" | GPL-3.0 | Docker；可离线 | 可部署 |

---

## 四、机会与空白

- **跨职位长期职业库没有成熟产品**：学术侧有原型（Career-Aware），工程侧最近只是「主档案」这种轻量近似。
- **低经验匹配退化没有工程解法**：学术侧已暴露（−8.0），工程侧无对应机制。
- **版本管理与简历历史普遍薄弱**：多数工具聚焦单次生成，只有 Resume-Libre 做出完整版本控制。
- **确定性检查 + LLM 修复的闭环值得复用**：31 项规则检查是当前最可借鉴的工程质量设计。

---

## 五、来源台账

| 编号 | 信息点 | 来源链接 | 类型 | 核实状态 | 备注 |
|---|---|---|---|---|---|
| S1 | ResumeFlow：SIGIR 2024 Demo，LLM 即时定制简历 | https://arxiv.org/abs/2402.06221 | 论文 | 已读摘要 | SIGIR 2024 Proceedings, pp. 2781-2785；DOI: 10.1145/3626772.3657680 |
| S2 | ResumeFlow 使用 GPT-4/Gemini，off-the-shelf，提对齐/幻觉指标 | 同 S1 | 论文 | 已读摘要 | — |
| S3 | Career-Aware：多源 RAG + 长期职业库 | https://arxiv.org/abs/2605.05257 | 论文（预印本） | 已读摘要 | **预印本，未确认正式发表**；2026-05；6 页 5 表 |
| S4 | 评估数据：+7.8 / −8.0 / +2 | 同 S3 | 论文（预印本） | 已读摘要 | 单一候选人，9 个 JD |
| S5 | Resume-Libre：功能、31 项检查、AGPL-3.0、DEMO_MODE | https://github.com/EmaadAkhter/Resume-Libre | 仓库/文档 | 已读 README | 2025-11 创建 |
| S6 | Resume-Libre 技术栈 | 同 S5 | 仓库/文档 | 已读 README | — |
| S7 | resume-builder-llm：技术栈、Master Profile、ATS 评分 | https://github.com/jaganraajan/resume-builder-llm | 仓库/文档 | 已读 README | 2026-01 更新 |
| S8 | jobjob：Claude、pipx、MIT、`jobjob build` | https://pypi.org/project/jobjob/ | 包文档 | 已读 PyPI | 版本 3.0.1 |
| S9 | resi-builder：两个接口 | https://pypi.org/project/resi-builder/1.1.0/ | 包文档 | 已读 PyPI | 版本 1.1.0；维护状态未确认 |
| S10 | StackResume：LangGraph、Docker、Ollama、GPL-3.0 | https://hub.docker.com/r/sathvikrao/stackresume | 仓库/文档 | 已读 Docker Hub | 源仓库 github.com/Sathvik-Rao/StackResume |

---

## 六、维护约定

- 新增论文或工具：先在台账登记编号与核实状态，再在对比表加行，两处编号一致。
- 只写有来源的事实；未确认标「未明确标注」「待实测」。
- 预印本标注「预印本，未确认正式发表」；正式发表后更新状态并在 commit message 注明。
