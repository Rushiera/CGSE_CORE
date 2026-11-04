# CHANGELOG — CGSE_CORE

> 已完成项唯一真相。未完成项在 `todo.md`。

## v0.26 — 2026-11-05

### packing 结案 + 模型形态侦察 + 探针脚手架（`src/` 零改动）

**性质**：外部建议核查轮 → 转侦察轮。建议主张「困难实例 = 布尔装箱，应强化整数优化」，实测形态完全不符。

- **packing 族零入场**：11 个请求全跑，`count_packing_analysis`（入场成功标记）**零命中**；3 次 `CountPackingStructure.create` 全死在域闸 `high > 1`。放开 `CountPacking` 规模闸（256/1024 → 1024/4096）后 work **38,019,380 逐值不变** → 判死，已回滚。
- **形态画像**（funded x1000 四视图）：n=252~280，0/1 占 48% / 多值 46% / **无界 6%**（`high == null`，`peak=Long.MAX_VALUE`）；多值分层 `[≤4:15, ≤64:37, ≤4096:19, >4096:45]`；混合符号行占多数（69 行中 53 行）。
- **统一根因**：15~17 个无界变量 → `CountIntegerLpLearning.bounded()` 判否 → LP 支持面只剩 **128/439 变量、110/291 行（29%）**、`count_lp_structure: prepared=0; hits=0`；同时被拒的还有 Diophantine（`shape:upper==null`）/ DD / Partition / Boolean（`shape:not_binary`）/ Probing（`variables>128`）。
- **两条硬线索**：① `count_model: skipped; closure_recipes=8193; local_limit=8192`——**1 个单位卡掉整模型**（`GraphCompiler.java:331-335`）；② 冲突学习面为零（LCG `conflicts=0/1/3`、`count_conflict_pool: learned=1; used=0`），「信息交换」没有源头。
- **脚手架**：`.local/scaffold/probe.py`（apply / run / report / restore 四命令）——探针 3 处零语义影响，restore 后 `git status` 干净。

详见 `analysis-model-shape.md`；脚手架用法 `.local/scaffold/README.md`。

## v0.25 — 2026-11-05

### 36 条 `fixtures-complete` 复核：known-limits 的 36 条记录在当前代际全部不复现，只有 pool_mix7 两个排列变体仍 TIMEOUT（`src/` 零改动）

**性质**：纯跑测。`tests/known-limits.json` 里 123 条 UNRESOLVED 记录中前 36 条（`suite=fixtures-complete`，`source_revision: e788d3d3`）从未在当前代际对账，本轮补齐。

**口径**：HEAD `b7c4a81` · `--suite corpus` 按 case 名重跑 · `--work 20000000 --milliseconds 3000`（与 known-limits 记录同预算）· 默认引擎 · `src/` 零改动。

**结果**：36 条记录 = 3 个 selection（fixtures-0/1/2）× 12 条，去重后 **17 个 distinct case 名 + 排列变体，实跑 26 例：24 PASS / 2 UNRESOLVED / 0 failed**。

| 面 | 旧记录（e788d3d3） | 当前代际（b7c4a81） |
|:--|:--|:--|
| `pool_mix7` | TIMEOUT work=4,484,064 | **PASS** work=5,685,892 |
| `recycle_mix3` | TIMEOUT work=15,205,613 | **PASS** work=4,075,294 |
| `interval_n36_*`（3 名） | SEARCH_LIMIT ≈20.16M | 全部 PASS |
| `multi_n32_d{8,9,12}_w1000_*`（6 名） | SEARCH_LIMIT ≈20.4–20.7M | 全部 PASS |
| `multiway_n20_c{3,4}_s*`（6 名） | SEARCH_LIMIT ≈20.2–20.3M | 全部 PASS |
| `pool_mix7__perm0` / `__perm1` | 无直接记录 | **UNRESOLVED（TIMEOUT）** work=5,262,566 / 4,096,164 |

**判读（可复用）**：**同一 fixture 的结论依赖排列顺序**——`pool_mix7` 主名与 `perm2` 通过，`perm0` / `perm1` 仍超时。known-limits 的 `selection` 字段（fixtures-0/1/2 三次完整排列）不是可忽略的元数据；按 case 名重跑只能复核「该 case 是否还卡」，不能复核「该 selection 是否还卡」。

**当前剩余 UNRESOLVED 全清单（6 例 · 本日逐例复核）**：

| # | 面 | case 名 | id 前缀 | 20M 下 work |
|:--:|:--|:--|:--|--:|
| 1 | count-model | `generated/sparse-equality-n144-s2` | `1204d21049c4` | 19,995,512 |
| 2 | count-model | `generated/sparse-equality-n257-s1` | `867653b0b246` | 19,997,420 |
| 3 | count-model | `generated/sparse-equality-n257-s2` | `3f6275678665` | 19,997,992 |
| 4 | count-model | `generated/sparse-equality-n144-s1` | `898e00c6bbdb` | 19,995,399 |
| 5 | graph | `pool_mix7__perm0` | `51bbc7ff4220` | TIMEOUT 5,262,566 |
| 6 | graph | `pool_mix7__perm1` | `0957cea133ae` | TIMEOUT 4,096,164 |

count-model 4 例归因见 `m2-unresolved-attribution.md`（分支顺序否证 · 3 例 100M 仍无 witness · `898e00c6` 在 33.5M 即解出，属纯深度不足）。graph 2 例是同一 fixture 的排列变体，与 §判读同源。另有 4 例 UNSUPPORTED（`main-counts conversion requires finite upper bounds`）不属本表——它们不是预算未决，是转换面不支持。

**诚实边界**：未重放 fixtures-0/1/2 的完整排列（只重放同名 case 子集）；`--suite all` 仍未跑（@todo M5）；2 例 UNRESOLVED 按退出码语义记为预算未决，不是 UNSAT、也不是 PASS；corpus 侧只覆盖本轮 26 例子集，其余 graph case 未跑。

**复跑命令**（分批，单批 < 120 s）：

```text
python tests\run.py --java-home <repo>\runtime\jdk-17.0.2 --suite corpus --case "interval_n36_d2_s19_slack10|interval_n36_d2_s19_slack1|interval_n36_d2_s7_slack1|multi_n32_d12_w1000_s19_shift0|multi_n32_d12_w1000_s7_shift0|multi_n32_d8_w1000_s19_shift0|multi_n32_d8_w1000_s7_shift0|multi_n32_d9_w1000_s19_shift0|multi_n32_d9_w1000_s7_shift0" --work 20000000 --milliseconds 3000 --keep-going --output .local\fxc-a
python tests\run.py --java-home <repo>\runtime\jdk-17.0.2 --suite corpus --case "multiway_n20|pool_mix7|recycle_mix3" --work 20000000 --milliseconds 3000 --keep-going --output .local\fxc-b
python tests\run.py --java-home <repo>\runtime\jdk-17.0.2 --suite models --case "generated/sparse-equality-n144-s1|generated/sparse-equality-n144-s2|generated/sparse-equality-n257-s1|generated/sparse-equality-n257-s2" --work 20000000 --milliseconds 0 --model-engine main-counts --keep-going --output .local\m2-recheck
```

产物：`.local/fxc-a/graph-0.log`（12 例全 PASS）· `.local/fxc-b/graph-0.log`（14 例：12 PASS / 2 UNRESOLVED）。

## v0.24 — 2026-11-04

### δ（段内 summary 去重）判定：直测 2.66% 总 work，20,230 次 0 值失配

**性质**：承 v0.23「段内 `SummaryComputation` 占 65%」，判「p3→p7 的 body 重算可否线性推导」。机制成立（`SequenceSummary.repeat` 闭式 + `known` 复用有上游先例 `PlanVerification`），但原「上界 7.6%」是 p7 全量，δ 只吃掉其中一遍 body 重算（详见 `analysis-w5.md`）。

- **真实负载直测**：p7 分支用独立 budget（不占被测 work）旁路跑四份计算（unit / total / known / tail），20,230 次事件——Σunit 1,051,669 · Σtotal 3,547,282（9.33% 总 work）· **省 1,011,209 = 2.66% 总 work = p7 的 28.5%**；请求 work 38,019,380 与 w4 逐值一致 → 零扰动
- **值保真**：known 复用与纯代数 `repeat().then()` 双路各 **0 / 20,230 失配**；Σ尾波 == Σ一遍 body（结构同一，交叉印证）
- **再进一步已否证**：尾波按比例闭式在净产出配方上给错值——`SequenceSummary.repeat:90/92` 的 `delta ≥ 0` 分支不放大 need / peak
- **口径修正**：采样对短相位系统低估 **1.23×**（p7 采样 7.56% ↔ 直测 9.33%；每次平均 175 work ≪ 1,024 work 采样阈值）→ v0.23 的 phase 分布不能当绝对量，β 12.58% / ε 14.6% 需按直测口径复核
- 分档：≤4 向量 73.4% 事件只值 0.42%；>32 向量 50 次事件值 1.27%（最贵 100 次占省量 49.2%）

## v0.23 — 2026-11-04

### 段级脚手架：观测面从 region_counts 抬到 region 求解段，可省上界实测 12.58%（`src/` 零改动）

**性质**：承 v0.22 的 α/β 分叉，判 β 的收益必须把成本面从「计数」抬到「整个 region 求解」。抬的过程踩了三个坑，每个坑给出一条可复用判据（详见 `analysis-w4.md` §八）。

