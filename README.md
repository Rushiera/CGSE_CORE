# CGSE — Compiled Graph Solving Engine / 编译图求解引擎

> 本仓库是上游 [autumn-2-net/CGSE_CORE](https://github.com/autumn-2-net/CGSE_CORE) 的**协作 fork**（`Rushiera/CGSE_CORE`）。
> 上游是唯一权威；本 fork 承载本协作方对求解器的诊断与改造，供上游评审。上游原文说明保留在 `README.txt`。

---

## 一、与上游的关系

| 项 | 说明 |
|:--|:--|
| 上游 | `origin` → `https://github.com/autumn-2-net/CGSE_CORE.git`，**只读**（不向它 push，不提 PR） |
| 本仓 | `fork` → `https://github.com/Rushiera/CGSE_CORE.git`，本地 `main` 推这里 |
| 基线 | 上游 `main` @ `b7c4a81`（v0.3.2，2026-10-27）；本仓改动全部基于该提交 |
| 未触碰 | `LICENSE` / `NOTICE` / 宿主适配面（Minecraft 渲染、包、菜单、本地化、`GraphJobCodec` 均在 GTLCore，不在本仓） |
| 许可 | 继承上游 **MPL-2.0**；本仓改动同样按 MPL-2.0 提供 |

**改动性质：** 不给求解语义换算法、不改判定阈值（唯一例外是 M4-G2 一处宽度闸，实测零收益已如实记录）。主线是**让拒绝可归因**——把「没答案也没说为什么」变成「哪条闸、因为什么条件拒绝」。

---

## 二、相对上游的改动清单

改动基线：`b7c4a81`。源码改动 37 个文件（36 个 Java 源 / 测试 + `.gitignore`），净 +457 / −66 行；另加本 README 与 `project-notes/`（27 个文档，相对上游新增）。

### 1. 准入拒绝具名化（M1）——31 处 / 24 个类

原来引擎在准入闸返回 `complete = true` 时不留任何痕迹，上层只看到 `UNKNOWN`。现在统一写：

```text
stage  = admission_refused
detail = owner=<类名> <axis>:<条件> [<axis>:<条件> ...]
```

`axis` 四值可枚举、可 grep、可计数：`size`（规模超限）/ `shape`（结构不匹配）/ `quota`（配额不足）/ `memory`（内存预留失败）。

- 覆盖 `CountAffineLattice`（3）· `CountBinPackingSearch`（2）· `CountRangedKernel`（3）等 24 个引擎类，逐类清单见 `project-notes/m1-admission.md` §三。
- 原 `a || b || c` 短路闸改为逐轴判定后拼接：语义等价（任一成立即拒），但一次报全所有成立轴。
- **判定逻辑、阈值、短路顺序零改动**——纯观测面。
- 实证：274 例构造级探针中，改前 `REFUSED` 三字无理由，改后每例都带 `owner=` + 轴串。

### 2. 第三态 `refusal()`（M1b）——接口扩展 + 父层消费

- `CountContinuation` 新增 `default String refusal() { return ""; }`（空串 = 已准入）。
- 置值实现：`CountBoolean` / `CountDecisionDiagram` / `CountDiophantine` / `CountDomainSearch` / `CountMeetInMiddle` / `CountSeparator`（M1c 补齐）；`CountViewSearch` 汇总 specialist 的第三态（按引擎名去重拼接）。
- 父层消费：`CountModelReplay` 结果加 `refusal` 分量（反射读取，对 pre-channel 基线仍可编译）；`CountModelRegression` 输出 `REFUSED <原因>` 与 `refused=` 计数。
- **退出码语义不变**：`REFUSED` 与 `UNRESOLVED` 同样走 `2`（预算/准入未决，绝不当 PASS，也绝不当 UNSAT）。

### 3. 静默拒绝点齐平 + `trial` 契约接线（M1c）——24 处出口

- 23 处「`complete = true` 既无 note 也无 refusal」的出口补齐 `admission_refused`；另 1 处存量 note 补 `refusal()`（`CountDiophantine` 的 `faces_not_equal`）。
- 「算法放弃」3 处按裸条件口径并入清点（`restore_out_of_domain` / `restore_row_violation` / `components_unresolved`）。
- **`trial` 死字段接线**：`CountDomainSearch` 此前注释承诺「试算面可出 witness，但不得导出负结论与学习子句」，字段从未赋值。现在新增 `trial(boolean)` / `faceInfeasible()`，`CountSoftSearch` / `CountQuickSolve` 按面消费：
  - 试算面学习子句不再入全局池（此前恒泄漏）；
  - 控制流零变化（两处条件消费改读 `faceInfeasible()`，导出值恒等）。
- 实测（TrialProbe）：`bounded_face` 改前 `infeasible=true / trial_field=false / learned_exported=1` → 改后 `false / true / 0`。

### 4. 数值精度（M6）——`CountDemandProjection.repair` 平局判定

- 问题：候选权重 `sum(after / gain)` 在 `Infinity / Infinity` 时得 `NaN`，`<` 与 `==` 同时为 false → 平局失效，退化为遍历顺序依赖，且无信号（与上游「计数不转浮点」口径冲突）。
- 改法：照上游已接受的 `CountJump.compareMoves` 先例——先比 `negative`，`weight` 有限且不等时走原 double 快路径，**相等或 NaN 时回落精确残差**（`BigInteger` 逐行相减 hinge，经 `CountJump.ratio` 转有界比值累加定序）。
- `CountJump.ratio` 由 `private` 放宽为包内可见（复用同一实现，语义未动）。
- 实测：双 Infinity 用例改前 `picked=variable 0` → 改后 `picked=variable 1`（精确残差更小者）；models 274 例逐例严格相同 274/274。

### 5. 闸门口径（M4）——一处放宽 + 一处诊断行

- **G2 宽度闸**：`CountDecisionDiagram` 的 `domain_width > 8` 放到 `> 16`（变量数 > 12 时）。census 显示候选面仅 2 例，**放开后 274 例终态零变化 → 实测收益 0**；改动保留为口径对齐，回滚即改回 `8`。
- **G3 诊断行**：`CountDiophantine` 增加 `multi_shape_conflict; row=...` 具名 note，把「多行不同系数形状」这个共因拒绝点显式化（G3 岔口拍板 c3）。

### 6. 测试与运行环境

- `CountDemandProjectionTest` 新增 `repairTieBreak()`（10 条断言 + 两个控制辅助）。
- `CountModelReplay` / `CountModelRegression` 输出第三态（见 §2）。
- `.gitignore` 增加 `/runtime/`：项目内的 JDK 17 + Gradle 8.8 运行环境，约 500 MB，不入库、不动系统环境变量。

---

## 三、实测结论（含零收益与未决，如实列出）

| 项 | 结果 |
|:--|:--|
| `gradle jar` | BUILD SUCCESSFUL |
| ABI 门禁 `check_abi.py` | **PASS 1502 条**（0.3 基线） |
| core 套件全量 | **385 PASS / 0 FAIL** |
| models 274 例 | 与改动前逐例严格相同（274/274） |
| 标准套件 | 411/412 PASS（第 412 项 `PortfolioOracle` 默认 2000 例约 125 s，单跑通过 `missed=0 unsound=0`） |
| 剩余 UNRESOLVED | **6 例**：count-model 4 例 `generated/sparse-equality-n{144,257}-s{1,2}`（3 例 100M 预算仍无 witness，1 例 33.5M 即解出）+ graph 2 例 `pool_mix7__perm0/perm1`（同一 fixture 的排列变体） |

**已验证为「零收益 / 不可行」的改造方向（不再重复尝试）：**

- packing 族强化：11 个请求 `count_packing_analysis` 零命中，放宽规模闸后 work 38,019,380 **逐值不变**。
- Obbt 弃权（P1 / P1b）：强制上场与入场条件调整均零状态变化。
- 配额上限 ×4：零状态变化（受 `min(4096, …)` 字面量硬帽约束）。
- 窗口 4096→32768：状态 0 变化，work +2.44%。
- G3 c1（分组后跨组耦合）：放宽闸后 274 例终态 0 变化，两例 200M 仍 UNRESOLVED。

**当前主要矛盾（供上游参考）：** 15~17 个无界变量（`high == null`）导致 `CountIntegerLpLearning.bounded()` 判否，LP 支持面只剩 29%；同时被拒的还有 Diophantine（`shape:upper==null`）/ DD / Partition / Boolean / Probing。`count_model: skipped; closure_recipes=8193; local_limit=8192`——**1 个单位卡掉整个模型**（`GraphCompiler.java:331-335`）。

---

## 四、构建与跑测

```text
# 纯库构件（用项目内 runtime，绕开 wrapper 联网）
java -classpath "runtime\gradle-8.8\lib\*" org.gradle.launcher.GradleMain -p <repo> -g runtime\gradle-home jar

# 标准回归
python tests\run.py --java-home <repo>\runtime\jdk-17.0.2 --suite standard --timeout 600

# 全量回归
python tests\run.py --java-home <repo>\runtime\jdk-17.0.2 --suite all --milliseconds 0 --model-engine main-counts --output build/all-tests

# ABI / 边界
python tools\release\check_abi.py
python tools\verify_boundaries.py
```

- **退出码语义**：`0` 全过 · `1` 断言/编译/夹具失败 · **`2` = 预算未决（UNRESOLVED / REFUSED）——绝不可当 PASS，也绝不可当 UNSAT**。
- 上游脚本 `verify_boundaries.py` 在基线上就是红的：它按 `package org.cgse.core;` 判定首行，而源文件首行是版权注释 → 大量假阳性。读输出时先减掉这一类。
- `runtime/` 不在仓库内；没有它时用系统 JDK 17 + Gradle 8.8（`gradlew.bat` 首次会联网下载分发）。

---

## 五、文档导航

### `project-notes/` — 本协作方的完整工作档案（相对上游新增）

| 想了解 | 读什么 |
|:--|:--|
| 项目定位、边界、裁决、构建入口、关键判据 | `project-notes/project.md` |
| 全部已完成工作（26 轮，倒序） | `project-notes/CHANGELOG.md` |
| 未完成项 | `project-notes/todo.md` |
| 上游 73 篇 `docs/` 怎么查 | `project-notes/docs-index.md`（逐篇关键词索引 + 反向表） |
| 求解器静态分析第一轮（13 处证书门） | `project-notes/analysis-solver.md` |
| 结构审计脚手架（342 处 `complete=true` 分类） | `project-notes/analysis-scaffold.md` |
| 性能归因（work 构成 / 84.9% 推进步数税） | `project-notes/analysis-attribution.md` · `analysis-strengthen.md` |
| 各改造项的改动面与前后对照 | `m1-admission.md` · `m1b-third-state.md` · `m1c-silence.md` · `m4-g2-g3.md` · `m4-g3-decision.md` · `m6-baseline.md` |
| 已判死的方向（避免重复投入） | `p1-obbt-ab.md` · `m2-quota-ab.md` · `analysis-model-shape.md` |

> 档案内使用 `workspace:` / `ccbp:` 前缀表示协作方本地工作区的逻辑根，读时当作「仓库根」与「档案目录」即可。

### 上游自己的文档

- `docs/` — 73 篇专题设计文档（契约、预算、数值、LP、packing、回归等），入口先用 `project-notes/docs-index.md`。
- `tests/README.md` — 跑测面全集（suite 表 / 退出码 / 预算语义）。
- `docs/architecture.txt` — 维护边界与入口点契约（改码前首读）。

---

## 六、边界

- 本 fork **不向上游 push、不提 PR**；成果由上游评审决定是否吸收。
- 不修改适配层（GTLCore 在别仓）；不改 `LICENSE` / `NOTICE`。
- 所有「未跑」「未决」「零收益」都按原样记录，不用「应该没问题」替代实测。
