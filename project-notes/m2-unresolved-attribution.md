# M2 未决归因 · 4 例 UNRESOLVED 的难度分解

> 日期：2026-11-03 · 性质：**插桩跑测**——仓库 `src/` 零改动（`AttrProbe` 在 `src/` 命中 0）；插桩与变体全在 `.local/ab/`
> 授权：`project.md` §范围与边界「裁决记录」（上游源码可改 / 不 push / 不碰 GTLCore / 不动 LICENSE·NOTICE）
> 前置：`m2-quota-ab.md`（D 靶面撤回 · 配额上限 ×4 零状态变化 · 旧 87 条 2.5M 0 复现）
> 方法（已拍板）：逐例**分支与调度顺序插桩**——成本最低，且能一次分离「搜索难度 / 启发式缺失 / 分支与调度顺序」
> 结论三条：
> ① **分支顺序不是活变量**——4 例全程只创建过 **1 个分支对象**（`branches=1`），`wave_sum = dispatch`（每波 1 个），`peak_width=1`；`branchPoint()` 的启发式顺序一次都没被执行到
> ② **调度面未找到浪费性重复**——LCG 单实例创建 3 次、续跑 1,983–2,301 次，无「每片重建」；restore 工作量 **0**；scout 仅 17k–26k（0.1%）
> ③ **主因是搜索难度，分两种**——1 例是纯深度不足（`898e00c6` 在 **33,512,894** work 即 PASS = 预算的 1.68×），3 例在 **5 倍预算（100M）下仍 0 witness**、决策数只增长 2.4×，属当前启发式在该单节点上不收敛

---

## 一、口径与代际

| 项 | 值 |
|:--|:--|
| 语料 | 274 例 count-model，`.local/ab/bins/count-model-*.bin`（每 `.bin` 7 例） |
| 引擎 | `main-counts`——`CountModelReplay:108` → `IntegerCountSearch` → `IntegerCountBranch` → `CountViewSearch` |
| 预算 | `milliseconds=0`（关墙钟闸）· `work=20,000,000`（对照探针另跑 100,000,000）· `memory=256 MB` |
| A 侧 | `.local/ab/base/classes`（v0.9 全量 `javac`，与 `build/tests/classes` 1648 类逐字节相同——**本轮未重跑该核对**，沿用 v0.9 结论） |
| 插桩侧 | `.local/ab/attr/classes` = A 侧整棵复制 + 4 文件重编（`AttrProbe` 新增 1 类） |
| 本轮覆盖 | **28 例**（4 批 × 7）× 20M + **28 例** × 100M + 4 例专门归因；**274 例未全量复跑**，计数判据仍引用 v0.9 |
| 退出码语义 | **2 = UNRESOLVED**——不当 PASS、不当 UNSAT；本轮所有 `exit=2` 均按未决记录 |

**插桩单变量核对**（等价性证据，非声明）：

| 检查 | 结果 |
|:--|:--|
| 字节差异 | `attr` vs `base`：`PlanningBudget` / `CountViewSearch`(+2 内部类) / `IntegerCountSearch`(+1 内部类) / `CountModelRegression` + 新增 `AttrProbe` = **7 改 1 增**，共 1649 类 |
| 行为等价 | 4 例状态与 work **逐例相同**（19,995,512 / 19,997,420 / 19,997,992 / 19,995,399）——`work` 全同是"未生效"的判据，此处全同即"只插桩不动路径" |
| ABI | `PASS binary API 0.3: 1502 baseline entries preserved` |

## 二、工作量归属（20M 口径 · 精确切法）

插桩点：`CountViewSearch.step()` 的 `finally`（`budget.threadWork()` 增量 = 该切片真实消耗）、`IntegerCountSearch:314` 的派发窗口。

