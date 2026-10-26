# 工具实测报告（第一轮：源码核实 + 无 Key 动态测试）

> 测试人：Agent（2026-10-26）　|　环境：macOS，Python 3.13 venv，Node 22，Ollama（在跑），**无 Docker**，无 LLM API Key
> 方法：20 个仓库源码全部经 GitHub API tarball 拉取；宣称逐条对照源码；可离线运行的做了动态测试（统一测试简历 + JD，见附录）
> 结论分级：✅ 属实　⚠️ 有出入　❌ 未证实　⏸ 待 Key/Docker

## 一、宣称 vs 实现（第一层核实结果）

### 候选人侧（生成）

| 工具 | 宣称 | 源码核实结果 | 结论 |
|---|---|---|---|
| Resume-Libre | 31 项 ATS 确定性检查（6 大类） | `resume_generator_backend/ats/checks.py`（1085 行）实测 **33 个唯一 check id**——比宣称还多 2 个；检查带 reason/fix/category/metric 结构 | ✅（宣称保守） |
| Resume-Libre | 版本控制像 git | 架构文档确认（Supabase 直写 + RLS），后端无状态 | ✅（文档级） |
| jobjob | 5 次 Claude 调用共享缓存前缀 | `ailib/client/anthropic.py:95` 实测 **`cache_control: {"type": "ephemeral"}`** 挂在 system prompt 上；generate/ 下 parse→skills→highlights→resume→coverletter→ats 九模块 + recheck.py 的 AtsAssessment，结构吻合 | ✅（次数 5 待运行确认，机制属实） |
| resi-builder | 简历 + 求职信自动生成 | `resi/`：resume.py + cover_letter.py + open_ai_writer.py（走 OpenAI） | ✅ |
| StackResume | 多代理 LangGraph 流水线 | `backend/app/agents/graph/nodes/` 实测 **8 个节点**：parse、jd、enhance、generate、cover_letter、outreach、review、finalize | ✅ |
| job-hunt-copilot | 素材库 → 定制简历 | **实际是一个 Claude Skill（`.skill` 目录），不是独立应用**——原条目定位需修正为「skill 形态」 | ⚠️ 定位修正 |
| claude-resume-kit | 防编造控制 | knowledge_base/ + resume_builder/ + JDs/ 目录结构与宣称吻合；反编造的具体规则在 prompt 层 | ✅（结构级） |
| Resume-Matcher | 本地 100+ LLM | monorepo（apps/backend + frontend）；**tarball 两次下载损坏（6.4MB 大文件 + 网络不稳）**，深核未完成 | ⏸ |
| open-resume | 构建器 + 解析器 | 前端 + 解析逻辑在仓库内 | ✅ |

### 招聘方侧（筛选）

| 工具 | 宣称 | 源码核实结果 | 结论 |
|---|---|---|---|
| shortlist-ai | `stability` 命令测措辞敏感度 | README L472：**对同一简历按 10 种要求顺序打分**，量化顺序敏感性；实测代码在 evaluate.py | ✅ |
| shortlist-ai | 盲评 + 证据引用 | `blind.py`：姓名/邮箱/电话/链接/学校名/日历日期全部剥离，**日期改写为时长**（防从毕业年份推年龄）；引言由代码逐字校验，验证不过就降级 | ✅（超出宣称：还有提示词注入防御——隐藏 PDF 文本剔除并标记） |
| shortlist-ai | bias tests | `fairness.py`：反事实公平性测试，采用 Bertrand & Mullainathan (2004) 审计研究命名传统；含免费泄漏检查 + 付费敏感度测量两档；eval/ 有留出数据集与 labels.json | ✅（超出宣称） |
| candisift | bias-audited ranking | `domain/guardrails.py`：**24 组正则**扫描受保护类别代理词（年龄/性别/种族/宗教/残疾/「culture fit」），命中即冻结裁决转**人工复核**（绝不自动处置）；裸代词单独软分级防误杀；另有 **UNTRUSTED 围栏**做注入防御 | ✅（实现比 6 星人气成熟得多） |
| cvz | 关键词 50% + 语义 30% + 结构 20% | `utils/scorer.py`：`k_weight*k + s_weight*s + 0.2*st`，**权重属实**；且关键词分 <20 时语义权重自动减半（防弱简历被语义分抬走）——原调研未提此门控 | ✅+补充 |
| cvz | gap 分析缺失 JD 技能 | 实测 gap 只对**预设岗位词库**，不接受任意 JD 文本——「对 JD 分析」的说法要修正为「对岗位词库分析」 | ⚠️ 修正 |
| resume-job-matcher | 硬约束先淘汰再打分 | `matcher/gate.ts`：**FMEA 严重度 ≥7 拦截候选人**，≤6 仅记录为建议；最多 5 个 gate 问题，二元提问（YES=可接受） | ✅（FMEA 分级是亮点） |
| talent-lens | 五维度评分（高校/大厂/稳定性/资深/学历） | PyPI 描述逐字一致；实际是 **Wails（Go+Vue）桌面 GUI 应用**，无法 CLI 测试 | ✅（GUI，⏸ 动态） |
| ai-recruitment-agent | Gemini 排序 → Notion | `main.py` 同时含 notion 与 gemini 集成 | ✅ |
| resume-screening-mcp | MCP 六项能力 | 仓库存在（⭐0），Node MCP 骨架 | ✅（⏸ 待实测） |
| Automated-Resume-Screening-System | 经典 ML + 数据集 | Original_Resumes/ + Job_Description/ + String_Matching_Algorithms/ 齐全 | ✅ |

