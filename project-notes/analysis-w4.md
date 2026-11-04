# B' 前置插桩（w4）——region 级跨轮输入复用率

> 建立：2026-11-04 · 承 `analysis-w3.md` §10.3「B' 跨轮增量复用 = 唯一与数据相容的方向」
> 数据源：`funded-3` 单请求（work 38,019,380 · FEASIBLE · 17.7 s）· `.local/w4/out/rtrace-funded3.tsv`（574 事件 / 188 region / 107 solve）
> 脚本：`.local/w4/build_w4.py`（装配）· `w4_reuse.py`（命中率）· `w4_reuse2.py`（分成本命中率）· `w4_stage.py`（口径对照）
> 探针：`.local/w4/overlay/org/cgse/core/RegionTrace.java`

---

## 〇、一句话结论

**命中率 78.0%（对）/ 65.8%（work 加权）——超过 50% 阈；但可复用面只有 1.0% work。** `region_counts` 自身成本实测 **0.994%**（wstack 采样归因），`analysis-w3.md` §10.2 的「46.2% work」是 **note 区间归因** 的产物（本报告同法复现 54.4%）。故按原定义（复用区域计数）收益上限 **0.57%**（命中对 work 217,388 / 总 work 38,019,380）。

**第二轮把观测面抬到 region 求解段**（§八）：段成本 **29.67%** 总 work（与 `RegionSelection.step` 采样 30.8% 吻合），段命中率 **72.9%（对）/ 76.8%（成本加权）**，**可省上界 = 12.58% 总 work**。岔口交裁（§五）。

---

## 一、插桩设计

| 件 | 位置 | 内容 |
|:--|:--|:--|
| 事件 | `RegionCounts.finish` | 每次计数尝试一条（含 `detail` 结局：`witness` / `selected_counts_blocked` / `model_limit` …） |
| region 身份 | 排序后的 recipe id 集 SHA-256 前 16 hex | SCC 无 id，故用「成员集」当身份；`region_digest` = recipe id+输入/输出/执行输出量（图内容） |
| 输入摘要 | **局部 key 集**上的 `goals / stock / seedCredits / external / preserve` | key 集 = 该 region 全部 recipe 的输入 ∪ 输出 ∪ 执行输出——全局 demand 摘要会因无关键变动而系统性低估复用 |
| 轮标识 | `GraphSolve` 实例 id | 每轮 `GraphStockViewWork` 新建一个 `GraphSolve`（= 一轮视图求解）；不用 `rounds` 字段——多视图交错 step 会串 |
| work | `budget.threadWork()` 差 | 与请求总 work 同单位 |

探针仅在 `-Dcgse.rtrace=true` 时生效，不改变 work（实测 38,019,380 逐值一致）。

---

## 二、实测结果

### 2.1 事件面

| 项 | 值 |
|:--|--:|
| region_counts 事件 | 574 |
| 唯一 region（内容身份） | 188 |
| solve（轮） | 107 |
| region_counts 自身 work 合计 | 381,548（**1.00%** 总 work） |
| 同轮内重复（同一 solve 内同 region 重跑） | **0 对** |

### 2.2 跨轮相邻对（同一 region 相邻两次出现）

| 口径 | 对 | 命中 | 命中率 |
|:--|--:|--:|--:|
| 按对计 | 386 | 301 | **78.0%** |
| 按 work 计 | 330,129 | 217,388 | **65.8%** |

成因分解：`input 同` 301 对 / 217,388 work（65.8%）· `图同·输入变` 85 对 / 112,741 work（34.2%）· **`图也变` 0 对**（同一 recipe id 集的 region 内容从不变化）。

### 2.3 分成本命中率（贵 region 是否也常命中）

| region_hash | 事件 | 对 | 命中 | 命中率 | work | work 占比 |
|:--|--:|--:|--:|--:|--:|--:|
| c8ad0c59866fe391 | 25 | 24 | 12 | 50% | 110,467 | 29.0% |
| e39a7fdda6f60721 | 26 | 25 | 18 | 72% | 109,469 | 28.7% |
| ae640c653e6440e8 | 18 | 17 | 13 | 76% | 26,604 | 7.0% |
| c1e7d1bf43fdf9c3 | 22 | 21 | 16 | 76% | 18,283 | 4.8% |
| d69ba7b2851d2ed5 | 3 | 2 | 0 | 0% | 12,007 | 3.1% |

分档（按 region 自身 work 累加）：

| 档 | region 数 | 对 | 命中率 | work 加权命中 | 覆盖 work |
|:--|--:|--:|--:|--:|--:|
| 前 50% work | 2 | 49 | 61.2% | 61.4% | 219,936 |
| 前 80% work | 8 | 160 | 73.8% | 64.4% | 306,675 |

