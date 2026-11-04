# 求解器 work 归因脚手架（w2）——步数税与能力定义的修正

> 日期：2026-11-04 · 性质：**脚手架轮**——`src/` 零改动，全部在 `.local/w2`（`.gitignore` 已排除）
> 授权：`project.md` §范围与边界「裁决记录」（不 push / 不改适配面 / 不动 LICENSE·NOTICE）
> 前置：`analysis-strengthen.md`（七次「加机制」零收益）
> 缘起：Rushiera「找不到怎么强化求解能力，那就找如何找到这种能力」

---

## 〇、一句话结论

**work 的 70.3% 不是算法计算，而是推进步数税——约 2,670 万次 `budget.check()`，每次固定 1 work（= 4 次 SCAN 的价位）。** 去掉这笔计费后，同一 40M 预算在 5 分钟内跑不出结果（基线 17.3 秒）——它同时是预算的主要消耗项与搜索的限流器。

---

## 一、为什么先建尺子

七次实验的共同点是**选点靠读码直觉**，而真正的开销分布（`STOCK_VIEW.SEARCH` 47.8% / `COORDINATOR.OTHER` 15.6%）是第七次才量出来的，且那两个桶内部仍是黑箱。改动落在 0.7%–6.4% 的面，零收益是必然。

本轮不猜机制，先造一把能**把 work 量到代码行**的尺子。

## 二、脚手架清单（可复跑）

全部在 `workspace:tools/CGSE_CORE/.local/w2/`。

| # | 件 | 作用 | 跑法 |
|:--:|:--|:--|:--|
| 1 | `overlay/org/cgse/core/WorkStack.java` | work 加权栈采样器：按 tick 量采样（非时间），栈帧聚合 | 由 build 脚本自动装配 |
| 2 | `build_w2.py` | 取仓库原件 → 单点插桩 → 全量 javac 到 `.local/w2/classes` | `python .local/w2/build_w2.py`（`--no-check-charge` 生成对照变体；`--check` 只自检） |
| 3 | `analyze_w2.py` | 把栈样本折成调用树 / 叶子路径 / 重入账 | `python .local/w2/analyze_w2.py <stacks.tsv> <outdir>` |

**插桩点（仅三处，均在 overlay 副本）**

| 位置 | 改动 | 覆盖 |
|:--|:--|:--|
| `PlanningBudget.traceCharge` | 加 `WorkStack.charge(ticks, compilation, trace.context())` | 100% 的 work 记账出口（`chargeTicks` 是唯一汇聚点） |
| `PlanningBudget.operation` / `operations` | 加 `WorkStack.operation(op, ticks)` | 基本操作类型分布 |
| `DatasetReplay.solve` | `enableMetrics()` 去掉 `scheduler != null` 前置 | workers=1 也带 Origin×Stage 维度 |

**产物六张表**：`wstack-summary / -exclusive / -inclusive / -context / -stacks / -operations`（`.tsv`）。

**跑法（funded x1000）**

```text
java -ea "-Dfile.encoding=UTF-8" -Xmx3g "-Dcgse.wstack=true" ^
     "-Dcgse.wstack.out=.local/w2/out/funded" "-Dcgse.wstack.threshold=16384" ^
     -cp .local\w2\classes org.cgse.core.DatasetReplay ^
     build\d4\funded-3\catalog.bin .local\w2\funded.jsonl 1 false
```

## 三、自证：采样与既有构成图逐行吻合

同一请求（funded `extra-first` x1000），采样覆盖 **608,307,464 / 608,310,072 ticks = 99.9996%**，37,072 样本，去重后 392 条栈。

| Origin/Stage | w2 采样 | `analysis-strengthen.md` §四 |
|:--|--:|--:|
| `STOCK_VIEW/SEARCH` | **47.75%** | 47.8% |
| `COORDINATOR/OTHER` | **15.37%** | 15.6% |
| `SOURCE_GRAPH/SEARCH` | **11.48%** | 11.5% |
| `SUPPORT_NEIGHBORHOOD/SEARCH` | 3.92% | 3.9% |
| `STOCK_VIEW/ASSEMBLE` | 3.20% | 3.1% |
| `ALLOCATION/SEARCH` | 2.67% | 2.8% |
| `COORDINATOR/SEARCH` | 2.59% | 2.3% |
| `BOOTSTRAP/SEARCH` | 1.10% | 1.1% |

work 总量 608,310,072 ticks ÷ 16 = **38,019,380**，与基线逐字节相同——采样不改行为。

## 四、方法级归因（exclusive = 该方法自己的代码吃掉的 work）

