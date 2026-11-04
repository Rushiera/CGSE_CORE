# CGSE_CORE — 求解器静态分析（第一轮）

> 触发：Rushiera 转述朋友意见「这东西好像求解器有点问题」
> 范围：**静态读码 + 本地构建跑测取证**，不改上游一行代码（本轮仅 `runtime/` 落地 + `.gitignore` 加一行）
> 证据分级：`[Observed]` = 实读源码 / 实跑命令 · `[Checked]` = 怀疑过、查证后不成立 · `[Inferred]` = 读码推断、未跑复现
> 基线：`main` @ `b7c4a81`（v0.3.2）· JDK 17.0.2 · 见 `CHANGELOG.md` v0.2
> 🔴 **文档面修订（第三轮）**——§二 / §四 / §六 的定性以 `analysis-docs.md` §三 修订清单为准：原则上游已立五处（缺的是实现承载位）· §四 有上游同构先例（`CountJump`）· §六 的 87 条属历史快照 + 单引擎额度

---

## 〇、先说结论

**上游自己记录了这件事**——不是臆测。两条硬证据：

| 证据 | 内容 |
|:--|:--|
| `docs/audit-followup.txt:139-187` | 「Remaining gaps, rather than completed checkboxes」——五项自认缺口，明写**「The next algorithm change should target a retained, reproducible failure in items 1 or 2」** |
| `tests/known-limits.json`（1,950 行） | 全量 UNRESOLVED 观测表，`oracle_truth: SAT` 却 `observed: UNRESOLVED` 的条目成片存在 |

所以「求解器有点问题」有两种可能，**性质完全不同**，必须分开：

| 类 | 表现 | 严重度 |
|:--|:--|:--|
| **A · 答错**（不健全） | 明明可行却判不可行 / 明明不可行却给出配方 | 🔴 真 bug |
| **B · 答不出**（不完备） | 该算的算不出来，返回 UNRESOLVED / 超预算 | 🟡 已知缺口，上游承认 |

本轮静态分析**没找到 A 类的实证**，但找到 **B 类的一处结构性放大**（§二）和 **一处「声明与实现不符」**（§三）。这两条最可能就是朋友体感「有点问题」的来源。

---

## 一、健全性主链：结论比预期结实 `[Observed]`

判「不可行」的路径有 13 个 `infeasible = true` 设置点，逐个查证书门：

| 位置 | 证书/证明门 | 判定 |
|:--|:--|:--|
| `ExactLinearProgram.java:453 / :504` | `validCertificate()`：证书系数全非负 + 列系数全非负 + upper<0，三条同真才 INFEASIBLE，否则降 UNKNOWN | ✅ 严格 |
| `ExactLinearProgram.java:601-614` | 上面那个门本身（唯一实现） | ✅ |
| `ExactColumnProgram.java:101-107` | 主问题 INFEASIBLE 后取 master 证书做完全列 Farkas 复核，improving 非空或 bound≥0 即降 UNKNOWN | ✅ 严格 |
| `ExactRevisedProgram.java:100-164` | phase0 判 INFEASIBLE 前必过 `CountProof.verify(..., 2_000_000) == VERIFIED` | ✅ 严格 |
| `CountObbt.java:61-63` | 依赖 `ExactLinearProgram` 的门（上表第 1 行）——**自身不再复核** | ✅（链条成立） |
| `CountSymmetry.java:274-297` | `CountProof.verify(...) != VERIFIED` 时**直接放弃**（complete=true，不置 infeasible） | ✅ 宁漏不误 |
| `CountLcg.java:1573-1577` | `highest == 0` 的冲突回放到 level 0，每步过 `reason()` 链 | ✅ 概念可靠 |
| `CountLcg.java:473-476` | `CountDifference.contradiction(...)` 产出 `differenceProof` 才置 | ✅ |
| `CountCdcl` / `CountComponents` / `CountCoreSoft` / `CountDecisionDiagram` / `CountHittingSet` / `CountReducedCost` / `CountProbing` / `CountNetwork` | 各自子求解器内部精确判定——**已补核（2026-11-04），逐类判据见下方「补核」小节** | ✅ 6 · 🟡 2 |