**贵 region 的命中率低于全域（61% vs 78%），但仍在 50% 以上**——命中率不是由「便宜且高频」的 region 撑起来的。

### 2.4 覆盖率

| 项 | 值 |
|:--|--:|
| 只出现 1 次的 region | 101 / 188 |
| 多次出现的 region | 87，出现 solve 数 p50=3 / p90=15 / max=30 |

---

## 三、前提修正：46.2% → 1.0%（三重互证）

`analysis-w3.md` §10.2 用「region_counts 占 46.2% work」支撑「B' 是唯一相容方向」。三种口径并排：

| 口径 | 算法 | region_counts 占比 |
|:--|:--|--:|
| **A 自报** | note detail 里的 `work=` 求和 | 346,043 / 38,019,380 = **0.91%**（450 条，diagnostics 上限 1024 截断） |
| **B 区间** | 相邻 note 的 `nodes()` 差归给前一条 note 的 stage | 20,688,450 / 38,019,380 = **54.42%**（w3 记 46.2%，同族口径） |
| **C 采样** | wstack inclusive（stack frame 归因，无截断） | `RegionCounts.step` = **0.994%** |
| **D 探针** | `RegionTrace` 全程无截断 | 381,548 / 38,019,380 = **1.00%** |

**B 口径为什么错**：`region_counts` 每次 finish 都很快（均值 770 work），note 之间夹着的是 `RegionSelection` 的其它阶段（ordering 尝试 / `SummaryComputation` / `CountSchedule`）。把「两条 region_counts note 之间的全部 work」算给 region_counts，等于把整个 region 求解都记在它头上——54.4% 是这么来的。A/C/D 三路独立、量级一致：**真实成本 1.0%**。

### 3.1 各阶段真实占比（wstack inclusive，采样 608,307,464 ticks = 37,072 样本）

| 帧 | 占比 |
|:--|--:|
| `GraphPlanningWork.step` | 303.3%（递归重复计数，非独立项） |
| `GraphStockViewPortfolio.step` | 60.7% |
| `GraphStockViewWork.step` | 56.4% |
| `GraphSolve.step` | 47.9% |
| **`RegionSelection.step`** | **30.8%** |
| `IntegerCountSearch.step` | 14.6% |
| `SummaryComputation.step` | 13.6% |
| `GraphCompilation.step` | 6.3% |
| `GraphSupportNeighborhood.step` | 4.3% |
| `GraphSourceRanking.create` | 4.0% |
| `CountRecovery.step` | 3.3% |
| `RegionOrder.step` | 2.0% |
| **`RegionCounts.step`** | **0.99%** |

---

## 四、可复用面的两种规模

| 复用对象 | 成本面 | 命中率 | 收益上界 | 实现面 |
|:--|--:|--:|--:|:--|
| **region 计数结果**（原 B' 定义） | 1.0% | 65.8% | **0.57%** 总 work（命中对 217,388） | 小：`RegionCounts` 前查缓存 + 输入摘要比对 |
| **region 求解结果**（`RegionSelection.Choice`） | **29.67%**（§八实测） | **72.9% / 76.8%（成本加权）** | **12.58%** 总 work（命中段成本 76,499,436 work） | 大：Choice 含 `body / summary / seeds / runs`；跨轮跨视图失效判定面宽 |

**健全性前提（两者共用）**：相同输入 ⇒ 相同输出只在预算不受限时成立。`RegionCounts.advance` 的 `work_limit` / `counts_incomplete` 分支与 `RegionSelection` 的 allowance 判定都会让「同输入不同预算」得到不同结果——命中是复用的**必要**条件，不是充分条件。

---

## 五、岔口（呈报，不自行选）

| # | 方向 | 收益上界 | 代价 / 风险 | 依据 |
|:--:|:--|--:|:--|:--|
| **α** | 复用 region 计数结果（原 B'） | 0.57% | 低：只在 `RegionCounts` 前加缓存；须补预算判定 | §2.2 · §三 |
| **β** | 复用 region 求解结果（`RegionSelection.Choice`） | **12.58%** | 中高：Choice 复用要过 budget / stale / fork 继承；失效判定面 = region 全输入 | §八 · §四 |
| **γ** | 不复用，转「减少 region 求解次数」（同输入 region 在同一时刻只解一次，跨视图共享） | 上界同 β（同一事实的另一种实现形态） | 中：要改 `GraphSolve` 的 region 遍历与视图调度；与 A 同源的调度耦合风险 | §2.1 轮内对 = 0 |

**推荐 β**，但落地前先补一项量（一轮跑测，不改算法）：

