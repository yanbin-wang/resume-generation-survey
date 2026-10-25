# 开源工具对比表

| 名称 | 链接 | 技术栈 | 核心功能 | ATS 支持 | 许可证 | 部署方式 | 成熟度 | 备注 |
|---|---|---|---|---|---|---|---|---|
| Resume-Libre | github.com/EmaadAkhter/Resume-Libre | React 19 + FastAPI + Supabase + LaTeX(Tectonic) | AI 生成、GitHub/LinkedIn 导入、JD 定制、LaTeX PDF 输出、版本控制、公开发布 | 31 项确定性检查（6 大类），非 LLM 路径；检查结果注入提示词触发修复重生成 | AGPL-3.0 | Docker Compose 自托管；DEMO_MODE 无需 API Key | 可部署 | 功能最全；内置 ATS 关键词匹配；有免费在线 ATS 检查器 |
| resume-builder-llm | github.com/jaganraajan/resume-builder-llm | Next.js 15 + TypeScript + Vercel AI SDK；OpenAI GPT-4o 或 AWS Bedrock 上的 Llama 3 70B | 简历定制、求职信生成、ATS 评分、实时 PDF 预览、Master Profile 管理 | 内置 ATS Score Calculator | 未在搜索结果中明确标注 | 本地 npm 运行 | 可部署（需 API Key） | 有“主档案”概念，AI 从中挑选相关内容 |
| jobjob | pypi.org/project/jobjob/ | Python ≥3.12；Claude 模型 | 从 JD 生成定制简历 + 求职信 + 技能差距分析；5 次 Claude 调用共享缓存前缀 | 未提及 | MIT | pipx/pip 安装；含本地浏览器面板 | 可部署 | CLI + 本地 dashboard 两种方式；可选上传 Google Drive |
| resi-builder | pypi.org/project/resi-builder/ | Python 库 | 提供 `build_resume_preview` / `build_resume_pdf` 接口，根据 JD 和用户历史数据生成简历预览与 PDF | 未提及 | 未在搜索结果中明确标注 | Python 包安装 | 早期库/构建块 | 抽象程度低，更像可编程模块而非开箱产品；文档示例较简单 |
| StackResume | github.com/Sathvik-Rao/StackResume（Docker Hub: sathvikrao/stackresume） | 多代理 LangGraph 流水线；支持 OpenAI / Anthropic / Gemini / Ollama | 一句话生成简历；JD 定制（0–100 强度）；输出 PDF/Word/ODT；应用跟踪；对话式 UI | 未明确提及 ATS 检查器，但描述为“ATS-optimised” | GPL-3.0 | Docker 单命令运行；支持完全离线（Ollama） | 可部署 | 面向软件开发者；强调自托管与数据本地化 |

> 注：上一轮提到的 Reactive Resume 属于传统编辑器，不具备 AI 自动生成功能，未列入本表。如果你希望保留作为对照，可另加一行并备注“无 AI 生成”。
