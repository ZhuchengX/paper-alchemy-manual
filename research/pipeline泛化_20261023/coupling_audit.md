# 论文炼金 pipeline 主题耦合审计（只读诊断，2026-10-23）

审计对象：`/Users/xzc/HITSZ/PaperAlchemy/pipeline/`（未改任何文件）。行号以 2026-10-23 14:20 的工作区为准。
参照：`/Users/xzc/HITSZ/PaperAlchemy/cases/run3/chessboard/axes.json`（只看定轴方式）。
未读：`eval/`、answer_key*、alchemy_heldout、baseline、run2、~/.claude/projects、密钥。

---

## 1. 一句话结论 + 总览

**一句话**：流程骨架（运行目录隔离 → 提示词渲染 → 闸门 → MDL 打分 → 稳健性 → 写作 → 溯源 → 评估）基本通用；**主题被写死在三处：抽取 schema/DSL 与方法族枚举、build_db 的 pairs 配对规则（「交互方法 vs 同调用投票」）、筛选层的 core 规则**。换到 RLHF / 检索 / 视觉等领域时，这三处不是“效果变差”，而是**直接把论文挡掉、闸门报错或表变空**。

作者自己知道 schema 要改：`pipeline/RUNBOOK.md:77`「schema 与算子 DSL 是为“多智能体 / 推理时扩展”设计的。换领域时……要同时改 `config/schema.md` 的算子表和方法族」。本审计补上的是：**除了 schema，还有哪些代码与模板跟着写死**，以及哪几处会直接报错。

| 层 | 耦合 | 主要证据（file:line） |
|---|---|---|
| 采购 / 筛选 | 🔴 写死 | `pipeline/templates/01b_triage.md:11`（family 枚举）、`:14`（cheap_vs_strong）、`:16`（会议白名单没有 CVPR/KDD/S&P 等）、`:24`（「training the policy only」= out）、`:25`（core 必须 train_free）；`pipeline/triage_merge.py:18,48-51`；`pipeline/templates/01c_affiliation.md:13`。采购模板 `01_purchase.md` 本身 🟢（`:16` 只认 arXiv 日期，🟡） |
| 抽取 schema + DSL | 🔴 写死 | `config/schema.md:11-22`（算子表）、`:30-44`（calls/rounds）、`:67-81`（family/selector/rounds 等字段）、`:102-104`（只记“reasoning benchmarks”）；`pipeline/templates/02_extract.md:16,20`；`pipeline/check.py:49-50`（family 枚举硬闸门）；`pipeline/init_run.py:39`（只拷项目 schema，无按 run 覆盖） |
| 建库 | 🔴 写死 | `scripts/build_db.py:25-26`（OPS/FAMILIES）、`:30-92`（DSL 解析、调用数推断）、`:98-111`（模型、基准别名）、`:210`（0–1 值乘 100）、`:265-266`（指标正则）、`:280-382`（pairs = interactive vs 投票） |
| 打分检验 | 🟢 通用（有 🟡 小处） | 引擎 `scripts/score_candidates.py:111-151` 与表无关；🟡 `:60` 固定读 pairs.csv（pairs 为空时会崩）、`:62` 可选表名单写死；`scripts/meta_analysis.py:10-11,189-195` 内置簇只针对 pairs 🔴 |
| 提议角度 | 🟡 半通用 | `pipeline/angles.json:4,8,12`（P1–P3 辩论）、`:24,28,32,36`（R1–R4 便宜 vs 贵）🔴；`:16,20,40`（N1/N2/C0）🟢；`pipeline/templates/03_propose.md:23` 举例写死 🟡 |
| 棋盘 | 🟡 半通用 | 引擎 `scripts/chessboard.py:8` 声明“不含任何基准名或方法族名”，基本做到；但 `:84-108` 只认“同调用投票”Δ，`:182-184` 只认 interactive，`:294-298,354,361` 页面文字写死“Δ投票 / 4 列”；`scripts/chessboard_cards.py:218-254` 同上。`axes.json` 本身是领域插件 |
| 美元层 | 🔴 写死（应整体当插件） | `scripts/add_dollars.py:1-12,33-38`（每题 token × $/M token）；`config/model_prices.csv`（1,114 行 LLM API 价）；`scripts/build_cheap_strong.py:10-15,80-100,147-162`；`scripts/ladders.py:7,68`；`pipeline/init_run.py:35-36,41-42` 把它列为必拷 |
| 写作 | 🟢 通用（🟡 少量） | `pipeline/writing_guide/*` 无主题词；🟡 `05_write.md:15`（方法节写死“calls per question”）、`:20`（按 accuracy 估样本量）；🔴 `04b_attach.md:4,9,24`、`05b_attach_write.md:9-10`（写死 run3 的 v2 路径与 G1–G10） |
| 评估 | 🟢 通用 | `08_evaluate.md:9-31` 只依赖答案键与文件；`06_provenance.md`、`07_translate.md` 通用；🟡 `check.py:14,108`、`regen_appendix_a.py:80-99` 默认 arXiv 编号与 triage 文件存在 |