**另有一条全局护栏做得对**：`CountQuickSolve` 的 `trial` 标志（`sourceFace()` 造的试算面）在 `:169 / :264 / :360` 一律 `infeasible = impossible && !trial`——试算面的否定结论不外传。

> 这一条是本轮唯一**主动设计过**的「防误报不可行」机制。上游显然知道这个坑。

### 补核（2026-11-04）：8 类待验逐类判据 `[Observed]`

| 类 | 置位点与门 | 判定 |
|:--|:--|:--|
| `CountCdcl` | `analyze():536-540` level-0 空子句才置；`explanation():510-528` 自检加权理由完整性（不足即抛 `"Incomplete weighted reason"`） | ✅ 严格 |
| `CountComponents` | `fail():413-416` 链三个子求解器（`:137/:150/:165`）的结论，自身不判 | ✅ 链条成立 |
| `CountCoreSoft` | `:60-62` 空核置位；`CountCoreMinimize:53-94/:207-208` 每次 shrink 过 `CountProof.verify`（`:114-117` 注释明写 traceRoot 非证书） | ✅ 严格 |
| `CountHittingSet` | `:70-73` 空核置位（精确）；准入拒绝已带 `admission_refused` note（`:53-56`） | ✅ 严格 |
| `CountProbing` | 收紧一律经 `certify():91-101` → `CountProof.verify != VERIFIED` 即 false（`:132-145`） | ✅ 严格 |
| `CountNetwork` | `certifyCut():361-398`，`:395` `verify != VERIFIED` 即 return | ✅ 严格 |
| `CountDecisionDiagram` | 精确枚举构造，`:139-141` frontier 空 / `:154-155` 全负残差才置；但 infeasible 的 `Diagram` 证明只入 `proofJournal`（`:232-233`）**无 verify 复核**（低于 `ExactRevisedProgram` 的 2M verify 门）；另有 3 处静默准入无 note（`:58-60` upper==null / `:63-65` 宽度越界 / `:78-80` tryReserve） | 🟡 判定自洽 · 证据层级低一档 |
| `CountReducedCost` | `:57-59` 链 `ExactLinearProgram` 门 ✅；`:115-117` 每条 cut 过 `CountProof.verify` ✅；**`:121` `objectiveLower > cutoff` 处无证明复核**——条件性结论（只在 `c·x ≤ cutoff` 约束下成立） | 🟡 现况安全 · 同族隐患 |

**辅助类（上表外顺带核）**：`CountBoolean:325-331` `infeasible = pinned == 0`——`pinned` 计数语义（`:116-120`：lower>0 且未锁死 = 试算面孔）与注释一致 ✅；`CountMeetInMiddle:1124-1131` `finish()` 总守卫 `infeasible = impossible && trials == 0` + 逐处置位带 note（观测性最好的一类）✅。

**`CountReducedCost:121` 的消费点核查**：唯一消费方 `CountSoftSearch:121-136` 只用 `fixing.infeasible()` 证 incumbent 最优（`closed` → `finish(true)`，不置自身 `infeasible`），且 `best == null` 时不可能进入该分支——**条件结论未被当全局结论消费**。风险是潜伏态：与 §三 `trial` 死字段同族（「条件性负结论可外传」），将来新消费点直读 `infeasible()` 即踩雷。

---

## 二、🔴 主发现：准入闸拒绝时**没有任何信号**，会被上层当成「算完了」

### 现象 `[Observed]`

核心求解器里，**准入被拒 / 资源不足 / 域退化** 三种情况统一用同一个出口表达：`complete = true; return;`

以 `CountDomainSearch` 构造器为例（`src/main/java/org/cgse/core/CountDomainSearch.java:47-77`）：

