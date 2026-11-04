# M4 · G3 岔口呈报（c1 / c3 / 结案）

> 日期：2026-11-04 · 性质：**呈报（停在此处等拍板）**——`src/` 一行未改。
> 🔴 **拍板结果（同日晚）**：**取 c3（显式 note 收口）· c1 判死 · c2 不启**——依据是本轮新增的放宽实测（闸拿掉后 274 例状态 0 变化）。执行与前后对照 → `m4-g3-decision.md`。
> 依据：`m4-g2-g3.md` §五（c1 / c2 / c3 三案）· 拍板口径「c1 若开工需先做分组后跨组耦合的可行组合设计 + 健全性论证（不误判不可行）；到需要取舍的岔口停下呈报，不自行选」。
> 本轮产出：`.local/g2/out/g3-c1-design.md`（c1 设计 + 健全性论证）· `.local/g2/out/g3-table.md`（G3/G3b 各 27 例）· `.local/g2/out/g3-coupling.md`（分组后跨组耦合结构）· `.local/m4/out/m4-gate-census.md`（三闸分类口径，本轮复跑）。
> 硬约束沿用：`src/` 可改但不 push · 不改 LICENSE / NOTICE / 适配面 · 探针与中介落 `.local/` · 回归不绿即回滚。

---

## 一、本轮做完了什么

| 项 | 状态 | 证据 |
|:--|:--|:--|
| 三闸 census 复跑（G3/G3b 分类口径） | ✅ 274 例全量 | `.local/m4/out/m4-gate-census.md`：G1 0/0/0 · G2 22（∩U 2）· G3 84（∩U 27）· G3b 44（∩U 27） |
| c1 的「分组后跨组耦合」可行组合设计 | ✅ 已写 | `.local/g2/out/g3-c1-design.md` §二 / §三 |
| c1 的健全性论证（不误判不可行） | ✅ 已写 | 同上 §四 |
| c2 判否复核 | ✅ 不再启 | 27/27 例变量子集存在重叠（重叠对 82–2592）· 本轮另测分组后**变量连通分量数 27/27 = 1** |
| models 274 例当前字节复跑 | ✅ PASS 264 · UNRESOLVED 4 · UNSUPPORTED 6 | `.local/g2/out/models-rerun-summary.json`（9 片 · `main-counts` · 20M work） |
| core 全量回归 | ✅ 389 ran / 389 PASS / 0 FAIL（唯一测试名 385） | `.local/g2/summary_chunks.py` 汇总（24 块） |
| ABI 门禁 | ✅ PASS binary API 0.3: 1502 baseline entries preserved | `tools/release/check_abi.py` |
| c1 开工 | ⛔ **未开工**（按拍板停在取舍处） | — |

---

## 二、三案对照（呈报主体）

| 案 | 内容 | 改码量 | 语料级预期 | 风险 | 状态 |
|:--|:--|:--|:--|:--|:--|
| **c1** 多 shape 枚举 | 按 shape 分组，每组独立走 faces 路径；组间需新增松弛 / 冲突传播 | 新结构（组 → 组传播 + 三态剪枝 + 覆盖计数） | 理论上限 27 例（G3∩U）→ **当前字节净面 2 例** | 高——引入「局部无解」语义即引入误判不可行面 | **待拍板** |
| **c2** 变量不相交分组 | 仅当多变量 shape 的变量子集两两互斥时可分组求解 | — | **0 例可分组** | — | ✅ **已实测判否，不再启** |
| **c3** 不扩面 · 显式 note | 保留拒绝，把「多 shape 共因」写进诊断面 | 0（M1c v0.12 已接线具名 note + `refusal()` 齐平） | 解出增量 0；收益 = 归因可读性 | 零 | ✅ **已落地**（本轮补语料级留档） |
| **结案** | G3 / G3b 判为结构性未决面，本支不再推进 | 0 | 0 | 零 | **待拍板** |

---

## 三、三条支撑证据

### 3.1 c1 的分解形态在语料上不成立（新探针 `G3ComponentProbe` · 274 例全量）

- **变量连通分量数：27/27 例均为 1**——按 shape 签名分组后，多变量 shape 在变量层面仍是**一个耦合体**，不存在可独立求解的分组切分。残余耦合度 100%。
- 单个分量的规模：组数 16–297 · 变量数 27–143 · 共享变量的组对（冲突边）164–5184。
- c2 与 c1 败于同一处：这些例子的多变量 shape 在变量层面连通。c2 要求「变量子集两两互斥」不成立；c1 若沿用同一分解思路要求的「分组后可独立求解」同样不成立。
- **c1 的剩余形态只能是「单分量内的组间组合搜索」**——不分解，在 16–297 组 / 164–5184 冲突边的图上做组合枚举。20M work 预算下该空间的覆盖性**无法论证**。

### 3.2 c1 的真实收益面是 2 例，不是 27 例

census 的 G3∩UNRESOLVED 27 例来自 v0.4 M0 旧快照。本轮在**当前字节**上重跑 models 274 例：

| 闸 | census 旧快照 ∩U | 当前字节复跑 | 当前仍 UNRESOLVED |
|:--|--:|--:|:--|
| G3 `shape:faces_not_equal` | 27 | PASS 25 · UNRESOLVED 2 | `1204d21049c4`（work 19,995,512）· `898e00c6bbdb`（work 19,995,399） |
| G3b `faces_unresolved` | 27 | PASS 27 · UNRESOLVED 0 | — |

两例 work 均触顶 20M——是深度不足，不是形状闸独有。

