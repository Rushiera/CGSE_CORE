# CGSE_CORE — 上游 docs 精读（第三轮：文档面）

> 触发：Rushiera「继续精读文档」
> 范围：`workspace:tools/CGSE_CORE/docs/` 73 篇 `.txt`——① 建主题族导航（73/73 全覆盖）② 精读与前两轮发现相关的篇目 ③ 核对上游文档是否已解释或修复 `analysis-solver.md` §二 / §三 / §四
> 🔴 **后续**：73 篇已于同日**全部逐字读完**，检索面改由 `docs-index.md`（逐篇卡 + 反向关键词表 + 缺口清单）承载；本文件保留主题族导航与九条精读结论。
> 证据分级：`[Observed]` = 实读原文 / 实跑 · `[Checked]` = 怀疑过、查证不成立 · `[Inferred]` = 读码推断未跑
> 基线：`main` @ `b7c4a81`（v0.3.2）· 文档实体在 `workspace:tools/CGSE_CORE/docs/`

---

## 〇、这一轮改变了什么

前两轮把问题定位在**代码结构**（准入闸无声 / trial 死字段 / 浮点排序）。本轮读文档面，结论是**需要修正其中两条定位**——上游知道这些，并且在别处立了原则；问题不在「他们没意识」，而在**原则落在契约层、没有落在实现层**。

| 前两轮结论 | 本轮文档面核对结果 |
|:--|:--|
| §二「准入拒绝没有任何信号」定性为先例缺陷 | **半修正**——原则已在上游文档里立了至少 5 处（§二.1 逐条给位置），且明写这些终端失败**不得**被当作不可行。真正的缺口是：原则写在文档里，实现里没有承载位（`CountContinuation` 接口无归因口）。性质从「设计疏漏」改成「**声明在文档、契约层缺位**」 |
| §三 `trial` 死字段 = 声明比实现强 | **确认**，且找到旁证：`CountQuickSolve.trial` 有真实赋值（`:74` `solving.trial = true`），`CountDomainSearch.trial` 三个构造器全无赋值 → 同名不同命，全局保护存在但这一份没接线 |
| §四 `CountDemandProjection.repair` 浮点 NaN | **确认并已修复（2026-11-02，M6）**——上游有同构先例 `full-regression-20261011.txt:10-14`（修 CountJump 的「两个大负收益舍成同一 double」用的正是「先减精确残差再转换」）。已照此改 `repair()`，实测改前按遍历顺序、改后按精确残差 → `m6-baseline.md` |
| scaffold §三「1/8 是被无意切走」 | **修正**——`count-corpus-followup.txt:9-11` 明写这是「one CountLcg local allowance, not through the whole integer-count portfolio」，即**有意的本地额度**。可争论的点收窄为「固定分母 8 而非按剩余额度分档」 |

---

## 一、73 篇主题族导航（全量）

> 归族依据：文件首行标题 + 与源码族的对应关系。`★` = 本轮精读；`☆` = 部分精读（节选）；其余 = 标题级已确认。

### A. 契约与架构（4）

| 文档 | 主题 |
|:--|:--|
| ★ `architecture.txt` | Maintenance boundaries——可移植核心 / view / 适配层的维护归属 |
| ★ `public-input-contracts.txt` | 输入与结果契约——ID 唯一 / 快照只读 / `push()` 非 null → IN_DOUBT / **预算 ≠ 不可行** |
| `artifact-identity.txt` | 构件身份（构建身份摘要覆盖哪些路径） |
| ★ `migration-validation.txt` | 可移植边界（`java.base` + `java.management`）与迁移校验基线 |

### B. 求解器总览与调度（9）

