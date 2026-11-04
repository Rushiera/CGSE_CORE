# M1c — 静默拒绝点 note+refusal 齐平 + trial 契约接线（P1）

> 范围：`src/main/java/org/cgse/core/` 24 处静默拒绝出口补 `admission_refused` note（其中实现类 15 处同补 `refusal()` 第三态）+ `CountDomainSearch.trial` 契约接线（`analysis-solver.md` §三 P1）；不改任何判定条件、阈值、配额除数、短路语义
> 基线：`main@b7c4a81`（v0.3.2）+ v0.4 M6 + v0.5 M1 + v0.6 M1b 的未提交工作树（本轮同为未提交——不 push）
> 授权：`project.md` §裁决记录（结构不匹配就改结构，代价后置上游评审）· 约束全守：未 push · 未碰 `LICENSE` / `NOTICE` / 适配面（GTLCore 在别仓）
> 拍板（2026-11-04）：① 条件结论处置 = **B 案 P1 接线**（trial 真传；P2 删字段与 `CountReducedCost:121` CountProof 门均不做）② 「算法放弃」3 处 = **并入 M1c 清点补 note** ③ 下一执行项 = **M1c 先行**
> 口径：静默拒绝点 = 「`complete = true` 既无 note 也无 refusal」（`m1b-third-state.md` §五原文）；分类轴对齐 P4（`.local/audit/out/unknown-classes.md` 八轴 → M1 四轴 + 裸条件）

---

## 一、要解决的问题

M1 把 30 处准入闸写进 note、M1b 给 5 处置 `refusal()`，但 m1b §五明写「静默拒绝点未纳入」——`complete = true` 既无 note 也无 refusal 的出口，父层与日志双盲。本轮按 B 案「齐平」清点补齐这类出口，并把 §三的 `trial` 死字段假契约接线成真契约。

## 二、M1c 改动面（24 处 note + 1 处存量 refusal 齐平）

### 实现类（`CountContinuation`——note + refusal 双通道，refusal 串 = note 的轴串）

| 类 | 出口条件 | 轴串 | 处数 |
|:--|:--|:--|--:|
| `CountDecisionDiagram` | `upper==null` · `width<0 / width>64` · `tryReserve` | `shape:upper==null` · `shape:width<0` / `size:width>64` · `memory:tryReserve` | 3 |
| `CountDiophantine` | `upper==null / lower>upper` · `free<2` · `tryReserve` · `faceChoices<2` · `remaining<0` · `faces_unresolved` | 二择 · `shape:free<2` · `memory:tryReserve` · `shape:faceChoices<2` · `shape:remaining<0` · `faces_unresolved`（裸） | 6 |
| `CountDiophantine`（存量齐平） | `faces_not_equal`（M1 存量 note 无 refusal） | `shape:faces_not_equal` | +1 |
| `CountDomainSearch` | `span<0` · `tryReserve` | `shape:span<0` · `memory:tryReserve` | 2 |
| `CountSeparator` | 混合闸（`variables>256` / `rows>2048` / `allowance<1024` / `tryReserve`） | 拼轴（M1 ADMIT_MIXED 拆轴规则） | 1 |
| `CountMeetInMiddle` | 混合闸（`allowance<1024` / `tryReserve`） | `quota:allowance<1024` / `memory:tryReserve` 二择 | 1 |
| `CountBoolean` | `tryReserve` | `memory:tryReserve` | 1 |

### 非实现类（note 单通道——同 `CountObbt` / `CountSoftSearch` 既有口径）

| 类 | 出口条件 | 轴串 | 处数 |
|:--|:--|:--|--:|
| `CountSoftSearch` | `tryReserve` · `endpoint==null` | `memory:tryReserve` · `shape:endpoint==null` | 2 |
| `CountRangedKernel` | `tryReserve` | `memory:tryReserve` | 1 |
| `CountKernelSearch` | 构造混合闸（`basis<2` / `basis>=variables` / `basis>48` / `variables>128` / `rows>512` / `allowance<2048` / `tryReserve`）· `restore_out_of_domain` · `restore_row_violation` | 拼轴 · 裸条件（算法放弃） | 3 |
| `CountComponents` | 混合闸（`allowance<1024` / `tryReserve`）· `components_unresolved` | 二择 · 裸条件（算法放弃） | 2 |
| `CountChainSchedule` | `reserve()` 助手 `tryReserve` | `memory:tryReserve` | 1 |