| case | 状态 | work | 切片数 | 视图内 work | scout | restore | witness | 派发次数 | 窗口 work 和 | 每波分支 | LCG 建 | LCG 续 |
|:--|:--|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| `1204d21049c4` | UNRESOLVED | 19,995,512 | 3,647,080 | 15,559,949 | 22,454 | 0 | 0 | 4,717 | 19,284,440 | 4,717 | 3 | 2,301 |
| `867653b0b246` | UNRESOLVED | 19,997,420 | 3,607,540 | 14,372,946 | 26,434 | 0 | 0 | 4,631 | 18,930,953 | 4,631 | 3 | 2,043 |
| `3f6275678665` | UNRESOLVED | 19,997,992 | 3,301,288 | 14,602,765 | 17,050 | 0 | 0 | 4,595 | 18,790,153 | 4,595 | 3 | 1,983 |
| `898e00c6bbdb` | UNRESOLVED | 19,995,399 | 4,016,163 | 14,832,954 | 22,324 | 0 | 0 | 4,741 | 19,379,728 | 4,741 | 3 | 2,197 |

派生量：

| 量 | `1204d2` | `867653` | `3f6275` | `898e00` |
|:--|--:|--:|--:|--:|
| 平均单次窗口 | **4,088** | 4,088 | 4,089 | 4,088 |
| 窗口 work 占总额 | 96.4% | 94.7% | 94.0% | 96.9% |
| 视图外（分支层）work | 4,413,109（22.1%） | 5,598,040（28.0%） | 5,378,177（26.9%） | 5,140,121（25.7%） |
| 每派发视图外均值 | 935 | 1,209 | 1,170 | 1,084 |
| 每切片均值 | 4.27 | 3.98 | 4.42 | 3.69 |

**第二种切法（note 时间序，按引擎归因，以 `1204d2` 全史为例）**——诊断环放宽到 16384 后完整拿到 6,866 条 note（自 work=50,907 起）：

| note | 条数 | 该段 work | 占比 |
|:--|--:|--:|--:|
| `count_lcg`（LCG 片停） | 2,558 | 12,977,369 | 65% |
| `count_view`（其余视图记账） | 3,609 | 3,581,691 | 18% |
| `count_jump` | 381 | 1,990,501 | 10% |
| `finite_hint` | 228 | 1,008,288 | 5% |
| `integer_counts_pause` | 17 | 210,692 | 1% |
| 其余 20 类（15 类有量） | — | 226,971 | 1.1% |

> 🔴 **两种切法不可相加对照**：上表把「以该 note 收尾的那段 work」记给该 note（LCG 在自己 `local_pause` 处报账），与 §二的「视图内 / 视图外」不是同一刀。前者回答「哪个引擎在烧钱」，后者回答「钱有没有漏在视图之外」。

## 三、分支结构：顺序假设的直接否证

| 观测量 | `1204d2` | `867653` | `3f6275` | `898e00` |
|:--|--:|--:|--:|--:|
| `integer_counts.branches`（分支对象创建数） | 1 | 1 | 1 | 1 |
| `wave_sum / dispatch`（每波实际分支数） | 1.00 | 1.00 | 1.00 | 1.00 |
| `peak_width` | 1 | 1 | 1 | 1 |
| 前 5% 工作量时 `branches` | 1（@555,780） | 1 | 1 | 1 |

**判读**：`branches` 在 `IntegerCountSearch` 的四处自增（`:684` 调度续跑 / `:906` 修复 / `:938` portfolio / `:951` 分裂）里**只被触发过一次**——根分支。整轮 20M 全部落在这一个 `IntegerCountBranch` 对象的 `run()` 窗口里。

因此 `IntegerCountBranch.branchPoint()`（`:1403-1457`，incumbent 邻域 → Gomory → OBBT → branchProbe）**一次都没被执行到**：没有第二个节点可供排序。「分支与调度顺序」在这 4 例上不构成解释项——不是「顺序选错了」，而是**没有顺序可选**。

## 四、调度面排查：没有找到重复劳动

