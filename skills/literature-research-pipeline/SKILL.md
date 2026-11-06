---
name: literature-research-pipeline
description: Use when a user needs an evidence-traceable overview of a research field, systematic literature review, state-of-the-art survey, annotated bibliography, research-gap analysis, or cross-paper method comparison.
---

# Literature Research Pipeline

Use this skill to turn a research question into a scoped, cited overview of a field. Keep three evidence jobs separate:

1. **Paper discovery** — use Firecrawl's research-paper index first when available; do not silently replace it with an arXiv-only workflow.
2. **Primary-source verification** — use the `research` skill or equivalent first-party reading workflow.
3. **Cross-paper synthesis** — use `systematic-literature-review` when available, or apply the synthesis contract below manually.

Do not let generic web research replace paper-index retrieval, and do not treat an abstract or search snippet as a verified empirical claim.

## When to use

Use for requests such as:

- “What is the current research status of …?”
- “Survey recent work on …”
- “Compare methods across papers on …”
- “Find research gaps in …”
- “Prepare a systematic or scoping review.”

Do not use for a single-paper peer review, a single factual lookup, or a product recommendation.

## Operating contract

Before retrieval, define the research question, date range, discipline, target paper count, inclusion/exclusion criteria, and citation style. For a formal report, also define the output path; for an inline answer, do not require a file path. If one of these is missing but a reasonable default will not change the task, use a default and state it in the report.

Default to 20 papers and a maximum of 50 papers in one synthesis. For larger corpora, split the question into subtopics and synthesize the sub-reports.

Produce these artifacts when the user wants a formal report:

```text
outputs/
├── research-brief.yaml
├── candidate-papers.json
├── evidence-table.md
├── synthesis.md
└── final-literature-review.md
```

If the user asks for only an inline answer, keep the same evidence distinctions but do not create unnecessary files.

## Phase 1: Scope the research

Use this brief before searching:

```yaml
topic: "{{research area}}"
research_question: "{{answerable question}}"
scope:
  period: "{{start date}} to {{end date}}"
  disciplines: ["{{discipline}}"]
  geography_or_population: "{{optional}}"
  paper_count: 20
  citation_style: "APA"
inclusion_criteria:
  - directly addresses the research question
  - contains identifiable methods and findings
  - has a stable paper or publisher URL
exclusion_criteria:
  - duplicate versions of the same study
  - papers that mention the topic only in passing
  - unverifiable or citation-only records
deliverables:
  - research themes
  - method comparison
  - consensus and disagreement
  - research gaps
  - limitations and search log
output_path: "outputs/final-literature-review.md"
```

Ask one concise clarification only when the missing choice materially changes the search or report. Otherwise make the default explicit and proceed.

## Phase 2: Discover papers with Firecrawl

For academic literature, the routing order is mandatory:

1. Use Firecrawl's `research-index` capability when the Firecrawl connector is available.
2. Use another multi-source academic index only when Firecrawl is unavailable or its coverage is materially insufficient.
3. Use the local `systematic-literature-review` arXiv workflow as a disclosed fallback, not as the default when Firecrawl is available.

Never describe an arXiv-only result as a complete field review. State the actual index, dates, languages, and venue coverage in the report.

Do not use the generic `firecrawl-deep-research` workflow as a substitute for a paper index. Use the research-index capability if the connector is available:

```text
firecrawl_research_search_papers(
  query="{{2–3 core keywords}}",
  k={{target count}},
  sort="relevance",
  from="{{YYYY-MM-DD}}",
  to="{{YYYY-MM-DD}}"
)
```

CLI equivalent:

```bash
firecrawl research search-papers "{{2–3 core keywords}}" \
  --k {{target count}} \
  --sort relevance \
  --from {{YYYY-MM-DD}} \
  --to {{YYYY-MM-DD}}
```

Search rules:

