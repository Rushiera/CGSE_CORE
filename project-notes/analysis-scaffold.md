# CGSE_CORE — 求解器结构审计（脚手架 + 第一轮）

> 触发：Rushiera「听说求解器有点问题，正攻之前先搭脚手架，在结构和静态分析上先找出一轮内容」
> 范围：**只读分析 + 本地探针**，不改上游一行代码。脚手架全部落在 `.local/audit/`（`.gitignore` 已排除 `.local/`，不污染上游仓）
> 证据分级：`[Observed]` = 实跑探针 / 实读源码 · `[Inferred]` = 读码推断未跑 · `[Checked]` = 怀疑过、查证不成立
> 基线：`main` @ `b7c4a81`（v0.3.2）· JDK 17.0.2 · Gradle 8.8（均在 `runtime/`）
> 🔴 **文档面修订（第三轮）**——§三「1/8 切走」改读为「有意的单引擎本地额度 · 固定分母」（`count-corpus-followup.txt:9-11`）；§2.1「沉默硬闸」补准确性说明（`note` 机制存在，沉默发生在提前 return 的拒绝路径）。详见 `analysis-docs.md` §2.5 / §2.2

---

## 〇、这一轮解决了什么

上一轮（`analysis-solver.md`）是**举例三处**的静态读码。这一轮把它变成**全库量化 + 可复跑的动态探针**：

| 上一轮 | 这一轮 |
|:--|:--|
| 「同款出口在 CountDomainSearch / CountObbt / CountSoftSearch 出现三次」 | 全库 **342 处** `complete = true` 逐点分类，每点给出语义归属与留痕状态 |
| 「准入拒绝点补 budget.note 是一行成本」 | 实测**只有 10.2% 的拒绝分支自身留痕**，其余 89.8% 无声 |
| 「models 的 8 个 unresolved 卡 7.5M，查是谁切的」 | 定位到**唯一绑定点 `CountLcg.java:405`**，并用三种预算实测复现比例关系 |
| 「挑一个案例跑 --model-trace」 | 已跑；同时搭出**构造级探针**，一次扫完 274 例给出每例四个引擎的准入判定 |

---

## 一、脚手架清单（可复跑）

全部在 `workspace:tools/CGSE_CORE/.local/audit/`，产物在 `out/`。

| # | 件 | 作用 | 跑法 |
|:--:|:--|:--|:--|
| 1 | `scan_complete.py` | 按花括号深度切块，对 342 处 `complete = true` 定位直接支配条件 → 语义分类 + 留痕双档 | `python .local/audit/scan_complete.py` |
| 2 | `scan_allowance.py` | 全库 `allowance` 派生表达式，抽（类 / 方法 / 除数 N / 上限 cap）→ 配额链路量化 | `python .local/audit/scan_allowance.py` |
| 3 | `scan_limits.py` | `tests/known-limits.json` 观测表量化，与除数表对照 | `python .local/audit/scan_limits.py` |
| 4 | `build_model_bins.py` | 复用上游 `tests/cases/run.py::write_models` 把 274 例装成 `.bin`（与 `CountModelRegression` 读的同一份字节） | `python .local/audit/build_model_bins.py` |
| 5 | `probe/org/cgse/core/AuditGate.java` | **构造级探针**：反射读各引擎私有 `complete`/`infeasible`/`allowance`/`lazy`，判定「准入 / 降级 / 拒绝」，不跑搜索 | 见下方编译与运行 |

**探针编译与运行（实跑通过）：**

```
javac -encoding UTF-8 --release 17 -cp <repo>\build\tests\classes -d .local\audit\out\probe-classes .local\audit\probe\org\cgse\core\AuditGate.java
java -cp "<repo>\build\tests\classes;<repo>\.local\audit\out\probe-classes" org.cgse.core.AuditGate <bin...> [work]
```

**脚手架的前置依赖（诚实标注）**：探针的 `-cp` 指向 `build/tests/classes`——那是**上次跑测留下的编译产物**。上游源码一旦改动，必须重编这份 classes（`python tests\run.py --suite core --output ...`，或去掉 `--no-compile` 跑一次），否则探针读到的是旧字节。探针本身不参与上游构建，`build.gradle` 与 `tests/` 均未动。

---

## 二、结构发现：342 处 `complete = true` 的分类

`[Observed]` 全库 326 个 `.java`，342 处 `complete = true`。按**「加预算能否绕开」**切分——这是回答「为什么算不出」的中轴：

