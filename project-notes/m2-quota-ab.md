# M2 配额复核 · A/B 对照（D 靶面证伪 + 配额杠杆上界）

> 日期：2026-11-03 · 性质：**纯跑测**——仓库 `src/` 零改动，A/B 两侧在 `.local/ab/` 内独立编译
> 授权：`project.md` §范围与边界「裁决记录」（上游源码可改 / 不 push / 不碰 GTLCore / 不动 LICENSE·NOTICE）
> 前置：`m7-quota-census.md`（312 处分类）· `analysis-scaffold.md` §3.2/3.3（87 例 2.5M 归因）
> 结论三条：
> ① **旧 known-limits 的 87 条 2.5M 记录在当前代际 0 条复现**——79 条已 PASS，其余 8 条分别是用满 20M 的 UNRESOLVED（4）与转换不支持的 UNSUPPORTED（4）
> ② **D 靶面（`CountViewSearch` 续跑窗口）撤回**——补丁未生效，274 例 work 逐例完全相同
> ③ **配额上限 ×4 探针：274 例状态零变化**——工作量配额切分在当前语料上不是求解结果的决定因素

---

## 一、口径

| 项 | 值 |
|:--|:--|
| 语料 | 274 例 count-model（`tests/cases/manifest.json` 同批，`.local/ab/bins/count-model-*.bin`） |
| 引擎 | `main-counts`——`CountModelReplay:108` → `IntegerCountSearch` → `IntegerCountBranch:1050` → `CountViewSearch` |
| 预算 | `milliseconds=0`（关墙钟闸）· `work=20,000,000` · `memory=256 MB` |
| 分批 | 8 例/批 × 34 + 2 例；A/B 读**同一批 .bin** |
| A 侧 | 当前工作树全量 `javac` → `.local/ab/base/classes` |
| 变体侧 | A 侧类目录整棵复制 + **仅重编一个文件** |
| 仓库源码 | 零改动（`git status` 与轮次开始一致） |

## 二、代际核对（要求项）

| 检查 | 结果 |
|:--|:--|
| A 侧 vs `build/tests/classes` | 共有 **1648 类逐字节相同（字节不同 0）**；现成目录多 10 个 `org/cgse/view/*` 类（本 A/B 不涉及） |
| 判据 | **同代**，A 侧可作基线 |
| D 侧 vs A 侧 | 1648 类里仅 `org/cgse/core/CountViewSearch.class` 不同（单变量） |
| Q 侧 vs A 侧 | 1648 类里仅 `org/cgse/core/CountPortfolioPolicy.class` 不同（单变量） |

## 三、实验 1：D 续跑窗口（撤回）

**构造**：`CountViewSearch.java:864` `Math.min(4_194_304, budget.remainingWork() / 8)` → `/ 2`。

| 面 | A 侧 | D 侧 | 差 |
|:--|--:|--:|--:|
| PASS | 264 | 264 | 0 |
| UNRESOLVED（退出码 2） | 4 | 4 | 0 |
| UNSUPPORTED | 6 | 6 | 0 |
| 逐例 work 相同 | — | — | **274 / 274（全同）** |

**判读：补丁未生效。** work 逐例完全相同说明该分支在 274 例上从未改变 `continuationLimit`。代码复核给出同向证据：`:861-865` 的 `opportunity` 只对 `CountGroupedPartition` / `CountVectorPartition` / `CountRelaxationNeighborhood` / `CountPackingCore` 四类 specialist 生效（`:837-842` 的 instanceof 过滤），`main-counts` 主链的 `CountLcg` 不在其中。

**处置：D 候选撤回**——上一轮「87 例截停点在此」的定位不成立。

## 四、实验 2：配额上限探针（生效，但零状态变化）

**构造**：`CountPortfolioPolicy.java:18` `MAX_QUANTUM = 32768` → `131072`（单次引擎额度上限 ×4）。

| 面 | A 侧 | Q 侧 | 差 |
|:--|--:|--:|--:|
| PASS | 264 | 264 | 0 |
| UNRESOLVED | 4 | 4 | 0 |
| UNSUPPORTED | 6 | 6 | 0 |
| 逐例 work 相同 | — | — | 222 / 274 |

**生效核验**：52 例 work 有差，最大 `+449.17%`（`cb3e61011487` 225,930 → 1,240,741）与 `-89.19%`（`706316fb15d8` 3,466,697 → 374,847）——补丁确实改变了调度路径。

**判读**：配额上限放大 4 倍后，**264 PASS / 4 UNRESOLVED / 6 UNSUPPORTED 逐条不变**；4 例 UNRESOLVED 仍用满约 19.99M。

## 五、旧 known-limits 对账（本轮关键）

`tests/known-limits.json` 的 87 条 model-corpus（`source_revision: e788d3d3`）与当前代际实跑逐例对上：

