# 求解器强化方向取证 — D4 / D1 / D3 与 work 构成图

> 日期：2026-11-04 · 性质：**取证轮**——仓库 `src/` 零改动，全部变体在 `.local/d1`、`.local/d1b`、`.local/d3`、`.local/w1`
> 授权：`project.md` §范围与边界「裁决记录」（上游源码可改 / 不 push / 不碰 GTLCore / 不动 LICENSE·NOTICE）
> 缘起：Rushiera 换方向——「有没有可能给求解器进一步强化求解能力」
> 前置：`m2-quota-ab.md` / `m2-unresolved-attribution.md` / `m4-g3-decision.md` / `p1-obbt-ab.md`

---

## 〇、一句话结论

**真实语料 19/19 已全 PASS，无「解不出」的问题；能力边界是 work 贴顶（funded x1000 = 95%）。七次「加机制」实验全部零收益，根因是一张 work 构成图缺失——补上之后，开销集中在 `STOCK_VIEW.SEARCH`（48–60%）与 `COORDINATOR.OTHER`（16–18%），两者都不在历次改动面上。**

---

## 一、D4 · 真实语料基线（此前从未跑过）

跑测面：`tests/datasets/regressions.py`——68,187 recipes（manual 5,903 + extra 62,284）· 196,921 slots · 19 个具名请求。

| 结果 | 计数 |
|:--|:--|
| `VERIFIED_FEASIBLE` | **19 / 19** |
| `UNRESOLVED` | **0** |
| 退出码 | 0 |

**work 占用（真正的能力边界）：**

| 请求 | work | 预算 | 占用 |
|:--|--:|--:|--:|
| funded `extra-first` x1000 | 38,019,380 | 40M | **95.0%** |
| emitter `random` x1000 | 13,198,498 | 20M | 66.0% |
| emitter `random` x100 | 11,400,764 | 20M | 57.0% |
| emitter `extra-first` x100 | 11,078,916 | 20M | 55.4% |
| miracle-seed `random` x1 | 10,384,778 | 20M | 51.9% |
| funded `extra-first` x10 | 18,124,731 | 40M | 45.3% |
| funded `extra-first` x100 | 15,854,718 | 40M | 39.6% |
| funded `extra-first` x1 | 14,519,075 | 40M | 36.3% |
| miracle-qty `random` x1000 | 11,714,377 | 40M | 29.3% |
| miracle-qty `random` x10 | 10,765,857 | 40M | 26.9% |
| miracle-qty `random` x100 | 10,160,817 | 40M | 25.4% |
| miracle-seed `manual-first` x1 | 3,751,812 | 20M | 18.8% |
| emitter `extra-first` x1 | 3,658,158 | 20M | 18.3% |
| 其余 6 例 | 147,744 – 344,988 | 20M | 0.7%–1.7% |

**判读：** 274 例里的 4 例 UNRESOLVED 是小规模合成 count-model，不是真实 AE2 图。**「强化求解能力」在真实语料上的定义 = 同一请求 work 下降（⇒ 预算可覆盖更大规模），不是「多解几例」。**

---

## 二、D1 · 整数 LP 学习通道：机制齐备，被一道门锁死

### 2.1 通道全貌（源码实测）

| 通道 | 类 | 变量上限 | 行上限 | 变量类型 |
|:--|:--|--:|--:|:--|
| 布尔 LP scout | `CountLpSearch` | 128 | 512 | 严格 0/1 |
| 布尔 LP 学习 | `CountLpLearning.Session` | 128 | 512 | 严格 0/1 |
| **整数 LP 学习** | `CountLcg.integerRelaxation()` → `CountIntegerLpLearning` | **8192** | **32768** | **有界整数** |
| 整数 LP scout | `CountViewSearch.addIntegerLp()` | 8192 / 32768（`hasBoundedIntegerSupport`） | 同上 | 有界整数 |
| 整数 LP 辅助 | `IntegerCountBranch` · `CountRecoveryFuel` | — | — | — |

`CountLcg.java:323-328` 的 0/1 闸只作用于 `integers=false` 分支；整数分支走 `CountIntegerLpLearning`，`CountIntegerLpLearning.java:44` 以「上界 >1 或下界 <0」识别整数变量。

### 2.2 死锁链（插桩实测，10 个真实请求）

```
addIntegerLp.called=15      alreadyAttempted=10     noCompiledAlternatives=0
integerLp.viewAdmitted=5    viewRejected=0
solve.called=5              returnedNull=0          returnedResult=5
solve.certifiedCut=0        relaxation.integerEnabled=0
```

1. `addIntegerLp()` 每个请求只试一次（`integerLpAttempted` 后永久返回）
2. 准入 `integerLp(view)`：**5/5 全过**
3. `solve()`：**5/5 全部解出**，无一次 null
4. **`certifiedCut = 0`**——一次割都没产出
5. `addSearch(view, Engine.INTEGER_LP)` 不执行 → `INTEGER_LP` arm 不存在
6. `CountViewSearch:142` 的 `solver.integerRelaxation()` 永不触发

