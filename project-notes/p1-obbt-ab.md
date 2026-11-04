# P1 — Obbt 弃权 A/B（性能影响实测）

> 日期：2026-11-03 · 性质：**纯跑测**——上游源码零改动，A/B 两侧在 `.local/ab/` 内独立编译
> 授权：`project.md` §范围与边界「裁决记录」；本轮未改任何仓库文件（`src/` `tests/` `build/` 均未触碰）
> 结论：**强制 Obbt 上场不改变任何一例**——解出率、逐例状态、逐例 work 全部相同；原因不是"改善很小"，是 **Obbt 在实链上几乎不上场**（274 例里只被构造 1 次）

---

## 一、口径

| 项 | 值 |
|:--|:--|
| 语料 | 274 例 count-model（`tests/cases/manifest.json` · 与构造级审计同一批） |
| 引擎 | `main-counts`——**唯一会构造 `CountObbt` 的入口**：`CountModelReplay` → `IntegerCountSearch` → `IntegerCountBranch:1437` |
| 预算 | `milliseconds=0`（关墙钟闸，只留 work 上限）· `work=20,000,000` · `memory=256 MB` |
| 分批 | 8 例/批 × 35 批；A/B 读**同一批 .bin 文件**（`.local/ab/bins/`） |
| A 侧 | 现源全量 javac → `.local/ab/base/classes` |
| B 侧 | A 侧类目录整棵复制 + **仅重编 `CountObbt.java`**：`lower.length > 64` → `> 1000000`、`rows.size() > 256` → `> 1000000`（quota 闸不动） |

---

## 二、代际核对（A/B 基线能不能用）

| 检查 | 结果 |
|:--|:--|
| A 侧 vs 现成 `build/tests/classes` | 共有 **1648 类逐字节相同**；现成目录多 10 个 `org/cgse/view/*` 类（本 A/B 不涉及）；A 侧无独有类 |
| 判据 | **同代**，可用 |

B 侧 vs A 侧：1648 类里**只有 `org/cgse/core/CountObbt.class` 一个字节不同**——单变量。

---

## 三、结果

### 3.1 解出率与步数

| 面 | A 侧（原路径） | B 侧（强制上场） | 差 |
|:--|--:|--:|--:|
| PASS | 264 | 264 | 0 |
| UNRESOLVED | 4 | 4 | 0 |
| UNSUPPORTED | 6 | 6 | 0 |
| 解出率 | 264/274 = 96.35% | 264/274 = 96.35% | **0** |
| work 合计 | 277,102,520 | 277,102,520 | **0** |

### 3.2 逐例状态差

- 快照对账（268 例有 work 的行）：**语义相同 268 · 严格相同 268 · 差异 0**
- 6 例 UNSUPPORTED（`main-counts conversion requires finite upper bounds`）两侧逐条相同
- 日志逐字节 diff：差异**只出现在 `elapsed_ms` / `improvement_ms` 两个墙钟浮点**，逻辑字段（work / branches / 计数 / 状态）零变化

### 3.3 补丁生效核验（否则"结果相同"无意义）

| 构造级探针（`AuditGate`，同一 bin / 同一 work） | A 侧 | B 侧 |
|:--|--:|--:|
| `refusedObbt` | **52** | **0** |
| `refusedAny` | 81 | 48 |

补丁确实把 52 例 size 弃权全部消除——**端到端仍逐例相同**。

---

## 四、为什么相同：Obbt 在实链上的可及性

插桩计数（第三套类目录，静态 `AtomicLong`——不受 `PlanningBudget.note` 32 条环形缓冲淘汰影响）：

| 观测 | 值 |
|:--|:--|
| 274 例里 `CountObbt` 被构造的例数 | **1**（`af3a59ddc15f…`，该例 PASS，work=18,087,393） |
| size 闸在实链上的拒绝次数 | **0** |
| 实跑步数 | 98 |

