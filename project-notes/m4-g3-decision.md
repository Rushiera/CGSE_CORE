# M4 · G3 岔口拍板与落地（c3 收口 · c1 判死）

> 日期：2026-11-04 · 性质：**拍板 + 执行轮**——`src/` 改动一处（诊断行），回归全绿。
> 前序：`m4-g3-verdict.md`（呈报 · 三案 + 净收益面 2 例）· `.local/g2/out/g3-c1-design.md`（c1 设计与健全性义务）
> 拍板口径（Rushiera）：c1 若开工须先做「分组后跨组耦合可行组合设计 + 健全性论证」——已复用设计稿；本轮在此前提下实测拍板。
> 硬约束：`src/` 可改不 push · 不改 LICENSE / NOTICE / 适配面 · 探针与中介落 `.local/` · 回归不绿即回滚

---

## 一、拍板结论

**取 c3（显式 note 收口）；c1 判死不开工；c2 已实测判否不启。**

| 案 | 结论 | 依据 |
|:--|:--|:--|
| c1 多 shape 枚举 | **判死** | 把 `shape:faces_not_equal` 闸整个拿掉后，models 274 例**状态 0 例变化**（PASS 264 / UNRESOLVED 4 / UNSUPPORTED 6 逐例同），census 274 例 Diophantine SOLVED 26→26。闸不是阻塞点 |
| c2 变量不相交分组 | 不启（上轮判否） | 27/27 例变量子集重叠 |
| c3 显式 note 收口 | **采纳并落地** | 拒绝时补「多 shape 共因」诊断行（首个冲突行 + 两侧变量数）；274 例终态与 work **零变化** |

---

## 二、c1 判死的实测（本轮新增证据）

### 2.1 实验：放宽 G3 闸 → 全语料零变化

| 步 | 做法 |
|:--:|:--|
| 1 | 复制 `CountDiophantine.java` 到 `.local/g3/rel-src/`，把 `prepareFaces` 里 `shape:faces_not_equal` 的 `complete = true; return;` 删掉——只记 note 并继续走 faces 路径 |
| 2 | JDK 17 全量重编译整套 classes 到 `.local/g3/relax-1204/classes` |
| 3 | 同入口跑 models 274 例（`main-counts` · work 20M · 9 片）两遍：base vs relax |
| 4 | 另跑 census 探针（`M4GateProbe`）274 例，比对 Diophantine 终态与轴集合 |

### 2.2 结果表

| 口径 | base | relax | 差异 |
|:--|:--|:--|:--|
| models 274 例状态 | PASS 264 · UNRESOLVED 4 · UNSUPPORTED 6 | 同左 | **0 例状态变化**（两例目标 work 微差 1 / 262，状态同 UNRESOLVED） |
| census Diophantine SOLVED | 26 | 26 | 0 |
| census G3 轴命中 | 84 | 84 | 0 |
| 轴集合变化 | — | 77 例从 `shape:faces_not_equal` 挪到 `shape:faceChoices<2>` / `faces_unresolved` | 只换拒绝理由，不出解 |

两例目标（`1204d21049c4` · `898e00c6bbdb`）在 relax 字节下仍 UNRESOLVED，新拒绝理由是 `faces_unresolved`——闸后面的 faces 路径自身走不通。

### 2.3 两条佐证

- **预算不是瓶颈**：`1204d21049c4` 把 work 提到 **200M**（10×）仍 UNRESOLVED（work=199,995,657）。
- **不是形状问题是等式系统问题**：离线 RREF 探针（`.local/g3/out/solve-probe.md`）——两例 = **72 条 ±成对行合成等式（秩 72）+ 72 条剩余不等式 / 144 变量**；自由变量取下界回代得非整数/出盒点，剩余不等式违反 0 条。这类面要 LP 松弛 + 分支，不是「按 shape 分组枚举面」。

### 2.4 判死理由

c1 的核心假设是「形状不等挡住候选面，放开后多 shape 可枚举出解」。放开后**解出数增量 0**，两例 10× 预算也不动。完整 c1（分组 + 组间松弛 + 三态剪枝）要付 `m4-g3-verdict.md` §四 的五条健全性义务成本，换 0 收益。

---

## 三、c3 落地（本轮唯一改动）

**文件**：`workspace:tools/CGSE_CORE/src/main/java/org/cgse/core/CountDiophantine.java` · `prepareFaces`

拒绝时除既有 `admission_refused; owner=CountDiophantine shape:faces_not_equal` 外，再记一条：

```
count_diophantine: multi_shape_conflict; row=<首个冲突行序号>/<行数>; equation_vars=<候选面变量数>; row_vars=<冲突行变量数>
```