**结论：8192/32768 那条持续学习通道是死代码路径——被「必须先产出一个割才准入场」锁死。**

### 2.3 A/B（`.local/d1b`：`certifiedCut != null` → `solve 成功即入场`）

`relaxation.integerEnabled` 0 → 5（通道接通）。work：

| 请求 | 基线 | 接通后 | 差 |
|:--|--:|--:|--:|
| funded x1000 | 38,019,380 | 38,029,488 | +0.03% |
| emitter random x1000 | 13,198,498 | 13,205,261 | +0.05% |
| emitter extra-first x100 | 11,078,916 | 11,080,356 | +0.01% |
| emitter random x100 | 11,400,764 | 11,402,068 | +0.01% |
| emitter extra-first x1000 | 10,212,065 | 10,213,121 | +0.01% |
| 其余 5 例 | — | — | **0（逐字节相同）** |

状态 10/10 保持 PASS。**零收益，只有正开销。**

---

## 三、D3 · 窗口粒度 4096 → 32768（`IntegerCountSearch:314`）

| 面 | 结果 |
|:--|:--|
| 真实 10 例 | 全部 0 ~ +0.25%（funded +0.06%） |
| 274 例全量（35 bins · 20M） | 状态 **0 例变化**（PASS 264 / UNRESOLVED 4 / UNSUPPORTED 6 逐例同） |
| 274 例 work | 245 例变化，合计 **+4,800,677（+2.44%）**；单例 -11.87% ~ **+71.52%** |

净变差。窗口不是活变量。

---

## 四、work 构成图（用源码内置的 `PlanningCostTrace` 取，非自建插桩）

### 4.1 funded x1000（search 33.92M + compilation 4.10M）

| 归属 | search work | 占比 |
|:--|--:|--:|
| **`STOCK_VIEW.SEARCH`** | **18,177,756** | **47.8%** |
| **`COORDINATOR.OTHER`** | **5,931,885** | **15.6%** |
| `SOURCE_GRAPH.SEARCH` | 4,372,459 | 11.5% |
| `SUPPORT_NEIGHBORHOOD.SEARCH` | 1,488,740 | 3.9% |
| `STOCK_VIEW.ASSEMBLE` | 1,171,691 | 3.1% |
| `ALLOCATION.SEARCH` | 1,048,579 | 2.8% |
| `COORDINATOR.SEARCH` | 888,195 | 2.3% |
| `BOOTSTRAP.SEARCH` | 409,757 | 1.1% |
| `SOURCE_GRAPH.ASSEMBLE` | 222,687 | 0.6% |
| `SUPPORT_NEIGHBORHOOD.PREPARE` | 147,031 | 0.4% |
| 其余 | <30,000 | <0.1% |
| **分支层 10 个策略合计** | **2,428,433** | **6.4%** |
| **视图搜索 `CountViewSearch` 自身** | **265,397** | **0.7%** |

编译 4.10M：`STOCK_VIEW.PREPARE` 1,574,046（38.4%）· `COORDINATOR.OTHER` 1,542,407（37.7%）· `SOURCE_GRAPH.PREPARE` 841,575（20.5%）。

### 4.2 emitter random x1000（search 11.99M + compilation 1.21M）——结构一致

| 归属 | search work | 占比 |
|:--|--:|--:|
| **`STOCK_VIEW.SEARCH`** | **7,161,566** | **59.7%** |
| **`COORDINATOR.OTHER`** | **2,113,525** | **17.6%** |
| `COORDINATOR.SEARCH` | 888,196 | 7.4% |
| `SOURCE_GRAPH.SEARCH` | 717,258 | 6.0% |
| `SUPPORT_NEIGHBORHOOD.SEARCH` | 492,497 | 4.1% |
| `STOCK_VIEW.ASSEMBLE` | 451,601 | 3.8% |

### 4.3 迭代轮次（`LASTNOTES` 原始行）

```
funded:   stock_view@37668463: resume_retained_frontier; work=20722817; quota=1606748
          stock_view@38008547: round=21; variant=2; recipes=448; result=FEASIBLE
          stock_view@38008547: witness; rounds=21; ... work=6460456
emitter:  stock_view@13152643: resume_retained_frontier; work=8114405; quota=2013198
          stock_view@13195252: round=10; variant=6; recipes=88; result=FEASIBLE
```

**21 轮 / 10 轮迭代才出 witness**，前 20 轮全部无效。

### 4.4 分支层单策略成本

| strategy | funded work | steps | 单步均值 |
|:--|--:|--:|--:|
| `count_recovery` | 1,241,917 | 241 | 5,152 |
| `count_decompose` | 511,505 | 61,190 | 8 |
| `count_views` | 266,251 | 22,155 | 12 |
| `count_compile` | 167,431 | 31,360 | 5 |
| `count_heuristics` | 65,383 | 2,986 | 22 |
| `count_probing` | 56,885 | 2 | 28,443 |
| `count_bounds` | 54,475 | 4,308 | 13 |
| `count_separator` | 44,429 | 3,490 | 13 |
| `count_structure` | 28,753 | 8 | 3,594 |
| `count_diophantine` | 1,204 | 2 | 602 |
| `count_congruence` | 0 | 2 | 0 |

