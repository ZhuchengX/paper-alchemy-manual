# 论文炼金 · 使用说明书（网页版）

在线阅读：https://zhuchengx.github.io/paper-alchemy-manual/

这里只放说明书和它链接的调研页，由维护者从私密仓库 `ZhuchengX/paper-alchemy` 自动发布，不要直接改这里的文件。

## 更新记录（新的在上）

### 2026-10-27 18:10 · 流程图每段卡先写目的

- [全流程图](https://zhuchengx.github.io/paper-alchemy-manual/#flow)：每段卡最上面两行改成「目的」「做完得到什么」，先说这一段为什么做、做完拿到什么，下面的技术细节照旧保留。
- 段 2 改名「启发池、采购与初筛」：图上点启发池（S00a）跳到的就是这一段，卡里现在先讲启发池。

### 2026-10-27 17:30 · B9 完成：按菜单详略定提议路数，启发池接上构思出卡

- [进度表](https://zhuchengx.github.io/paper-alchemy-manual/#progress)：B9 启发池由 🟡 改为 ✅。
- 提议（S12）路数随菜单详略变：方案完整 3 路、较模糊 4 路、只给一句话 6 路。档位写在闸①的锚回显里，不对就在菜单加 `## Detail level` 一节改；`--set ANGLES=P1,E1` 可直接指定。做法见[设计细节 · 启发池](设计细节.html#b9)。
- 新增两路提议：E1 从启发池草稿出发，E2 从读者要做的决定倒推。
- 构思出卡（S25）也读启发池，草稿只当候选起点；卡用到哪份草稿要写出文件名。

### 2026-10-27 16:30 · 全流程图改成 Team Paper Lite 同款

- [全流程图](https://zhuchengx.github.io/paper-alchemy-manual/#flow)：改为一张完整 graph，排版与配色照 Team Paper Lite 说明书：每段一条色带，阶段是带编号的方框，★ 人工闸 ①–⑦ 是琥珀粗框，◆ 条件闸 ⑧ ⑨ 只在出事时停，┄ 左侧虚线是闸答 no 时的返回环，右栏小牌是机器闸。点任一段方框展开它的卡；深色模式有单独配色；手机上图在框内左右滑。

### 2026-10-27 15:20 · 阅读页文风闸 + trial 重打包

- 阅读页（S23r）多了文风检查，不过就停：卡片「一句话」要写结论，不能只列计数；「边界」≤ 100 字；不许出现流水线内部词；汉字与英文、数字之间留空格；日期写成 `2024-01`。规则照 user 认可的手写版定。
- [case 现状](https://zhuchengx.github.io/paper-alchemy-manual/#cases)：trial_harness 按新规则重写阅读页（codex 1 次）并重打交付包，295 个数字全部一致；只剩翻译（S22）待重跑。

### 2026-10-27 · 第四批 4b + 说明书精简

- 全篇压缩：每节只留要用的信息，重复说明删掉；模块细节在设计细节子页。
- [全流程图](https://zhuchengx.github.io/paper-alchemy-manual/#flow)：改为竖排一张图，段 → 闸 → 段，手机上也能顺着看；每段的卡默认折叠，点段展开。
- 新增 [case 现状](https://zhuchengx.github.io/paper-alchemy-manual/#cases)：每个 case 跑到哪、还差什么、台账几行。
- [你在哪些点介入](https://zhuchengx.github.io/paper-alchemy-manual/#gates)：补上条件闸 ⑧（新眼终审要求重写）和 ⑨（中文稿出现禁用译法），各最多回 1 次。
- 第 6 段新增中文阅读页（S23r）：每条结论按独立来源组数标强度（🟢 ≥7 组、🟡 4–6、🟠 1–3），进交付包根目录 `reader_zh.html`。
- S22 翻译改为按本轮主题生成术语表（保留英文 / 禁用译法），中文稿出现禁用译法就停在 ⑨。
- 数字机核不再把 `173.84M` 切成 173、把 `T-1` 读成 −1。

### 2026-10-25 21:10 · 第四批 4a：正文不再被悄悄改、判决有去向

- 正文每改一次留一版在 `traces/paper_versions/`（S20 / S21 / S21b），`paper/gold_paper.md` 始终是最新版；S21b 的修订稿自动并回正文。
- 顺序改为 S20 成文 → S21 溯源 → S21b 新眼审稿 → S20b 数字机核（读正文的最后一站）→ S22 → S23b。
- S21b 有 REOPEN 时停在条件闸 ⑧：`--answer ok` 接受，`--answer no:reopen` 回成文（最多 1 次）。
- 闸号一律按展示号 ①–⑧；`--set` 会记进 run.json；已完成阶段要重跑时先列清单停下，加 `--rerun-stale` 才跑。
- 新命令 `orchestrate.py accept RUN_DIR SID --reason "…"`：确认产物没问题时只重盖章、不重跑（旧运行目录用 `accept RUN_DIR --migrate` 迁到新指纹）。

### 2026-10-24 19:45 · 说明书瘦身：主页只留要用的，细节进子页

- [这是什么](https://zhuchengx.github.io/paper-alchemy-manual/#what)：新增「进度表」（[直达](https://zhuchengx.github.io/paper-alchemy-manual/#progress)），一张表看 B0–B9 各做到哪、T1–T3 还在规划。
- [跑第一轮](https://zhuchengx.github.io/paper-alchemy-manual/#first)：围绕一条命令 `orchestrate.py` 讲；手动逐步跑的写法折叠在「展开」里。
- [它怎么思考](https://zhuchengx.github.io/paper-alchemy-manual/#think)：压成 4 条核心想法 + 流程图，每条下面一行小字注明出处。
- [你在哪些点介入](https://zhuchengx.github.io/paper-alchemy-manual/#gates)：7 个人工闸改成一张表：看什么、只回答哪一个问题。
- B2–B9 各模块的完整做法、每个闸的详细说明、完整菜单和完整目录树，原文挪到新子页「设计细节」（侧栏最下方），一字未删。
- [更新记录](https://zhuchengx.github.io/paper-alchemy-manual/#news)：从页首挪到页末并整体折叠，点开看最新 3 条。

### 2026-10-24 17:25 · B9 启发池：先发散，再炼金

- [跑第一轮](https://zhuchengx.github.io/paper-alchemy-manual/#first)「④ 启发池」：改为已实现。新增「怎么跑」：编排器在 S00a 自动做完，约 27 次 codex；跑完看 `inspire/INDEX.md`（草稿目录）和 `inspire/diversity.md`（去重后几份、覆盖几个算子）。启发池现在只交给提议者（S12），构思出卡（S25）还没接上。
- [跑第一轮](https://zhuchengx.github.io/paper-alchemy-manual/#first)「④ 启发池」：新增「还没做」。B9 另一半「按菜单详略分三档」仍在规划。
- [输出目录详解](https://zhuchengx.github.io/paper-alchemy-manual/#output)：`inspire/` 下新增 `INDEX.md` 和 `diversity.md`。
- [流程图模块 1](https://zhuchengx.github.io/paper-alchemy-manual/#fl-m1)：S00a 由规划中改为已实现，排到 S00 之后。

### 2026-10-24 16:42 · B8 交付包：解包即可复算全部数字

- [它怎么思考](https://zhuchengx.github.io/paper-alchemy-manual/#think)：新增「最后一步打包交付」一段。交付包含正文、PDF、证据包、最终报告。在禁读原机器文件的沙箱里，包内脚本重新打分、逐条核对正文数字，全部 MATCH 才算通过。报告把数字分为「从 db 重新算出」和「读随包输入」两类。摘要可交没参与写作的 reader 重写，不许含数字。
- [输出目录详解](https://zhuchengx.github.io/paper-alchemy-manual/#output)：目录树新增 `deliver/`，`traces/` 下新增编排器的 `state.json`。
- [流程图模块 5](https://zhuchengx.github.io/paper-alchemy-manual/#fl-m5)：S23b 由规划中改为已实现。

### 2026-10-24 16:28 · B7 编排器：一条命令跑全流程

- [跑第一轮](https://zhuchengx.github.io/paper-alchemy-manual/#first)「②」：新增 `orchestrate.py` 用法。它按阶段表逐个执行，每阶段过闸才往下；改了哪个模板或脚本，只重跑用到它的阶段。
- [跑第一轮](https://zhuchengx.github.io/paper-alchemy-manual/#first)「③ 跟跑」：中途离开后，再跑同一条命令接着来，被中断的阶段重跑。
- [你在哪些点介入](https://zhuchengx.github.io/paper-alchemy-manual/#gates)：7 个闸统一由编排器停下提问，用 `--answer` 回答；auto 模式直接放行，记为 auto-approved。
- [它怎么思考](https://zhuchengx.github.io/paper-alchemy-manual/#think) 流程图说明：新增「谁沿着这张图跑」，说明编排器按图上从左到右执行、停在每个 ★ 闸。

### 2026-10-24 16:05 · B6 可行性闸 + 扩展牌组

- [它怎么思考](https://zhuchengx.github.io/paper-alchemy-manual/#think)：新增「架构师之前」一段。先判这一轮撑不撑得起金子：撑不起判 KILL，写 null memo，不成文；只撑得起边界则只写描述性论文。再给每条 G 发 7 张扩展卡，每张判 holds / narrows / descriptive_only / n_a / needs_call。
- [你在哪些点介入](https://zhuchengx.github.io/paper-alchemy-manual/#gates)：闸由 6 个增为 7 个。新增闸⑦「哪几张扩展卡进论文？」。
- [输出目录详解](https://zhuchengx.github.io/paper-alchemy-manual/#output)：目录树新增 `deck/`。
- [流程图模块 4](https://zhuchengx.github.io/paper-alchemy-manual/#fl-m4)：S18b 由规划中改为已实现。

### 2026-10-24 15:40 · 新增更新记录

- [更新记录](https://zhuchengx.github.io/paper-alchemy-manual/#news)：说明书最前面新增本节，列出每次发布改了哪里，新的在上，最新 3 条展开。侧栏日期改为最近一次更新时间。网页仓库 README 同步这份记录。

### 2026-10-24 14:45 · B5 架构师三棱镜 + 禁说清单

- [它怎么思考](https://zhuchengx.github.io/paper-alchemy-manual/#think)：新增「成文之前」一段。先零调用从打分生成禁说清单，再由 novelty / narrative / impact 三个架构师各出一份骨架，decider 选一份；成稿由脚本逐条核对禁说。
- [你在哪些点介入](https://zhuchengx.github.io/paper-alchemy-manual/#gates)：闸由 5 个增为 6 个。新增闸⑥「用哪份骨架写？」，时间上在闸⑤之前。
- [输出目录详解](https://zhuchengx.github.io/paper-alchemy-manual/#output)：目录树新增 `architecture/`。
- [流程图模块 5](https://zhuchengx.github.io/paper-alchemy-manual/#fl-m5)：S19b 由规划中改为已实现。

### 2026-10-24 10:32 · B4 五角色新眼审稿

- [它怎么思考](https://zhuchengx.github.io/paper-alchemy-manual/#think)：新增「成稿之后」一段。五个角色只读成稿和语料，修订只许改被点名的段落，终审判 ACCEPT 或 REOPEN，REOPEN 由你决定是否回成文。
- [流程图模块 5](https://zhuchengx.github.io/paper-alchemy-manual/#fl-m5)：S21b 由规划中改为已实现。

### 2026-10-23 20:31 · B2 逐候选审查、B3 全文数字机核

- [它怎么思考](https://zhuchengx.github.io/paper-alchemy-manual/#think)：审查改为两层。每个候选一个 critic，判 VALIDATED / REVISE / DROP；REVISE 最多改写 2 轮；judge 只从 VALIDATED 里选。
- [输出目录详解](https://zhuchengx.github.io/paper-alchemy-manual/#output)「常见疑问」：「数字从哪来」一问补上 S20b 数字机核。正文每个数字都要有来源指针，对不上即 FAIL。
- [流程图模块 3](https://zhuchengx.github.io/paper-alchemy-manual/#fl-m3)、[模块 5](https://zhuchengx.github.io/paper-alchemy-manual/#fl-m5)：B2、B3 对应阶段由规划中改为已实现。

### 2026-10-23 19:47 · LLM 阶段改由 codex 执行，B1 验矿卡

- [这是什么](https://zhuchengx.github.io/paper-alchemy-manual/#what)、[安装与准备](https://zhuchengx.github.io/paper-alchemy-manual/#install)：跑 case 的 LLM 阶段（采购到成文）改由 Codex CLI（gpt-6-astra）经 `pipeline/run_agent.py` 执行，Claude Code 主线程只改 pipeline、编排、核验。安装表新增 Codex CLI。
- [跑第一轮](https://zhuchengx.github.io/paper-alchemy-manual/#first)：第 ② 步改为「渲染提示词 → 交给 codex → 过闸门」，并给出留痕位置 `traces/agent_runs/`。
- [流程图模块 2](https://zhuchengx.github.io/paper-alchemy-manual/#fl-m2)：B1 验矿卡（`scripts/solidity.py`）由规划中改为已实现。

### 2026-10-23 17:04 · 中心问题

- [这是什么](https://zhuchengx.github.io/paper-alchemy-manual/#what)：菜单新增「中心问题」：一句话，外行一分钟能听懂，并说清什么算答上了。审查用它判终稿是否跑题。附 run3 的中心问题作例子。

### 2026-10-23 16:47 · 启发池出 idea 草稿

- [这是什么](https://zhuchengx.github.io/paper-alchemy-manual/#what)「④ 启发池」：新增构思算子出草稿。13 个算子各出一份，另有不超过 13 份算子组合，写明先后顺序和理由。草稿只用来启发，不是正式卡。

### 2026-10-23 16:06 · 启发池（B9）写进正文

- [这是什么](https://zhuchengx.github.io/paper-alchemy-manual/#what)：新增「④ 启发池」卡片，写明算子从哪来、怎么用、为什么做、放在哪。输入侧、镜头、目录树同步加上 `inspire/`。

### 2026-10-23 15:51 · 流程图加启发池

- [流程图模块 1](https://zhuchengx.github.io/paper-alchemy-manual/#fl-m1)：新增规划中阶段 S00a 启发池。

### 2026-10-23 15:43 · 蓝图加 B9

- 蓝图（[这是什么](https://zhuchengx.github.io/paper-alchemy-manual/#what) 末尾链接的 TPL 对照页）新增 B9「输入广度」。说明书正文不变。

### 2026-10-23 15:35 · 首次发布

- 8 章：这是什么、安装与准备、跑第一轮、它怎么思考、你在哪些点介入、输出目录详解、规则与红线、常见问题。章节结构对照 Team Paper Lite 公开说明书。
- [它怎么思考](https://zhuchengx.github.io/paper-alchemy-manual/#think)：两层流程图，由 `pipeline/stages.json` 自动生成。第一层 6 个模块，第二层每个模块折叠展开。
- [你在哪些点介入](https://zhuchengx.github.io/paper-alchemy-manual/#gates)：5 个闸，每个只问一个问题。
