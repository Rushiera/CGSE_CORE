# 模型形态侦察（v0.24）

> 起点：外部建议称「困难实例 = 100~500 布尔选择 / 5~30 资源维度，应强化整数优化（Branch-and-Cut / PB 传播 / LP 松弛）」，并主张 CGSE 已有的 `CountPackingSearch` / `CountCoverCuts` / `CountPackingLpSearch` / `CountReducedCost` 不应重实现。
> 本文件回答：**这条建议假设的形态，与真实语料是不是同一类问题。**

---

## 一、结论先行

真实语料的整数模型是「**带 15~17 个无界整型变量的混合符号线性约束系统**」，不是 0/1 多维装箱。三个看似独立的失败——packing 零入场、LP 只覆盖 29%、冲突学习面为零——由**同一个根因**驱动：无界变量。

## 二、实测画像（funded x1000，四个编译视图）

| 视图 | n | 0/1 | 多值 | **无界** | 行 | 混合符号行 | 项数 |
|:--|--:|--:|--:|--:|--:|--:|--:|
| closure（初） | 439 | 145 | 238 | **56** | 291 | 237 | 740 |
| closure（后） | 308 | 26 | 234 | **48** | 832 | 223 | 1292 |
| light_compile | 252 | 121 | 116 | **15** | 68 | 53 | 199 |
| reduced | 145 | 15 | 116 | **14** | 366 | 56 | 545 |
| original | 280 | 147 | 116 | **17** | 772 | 192 | 2425 |

**不变量：** `multi=116` 在所有视图恒等；`unbounded=14~17`；`peak_high=9223372036854775807`（`Long.MAX_VALUE` = 无上界哨兵）；多值分层 `[≤4:15, ≤64:37, ≤4096:19, >4096:45]`（45 个上界 >4096 或无穷）。

对照外部假设：变量数 252~280 落在「100~500」区间内——**只有量级对**；域结构完全不同（布尔占 48%，多值 46%，无界 6%），行结构也非「5~30 维容量行」（69~772 行中混合符号行占多数）。

## 三、根因链

```
15~17 个无界变量（high == null）
  ├→ CountPackingStructure.create 域闸：high > 1 → 拒（packing 全族零入场）
  ├→ CountIntegerLpLearning.eligible：!bounded(low, high) → 该行整行排除
  │     → LP 支持面 128/439 变量、110/291 行（29%）
  │     → count_lp_structure: prepared=0; hits=0（结构从未启用）
  ├→ CountDiophantine`shape:upper==null` / CountDecisionDiagram`shape:upper==null`
  └→ CountPartition / CountBoolean `shape:not_binary`
```

`bounded()` 定义（`CountIntegerLpLearning.java:232`）：`high != null && low <= high && low.bitLength() <= 1024 && high.bitLength() <= 1024`——**`high == null` 直接判否**，与位长无关。

## 四、顺带取到的四条硬线索

| # | 线索 | 证据 |
|:--:|:--|:--|
| 1 | **8192/8193 边界卡掉整模型** | `count_model: skipped; closure_recipes=8193; local_limit=8192`；`admission_refused: owner=GraphOutputView size:catalog>8192`；源码 `GraphCompiler.java:331-335` `found.size() > maxRecipes` → `return null` |
| 2 | **LP 支持面受双约束** | `MAX_COLUMNS=128`（`CountIntegerLpLearning.java:12`）+ 行内 `bounded()` 筛；实测 `variables=128/439; rows=110/291` |
| 3 | **冲突学习面为零** | LCG `conflicts=0/1/3`、`backjumps=0/1`、`restarts=0`、`minimized=0`；`count_conflict_pool: learned=1; used=0` |
| 4 | **专用引擎集体形态不符** | `CountDiophantine shape:upper==null` ×7 · `CountPartition shape:not_binary` ×12 · `CountBoolean shape:not_binary` ×10 · `CountCardinalitySearch/CountBinPacking unsupported_domain` · `CountProbing size:variables>128` —— 只剩 LCG / JUMP / MATCH / SEPARATOR 在干活 |

## 五、对外部建议的逐条判定

| 建议手段 | 在真实形态下 |
|:--|:--|
| Branch-and-Cut / PB 传播 | 需有限域与纯正系数——45 个变量无界、混合符号行占多数 |
| 变量耦合少时分解子问题 | `CountComponents: variables=123->111; component_sizes=[54,3,5,2,25,...]`——最大分量 54/123 |
| 小容量小维度精确 DP | 无界域直接排除 |
| 加强「全局上下界 ↔ 局部搜索」交换 | 冲突池 `learned=1 used=0`——**没有信息可交换**，源头为空 |
| LP 对偶价格判不可能满足预算 | **唯一能落地的**，但落点不在 packing，而在「LP 覆盖 29% 且从未 prepared」 |
| 不要重实现已有机制 | 成立——但更该说的是「已有机制压根没进过场」 |

上游 `docs/packing-priced-propagation.txt:27-28` 明确写「不外传到其他 face / view / order / 全局不可行通道」——是健全性设计（Boolean box 结论不能外推），不是漏做。

## 六、脚手架

`workspace:tools/CGSE_CORE/.local/scaffold/probe.py`（+ 同目录 `README.md`）

```bat
python .local\scaffold\probe.py apply
python .local\scaffold\probe.py run --target gtlcore:miracle_crystal --modes extra-first --amounts 1000 --out .local\scaffold\out
python .local\scaffold\probe.py report --out .local\scaffold\out
python .local\scaffold\probe.py restore
```

探针 3 处（note 容量 32→4096 · `DatasetReplay` 暴露 diagnostics · 域闸打形态摘要），零语义影响（apply 前后 work 逐值同为 38,019,380）。`restore` 后 `git status` 输出 `[clean]`。

## 七、下一轮靶点（按成本排序）

| 序 | 动作 | 成本 | 判据 |
|:--:|:--|:--|:--|
| 1 | `GraphCompiler` `maxRecipes` 8192 → 8193（或按预算动态） | 一处常量 + 一轮跑测 | `count_model` 不再 skip；work / 状态变化 |
| 2 | 查 `count_lp_structure: prepared=0` 的调用链为何不触发 | 静态读码 | `prepared > 0` 且 `hits > 0` |
| 3 | 给无界变量推上界（stock / 需求推导）后喂现有引擎 | 中等 | 被域拒的引擎能否入场（`admission_refused` 减少） |

## 八、诚实边界

- 只跑了 1 个请求（funded x1000）+ 11 个请求的准入扫描；小规模实例可能形态不同。
- `random` 模式在当前代际 4/4 TIMEOUT（回滚后复现，与探针无关）；`regressions.json` 的 19 请求并非全绿，与 v0.17 记录的「19/19」口径不同源（`regressions.py` vs `regressions.json`），未核。
- 「无界变量是唯一根因」是相关性证据 + 代码判据的合成，未做单变量实验（例如只给 1 个无界变量补上界看准入变化）。
- `count_bin_packing` 恒出现 1 条 `unsupported_domain`，属 `CountBinPackingSearch`（另一支），未深究。