| 文档 | 主题 |
|:--|:--|
| ★ `continuations.txt` | 续跑审计——哪些本地限额**故意**保持终端、哪些可续；DD 节点载荷 1/8 字节配额 |
| ★ `retained-progress-scheduling.txt` | 保留进度调度——LP / 视图两类的优先配额与停滞窗口（半额 / 7/8 上限） |
| `effort-and-rational-lp.txt` | 续跑调度与精确 LP 提议的复核跟进 |
| `numeric-progress-and-grants.txt` | 数值进度认定与保留 owner 授权 |
| `decision-queue-and-publication.txt` | 保留布尔选择与冲突发布 |
| `explicit-plan-reuse.txt` | 完整计划复用 |
| ★ `pipeline-cost-attribution.txt` | 成本归因矩阵（11 origin × 9 stage）与终端发布 |
| ★ `request-work-clocks.txt` | **请求局部 worker 时钟**——跨请求浮点残留污染切片边界（已修） |
| `solver-throughput.txt` | 固定 work 下的吞吐（算术实现优化；明写「计数不转浮点」） |

### C. 预算、准入与限额（7）——本轮重点

| 文档 | 主题 |
|:--|:--|
| ★ `audit-followup.txt` | 五项自认缺口（8192 资源准入 / 数值可达 / J 归因 / H 驻留 / 集成与旧限） |
| ★ `hall-support-admission.txt` | Hall 分析在真实布尔支撑上的准入（ambient 8192/32768、matching 512/2048） |
| ★ `stateful-inputs.txt` | 有状态输入与物理返还（`min(4096, remaining/32)`、8 轮上限、1/16 生命期） |
| ★ `root-execution-reachability.txt` | 根执行可达性（两条 `min(…, remaining/N)` 配额式） |
| `structural-completion-service.txt` | 已选结构机会的补完服务（262144 批次） |
| ★ `count-corpus-followup.txt` | **语料跟进——明确 87 条 unresolved 只过了一个 CountLcg 本地额度** |
| `source-refinement-and-dispatch.txt` | 源修复、本地计数准入与协作式分派 |

### D. 数值与格（11）

`affine-gram-admission` · `integral-lattice-factors` · `sparse-affine-basis` · `centered-subset-lattice` · `lattice-box-enumeration` · `lattice-potential-refinement` · `quarter-sum-reuse` · `parity-and-root-continuation` · `projected-startup-proofs` · ★ `numeric-continuation-and-root-proofs` · `cold-integer-lp-continuation`

> 共同结论（`numeric-continuation…:39-40`）：「Precision refusal, admission failure and natural trial face limits remain terminal candidate failures. None proves infeasibility.」48 坐标格准入与精度限额未放宽。

### E. 整数 LP / 剪枝 / 学习（9）

`integer-lp-and-row-templates` ★ · `retained-integer-lp-portfolio` · `retained-finite-hints` · `exact-row-activities` · `integer-conflict-minimization` · `incremental-rounding-locks` · `count-row-overlap` · `count-group-transfers` · `row-template-concurrency`

### F. 打包（packing）族（8）

| 文档 | 主题 |
|:--|:--|
| `packing-phase-reuse` | 枚举/PB 相位的保留与返还 |
| `packing-conflict-reuse` | 枚举恢复时的限定冲突复用 |
| `packing-lp-reuse` / `packing-lp-continuation` | 请求内 LP 复用 / 修复后的保留 LP 候选搜索 |
| `packing-priced-propagation` | 保留 LP 内的精确价格传播 |
| `fractional-packing-neighborhood` / `fractional-packing-rounding` | 分数打包邻域 / 完成 |
| `asymmetric-packing-exchange` | 互补非对称打包交换 |

### G. 等式与匹配（8）

`equality-sparsification` ★ · `equality-message-search` · `equality-word-support` · `sparse-equality-support` · `scalar-count-matching` · `residue-count-matching` · `complement-count-matching` · `finite-integer-propagation`

### H. 向量分配 / 源-燃料（5）

`deferred-vector-allocations` · `vector-allocation-admission` · `vector-candidate-reuse` · `vector-neighborhood-repair` · `refill-joint-sources`

### I. 宿主捕获与状态（7）

`captured-candidate-domains` ☆ · `captured-host-integration` · `captured-result-handoff` · `missing-material-expansion` · `funded-cycle-entries` · `seed-preservation-and-loop-growth` · ★ `state-review-followup` ☆

### J. 回归与复核记录（5）