- **坑一 · 累计量差值失效**：多视图在同一 budget 上公平轮转，`threadWork` 差值算出段合计 = 总 work 的 **682%**（同时 `RegionSelection.step` 采样只有 30.8%，两者不可能同时成立）。判据：累计量的推进方不唯一 → 差值无意义
- **坑二 · 活跃段不止一个**：单槽 `ThreadLocal` 让 A 的 `endRegion` 清掉 B 的段，段合计塌到 **0.02%**。修法：按 `solveId` 分槽
- **坑三 · 未收口段无界累积**：只靠 begin/end 配对时，未 `close()` 的 solve 让段一直开着，段合计涨到 **79.4%**。修法：活跃性绑到 `RegionSelection.step` 的执行窗口（`markActive` in/out）
- **结果**：段成本 **29.67%** 总 work（↔ `RegionSelection.step` 30.8%，吻合）；4765 段 / 1216 region / 120 solve；跨轮相邻段 3540 对，命中 **72.9%**（对）/ **76.8%**（成本加权）；**可省上界 12.58%** 总 work
- **第三重印证**：段内 p10+p13（`RegionCounts`）= 4.05% 段成本 = 1.2% 总 work ↔ 第一轮探针 1.00% ↔ `RegionCounts.step` 采样 0.99%
- **顺带发现**：段内 `SummaryComputation` 跑两次（p3 39.9% + p7 25.5% = 段成本 65%）。这是**段内**重复，不是跨轮重复——与 A / B 都不同族，属新候选面
- **β 落地前只剩一项量**：预算敏感性（同输入不同余量是否得不同结果）

详见 `analysis-w4.md` §八（方法清单五条 + 结果表 + 验收判据）。

## v0.22 — 2026-11-04

### B' 前置插桩（w4）完成：region 级跨轮输入命中率 78%，但前提「region_counts 占 46.2% work」被推翻（`src/` 零改动）

**性质**：插桩探针（只读，`-Dcgse.rtrace` 开关），work 逐值不变（38,019,380）。数据落 `.local/w4/out/rtrace-funded3.tsv`（574 事件 / 188 region / 107 solve）。

- **命中率达标**：同一 region 跨轮相邻出现 386 对，input 摘要相同 **301 对 = 78.0%**；按 work 加权 **65.8%**；**图内容变 0 对**；同轮内重复 0 对。分成本看，最贵的两个 region（各占 29% / 28.7% work）命中率 50% / 72%——命中不是「便宜且高频」撑起来的
- **前提修正（三重互证）**：`region_counts` 自身 work = **1.0%**（探针 381,548 / wstack 采样 `RegionCounts.step` 0.994% / note 自报 0.91%）。§10.2 的「46.2%」是 **note 区间归因**——同法复现 54.4%，等于把两条 region_counts note 之间整个 `RegionSelection` 的求解都记在它头上
- **可复用面重估**：计数级（原 B' 定义）收益上限 **0.57%**（命中对 work 217,388 / 总 38,019,380）；同一命中率若扩到 region 求解级（`RegionSelection.step` = 30.8% 采样占比）上界 **≈20%**
- **岔口呈报待裁**：α 计数级复用（0.57%，低风险）/ β region 求解结果复用（≈20%，需先补「按 region 成本加权的命中率」与「预算敏感性」）/ γ 同输入 region 单次求解（跨视图共享，与 A 同源的调度耦合风险）——推荐 β 并先补两项量，详见 `analysis-w4.md` §五

详见 `analysis-w4.md`（插桩设计 / 四种口径对照 / 各阶段真实占比 / 诚实边界）。

## v0.21 — 2026-11-04

### B / D 双验证：换算子与入场筛都判死，指向「跨轮增量复用」（`src/` 零改动）

**性质**：Rushiera 裁决「B 和 D 都做」。两条均执行，均在离线数据上被否。

- **D（准入层前置）判死**：15 请求 / 161 线的首轮特征零分离度——成功线首轮密度 p50 0.516 vs 失败线 0.630；按首轮密度排名，成功线 **p50 = 第 10 名**，仅 1/15 个请求排第 1；每请求留 top-1 只保住 1 个请求。首轮是侦察轮，成功/失败差异只在后续推进显现
- **B（提高每步信息量）转向**：先记一条方法失败——用 `checkhist` 按轮切 check 数不可行（note 的 `work@` 是采样点，同轮工作分散在多个间隔，实测出现 spent 2,877 / checks 1,466,346），脚本 `w3_b.py` 标作废
  - 可靠口径①：**停滞轮 spend 368.6K vs 下降轮 421.6K（比 0.87）**；线内 CV `V2.0`=0.19 / `V6.0`=0.21 —— 成本与产出无关，由每轮固定动作决定
  - 可靠口径②：轮间 stage 出现率 `region_counts` **81.2%**、`support_neighborhood` 48.4%、`region` 23.4%；与 §10.2 的 work 占比（region_counts 46.2% / stock_view 21.4% / region 12.0%）对齐
  - 结论：操作对象不是单步算子（§11.5 封顶 13.1%），而是**每轮重建**（region_counts 重扫 1146 个 region + compile 重编 + region 重跑）
- **下一步（B'）**：跨轮增量复用——需先插桩量「同一 region 连续两轮的输入是否真的变了」（region id + 输入摘要 + 命中率），成本约一轮跑测；架构级改动，健全性风险在失效判定

详见 `analysis-w3.md` §十（含 D 的阈值/top-k 扫描与 B 的两种口径表）。

## v0.20 — 2026-11-04

### C 案扩样 + A 实测：判据成立但解法失败（`src/` 已回滚至改动前）

**性质**：Rushiera 裁决「C → A」。C（扩样本）完成，A（视图淘汰）实施两版后判死回滚。

- **C 完成**：w2 变体跑 7 个数据集面 → **15 个有轮次记录的请求 / 161 线 / 183.2M work**（round spent 90.5M = 49.4%）
  - 停滞占比：P 52.8% · Q 72.4% · **R 40.9%**（round 面）——R 档仍 >30%，分叉方向维持
  - 成功线停滞轮的 `missing_keys` = `[1×26, 2, 5, 22×6, 27, 58]`——**缺口绝对值门失效**（22/27/58 的成功线都被 M=2 误杀）
  - 组合闸重扫：**R k=3 M=2 → 砍 47 线 / 省 17.66M（19.5%）/ 误杀 0 / 误砍请求 0**（参数不再凑）
  - 成功线只占各请求 round spent 的 11.3%–42.2%（中位 ≈23%）→ 理论上限 ≈77%，闸只兑现 19.5%，差额在「触发前已付的 3 轮」
- **A 两版实测（20 请求）**：
  - **A1**（`retired` → `done` + `live--`）：全 FEASIBLE、误杀 0，但总 work **+7.7%**（−25% … **+74%**）
  - **A2**（只跳过公平轮转、不碰 live/done）：总 work **+42.0%**，`miracle-quantity x1000` → **ERROR / SEARCH_LIMIT**
- **根因（结构耦合）**：`Portfolio.step()` 的 `viewsComplete`（`:150-152`）同时是 `addQuantitativeViews()` 与 `pause()` 的前置（`:78-91`）。A1 提前触发交接 → 部分请求被拉长；A2 让 retired 视图永不 done → `live` 永不为 0 → 量化视图永不入场、`pause()` 不发生 → 烧穿预算。「放弃一条视图」与「让视图族完成」是同一件事。
- **回滚**：两处改动全撤（`GraphStockViewWork` 的 `retired/stagnantRounds/bestDensity` · `GraphStockViewPortfolio` 的 `fairPick/selectable`），重编后 `funded-3` 复核 **work = 38,019,380**（与基线逐值一致）；全库 `retireIfStalled` / `fairPick` 零命中
- **结论修正**：「无效推进占比 >30%」成立，但「提前放弃」**未证实**——判据成立 ≠ 解法成立；模拟未计入调度耦合

详见 `analysis-w3.md` §九（含 A1/A2 逐请求对照与复跑命令）。

## v0.19 — 2026-11-04

### 有效推进归因（w3 件 1/件 2）：无效推进占轮次成本 41.7%–60.4%，分叉指向「提前放弃」（`src/` 零改动）

**性质**：纯分析轮——把「一步是否有效」量到轮次级。全部在 `.local/w3/`。

