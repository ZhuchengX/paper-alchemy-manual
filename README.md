# 论文炼金 · 使用说明书（网页版）

在线阅读：https://zhuchengx.github.io/paper-alchemy-manual/

这里只放说明书和它链接的调研页，由维护者从私密仓库 `ZhuchengX/paper-alchemy` 自动发布，不要直接改这里的文件。

## 更新记录（新的在上）

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
