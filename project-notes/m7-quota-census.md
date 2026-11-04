# M7 · 配额分母分类清单（allowance / remainingWork）

> 轮次：M7 分类轮（只读）· 2026-11-03
> 任务：`todo.md`「M7 配额复核」+ `analysis-docs.md` §2.5 —— 先出「裸固定分母 vs 文档侧双限式」分类清单，再定 M2 改法。
> 定性依据 → `analysis-docs.md` §2.5；脚手架 → `analysis-scaffold.md` §七 P5。
> 附表（全量逐处）→ `m7-allowance-census.md`（机读，312 行表）。
> 🔴 **勘误（2026-11-03）**：§一「87 例同一个除法」与 §3.2/3.3 的 2.5M 归因基于 `known-limits.json`（`source_revision e788d3d3`）旧记录——当前 `HEAD b7c4a81` 上 87 条 **0 条复现 2.5M**（79 PASS / 4 UNRESOLVED 用满 19.99M / 4 UNSUPPORTED）。实测见 `m2-quota-ab.md` §五。

---

## 〇、本轮边界（实测声明）

| 项 | 实况 |
|:--|:--|
| 源码代际 | `HEAD b7c4a81`（2026-10-27 20:02:37）+ 工作区未提交改动 32 文件（M1 系列，`git status --porcelain` 实测） |
| 本轮动作 | 只读源码 + 只跑静态扫描；**零源码改动、零编译产物、零跑测**——故无退出码，不涉 `--no-compile` 与 `check_abi.py` |
| 排除面（未触碰） | `64 / 256 / 8192` 阈值 · `CountObbt` 的 `64/256` 与 `allowance` 判定 · `IntegerCountBranch:1433-1438` 入场条件 · `PlanSourceCycles:244` |
| 授权面 | 未改任何源文件；不 push；不碰 GTLCore / LICENSE / NOTICE |

---

## 一、计数口径（先写死）

**实测（2026-11-03，`src/main/java/org/cgse/core`，326 个 `.java`）**

| 量 | 实测值 | 取法 |
|:--|--:|:--|
| `remainingWork()` 出现次数（含比较 / 相等 / 整值传递） | **327** | `text-grep` |
| 其中分母为 `remainingWork()` 类（含 `choiceRepair.`） | 200 | 判定器 v2 |
| —— 字面量除数 | 197 | 判定器 v2 |
| —— 除数非字面量 | 3（`CountLcg:332` · `CountLpProximity:82` · `GraphPlanningWork:952`） | 判定器 v2 |
| **分母为额度 / 剩余 / 总量的除法点（本轮全量）** | **312** | 判定器 v2 |

**312 的分母对象分解**

| 分母对象 | 处数 |
|:--|--:|
| `budget.remainingWork()` | 198 |
| `allowance`（本地已派生额度） | 49 |
| `budget.availableBytes()`（内存轴） | 53 |
| `maximumWork`（父层传入额度） | 9 |
| `choiceRepair.remainingWork()` | 2 |
| `this.allowance` | 1 |
| **合计** | **312** |

**三口径对齐（对存量两说的处置）**

| 口径 | 值 | 构成 | 状态 |
|:--|--:|:--|:--|
| todo M7 | 247 | **198（`budget.remainingWork()`）+ 49（`allowance`）= 247**（算术复原） | 采信为**子集口径**：请求剩余 + 本地额度，不含内存轴与父层额度 |
| `analysis-scaffold` §七 P5 | 246 | 上一轮 `scan_allowance.py` 输出（`cap` 列只在赋值左值为 `allowance` 时识别 `Math.min`；且跳过 `//` 出现在 `allowance` 之前的行） | 与 247 差 1 为脚本口径差异，非实体差异；**cap 列不可用作分类依据**（实证：`CountRowSparsifier.java:34` 的 `min(65536, remainingWork()/64)` 被记为无 cap） |
| 本轮 | **312** | 六个分母对象全量 | **本轮采用** |

> 结论：`remainingWork()/N` 的实体总数是 198（另有 2 处走 `choiceRepair.remainingWork()`）；把 247 理解为「含 `allowance/N` 的派生链全量」才成立。清单正文按 **312 全量**出，族分组与小计同时给出 247 子集位置，避免下一轮再对不上账。

**脚手架代际核查（要求项）**