| 文档 | 数字基线 |
|:--|:--|
| ★ `full-regression-20261011.txt` | 20653 manifest：19642 PASS / 1011 UNRESOLVED / 0 FAIL；count 822 行：790 PASS / 14 UNRESOLVED / 18 UNSUPPORTED |
| `regression-0.1.5.txt` | 0.1.5 全量 20,927 记录：20765 PASS / 156 unresolved / 6 unsupported |
| `review-0.1.5-followup.txt` | 0.1.5 复核跟进 |
| ★ `historical-hard-families.txt` | **canonical backlog 结构**：156 = 150 布尔打包 + 4 稀疏等式 + 2 源/燃料分配（+ seed31） |
| `releases.txt` | 发布链与 mods 落点 |

---

## 二、精读结论（九条）

### 2.1 「准入拒绝 ≠ 不可行」原则已立，且立了五处 `[Observed]`

| 文档:行 | 原文要点 |
|:--|:--|
| `continuations.txt:87` | 「Precision refusal, failed admission and natural face limits remain terminal」 |
| `numeric-continuation-and-root-proofs.txt:39-40` | 「…remain terminal candidate failures. **None proves infeasibility.**」 |
| `retained-progress-scheduling.txt:111` | 「Local admission refusals remain UNKNOWN, not UNSUPPORTED conversions or UNSAT.」 |
| `hall-support-admission.txt` | 「rejected admission … 不证明可行性/不可行性」 |
| `captured-candidate-domains.txt:187-189` | 「Admission refusal or a local cutoff never means infeasible.」 |

**判读：** 语义层面上游**完全一致**地守住了这条纪律。所以 `analysis-solver.md` §二 的定性要改：这不是「他们没意识到」，是「**原则的落点在文档与调用方纪律，不在实现载体**」——`CountContinuation` 接口（`CountContinuation.java:10-28`）没有承载拒绝原因的位置，`complete` 只是各实现类私有字段。

> 这对齐 `Wisdom §1.20`「落点」：规则落在读得到的文档里，执行那一刻手里只有 `boolean complete`。

### 2.2 `CountObbt` 的沉默位置精确化 `[Observed]`

`CountObbt.java` 实际有 `budget.note`（`:131` `finish()` 内 `count_obbt` + `bounds/witness/infeasible/work`）——**但只在正常收敛路径**。四个准入/资源出口（`:28-31` 规模三轴、`:34-37` `tryReserve` 失败）在 `finish()` 之前 `complete = true; return;`，**note 永不发生**。

所以 scaffold 的说法要收敛为一句更准的话：**`note` 机制存在且被使用，沉默发生在「被拒绝时提前 return」这一类出口**——不是「没写日志」，是「写日志的钩子挂在完成路径上」。

### 2.3 `trial` 死字段确认，且找到对照物 `[Observed]`

| 类 | trial 赋值 | 保护生效 |
|:--|:--|:--|
| `CountQuickSolve` | `:74` `solving.trial = true`（`sourceFace()` 造试算面）+ `:123` | ✅ `infeasible = impossible && !trial` |
| `CountDomainSearch` | **三个构造器均无**（`:32-38` / `:37-...` 全无 `this.trial`） | ❌ `!trial` 恒 true |

`CountSoftSearch:133` 的 `trial` 是**它自己**的局部变量（`trial = best == null ? upper : …`），与 `CountDomainSearch.trial` 同名不同物。`CountSoftSearch:139-142` / `CountQuickSolve:300-335` 构造 `CountDomainSearch` 时都不传 trial。

**结论不变，但归因更准：** 上游在 `CountQuickSolve` 上做对了这件事，`CountDomainSearch` 漏了——是**单点遗漏**，不是普遍认知缺失。

### 2.4 `CountJump` 的浮点修法 = §四 的现成先例 `[Observed]`

`full-regression-20261011.txt:10-14`：

> 「CountJump could select the integer immediately below a feasible breakpoint when two large negative gains rounded to the same double. **Compare tied moves by subtracting their exact residual violations before conversion.** This only changes candidate ranking, never proof arithmetic.」

这正是 `analysis-solver.md` §四 给 `CountDemandProjection.repair` 的 P2 建议（「对 Infinity/NaN 给确定性兜底」）的上游版本——**先做精确残差相减，再决定平局**。

