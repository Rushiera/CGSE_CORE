# δ 可行性判定（w5）——段内两次 SummaryComputation 的去重

> 建立：2026-11-04 · 承 `analysis-w4.md` §五（岔口）/ §八（段内 phase 分布：p3 39.93% + p7 25.49% = 段内 65%）
> 判据来源：段成本 29.67% 总 work ↔ `RegionSelection.step` 30.8%；p10+p13 4.05% ↔ `RegionCounts.step` 0.99%
> 单元件：`.local/w5/src/org/cgse/core/DeltaSummaryProbe.java`（合成四形）
> 实测件：`.local/w5/build_w5.py`（装配）+ `.local/w5/overlay/org/cgse/core/DeltaTrace.java`（真实负载直测）+ `.local/w5/analyze_delta.py`（判读）
> 产物：`.local/w5/out/{delta.tsv, funded3.jsonl, rtrace.tsv}`
> 边界：本文件只给可行性判定与收益实测，不含实现。

---

## 〇、一句话结论

**δ 成立且已量出确数：省 1,011,209 work = 2.66% 总 work（p7 的 28.5%），20,230 次真实 region 计算 0 次值失配。** 原「上界 7.6%」不成立——那是 p7 全量，δ 只吃掉 p7 里的一遍 body 重算。**2.66% 的成本-收益比低于 β（12.58%）与 ε（14.6%），但两个对照数都是采样口径，本轮的直测已证明采样对该相位低估 1.23 倍**，故下一步应先按同一口径复核 β / ε，再决定是否值得为 δ 付改造 + 回归的代价。

---

## 一、重复点定位（读码，非推测）

| 件 | 位置 | 事实 |
|:--|:--|:--|
| 段内第一次汇总（p3） | `RegionSelection:193`（初建）/ `211` / `257` / `282`（重建后重进 p3）→ `case 3`（`:197-221`） | `new SummaryComputation<>(body, byId, budget)`，跑完 `summary = computation.result()` |
| p7 body 的构造 | `RegionSelection:304-308` | `body = Sequence([Repeat(<p3 的同一 body 实例>, full), parallelBody(tail)])`，再 `new SummaryComputation<>(body, …)`、`runs = 1`、`phase = 7` |
| 段内第二次汇总（p7） | `case 7`（`:333-338`） | 只做 `computation.step()`，结果覆盖 `summary` |
| p7 的触发条件 | `:297-313` | `recipes.size() > 1 && workingCopies > 1 && tail != 0`；`tail = runs % workingCopies` |
| body 不可变 | `PlanStep`（sealed record + `FrozenPlanSteps.freeze`） | p3 与 p7 的 body 是**同一实例**，中间只有外层多包了 `Repeat` 与尾波 |

## 二、为什么它是「合法复用」而非新语义

| 依据 | 位置 | 含义 |
|:--|:--|:--|
| repeat 的闭式本来就在 | `SequenceSummary.repeat:82-95` | `need = required − Δ(t−1)`（Δ<0）/ `change = Δt` / `peak = peak + Δ(t−1)`（Δ>0） |
| 参考实现已在用 | `SequenceSummary.of:140` | `shared.get(repeat.body()).repeat(repeat.times())`——先求 body 再线性折 |
| 同 planning pass 的 `known` 复用有上游先例 | `SummaryComputation:20/30-37` · `PlanVerification:44-49 / 64-69 / 100-103` | 以 `IdentityHashMap` 按**词实例**复用已完成 fold；「No cross-request cache」 |
| summary 不可变 | `SequenceSummary:19-37` | 冻结映射（`FrozenValues` / `owned`），消费方只读 |

## 三、真实负载直测（funded-3，一次 replay）

**量法**：在 p7 body 的构造点（`RegionSelection` 相位 5，`tail != 0` 分支）用**独立 budget** 把四个计算各跑一遍——不占真实 work，因此零扰动（实测 `work` 逐值一致）：

| 量 | 含义 |
|:--|:--|
| a = `summary(unit)` | p3 那一遍（p7 里被重复的那遍） |
| b = `summary(total)` | p7 的真实计算（同代码同输入） |
| c = `summary(total, known={unit→a})` | δ 复用的等价计算 |
| d = `summary(tail)` | 尾波一遍（δ 省不掉的那截） |

`DeltaTrace` 每步补一次 `budget.check()` 镜像 `RegionSelection.step` 的记账开销；`RegionSelection.step` 每步也是 1+1 两次 check（`advance()` 入口 + `SummaryComputation.advance()`），故口径一致。

### 3.1 结果