- Extract 2–3 core keywords; do not pass a long natural-language question as one quoted phrase.
- Start with relevance ranking. Add date bounds when freshness matters.
- Run a small number of focused variants: core topic, method family, and application/benchmark.
- Prefer the paper index for paper discovery. Use general web search only for missing publisher pages, project pages, standards, datasets, or author-owned material.
- Preserve the search date, query, source, and returned count in the search log.
- If the connector is unavailable, first use the best available academic index; if no academic index is available, ask the user for a paper list or use the local arXiv workflow. Clearly label the fallback and its coverage limits.

Ask the retrieval layer to return at least:

```text
title
authors
published_date
updated_date
venue
doi_or_index_id
abstract
paper_url
publisher_url
related_papers
```

If a field is unavailable, write `null` or `unknown`; never infer it. Use paper inspection or the formal publisher page to fill missing metadata when that source is available.

Save candidates as structured data before synthesis. Do not silently pad a search that returns fewer papers than requested.

## Phase 3: Deduplicate and verify evidence

Treat search results as candidates, not verified facts. Use `research` for primary-source verification: read the paper, formal publication page, official project page, dataset documentation, or official code repository that owns the claim. Save verified findings to one Markdown evidence file.

For every candidate, create a canonical record:

```text
canonical_id:
canonical_title:
authors:
earliest_preprint:
conference_version:
journal_version:
latest_version:
doi:
primary_url:
related_versions:
deduplication_reason:
verification_status: verified | abstract-only | unverified | excluded
```

Deduplication rules:

1. Same DOI means the same work.
2. Normalize an arXiv URL or ID to its base identifier by removing `arXiv:` prefixes, URL variants, and trailing version suffixes such as `v1`, `v2`, or `v3`; those versions are one work.
3. Similar title plus strongly overlapping authors requires manual review, not blind merging.
4. Merge a preprint and its formal publication as one study when title, authors, abstract, and contribution substantially match; retain both version URLs and the formal DOI when available.
5. Treat a substantially expanded study as separate only when it adds material theory, data, experiments, or results; record the “extended from” relationship.
6. Never infer that two papers are duplicates from title similarity alone.

Evidence states:

- **Verified** — the source directly supports the statement and the version/date is recorded.
- **Abstract-only** — the statement is supported only by an abstract or index record.
- **Unverified** — a candidate claim has not been checked against the owning source.
- **Excluded** — removed by scope, duplication, or evidence criteria.

Never upgrade abstract-only or unverified evidence by wording it more confidently.

Use this verification prompt when delegating to `research`:

```text
Verify the following candidate papers against primary sources and write one Markdown evidence file.

Research question:
{{research_question}}

Candidate papers:
{{candidate_papers}}

For each paper, record:
- canonical citation and version relationship
- research question
- methodology
- data or benchmark
- evaluation method
- key findings
- limitations
- reproducibility information
- verification_status
- source URLs supporting each important claim

Rules:
1. Prefer the paper text, formal publisher page, official project page, dataset documentation, or official repository.
2. Do not fill missing details from memory.
3. Mark abstract-only and unverified details explicitly.
4. Distinguish what the paper states from your synthesis across papers.
5. List conflicts, missing evidence, and papers needing follow-up.
```

Use an evidence matrix:

```markdown
| Claim | Source type | Primary URL | Directly supported? | Version/date | Evidence state | Notes |
|---|---|---|---|---|---|---|
| ... | paper experiment | ... | yes/partial/no | ... | verified | ... |
```

## Phase 4: Synthesize the literature

Use `systematic-literature-review` when available for structured extraction and thematic synthesis, but do not invoke its built-in arXiv retrieval phase after Firecrawl has supplied the corpus. Instead, pass the deduplicated Firecrawl records into its extraction and synthesis stages, or use this skill's extraction/synthesis contract manually. Only invoke the local arXiv retrieval phase when Firecrawl and other academic indexes are unavailable or materially insufficient; record that fallback before searching.

For each included paper, extract:

- research question
- method
- data or experimental setting
- evaluation method
- key findings
- limitations
- evidence state

The synthesis must contain all of the following:

1. **Three to six themes** — recurring research directions, methods, or problem framings.
2. **Convergences** — findings supported by multiple papers.
3. **Disagreements** — conflicting findings, incompatible setups, or different definitions.
4. **Gaps** — missing populations, datasets, comparisons, mechanisms, or reproducibility evidence.
5. **Coverage limitations** — databases, dates, language, paywalls, preprints, or indexing gaps.

Do not turn “the paper did not discuss X” into a research gap by itself. A gap needs evidence from the search scope, repeated limitations, missing comparisons, conflicting findings, or a clearly bounded absence of evidence.

Use this synthesis prompt:

```text
Synthesize the verified paper evidence below into a literature-state report.

Topic: {{topic}}
Research question: {{research_question}}
Scope: {{scope}}
Evidence file: {{evidence_file}}

Requirements:
1. Identify 3–6 research themes.
2. Compare methods, data, evaluation settings, and limitations.
3. Separate convergence, disagreement, and incomparability.
4. Identify research gaps only when the evidence supports them.
5. Distinguish direct paper claims from cross-paper inference.
6. Do not invent citations or upgrade abstract-only evidence.
7. Report the search boundary and what may be missing.
```

## Phase 5: Write the final report

Use a consistent citation style selected in the research brief. Put citations next to the claims they support. Include a short methodology section so another researcher can reproduce the search boundary.

Final report template:

```markdown
# {{Topic}}：研究现状综述

## 1. 执行摘要

用 3–5 句话回答：当前主流方向是什么、主要共识是什么、最大分歧是什么、最重要的研究空白是什么。

## 2. 研究问题与范围

- Research question:
- Search date:
- Date range:
- Disciplines/geography:
- Target and actual paper count:
- Citation style:

## 3. 检索与筛选方法

### 3.1 数据源

### 3.2 检索词与检索式

### 3.3 纳入与排除标准

### 3.4 版本去重规则

### 3.5 筛选数量

| Stage | Count |
|---|---:|
| Initial candidates | ... |
| After deduplication | ... |
| After scope screening | ... |
| Fully verified | ... |
| Included in synthesis | ... |

## 4. 主要研究主题

### 4.1 {{Theme 1}}

研究问题、主流方法、代表性论文、主要结果和限制。

### 4.2 {{Theme 2}}

...

## 5. 方法与证据比较

| Approach | Core idea | Data/benchmark | Strengths | Limitations | Representative papers |
|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... |

## 6. 主要共识

1. ...

## 7. 主要分歧与不可比之处

| Issue | Supporting evidence | Contrasting evidence | Likely reason |
|---|---|---|---|
| ... | ... | ... | ... |

## 8. 研究空白

For each gap, state the evidence, whether it is “not studied”, “under-studied”, or “conflicting evidence”, and a confidence level.

## 9. 后续研究问题

1. ...

## 10. 结论

## 11. 参考文献

## 附录 A：论文版本与去重表

## 附录 B：证据核查矩阵

## 附录 C：检索日志与覆盖限制
```

## Common failure modes

| Failure | Required correction |
|---|---|
| Using generic web deep research as the paper corpus | Switch to the paper index or disclose the fallback. |
| Treating abstracts as verified results | Mark `abstract-only` and verify the full source. |
| Counting preprint and journal versions twice | Create one canonical record and preserve version links. |
| Listing papers without synthesis | Add themes, convergence, disagreement, gaps, and limitations. |
| Calling an author’s future-work list an objective gap | Check later literature and state the evidence basis. |
| Claiming exhaustive coverage | Report actual sources, dates, limits, and missing areas. |
| Mixing web facts and paper findings | Keep source types separate in the evidence table and report. |

## Completion checklist

Before claiming completion, confirm:

- The research question and boundaries are explicit.
- The paper corpus was searched with short, relevance-oriented queries.
- Candidate papers were deduplicated by study/version.
- Important claims have primary-source URLs or an explicit evidence state.
- The report distinguishes fact, synthesis, and inference.
- Themes, consensus, disagreement, gaps, and coverage limitations are present.
- Citation format is consistent.
- Unverified claims are not presented as established findings.