| 项 | 值 |
|:--|:--|
| 旧记录 detail 的 work 分布 | **87 / 87 落在 2.4M–2.6M**（全部是同一个截停） |
| 当前代际状态 | **PASS 79 · UNRESOLVED 4 · UNSUPPORTED 4** |
| 当前仍停在 2.5M 的 | **0 条** |

仍未 PASS 的 8 条（全新 work 值）：

| case | 当前 | 当前 work | 旧 work |
|:--|:--|--:|--:|
| `1204d21049c4…` | UNRESOLVED | 19,995,512 | 2,500,000 |
| `867653b0b246…` | UNRESOLVED | 19,997,420 | 2,500,068 |
| `898e00c6bbdb…` | UNRESOLVED | 19,995,399 | 2,500,000 |
| `3f6275678665…` | UNRESOLVED | 19,997,992 | 2,500,006 |
| `1d98538b025a…` | UNSUPPORTED | 0 | 2,500,270 |
| `a9d83be7942d…` | UNSUPPORTED | 0 | 2,500,000 |
| `b77762acc50f…` | UNSUPPORTED | 0 | 2,500,545 |
| `e4eb46e62f7d…` | UNSUPPORTED | 0 | 2,500,003 |

**判读**：2.5M 截停是**旧代际现象**，在当前 `HEAD b7c4a81` 上已不复现。因此：

- `m7-quota-census.md` §一「87 例算不出的共同特征……是同一个除法」——**依据过期，当前不成立**
- `analysis-scaffold.md` §3.2「中位数 2,500,001 与 20,000,000 / 8 一致」、§3.3「lcg-first 停在 1/8」——**基于 `e788d3d3` 的旧记录，不代表当前代际**
- M2 原案（`CountLcg` 1/8 改分层配额）的动机随之消失

## 六、对 M2 三靶面的结论

| 靶面 | 处数 | 本轮证据 | 判定 |
|:--|--:|:--|:--|
| A 裸固定分母 | 12 | 未逐处实测 | 无收益证据 |
| B 本地额度二次切 | 49 | 未逐处实测 | 无收益证据 |
| C 内存轴 | 53 | 未逐处实测 | 无收益证据 |
| ——同轴全局杠杆 | 1（`MAX_QUANTUM`） | 状态差 0/274 | 工作量配额「量」不影响结果 |

**当前真实剩余**：4 例 UNRESOLVED（用满 20M，属搜索难度或分支策略）+ 4 例 UNSUPPORTED（`main-counts conversion requires finite upper bounds`，属转换面）。二者都不是配额切分能解决的。

**建议（待裁）**：M2 暂停；先把 4 例 UNRESOLVED 归因（难度 / 启发式 / 分支顺序），4 例 UNSUPPORTED 属可移植边界之外，不改。

> **已执行**（2026-11-03）→ `m2-unresolved-attribution.md`：分支顺序否证（全程 1 个分支对象）· 1 例深度不足（1.68× 解出）· 3 例 5× 预算仍无 witness（启发式到顶）· 附表解释本轮「MAX_QUANTUM ×4 零状态变化」= `IntegerCountSearch:314` 的 4096 字面量窗口硬帽。

## 七、诚实边界

- 只覆盖 274 例 count-model；`standard` 412 项、graph 语料、models 之外的套件**未跑**
- 三靶面 A / B / C **未逐处实测**；「配额切分无收益」的直接证据只有 Q 探针（单次额度上限）一条，属同轴间接证据，不等于 A/B/C 每一处都零影响
- D「未生效」依据是 274 例 work 全同 + 代码结构复核，**未插桩计数**
- UNSUPPORTED 的 `work=0` 是脚本读法（该行无 work 字段），不是真实消耗 0
- 4 例 UNRESOLVED 的退出码 2 按 UNRESOLVED 记录，**未当 PASS、未当 UNSAT**
- 旧 known-limits 的 87 条全部命中 2.4M–2.6M 区间是字符串解析口径；区间外若有条目不在本次统计内（实测为 0）

## 八、复跑命令（cwd = `tools/CGSE_CORE`）

```
python .local\ab\prepare_ab.py                  # A 侧全量编译 + 代际核对
python .local\ab\build_d.py                     # D 变体（续跑窗口）
python .local\ab\build_q.py                     # Q 变体（MAX_QUANTUM）
python .local\ab\run_ab.py --arm base --bins 0,8,24,32,40,48,56,72   # 每批 8 例，120 s 窗口内分批
python .local\ab\run_ab.py --arm d --bins all
python .local\ab\run_ab.py --arm q --bins all
python .local\ab\diff_ab.py --right d
python .local\ab\diff_ab.py --right q
python .local\audit\diff_known_limits.py
python tools\release\check_abi.py --jar build\libs\cgse-core-0.3.2.jar --java runtime\jdk-17.0.2\bin\java.exe
```

ABI 基线（本轮末跑）：`PASS binary API 0.3: 1502 baseline entries preserved`。