- **「算法放弃」3 处**（`CountKernelSearch` `restore_out_of_domain` / `restore_row_violation` · `CountComponents` `components_unresolved`）按拍板并入清点——裸条件口径与 M1 的 `iterations>8192` 先例一致（P4 FINISH 轴子族）
- **混合闸拆轴**按 M1「ADMIT_MIXED 拆轴规则」：纯条件轴先行求值成串、`tryReserve` 只在闸内跑一次——任一成立即拒的语义不变，一次报全所有成立轴（`CountSeparator` / `CountMeetInMiddle` / `CountKernelSearch` / `CountComponents` 四闸逐案短路等价）
- **判定逻辑零改动**：阈值、条件、短路顺序、`complete = true` 出口形态逐处保持——M1「只加观测」口径延续

## 三、trial 契约接线（`analysis-solver.md` §三 P1，B 案）

`CountDomainSearch` 类注释承诺「试算面可出 witness，但不得导出负结论与学习子句」——`trial` 字段此前从未赋值，两条承诺悬空（v0.11 TrialProbe 实锤）。接线：

| 改动 | 内容 |
|:--|:--|
| `CountDomainSearch` | 新增 `trial(boolean)`（fluent，同 `retained()` 风格）与 `faceInfeasible()`；`:157` 拆为 `faceInfeasible = search.infeasible(); infeasible = faceInfeasible && !trial;`——`infeasible()` 只导出非试算面结论，面判定走 `faceInfeasible()` |
| `CountSoftSearch:144` | 构造链 `.trial(best != null)`——`best==null` 时界行恒真（`trial=upper=Σweights`）不构成收紧，不是试算面；`best!=null` 时界行真收紧即试算面 |
| `CountSoftSearch:149` | 条件消费改读 `faceInfeasible()`——incumbent 最优证明（`:121-136` 面判定消费）不受导出屏蔽影响；`:156-158` 导出口不动 |
| `CountQuickSolve:319` | 构造链 `.trial(this.trial)`；`:322` 改读 `faceInfeasible()`——`impossible` 只进 `:360 infeasible = impossible && !trial`，改前后导出值恒等 |
| `TrialProbe`（`.local/ab/probe-src`） | 三用例：`unrestricted` / `bounded_face`（`trial(true)`）/ `bounded_unmarked`（对照 opt-in 语义） |

**消费面影响**：学习子句出口 `if (!trial)` 由恒开变为按面生效——试算面学习子句不再入全局池（类注释承诺；此前恒泄漏）。两处条件消费（`CountSoftSearch` / `CountQuickSolve`）都改读 `faceInfeasible()`，控制流零变化。
**不做**：P2（删 `trial` 字段 + 注释对齐）与 `CountReducedCost:121` 的 CountProof 门——拍板 B 案单选；`:121` 条件结论维持 `analysis-solver.md` §一「现况安全 · 同族隐患」定性。

## 四、观测实证（改前 / 改后）

### TrialProbe（`trial` 契约，同探针同命令）

```text
CASE              COUNTS    INFEASIBLE       TRIAL_FIELD     LEARNED_EXPORTED
unrestricted      [1, 1]    false            false           0
bounded_face      null      true → false     false → true    1 → 0
```

改后新增 `bounded_unmarked` 行（`infeasible=true / trial_field=false / learned_exported=1`）显证 opt-in 语义——未标记者维持原行为。

### AuditGate 274 例构造级（9 片 `count-model-0…256.bin`，work=20000000）

| 口径 | 改前 | 改后 | 差 |
|:--|--:|--:|--:|
| refusedDomain | 5 | 5 | 0 |
| refusedObbt | 52 | 52 | 0 |
| refusedSoft | 8 | 8 | 0 |
| refusedLcg | 0 | 0 | 0 |
| refusedAny | 81 | 81 | 0 |

逐片 SUMMARY 对账（32×8 + 18 = 274）：delegated 40 逐片同、归因条目逐行同改前。**本语料构造级不触达本轮新增出口**（CDD / CDioph / CKS / CC 等新轴 274 例零触发）——新出口的存在性由 M1cProbe 实证（下）。

### M1cProbe（`.local/m1c/probe-src`，新增 6 用例，逐案强制触达新出口）

```text
CASE                REFUSAL                   COMPLETE  NOTE_HIT
cdd_width_negative  shape:width<0             true      true
dio_lower_gt_upper  shape:lower>upper         true      true
cds_span_negative   shape:span<0              true      true
cc_quota            (no-refusal-channel)      true      true
sep_quota           quota:allowance<1024      true      true
ks_basis_lt2        (no-refusal-channel)      true      true
```