| 项 | 值 |
|:--|--:|
| p7 事件 | **20,230**（平均 175.3 work / 次） |
| Σ 一遍 body | 1,051,669 |
| Σ p7 全量 | 3,547,282 = **9.33%** 总 work |
| Σ p7 带 known（δ 后） | 2,536,073 |
| Σ 尾波一遍 | 1,051,669（**与 body 一遍逐值相等**——结构同一，交叉印证） |
| **δ 省下** | **1,011,209 = 2.66% 总 work = p7 的 28.5%** |
| 值保真（known 复用） | **0 / 20,230 失配** |
| 值保真（纯代数 `repeat().then()`） | **0 / 20,230 失配** |
| 每次省 | `unit − 2`（记账粒度差，非语义差；合成四形当时是 −1，差一即路径深度不同） |

### 3.2 分档（向量规模 = p7 body 的键向量长度）

| 档 | 事件数 | 占比 | Σ 一遍 body | Σ p7 | 省下/p7 | 省下/总 work |
|:--|--:|--:|--:|--:|--:|--:|
| ≤4 | 14,849 | 73.4% | 191,251 | 831,478 | 19.4% | 0.42% |
| 5-8 | 1,808 | 8.9% | 40,994 | 167,760 | 22.3% | 0.10% |
| 9-16 | 2,505 | 12.4% | 154,049 | 499,528 | 29.8% | 0.39% |
| 17-32 | 1,018 | 5.0% | 180,772 | 580,916 | 30.8% | 0.47% |
| >32 | 50 | 0.2% | 484,603 | 1,467,600 | 33.0% | **1.27%** |

结论：**事件数由小 region 堆出来（73.4%），work 由大 region 决定（50 次占 1.27 点）**；省下/p7 随向量规模单调上升（19.4% → 33.0%），与合成结论（ordinary 快路径便宜 / 带回路的配方贵）同向。最贵 100 次占 δ 总量的 49.2%。

### 3.3 零扰动与口径交叉

| 项 | 值 | 判据 |
|:--|:--|:--|
| 请求结果 | FEASIBLE / VERIFIED_FEASIBLE | 与 w4 一致 |
| 请求 work | 38,019,380 | **与 w4 逐值一致**——探针不占真实 budget |
| 墙钟 | 23.3 s（w4 为 17.7 s） | 多出的 5.6 s 是 20,230×4 次旁路计算的开销，只影响观察，不影响被测语义 |
| 采样口径对照 | w4 采样 p7 = 25.49% 段成本 = 7.56% 总 work；本轮直测 **9.33%** | 直测 / 采样 = 1.23× |

**采样低估的根因（已量出幅度）**：w4 的采样按「段 + phase」建键，低于 16,384 ticks（=1,024 work）的键按 0 计（w4 §七-7 已记该偏差方向）。而 p7 平均每次只有 **175 work**——远低于阈值，于是大量 p7 在采样里被记成 0。**结论：w4 §8.5 的 phase 分布是「短相位系统性低估」的口径，不能当绝对量用。**

## 四、δ 的两个形态与「再进一步」的否证

| 形态 | 做法 | 真实负载值一致 | 代价 |
|:--|:--|:--:|:--|
| **δ-a · known 注入** | `RegionSelection` 持 `IdentityHashMap`，p3 完成后 `put(body, summary)`，p7 构造时传入；`close()` 释放；字节按 `SummaryComputation:219-225` 的 `reserve` 模式记账 | ✅ 0/20,230 | 缓存生命周期与内存记账自管；body 每 trial 新建 → 单条目/按 trial 清理 |
| **δ-b · 代数组合** | p7 不建 `SummaryComputation`：`summary = p3summary.repeat(full).then(summary(尾波))` | ✅ 0/20,230 | 逐键算术落在 `SummaryComputation` 之外 → 必须手动 `budget.check()`，否则省下的是记账幻觉 |
| ~~δ-c · 尾波也闭式~~ | 由 `summary(body)` 按比例缩放直接得尾波 summary | ❌ **否证** | `SequenceSummary.repeat:90/92`：`delta ≥ 0` 时 `need`/`peak` 不随次数放大 → 净产出配方上给错值（合成实测 homogeneity=false，真实负载尾波独立算也印证不可省） |

## 五、收益对照与岔口（呈报，不自行选）

| 方向 | 收益 | 口径 | 备注 |
|:--|--:|:--|:--|
| **δ（本轮）** | **2.66% 总 work** | **直测（确数）** | 20,230 次事件、0 失配；约 0.47 s / 请求（按 work 线性折算） |
| β（跨轮复用 Choice） | 12.58% | 采样 | 上界；需补预算敏感性闸 |
| ε（IntegerCountSearch） | 14.6% | 采样 | 未测；同样受「短相位低估」影响，真值方向未知 |