| 项 | 结果 |
|:--|:--|
| `scan_allowance.py` 上次输出 | `allowance-sites.csv` 2026-11-02 00:34（17,715 B） |
| 本轮是否过期 | 判为「可用但不足」——重跑结果仍为 246 处（与 P5 记录一致，无漂移），但 cap 列有系统性漏判（§一表内实证） |
| 处置 | 重跑 v1 复核基数；新增判定器 v2（与左值名无关，看 `Math.min` 实参结构）作为分类依据 |

---

## 二、分类判据（两档主判 + 三个补充档）

**两条线（本轮采用）**

| 线 | 问什么 | 判据 |
|:--|:--|:--|
| A 线 | 除数与被切对象相关吗 | 除数是**字面量 N** → 固定分母；除数是对象规模 / 参数表达式 → 非固定（本清单 ③ 档，仅 3 处） |
| B 线 | 是否与预算剩余 / 声明上限同源 | 除法处于 `Math.min(...)` 实参位 → **双限式**（另一侧实参即上限来源）；不在任何 `Math.min` 内 → **裸固定分母** |

**归档**

| 档 | 定义 | 处数 |
|:--|:--|--:|
| ① | 裸固定分母——`remainingWork() / N` 没有第二限 | 12 |
| ②a | 双限式——`min(绝对常量, remainingWork()/N)` | 144 |
| ②b | 双限式——`min(maximumWork / MAX_WORK / maxWork, remainingWork()/N)`（上限来自父层传入额度） | 29 |
| ②c | 双限式——`min(allowance / continuationLimit-allowance, remainingWork()/N)`（上限=本地剩余额度） | 12 |
| ③ | 除数非字面量（除数与被切对象或分支相关） | 4 |
| ④ | 分母不是请求剩余——本地额度二次切 49 · 内存轴 53 · 总量切分 9 | 111 |

> ②b 的 `maximumWork` 是**构造器形参**，不是请求总量：调用方传入 `Math.min(65536, allowance)`（`CountDomainSearch:62/71/76`）、`Math.min(32768, allowance - work)`（`CountNeighborhood:354`）或 `Math.min(max(0, allowance-work), min(262144-w, R/16))`（`CountLcg:859`）。所以 ②b 是**链式额度收窄**，不是「cap 支永假」。

---

## 三、形态小计

| 形态 | 处数 | 占 312 |
|:--|--:|--:|
| ① 裸固定分母 | 12 | 3.8% |
| ②a 双限·绝对常量 | 144 | 46.2% |
| ②b 双限·父层额度 | 29 | 9.3% |
| ②c 双限·本地剩余 | 12 | 3.8% |
| ③ 除数非字面量 | 4 | 1.3% |
| ④ 本地额度二次切 | 49 | 15.7% |
| ④ 内存轴（`availableBytes`） | 53 | 17.0% |
| ④ 总量切分 | 9 | 2.9% |
| **合计** | **312** | 100% |

**只看「分母 = 请求剩余」的 200 处（198 + 2）：**

| 判定 | 处数 | 占比 |
|:--|--:|--:|
| 双限式（②a + ②b + ②c） | 185 | 92.5% |
| 裸固定分母（①） | 12 | 6.0% |
| 除数非字面量（③，其中 1 处与对象相关） | 3 | 1.5% |

> **对上一轮定性的修正：** `analysis-docs.md` §2.5 记「247 处配额派生几乎全是 `remainingWork()/N` 的固定 N，与文档侧双限式不一致」——**不成立**。除以请求剩余为分母的 200 处里，185 处（92.5%）自带宽限 `Math.min`，与文档 `equality-sparsification.txt:24-25` 声明的 `min(65536, remaining request work / 64)` 同形（实证：`CountRowSparsifier.java:34` 逐字一致）。差异面收窄到 **12 处裸固定分母 + 3 处非字面量除数**。

---

## 四、引擎族分组