```
if (lower.length > 1024 || rows.size() > 4096) { complete = true; return; }        // 规模超限
if (span.signum() < 0)                        { complete = true; return; }        // 域本身已空（low>high）
if (!budget.tryReserve(bytes))                { complete = true; return; }        // 内存不足
```

同款出口在 `CountObbt.java:23-45` 出现三次（`lower>64` / `rows>256` / `allowance<4096` / `tryReserve` 失败），在 `CountSoftSearch.java:56-77` 出现三次。

### 问题在哪 `[Observed]`

1. **`complete` 一个布尔承载四种语义**：真搜完 / 准入拒绝 / 域退化 / 内存不足。调用方 `complete()` 只能看到 `true`，分不出是哪种。
2. **准入被拒的阈值又特别窄**，窄到正常 AE 合成图容易撞：
   - `CountEqualitySupport.MAX_TERMS = 12` / `MAX_WIDTH = 15` / `MAX_RANGE = 2048`（`:12`）
   - `CountMessageSearch` 的 `MAX_WIDTH = 15`
   - `CountBoundedSchedule`：`total ≤32` / `active ≤16` / `total*active ≤512`
   - `CountProof` 的 0-1 背包核验：profit ≤ 17 bit 且 total ≤ 131072
   - 全量计数模型：**8,192 资源/配方的准入上限**（`docs/audit-followup.txt:141-146` 自认）
   - `CountAffineLattice`：48 坐标上限 + 精度守卫（`:148-153` 自认）
3. **大多数拒绝点不写 `budget.note`**（对比：`CountDomainSearch:197` 完成时写了 note、`CountEqualitySupport:103` 配对时写了 note）。也就是说**成功有日志、被限流没日志**。

### 为什么这会被体感成「求解器有问题」

`docs/public-input-contracts.txt:36-39` 立了原则：**「预算耗尽不是不可行证明」**。这条原则在**有 UNKNOWN 出口的地方守住了**。

但上面这些准入闸走的是 `complete=true`，**不产生 UNKNOWN**——上层拿到的是「没答案、也没说为什么」，最终落到玩家侧就是「这玩意儿合不出来」，而不是「求解器超限了」。

> 判据对齐：这是 `AGENTS.md` §一-4「失败必须可见——回落默认值必须可观测」的同类病。上游对 **UNKNOWN** 有极严格的纪律，对 **准入拒绝** 没有。

### 建议方向（不改上游代码，供上游参考）

| 优先级 | 动作 |
|:--:|:--|
| P1 | `complete=true` 的准入拒绝点补 `budget.note("admission_refused", reason)`——成本一行，收益是所有「算不出」可归因 |
| P2 | 把 `complete` 拆成 `finished` / `refused(reason)` 两态（或加一个 `refusalReason` 字段），让调用方能区分 |
| P3 | 被最频繁撞上的闸（`MAX_TERMS=12` / `MAX_WIDTH=15` / Obbt 的 `lower>64`）配一个「撞闸了」的遥测计数 |

---

## 三、🟡 次发现：`CountDomainSearch.trial` 是**死字段**，类注释承诺的保护没接线

### 现象 `[Observed]`

`CountDomainSearch` 的类注释（`:10-14`）明确写：

> *An optional lower-bound face of a wide domain can yield a witness but **cannot export a negative conclusion or a learned clause** about the unrestricted model.*

对应实现是该类的 `trial` 字段，两处消费：

| 行 | 代码 | 意图 |
|:--|:--|:--|
| `:151` | `infeasible = search.infeasible() && !trial;` | 试算面的「不可行」不外传 |
| `:170` | `if (!trial) for (var conflict : search.learnedConflicts())` | 试算面的学习子句不入全局池 |

**但 `trial` 在整个类里从未被赋值**——`private boolean trial` 声明在 `:30`，构造器 `:32-104` 三个重载都没有 `this.trial = ...`，也没有 setter 被外部调用（`CountSoftSearch` 构造 `CountDomainSearch` 时不传 trial，`CountQuickSolve` 的 `trial` 是它**自己的**字段）。

结论：`!trial` 恒为 `true`。