| 语义 | 处数 | 占比 | 加预算能否绕开 | 分支自身留痕 |
|:--|--:|--:|:--|--:|
| **ADMIT_STATIC** | 13 | 3.8% | ❌ **绕不开** | 2/13 |
| **ADMIT_MIXED** | 20 | 5.8% | 🔶 部分（规模与配额合流同一条件） | 2/20 |
| ALLOWANCE | 14 | 4.1% | ✅ 能 | 1/14 |
| RESERVE | 54 | 15.8% | ✅ 能 | 2/54 |
| PRECISION | 7 | 2.0% | — 浮点/精度守卫 | 0/7 |
| DOMAIN | 19 | 5.6% | — 输入侧 | 1/19 |
| FINISH | 24 | 7.0% | — 正常收敛 | 8/24 |
| BARE_EXIT | 73 | 21.3% | — 无条件出口（无支配条件） | 0/73 |
| UNKNOWN | 118 | 34.5% | ? 待人工复核 | 19/118 |

**留痕总账：拒绝分支体自身调用 `budget.note` 的只有 35/342（10.2%）。**

> 这个 10.2% 是上一轮「成功有日志、被限流没日志」的量化版。上游对 **UNKNOWN** 有极严格的纪律（`public-input-contracts.txt:36-39`），对**准入拒绝**没有——而准入拒绝恰恰不产生 UNKNOWN。

### 2.1 沉默的硬闸（ADMIT_STATIC 且分支体无 note）—— 11 处

加预算也绕不开、且不留痕的，是真「黑洞」：

| 位置 | 类 · 成员 | 条件 |
|:--|:--|:--|
| `CountAffineLattice.java:102` | `step` | `lower.length > 48 && basis.size() > 48` |
| `CountAffineLattice.java:266` | `prepare` | `lower.length > 48 && (!exactWeighted \|\| free - exactEquations > 48)` |
| `CountBinPackingSearch.java:50` | 构造器 | `entries > 32768` |
| `CountChainSchedule.java:178` | `step` | `!progress \|\| ++rounds == 32`（**轮数上限，非规模**） |
| `CountDecisionDiagram.java:64` | 构造器 | `n > 12 && widths 任一 > 8` |
| `CountDiophantine.java:247` | `prepareFaces` | `other.size() > 1 && !shape.equals(other)`（**形状不匹配**） |
| `CountDomainSearch.java:48` | 构造器 | `lower.length > 1024 \|\| rows.size() > 4096` |
| `CountPackingLpSearch.java:210` | `checkpoint` | `++frame.iterations > 8192`（**迭代上限**） |
| `CountRangedKernel.java:34` | 构造器 | `low.length != 2 \|\| rows.size() > 768`（**只接二元**） |
| `CountRangedKernel.java:53` | `check` | 系数下标越出 0/1 或位长 > 见源码 |
| `PlanSourceCycles.java:244` | `summary` | `pass < 16`（**拓扑收敛轮数**） |

注意其中四处不是「规模太大」，而是**结构不匹配**：`low.length != 2`（只接二元）、`widths 任一 > 8`、形状不等、系数下标越界。这些拒绝在语义上是对的（该子求解器本来就只处理这一类），问题在于**拒绝之后没有信号**，调用方无从区分「不属于我的活」与「我算完了」。

### 2.2 ADMIT_MIXED —— 20 处需要逐点拆的

规模阈值与 `allowance` 合流到同一个 `if`，任一成立就 `complete = true`，**日志里分不出是哪个触发**。全表见 `scan_complete.py` 输出；典型两个：

```java
// CountObbt.java:29
if (lower.length > 64 || rows.size() > 256 || allowance < 4096) { complete = true; return; }
// CountCdcl.java:159
if (lower.length > 1024 || rows.size() > 4096 || terms > 65536 || allowance < 1024) { complete = true; }
```

---

## 三、动态发现一：请求预算在顶层就被切走 1/8，且这是唯一绑定点 `[Observed]`

### 3.1 定位

`CountLcg.java:405`（`lcg-first` 引擎的入口）：

```java
allowance = Math.min(maximumWork, budget.remainingWork() / 8);
```

`[Observed]` 三档预算实测（案例 `0692709435ad`，oracle=SAT，22 变量 2 行）：

| 传入 `--work` | `remainingWork()/8` | 实测截停 work |
|--:|--:|--:|
| 20,000,000 | 2,500,000 | **2,499,999** |
| 160,000,000 | 20,000,000 | **20,000,000** |
| 400,000,000 | 50,000,000 | **50,000,000** |

三档全部严丝合缝落在 `remainingWork()/8` 上，**不是**落在请求预算上。也就是说：**用户传的 work 上限，在这一层永远只兑现 1/8**；除非 `maximumWork < remaining/8`（`Math.min` 的另一支），而 `lcg-first` 恰好把 `maximumWork` 传成了同一个 work，于是 `remaining/8` 恒为绑定项。

### 3.2 与上游自记录对照

`tests/known-limits.json` 的 87 条 model-corpus UNRESOLVED，`detail` 全部形如 `work=2500001`，其 `budget.work` 全部是 `20000000`。`[Observed]` 中位数 **2,500,001**，与 `20,000,000 / 8 = 2,500,000` 完全一致。