| 疑点 | 判据 | 结果 |
|:--|:--|:--|
| 每片重建求解器（树被丢掉重来） | `lcg_created=3`（对应 original / normalized / reduced 三个视图）、`lcg_resume=1,983–2,301` | ❌ 不成立——单实例多次续跑，无重建 |
| 每片恢复候选代价 | `restore` 累计 **0**（4 例全 0：从无候选可恢复） | ❌ 不成立 |
| scout 引擎抢预算 | `scout=17,050–26,434`（0.1% 量级） | ❌ 不成立 |
| 预算漏在窗口外 | 窗口 work 和 / 总 work = 94.0%–96.9% | ❌ 基本不漏；剩余 22–28% 是分支层 `advance` 与检查，**未细分（未取证）** |
| 配额上限被 policy 层卡住 | 单次窗口恒为 **4,088 ≈ 4,096**（`IntegerCountSearch:314` 字面量硬帽）；Q 探针（`MAX_QUANTUM` ×4）在这 4 例上 work 只差 **+68 / −8**（10⁻³ 量级） | ✅ 有效——policy 上限不在本路径上，解释了 v0.9「×4 零状态变化」 |

## 五、预算放大探针（100M · 判「深度」还是「到顶」）

> 🔴 **不可对照列**：`work` 列（预算不同）、per-engine work 明细（100M 侧诊断环仍是 32，只留尾部 31 条 note）。
> ✅ **可对照列**：状态 / 是否翻转 / 决策与冲突累计、`restarts`。

| case | 20M | 100M | 判读 |
|:--|:--|:--|:--|
| `898e00c6bbdb` | UNRESOLVED 19,995,399 | **PASS SAT @33,512,894** | 纯深度——**1.68×** 预算即解出 |
| `1204d21049c4` | UNRESOLVED 19,995,512（decisions 4,085 / conflicts 2,153 / restarts 15） | UNRESOLVED 99,996,074（decisions **9,678** / conflicts **5,443** / restarts **31**） | 5× 预算：决策仅 2.37×、冲突 2.53×、witness 仍 0；尾部 progress 17 |
| `867653b0b246` | UNRESOLVED 19,997,420 | UNRESOLVED 99,997,548 | 同上 |
| `3f6275678665` | UNRESOLVED 19,997,992 | UNRESOLVED 99,998,031 | 同上 |

**判读**：work → 决策的转化率随深度下降（`1204d2` 20M 侧 LCG 段 12.98M / 4,085 decisions ≈ 3,177；到 100M 侧同口径不可取，但冲突率 53%→56% 未见改善）。这是**收益递减**的形状，不是「差一点就到了」。

## 六、判定

| 假设 | 判据 | 结论 |
|:--|:--|:--|
| **H1 分支顺序 / 调度顺序** | 全程 1 个分支对象、每波 1 个、`branchPoint` 未执行 | **否证**（不是解释项） |
| **H2 调度缺陷（重复劳动 / 预算泄漏）** | 无重建、restore=0、scout 0.1%、窗口覆盖率 94–97% | **未获证据支持** |
| **H3 搜索难度（深度不足）** | `898e00` 在 1.68× 预算处 PASS | **成立（1/4）** |
| **H4 启发式不收敛（当前组合到顶）** | 3 例在 5× 预算下 0 witness、单节点、决策增长 2.4×、progress 停滞；LCG 冲突率 53%、JUMP `pairs=0 / transfers=0`、`equality_support changes=0`、`equality_messages best_unsatisfied = 全部变量`（72 / 127 / 69） | **成立（3/4）** |

**总判定**：4 例 UNRESOLVED = **1 例深度不足 + 3 例启发式在该单节点上到顶**；**「分支与调度顺序」不是成因**，调度面只剩一个未排除项（下节）。

附带修正一条：v0.9「配额上限 ×4 零状态变化」现在有机制解释——`main-counts` 主链的分支窗口被 `IntegerCountSearch:314` 的 `Math.min(4096, …)` 钉死，`CountPortfolioPolicy.MAX_QUANTUM` 在该路径上不是约束。

## 七、待裁（2–3 条，各自代价与判据）