---

## 2. 每个 🔴 / 🟡 点

格式：**位置** — 写死了什么（原文一行）— 换领域怎么坏（例子）— 归「通用内核」还是「领域插件」。

### 2.1 采购 / 筛选

1. 🔴 **`pipeline/templates/01b_triage.md:11`**
   - 原文：`"family": "multi-agent | parallel-select | mid-run-share | new-evidence | stopping | weak-vs-strong | cascade | analysis | other"`
   - 怎么坏：RAG 论文（检索器、重排、chunking）全部落 `other`。下游 `triage_merge.py:18` 的 MECH 名单又只认这几个族，`other` 永远得不到 rescue。
   - 归属：**领域插件**（族表应来自 menu / 领域配置）。

2. 🔴 **`01b_triage.md:24-25`**
   - 原文：`out: fails the menu (training the policy only, single-chain only, survey, no task accuracy)`；core 要求 `in scope AND train-free AND reports compute or cost AND at least one of: cheap_vs_strong, controlled_baseline, ...`
   - 怎么坏：**RLHF / DPO / 对齐论文的本职就是训练策略**，按这一行全部判 out；就算 menu 把它们纳入，core 仍要求 train_free，0 篇能进抽取。视觉、代码训练类论文同理。
   - 归属：**领域插件**（core 判据交给 menu）。“solid”判据（`:18`、`:25` 后半）可留在内核。

3. 🔴 **`01b_triage.md:14`**：`"cheap_vs_strong": true|false (does it compare many calls / agents of a cheaper or smaller model against a stronger ...)`
   - 怎么坏：这是 run3 的中心问题。安全论文（越狱攻防）几乎都是 false，与 `:15` 一起成为 core 的必要条件之一，进一步压低 core 数量。
   - 归属：**领域插件**（“本轮中心问题的布尔标签”应是 menu 里可配的一项）。

4. 🔴 **`01b_triage.md:16`** 与 **`01c_affiliation.md:13`**：会议白名单 `NeurIPS, ICML, ICLR, ACL, EMNLP, NAACL, COLM, AAAI, IJCAI, TMLR, JMLR, TACL`
   - 怎么坏：**视觉论文中了 CVPR/ICCV/ECCV 不算 venue_accepted**；检索类的 SIGIR/KDD/WWW、安全类的 S&P/CCS/USENIX Security、机器人类的 CoRL/RSS 同样不算。这些论文只能靠 top_lab 或 solid_score ≥ 4 进 core。
   - 归属：**领域插件**（白名单按领域给；“workshop 不算、Findings 算”这条规则留内核）。

5. 🔴 **`pipeline/triage_merge.py:18,48-51`**
   - 原文：`MECH = {"stopping", "mid-run-share", "new-evidence", "analysis", "cascade", "weak-vs-strong"}`；rescue 条件含 `tf == "partial"`、`compute_accounting in ("dollars", "tokens", "calls", "samples")`、`cheap_vs_strong or controlled_baseline`
   - 怎么坏：rescue 只救“训练了选择器但策略冻结”的 run3 情形。RL 论文的算力单位是 GPU·小时 / 环境步数，不在名单里，不会被救。
   - 归属：**领域插件**（规则体可留内核，常量从配置读）。

6. 🟡 **`01_purchase.md:16`**：`verify the first-version date with the arXiv API ... Out-of-window = excluded`
   - 怎么坏：只发在 ACL Anthology / CVF Open Access / OpenReview 而无 arXiv 版的论文全部出局。`:20` 已有 OpenReview 补充通道，但 `:25` 写明这些论文“stay OUT of corpus.json”。
   - 归属：**通用内核**（数据源适配，与 AI 子领域无关；视觉、安全领域无 arXiv 版的比例更高，待核）。

### 2.2 抽取 schema + DSL