> 结论：那 87 条「算不出」的共同特征不是「模型太难」，而是**同一个除法**。这是可归因的工程问题，不是算法极限。

### 3.3 「切了就不回来」

`CountLcg.resume(quantum)` 重开时重新算 allowance（`CountLcg.java:1859-1874`），但每轮 quantum 传的是 `Math.min(262144, remainingWork())`——即**续跑粒度 256K，不再重新享受 1/8**。所以 `lcg-retained` 能一路推进到顶（实测把 `0692709435ad` 从 2.5M 推到了 20M 仍未解），而 `lcg-first` 停在 1/8。

这条解释了一个现象：`[Observed]` 同一个案例 `lcg-first` 停在 2.5M，`lcg-retained` 能跑到 20M 上限。**两者不是同一个搜索深度**，所以「换引擎对比」在口径上不成立，除非同时报告 allowance。

### 3.4 全库除数分布 `[Observed]`

| 除数 N | 处数 | 60M 预算下可见额度 |
|--:|--:|--:|
| 2 | 35 | 30,000,000 |
| 4 | 46 | 15,000,000 |
| **8** | **49** | **7,500,000** |
| 16 | 55 | 3,750,000 |
| 32 | 39 | 1,875,000 |
| 其他（3/5/6/64/128） | 22 | — |

246 处配额派生点，**没有任何一处留下「我切了多少」的 note**。

---

## 四、动态发现二：CountObbt 对 52/274 例静默弃权 `[Observed]`

`AuditGate` 扫全部 274 个 count-model 用例（构造级，不跑搜索，20M 预算）：

| 引擎 | 结果 |
|:--|:--|
| `CountDomainSearch` | 拒绝 5 例（全部是 1690–2205 变量的大模型）· **降级 40 例**（域宽 >32 或编码变量 >1024，转投 `CountLcg`） |
| **`CountObbt`** | **拒绝 52 例** |
| `CountSoftSearch` | 拒绝 8 例（全部 548 变量，`low.length > 512`） |
| `CountLcg` | 拒绝 0 例 |

`CountObbt` 的 52 例拒绝全部来自 `CountObbt.java:29` 那条 ADMIT_MIXED，且**两个轴都在打**：

| 案例 | 变量 | 行数 | 触发的轴 |
|:--|--:|--:|:--|
| `0442c547228f` | 156 | 962 | `rows > 256` |
| `30768487f8c2` | 89 | 28 | `lower.length > 64` |
| `a9d83be7942d` | 1699 | 1670 | 两条都超 |

`CountObbt` 是**优化式界收紧**（OBBT）——它在整条求解链里负责收窄变量域。它弃权时，下游 `CountLcg` 拿到的是一个**没有被收紧过的模型**。这不会让答案错，但直接解释了「同一张图有时快有时慢」：**收紧器是否上场，取决于 64/256 这两个与被收紧对象无关的常数**，而这个决定**全程无声**——`CountObbt` 的 `complete` 只被它自己的 `step()` 读（`if (complete) return true;`，`:48`），父层看不到。

---

## 五、静态发现补充（上一轮未覆盖的）

| # | 发现 | 位置 | 性质 |
|:--:|:--|:--|:--|
| 1 | **`CountContinuation` 接口没有「为何结束」的读取口**——方法面是 `step / paused / resume / counts / matches / work / progress / close`，`complete` 不在接口上；342 处拒绝点写的 `complete` 是各实现类的**私有字段**，外部只能通过 `Count*` 类自己暴露的访问器读到，而这类访问器全库只有 3 处（`CountLpCoreSearch:204` / `CountLpPriceQueue:28` / `CountLpProximity:557`，都不在本次审计的引擎上） | `CountContinuation.java:10-28` | **这是 §二 的根因**——不是「忘了写日志」，是**契约层没有承载归因的位置** |
| 2 | `CountModelReplay` 注释声明「A local admission refusal stays UNKNOWN.」，但 `Result` 记录只有 `counts` + `infeasible` 两个字段 | `tests/corpus/java/CountModelReplay.java:10,17-22` | 声明比实现强——与上一轮 §三 `trial` 死字段同族 |
| 3 | `PlanningBudget.note` 是**32 条环形缓冲**，超出即淘汰 | `PlanningBudget.java:394-399` | 即使补了 note，长时间搜索也会把早期的准入拒绝挤掉；`--model-trace` 只在退出时一次性 dump |
| 4 | `CountLatticeEnumeration` 7 处 `!Double.isFinite(...)` 精度守卫，**全部无声**（7/7 无 note） | `CountLatticeEnumeration.java:87-138` | 单类占 PRECISION 全部 7 处；与 `solver-throughput.txt:26-27`「计数不得转浮点」的口径并存，属自认代价但零观测 |
| 5 | `CountRangedKernel` 只接 `low.length == 2` | `CountRangedKernel.java:34` | 二元的专用后端；不匹配时静默弃权 |