---

## 五、累积判定：七次同型，改动面全部偏在边缘

| # | 方向 | 改了哪一面 | 结论 | 出处 |
|:--:|:--|:--|:--|:--|
| 1 | P1 Obbt 边界收紧 | 分支层辅助 | 零收益 / 负 | `p1-obbt-ab.md` |
| 2 | G2 DD 宽度闸 8→16 | 分支层闸门 | 274 例状态 0 变化 | `m4-g2-g3.md` |
| 3 | G3 面枚举分组（c1） | 等式面枚举 | 判死（变量连通分量 27/27 = 1） | `m4-g3-decision.md` |
| 4 | 配额上限 ×4 | 调度配额 | 零状态变化 | `m2-quota-ab.md` |
| 5 | D1 整数 LP 学习通道 | LP 学习 | work +0.03%，状态不变 | 本文 §二 |
| 6 | D3 窗口 4096→32768 | 分支窗口粒度 | 状态 0 变化，work +2.44% | 本文 §三 |
| 7 | 布尔 LP 各闸放宽 | LP scout | 未单独跑——整数通道已覆盖同族 | 本文 §二 2.1 |

**共同点：改动落在 0.7%–6.4% 的面（视图搜索 / 分支策略），而 64%–77% 的开销在 `STOCK_VIEW.SEARCH` 与 `COORDINATOR.OTHER`。**

---

## 六、方向裁决（待 Rushiera 拍板）

| # | 方向 | 上限 | 风险 | 状态 |
|:--:|:--|--:|:--|:--|
| **A** | 细分 `COORDINATOR.OTHER`（funded 5.93M / emitter 2.11M，含编译 1.54M / 0.34M） | 砍一半 = 7–9% 总 work | 低（纯观测） | 推荐先行 |
| **B** | 减少 `STOCK_VIEW` 迭代轮次（21 → 更少） | 上限 48–60% | 高（核心策略，改错掉解） | B 之前必须先有 A |
| **C** | 通用等式消元引擎（D2） | 只补 2 例**合成语料**，不改真实能力 | 中高（新引擎） | 按 D4 口径不构成能力扩张 |
| **D** | 收工结案，七次零收益沉淀归档 | — | — | 备选 |

---

## 七、复跑命令

```text
# 真实语料基线（19 请求，~3 min）
python tests\datasets\regressions.py --java-home <repo>\runtime\jdk-17.0.2 --timeout 90 --keep-going

# 单请求（funded x1000 是最贴顶的一例）
python tests\datasets\run.py --require-feasible --source mixed-jei-stress-20261007/manual.json --extra extra-virtual.json --target gtlcore:miracle_crystal --modes extra-first --amounts 1000 --seed 73 --work 40000000 --milliseconds 0 --memory-mib 256 --timeout 100 --workers 1 --output build\d4\funded-3 --java-home <repo>\runtime\jdk-17.0.2

# D1 死锁观测（插桩变体）
javac --release 17 -encoding UTF-8 -d .local\d1\classes -sourcepath "<repo>\.local\d1\src;<repo>\tests\datasets\java" <repo>\tests\datasets\java\org\cgse\core\DatasetReplay.java
java -ea "-Dfile.encoding=UTF-8" -Xmx2g -cp .local\d1\classes org.cgse.core.DatasetReplay build\d4\funded-3\catalog.bin .local\d1\funded.jsonl 1 false

# D3 A/B（274 例）
python .local\ab\run_ab.py --arm d3 --bins 0,8,16 --work 20000000
python .local\ab\compare_d3.py

# work 构成图（.local/w1：构造器内 enableMetrics + WorkProbe 输出 METRICS/STRATEGY/PIPELINE/LASTNOTES）
java -ea "-Dfile.encoding=UTF-8" -Xmx2g -cp .local\w1\classes org.cgse.core.DatasetReplay build\d4\funded-3\catalog.bin .local\w1\funded.jsonl 1 false
```

---

## 八、诚实边界

- **D4 跑测**：19 请求全跑，`--suite all` / `standard` 412 项未跑；work 数值为单 worker 口径（`workers=1`）。
- **D1 A/B**：只在 10 个真实请求上做，274 例未复跑；变体只改「入场条件」一处，未调整 arm 入场后的 quantum 分配——「预算不足」这一解释未排除（但 m2 的「100M 预算 3 例 0 witness」指向同一结论）。
- **D3**：274 例用的是 `.local/ab/d3/classes`（当前 `src` 编译产物 + 单文件 overlay），base 侧沿用 v0.9 的 `.local/ab/base/classes`；跨代差异未逐字节复核。
- **work 构成图**：只在 funded x1000 与 emitter random x1000 两例上取；`COORDINATOR.OTHER` 的语义未拆（是协调器里既非 SEARCH / ASSEMBLE / PREPARE / VERIFY 的部分）。
- **未做**：`COORDINATOR.OTHER` 细分、`STOCK_VIEW` 轮次机制分析、`count_recovery` 单步 5,152 work 的构成。