### 影响 `[Observed]`

| 面 | 现状 | 风险 |
|:--|:--|:--|
| `:151` 的 infeasible 过滤 | 恒不生效 | 若将来真有「宽域下界面」走这条路，其否定结论会**直接外传**——正是注释要防的那件事 |
| `:170` 的学习子句过滤 | 恒生效（全导出） | 若将来有试算面，子句会污染跨策略池 |

**当前是否有活路径踩到**：`CountSoftSearch` 传给 `CountDomainSearch` 的行里带了一条 `objective ≤ trial` 的**搜索界行**（`CountSoftSearch.java:139-142`）。这条行会让子搜索在界不够时判 infeasible。我核过这条路的出口（`CountSoftSearch.java:145-152`）：`if (best == null) { infeasible = true; ... }`，而 `best == null` 时 `trial = upper = Σweights`（所有软约束权重之和），该界恒真、不构成收紧 → **当前不会误报**。`[Checked]`

但「现在是安全的」靠的是 `trial` 初值恰好取到了平凡上界这条巧合，不是靠注释承诺的机制。这是**声明与实现不符**，属结构性隐患。

### 动态实锤（2026-11-04 · 本地探针）`[Observed]`

探针 `.local/ab/probe-src/org/cgse/core/TrialProbe.java`（`src/` 零改动）：unrestricted（x+y≥2，见证 (1,1)）与 bounded_face（加界行 x+y≤1）对照，反射读 `trial` / `learned` 字段。

```
CASE           COUNTS     INFEASIBLE  TRIAL_FIELD  LEARNED_EXPORTED
unrestricted   [1, 1]     false       false        0
bounded_face   null       true        false        1
```

**结论**：带界行的面判 infeasible **直接外传**（`infeasible()==true`，而 unrestricted 模型明明有解 (1,1)），`trial` 字段恒 `false`——类注释「cannot export a negative conclusion or a learned clause」两条承诺**均未接线**（负结论外传 + 学习子句入池 1 条）。上文「当前不会误报」的 `[Checked]` 仍成立，但安全性升格定性：**API 层是假契约，消费点全靠调用侧巧合守卫**。复跑命令：`runtime/jdk-17.0.2/bin/javac -cp .local/ab/base/classes -d .local/ab/base/classes .local/ab/probe-src/org/cgse/core/TrialProbe.java` → `java -cp .local/ab/base/classes org.cgse.core.TrialProbe`。

### 建议方向

| 优先级 | 动作 |
|:--:|:--|
| P1 | 给 `CountDomainSearch` 补 `trial(boolean)` 或构造参数，并让 `CountSoftSearch` / `CountQuickSolve` 的试算面真正传进去 |
| P2 | 或者删掉 `trial` 字段与两处 `!trial`，把类注释改成与实际一致——**声明不该比实现强** |

> ✅ **已落地（2026-11-04，M1c 轮 · 拍板 B 案）**——P1 接线：`trial(boolean)` + `faceInfeasible()`，`CountSoftSearch:144/:149` · `CountQuickSolve:319/:322` 试算面真传；P2 不做。TrialProbe 三用例实锤 → `m1c-silence.md` §三 / §四。

---

## 四、🟡 第三发现：浮点进入了计数候选排序（违反自身口径）

### 现象 `[Observed]`

`CountDemandProjection.repair()`（`:457-481`）用 `double` 做候选权重排序：

```java
BigInteger extra = CheckedAmounts.ceilDiv(debts[row], source.getValue().negate());
double weight = 0;
for (var e : columns.get(v)) {
    BigInteger after = debts[e.row()].add(e.coefficient().multiply(extra));
    if (after.signum() <= 0) continue;
    BigInteger gain = ...;                       // 该行最大负系数
    weight += after.doubleValue() / gain.doubleValue();   // ← 计数转浮点
}
if (negative < bestDebts || negative == bestDebts && (weight < bestWeight || ...))
```

同一份代码里 `debts` / `extra` / `gain` 全是 `BigInteger`（精确），只有**打分**这一步转 `double`。