6/6：实现类第三态可读（`refusal()`），非实现类按口径只走 note 通道；`budget.diagnostics()` 逐案含具名 `admission_refused` 条目。

## 五、回归

| 项 | 命令 | 结果 |
|:--|:--|:--|
| 编译 | `GradleMain -p . jar --offline` | **BUILD SUCCESSFUL in 17s**（compileJava 全量重编） |
| core 套件 | `python tests\run.py --suite core --shard 0/4…3/4`（首片编译） | **169 PASS · 0 FAIL**（30 / 45 / 48 / 46；对照 M1 基线 139——同口径每片 120 s 截断，为覆盖数非全绿） |
| ABI | `python tools\release\check_abi.py --jar build\libs\cgse-core-0.3.2.jar` | **PASS binary API 0.3: 1502 baseline entries preserved** |

## 六、诚实边界

- **语料未触达新出口**：AuditGate 274 例对本轮 24 处新 note 零触发——判定零变化是设计目标（只加观测），但语料级归因覆盖率未因此提升；新出口的运行时实证在 M1cProbe（6/6）
- **非 `complete=true` 族静默出口未纳入**（口径边界，逐处留档）：工厂 `return null`（`CountGroupedPartition:91` · `CountMessageSearch:71` · `CountPackingCore:75` · `CountRelaxationNeighborhood:40` · `CountVectorNeighborhood:51` · `CountVectorPartition:73`——null/IN_DOUBT 通道，非本通道）· `throw Declined`（`CountGroupedPartition:352/:389`）· `CountKernelSearch:228` `Stop|PrecisionLimit` 混合 catch（预算/精度到顶——「预算耗尽不是不可行证明」，入 `admission_refused` 通道属语义污染，不动）· `CountPackingLpSearch:217` `state<0`（`refusals++` 自带计数但 `primal()` 返回语义未读明，不动待裁）
- **`CountNeighborhood:346-347` 试算面未接线**——其构造带 `objective≤old-1` 界行，属试算面；`infeasible()` 消费面未审计，超出 P1 点名面（`CountSoftSearch` / `CountQuickSolve`），待裁是否补 `.trial(true)`
- **环形缓冲 32 条上限不变**——note 可达但不保证长存（M1 §六同口径）
- **未跑 `--suite all`**（M5 范围）；core 169 为分片覆盖数

## 七、复跑

```text
runtime\jdk-17.0.2\bin\java.exe -classpath "runtime\gradle-8.8\lib\*" org.gradle.launcher.GradleMain -p . -g runtime\gradle-home jar --offline
python tests\run.py --java-home runtime\jdk-17.0.2 --suite core --shard 0/4
python tests\run.py --java-home runtime\jdk-17.0.2 --suite core --no-compile --shard 1/4
python tools\release\check_abi.py --jar build\libs\cgse-core-0.3.2.jar --java runtime\jdk-17.0.2\bin\java.exe
runtime\jdk-17.0.2\bin\javac.exe -encoding UTF-8 -cp build\tests\classes -d .local\ab\probe-classes .local\ab\probe-src\org\cgse\core\TrialProbe.java
runtime\jdk-17.0.2\bin\java.exe -cp "build\tests\classes;.local\ab\probe-classes" org.cgse.core.TrialProbe
runtime\jdk-17.0.2\bin\javac.exe -encoding UTF-8 -cp build\tests\classes -d .local\m1c\probe-classes .local\m1c\probe-src\org\cgse\core\M1cProbe.java
runtime\jdk-17.0.2\bin\java.exe -cp "build\tests\classes;.local\m1c\probe-classes" org.cgse.core.M1cProbe
runtime\jdk-17.0.2\bin\java.exe -cp "build\tests\classes;.local\audit\out\probe-classes" org.cgse.core.AuditGate .local\audit\out\bins\count-model-0.bin 20000000
```

> 顺序依赖同 M1：探针读 `build/tests/classes`，改源码后先跑一次不带 `--no-compile` 的 `run.py`。AuditGate 9 片逐片跑——并发跑会丢片输出（本轮实测 5 并发丢 1 片 SUMMARY，单发复跑即回）。改码回滚备份：`.local/m1c-backup/`（本轮 4 文件）；域内改动未留逐件备份，回滚按 `m1c-silence.md` §二/§三改动面逐条反做。