7. 🔴 **`config/schema.md:11-22`**（算子表 Gen/Comm/Revise/Critique/Loop/Stop/Share/Escalate/Agg/Tool）
   - 原文一例：`| Comm(topo, content) | agents see others' outputs | topo in {all, ring, star, pair, judge, sparse} ...`
   - 怎么坏：DPO vs PPO、LoRA vs 全参、ViT vs CNN、RAG 的“检索器 + 重排 + 生成器”都写不成这套程序；抽取 agent 只能硬塞成 `Gen(1,plain,same); Agg(single)`，积木库（`build_db.py:248-257` blocks.json）全是 Gen + Agg，P2/R2 角度无事可做。
   - 归属：**领域插件**（算子表）；“每个方法写成算子程序”这个做法本身是**通用内核**。

8. 🔴 **`config/schema.md:30-44`**（calls_per_question、rounds、calls_unit 等算力口径）
   - 原文：`calls_per_question = total LLM calls per test question (e.g. debate 3 agents x (1+2 rounds) = 9).`
   - 怎么坏：**RLHF 论文没有 calls_per_question**（算力在训练端：GPU·小时、样本数、偏好对数）；视觉推理的算力是 FLOPs / 分辨率。全部记 null → pairs 无法判 matched（`build_db.py:300-303`），设计原则 6（`CLAUDE.md:23` 按算力归一化）在新领域落空。
   - 归属：**通用内核**保留“必须有一个算力列 + 口径来源”；**领域插件**给出算力单位与公式。

9. 🔴 **`config/schema.md:67-81`**（方法字段）
   - 原文：`"family": "interactive | sample-aggregate | self-refine | single | cascade"`，以及 `stop_rule / mid_run_share / evidence / selector / interaction_partner / pool_members / n_agents / rounds / heterogeneous_models`
   - 怎么坏：检索论文的关键变量（检索器类型、top-k、chunk 大小）、RL 论文的（奖励模型、KL 系数、on/off-policy）都没有字段，只能塞进 extractor_notes，建库时丢失。
   - 归属：**领域插件**（字段组）；`is_ablation_of / actually_run / copied_from / method_origin_paper`（`:73-76`）是**通用内核**，任何领域都要。

10. 🔴 **`config/schema.md:102-104`** 与 **`02_extract.md:16`**：
    - 原文：`record EVERY row of the main results tables for the reasoning benchmarks, including ALL baselines (single agent, CoT, self-consistency, other multi-agent methods)`
    - 怎么坏：视觉论文主表是 ImageNet / COCO，按字面算“非 reasoning benchmark”，可能被少记甚至不记（取决于 agent 的理解，待核）。
    - 归属：**领域插件**（基准范围由 menu 给）；“记全部行、基线同样重要”是**通用内核**。

11. 🔴 **`config/schema.md:112-113`**：`has_vote_baseline (paper): true if the paper itself runs majority vote / self-consistency`；**`02_extract.md:17`** 同。
    - 怎么坏：只对推理时扩展有意义，其他领域恒为 false，`build_db.py:159` 白存一列。
    - 归属：**领域插件**。

12. 🔴 **`pipeline/check.py:49-50`**（抽取闸门）
    - 原文：`if m.get("family") not in ("interactive", "sample-aggregate", "self-refine", "single", "cascade")`
    - 怎么坏：只要 schema 加了新族（如 `rlhf-online` / `dpo-offline`），**抽取闸门直接 FAIL**，pipeline 停在第 5–9 步。必须和 `build_db.py:26` 的 FAMILIES 一起改，两处是同一个常量的两份拷贝。
    - 归属：**通用内核**的检查逻辑 + **领域插件**的枚举（应从同一个配置读）。

13. 🟡 **`pipeline/init_run.py:39`**：`shutil.copy(os.path.join(PROJ, "config", "schema.md"), os.path.join(R, "config", "schema.md"))`
    - 怎么坏：schema 只有项目级一份，没有 `--schema` 参数。想给 RAG 跑一轮，只能改项目里的 schema.md，会改到后面所有 run。
    - 归属：**通用内核**（加按 run 覆盖的入口即可，属结构问题）。

14. 🔴 **`config/schema.md:152-163`**：积木库槽位 ↔ DSL 对照表，对应 AggAgent 的 `brick_library`。
    - 归属：**领域插件**（只对多智能体并行聚合有意义）。

### 2.3 建库 `scripts/build_db.py`

15. 🔴 **`:25-26`**：`OPS = ["Gen", "Comm", ...]`，`FAMILIES = {"interactive", "sample-aggregate", "self-refine", "single", "cascade"}`
    - 怎么坏：同第 7、12 条；不在 FAMILIES 里的族会退到 `infer_family`（`:170-177`），后者只会产出这 4 个族之一（`:53-63`），新族被**悄悄改写**成旧族。
    - 归属：**领域插件**。

