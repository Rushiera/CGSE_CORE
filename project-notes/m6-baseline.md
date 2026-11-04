# M6 改动前后对照 —— CountDemandProjection.repair 平局判定

建立：2026-11-02 · 基线 `main@b7c4a8103657572014325b63f8a4110e0337a8a8`（工作树仅 `.gitignore` 一行改动：`/runtime/`）
复跑入口：`.local/audit/probe/org/cgse/core/RepairTieBreakProbe.java` + `.local/audit/snapshot_models.py` / `diff_models.py` / `coverage.py`
字节码对照物：`build/baseline-standard/classes`（**改前**编译产物）· `build/full/classes`（**改后**）——探针靠这两个目录分别复现两种判定，勿删。

## 一、用例：after/gain 双 Infinity

构造（探针内 `build()` 写入私有状态，反射驱动 `repair(0)`）：

| 行 | 内容 | 作用 |
|:--|:--|:--|
| row 0 | `-x0 - x1 <= -1` | 两个候选各一轮（`extra = 1`）都能清零欠账 |
| row 1 | `-x0 - 2^1024-1·x2 <= 0` | 候选 0 一轮后该行欠 `2^1024-1`，其最大负系数同为 `2^1024-1` |
| row 2 | `-x1 - 2^1024-1·x3 <= 0` | 候选 1 一轮后该行欠 `2^1024-2`，增益同量级 |
| debts | `[1, 1+(2^1024-1), 1+(2^1024-2)]` | 行 1 / 行 2 的欠账正好是「候选清零后剩余」 |

`bounded()` 上限 `bitLength <= 1024`，所以 `2^1024-1` 是**可及的最大量级**；两者 `doubleValue()` 均为 `Infinity`，商为 `NaN`。

### 改前（`build/baseline-standard/classes`，24,854 B）

```
  v0 row 1 after.bits=1024 after.double=Infinity gain.double=Infinity term=NaN
v0: negative=1 weight=NaN isNaN=true
  v1 row 2 after.bits=1024 after.double=Infinity gain.double=Infinity term=NaN
v1: negative=1 weight=NaN isNaN=true
picked=variable 0 (first visited, exact residual 2^1024-1)
```

两个候选 `negative` 相同、`weight` 全 `NaN` → `weight < bestWeight` 与 `weight == bestWeight` **同时为 false** → 平局判定完全失效，**先遍历到的赢**。精确残差更大的候选 0 被选中。

### 改后（`build/full/classes`，26,733 B）

```
picked=variable 1 (second visited, exact residual 2^1024-2)
```

按精确残差选中更小者；把两条边的遍历顺序对调，结论不变（`CountDemandProjectionTest.repairTieBreak` 已断言方向与反对称性）。

## 二、标准套件前后

| 面 | 改前 | 改后 |
|:--|:--|:--|
| 标准套件（`--suite standard` 现行口径 = view + core + oracles + 边界批，412 项） | 基线记录 26/26（`CHANGELOG` v0.2）；本次实测改前跑到 27 项全 PASS 后被工具 120 s 窗口截断——**基线以 `CHANGELOG` 记录 + 274 例快照为准** | **411/412 PASS**（分批 + 单点补跑；411 项由合并扫描统计）。第 412 项 `PortfolioOracle` 见下 |
| 目标用例 | `CountDemandProjectionTest` PASS | PASS（新增 tie-break 断言） |
| `CountJump*` 三件 + `ExactRationalRegression` | PASS | PASS（`CountJump.ratio` 仅由 `private` 放宽为包内可见，语义未动） |
| `check_abi.py`（0.3 基线） | 未跑（本轮只在改后跑） | **PASS 1502 条**（改动全在包内可见性面，无公共签名变化） |
| `jar` 构建 | BUILD SUCCESSFUL（`CHANGELOG` v0.2 记录 37s） | **BUILD SUCCESSFUL in 16s** |
| `verify_boundaries.py` | ❌ 对全库源文件报 `unexpected package` | ❌ 同样报错，**与本次改动无关**（见 §五） |

**唯一未收口项 `PortfolioOracle`**：套件按无参调用 = 2000 例抽样，需约 125 s，超出单次工具调用 120 s 上限（改前同样在 1900/2000 处被同一窗口截断，两轮截断点一致）。改后按 1500 例跑通：
`ORACLE total=1500 reachable=647 missed=0 unsound=0`，且逐百例进度与被截断的改前 2000 例跑测**前 1500 行逐行相同**——语义结果未变，未收口的只是墙钟窗口。另单跑 200 例全量收口（`reachable=90 missed=0 unsound=0`）。

## 三、models 274 例快照

| 面 | 改前 | 改后 |
|:--|:--|:--|
| 逐例 status / solution / work | PASS 200 · UNRESOLVED 74 | PASS 200 · UNRESOLVED 74 |
| 严格相同（三项全等） | —— | **274 / 274** |
| 语义相同（status + solution） | —— | **274 / 274** |

口径：`--suite models --milliseconds 0 --work 60000000 --jobs 4`（`milliseconds=0` 去掉墙钟截停，结果可复现）。
快照脚本 `snapshot_models.py` → `baseline-models-det.json` / `after-models.json`；比对 `diff_models.py`。

## 四、改动面

| 文件 | 变更 |
|:--|:--|
| `src/main/java/org/cgse/core/CountDemandProjection.java` | `repair()` 改为构造 `Repair` 候选 + `compareRepairs` 比较；新增 `Repair` record、`ResidualProfile`、`compareResidualSums`、`repairGain`（`:457-547`） |
| `src/main/java/org/cgse/core/CountJump.java` | `ratio()` 由 `private` 放宽为包内可见 + 注释（`:707`）——同一套「浮点只排序、精确算账」的实现复用一份 |
| `tests/core/java/org/cgse/core/CountDemandProjectionTest.java` | 新增 `repairTieBreak()` 10 条断言 + `legacyWeight` / `legacyBetter` / `profile` 三个测试内辅助 |

判据纪律：`after` / `gain` / `hinge` 全程 `BigInteger`；`double` 只在**已经逐行精确相减之后**承担排序（与 `CountJump.compareMoves:337-352` 同构）。published counts 与最终校验（`phase 6`）未动。

## 五、诚实边界

- **`weight` 有限且不等时走快路径**（一次 `double` 比较），只有「相等或 NaN」才回落精确残差——这样常规路径的 `work` 不增加。274 例快照里 `work` 逐例相同，佐证该路径未触发可观测差异。
- **`compareResidualSums` 仍是排序判据，不是证明**：它不参与 `counts` 的算术，也不参与 `phase 6` 的原始行校验。
- 未做：M5 全量回归（`--suite all` 428 项需跨多次调用）；`PortfolioOracle` 默认 2000 例未在单次工具窗口内收口（1500 例跑通）。
- **`verify_boundaries.py` 在基线上就是红的**：脚本判定 `text.startswith('package org.cgse.core;')`，而核心源文件首行是 `// Copyright (c) 2026 autumn` → 几乎每个文件都报 `unexpected package`（我改的三个文件与大量未碰的文件同在名单里）。改前改后一致，属上游脚本与源码头的既有不一致；本轮未擅自修它（写面纪律：不动与目标无关的上游文件）。