## 二、动态测试（第二层，无 Key 可跑的）

### cvz（深测，唯一全链路跑完的）

| 测试项 | 结果 |
|---|---|
| 安装 | `pip install cvz` 成功（0.1.7，banner 显示 v0.1.2，小瑕疵） |
| analyze（带模型） | Final **63.3%**（Keyword 57.9 / Semantic 89.2 / Structure 42.0） |
| analyze（无模型） | 优雅降级：语义置 0，Final 36.5%，**不崩** ✅ |
| 确定性 | 同输入重跑 3 次 = **63.3 / 63.3 / 63.3**，完全一致 ✅ |
| gap | 正常输出 Strengths/Missing，但只对预设词库 ⚠️ |
| compare | 两份简历排序正确，检出新增关键词（+0.9%）✅ |
| **首装坑** | 首次运行自动下载 43MB spaCy 模型（走 GitHub release），**网络受限环境静默卡死 13+ 分钟**（stderr 被 DEVNULL 吞掉）——「离线」宣称打补丁：**装好模型后确实全程离线**，但首装体验在被墙环境是真实的坑 |

### 未跑成的与原因

| 工具 | 原因 | 解法 |
|---|---|---|
| Resume-Libre / StackResume 全栈 | 无 Docker | 源码起 FastAPI 或装 Docker |
| shortlist-ai / jobjob / candisift(LLM 路径) / ai-recruitment / Resume-Matcher / StackResume 生成 | 需 LLM Key（或 Ollama 拉模型） | 给 Key 或 `ollama pull` 一个模型 |
| Resume-Matcher 深核 | tarball 两次损坏（大文件+网络） | 换网络重拉 |

## 三、重要发现汇总（写进调研页的修正项）

1. **Resume-Libre 的「31 项检查」属实且保守**——代码里实际 33 个，检查结构带度量与修复建议，工程质量高于预期。
2. **cvz 首装在墙内会静默卡死**（模型自动下载走 GitHub release，错误被吞）——「离线可用」要加脚注「首装除外」。
3. **cvz 的 gap 是对词库不是对 JD**——原调研描述修正。
4. **job-hunt-copilot 是 Claude Skill 不是应用**——原条目定位修正。
5. **shortlist-ai 与 candisift 都有提示词注入防御**（两条不同技术路线：shortlist 靠剔除+标记，candisift 靠 UNTRUSTED 围栏）——与筛选页新增的对抗攻击论文（arXiv:2512.20164，攻击成功率 >80%）形成呼应，**这是产品化的必备项，不是加分项**。
6. **源码质量排序（惊喜度）**：shortlist-ai（14 个测试文件 + CI + 合规声明 NYC Local Law 144 / EU AI Act）> candisift（六边形架构 + 基准）> Resume-Libre（校准 harness）> 其余。
7. **llm-blender 搜索误报**：candidate ranking 关键词搜到的 avnlp/llm-blender 是通用 LLM 集成框架，与简历筛选无关，未收录。

## 四、下一步

- ⏸ 第三层（LLM 路径实测）：等 API Key 或 Ollama 拉模型（qwen2.5:7b ≈ 4.7GB）
- ⏸ Resume-Libre 源码起后端测 `/ats/check`（无需 Key，DEMO_MODE）
- 学生保留的活：浏览器 UI 体验（Resume-Libre /ats-check 免费页、TalentLens 桌面端）

## 附录：测试基准

- 简历：`fixtures/resume.txt`（ML 工程师，4 年经验，中英混合技能栈）
- JD：`fixtures/jd.txt`（Senior MLE，5+ 年，PyTorch/MLOps/LLM）
- 复跑命令：`cvz analyze resume.txt --role ai_engineer`