| # | 方向 | 做法 | 能判什么 | 代价 | 风险 |
|:--:|:--|:--|:--|:--|:--|
| **A** | 窗口粒度单变量探针（推荐） | `.local` 变体：`IntegerCountSearch:314` `4096 → 32768`，跑 4 例 ×(20M/100M) + 274 例状态扫 | 排除最后一个「调度缺陷」候选：粒度是否影响结果 | 构建 1 min + 274 例分批 ~20 min + 深跑 3 min | **触碰已暂停的配额切分面**（靶点不同：`:314` 字面量，非 `MAX_QUANTUM`）；若结果为「有效」，改法需另行裁决 |
| **B** | 分支层 22–28% 细分插桩 | 纯测量：在 `IntegerCountBranch.run/advance` 加计数器，拆 4.4–5.6M 的构成 | 这部分是否含可省的重复准备 | ~10 min，无行为变化 | 无（只读计数），但可能得出「就是必要开销」 |
| **C** | 结案 | 3 例登记为当前启发式上限（known-limits 说明补「5× 预算不复现」），`898e00` 标为深度不足 | 收口 M2 | ~10 min 文档 | 放弃 3 例的提升机会 |

**建议顺序**：A → 若 A 零差 → B；两条都空则 C。理由：A 是唯一还挂在「调度缺陷」名下的可判项，且单变量、可复跑。

## 八、诚实边界

- **未做**：274 例全量复跑（本轮只覆盖 28 例 × 2 预算 + 4 例专门归因）；`standard` 412 项 / graph 语料未跑
- **未取证**：分支层 22–28%（4.4–5.6M）的构成——`advance()` 里除视图搜索之外的路径（scheduling / support / program / 冲突检查）未拆分计数
- **不可对照**：100M 与 20M 的 `work` 列、100M 侧的 per-engine work（诊断环仍为 32，只有尾部）
- **代际**：`base/classes` 与 `build/tests/classes` 的 1648 类逐字节核对沿用 v0.9，**本轮未重跑**
- **工作树**：`git status` 有 32 个 `M`（前几轮改码的存量，非本轮产生）——本轮新增文件全在 `.local/`（被 `.gitignore` 覆盖），`src/` 内 `AttrProbe` 命中 0
- **两处读数口径**：`count_lcg` 的 `work=` 是 LCG 引擎内部计数，`@NNN` 才是全局 work；两者不可混用
- 退出码 2 一律记 UNRESOLVED，**未当 PASS、未当 UNSAT**

## 九、复跑命令（cwd = `tools/CGSE_CORE`）

```
python .local\ab\build_attr.py                                  # 插桩变体（base 复制 + 4 文件重编，打印字节差异）
python .local\ab\run_attr.py --arm attr --bins 16,64,128,136     # 20M · 每批 7 例，一批一进程
python .local\ab\run_attr.py --arm base --bins 16,64,128,136 --work 100000000 --tag=-w100m
python .local\ab\attr_report.py .local\ab\logs\attr\count-model-16.log ...     # ATTR 逐例增量表
python .local\ab\attr_notes.py .local\ab\logs\attr\count-model-16.log          # note 切片史归因
python .local\ab\show_case.py 898e00c6bbdb .local\ab\logs\base-w100m\count-model-136.log
python tools\release\check_abi.py --jar build\libs\cgse-core-0.3.2.jar --java runtime\jdk-17.0.2\bin\java.exe
```

本轮末 ABI：`PASS binary API 0.3: 1502 baseline entries preserved`。

## 十、关联

| 面 | 文件 |
|:--|:--|
| A/B 与配额上界 | `m2-quota-ab.md` §四 / §六 |
| 87 条 2.5M 旧归因 | `analysis-scaffold.md` §3.2 / §3.3（已被 v0.9 证伪） |
| 分支点启发式顺序原文 | `IntegerCountBranch.java:1403-1457` |
| 分支窗口硬帽 | `IntegerCountSearch.java:314` |
| LCG 本地额度 | `CountLcg.java:405`（`min(maximumWork, remaining/8)`）· `:456-464`（`local_pause`） |