16. 🔴 **`:30-92`**：`parse_program` / `infer_family` / `calls`（按 `n × (1 + rounds)` 推调用数）
    - 原文：`return float(n * (1 + (r or 0)))`（`:92`）
    - 怎么坏：完全基于辩论 / 投票的程序语法。非 DSL 领域推不出调用数，也推不出族。
    - 归属：**领域插件**（解析器随算子表走）。

17. 🔴 **`:98-111`**：`norm_model` 的 gpt-3.5 别名表、`BENCH_ALIAS = {"gsm8k": ..., "mmlu": ..., "hotpotqa": ...}`
    - 怎么坏：不会报错，但视觉 / 检索基准的变体（`coco-val2017` vs `coco`）不会被合并，同一基准分成多格，pairs 的同格比较变少。
    - 归属：**领域插件**（别名表）；“只做精确别名、保留子集”（`:107` 注释）是**通用内核**。

18. 🔴 **`:210`**：`if v is not None and 0 < v <= 1.0 and (r.get("unit") in (None, "", "fraction")): v = v * 100`
    - 怎么坏：**RL 的归一化回报 0.73、检索的 nDCG@10 = 0.41、相关系数 0.62 都被悄悄乘 100**；同一张表里 1.2 的回报又不乘，同格数值尺度混在一起。
    - 归属：**通用内核**（应改成只对声明为比例的指标换算；这是 bug 级问题，不只是主题耦合）。

19. 🔴 **`:265-267`**（task_metric 正则）
    - 原文：`ACC = r"(^acc|accuracy|^f1|\bf1\b|^em\b|exact match|pass@1|solve|correct|pass rate|success rate)"`
    - 怎么坏（已在本机用同一正则实测）：`average return`、`reward`、`ndcg@10`、`recall@10`、`mAP`、`mIoU`、`win rate`、`harmlessness` 全部 **False** → 这些行不进 pairs（`:308`）。更糟的是 `attack success rate` 判 **True**，被当作越高越好；而 `scripts/build_cheap_strong.py:58` 的 LOWER 正则明确把 `attack success` 列为越低越好。**同一个安全指标，两张表符号相反。** `win rate` 同理：`build_db.py:262-264` 注释说要排除，`build_cheap_strong.py:59` 的 HIGHER 却包含 `win`。
    - 归属：**通用内核**（“指标方向”应是一张统一的表，两处共用）+ **领域插件**（各领域的指标名）。

20. 🔴 **`:280-382`**（pairs 构造）
    - 原文：`inter = g[g.family == "interactive"]`（`:310`），`sa_all = g[(g.family == "sample-aggregate") ...]`（`:316`），`SA_BASELINE ... pure_vote`（`:280-288`）
    - 怎么坏：**整个 pairs 表只回答“交互方法比同调用投票好多少”**。RLHF / 检索 / 视觉语料里没有 interactive 方法 → `pairs` 为 0 行 → `:383` 写出一个只有换行符、没有表头的 csv。已实测：`pd.read_csv` 读这种文件抛 `EmptyDataError`。下游无条件读 pairs.csv 的有 `scripts/score_candidates.py:60`、`scripts/robustness.py:30`、`scripts/brick_reuse.py:36`、`scripts/chessboard.py:32` → **打分、稳健性、棋盘全部崩溃**。`pipeline/check.py:76-77` 的 db 闸门只查文件存在，会放行，错误到下一阶段才暴露。
    - 归属：配对机制“同 paper × model × benchmark × metric、按算力就近配基线”（`:309`、`:291-304`）是**通用内核**；“谁是处理组、谁是基线”（interactive vs 投票）是**领域插件**，应像棋盘的 `axes.json` 一样写成配置。

21. 🔴 **`:394-442`**（summary.json）：`share_of_gain_explained_by_sa`、`pw_delta_vs_sa_*` 等
    - 怎么坏：pairs 为空时这些键整体缺失（`:406` 有 `if len(D)` 保护，不崩）；P1 角度（`angles.json:3`）指定先读 summary.json，读到的只有计数。
    - 归属：**领域插件**（摘要统计随配对规则走）。

### 2.4 打分检验

22. 🟡 **`scripts/score_candidates.py:60,62`**
    - 原文：`T = {k: pd.read_csv(...) for k in ["pairs", "results", "methods", "papers"]}`；`for k in ["cheap_vs_strong", "results_usd", "qual_findings"]:`
    - 怎么坏：pairs 为空即崩（见第 20 条）；领域新增的表（如 RL 的 `reward_ablation.csv`）不在名单里，测试写 `"table": "reward_ablation"` 会 KeyError。
    - 归属：**通用内核**（改成读 db/ 下全部 csv、空表容错）。MDL 计算（`:114-146`）、时间切分（`:93-100`）、独立组（`:76-89`）、按论文多数（`:103-109`）全部**通用**。