静态面同源：`CountObbt` 唯一构造点在 `IntegerCountBranch:1437`，条件是 `current.isEmpty() && !obbtTried && !feedbackRegion && repairWork < 262144` **且** LP 松驰给出分数点——实链上极难走到。

**52 例构造级弃权在实链上的分布**：43 PASS · 4 UNRESOLVED · 5 UNSUPPORTED。
即：被标记"弃权"的案例绝大多数在实链里根本没让 Obbt 上场，直接由别的引擎解掉了。

---

## 五、诚实边界

- **只在 `main-counts` 链上成立**：`lcg-first` / `views` 等引擎不经过 `CountObbt`（唯一构造点在上表），对本轮问题无信息量
- **未评价 64 / 256 阈值本身是否合理**——只证明「在本语料、本链上放宽它们没有可测收益」
- 预算口径：`work=20M`，且 `IntegerCountSearch:103` 对内部整数阶段另有 `min(2M, 剩余)` 上限（两侧同口径）；4 例 UNRESOLVED 都是 ~19.99M work 耗尽，非预算外因素
- 未跑 core / ABI 门禁——本轮源码零改动，产物全部在 `.local/`

---

## 六、复跑命令（全参数）

```bat
:: 1) 语料装订（274 例装成 35 批）
python .local/audit/build_model_bins.py --batch-size 8 --out .local/ab/bins

:: 2) A / B 两侧类目录（含代际核对与单变量核验输出）
python .local/ab/prepare_ab.py

:: 3) 跑测（A 侧；B 侧把 --arm base 换成 forced）
python .local/ab/run_ab.py --arm base --bins all --work 20000000 --engine main-counts --trace true

:: 4) 单批等价的原生命令
runtime\jdk-17.0.2\bin\java.exe -ea -Dfile.encoding=UTF-8 -Xmx2g -cp .local\ab\base\classes org.cgse.core.CountModelRegression .local\ab\bins\count-model-0.bin 0 20000000 main-counts true

:: 5) 补丁生效核验（构造级 52 -> 0）
python .local/ab/verify_patch.py

:: 6) Obbt 可及性插桩（构造 / size 拒绝 / 实跑步数）
python .local/ab/build_probe.py
python .local/ab/run_probe.py --work 20000000

:: 7) 快照与对账
python .local/audit/snapshot_models.py .local/ab/logs/base .local/ab/snap-base.json
python .local/audit/snapshot_models.py .local/ab/logs/forced .local/ab/snap-forced.json
python .local/audit/diff_models.py .local/ab/snap-base.json .local/ab/snap-forced.json
python .local/ab/p1_summary.py
```

---

## 七、产物

| 路径 | 内容 |
|:--|:--|
| `.local/ab/base/classes` | A 侧类目录（现源） |
| `.local/ab/forced/classes` | B 侧类目录（仅 `CountObbt.class` 不同） |
| `.local/ab/probe/classes` | 插桩类目录（静态计数器版 `CountObbt` + `ObbtProbe`） |
| `.local/ab/bins/` | 35 批语料 |
| `.local/ab/logs/base` · `logs/forced` | 两侧逐批日志（含 `DIAGNOSTICS`） |
| `.local/ab/snap-base.json` · `snap-forced.json` | 逐例快照 |
| `.local/ab/logs/probe.txt` · `logs/gate-all.txt` | 插桩明细 / 构造级全表 |

---

## 八、对下一轮的意义

预置判据是「强制上场能解出更多例 → M3；否则 → M2/M7」。实测落在**后者**，且读数更硬：

- **M3（放宽 64/256）在本语料链上的预期收益为 0**——不是因为改善被别处吃掉，是这两条闸从来不是瓶颈（实链拒绝 0 次）
- 真正的闸门在 Obbt 的**入场条件**（`IntegerCountBranch:1433-1438`），不在 `CountObbt` 的 size 闸
- 4 例 UNRESOLVED 全在 52 例弃权名单里，但它们在实链里同样没触发 Obbt——其未解不是 Obbt 造成的