- **样本**：`.local/w2/spent.jsonl` 单请求 · 65 条 round · **11 条线**（variant × round 递增链；round 回退 = 不同视图实例，V0 拆 2 条）· round spent 14,999,071（39.5% 总 work）
- **件 1 轮次序列**：只有 **V2.0 一条线有正推进速度**（36→0 / 16 轮 = +2.25）；其余 10 条末值 ≥ 首值或 UNKNOWN 收尾，合计 8.66M（57.8% round spent）。V2.0 前 6 轮完成 34/36 缺口下降，第 9–15 轮连续 7 轮停在 1 个缺口
- **口径修正**：`missing_keys` 随 recipes 扩张而暴涨（V9 `1→27→66→39→33→23`）——P（与上轮比）/Q（best-so-far 绝对值）会把「视野扩大后的暴露」误判为停滞；新增 **R 档（best-so-far 缺口密度 missing/recipes）**，R 下 V9 停滞只剩 0.01M
- **件 2 停滞占比**：P 32 轮 / 6.26M（41.7% round · 16.5% total）· Q 43 轮 / 9.06M（60.4% · 23.8%）· **R 31 轮 / 7.14M（47.6% · 18.8%）** · P∪UNKNOWN 41 轮 / 7.31M（48.7% · 19.2%）
- **分叉裁决**：分母取「轮次可归因面」（轮外 60.5% 由轮次驱动，用总 work 做分母会把轮外成本算成非停滞）→ 41.7% / 47.6% **>30% → 提前放弃**；对分母维不稳健（总 work 分母下 16.5%–18.8% → 反转）
- **机制修正**：`stalledRounds` **不是放弃判据**——只进 `estimatedRepairCost()`（`GraphStockViewWork.java:581`），消费点 `Portfolio:177/324` 的视图选择排序；`eligible()` 无停滞条件 ⇒ 停滞视图永不淘汰。候选 D「阈值收紧」不成立，「提前放弃」= 新增淘汰机制
- **可兑现收益**：简单连续停滞闸（k=2/3/5）**全部误杀唯一成功线 V2.0**；加缺口门后（连续 3 轮停滞 + `UNKNOWN 或 missing≥2`）省 **2.75M = 18.4% round spent / 7.2% 总 work，误杀 0**

详见 `analysis-w3.md`（含复跑命令与诚实边界）；岔口呈报见其 §六（A 淘汰 / B 提高每步信息量 / C 先扩样本），未拍板。

## v0.18 — 2026-11-04

### work 归因脚手架（w2）：70% 预算是推进步数税（`src/` 零改动）

**性质**：脚手架轮——不猜机制，先造能把 work 量到代码行的尺子。全部在 `.local/w2/`。

- **脚手架三件**：`WorkStack.java`（work 加权栈采样——按 tick 不按时间）+ `build_w2.py`（单点插桩 `PlanningBudget.traceCharge` → 全量 javac）+ `analyze_w2.py`（调用树 / 叶子路径 / 重入账）
- **自证**：采样覆盖 99.9996%；Origin×Stage 与 `analysis-strengthen.md` §四 逐行吻合（47.75 / 15.37 / 11.48 …）；work 总数 38,019,380 与基线逐字节相同
- **方法级**：前 5 名占 70.5%（`GraphPlanningWork.step` 19.64% · `SummaryComputation.advance` 13.57% · `GraphSolve.step` 13.44% · `RegionSelection.advance` 12.97% · `GraphStockViewWork.check` 10.89%）
- **行级**：前 6 行 65.1%，全部是 `budget.check()` / `compilationCheck()`（固定 1 work）；推算约 2,673 万次检查点
- **A/B**：check 不计费 → 5 分钟未完成（基线 17.35 s）⇒ 步数税是限流器，不是冗余
- **结论**：work 预算 = 步数税 70% + 真计算 30%——七次零收益的根因是改动全落在「真计算」里的 0.7%–6.4% 面
- **步数归因（同日续，方向 A）**：精确计数 `check()` 全量调用——**32,297,205 次 = 84.9% 的 work**；note 时间轴显示 65 条 round 记录里只有 1 条产出解，witness 出现在 work 的 99.97% 处
- **C1 失败模式分层**（`spent` 精确口径）：round 自身 spent 合计 14.99M（39.5%）；`MISSING_SEED` 36 条吃 86%，平均 35.8 万/轮；成功者 `variant=2` 自己也要 16 轮（前 15 轮全 MISSING_SEED），每轮 85% 是 check
- **C2 结构实测**：轮转是**时间片制**（每视图 32,768 work 一拍，`GraphStockViewPortfolio.java:150`），不是串行策略扫描——初版「策略顺序可省 97%」的推测作废；每轮 note 此前不落 `spent`（本轮变体补上）

详见 `analysis-attribution.md` §十 / §十一。

## v0.17 — 2026-11-04

### 强化方向取证：真实语料 19/19 PASS · 七次零收益归因 · work 构成图（`src/` 零改动）

**性质**：取证轮——Rushiera 换方向（「给求解器进一步强化求解能力」），先证伪再找靶点。全部变体在 `.local/`。

**四条实测**：

| # | 实验 | 结果 |
|:--:|:--|:--|
| D4 | 真实语料基线（`tests/datasets/regressions.py`，68,187 recipes / 19 请求） | **19/19 `VERIFIED_FEASIBLE`**，0 UNRESOLVED；funded x1000 = 38,019,380 / 40M（**95.0%**），6 例 >45% |
| D1 | 整数 LP 学习通道 | 观测：`viewAdmitted=5 / solve=5 / certifiedCut=0 / integerEnabled=0`——被「须先产割才入场」锁死；接通后 work **+0.01~0.05%**，状态全不变 → 判死 |
| D3 | `IntegerCountSearch:314` 窗口 4096→32768 | 274 例状态 **0 变化**，work **+2.44%**（单例最差 +71.52%）→ 判死 |
| — | work 构成图（源码内置 `PlanningCostTrace`） | `STOCK_VIEW.SEARCH` **47.8% / 59.7%** · `COORDINATOR.OTHER` **15.6% / 17.6%** · 分支层各策略合计 **6.4%** · `CountViewSearch` 自身 **0.7%** |

**关键修正**：此前以为「LP 是布尔专用」——实为 `CountLcg.integerRelaxation()` 已有 8192/32768 的整数通道；改布尔闸没有对象。

**根因判读**：七次「加机制」全部零收益，是因为改动面都落在 0.7%–6.4% 的边缘，而 64%–77% 的开销在 `STOCK_VIEW.SEARCH`（21 轮 / 10 轮迭代才出 witness）与 `COORDINATOR.OTHER`（未拆过）。

**产物**：`.local/d1`（死锁观测）、`.local/d1b`（接通 A/B）、`.local/d3`（窗口 A/B）、`.local/w1`（构成图）、`.local/ab/compare_d3.py`；Project 侧 `analysis-strengthen.md`。

**诚实边界**：D1 A/B 只覆盖 10 个真实请求；274 例未复跑 D1；`COORDINATOR.OTHER` 语义未拆；`--suite all` 未跑。

→ 详见 `analysis-strengthen.md`

## v0.16 — 2026-11-04

### G3 岔口拍板：c3 显式 note 收口落地 · c1 判死（改码一行）

**性质**：拍板 + 执行轮——依据是本轮新增的**放宽实测**，不是推论。

**判死 c1 的决定性证据**：把 `shape:faces_not_equal` 闸整个拿掉（`.local/g3/rel-src/` 副本改 + 全量重编译）后：
- models 274 例（`main-counts` · work 20M · 9 片）状态 **0 例变化**（PASS 264 / UNRESOLVED 4 / UNSUPPORTED 6 逐例同；两例目标 work 微差 1 / 262，状态不变）；
- census 探针 274 例 Diophantine SOLVED **26→26**；77 例只是把拒绝理由从 `shape:faces_not_equal` 换成 `shape:faceChoices<2>` / `faces_unresolved`——**不出解**；
- 两例目标（`1204d21049c4` / `898e00c6bbdb`）在 10× 预算（200M）下仍 UNRESOLVED（work=199,995,657）；
- 离线 RREF 探针：两例 = 72 条 ±成对行合成等式（秩 72）+ 72 条剩余不等式 / 144 变量，自由变量取下界回代即出盒——**要 LP 松弛 + 分支，不是面枚举**。

**c3 落地（唯一改动）**：`CountDiophantine.prepareFaces` 拒绝时补诊断行
`count_diophantine: multi_shape_conflict; row=<首个冲突行>/<行数>; equation_vars=<…>; row_vars=<…>`——拒绝原因是形状冲突、不是容量；实跑样例 `row=3/432; equation_vars=9; row_vars=9`。

**前后对照（c3 是纯诊断增量）**：models 274 例终态 **0 变化** · 逐例 work **0 变化**；core 385 ran / 385 PASS / 0 FAIL（唯一 385）；`gradle jar --offline` BUILD SUCCESSFUL（7s）；`check_abi` **PASS 1502**。

**产物**：`.local/g3/`（探针 `relax_probe.py` / `relax_compare.py` / `compare2.py` / `compare_work.py` / `solve_probe.py` / `run_models_shard2.py` / `summary_round.py` + `out/g3-decision.md` 等）· Project 侧 `m4-g3-decision.md`。

**诚实边界**：放宽实验改的是闸本身（c1 最激进形态），完整 c1（分组 + 组间松弛 + 三态剪枝）未做原型——「完整 c1 也零收益」是推论；反证门槛低（放开只要多解一例即成立），实测一例都没有。`--suite all` 未跑。

→ 详见 `m4-g3-decision.md`

## v0.15 — 2026-11-04

### G3 岔口呈报：c1 组合设计 + 健全性论证（改码零行）

**性质**：取证 + 呈报轮——`src/` **零改动**；全部产物落 `.local/` 与 Project 侧档案。

**新增探针**：`G3ComponentProbe.java`（`.local/g2/probe-src/`，274 例全量）——把每个 shape 组连成超边，对组内变量做并查集，测**分组后的变量连通分量**。

**三条结论**：