`CountJump.java:329-345` 实读：`score == scores[variable] && compareMoves(...) < 0` 走精确比较，`compareMoves` 内部 `leftViolation` / `rightViolation` 全 `BigInteger`。

**执行含义：** §四 的改造不需要发明方案，照 `CountJump` 的结构改 `repair()` 的平局判定即可——与上游已接受的修法同构，评审阻力最小。

### 2.5 1/8 是「有意的本地额度」，但分母固定 `[Observed]`

`count-corpus-followup.txt:9-11`：

> 「Its 87 unresolved count models were **tested through one CountLcg local allowance**, not through the whole integer-count portfolio. That allowance can end well before the request's 20M work cap.」

**判读修正：** scaffold §三 把 1/8 说成「顶层被切走」——准确说是「**单引擎本地额度**」，且上游知道它会远早于请求上限结束。可争的点收窄为：247 处配额派生几乎全是 `remainingWork()/N` 的**固定 N**，而文档侧多处给的是 `min(cap, remaining/N)` 的**双限式**（`equality-sparsification:23-32` / `stateful-inputs:154-171` / `root-execution-reachability:14-54`）。**固定分母 vs 双限式**才是可提的结构问题。

### 2.6 剩余未解有完整分类账，不是「随机算不出」`[Observed]`

`historical-hard-families.txt:1-30`（0.1.5 全量 20,927 记录）：

```
156 unresolved = 150 布尔多维资源/收益选择
              + 4 稀疏整数等式（144 或 257 变量）
              + 2 源/燃料分配（96 或 128 组 / 9 维）
（+1 直接分配 seed31；72 长磨损变体在别的增量中已全解）
```

且 150 条布尔打包的**终止相位**也被归类：57 请求上限（PB 冲突修复中）· 56 本地上限（距离修复中）· 34 请求上限（距离修复中）· 9 请求上限（普通枚举）。

**判读：** 这说明「求解器有点问题」的玩家体感，在上游眼里是**四条机制族的容量边界**，每条有相位归因。我们要动的地方（准入闸归因 / 配额分档）与这四条族的关系是：**归因缺口让这四条族无法被外部观测**，而不是让它们消失。

### 2.7 `known-limits.json` 是历史快照，不可当现行失败账 `[Observed]`

`audit-followup.txt:172-174`：

> 「known-limits.json is explicitly an e788d3d3 observation; its old unresolved entries must not all be called current failures.」

`count-corpus-followup.txt:8-12` 补：那个观测走的是单引擎本地额度；同一批在当前提交上重放是 194 conclusions / 80 unresolved（原 87 条里有 7 条早已能解）。

**对 `analysis-solver.md` §六 的影响：** 上一轮把「87 条 work=2500001」当现行失败账来统计——口径要标注为「历史快照 + 单引擎额度」，不能直接当「当前求解器有 87 个失败」。

### 2.8 上游对「保留进度」有明确的分级判据 `[Observed]`

`retained-progress-scheduling.txt:78-95` 区分三种阶段：**productive retained work**（有新精确后果）· **cheap complementary first visit**（必须先付的强制首次访问）· **completion/proof**（真完成）。并明写：「turn counts or the existence of a conflict alone cannot identify those stages.」

`structural-completion-service.txt:20-35` 给了实现：初始编译样本后，若「已选结构机会仍在」，按 262144 批次续同一组合；「An idle feedback interval within one of those opportunities no longer yields the portfolio as wholly stalled.」

**判读：** 这是上游调度侧最成熟的一块，也是我们 P1/P2 建议（拒绝语义可归因）最容易与之对齐的落点——把 `refusal()` 枚举对齐到这三阶段，父层才可能做出与上游调度哲学一致的判断。

### 2.9 「测试全绿」不覆盖语义目标 `[Observed]`

`full-regression-20261011.txt:47-60` 同时给了：205 文件指纹一致 · standard 83 项 PASS · 244×3 fixtures 全绿 · 20653 manifest 零 FAIL。`audit-followup.txt:182-183` 紧接着补：「Historical probe assertions have not all been freshly replayed. Passing graph fixtures does not certify every historical integer model.」