| # | 方法 | 占比 |
|:--:|:--|--:|
| 1 | `GraphPlanningWork.step` | 19.64% |
| 2 | `SummaryComputation.advance` | 13.57% |
| 3 | `GraphSolve.step` | 13.44% |
| 4 | `RegionSelection.advance` | 12.97% |
| 5 | `GraphStockViewWork.check` | 10.89% |
| 6 | `GraphCompilation.step` | 3.94% |
| 7 | `GraphSourceRanking.rankingCheck` | 2.76% |
| 8 | `GraphStockViewWork.stepMeasured` | 2.01% |
| 9 | `GraphSourceRanking.scan` | 1.93% |

**前 5 名合计 70.5%。** 结构上：`GraphPlanningWork.step` 在单条栈里最多出现 6 次（3 层各占 100%），即求解跑在多层 step 递归里。

## 五、行级归因：前 6 行 = 65.1%

| # | 位置 | 占比 | 该行是什么 |
|:--:|:--|--:|:--|
| 1 | `GraphPlanningWork.step:223` | 15.43% | `budget.check()` |
| 2 | `GraphSolve.step:109` | 13.20% | `budget.check()` |
| 3 | `RegionSelection.advance:129` | 12.97% | `budget.check()` |
| 4 | `GraphStockViewWork.check:82` | 10.89% | `budget.check()`（私有包装） |
| 5 | `SummaryComputation.advance:49` | 7.57% | `budget.check()` |
| 6 | `SummaryComputation.advance:73` | 6.00% | 循环体内每迭代一次 `budget.check()` |
| 7 | `GraphPlanningWork.step:222` | 4.21% | `budget.compilationCheck()` |

`PlanningBudget.check()` 的实现是 `chargeTicks(WORK_SCALE)`——**固定 1 work**，不随工作量变化（`PlanningBudget.java:205-207`）。

按此推算：0.703 × 38,019,380 ≈ **2,673 万次检查点**。一次 check 的价位 = 1 work = 4 次 `SCAN`（`SCAN` 4 ticks）。

对照：`operation()` / `operations()` 两条基本操作入口合计只占 **4.12%**（其中 `SCAN` 3.50% / `INTEGER` 0.62% / `RATIONAL` 0%）——**96% 的 work 是算法自报的估算，不是逐操作计费**。

## 六、A/B：这 70% 是限流器，不是冗余

`.local/w2/classes-nocheck` 变体：`check()` / `compilationCheck()` 只做 `checkpoint()`（取消语义保留），不计费。

| 侧 | work | elapsed | 状态 |
|:--|--:|--:|:--|
| 基线 | 38,019,380 | 17.35 s | `VERIFIED_FEASIBLE` |
| 不计费 | —（未取到） | **> 300 s 未完成**（工具超时，进程已清理） | 未跑完 |

**判读：** 检查点的计费不是记账噪声，它承担着「限制推进步数」的职能。去掉后，同一 40M 预算可容纳远多于此前的步数，运行时间失去约束。

**这条 A/B 否掉了一个看似显然的方向：** 「把 check 改成免费就能回收 70% 预算」——不成立，会把预算约束变成时间失控。

## 七、能力定义的修正

原来的问题「怎么让求解器更强」在数据面前的正确形态是：

> **work 预算 = 步数税 + 真计算量。真计算量只占 30%，其余 70% 由「推进了多少步」决定。**
> 所以「能力」≈ 每步的信息量 × 预算内能走的步数。

这解释了七次零收益：那些实验都在改「真计算量」里的算法（0.7%–6.4% 的分支策略面），而 70% 的预算消耗由**步数**决定，与它们无关。

进一步：D4 已证「21 轮 / 10 轮迭代才出 witness，前 20 轮无效」——无效轮次付出的是全额步数税。

## 八、下一步候选（待裁）

| # | 方向 | 上限 | 性质 | 风险 |
|:--:|:--|--:|:--|:--|
| **A** | **步数归因**：把 2,670 万次 check 按「轮次 / 阶段 / 子目标」切开，量出无效轮次占多少步 | 观测，不改码 | 纯脚手架延伸 | 低 |
| **B** | **减少推进步数**：合并检查点粒度（如循环体内每 N 次迭代检查一次） | 上限 70%，但改变 work 口径语义 | 需上游评审 | 中高（口径契约） |
| **C** | **无效轮次判据**：找出能提前判定「本轮无望」的信号 | 只削无效部分 | 算法改动 | 中（可能改变结果） |
| **D** | 结案：把本轮结论并入归档 | — | — | — |