| 族 | 处数 | ① 裸 | ②a 绝对 | ②b 父额度 | ②c 本地剩余 | ③ 非字面量 | ④ 本地额度 | ④ 内存轴 | ④ 总量 | 除数集合 |
|:--|--:|--:|--:|--:|--:|--:|--:|--:|--:|:--|
| Count* 其余 | 181 | 6 | 73 | 19 | 5 | 1 | 36 | 34 | 7 | 非字面量, 2, 3, 4, 6, 8, 16, 32, 64 |
| Graph* | 25 | 0 | 16 | 1 | 0 | 1 | 2 | 5 | 0 | 非字面量, 2, 3, 4, 8, 16 |
| Captured* | 22 | 4 | 10 | 4 | 0 | 0 | 2 | 0 | 2 | 2, 4, 8, 16 |
| **CountLcg** | **18** | 1 | 4 | 1 | 3 | 1 | 0 | 8 | 0 | 非字面量, 4, 8, 16, 32, 128 |
| Refill* | 17 | 0 | 8 | 0 | 0 | 1 | 3 | 5 | 0 | 非字面量, 2, 4, 8, 16 |
| IntegerCount* | 14 | 0 | 9 | 1 | 3 | 0 | 1 | 0 | 0 | 2, 4, 5, 8, 16, 32 |
| 单点杂项 | 10 | 1 | 4 | 2 | 0 | 0 | 3 | 0 | 0 | 2, 4, 8, 16, 32 |
| Exact* | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0 | 0 | 4, 16, 64 |
| MissingMaterialExpansion | 3 | 0 | 2 | 0 | 0 | 0 | 0 | 1 | 0 | 4, 16 |
| Plan* | 3 | 0 | 3 | 0 | 0 | 0 | 0 | 0 | 0 | 16, 32, 64 |
| Seed* | 3 | 0 | 2 | 0 | 0 | 0 | 1 | 0 | 0 | 8, 16 |
| State* | 3 | 0 | 3 | 0 | 0 | 0 | 0 | 0 | 0 | 16, 32 |
| **CountSoftSearch** | **2** | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 3, 16 |
| Quantity* | 2 | 0 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 8 |
| Region* | 2 | 0 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 4, 8 |
| **CountJump** | **1** | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 8 |
| **CountObbt** | **1** | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 32 |
| **CountDomainSearch** | **0** | — | — | — | — | — | — | — | — | — |
| **合计** | **312** | 12 | 144 | 29 | 12 | 4 | 49 | 53 | 9 | — |

**族级读法**

- **CountDomainSearch 0 处**——该族不做任何配额切分：接收父给 `allowance`（`:115` 用 `min(65536, allowance)`），并在三处形状外流时把 `min(65536, allowance)` 转交 `CountLcg`（`:62/71/76`）。上一轮「1/8 切走」的怀疑对象不在这一族。
- **CountLcg 18 处里 8 处是内存轴**（`availableBytes/16/…`），工作量侧只有 10 处；1/8 只出现 1 次（`:405`）。
- **CountObbt 1 处**（`:27` `min(65536, R/32)`）——本轮排除面，仅登记。
- **④ 内存轴 53 处**是本轮新增账目：此前所有轮次都只算工作量轴，内存预算同样按 `/N` 派生且多数没有第二限。

---

## 五、焦点明细（12 处，逐处判据）

| # | 调用点 | 除数 | 上限来源 | 档 | 一句判据 |
|--:|:--|--:|:--|:--:|:--|
| 1 | `CountLcg.java:405` | 8 | `maximumWork`（构造器形参） | ②b | `allowance = min(maximumWork, R/8)`；绑定支取决调用方——`CountDomainSearch:62` 传 `min(65536, allowance)`，此时 1/8 支不生效 |
| 2 | `CountLcg.java:921` | 8 | 三层：`65536` ∧ `max(0, allowance-work)` | ②c | `min(65536, min(R/8, allowance-work))`——全清单唯一带「本引擎剩余额度」的形态，最接近按需申领 |
| 3 | `CountLcg.java:332` | `max(1, 8L*low.length)` | `min(2048, …)` | ③ | 除数随被切对象规模变化（`low.length`）——**全清单唯一「除数与被切对象相关」**的正面样本 |
| 4 | `CountLcg.java:864` | 16 | 无 | ① | `budget.remainingWork()/16 < 1024` 是**准入门槛**（不派发额度），与「派生」同形不同义 |
| 5 | `CountDomainSearch.java:62/71/76` | — | `min(65536, allowance)` | — | 该族零除法：接父额度并原样转交，1/8 不在此发生 |
| 6 | `CountObbt.java:27` | 32 | `65536` | ②a | `min(65536, R/32)` + `:31` `allowance<4096` 即拒 → 等效要求 `R ≥ 131072`；排除面，仅登记 |
| 7 | `CountSoftSearch.java:49` | 16 | `maximumWork` | ②b | 与 1 同构；`:107` 另有 `min(65536, allowance/3)` 二次切（④） |
| 8 | `CountJump.java:97` | 8 | `allowance`（形参） | ②c | `min(allowance, R/8)`——父额度与剩余/8 双限，父子链收窄的干净样本 |
| 9 | `CountReduction.java:73` | 8 | `131_072` | ②a | 12 处里唯一「派生」；其余 11 处全是 `work < allowance/2` **门槛**（用掉本地额度一半就让位） |
| 10 | `CountBounds.java:544` | 8 | `continuationLimit - allowance` | ②c | 续期增量 = `min(续期余量, R/8)`——「剩余额度」而非「剩余工作量」当上限，M2 的现成范式 |
| 11 | `GraphCatalogIndex.java:94` | 8 | `8L << 20` | ④ | 分母是 `availableBytes()`（内存），与工作量配额无关——口径澄清项，不计入 247 子集 |
| 12 | `CountRelaxationNeighborhood.java:98` | 4 | `524_288` | ④ | `min(524288, maximumWork/4)`——切的是**请求总量**而非剩余，与「按剩余比例」不同源 |