### 3.3 G3 与 G3b 不是「连动面 / 共因」（上轮口径勘误）

实测 **G3∩U 与 G3b∩U 交集 = 0**（互斥，各 27 例，合计 54 例未决）。互斥是结构性的：`faces_unresolved` 只在 `prepareFaces` 成功后进入 `stepFaces` 才可能发生；被 `shape:faces_not_equal` 拒掉的例到不了 faces 尝试。二者是**两个不同覆盖面**的闸——c1 语义只覆盖 G3 侧，G3b 侧（16 次尝试封顶）不在其内。

---

## 四、c1 若开工的健全性义务（写给拍板看）

c1 的主要成本不在枚举，而在「如何不把未决误报为不可行」。必须先满足：

1. **三态返回**——每个组的子搜索返回 {witness / 该组已完整覆盖且无解 / 预算未决}；第三态必须向上传播为整例「未决」，禁止折叠为无解。
2. **剪枝只认完整覆盖**——只有「该组在给定松弛下已被完整枚举且无一解」才允许剪枝；每次剪枝必须留可核验的覆盖计数，未达覆盖即判未决。
3. **不可行证书独立**——整例判 UNSAT 只能来自既有 canonical 不可行证明路径，不得由组的组合失败推出（项目判据 #4：预算耗尽 ≠ 不可行证明）。
4. **诊断面齐平**——组级 note 带组号 / 变量集 / 覆盖计数；整例 refusal 区分「多 shape 未覆盖」与「形状不等」。
5. **回归不变式**——扩面后 UNRESOLVED 集只能缩小、UNSAT 集不得新增（当前基线 274 例中 UNSUPPORTED 6）；新增 UNSAT 一律回滚。

现行代码侧已核实的不变式：`shape:faces_not_equal`（`:263-264`）与 `faces_unresolved`（`:327-330`）走 `budget.note("admission_refused", ...)` 并置 `refusal()`（`:381`）——是「准入被拒 / 未决」，不是不可行证书。census 侧佐证：G3 触发 84 例中 57 例终态 PASS，它们是在 Diophantine 拒绝后由后续引擎解出的，**无一例因 G3 被判不可行**。

---

## 五、推荐与代价（供参考，拍板权在 Rushiera）

**推荐 c3 + 结案**：c3 零改码零风险（已接线），把 54 例的 G3 / G3b 归因写进留档；c1 的净收益面是 2 例，代价是新结构 + 一套「不误判不可行」的证明义务，且分解形态已被实测否掉。

**若要开 c1**：建议缩到「只针对 2 例净面」——即按例（`1204d21049c4` / `898e00c6bbdb`）设计形状处理，而非通用分组枚举；但这两例的形状组规模是 72 / 71 组、冲突边 2386 / 1348，单例级设计仍需覆盖性论证。

**c2**：不再启。

---

## 六、诚实边界

- census 终态源仍是 v0.4 M0 旧快照；本轮用**当前字节 models 复跑**替代它作终态判据（`models-rerun-summary.json`），但 census 的**触发面**（轴串）仍来自旧快照口径的探针。
- `--suite all` 未跑：core（385 唯一）+ models（274）+ ABI（1502）已跑；view / oracles / fixtures / probes / corpus / benchmarks 未跑。
- `IntegerCountBranch:520`（DD/Diophantine 第二消费点）入口未覆盖——census 入口口径 = QuickSolve 基形；`sourceFace` / `saturate` 变体入口未覆盖。
- c1 的**组合搜索设计**只写到形态与义务层，未做原型、未量化剪枝强度——「无法论证覆盖性」是判断，不是实测结论。
- 本轮 `src/` **零改动**；工作区改动仅 `.local/` 下的探针、脚本与产物（`git status` 37 项改动为前轮 M1/M2/M4 遗留，本轮未新增源码改动）。
- `m4-g2-g3.md` §五 的「G3 / G3b 连动面」表述已被本轮实测否证（见 §3.3），该文件已加修正前置说明（保留当轮口径作决策轨迹）。

---

## 七、复跑

```text
cd D:\Mau\WorkSpace\tools\CGSE_CORE
python .local\m4\m4_gate_census.py --shard 0/9   # …逐片到 8/9
python .local\m4\m4_gate_census.py --summarize     # → .local/m4/out/m4-gate-census.md

runtime\jdk-17.0.2\bin\javac.exe -encoding UTF-8 -cp build\tests\classes -d .local\g2\probe-classes .local\g2\probe-src\org\cgse\core\G3ComponentProbe.java
python .local\g2\run_g3_components.py     # 274 例 → .local/g2/out/g3-components.tsv
python .local\g2\g3_coupling_table.py     # → .local/g2/out/g3-coupling.md
python .local\g2\g3_component_summary.py  # 分量数分布
python .local\g2\g3_cross_check.py        # G3 / G3b 集合关系
python .local\g2\g3_fallback_check.py     # G3 触发例兜底路径与终态分布
python .local\g2\g3_table.py              # → .local/g2/out/g3-table.md
python .local\g2\g3_current_cross.py      # 旧快照 ∩U × 当前字节复跑交叉

python .local\g2\run_models_shard.py <k> 9    # k=0..8 → .local/g2/models-<k>
python .local\g2\summarize_models_rerun.py    # → .local/g2/out/models-rerun-summary.json
python .local\g2\run_chunk.py <起> <块长>     # core 分块（24 块）
python .local\g2\summary_chunks.py            # core 汇总
python tools\release\check_abi.py --jar build\libs\cgse-core-0.3.2.jar --java runtime\jdk-17.0.2\bin\java.exe
```