同族 `integer-lp-and-row-templates.txt` 结尾也反复强调「These are standalone solver tests, not live Minecraft/AE validation」。

---

## 三、对前两轮报告的修订清单

> 只列需要动字的地方，不重写报告。

| 报告 | 位置 | 修订 |
|:--|:--|:--|
| `analysis-solver.md` | §二「主发现」 | 定性从「没有信号 / 没意识到」改为「**原则已立五处，缺实现承载位**」；补 `CountContinuation` 接口无归因口这一根因 |
| `analysis-solver.md` | §四 | 补一条「上游已有同构修法先例」（`CountJump`，`full-regression-20261011:10-14`）——建议从「可选」升为「照先例做」 |
| `analysis-solver.md` | §六 基线 | 「87 条 UNRESOLVED」标注为**历史快照（e788d3d3）+ 单引擎额度**，不作现行失败账 |
| `analysis-scaffold.md` | §三「1/8 切走」 | 改为「**有意的单引擎本地额度**，固定分母」；悬念收窄为「固定 N vs 文档侧双限式 `min(cap, remaining/N)` 的不一致」 |
| `analysis-scaffold.md` | §2.1「11 处沉默硬闸」 | 补准确性说明：`note` 机制存在，沉默发生在**提前 return 的拒绝路径**（`CountObbt:28-37` 为实例） |

---

## 四、下一轮靶点（按「能出实证」重排）

| 优先级 | 靶点 | 判据 |
|:--:|:--|:--|
| P1 | ~~**`CountJump` 先例移植到 `CountDemandProjection.repair`**~~ ✅ **已落地（2026-11-02，M6）** —— 平局判定先做精确残差相减，去掉 NaN 退化路径 | 构造 `after/gain` 双 Infinity 用例，改前按遍历顺序、改后按精确权重；标准套件 + models 全绿 —— **实跑兑现**：改前 `picked=variable 0`（顺序），改后 `picked=variable 1`（精确）；标准套件 411/412（第 412 项为工具窗口）、models 274 例严格相同 → `m6-baseline.md` |
| P2 | **固定分母 vs 双限式**——247 处 `remainingWork()/N` 中，哪些有文档侧的 `min(cap, …)` 对应项、哪些是裸固定分母 | 逐处一句判据；给出「裸固定分母」清单 |
| P3 | **`CountContinuation` 加 `refusal()` 枚举**，对齐上游三阶段（productive / first-visit / completion） | 13 处 ADMIT_STATIC + 20 处 ADMIT_MIXED 能报出具名原因；`--model-trace` 可区分四轴 |
| P4 | 13 处 `infeasible = true` 证书门核完剩 8 处 | 13/13 填满判定 |
| P5 | `CountDomainSearch.trial` 接线（或删字段改注释） | 二选一落地，测试覆盖 |

---

## 五、诚实边界

> 🔴 **本节已被文档面第二轮更新**（73/73 逐字读毕，见 `CHANGELOG.md` v0.4 第二条 + `docs-index.md`）。以下保留首轮原貌，覆盖度一条已过期。

- ~~精读覆盖：全文 10 篇 · 节选 9 篇 · 标题级 54 篇~~ → **已更新为 73/73 逐字读毕**（三篇超 `text-read` 20000 字符上限的分段补读）
- 本轮**未跑任何测试**，全部结论来自文档与源码互读；`analysis-solver` / `analysis-scaffold` 的跑测基线仍是上一轮的
- ~~`state-review-followup.txt` 只读到前 200 行 + 尾部，中段未读~~ → 已全文读完（753 行）
- `CountObbt` 52 例弃权的**性能影响**依然未证（只证了弃权发生）——P1 的 A/B 仍然要做
- 文档日期跨度 2026-10-07 → 2026-10-27，**旧文档的「已修」不代表当前 HEAD 仍成立**（`audit-followup` 自己就说历史断言未全部重放）
- 索引锚点抽查 12 处全部回原文对齐、修正 3 处（`docs-index.md` §〇），其余 60 余处锚点为一次性落笔、未二次校验