| # | 结论 | 证据 |
|:--|:--|:--|
| 1 | **c1 的分解形态在语料上不成立** | 分组后**变量连通分量数 27/27 = 1**（残余耦合度 100%）；分量规模：组数 16–297 · 变量数 27–143 · 冲突边 164–5184。c2 与 c1 败于同一处 |
| 2 | **c1 的净收益面是 2 例，不是 27 例** | census 的 27 例来自 v0.4 M0 旧快照；当前字节 models 复跑（274 例 · `main-counts` · 20M）后 G3∩U 的 27 例有 25 例已 PASS、G3b∩U 的 27 例全 PASS；仍 UNRESOLVED 且命中 G3 的只有 `1204d21049c4`（work 19,995,512）· `898e00c6bbdb`（work 19,995,399），两例 work 触顶 20M |
| 3 | **G3 与 G3b 不是「连动面 / 共因」**（上轮口径勘误） | 实测两闸 ∩UNRESOLVED **交集 = 0**（各 27 例，合计 54 例）；互斥是结构性的——被 `faces_not_equal` 拒的例到不了 faces 尝试 |

**c1 设计 + 健全性论证**（`.local/g2/out/g3-c1-design.md`）：分解形态否掉后剩余「单分量内组间组合搜索」形态，20M work 下覆盖性无法论证；健全性义务四条（三态返回 / 剪枝只认完整覆盖 / 不可行证书独立 / 诊断面齐平）+ 回归不变式一条已写成开工前置。

**回归（全绿）**：core 套件 **389 ran / 389 PASS / 0 FAIL**（唯一测试名 385）· models 274 例 **PASS 264 / UNRESOLVED 4 / UNSUPPORTED 6** · `check_abi` **PASS 1502** · census 复跑 274 例（G1 0/0/0 · G2 22∩2 · G3 84∩27 · G3b 44∩27）。

**口径勘误已回写**：`m4-g2-g3.md` §五「三案取舍」加修正前置说明（保留当轮口径作决策轨迹）· `m4-g3-verdict.md`（呈报全文，入 project.md 索引）。

**判例沉淀**：`L2/Worker/Coder/history/2026-11-04_CGSE_G3岔口三判例_收益面代际漂移与分解判据.md`（收益面代际漂移 / 分解判据强度链 / 关系词需集合证据 + 扩面类改造的健全性义务通用形态）。

**诚实边界**：census 终态源仍是 v0.4 M0 旧快照（本轮用当前字节 models 复跑替代其作终态判据，触发面口径未变）；`--suite all` 未跑（core/models/ABI 已跑，view/oracles/fixtures/probes/corpus/benchmarks 未跑）；`IntegerCountBranch:520` 第二消费点入口未覆盖；c1 组合搜索只写到形态与义务层，未做原型、未量化剪枝强度——「无法论证覆盖性」是判断不是实测结论。

→ 详见 `m4-g3-verdict.md`

## v0.14 — 2026-11-04

### M4 三闸口径落地：G2 b1 改码 + 前后对照 + G3 岔口呈报

**改码（唯一一处）**：`CountDecisionDiagram.java:75-76` 宽度闸 `w > 8` → `w > 16`，轴串同步 `shape:domain_width>8_with_variables>12` → `>16_with_variables>12`（census 驱动 `.local/m4/m4_gate_census.py:38` 同改，否则认不出该闸）。上游 `size:width>64` 未动。

**前后对照（结论：收益 0）**：
- census 274 例全量重跑：G2 触发 30 → 22，∩UNRESOLVED 2 → 2；行级变化 30 例**只改 `cdd` 列**，`status` 列**零变化**；G1/G3/G3b 逐例同。
- models 套件 `main-counts` 20M：pre8（阈值 8 字节）vs post16（阈值 16 字节）逐例对照 —— 覆盖 268 例（另 6 例 UNSUPPORTED），**status 变化 0 例**，work 变化 2 例（+0.9%）。census 指名两例 `c28b807b80f5` / `fec30a3add39` 在两版下均 PASS（UNRESOLVED 来自 v0.4 旧快照代际，非宽度闸阻塞）。
- 结论：b1 把 8 例从「宽度闸拒」变为「放行由后续引擎处置」，其中 **0 例**因此 UNRESOLVED→PASS。

**回归（全绿）**：`gradle jar --offline` BUILD SUCCESSFUL in 20s · **core 套件全量 389 ran / 389 PASS / 0 FAIL**（唯一名 385 + `GraphBenchmark` 属 benchmarks 套件）· models 274 例 PASS 264 / UNRESOLVED 4 / UNSUPPORTED 6 · `check_abi` **PASS 1502** · census 复跑 274 例（G1 0/0/0 · G2 22∩2 · G3 84∩27 · G3b 44∩27）。

**分片口径修正**：宿主单次命令 120 s 上限 → core 改 `--case` 正则分块跑（`.local/g2/plan_groups.py` + `run_chunk.py`，24 块），取代原先「每片 120 s 截断只覆盖 169」的计数口径。

**G3 岔口到实现层后呈报**（未开工）：新探针 `G3ShapeProbe` + `G3ShapeOverlapProbe`（`.local/g2/probe-src/`，全量 274 例）实测 G3 ∩UNRESOLVED 27 例 —— shape 分组数中位 117（max 369）、多变量 shape 变量子集数中位 28、重叠对 82–2592、**变量互斥 0/27** → **c2 判否**（前提在语料上不成立）；**c3 零改码**（M1c 已给 `faces_not_equal` 具名 note + `refusal()` 齐平）；**c1 待裁**（语义新设计 + 健全性要证，且分组后仍跨组耦合）。

**诚实边界**：census 终态源为 v0.4 旧快照（M1c 后未重跑），改造前后判据改用 pre8 同字节逐例对照（不依赖旧快照）；`IntegerCountBranch:937/:2128`（DD 第二消费点）未覆盖；**未跑 `--suite all`**（view / oracles / fixtures / probes / corpus 未跑）。

→ 详见 `m4-g2-g3.md`

## v0.13 — 2026-11-04

### M4-0 三闸触发面 census（拍板 A 案——先量后改）

**产出**：`M4GateProbe.java`（直方图探针，`.local/m4/probe-src/`）+ `m4_gate_census.py`（驱动 + 聚合，`.local/m4/`）· `--shard 0/9…8/9` 分片 + `--summarize` 聚合。**census 文档 = `workspace:tools/CGSE_CORE/.local/m4/out/m4-gate-census.md`**（已入 project.md §文档索引，🔴 M4 拍板轮必读）。

**方法**：QuickSolve 基形入口复刻（bound 行扩充 :104-107 + CountBounds + CountReduction）→ 引擎构造面同 QuickSolve:176（Diophantine）/:327（DD）→ CRK 链经 CountAffineLattice 步进至 beginKernel；每例共享预算 20M work，步进帽 1M；轴取 `budget.diagnostics()` 的 `admission_refused` 项；终态联 `.local/audit/baseline-models-det.json`（v0.4 M0 快照）。

**计数（274 例全量 · PASS 200 / UNRESOLVED 74）**：

| 闸 | 轴 | 触发 | ∩UNRESOLVED |
|:--|:--|--:|--:|
| G1（RangedKernel 构造/行面/域面三闸） | `shape:variables!=2` 等 | **0** | 0 |
| G2（DD 宽度） | `shape:domain_width>8_with_variables>12` | 30 | **2**（c28b807b80f5 · fec30a3add39） |
| G3（Diophantine 形状） | `shape:faces_not_equal` | 84 | **27** |
| G3b（连动） | `faces_unresolved`（16 次封顶） | 44 | **27** |

**CRK 链**：274 例中 238 例未达 kernel；达 kernel 的 36 例 d ∈ [4,42]，**0 例 d=2**（`CountKernelSearch:123` `d==2` 分支静态结论 + 动态 0 触发双重实锤）。G1 三闸 = src 面不可达死闸。

**读法**：G1 改造收益 = 0（死闸，唯一候选 a2 位长放宽亦无触发面）；G2 候选面上界 2 例；G3 为 M4 真实面（27/74 UNRESOLVED 过其闸，且 27 例 UNRESOLVED∩faces_unresolved 与之完全同集——「多行不同系数形状」是 Diophantine faces 路径对这批案例的**共因拒绝点**）。触发∩UNRESOLVED 是候选上界，放宽后是否真解出须改造后跑测对照（P1 同族判据）。

**回归**：纯探针轮——src/ 零改动（jar 未重编、core 套件与 ABI 不受影响）；复跑命令见 census 文档。

**诚实边界**：终态源为 v0.4 M0 快照（2026-11-02），M1c 后 models 快照未重跑——UNRESOLVED 集可能有代际漂移；入口口径 = QuickSolve 基形，`IntegerCountBranch:937/:2128`（DD 第二消费点）与 sourceFace/saturate 变体入口未覆盖；budget 环形缓冲 32 条上限。

## v0.12 — 2026-11-04

### M1c 静默拒绝点 note+refusal 齐平 + trial 契约接线（P1，拍板 B 案）

**M1c 清点 24 处**（口径「`complete = true` 既无 note 也无 refusal」，P4 八轴对齐）：实现类 15 处补 `admission_refused` note + `refusal()` 第三态（`CountDecisionDiagram` 3 · `CountDiophantine` 7 含存量 `faces_not_equal` 补 refusal 齐平 · `CountDomainSearch` 2 · `CountSeparator` 1 混合闸拆轴 · `CountMeetInMiddle` 1 · `CountBoolean` 1）；非实现类 9 处补 note（`CountSoftSearch` 2 · `CountRangedKernel` 1 · `CountKernelSearch` 3 含 6 轴构造闸拆轴 · `CountComponents` 2 · `CountChainSchedule` 1）。「算法放弃」3 处并入清点（`restore_out_of_domain` / `restore_row_violation` / `components_unresolved`，裸条件同 `iterations>8192` 先例）。→ `m1c-silence.md`