语义：拒绝原因是**形状冲突**（候选面只覆盖单一加权分摊），不是容量或预算；`row` 指出冲突落在哪一行。
诊断面从「只知道被拒」变成「知道被哪一行、以什么规模拒的」。

实跑样例（`1204d21049c4`，144 变量 / 降到 432 行）：
`count_diophantine@23723: multi_shape_conflict; row=3/432; equation_vars=9; row_vars=9`

---

## 四、前后对照数

| 门 | 结果 | 口径 |
|:--|:--|:--|
| core 全量 | **385 ran / 385 PASS / 0 FAIL**（唯一测试名 385，无重复覆盖） | `.local/g2/run_chunk.py` 24 块 + `.local/g3/summary_round.py` |
| models 274 例 | **PASS 264 · UNRESOLVED 4 · UNSUPPORTED 6**（逐例同 base · work 0 例变化） | `main-counts` · work 20M · 9 片 · `.local/g3/compare2.py` / `compare_work.py` |
| ABI 门禁 | **PASS binary API 0.3: 1502 baseline entries preserved** | `tools/release/check_abi.py` + 重打 `build/libs/cgse-core-0.3.2.jar`（gradle jar --offline · 7s） |
| jar 构建 | BUILD SUCCESSFUL | `runtime/gradle-8.8/bin/gradle.bat --offline jar` |

c3 是纯诊断增量：**274 例终态 0 变化 · 274 例 work 0 变化**。

---

## 五、诚实边界

- 放宽实验改的是**闸本身**（c1 最激进形态）；完整 c1（分组 + 组间松弛 + 三态剪枝）没做原型，因此「完整 c1 也零收益」是**推论**。反证门槛很低——闸放开只要多解一例就说明 c1 有面，实测一例都没有。
- models 274 例的 4 例 UNRESOLVED 中 2 例是本案两例；另 2 例不在 G3 轴内。
- 离线 RREF 只做「等式 + 自由变量取下界」一种取点策略，未做 LP 松弛/分支——它证明「两例不是面枚举问题」，不证明「不可解」。
- `--suite all` 未跑（view / oracles / fixtures / probes / corpus / benchmarks 未跑）；`IntegerCountBranch:520` 第二消费点未覆盖（与上轮同口径）。
- 本轮不 push；工作区 `src/` 仍含前轮 M1/M2/M4 遗留改动，本轮新增仅 `CountDiophantine` 的诊断行。
- **m-base 的可复跑口径**：base = 当前 `src` 去掉本轮新增的两行诊断 note（`count_diophantine: multi_shape_conflict…` 与注释），重编译即得；`m-base-*` 的日志是改前字节的实跑记录。
- core 全量 385 跑在 c3 首版字节上；其后改动仅注释措辞（中文→英文），重编译后复跑 Diophantine 相关 4 个测试类（`CountEquationResidueTest` / `CountEquationSlicesTest` / `CountProgressSchedulingTest` / `NumericContinuationTest`）全 PASS，jar 重打后 ABI 仍 1502。

---

## 六、复跑

```text
cd D:\Mau\WorkSpace\tools\CGSE_CORE

# c3 诊断面（census 探针 274 例）
runtime\jdk-17.0.2\bin\javac.exe -encoding UTF-8 -cp build\tests\classes -d .local\g3\probe-classes .local\m4\probe-src\org\cgse\core\M4GateProbe.java
python .local\g3\relax_probe.py build\tests\classes note          # → .local/g3/out/relax-probe-note.md

# 逐例前后对照（9 片）
python .local\g3\run_models_shard2.py <k> 9 build\tests\classes .local\g3\m-note-<k>
python .local\g3\compare2.py m-base m-note                        # 终态 0 差异
python .local\g3\compare_work.py                                  # work 0 差异

# c1 放宽实验
python .local\g3\run_models_shard2.py <k> 9 .local\g3\relax-1204\classes .local\g3\m-relax-<k>
python .local\g3\compare2.py m-base m-relax
# 补丁全文（唯一差异）→ .local\g3\rel-src\PATCH.md

# 回归三闸
python .local\g2\run_chunk.py <起> <块长> · python .local\g3\summary_round.py <epoch>
python tools\release\check_abi.py --jar build\libs\cgse-core-0.3.2.jar --java runtime\jdk-17.0.2\bin\java.exe

# 两例预算对照 + 等式结构
python tests\run.py --java-home runtime\jdk-17.0.2 --suite models --case 1204d21049c4 --work 200000000 --milliseconds 0 --model-engine main-counts --output .local\g3\w200m-1204
python .local\g3\solve_probe.py                                   # → .local/g3/out/solve-probe.md
```