1. ~~按 region 总成本加权的命中率~~ —— **已完成（§八）**：76.8%，可省上界 12.58%。
2. **预算敏感性**——同一 region 同输入但 `allowance - currentWork()` 不同时，结果差异有多大。量法：在段摘要里附余量，统计命中对里的余量分布。这是 β 的健全性闸：余量不足时 `RegionCounts` 会 `work_limit` 提前收口、`RegionSelection` 会 `complete()` 放弃，同输入可得不同结果。

---

## 六、复跑命令

```bat
rem 1) 装配 overlay 编译（复用 w2 的 patch_budget / patch_stock_view / patch_replay）
python .local\w4\build_w4.py --check
python .local\w4\build_w4.py

rem 2) 跑测（探针 + 采样器必须同开——段成本只来自采样归因）
runtime\jdk-17.0.2\bin\java.exe -ea "-Dcgse.rtrace=true" "-Dcgse.rtrace.out=<abs>\rtrace.tsv" ^
     "-Dcgse.wstack=true" "-Dcgse.wstack.out=<abs>\wstack" "-Dfile.encoding=UTF-8" -Xmx3g ^
     -cp .local\w4\classes org.cgse.core.DatasetReplay build\d4\funded-3\catalog.bin .local\w4\out\funded3.jsonl 1 false

rem 3) 判读——第一轮（计数级）
python .local\w4\w4_reuse.py   <abs>\rtrace.tsv
python .local\w4\w4_reuse2.py  <abs>\rtrace.tsv
python .local\w4\w4_stage.py   .local\w4\out\funded3.jsonl

rem 4) 判读——第二轮（段级，成本来自 wstack-segments.tsv）
python .local\w4\w4_segments2.py <abs>\rtrace.tsv.regions.tsv <abs>\wstack\wstack-segments.tsv 38019380
```

判读顺序：`w4_reuse.py` ①事件面 → ②跨轮命中率 → ③轮内对（应为 0）；`w4_segments2.py` 段成本（应 ≈ `wstack-inclusive.tsv` 的 `RegionSelection.step`）→ 命中率 → phase 分解。

**验收判据（本轮实测成立的对照）**：段成本 29.67% ↔ `RegionSelection.step` 30.8%；段内 p10+p13 4.05% ↔ `RegionCounts.step` 0.99% + 计数级探针 1.00%。两对不吻合即脚手架坏了。

---

## 七、诚实边界

1. **n=1 请求**（funded-3 / seed=73 / amount=1000 / 40M 预算）。命中率未跨请求验证；`analysis-w3.md` §9.1 的 15 请求样本尚未接探针。
2. **摘要不含预算**——`allowance` 与 `remainingWork` 不进 digest，故「命中」不等于「可复用」（§四 健全性前提）。段级 digest 已补 `forceTarget / regionScout / consumeTargetSeed` 三个标志（补后命中率 73.2% → 72.9%，影响很小），但 `catalystPolicy` 未入摘要。
3. **摘要只覆盖 region 局部 key**——这是有意的（全局 digest 会让命中率系统性偏低），但若某 region 的求解实际依赖区域外的键，摘要会漏判。当前未发现此类依赖，未证伪。
4. **`region_counts` 的 work 记账漏掉「close 未 finish」的实例**（预算中断时）。采样归因（0.994%）与探针（1.00%）量级一致，说明漏额小；未单独量化。
5. **phase 归因是近似**：`enterPhase` 在 `RegionSelection.step` 入口设置，一个 step 内跨多个 phase 时归给入口那个。p3/p7 的具体占比因此有偏，但**同族高占比**这个结论不受影响（两种算法都把 `SummaryComputation` 排第一）。
6. **`analysis-w3.md` §10.2 的 46.2% 未被判为「错」而是「口径不同」**——它度量的是「region_counts 事件区间的 work」，本报告度量的是「RegionCounts 类自身 work」。前者对「哪些阶段占轮间时间」仍有信息量，只是不能被当作该类的成本。
7. **段成本含采样阈值偏差**：低于 16,384 ticks（=1024 work）的未采样段成本按 0 计，段合计因此略低于真值；`RegionSelection.step` 采样 30.8% 与段合计 29.67% 的 1.1 点差即由此而来。

---

## 八、第二轮：段级脚手架（方法修正 + 结果）

> 承 §五 的 α/β 分叉：要判 β 的收益，必须把观测面从 `RegionCounts` 抬到「region 求解段」。抬的过程先踩了三个坑，每个坑都给出一个可复用的判据。

### 8.1 坑一：`threadWork` 差值法在多视图交替下不可用