`after` / `gain` 一旦超过 `1e308`（AE 里长整数计数 + 大系数完全可达），`doubleValue()` 返回 `Infinity`：
- `Infinity / Infinity` = **NaN** → `weight < bestWeight` 与 `weight == bestWeight` **同时为 false** → 该候选的权重比较整个失效，退化成「先遍历到的赢」。
- 即：**排序从「按权重」退化成「按遍历顺序」**，且无任何信号。

### 与上游自身口径的冲突 `[Observed]`

| 文档 | 原话 |
|:--|:--|
| `docs/solver-throughput.txt:26-27` | *No fluid/count quantity is converted through floating point.* |
| `docs/public-input-contracts.txt`（架构侧） | 计数不得转浮点、长整数必须精确保留 |

同一套代码里，浮点被允许用于**候选选择**（`CountJump.compareMoves` 有注释明说「浮点只选候选，score 与 witness 仍走精确整数」——这条纪律是对的），但 `repair()` 这处**没有对应注释，也没有 fallback**。

### 影响评估 `[Inferred]`

- 不产生错的结论（repair 是 increment-only，最终计划另有校验）→ **不是 A 类 bug**
- 但会让**同一输入在不同遍历顺序下产出不同的修复路径**，进而影响搜索步数——表现为「同样的图，有时快有时慢」，正是「好像有点问题」的典型体感
- 且 `NaN` 分支没有任何告警

### 建议方向

> ✅ **已落地（2026-11-02，M6）**——按 P1 的方向实现，但保留部分 double 快路径：`compareRepairs` 先比违例行数，`weight` 有限且不等时走原 double 比较，**相等或 NaN 时回落精确残差**（逐行 `BigInteger` 相减 hinge，经 `CountJump.ratio` 转有界比值累加）。改前 `picked=variable 0`（遍历顺序），改后 `picked=variable 1`（精确残差）；models 274 例逐例 work 不变 → 详见 `m6-baseline.md`。

| 优先级 | 动作 | 状态 |
|:--|:--|:--|
| P1 | 要么按 `CountJump` 的口径加注释说明「浮点只影响候选顺序」，要么把 `weight` 换成 `BigInteger` 交叉相乘比较（精确、无 NaN） | ✅ 采「精确残差 + 注释」，并在 `repair()` 处写明「转换只是快路径」 |
| P2 | 若要保留 double，至少对 `Infinity/NaN` 给一个确定性兜底（如按 `after` 的 `bitLength()` 降级比较） | ✅ NaN 已由精确残差兜底，无需位长近似 |

---

## 五、`[Checked]` 查过但不成立的怀疑（留档，避免下轮重复）

| 怀疑 | 查证结果 |
|:--|:--|
| `CountLcg.holeReasons[variable]` 数组越界（`new Hole[16]` 只给 width≤15 的变量分配） | **不成立**。`removeHoles` 的 `removed = domainMasks & ~allowed`，未分配变量的 `domainMasks` 是默认值 0 → `removed == 0` 立即 return；width≤15 时下标范围 0..15，正好装满 16 |
| `CountLcg.currentMask` / `hull` 的 `1 << (last-first+1)` 在宽度 32 时移位取模 | **不成立**。上游 width 被 `MAX_WIDTH=15` 卡死，`last-first ≤ 15`，够不到 31 |
| `CountSoftSearch` 在目标界行不可行时误报原问题不可行 | **不成立**。`best == null` 时 `trial = upper = Σweights` 恒真，不构成收紧（详见 §三） |
| `push()` 返回 null 的 IN_DOUBT 语义没实现 | **不成立**。`tests/core` 里有 `GraphRuntimeNullOutcomeTest` 专测这条 |
| `CountObbt` 用 LP 的 INFEASIBLE 直接判死、无证书 | **不成立**。`ExactLinearProgram` 在抛 INFEASIBLE 之前必过 `validCertificate()`，Obbt 只是这个门的消费者 |
| 空 catch 吞异常 | **不成立**。全库 281 处 `catch` 多带具名异常；254 处 `) {}` 绝大多数是 record 紧凑构造器，非空 catch；`TODO` 全库 0 处 |