**建议 A 先行**：它决定 B 与 C 的上限——不知道无效轮次占多少步，就无法判断合并检查点或提前放弃能拿回多少。

## 九、诚实边界

- **nocheck A/B 未跑完**：工具 300 s 超时后进程被清理，未取到该侧 work 值；「5 分钟未完」是实况，不构成「永远跑不完」的断言。
- **emitter 样本混合 9 个请求**：`wstack` 无按请求分段能力，emitter 目录的 totalTicks 是 9 例合计；本报告的定量结论全部基于 funded 单请求样本。
- **方法级归因依赖 JIT 内联形态**：叶子帧 = 计费发生的逻辑方法（行号来自 javac `-g`）；极端内联下的帧折叠未单独验证。
- **未做**：`GraphSourceRanking.rankingCheck:443`（2.76%）语义未读；`GraphStockViewWork.stepMeasured:109`（2.01%）未读。
- **步数税 70.3% 是按行级 top7 求和**（15.43+13.20+12.97+10.89+7.57+6.00+4.21）；另有散落在其它方法的 `check()` 调用点未单列，总占比只会更高。

---

## 十、步数归因（方向 A 已执行）：84.9% 是步数税，93.6% 花在无结果的尝试上

**做法**：`PlanningBudget.check()` 每调用一次计 1（`WorkStack.check()`），按 work 位置分桶（每 1,000,000 ticks）出直方图；同时把 `budget.note()` 的环形缓冲上限 32 → 1024，`DatasetReplay` 落盘 `diagnostics`。note 自带 `stage@work` 时间戳——轮次边界不必另建状态。

```text
python .local/w2/analyze_steps.py .local/w2/phases.jsonl .local/w2/out/phases/wstack-checkhist.tsv
```

### 10.1 总账

| 项 | 值 |
|:--|--:|
| 总 work | 38,019,380 |
| **总 check** | **32,297,205**（1 work / 次） |
| **步数税占 work** | **84.9%** |
| 真计算余量 | 5,722,175（15.1%） |

比行级 top7 求和的 70.3% 高出 14.6 个百分点——散落在 100+ 个调用点上的 `check()` 未进 top 榜。**步数税是 work 预算的主体，不是边角。**

### 10.2 按阶段（note 时间轴切段）

| stage | work | 其中 check | check 率 | 占 work |
|:--|--:|--:|--:|--:|
| `region_counts` | 17,549,658 | 16,261,525 | 93% | 46.16% |
| `stock_view` | 8,138,886 | 6,523,647 | 80% | 21.41% |
| `region` | 4,541,786 | 4,400,105 | 97% | 11.95% |
| `compile` | 1,486,332 | 301,733 | 20% | 3.91% |
| `count_closure` | 921,922 | 924,851 | 100% | 2.42% |
| `linear_macro` | 724,820 | 750,000 | 103% | 1.91% |
| `plan_egraph` | 604,594 | 662,036 | 110% | 1.59% |

前三阶段合计 79.5%，check 率 80%–97%——**这些阶段的"计算"几乎就是检查点本身**。

### 10.3 轮次序列：65 条 round 记录，只有 1 条产出解

note 里带 `round=` 的条目共 **65 条**（9 个 variant 各自一串轮次），其中 `result=FEASIBLE` 的只有 **1 条**：

| 项 | 值 |
|:--|--:|
| 唯一成功轮 | `variant=2, round=21`，`work@38,008,547` |
| 该位置占总 work | **99.97%**（38,008,547 / 38,019,380） |
| 65 条 round 记录合计 | 7,833,633 work（**20.6%**） |

前 64 条记录的结果分布：`MISSING_INPUT` / `MISSING_SEED` / `UNKNOWN`——没有任何一条产出候选解。

**修正（本轮自纠）**：初版把「成功前的累计 work（35.6M / 93.6%）」写成「64 次无效尝试的消耗」。按 round 条目自身的时间差重新核算，65 条 round 记录**合计只占 7.83M（20.6%）**，其余 79.4% 落在 round 之外的子阶段（`region_counts` / `region` / `compile`，属于每轮内部的子计算）。时间轴事实（witness 出现在 99.97% 处）成立，归因对象写错，此处更正。

### 10.3.1 失败模式分层（C1）

| result | 条数 | spent | 占 round spent | 平均 | 最大 |
|:--|--:|--:|--:|--:|--:|
| `MISSING_SEED` | 36 | 12,903,150 | **86.0%** | 358,420 | 596,437 |
| `UNKNOWN` | 11 | 1,365,420 | 9.1% | 124,129 | 237,509 |
| `FEASIBLE` | 1 | 422,809 | 2.8% | 422,809 | 422,809 |
| `MISSING_INPUT` | 17 | 307,692 | 2.1% | 18,099 | 265,020 |

