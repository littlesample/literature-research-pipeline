# literature-research-pipeline

一个用于开展可追溯文献调研、系统性综述、研究现状分析和研究空白识别的 Codex Agent Skill。

它将文献工作拆成三个环节：

1. 使用 Firecrawl Research Index 发现论文；
2. 回到论文正文、出版社页面、官方规范或官方代码仓库核查关键结论；
3. 对去重后的论文进行主题综合、方法比较、共识/分歧分析和研究空白提炼。

## 适用场景

- 调研某一研究领域的国内外研究现状；
- 制作系统性综述、范围综述或注释型书目；
- 比较不同论文的方法、数据集、实验设置和结论；
- 寻找有证据支持的研究空白和后续研究问题；
- 区分正式论文、预印本、摘要证据和未核实候选记录。

## 安装

### 使用 skills CLI（推荐）

需要 Node.js 22.20 或更高版本：

```bash
npx skills add littlesample/literature-research-pipeline --skill literature-research-pipeline
```

安装到所有项目：

```bash
npx skills add littlesample/literature-research-pipeline --skill literature-research-pipeline -g
```

检查安装结果：

```bash
npx skills list
```

### 手动安装到 Codex

将以下文件复制到 Codex skill 目录：

```text
skills/literature-research-pipeline/SKILL.md
→ ~/.codex/skills/literature-research-pipeline/SKILL.md
```

Windows 默认位置通常是：

```text
C:\Users\<用户名>\.codex\skills\literature-research-pipeline\SKILL.md
```

### 只试用、不安装

```bash
npx skills use littlesample/literature-research-pipeline@literature-research-pipeline
```

## 使用方式

安装后可以直接用自然语言描述调研任务，也可以显式指定 skill：

```text
$literature-research-pipeline

调研“卫星网络通信协议设计安全性”的研究现状。
时间范围：2012-01-01 至 2026-11-06。
优先使用 Firecrawl Research Index，去除预印本和正式论文的重复版本，核查关键结论的一手来源，归纳主要研究主题、共识、分歧和研究空白，并输出 Markdown 报告。
```

限定协议、数据集或应用场景时，应直接写入研究问题和筛选条件：

```text
调研“低轨卫星网络路由协议的设计安全性”。
只考虑协议本身的信任模型、状态机、认证、重放、密钥生命周期和资源消耗问题；排除纯实现漏洞、配置错误和物理层攻击。
优先选择有公开规范、开源实现和低成本仿真环境的协议。
```

## 正确使用流程

skill 会按以下顺序工作：

1. 明确研究问题、时间范围、学科、目标论文数量、纳入/排除标准和引用格式；
2. 优先通过 Firecrawl Research Index 检索论文，而不是把普通网页搜索结果当作论文语料；
3. 对同一研究的 arXiv、会议、期刊和更新版本建立一个 canonical record；
4. 读取论文正文、正式出版页面、官方项目页面、标准文档或官方仓库，核查重要结论；
5. 为每篇候选论文标记证据状态；
6. 在去重和核查后进行主题综合，而不是简单罗列标题；
7. 输出研究主题、方法比较、共识、分歧、不可比之处、研究空白和检索限制。

## 用户需要提供哪些信息

至少提供以下信息。缺少非关键项时，skill 会采用默认值并在报告中说明。

| 信息 | 示例 |
|---|---|
| 研究主题 | 卫星网络通信协议安全性 |
| 核心研究问题 | 哪些开放协议存在尚未充分研究的设计层安全问题？ |
| 时间范围 | 2012-01-01 至 2026-11-06 |
| 研究领域 | 计算机科学、网络安全、卫星网络 |
| 目标论文数量 | 40 |
| 是否纳入预印本 | 是/否 |
| 是否只纳入同行评审论文 | 是/否 |
| 语言范围 | 中文、英文或不限 |
| 重点方法/数据集/应用 | 不限，或指定方法、数据集、协议、应用场景 |
| 地域范围 | 国内外、国外为主、指定国家或地区 |
| 引用格式 | APA、IEEE 或 BibTeX |
| 输出形式 | Markdown、聊天回复、表格或文件 |
| 排除条件 | 实现漏洞、纯物理层攻击、专有协议等 |