---

## 六、诚实边界

- **本轮不改上游代码**，所有产物在 `.local/audit/`；`.gitignore` 已有 `.local/` 排除项，未修改 `.gitignore`
- **探针读的是 `build/tests/classes` 里的既有字节**——上游源码变更后须重编，否则结论过期
- 342 处分类里 **UNKNOWN 旧记 118 处（34.5%）**：支配条件没抓到（跨行三元表达式、lambda 内嵌、字段初始化器内嵌）。这部分要靠人工复核，脚本没资格替它定性。✅ **已定性（2026-11-04）**——现行版复扫口径 UNKNOWN 140 / 342（`BARE_EXIT` 单列 75，总数 342 不变），逐站点归八轴覆盖 100% → `.local/audit/out/unknown-classes.md`
- `ADMIT_STATIC` 的 13 处里，`CountChainSchedule:178`（轮数 32）、`CountPackingLpSearch:210`（迭代 8192）、`PlanSourceCycles:244`（拓扑 16 轮）三处是**收敛轮数上限**，不是规模阈值——语义上更接近「算法到顶」，分类边界我按字面判据归入 ADMIT_STATIC，实际性质需上游确认
- `CountObbt` 弃权是否真的导致变慢，**本轮只证明了弃权发生，没有证明其性能影响**——那需要 A/B：强制 Obbt 上场 vs 弃权，同案例对比搜索步数。这是下一轮的活
- 87 条 known-limits 的 `work=2500001` 与 `20,000,000/8` 的对应是**数值吻合 + 代码定位**双重成立，但我**没有**逐条重跑这 87 例确认——只跑了其中 1 例（`0692709435ad`）在 20M 下复现 2,499,999
- 阈值「64 / 256 / 8192」这些常数本身是否合理，**本轮不评价**——那是上游的领域判断，我只报「它存在、它无声、它在打」

---

## 七、下一轮靶点（按「能出实证」排序）

| 优先级 | 靶点 | 判据（怎么算做完） | 用的脚手架 |
|:--:|:--|:--|:--|
| P1 | **Obbt 弃权的性能影响 A/B** —— 同一批 52 个弃权案例，一次走原路径、一次强制 Obbt 上场，比搜索步数与是否解出 | 给出「弃权 vs 上场」的步数差与解出率差 | `AuditGate` + `--work` 调参 |
| P2 | **补 `budget.note` 的最小改动面** —— 只在 13 处 ADMIT_STATIC + 20 处 ADMIT_MIXED 上补，测回归是否全绿 ✅ **已落地（M1，2026-11-02）**：30 处补 `admission_refused`，四轴枚举；`PlanSourceCycles:244` 实测为局部变量误判，剔除 → 硬闸 10 处 + 合流 20 处。core 139 项 PASS · ABI 1502 条 PASS · 构造级 96 例归因 100% → `m1-admission.md` | 标准套件 + models 套件仍全绿，且 `--model-trace` 能区分四个拒绝轴 | `scan_complete.py` 输出直接当改动清单 |
| P3 | **`CountContinuation` 契约扩展** —— 加 `refusal()` 返回枚举，让父层能分「拒绝 / 耗尽 / 收敛」 | 接口改完后，`CountModelReplay` 的 `Result` 能带第三态 | 需先有 P2 的分类 |
| P4 | **UNKNOWN 人工定性** | 分类覆盖率 ≥ 95% ✅ **已落地（2026-11-04）**：现行口径 140 处归八轴覆盖 100%——FINISH 82 / ADMIT_STATIC 25 / DOMAIN 14 / ADMIT_MIXED 7 / ALLOWANCE 6 / RESERVE 5 / PRECISION 1；68 处 FINISH 为条件文本判据 + 3 处「算法放弃」子族（无 note，喂 M1c）→ `.local/audit/out/unknown-classes.md` | `scan_complete.py` + `list_unknown.py` + 人工读码 |
| P5 | **allowance 除数表复核** —— 246 处里哪些是「合理的分层预算」、哪些是「无意切走」 | 每处一句判据；给出可用额度排行 | `scan_allowance.py` |
| P6 | ~~上一轮遗留：13 处 `infeasible` 证书门核完剩 8 处~~ ✅ **已落地（2026-11-04）**：13/13 全表填满（8 类补核 + 辅助 2 类）——✅ 11 · 🟡 2（`CountDecisionDiagram` 证据层级低一档 / `CountReducedCost:121` 无证明复核）→ `analysis-solver.md` §一「补核」 | 13/13 全表填满 | 读码 |