**trial 契约接线**（`analysis-solver.md` §三 P1）：`CountDomainSearch.trial(boolean)` + `faceInfeasible()`（`:157` 拆原始判定与导出赋值）· `CountSoftSearch:144` `.trial(best != null)` + `:149` 改读面判定 · `CountQuickSolve:319/:322` 同改。TrialProbe 三用例实锤：`bounded_face` 改前 `infeasible=true / trial_field=false / learned_exported=1` → 改后 `false / true / 0`（类注释两条承诺接线）；`bounded_unmarked` 显证 opt-in。P2 删字段案与 `CountReducedCost:121` CountProof 门不做（拍板 B 案单选）。

**回归**：`gradle jar` BUILD SUCCESSFUL · core 套件 **169 PASS · 0 FAIL**（0/4…3/4 分片，对照 M1 基线 139）· `check_abi` **PASS 1502** · AuditGate 274 例计数逐项同改前（Domain 5 / Obbt 52 / Soft 8 / Lcg 0 / Any 81，判定零变化）· M1cProbe 6/6 新 note+refusal 触发实锤。

**未竟**：`CountNeighborhood:346-347` 试算面未接线（消费面未审计，待裁）；非 `complete=true` 族静默出口（工厂 `return null` / `CountKernelSearch:228` Stop|PrecisionLimit / `CountPackingLpSearch:217` state<0）不属 `admission_refused` 通道，留档待裁。

## v0.11 — 2026-11-04

### 求解器原因收口：证书门 13/13 补核 + trial 死字段实锤 + P4 UNKNOWN 定性

**证书门补核 8 类**（13/13 全表填满）：✅ 6 严（`CountCdcl` level-0 空子句 + 理由自检 · `CountComponents` 链式继承 · `CountCoreSoft` shrink 逐证 · `CountHittingSet` · `CountProbing` · `CountNetwork`）· 🟡 2（`CountDecisionDiagram` 的 `Diagram` 证明只入 journal 无 verify 复核 + 3 处静默准入；`CountReducedCost:121` 条件结论无证明复核——唯一消费方 `CountSoftSearch:121-136` 只证 incumbent 最优、现况安全）。辅助核：`CountBoolean` `pinned` 守卫 / `CountMeetInMiddle` `trials` 总守卫均与注释一致。→ `analysis-solver.md` §一「补核」

**trial 死字段动态实锤**（探针 `.local/ab/probe-src/org/cgse/core/TrialProbe.java`，`src/` 零改动）：带界行的面判 infeasible **直接外传**（unrestricted 有解 (1,1) 仍报 true）+ 学习子句漏 1 条入池——类注释「不外传负结论 / 学习子句」两承诺均未接线，**API 假契约**，消费点安全全靠调用侧巧合守卫。→ `analysis-solver.md` §三「动态实锤」

**P4 UNKNOWN 人工定性**：现行口径 140/342 归八轴覆盖 100%（FINISH 82 / ADMIT_STATIC 25 / DOMAIN 14 / ADMIT_MIXED 7 / ALLOWANCE 6 / RESERVE 5 / PRECISION 1；旧记 118 属旧版脚本口径）；新发现「算法放弃」子族 3 处无 note（`CountKernelSearch:211/:223` · `CountComponents:181` · `CountDiophantine:313`）——喂 M1c 清点。工具：`list_unknown.py` + 产出 `out/unknown-list.txt` / `out/unknown-classes.md`。

## v0.10 — 2026-11-03

### M2 未决归因：4 例 UNRESOLVED 的难度分解（插桩跑测 · 源码零改动）

> `.local/ab/attr` 插桩变体；仓库 `src/` 零改动（`AttrProbe` 在 `src/` 命中 0）。详见 `m2-unresolved-attribution.md`。

**口径**：4 例 UNRESOLVED（`1204d21049c4` / `867653b0b246` / `3f6275678665` / `898e00c6bbdb`）· `main-counts` · `milliseconds=0` · `work=20M`，另跑 `work=100M` 对照；覆盖 28 例 × 2 预算 + 4 例专门归因（274 例未全量复跑）。

**插桩等价性**：`attr` vs `base` = 7 改 1 增（1649 类）；4 例状态与 work **逐例相同**；ABI `PASS 1502`。

**分支结构**：4 例全程 `branches=1`、`wave_sum=dispatch`、`peak_width=1` → 只有一个分支对象，`branchPoint()`（`:1403-1457`）一次未执行——**「分支顺序」不是活变量**。

**调度面**：LCG 单实例 3 建、1,983–2,301 续；`restore=0`；scout 0.1%；窗口 work 覆盖率 94.0%–96.9%。**未找到重复劳动**。

**预算放大**：`898e00c6` 在 **33,512,894**（1.68×）PASS；其余 3 例 100M 仍 UNRESOLVED（决策仅 2.37–2.53×，witness 0）。

**判定**：1 例深度不足 + 3 例启发式在该单节点上到顶；调度缺陷未获证据。附带机制解释：v0.9「配额上限 ×4 零状态变化」源于 `IntegerCountSearch:314` 的 `min(4096, …)` 字面量硬帽（单次窗口恒 4,088），`MAX_QUANTUM` 不在该路径。

**未竟**：分支层 22–28%（4.4–5.6M）构成未细分；窗口 4096 粒度未做单变量实验（待裁 A/B/C）。

## v0.9 — 2026-11-03

### M2 配额复核 A/B：D 靶面证伪 + 配额杠杆上界（纯跑测 · 源码零改动）

> A/B 两侧在 `.local/ab/` 独立编译，仓库 `src/` 零改动。详见 `m2-quota-ab.md`。

**口径**：274 例 count-model · `main-counts` · `milliseconds=0` / `work=20,000,000` · 8 例/批。

**代际核对**：A 侧 vs `build/tests/classes` 共有 **1648 类逐字节相同**（字节不同 0，现成多 10 个 view 类）；D / Q 两侧各只有 1 个类不同。

**实验 1 · D 续跑窗口**（`CountViewSearch:864` `R/8` → `R/2`）：**补丁未生效**——274 例状态差 0 且 **work 逐例完全相同 274/274**。代码复核：`:861-865` 只对 packing / partition / vector / neighborhood 四类 specialist 生效，`lcg` 主链不走。**D 候选撤回。**

**实验 2 · 配额上限探针**（`CountPortfolioPolicy:18` `MAX_QUANTUM 32768 → 131072`）：状态差 **0/274**（264 PASS / 4 UNRESOLVED / 6 UNSUPPORTED 逐条不变）；work 有差 **52/274**（最大 +449% / -89%）——**补丁生效、结果不变**。

**旧 known-limits 对账**：87 条 model-corpus（`source_revision e788d3d3`）detail **87/87 落在 2.4M–2.6M**；当前代际 **PASS 79 · UNRESOLVED 4 · UNSUPPORTED 4，0 条停在 2.5M**。4 例 UNRESOLVED 用满 ~19.99M；4 例 UNSUPPORTED 是 `finite upper bounds` 转换限制。

**结论**：2.5M 截停为旧代际现象，`m7-quota-census.md` §一与 `analysis-scaffold.md` §3.2/3.3 的归因在当前代际不成立；M2 原案动机消失。配额「量」在当前语料上不决定求解结果。

**未竟**：A / B / C 三靶面未逐处实测（只有同轴全局杠杆的间接证据）；4 例 UNRESOLVED 的难度归因未做。

## v0.8 — 2026-11-03

### M7 配额分母分类清单（只读轮 · 源码零改动）

> 只读源码 + 静态扫描；无编译产物、无跑测。详见 `m7-quota-census.md`（机读附表 `m7-allowance-census.md`）。

**计数口径**：`remainingWork()` 出现 327 次；分母为额度 / 剩余 / 总量的除法点 **312** 处（`budget.remainingWork()` 198 · `choiceRepair.remainingWork()` 2 · `allowance` 49 · `this.allowance` 1 · `availableBytes()` 53 · `maximumWork` 9）。存量两说对齐——**247 = 198 + 49**（请求剩余 + 本地额度子集）；**246** = 上一轮 `scan_allowance.py` 口径（`cap` 列只在左值为 `allowance` 时识别 `Math.min`，漏判实证 `CountRowSparsifier:34`）。

**分类**：① 裸固定分母 12 · ②a 双限（另一侧=绝对常量）144 · ②b 双限（另一侧=父层额度）29 · ②c 双限（另一侧=本地剩余）12 · ③ 除数非字面量 4 · ④ 本地额度二次切 49 · ④ 内存轴 53 · ④ 总量切分 9。分母为请求剩余的 200 处里，双限式 **185（92.5%）**。

**修正**：`analysis-docs.md` §2.5「247 处几乎全是固定 N」不成立——与 `equality-sparsification.txt:24-25` 声明的 `min(cap, remaining/N)` 同形。差异面收窄为 12 处裸分母 + 49 处本地额度二次切（29 门槛 / 20 再分配）+ 53 处内存轴无第二限（此前未进账）。