23. 🔴 **`scripts/meta_analysis.py:10-11,16,189-195`**
    - 原文：`内置簇（pairs 表，算力匹配 match_status == 'matched'）`；`matched: SA baseline is pure vote (stats_audit G-1)`
    - 怎么坏：内置簇全建在 delta_vs_sa 上，新领域为空。`--candidates` 模式（`:12`、`:200-201`）只接受 `pairs` 或 `cheap_vs_strong` 表。
    - 归属：随机效应合并本身是**通用内核**；内置簇是**领域插件**。

24. 🟡 **`pipeline/templates/04_review.md:15`**：`check the DB itself for labelling errors (family, actually_run, benchmark merging, calls)`；`:13(e)` 的 rival / critic 分法针对“某类方法被高估”的论点。
    - 归属：大体**通用内核**；列举的字段随 schema 变，**待核**是否需要按领域替换。

### 2.5 提议角度 `pipeline/angles.json`

25. 🔴 **`:4`（P1）、`:8`（P2）、`:12`（P3）**
    - 原文（P1）：`at what compute (calls per question), how do they vary with rounds, number of agents, model heterogeneity, aggregation rule and baseline strength`
    - 怎么坏：在 RLHF 语料上，提议 agent 被要求按“轮数、agent 数”切数据，可切的维度不存在，只能产出 GCD = 1 的空泛候选（`03_propose.md:24` 会拒掉）。
    - 归属：**领域插件**。

26. 🔴 **`:24,28,32`（R1–R3）、`:36`（R4）**：写死 cheap_vs_strong.csv、`model_prices_README.md`、`conformity, error propagation, degeneration of thought, sycophancy, overthinking`，并引用 G1–G7。
    - 归属：**领域插件**（R 系列是 run3 主题专用）。

27. 🟢 **`:15-20`（N1/N2）、`:38-41`（C0）**：不含主题词，任何领域可直接用。可作为新领域的默认角度。

28. 🟡 **`03_propose.md:23`**：`The GCD of a sub-family (only stopping rules, only cascades, only heterogeneous ensembles) is longer and narrower`
    - 归属：只是举例，**通用内核**（换成领域无关的例子即可）。GCD 准则（`:21-25`）和 L1/L2/L3 分层（`:15-19`）是全 pipeline 最通用的部分。

### 2.6 棋盘

29. 🟡 **`scripts/chessboard.py:84-108`**（`vote_deltas`）
    - 原文：`"""照 build_db.py pairs 规则：同 paper/model/benchmark/metric 内，配调用次数最近的投票列基线。"""`
    - 怎么坏：基线列名可配（`ax["delta"]["baseline_col"]`），但比较量写死为 `calls`（`:97-103`）。RL 语料的 calls 全空 → 每格 Δ 都是 0 组 → `blanks.csv` 的 `no_vote_delta` 类（`:222`）每格都“空白”，连片空白失去意义。
    - 归属：**通用内核**（把 `calls` 也改成 axes.json 里的 `cost_col`）。

30. 🟡 **`scripts/chessboard.py:182-184`**：`[r["col"] for r in ax["col_rules"] if any(c.get("in") == ["interactive"] for c in r["when"])]`
    - 怎么坏：没有 interactive 规则时 cross_check 列为空（不崩）。另外 `:32` 无条件读 pairs.csv，pairs 为空时崩（第 20 条）。
    - 归属：**通用内核**（已有 `cross_check_cols` 覆盖入口，`:183`）。

31. 🟡 **`scripts/chessboard.py:294-298,354,361,365`**、**`scripts/chessboard_cards.py:218-254`**：页面和输入包写死中文“Δ投票”“落不进 4 列”“同调用投票对照”。
    - 归属：**通用内核**（文案应读 axes.json 的列标签；只影响阅读，不影响计算）。

32. 🔴 **`paper_alchemy_run3/chessboard/axes.json`**：rows = exact / code / open，`row_rules` 是 GSM8K / SWE-bench 等基准正则，`cols` = 投票 / 打分挑选 / 同伴辩论 …
    - 归属：**领域插件**（设计上就是插件，不算缺陷）。新领域要新写一份。

### 2.7 美元层