| # | 下一步 | 代价 | 判据 |
|:--:|:--|:--|:--|
| **ε1** | 用本轮同一「直测计数器」口径复核 ε 与 β 的真值（插桩 + 一次 replay） | 小（复用 `DeltaTrace` 模式，只换打点位置） | 采样低估已被实锤 1.23×，两个对照数都要重算——否则 2.66% 与 12.58% / 14.6% 不可比 |
| **δ2** | 直接接线 δ（δ-b 更省事），做 A/B + 全量回归 | 中（改 `RegionSelection` 三处 + 记账 + 回归） | 2.66% 是否值这个代价，取决于上面复核结果 |
| **δ3** | 结案 δ，不接线（收益太薄） | 零 | 若复核后 ε / β 真值更大，δ 的边际价值进一步下降 |

**推荐 ε1**：δ 的机制侧已无未知（值保真 + 上游先例 + 确数齐备），继续投入的唯一理由是收益；而收益的比较基准（β / ε）刚刚被证明是低估口径。

## 六、风险与验收判据（接线时必须带上）

| # | 风险 | 判据 / 处置 |
|:--:|:--|:--|
| 1 | **行为合法变化**：`RegionSelection:136` 早收尾阈值吃 `currentWork()`——work 变少 → 触发更少 → 多试 order/variant → 结果可能不同（更完整） | 回归判据比 **FEASIBLE/UNRESOLVED 分布 + 计划等价或更优**，不只比 work；`tests/run.py --suite all` + `tests/datasets/run.py` 绿 |
| 2 | 记账偏移（δ-b） | 逐键算术处显式 `budget.check()`；省下的量必须对得上「一遍 body 重算」 |
| 3 | 缓存生命周期（δ-a） | 单条目/按 trial 清理 + `close()` 释放；字节沿用 `reserve/release` |
| 4 | 与 β 的收益重叠 | β 命中段整段省掉，δ 只剩未命中段的价值；δ 可先行但不可当 β 的替代 |

## 七、复跑

```bat
rem 1) 装配 + 编译（合并 w2 / w4 的补丁函数，只加 p7 打点）
python .local\w5\build_w5.py --check
python .local\w5\build_w5.py

rem 2) 跑测（探针只走独立 budget，不占被测 work）
runtime\jdk-17.0.2\bin\java.exe -ea "-Dcgse.delta=true" "-Dcgse.delta.out=<abs>\delta.tsv" ^
    "-Dcgse.rtrace=true" "-Dcgse.rtrace.out=<abs>\rtrace.tsv" "-Dfile.encoding=UTF-8" -Xmx3g ^
    -cp .local\w5\classes org.cgse.core.DatasetReplay build\d4\funded-3\catalog.bin .local\w5\out\funded3.jsonl 1 false

rem 3) 判读
python .local\w5\analyze_delta.py

rem 4) 合成四形（机制侧对照，与真实负载独立）
runtime\jdk-17.0.2\bin\javac.exe -encoding UTF-8 -nowarn -d .local\w5\classes ^
    -sourcepath src\main\java .local\w5\src\org\cgse\core\DeltaSummaryProbe.java
runtime\jdk-17.0.2\bin\java.exe -Xmx2g -cp .local\w5\classes org.cgse.core.DeltaSummaryProbe
```

**验收判据（本轮成立的对照）**：请求 `work` 与 w4 逐值一致（38,019,380）· 结果 FEASIBLE/VERIFIED_FEASIBLE · Σ 尾波 == Σ 一遍 body · known/composed 双路 0 失配 · δ = 2.66% 总 work。任一不吻合即脚手架坏了。

## 八、诚实边界

1. **n=1 请求**（funded-3 / seed=73 / amount=1000 / 40M 预算）。2.66% 未跨请求验证；w4 §9.1 的 15 请求样本仍未接探针。
2. **直测含镜像记账**：`DeltaTrace` 每步补一次 `budget.check()` 以对齐 `RegionSelection.step` 的开销。若真实 p7 的 per-step 记账与此不同，9.33% / 2.66% 会同比例偏移；已用「Σ 尾波 == Σ body」与「请求 work 逐值一致」两条内部对照兜底，但未做第三路独立口径。
3. **采样低估的幅度只对 p7 量过**（1.23×）。p3 39.93% / 段成本 29.67% / ε 14.6% / β 12.58% 是否等比例低估，未量——不要按 1.23× 直接换算别的相位。
4. **分档表的「向量规模」是 p7 body 的键向量长度**，不是 region 配方数；二者相关但不相同。
5. **`unit − 2` 的 2 是记账粒度差**，不是语义差；不据此宣称额外收益。
6. **墙钟 23.3 s 不可当作收益基线**：探针自身开销约 5.6 s。收益一律以 work 计。