**未竟**：②b 的绑定支未逐条判（18 个 `new CountLcg(` 调用点未全表）· 49 处④的形态为人工读上下文判读 · models 2,500,001 截停点未定位到具体行。

## v0.7 — 2026-11-03

### P1 Obbt 弃权 A/B：强制上场零收益（纯跑测 · 源码零改动）

> 授权同 v0.6。本轮**不改任何仓库文件**——A/B 两侧独立编译在 `.local/ab/`。详见 `p1-obbt-ab.md`。

**口径**：274 例 count-model · 引擎 `main-counts`（唯一构造 `CountObbt` 的入口）· `milliseconds=0` / `work=20,000,000` · 8 例一批 · A/B 读同一批 `.bin`。

**代际核对**：A 侧与现成 `build/tests/classes` 共有 **1648 类逐字节相同**（现成目录多 10 个 `org/cgse/view/*`，不涉本轮）。**单变量**：B 侧仅 `CountObbt.class` 不同（size 闸 `64/256` → `1,000,000`，quota 闸不动）。

**结果**

| 面 | A 侧 | B 侧 | 差 |
|:--|--:|--:|--:|
| PASS / UNRESOLVED / UNSUPPORTED | 264 / 4 / 6 | 264 / 4 / 6 | 0 |
| work 合计 | 277,102,520 | 277,102,520 | 0 |
| 逐例对账（268 例有 work） | — | — | 语义相同 268 · 严格相同 268 · 差异 0 |

日志逐字节 diff：差异只落在 `elapsed_ms` / `improvement_ms` 两个墙钟浮点。

**补丁核验**（防「补丁没生效 ⇒ 结果当然相同」）：构造级 `refusedObbt` **52 → 0**（`refusedAny` 81 → 48），端到端仍逐例相同。

**归因**（插桩静态计数，不受 note 环形缓冲淘汰）：274 例中 `CountObbt` 只被构造 **1 次**（`af3a59ddc15f…`，98 步，该例 PASS）、size 闸在实链 **0 次**拒绝；52 例构造级弃权在实链的分布是 43 PASS / 4 UNRESOLVED / 5 UNSUPPORTED。

**判据含义**：M3（放宽 64/256）在本语料链上预期收益为 0；真正的闸门在 `IntegerCountBranch:1433-1438` 的**入场条件**，不在 `CountObbt` 的 size 闸。下一轮方向待裁（M2/M7 或 入场条件 A/B）。

### P1b 入场条件 A/B（同上轮，Rushiera 拍板「按推荐」）

**直方图**（274 例 · 条件原样 · 插桩）：`branchPoint` 只被调用 **4 次**（3 例）· 分数点 **3 次**（2 例）· 被 `obbtTried` 挡 1 · 被 `feedbackRegion` 挡 3 · 被 `repairWork` 挡 1 · 真进场 1。**最外层稀缺是"LP 给分数点"，不是任何一条闸。**

**变体（每个只改入场布尔）**

| 变体 | Obbt 上场 | 状态 | work 合计 |
|:--|--:|:--|--:|
| `open-feedback`（去 feedback 闸） | 1 例 | 264/4/6（0 差） | 277,102,520（0 差） |
| `open-feedback-repair`（去 feedback + repair 闸） | **2 例 / 228 步** | **264/4/6（0 差）** | **+66,326**（`b11051990caf` 4,269,192 → 4,335,518，+1.55%） |
| `open-current` | 直方图 0 阻断 → 零机会（未跑） | — | — |
| `open-all`（恒真） | 181 例后 NPE（`fractionalChoice` 空指针） | 不成立 | 不成立 |

**结论**：入场闸同样不是杠杆——唯一多给的机会换来 0 状态变化 + 66,326 步成本；`open-all` 崩溃证明入场闸与 `obbtPoint` / `obbtTried` 单发生命周期耦合，不是可随意放开的参数。**M3 与入场闸两条路径实测收益 0 / 负，落点不在 Obbt。**

## v0.6 — 2026-11-03

### M1b 归因第三态：`CountContinuation.refusal()`（A 案·default 方法）

> 授权同 v0.5。约束全守：未 push · 未碰 GTLCore 适配面 · 未动 `LICENSE` / `NOTICE`。详见 `m1b-third-state.md`。

**问题**：M1 的 note 通道是预算环形缓冲（32 条），父层在代码里读不到——「准入拒绝 / 配额耗尽 / 搜索收敛」三种终止在父层同形（`step()==true` 且无 counts 且非 infeasible）。

**改动**：接口加 `default String refusal()`（空串=已准入）；`CountBoolean` / `CountDecisionDiagram`(×2 门) / `CountDiophantine` / `CountDomainSearch` 在各自准入门置值并覆写；`CountViewSearch` 汇总 specialist 第三态；`CountModelReplay.Result` 加 `refusal` 分量（反射读取，对 pre-channel 基线仍可编译）；`CountModelRegression` 打印 `REFUSED <原因>` 且**退出码 2 语义不变**。

**前后对照**

| 面 | 改前 | 改后 |
|:--|:--|:--|
| 构造级 274 例 refusedDomain / Obbt / Soft / Any | 5 / 52 / 8 / 81 | **5 / 52 / 8 / 81**（判定零变化） |
| domain 列内容 | 一律 `REFUSED` | **5/5 带具名轴**（`size:variables>1024` 等） |
| 回放 96 例（views）PASS/UNRESOLVED | 23/9 · 28/4 · 25/7 | **23/9/0 · 28/4/0 · 25/7/0**（逐例 work 一致） |

**回归**：`gradle jar` exit 0 · ABI **PASS 1502 entries** · core 四片 **184 项 PASS / 0 FAIL**（窗口截断，覆盖数非全绿）。

**边界**：接口 13 类中仅 4 类有准入门——Obbt 52 / Soft 8 仍只走 note 通道；静默 `complete = true`（`upper==null` / `width>64` / `free<2` / `tryReserve` 失败）本批不动，保持两通道逐点一致；回放级 `REFUSED` 分支在 274 例上未触发（views 不构造准入门引擎）。

## v0.5 — 2026-11-02

### M1 拒绝语义可归因：31 处准入闸补具名原因（`admission_refused`）

> 授权同 v0.4（`project.md` §裁决记录）。约束全守：未 push · 未碰 GTLCore 适配面 · 未动 `LICENSE` / `NOTICE`。

**问题**：全库 342 处 `complete = true` 里，拒绝分支自身留痕仅 35/342（10.2%）；13 处 ADMIT_STATIC 中 11 处无声、20 处 ADMIT_MIXED 全部无法区分「规模超限」与「配额不足」。上层收到的是「没答案也没说为什么」，玩家侧体感为「这玩意儿合不出来」。

**改动**：统一走 `PlanningBudget.note("admission_refused", "owner=<类> <axis>:<cond> ...")`，`axis` 四值可枚举——`size`（规模）/ `shape`（结构不匹配）/ `quota`（配额不足）/ `memory`（内存预留失败）。ADMIT_MIXED 拆轴：原 `a || b || c` 短路改为逐轴判定后拼接，**语义等价**（任一成立即拒），但一次报全所有成立轴。

| 面 | 处数 | 位置 |
|:--|--:|:--|
| ADMIT_STATIC（硬闸） | 11 | CountAffineLattice ×2 · CountBinPackingSearch · CountChainSchedule · CountDecisionDiagram · CountDiophantine · CountDomainSearch · CountPackingLpSearch · CountRangedKernel ×3 |
| ADMIT_MIXED（规模+配额合流） | 20 | CountAffineLattice · CountBinPackingSearch · CountBoolean · CountBranchProbe · CountCardinalitySearch · CountCdcl · CountCongruence · CountCoverCuts · CountDecisionDiagram · CountDiophantine · CountHall · CountHermite · CountHittingSet · CountObbt · CountPacking · CountPartition · CountProbing · CountSoftSearch · CountSymmetry · GraphOutputView |
| 合计 | **31** | — |

**一处与扫描清单的偏差（实测修正）**
- `PlanSourceCycles.java:244` 的 `complete` 是 `for (int pass...)` 内的**局部变量**，不是引擎拒绝点——扫描脚本按名匹配误判，本轮**不改**（11 处沉默硬闸中剔除 1，落 10 处）。
- 口径勘误（2026-11-05 复核）：本表原记 ADMIT_STATIC 10 / 合计 30——`CountRangedKernel` 实为 3 处（`variables!=2` / 域宽 / 行越界），合计 **31**。源码实测 `admission_refused` 54 处 = M1 31 + M1c 23（另 1 处存量 note 补 refusal，同 `m1c-silence.md` §二）。

**观测实证**（`.local/audit/` 探针，构造级，274 例可复跑）

```text
1d98538b025a  UNKNOWN  1690  4883  REFUSED  REFUSED  REFUSED  ADMITTED
  owner=CountDomainSearch size:variables>1024 size:rows>4096 ;
  owner=CountObbt size:variables>64 size:rows>256 ;
  owner=CountSoftSearch size:variables>512
```

改前：同一案例只有 `REFUSED` 三字，无理由。改后：三个引擎各自被哪几条轴拦下、同时成立的全部列出。

**回归**