33. 🔴 **`scripts/add_dollars.py:1-12,33-38`** + **`config/model_prices.csv`**（`model_raw, usd_per_m_input, usd_per_m_output`，LLM API 价）
    - 原文：`tokens : tokens reported for this row x list price of the row's model`
    - 怎么坏：视觉检测模型、RL 训练没有“每题 token”，`usd_basis` 全部为 `none`。
    - 归属：**领域插件**（整层可选）。

34. 🔴 **`scripts/build_cheap_strong.py:10-15,147-162`**：arm C = `{sample-aggregate, interactive, self-refine}` 且 calls ≥ 2；arm S = `single`；`:80-100` 的 TASK_RULES 只有 code / math / open-ended / qa 四类。
    - 怎么坏：整个表就是 run3 的中心问题“便宜多次 vs 贵一次”。其他领域为 0 行。
    - 归属：**领域插件**。

35. 🔴 **`scripts/ladders.py:7,68`**：`MULTI = {"sample-aggregate", "interactive", "self-refine"}`，“下一档便宜调用每美元买到多少分”。
    - 归属：**领域插件**（“同格相邻预算的边际收益”这个想法可推广到训练步数、检索 top-k，待核）。

36. 🟡 **`pipeline/init_run.py:35-36,41-42`**：REQUIRED 里有 `add_dollars.py`、`build_cheap_strong.py`，并必拷 `model_prices*`；**`RUNBOOK.md:31`** 的美元层行没有标“可选”。
    - 归属：**通用内核**（把美元层标为可选插件）。

### 2.8 筛选 / 定性 / 挂靠的其余点

37. 🔴 **`pipeline/templates/02b_qualitative.md:12`**（variable 枚举 `n_agents | rounds | n_samples | temperature | heterogeneity | ... | topology | roles`）与 **`pipeline/qual_merge.py:15`**（同一枚举，越界即报错，见 `:31`）。
    - 怎么坏：RLHF 的关键变量（KL 系数、奖励模型大小、偏好数据量）只能标 `other`，`scripts/contradiction_pairs.py:31` 按 variable 分组找矛盾，全挤进 `other` 一组。
    - 归属：**领域插件**（variable 枚举）；shape / quality / evidence 枚举（`02b:14-21`）是**通用内核**。

38. 🔴 **`02b_qualitative.md:30`**：`Skip findings outside this pipeline's task domain (e.g. image classification, sentiment) unless the paper's main method is in scope.`
    - 怎么坏：**视觉领域的定性发现会被按字面跳过**。
    - 归属：**领域插件**（排除范围应来自 menu）。

39. 🔴 **`04b_attach.md:4,9,24`**：路径写死 `synthesis/v2/attach/`，目标写死 `"G1".."G10"`，并预设 `debate beating vote is a counter or bound to G4`；**`05b_attach_write.md:9-10,14,17`** 同样写死 `synthesis/v2/` 与 G3 的例子。
    - 归属：**领域插件**（run3 残留，应改为占位符）。

40. 🔴 **`pipeline/templates/13_preapproval.md:10`**：`brick_library/constraints.json（/Users/xzc/HITSZ/000Project_AggAgent/brick_library/）的 active 项（当前 train_free、no_prune、same_base_model）`
    - 怎么坏：AggAgent 项目的约束混进了通用模板；对齐 / RL 实验卡会被判 violates train_free。
    - 归属：**领域插件**（项目约束）。

41. 🔴 **`10_institution_catalyst.md:11`**（问题举例：共享错误、羊群、何时停止辩论）、**`:34`** 表头“多智能体里卡住的地方”、**`:42-54`** 问法种子 S1–S13（混池、聚合器缓冲层、拓扑）；**`15_problem_lift.md:11`**（样板 = 便宜模型投票 vs 强模型单次 / Condorcet）。
    - 归属：方法（借人类制度当巨人肩膀、升维看问题）是**通用内核**；例子与种子是**领域插件**。对 RL / 对齐领域，种子大多仍可用（激励、委托代理），待核。

42. 🟡 **`09_novelty_check.md:14`**：写死 `库 = 我们自己的 513 篇抽取 + 2,237 篇语料`（run3 数字）；**`pipeline/novelty_check.py:11`** 注释同。代码本身按 `RUN_DIR` 读（`:39-42,100`），只是文字残留。
    - 归属：**通用内核**（改文字）。ScholarCatalyst 19 万篇库覆盖哪些 AI 子领域，**待核**。

43. 🟡 **`scripts/funnel.py:12,115-117`**：pairs 站把 `cheap_vs_strong.csv` 算进比较表。其余逻辑通用。**`scripts/convergence_overlap.py:5`**、**`funnel.py:4`** 默认路径 `synthesis/v2/`（run3 习惯，有回退）。
    - 归属：**通用内核**。