**口径**：`spent` = `usedWork() - completedWork`（`GraphStockViewWork` 自身维护的每轮实测成本），比相邻 note 的 work 差准确——后者只算 note 间隔，会漏掉同位置多条 note 与其它视图的时间片。round 自身 spent 合计 **14,999,071（39.5% 总 work）**，其余 60.5% 落在 round 之外的子阶段（编译 / region_counts / 预备）。

**三类失败的成本相差一个数量级**：`MISSING_INPUT` 平均 1.8 万（快速判死）、`MISSING_SEED` 平均 35.8 万（搜完才知缺种子）。**靶点只有一类：`MISSING_SEED`——36 条吃掉 round spent 的 86%。**

### 10.3.2 轮次成本与收敛形态（C1 复核）

| 项 | 值 |
|:--|--:|
| 单轮最大 | `variant=2, round=9` = **596,437**（4.0% of round spent） |
| 成功者 `variant=2` | 16 轮合计 **6,336,064 spent（16.7% 总 work）**，前 15 轮全部 `MISSING_SEED` |
| 单轮典型成本 | 35 万–60 万 work，**其中 85% 是 check** → 每轮有效计算约 6 万 |
| 其它 variant | `6`：10 轮 4.30M · `9`：8 轮 1.21M · `8`：9 轮 0.73M · 其余 ≤0.5M |

**判读（修正版）：** 不存在「某一条异常昂贵」——`variant=2` 的 16 轮成本曲线平坦（245K → 596K → 355K → … → 423K），**失败与成功之间没有成本差异**。求解靠的是「每轮推进一点前沿（`resume_retained_frontier`），16 轮后够到解」。真正的低效不是某一轮，而是**每轮的 85% 预算花在检查点上，只留 6 万给推进**。

### 10.4 与 §九 A/B 的合并判读

§六证明去掉 check 计费会让运行时间失控（预算不再限制步数）。本节补上另一半：

> **步数税不能免，但可以少付——少付的唯一途径是减少无效步数。**

84.9% 的预算被「推进了多少步」吃掉；witness 出现在 work 的 99.97% 处，而真正解出它的策略（`variant=2`）只用了 1.8% 的 work。能力上限由**策略顺序与提前放弃**决定，与单步算法优化无关——这是七次「加机制」零收益的最后一块拼图。

### 10.5 诚实边界（本节新增）

- **区间分配是近似的**：note 的 `@work` 是采样点，同位置可有多条 note；`checks_between` 按 work 区间累加，部分 stage 出现 check 率 >100%（`plan_egraph` 110% / `count_shared_demand` 182%）——区间边界与桶粒度（1,000,000 ticks）造成的错位，不影响总量结论。
- **round 记录跨 variant 混排**：9 个 variant 各自计轮次，表中"本轮 work"是相邻 note 的 work 差，不等于该 variant 的净消耗（其它 stage 的 note 插在中间）。
- **note 上限已从 32 调到 1024**（overlay 变体），原始上游仍是 32——即上游自带的诊断只能留最后 32 条，21 轮勉强够、多策略场景会截断。
- **总 check 数 32,297,205 为精确计数**（非采样），24 位小数列偏差不超过桶边界。

---

## 十一、C2：轮次机制的结构（源码实测）

### 11.1 时间片轮转，不是串行策略扫描

| 位置 | 事实 |
|:--|:--|
| `GraphStockViewPortfolio.java:150` | `if (done.get(active) \|\| selected.turn >= 32_768) selected.next();` |
| 同文件 `:134-137` | 每次推进后按 `budget.threadSearchWork()` 增量累加 `selected.turn` |
| 同文件 `:57` | 初始视图顺序 `new int[] { 0, 4, 5, 3, 2, 1 }` |

**每个视图每拿到 32,768 work 的片就被轮转**——不存在「一个策略跑到底再换下一个」的顺序问题。§10.3.2 初版「`variant=7` 独占 6.23M」的推测因此作废：那 6.23M 是两条 note 之间**全部视图**的时间片总和。

### 11.2 每轮成本此前不可观测（本轮已补）

`GraphStockViewWork.java:224-226` 已算出 `spent` 与平滑值 `roundCost`，但 `:238` 的 note 只写 `round / variant / recipes / result / missing_keys`——**成本字段没落盘**。本轮在变体里补 `spent=` 一项（零语义改动），§十 的精确轮次成本即由此取得。建议上游采纳同一行：诊断面不改任何判定。