| 项 | 结果 |
|:--|:--|
| `gradle jar` | BUILD SUCCESSFUL in 4s |
| `check_abi.py --jar build/libs/cgse-core-0.3.2.jar` | **PASS binary API 0.3: 1502 baseline entries preserved**（`CountContinuation` 等包私有类型不入公共 ABI，本改动不破基线） |
| `tests/run.py --suite core`（`--no-compile`，shard 0/4…3/4） | **139 项全 PASS，0 FAIL**；每片在工具 120 s 窗口边界截断，未跑完全部项（诚实边界） |
| `AuditGate` 274 例本批 96 例 | 拒绝归因覆盖率：REFUSED 的每一例均带 `admission_refused` 条目；无一条只剩 `REFUSED` |

**未做**：`CountContinuation.refusal()` 接口扩展（P3）——实测该接口仅 13 个实现类，覆盖不到 20 处不实现它的引擎；先做全覆盖的 note 通道，接口第三态留待 P3。models 274 例完整快照对照（`snapshot_models.py`）未重跑，本批以 AuditGate 构造级覆盖替代。

## v0.4 — 2026-11-02

### M0 改动基线 + M6 `repair()` 浮点平局改精确残差（首行改码）

> 授权：`project.md` §范围与边界「裁决记录」——结构不匹配就改结构。约束全守：未 push（`origin` 保持只读）· 未碰 GTLCore 适配面 · 未动 `LICENSE` / `NOTICE`。

**M0 基线（对照物）**

| 项 | 记录 |
|:--|:--|
| 仓库状态 | `main@b7c4a8103657572014325b63f8a4110e0337a8a8`（2026-10-27 20:02:37 +0800）· 工作树干净（唯一既有改动 `.gitignore` 一行 `/runtime/`，属本支运行环境，不动） |
| 标准套件 | `tests/run.py --suite standard --java-home runtime/jdk-17.0.2`——现行 412 项；改前实测前 27 项全 PASS 后被工具 120 s 窗口截断，基线全绿以 `CHANGELOG` v0.2「26/26」+ 下列快照为准 |
| models 274 例快照 | `--suite models --milliseconds 0 --work 60000000 --jobs 4` → **PASS 200 / UNRESOLVED 74**，逐例 status + solution + work 落 `.local/audit/baseline-models-det.json`（`snapshot_models.py`） |

脚手架四件（`.local/audit/`，均已入 `.gitignore`）：`snapshot_models.py`（跑测日志 → 逐例快照）· `diff_models.py`（两级比对：strict / semantic）· `coverage.py`（多目录并集统计 + 剔除无编译产物目录的误判）· `probe/org/cgse/core/RepairTieBreakProbe.java`（反射驱动私有 `repair`）。

**M6 改动（`CountDemandProjection.repair` `:457-482`）**

问题：候选权重 `sum(after.doubleValue() / gain.doubleValue())`，`after` / `gain` 超出 `1e308` 时 `Infinity/Infinity = NaN`，`weight < bestWeight` 与 `weight == bestWeight` **同时为 false** → 平局判定失效，退化为遍历顺序依赖，且无任何信号。与上游 `docs/solver-throughput.txt:26-27`「计数不转浮点」口径冲突。

改法（照 `CountJump.compareMoves:337-352` 的上游已接受先例）：把候选收成 `Repair` record，比较走 `compareRepairs` —— 先比 `negative`，`weight` 有限且不等时走原 `double` 快路径，**其余情况（相等或 NaN）回落精确残差** `compareResidualSums`：两候选按行归并，逐行 `BigInteger` 相减 hinge，差值经 `CountJump.ratio` 转有界比值累加定序，最后才用 `runs` 收口。`CountJump.ratio` 由 `private` 放宽为包内可见（复用同一实现，语义未动）。

**前后对照（详情 → `m6-baseline.md`）**

| 面 | 改前 | 改后 |
|:--|:--|:--|
| 双 Infinity 用例 | `picked=variable 0`（先遍历到的，精确残差 `2^1024-1`） | **`picked=variable 1`**（精确残差 `2^1024-2`） |
| 标准套件 412 项 | 基线绿（`CHANGELOG` v0.2 + 快照） | **411/412 PASS**；第 412 项 `PortfolioOracle` 按 1500 例跑通 `missed=0 unsound=0`（默认 2000 例需 ~125 s，超单次工具窗口，改前亦在 1900 处被同一窗口截断） |
| models 274 例 | PASS 200 / UNRESOLVED 74 | **PASS 200 / UNRESOLVED 74 · 严格相同 274/274** |
| `check_abi.py`（0.3） | 未跑 | **PASS 1502 条** |
| `jar` | BUILD SUCCESSFUL（37s，v0.2 记录） | **BUILD SUCCESSFUL in 16s** |

改动面 3 文件 +139/−10：`CountDemandProjection.java`（+83/−10）· `CountJump.java`（+2/−1）· `CountDemandProjectionTest.java`（+63，新增 `repairTieBreak()` 10 条断言 + `legacyWeight` / `legacyBetter` 两个控制辅助 + `profile` 构造桥）。
未提交：工作树保持 dirty（`git status` 显示三处 `M`），**未 `git add`、未 commit、未 push**——改动留给上游评审与朋友对照。

**两处诚实边界**：① `verify_boundaries.py` 在基线上就红——它用 `text.startswith('package org.cgse.core;')` 判定包声明，而核心源文件首行是 `// Copyright (c) 2026 autumn`，于是几乎每个文件都报 `unexpected package`（我改的三个文件、以及大量未碰的文件同在名单里）；属上游脚本与源码头的既有不一致，本轮未擅改。② `PortfolioOracle` 默认 2000 例需约 125 s，超单次工具窗口，按 1500 例跑通（`missed=0 unsound=0`），逐百例进度与改前 2000 例跑测前 1500 行逐行一致。

### 上游 docs 精读第三轮（73 篇导航 + 九条结论 + 前两轮定性修订）

产物：`analysis-docs.md`（233 行）。范围 73 篇 `.txt`（当时覆盖：全文 10 / 节选 9 / 标题级 54，随后全部逐字补读，见下条）。

**导航**：73/73 分 10 族（契约架构 4 · 求解器总览调度 9 · 预算准入限额 7 · 数值与格 11 · 整数 LP 剪枝 9 · packing 8 · 等式匹配 8 · 向量分配 5 · 宿主捕获状态 7 · 回归复核 5 = 73），落 `analysis-docs.md` §一。

**四条修订（原件已加头部指针，正文待回写 → todo）**

| 前两轮结论 | 修订 |
|:--|:--|
| §二「准入拒绝无信号」= 设计疏漏 | 原则上游已立**五处**（`continuations:87` · `numeric-continuation:39-40` · `retained-progress-scheduling:111` · `hall-support-admission` · `captured-candidate-domains:187-189`）；缺的是 `CountContinuation` 接口无归因口 |
| §四 浮点排序 = 待论证 | 上游有**同构先例**——`full-regression-20261011:10-14` 修 `CountJump`「两个大负收益舍成同一 double」用的是「先减精确残差再转换」；实读 `CountJump.java:329-345` 确认 |
| scaffold §三「1/8 被切走」 | `count-corpus-followup:9-11` 明写是**单引擎本地额度**（有意）；悬念收窄为「固定分母 N vs 文档侧双限式 `min(cap, remaining/N)`」 |
| scaffold §2.1「11 处沉默硬闸」 | `note` 机制存在（`CountObbt:131` 正常路径有 note）——沉默发生在**提前 return 的拒绝路径**（`:28-37`） |

**其余五条精读结论**：`known-limits.json` 是 e788d3d3 历史快照（不可当现行失败账）· canonical backlog 有完整分类账（156 = 150 打包 + 4 等式 + 2 分配）· 上游对「保留进度」有三级判据（productive / first-visit / completion）· 「测试全绿」不覆盖语义目标 · `trial` 死字段是单点遗漏（`CountQuickSolve:74` 做对了）。

**未改上游一行代码**——本轮纯文档面。

### 73 篇逐字读毕 + 关键词索引落地

产物：`docs-index.md`（437 行）。

**逐字读覆盖 73/73**（四批域内执行，工具台账 87 次）。三篇超 `text-read` 20000 字符上限的分段补读：`missing-material-expansion.txt`（1,090 行，补 `:283-334 / :592-635 / :818-832`）· `stateful-inputs.txt`（651 行，补 `:520-651`）· `review-0.1.5-followup.txt`（354 行）。

**索引三面**（`docs-index.md`）：
① **逐篇卡 73 张**——主题 / 关键词 / 行号锚点；② **反向关键词表**——类名 → 篇目（23 个 `Count*` 族 + `GraphPlanner` / `PlanningBudget` / `IntegerCountSearch`）· 概念 → 篇目（24 条，含「预算≠不可行」「准入拒绝保持 UNKNOWN」「1/8 额度」「262,144 批次」「浮点只用于候选排序」）· 测试类 → 篇目（`PlanningWorkClockTest` / `HallSupportTest` / `CountRowSparsifierTest` 等 13 组）；③ **未入文档的缺口清单**——五个类在 docs 全文 0 命中。

**新增实质发现**：`CountObbt` / `CountSoftSearch` / `CountBranchProbe` / `CountDomainSearch` / `CountContinuation` **全部 0 命中**——`analysis-solver.md` §二 / §三 的主题在上游文档里没有对应说明，只能读码取证。这反过来加强了「契约层缺位」的定性：不是上游写了但没实现，是**根本没进入文档视野**。