44. 🟡 **`scripts/merge_programs.py:40`**：`OPS = {'Gen', 'Comm', ...}`（归并按 DSL 编辑距离）。
    - 归属：归并算法**通用内核**，算子集**领域插件**（与第 7 条同源）。

45. 🔴 **`scripts/brick_overlap.py`**（RUNBOOK `:27` 调用 `/Users/xzc/HITSZ/000Project_AggAgent/brick_library`）：AggAgent 积木库专用。已标“可选”。
    - 归属：**领域插件**。

### 2.9 写作 / 评估

46. 🟡 **`05_write.md:15`**：`methods rewritten in the operator DSL; compute normalization by calls per question; within-paper comparisons`
    - 怎么坏：新领域的方法节会照写“calls per question”，与实际口径不符。
    - 归属：**通用内核**（改成“按 schema 的算力列归一化”）。

47. 🟡 **`05_write.md:20`**：`about 2,000 items for a 2-point difference at typical accuracy`
    - 怎么坏：RL 回报、检索 nDCG 不是比例，这个样本量换算不成立。
    - 归属：**领域插件**（统计口径随指标类型走）。

48. 🟡 **`pipeline/check.py:14,108`**（`\b\d{4}\.\d{4,5}\b` 只认 arXiv 编号）、**`pipeline/regen_appendix_a.py:80-99`**（`:82` 读 `triage/paper_tier.csv`，`:87` 断言每篇被引论文都在其中，`:99` 读 corpus 的 `method_family`）
    - 怎么坏：引用非 arXiv 论文时，check.py 认不出，可能漏检语料外引用；跳过 triage 的小语料 run 生成附录 A 时会触发断言（RUNBOOK 阶段表没把 triage 列为必经，**待核**是否每轮都会有）。
    - 归属：**通用内核**。

49. 🟡 **`docs/03_pipeline设计_22步.md:7,73,76`**：设计文档把领域写成“多智能体、test-time scaling、并行推理”，DSL 例子是自洽 / 辩论，算力换算写的是 token、调用次数、采样数。**`CLAUDE.md:6`** 同。只是文档。
    - 归属：文档。

50. 🟢 **`config/menu.md`**：整份是第一轮主题，但它本来就是每轮输入（RUNBOOK `:17`）。不算缺陷；`check.py:13-16` 只解析 “reserved for evaluation” 和日期窗，与主题无关。

---

## 3. 已经通用、可直接复用的部分

| 组件 | 位置 | 说明 |
|---|---|---|
| 运行目录隔离 + 冻结脚本快照 | `pipeline/init_run.py:24-54` | 除第 13、36 条外通用 |
| 提示词渲染与留痕 | `pipeline/render.py:1-41` | 无主题词 |
| 闸门：语料日期窗与锁定答案 | `pipeline/check.py:24-38`、`:104-128` | 通用（编号格式见第 48 条） |
| 闸门：抽取 trace 一篇一行 | `pipeline/check.py:54-73` | 通用 |
| 闸门：候选 level / tests、重打分 | `pipeline/check.py:84-102` | 通用 |
| 闸门：独立组数字核对 | `pipeline/check.py:129-163` | 通用 |
| MDL 打分、时间切分、按论文多数、独立组、例外全量落盘 | `scripts/score_candidates.py:24-156`（除 `:60,62`） | 通用内核的核心 |
| 留一组稳健性 | `scripts/robustness.py`（读 db 的方式同第 22 条） | 通用 |
| 独立组（共享 ≥ 2 作者） | `pipeline/independence_graph.py:1-79` | 与领域无关 |
| 审查纠错层 | `scripts/apply_corrections.py:1-44` | 表名、字段名都是参数 |
| 测试支撑行重合度 | `scripts/convergence_overlap.py`、`scripts/support_rows.py` | 通用 |
| 矛盾对 | `scripts/contradiction_pairs.py` | 逻辑通用，依赖第 37 条的 variable 枚举 |
| 漏斗台账 | `scripts/funnel.py`（除 `:115-117`） | 通用 |
| 采购模板 | `pipeline/templates/01_purchase.md:1-32` | 通用（arXiv 依赖见第 6 条） |
| 机构复核 | `01c_affiliation.md`（除 `:13` 白名单） | 通用 |
| solid 判据、“workshop 不算 / Findings 算” | `01b_triage.md:18,25` 后半、`triage_merge.py:46` | 通用 |
| 抽取的“去包装 / 不造数 / 基线同样重要 / 一表一格” | `config/schema.md:3-5,101-107`；`02_extract.md:13-15,21-28` | 通用 |
| 方法溯源字段 | `config/schema.md:73-76,126-133`（is_ablation_of / actually_run / copied_from / method_origin_paper） | 通用 |
| 提议的 GCD 准则与 L1/L2/L3 分层 | `03_propose.md:13-37` | 通用（第 28 条只是例子） |
| 去主题角度与闭卷对照 | `angles.json:14-21,38-41`（N1/N2/C0） | 通用 |
| 对抗审查流程 | `04_review.md:11-23` | 通用（第 24 条字段待核） |
| 复现价值排序 | `16_replication_value.md` | 通用（引 RECLAIM 的成功率是 AI 论文均值，待核是否分领域） |
| 写作规范与检查 | `pipeline/writing_guide/*`、`pipeline/check_writing.py`、`05_write.md:11-29`（除 `:15,20`） | 通用 |
| 溯源、翻译 | `06_provenance.md`、`07_translate.md`、`pipeline/check_translation.py` | 通用 |
| 评估与“发现 vs 记得”对照 | `08_evaluate.md:1-38` | 通用 |
| 棋盘引擎（落格、空格、连片空白、证据三档） | `scripts/chessboard.py:35-81,125-139,145-176,216-245` | 规则全在 axes.json，是“内核 + 插件”分法的样板 |
| 出卡 / 格雷粗筛 / 决策页 | `11_card.md`、`12_gray_screen.md`、`scripts/build_decision_page.py` | 通用（12 的 `data` 格写“调用数不明”，只是措辞） |
| 设想查重脚本 | `pipeline/novelty_check.py`（除注释 `:11`） | 通用 |