### 11.3 放弃判据已存在，阈值宽松

`GraphStockViewWork.java:232`：`stalledRounds = Math.min(8, stalledRounds + 1)`，仅在「`missingKeys` 未下降」时累加；`:235` 用 `missing >= missingKeys` 判「无进展」。

数据印证：`variant=2` 连续 15 轮 `MISSING_SEED` 仍未触发放弃——因为它的 **`missing_keys` 在下降**。判据测的是「有没有进展」，不是「有没有成功」，这在语义上是正确的；但 15 轮 × 40 万 work 的代价说明它仍然太慢。

### 11.4 可行动候选（待裁）

| # | 动作 | 预期 | 风险 | 依据 |
|:--:|:--|:--|:--|:--|
| **A** | 合并循环体内的 `check()`（每 N 次迭代一次） | **同时降 work 与真实 CPU** | 取消 / 超时粒度变粗 | §五 热行 6 单项 6%；全库同类点未枚举 |
| B | 时间片 32,768 → 131,072 | 减少切换与重复扫描 | 慢视图拖累快视图 | 需 A/B，无数据 |
| C | 上游补 `spent=` 到 round note | 零风险诊断增强 | 无 | 本轮已实测 |
| D | `stalledRounds` 阈值收紧 | 早放弃 | 可能砍掉「第 9 轮才成功」的路径 | 与 §11.3 的语义冲突 |

**A 是唯一同时降低 work 与真实耗时的方向**，但它需要先量上界——本轮已量（§11.5）：**上限只有 13.1%**，不是 85%。

### 11.5 侦察：循环内 check 的实际上限（1352 个调用点全量扫）

`.local/w2/scan_check_sites.py`：剥离注释与字符串，花括号栈判定每个 `check` / `checkpoint` / `compilationCheck` 调用点是否在 `for`/`while`/`do` 块内，再与行级 work 表 join。

| 项 | 值 |
|:--|--:|
| check 类调用点总数 | **1,352**（`check` 1,134 · `checkpoint` 114 · `compilationCheck` 104） |
| 其中循环体内 | **671（49.6%）** |
| 循环内热行贡献的 work | **13.1%**（79,634,068 ticks · 72 个热行） |
| 非循环热行贡献的 work | **81.7%**（496,949,476 ticks · 54 个热行） |

**判读：调用点数量与 work 占比反向。** 一半的调用点在循环里，但它们只贡献 13.1% 的 work；85% 的步数税来自「每方法 / 每步入口各一次」的检查点——与推进步数一一对应，合并调用点削不掉。

**结论（A 方向的封顶）：** 合并循环内 `check` 的上界是 13.1%，且需改 671 处中的一部分。它不能解决 85% 的步数税，只能拿回零头。**真正的杠杆是「减少推进步数」或「让每步承载更多信息」——那是算法与调度层，不是记账层。**

---

## 十二、下一轮脚手架设计：有效推进归因（w3，待开工）

**目标**：回答「一步是否有效」。现在步数可数（32.3M 次 check）、成本可量（84.9% 预算），但**「哪些步是有效推进」未知**——这是唯一还没量化的变量，也是「减少步数」方向的前提。

**三件（按成本排序，前两件零插桩）**

| # | 件 | 数据源 | 产出 | 判据 |
|:--:|:--|:--|:--|:--|
| 1 | **轮次进度序列** | `spent.jsonl` 的 round note（已含 `missing_keys=`） | 每轮 `missing_keys` 与 `spent` 的对应曲线 | 单调下降速度 > 0 = 有进展 |
| 2 | **停滞段划分** | 上件 + `stalledRounds` 语义 | 连续无进展轮次的 work 占比 | 占比 > 30% 才值得做「提前放弃」 |
| 3 | **前沿规模采样** | 需插桩：每 N 步记 frontier 规模 / 已证下界 | 步级推进曲线 | 曲线平台段 = 无效推进 |

**分叉判据（这条决定后续路线）：**

- 无效推进占比 **> 30%** → 走「提前放弃」，收益上限 = 该占比；
- 无效推进占比 **< 30%** → 步数本身是必要的，方向转向「提高每步信息量」（合并小步 / 换更强的推进算子）。

**验收**：新脚手架必须能解释 §10 那 84.9% 的步数里有多少是无效的——给不出这个数，就还没到能改算法的程度。

**已就绪的前置**：`.local/w2` 的采样器与编译链可直接复用（`build_w2.py` 加一处插桩即可再生变体）；轮次成本已能精确读取（`spent=`）。