第一版用 `budget.threadWork()` 的差值记段成本。实测**段 work 合计 = 总 work 的 682%**——同时 `wstack` 显示 `RegionSelection.step` 只占 30.8%，两个数不可能同时成立。

**根因**：`GraphStockViewPortfolio` 在同一 budget 上公平轮转多个视图，各视图的 `GraphSolve` 交替 step。线程累计量的差值会把「别的视图在这两步之间花掉的 work」算进本段。这正是 `analysis-w3.md` §10.5 记的已知近似。

**判据（可复用）**：拿累计量做差值前，先问「这个累计量在两次取样之间是否只由本对象推进」。有并发方共享 → 差值无意义。

### 8.2 修法：采样归因

`WorkStack` 每次采样时读 `RegionTrace.segmentKey()` / `phaseNow()`，把样本权重记到「段 + phase」上。样本来自当时的真实栈，与交替无关。

- 输出 `wstack-segments.tsv`（key = `regionHash@solve:region/pN`）
- 与 `rtrace.regions.tsv`（段元数据）按 key 关联
- 单位：采样记 ticks，1 work = 16 ticks

### 8.3 坑二：活跃段不止一个

段状态第一版是单槽 `ThreadLocal`。结果：A 的 `endRegion` 清掉了 B 的段（交替时同一线程上两段都开着），段合计塌到 0.02%。

**修法**：段状态按 `solveId` 存 `Map`——一个 solve 一个活段。

### 8.4 坑三：未收口的段会无界累积

只靠 `beginRegion`/`endRegion` 配对时，任何未 `close()` 的 `GraphSolve` 都会让段一直「开着」，后续采样不断累到它上面（实测段合计涨到 79.4%）。

**修法**：活跃性改由 `RegionSelection.step` 自己驱动——`markActive(true)` 在 step 入口、`markActive(false)` 在 finally。采样只在「一步正在执行」时归因，与配对是否完整无关。

### 8.5 结果（funded-3 · 4765 段 / 1216 region / 120 solve）

| 项 | 值 |
|:--|--:|
| 段成本合计 | **29.67%** 总 work（`RegionSelection.step` 采样 30.8%，吻合） |
| 有采样的段 | 756 / 4756（15.9%）——其余段低于采样阈值（16,384 ticks），成本可忽略 |
| 跨轮相邻段对 | 3540 |
| 命中（input 摘要同） | 2580 → **72.9%** |
| 按段成本加权命中 | **76.8%** |
| **可省上界** | **12.58%** 总 work（命中段成本 76,499,436 work） |
| 只出现 1 段的 region | 165 / 1216 |

段内 phase 分布（`RegionSelection` 分支编号 → 语义按源码 `case` 推断）：

| phase | 语义 | 占段成本 | 备注 |
|:--:|:--|--:|:--|
| p3 | `SummaryComputation`（第一次精确汇总） | 39.93% | 段内第一大项 |
| p7 | `SummaryComputation`（parallel waves 后重算） | 25.49% | 同一段内第二次汇总 |
| p9 | `RegionOrder` | 7.34% | 命中段里几乎为零（65K vs 32K） |
| p2 | 按 order 生成 `Batch` body | 6.91% | |
| p6 | 校验 missing | 6.41% | |
| p4 | 需求 → runs 的精确除法 | 5.22% | |
| p13 | `RegionCounts`（小 SCC） | 2.48% | |
| p10 | `RegionCounts`（大 SCC） | 1.57% | |
| p5 / p11 / p1 / p0 | seeds / singles / 初始化 / 建索引 | 4.5% | |

p10 + p13 = **4.05% 段成本 = 1.2% 总 work** —— 与第一轮 `region_counts` 自身 1.0% 独立吻合（第三重印证）。

**判据（顺带得到的）**：段内 `SummaryComputation` 跑两次（p3 + p7），占段成本 65%。这是**段内**重复，不是跨轮重复——与 A（弃线）/ B（合并 check）都不同族，属新的候选面。

### 8.6 方法清单——下次遇到「量不出来」时按这个顺序问

1. **量的单位对不对**：ticks 与 work 差 16 倍，混用会把 43.6% 写成 696%。
2. **累计量的推进方是否唯一**：多视图共享 budget → 差值法失效，换采样归因。
3. **同一个线程上是否有多个活跃对象**：交替 step 时单槽状态会互相踩，按身份分槽。
4. **状态配对是否会漏**：`begin`/`end` 配对依赖调用方自觉；把活跃性绑到「被测函数的执行窗口」比绑到配对可靠。
5. **交叉验证**：任何单一口径都可能自洽地错——本报告用了三条独立路径（探针自报 / 采样归因 / note 区间），三者一致才算数。