推荐直接复制下面的任务模板：

```yaml
topic: ""
research_question: ""
scope:
  period: "YYYY-MM-DD to YYYY-MM-DD"
  disciplines: []
  geography_or_population: ""
  paper_count: 20
  languages: ["不限"]
  citation_style: "APA"
  include_preprints: true
  peer_review_only: false
focus:
  methods: []
  datasets: []
  applications: []
inclusion_criteria:
  - "直接回答研究问题"
  - "具有可识别的方法和发现"
  - "具有稳定的论文或出版社链接"
exclusion_criteria:
  - "同一研究的重复版本"
  - "只在背景中提及主题"
  - "无法由一手来源核实的关键结论"
deliverables:
  - "3–6 个主要研究主题"
  - "方法和实验条件比较"
  - "共识、分歧和不可比之处"
  - "有证据支持的研究空白"
  - "检索来源、日期和局限"
output:
  format: "Markdown"
  path: "outputs/final-literature-review.md"
```

## 证据状态

- `verified`：已由论文正文、正式出版页面、官方规范、官方项目页面或官方代码仓库直接支持；
- `abstract-only`：目前只能由摘要或索引记录支持；
- `unverified`：已发现但尚未回到权威来源核查；
- `excluded`：因重复、超出范围、来源不可靠或不符合纳入标准而排除。

`abstract-only` 和 `unverified` 不应被写成已经确认的实验结论。若同一研究同时有预印本和正式发表版本，应合并为一个研究记录，并保留两个版本链接。

## 注意事项

### 预印本与同行评审的冲突

“纳入预印本”和“只纳入同行评审论文”存在冲突。推荐的保守做法是：预印本进入候选池，但正式论文优先用于最终结论；没有正式版本的预印本只能标注为预印本或 `abstract-only`，不能写成同行评审证据。

### 数据库覆盖

Firecrawl Research Index 适合做论文发现入口，但不能等同于 CNKI、万方、Web of Science、Scopus、IEEE Xplore 或 PubMed 的完整覆盖。报告必须写明实际使用的来源、检索日期、检索式和可能遗漏。

### 论文数量

目标数量是上限，不是无条件填充目标。如果高质量候选不足，应报告实际数量和原因，不要用低相关、重复或无法核查的记录凑数。单次综合建议不超过 50 篇；更大的语料应按主题拆分后再综合。

### 研究空白

某篇论文的 future work 没有讨论 X，不能单独证明 X 是客观研究空白。应结合多篇论文的限制、方法缺口、缺失比较、矛盾结果或明确的检索边界来判断。

### 设计问题与实现问题

如果研究问题关注协议设计，应重点分析信任模型、状态机、字段绑定、密钥生命周期、失败策略、重放和资源边界。内存错误、边界检查、API 误用和配置错误属于实现或部署问题，应单独标记。

## 所需连接和降级行为

- Firecrawl Research Index：用于优先发现论文；
- 论文正文、出版社页面或其他一手来源：用于核查关键结论；
- 官方代码、数据集或标准文档：用于确认实验条件和协议细节。

如果 Firecrawl Research Index 不可用，skill 可以使用其他学术索引或公开论文来源，但必须在报告中明确说明降级路径和覆盖限制。不能把 arXiv-only 结果描述为完整领域综述。

## 输出结构

正式报告通常应包含：

1. 执行摘要；
2. 研究问题与范围；
3. 检索与筛选方法；
4. 主要研究主题；
5. 方法与证据比较；
6. 共识、分歧与不可比之处；
7. 研究空白与后续问题；
8. 结论；
9. 参考文献；
10. 版本去重表、证据核查矩阵和检索日志。

## 目录结构

```text
skills/literature-research-pipeline/
└── SKILL.md
```

## 许可

本仓库当前未附带额外许可证。若要在组织、商业产品或公开 skill 集合中再发布，请先为仓库补充合适的许可证，并确认论文、工具和第三方内容各自遵守其许可证与使用条款。