**补读修正一处锚点**：`captured-candidate-domains.txt:65-70` 实读为 `min(32768, remaining search work / 4)` + 256-work 下限（首轮），`:186-189` 才是 `/8`——前一轮把两处混为一处，索引已按实读分开。

## v0.3 — 2026-11-01

### 结构审计脚手架 + 第二轮（全库量化 + 构造级探针）

产物：`analysis-scaffold.md`（报告）+ `workspace:tools/CGSE_CORE/.local/audit/`（脚手架，`.gitignore` 已排除 `.local/`）。

**五件脚手架（全部实跑通过）**

| 件 | 作用 |
|:--|:--|
| `scan_complete.py` | 按花括号深度切块，对全库 342 处 `complete = true` 定位直接支配条件 → 7 类语义 + 留痕双档 |
| `scan_allowance.py` | 全库 `allowance` 派生表达式，抽（类 / 方法 / 除数 / 上限）246 处 |
| `scan_limits.py` | `known-limits.json` 观测表量化，与除数表对照 |
| `build_model_bins.py` | 复用上游 `write_models` 把 274 例装成 `.bin`（与 `CountModelRegression` 同一份字节） |
| `probe/.../AuditGate.java` | 构造级探针：反射读各引擎私有 `complete`/`infeasible`/`allowance`/`lazy`，不跑搜索 |

**三条实证结论**

1. **`complete = true` 全库 342 处**，按「加预算能否绕开」切：13 处 ADMIT_STATIC（绕不开）· 20 处 ADMIT_MIXED · 54 处 RESERVE · 14 处 ALLOWANCE · 19 处 DOMAIN · 7 处 PRECISION · 24 处 FINISH · 73 处无条件出口 · 118 处待人工定性。**拒绝分支自身留痕仅 35/342（10.2%）**。
2. **请求预算在 `CountLcg.java:405` 被切 1/8 且是唯一绑定点**——三档预算实测（20M→2,499,999 / 160M→20,000,000 / 400M→50,000,000）严丝合缝落在 `remainingWork()/8`。`known-limits.json` 87 条 model-corpus UNRESOLVED 的截停值中位数 **2,500,001**，与 `20M/8` 完全吻合。根因：`CountContinuation` 接口没有「为何结束」的读取口，`complete` 只是各实现类私有字段。
3. **`CountObbt` 对 274 例中的 52 例静默弃权**（`lower.length > 64 || rows.size() > 256`，两轴都在打）；`CountDomainSearch` 降级 40 例、拒绝 5 例；`CountSoftSearch` 拒绝 8 例；`CountLcg` 拒绝 0 例。

**边界**：未改上游任何一行；探针依赖 `build/tests/classes` 既有字节（源码变更须重编）；118 处 UNKNOWN 未定性。

## v0.2 — 2026-11-01

### 环境落地 + 构建/跑测基线跑通 + 求解器静态分析第一轮

**运行环境（按指令落项目内）**

- JDK 17 装到 `workspace:tools/CGSE_CORE/runtime/jdk-17.0.2`（**OpenJDK 17.0.2**，来自华为云镜像 `openjdk-17.0.2_windows-x64_bin.zip` 177.6 MB / 100%）；`.gitignore` 加 `/runtime/`（单行改动，未动其他忽略项）。
- Gradle 8.8 装到 `runtime/gradle-8.8`（华为云镜像 131.6 MB / 13.2s）。**绕开 wrapper 联网**——直接用 `runtime/gradle-8.8/lib/*` 里的 `org.gradle.launcher.GradleMain`，`-g runtime/gradle-home` 隔离 GRADLE_USER_HOME。
- 下载失败四轮留档：`api.adoptium.net` SSL 握手失败 → TUNA 直链 403 → GitHub release 5.3 min 后在 20% 断连 → **华为云 OpenJDK 成功**（非 Adoptium 发行，但对 17 语言级别足够）。

**构建与跑测基线（全部实测）**

| 动作 | 结果 |
|:--|:--|
| `GradleMain jar` | **BUILD SUCCESSFUL in 37s**（4 tasks）——toolchain 17 由 runtime JDK 满足，**未改上游任何 gradle 文件** |
| `tests/run.py --suite standard` | **26/26 PASS** |
| `--suite core` | 24/24 PASS |
| `--suite oracles` | 17/17 PASS |
| `--suite fixtures`（shard 0/8、1/8） | 各 3/3 PASS |
| `--suite models --shard 0/8` | **INCONCLUSIVE**：count-model-0 solved 25 / unresolved 7；count-model-32 solved 2 / unresolved 1（全部 `work≈7.5M` 截停，远低于 60M 预算） |
| `--suite all --list-tests` | 428 项全集 |

**求解器静态分析第一轮**（产出 `analysis-solver.md`，不推 git——Rushiera 2026-11-01 定）

- 健全性主链：13 处 `infeasible = true` 核完 5 处——`ExactLinearProgram` 的 `validCertificate()` / `ExactColumnProgram` 的完全列 Farkas 复核 / `ExactRevisedProgram` 的 `CountProof.verify` / `CountSymmetry` 的「验证不过就放弃」/ `CountLcg` 的 level-0 回放 → **判「不可行」的链条比预期结实**
- 主发现：准入闸拒绝（规模 / 内存 / 域退化）统一走 `complete = true`，**无信号、无 note、不分语义**；而阈值偏窄（MAX_TERMS=12 / MAX_WIDTH=15 / Obbt lower>64 / 全量 8192 上限）→ 体感「算不出来」的来源
- 次发现：`CountDomainSearch.trial` **是死字段**（声明于 `:30`，构造器三处均未赋值，`:151` / `:170` 的 `!trial` 恒真）——类注释承诺的「试算面否定结论不外传」未接线；当前不误报靠的是 `trial=Σweights` 恒真的巧合
- 第三发现：`CountDemandProjection.repair` 用 `double` 打分，`Infinity/Infinity → NaN` 时权重比较整个失效，退化为遍历顺序依赖；与 `docs/solver-throughput.txt:26-27`「计数不转浮点」口径冲突
- `[Checked]` 留档五条查证不成立的怀疑（holeReasons 越界 / 位宽 32 移位 / CountSoftSearch 误报 / IN_DOUBT 未实现 / CountObbt 无证书 / 空 catch 吞异常）
- 诚实边界：**没找到 A 类「答错」实证**，只找到三个有明确代码位置与失效条件的结构性可疑点

## v0.1 — 2026-11-01

### 立项 + 克隆 + 勘察

**克隆**：`git clone https://github.com/autumn-2-net/CGSE_CORE.git` → `workspace:tools/CGSE_CORE/`（25,792 文件 · `main` @ `b7c4a81` · 275 commits · MPL-2.0）。

**落点判据**：属「外部克隆 / 未建档前试验工程」→ `workspace:tools/`（`Discipline` §三-16）。建档后**不迁到根**——上游仓是协作对象不是本项目产物，就近工作区更合适（同 `workbuddy-manager` / `MOSSCatalog` 形态：档案在 CCBP，工程在 tools）。

**勘察结论（全部实读取证）**

- 技术栈：Java 17 · Gradle 8.8（wrapper）· 单模块 `cgse-core` · 四 sourceSet（`main` / `test` / `view` / `viewTest`）
- 版本 **0.3.2** · 兼容线 **0.3** · ABI 基线三份（`api/0.1.txt` / `0.2.txt` / `0.3.txt`）
- 领域：Minecraft **AE2** 合成图求解引擎；Forge **1.20.1** mod（modId `cgse`），可选依赖 gtlcore / ae2
- 维护边界：本仓 = 可移植核心 + 可选 view 导出；**Minecraft 渲染 / 包 / 菜单 / 本地化 / `GraphJobCodec` 全在 GTLCore**
- 规模：`src/main/java/org/cgse/core/` 326 个 `.java` · `docs/` 73 篇专题 · 回归语料 20,927 例
- 构建入口 `gradlew.bat jar|assemble`；跑测入口 `tests/run.py --suite standard|all`；退出码 `2` = 预算未决（绝不等于 PASS/UNSAT）
- 发布仅手动 `workflow_dispatch`（只允许 `main`，tag 已存在即拒，三闸 `check_abi` → `verify_artifacts` → 上传）

**环境实测（阻塞项取证）**

- `where java` → 空；`C:\Program Files\Java` / `Eclipse Adoptium` 均不存在；`JAVA_HOME` 未设置
- 本机唯一 JDK = `workspace:tools/REA/tools/jdk21`（Temurin **21.0.12.1**）——可跑 Gradle 本体，但 `gradle/engine.gradle` 的 `toolchain.languageVersion = 17` 会让编译阶段找 17 失败
- Python：系统 `python` **3.12.10**（`tests/run.py` 可用）
- Gradle 8.8 wrapper 首次运行需联网下载（实测走到约 10%，中途超时终止，未完成下载）；因此**构建链尚未跑通一次**，`toolchain 17` 的缺失只在读码层判明（见 project.md §未决 2 的口径标注）

**建档产出**：`project.md`（定位 / 关键事实 / 工程结构 / 构建入口 / 关键判据 8 条 / 范围边界 6 条 / 未决 6 项 / 文档索引 / 素材清单）· `todo.md` · `CHANGELOG.md`（本文件）。