---

## 六、本地跑测基线 `[Observed]`

环境：JDK 17.0.2（`runtime/jdk-17.0.2`）+ Gradle 8.8（`runtime/gradle-8.8`）+ Python 3.12.10

| 动作 | 结果 |
|:--|:--|
| `gradle jar` | **BUILD SUCCESSFUL in 37s** · 4 tasks executed |
| `tests/run.py --suite standard` | **26/26 PASS**（含 core 24 + view 2） |
| `--suite core` 单独 | 24/24 PASS |
| `--suite oracles` | 17/17 PASS（截断处全部 PASS，未见 FAIL） |
| `--suite fixtures --shard 0/8` 与 `1/8` | 各 3/3 PASS（`fixtures-0/1/2`） |
| `--suite models --shard 0/8` | **INCONCLUSIVE** —— `count-model-0`：solved 25 / **unresolved 7**；`count-model-32`：solved 2 / **unresolved 1** |
| `--suite all --list-tests` | 428 项（Java 主程序 + oracles + Python + adapter + datasets 全集） |

**`models` 的 UNRESOLVED 全部是 `work=7500000` 附近截停**——远低于默认预算 60,000,000。也就是说这些模型不是撞了 60M 预算，而是在**内部子求解器约 7.5M（)`allowance` 的 1/8）时就被截断**。这与 §二 的 `allowance` 分配逻辑相关，值得作为下一轮**动态取证**的头号目标。

> 注意：`INCONCLUSIVE` 是 run.py 的正确语义（`2` = 预算未决，**绝不可当 PASS 也不可当 UNSAT**）。这本身是上游做对的地方。

---

## 七、下一轮建议（按性价比排序）

| # | 动作 | 判据（怎么算做完了） |
|:--:|:--|:--|
| 1 | **动态取证 §二**：挑一个 `known-limits.json` 里的 known-SAT unresolved 案例，用 `--model-trace` 跑，看它到底停在哪一类 `complete=true`（准入 / 内存 / 域 / 真耗尽） | 拿到该案例的实际 refusal 分类，而不是「超预算」 |
| 2 | **量化 `allowance` 分配**：`models` 那 8 个 unresolved 全卡在 7.5M，查是谁把 60M 切到 1/8 | 定位到具体的 `remainingWork()/N` 除法点，并给出该案例在满预算下是否可解 |
| 3 | **§三 的 trial 死字段**做最小复现：构造一个带界行的 `CountDomainSearch`，验证其 infeasible 会外传 | 有可复跑的用例，不靠读码断言 |
| 4 | 核完剩下 8 处 `infeasible = true` 的证书门 | 13/13 全表填满判定 |
| 5 | ~~`CountDemandProjection.repair` 的 NaN 场景~~ ✅ **已做（M6）** | 构造 `after/gain` 双 Infinity 的用例，看是否退化成顺序依赖 —— **实跑坐实**：改前 `picked=variable 0`（顺序），改后 `picked=variable 1`（精确残差）→ `m6-baseline.md` |

---

## 八、诚实边界

- 本轮是**静态读码 + 本地基线跑测**，**没有**构造反例去实际证伪某个判定 → 不宣称「找到了 bug」，只宣称「找到了三个结构性可疑点，其中两个有明确代码位置与失效条件」
- `count-model` 的 UNRESOLVED 是**观测**，不是「求解器错了」——上游文档已明写这类结果不可当 UNSAT
- `docs/` 73 篇只精读了 6 篇（architecture / releases / public-input-contracts / migration-validation / audit-followup / solver-throughput + full-regression 前 45 行），其余 67 篇未读 → 上游可能已在别处解释或修复了 §二/§三
- `infeasible = true` 的 13 处只核完 5 处
- 未做：把本地改动推给上游（Rushiera 已定：只做静态分析，不推 git）