---

## 九、P1b — 入场条件 A/B（闸门在更外面）

**动机**：P1 证明 size 闸在实链 0 次触发。于是下一问不是"放宽 size 闸"，而是"是不是**入场条件**把 Obbt 关在门外"。单变量换成 `IntegerCountBranch:1433` 的入场布尔，其余不动。

### 9.1 直方图（条件保持上游原样，仅插桩）

| 计数 | 合计 | 命中例 |
|:--|--:|--:|
| `branchPoint` 调用 | **4** | 3 |
| 分数点（`fractionalChoice >= 0`） | **3** | 2 |
| 被 `obbtTried` 挡 | 1 | 1 |
| 被 `!current.isEmpty()` 挡 | 0 | 0 |
| 被 `feedbackRegion` 挡 | 3 | 2 |
| 被 `repairWork >= 262144` 挡 | 1 | 1 |
| 真进场 | 1 | 1 |

**读法**：最外层的稀缺不是任何一条闸，而是「LP 松驰给出分数点」本身——274 例里只出现 3 次（`branchPoint` 总共只被调用 4 次）。闸只决定这 3 次里的 1 次进场。

### 9.2 变体（每个只改入场布尔，A 侧类目录同源）

| 变体 | 入场条件 | Obbt 上场 | 状态（PASS/UNRES/UNSUP） | work 合计 |
|:--|:--|--:|:--|--:|
| A（原路径） | 原样 | 1 例 / 98 步 | 264 / 4 / 6 | 277,102,520 |
| `open-feedback` | 去 `!feedbackRegion` | 1 例（另两处仍被 `obbtTried` / `repairWork` 同时挡） | 264 / 4 / 6 | 277,102,520（0 差） |
| `open-feedback-repair` | 去 `feedbackRegion` + `repairWork` 两闸 | **2 例 / 228 步** | **264 / 4 / 6（0 差）** | **+66,326** |
| `open-current` | 去 `!current.isEmpty()` | 直方图判定 0 次阻断 → 零机会（未跑） | — | — |
| `open-all` | 恒真（每次分数点都进） | **181 例后 NPE** | 不成立 | 不成立 |

- 唯一有变化的单例：`b11051990caf` work 4,269,192 → 4,335,518（**+66,326 步，+1.55%**），状态仍 PASS——**纯成本，无收益**
- `open-all` 崩栈：`IntegerCountBranch.fractionalChoice(:1664)` ← `branchPoint` ← `advance(:966)`，`point` 为 null——`obbtPoint` / `obbtTried` 的单发生命周期与入场判定耦合，**入场闸不是可随意放开的参数**

### 9.3 结论

- 入场闸同样不是杠杆：能多给的唯一机会（1 例）换来 0 状态变化 + 66,326 步成本
- 本语料里绝大多数模型在 LP 分支之前就被 LCG / Boolean / Matching / 视图等解掉，Obbt 的舞台基本不存在
- 因此 **M3（放宽 64/256）与「放宽入场闸」两条路径的实测收益都是 0 / 负**；要继续做性能面，落点不在 Obbt

### 9.4 复跑命令（全参数）

```bat
:: 变体装配（gate = 仅插桩；open-* = 单变量放宽）
python .local/ab/build_variant.py gate open-feedback open-feedback-repair open-all
python .local/ab/build_gate2.py

:: 跑测（274 例按 bin 分段；--probe 切两种探针）
python .local/ab/run_p1b.py --arm gate2 --probe P1bProbe2 --bins all --work 20000000
python .local/ab/run_p1b.py --arm open-feedback-repair --probe P1bProbe --bins all --work 20000000

:: 汇总
python .local/ab/report_gate2.py
python .local/ab/report_p1b.py gate open-feedback open-feedback-repair
```

产物：`.local/ab/{gate,gate2,open-feedback,open-feedback-repair,open-all}/classes` · `logs/p1b-*.txt`。