---

## 六、与文档侧双限式的对照

| 文档声明 | 代码对应 | 结果 |
|:--|:--|:--|
| `equality-sparsification.txt:24-25`「capped at min(65536, remaining request work / 64)」 | `CountRowSparsifier.java:34` `limit = Math.min(65536, budget.remainingWork() / 64)` | **逐字一致**（②a） |
| 同上模式在 `stateful-inputs.txt:154-171` / `root-execution-reachability.txt:14-54` | 全库 144 处 ②a 同形 | 覆盖成立 |

> 判读：**文档与代码在形态上已经同构**，问题不在「文档说双限、代码写单限」，而在三个更细的面：① 12 处裸固定分母（其中 8 处是门槛、4 处是直接派生）② 49 处本地额度二次切（多数是 `work < allowance/N` 就让位的门槛）③ 53 处内存轴完全无第二限。

---

## 七、待裁点（M2 方向，本轮不选）

| 候选靶面 | 处数 | 改动面 | 代价 |
|:--|--:|:--|:--|
| A 组：12 处裸固定分母（4 处派生 + 8 处门槛） | 12 | 小、可逐处给 cap | 门槛类改了只影响「何时让位」，收益未测 |
| B 组：49 处本地额度二次切（29 门槛 + 20 再分配） | 49 | 中、集中在 `CountReduction` 11 处 / `CountBounds` 8 处 | 改门槛会改变各引擎让位时机，需 A/B 才有说服力 |
| C 组：53 处内存轴无第二限 | 53 | 大、跨 20+ 文件 | 与工作量解耦，收益方向不明；且不属于 M7 原问 |

**未竟（需下一轮或拍板后补）**

1. ②b 的 29 处**绑定支未逐条判**——`maximumWork` 与 `R/N` 谁先绑定取决于调用方实参，本轮只核了 3 条调用链（`CountDomainSearch` / `CountNeighborhood` / `CountLcg:859`），未做 18 个 `new CountLcg(` 调用点全表。
2. 49 处 ④ 的「门槛 29 / 再分配 20」是人工读上下文判读（`text-grep` 带回 ±10 字符全量可见），未机读化。
3. `models` 套件 2,500,001 的截停点**未重新定位到具体行**——本轮只确认「分母为 `R/8` 且有父层额度上限」的形态，未证绑定支。

---

## 八、附表与复跑

| 项 | 位置 |
|:--|:--|
| 全量逐处表（312 行） | `ccbp:Project/CGSE_CORE/m7-allowance-census.md` |
| 判定器 v2（本轮新增） | `workspace:tools/CGSE_CORE/.local/audit/scan_allowance_v2.py`（与 v1 同址，可复用） |
| v1 复核输出 | `workspace:tools/CGSE_CORE/.local/audit/out/allowance-sites.csv`（2026-11-03 重跑） |

```
python .local\audit\scan_allowance_v2.py      # cwd = tools/CGSE_CORE
python .local\audit\scan_allowance.py         # cwd = tools/CGSE_CORE
```