---

## 4. 最小切口：只改 3 处让另一个 AI 领域跑起来

判据：拿一份**没有任何 interactive / sample-aggregate 方法**的语料（例如 RLHF：DPO vs PPO vs RLOO），沿闸门链 `corpus → triage → extract → db → candidates → final → paper` 走一遍，找出**第一个硬挡住**的位置。

| 顺序 | 卡在哪 | 现象 | 最小改法（只说位置，不展开设计） |
|---|---|---|---|
| ① 筛选 | `01b_triage.md:24-25` + `triage_merge.py:18,48-51` | “training the policy only”= out，core 要求 train_free → **0 篇进抽取** | 把 out / core 判据和族表改成从 menu 读（menu 加一节“core 条件”），保留 solid 判据 |
| ② 抽取闸门 | `check.py:49-50`（与 `build_db.py:26` 同一常量） | 新族（如 `offline-preference` / `online-rl`）→ **extract 闸门 FAIL**；就算改 schema.md 也过不去 | 族枚举改成从 `<RUN_DIR>/config/schema.md`（或一份 families 配置）读，check.py 与 build_db.py 共用。schema 的算子表与族表随之按领域写，`init_run.py:39` 允许传入领域 schema |
| ③ 建库配对 | `build_db.py:265`（指标正则）+ `:310,316`（interactive vs sample-aggregate）+ `:383`（空表无表头） | `average return`、`win rate` 不算任务指标；没有 interactive → pairs 0 行 → 无表头 csv → **`score_candidates.py:60` 抛 EmptyDataError**，打分 / 稳健性 / 棋盘全崩 | 把“处理组族 / 基线族 / 算力列 / 指标方向表”提成配置（像 axes.json 那样），并在 pairs 为空时照样写出表头 |

做完这 3 处，另一个领域可以走到成文：提议用 N1/N2 角度（`angles.json:14-21`），测试写在 `results` / `methods` / `papers` 表上，美元层与棋盘先跳过。

**这 3 处不解决的问题**（不影响“能跑通”，影响“跑出的东西好不好”）：
- `build_db.py:210` 的 0–1 值乘 100（第 18 条）和 `attack success rate` 符号相反（第 19 条）：不报错，但**数字会悄悄错**。做安全领域时应和第 ③ 处一起修。
- 算力归一化（`CLAUDE.md:23` 原则 6）在训练类领域没有对应的列（第 8 条），pairs 只能全是 `unknown`。
- 会议白名单缺 CVPR / KDD / S&P 等（第 4 条），视觉、检索、安全领域的 core 篇数会偏少。

---

## 疑似 dead / 残留（未删，待确认）

- 疑似 dead：`scripts/build_db.py:329-351`（旧 pairs 逻辑整段注释，未删，待确认）
- 疑似 dead：`scripts/score_candidates.py:28-29,54,76,130-135`（旧代码注释，未删，待确认）
- 疑似 dead：`pipeline/init_run.py:27-28`（旧四脚本清单注释，未删，待确认）
