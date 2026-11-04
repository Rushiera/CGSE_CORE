# M1 — 拒绝语义可归因（改动 + 前后对照）

> 范围：`src/main/java/org/cgse/core/` 31 处准入闸补具名原因；不改判定逻辑、不改阈值、不改接口签名
> 口径修正（2026-11-05）：原记 30 处——v0.5 表把 `CountRangedKernel` 记作 ×2，实为 ×3（`variables!=2` / 域宽 / 行越界）。实测 `admission_refused` 共 54 处 = M1 31 + M1c 23（M1c 另有 1 处存量 note 补 refusal），与本表逐类吻合。
> 基线：`main@b7c4a81`（v0.3.2）+ v0.4 的 M6 改动未提交工作树
> 授权：`project.md` §裁决记录（结构不匹配就改结构）· 约束：不 push / 不碰 GTLCore / 不动 `LICENSE`·`NOTICE`

---

## 一、要解决的问题

`analysis-scaffold.md` §二量化：全库 342 处 `complete = true`，拒绝分支自身留痕 **35/342（10.2%）**。其中 11 处 ADMIT_STATIC「加预算也绕不开且无声」、20 处 ADMIT_MIXED「规模阈值与配额合流，日志分不出哪个触发」。

上层拿到的是「没答案，也没说为什么」——这是 `AGENTS.md` §一-4「失败必须可见」的同类病：上游对 **UNKNOWN** 有极严格的纪律（`public-input-contracts.txt:36-39`），对**准入拒绝**没有。

## 二、格式契约

统一通道：`PlanningBudget.note(stage, detail)`（既有机制，环形 32 条）。

```text
stage  = admission_refused
detail = owner=<类名> <axis>:<条件> [<axis>:<条件> ...]
```

`axis` 四值全枚举，可 grep、可计数：

| axis | 含义 | 典型条件 |
|:--|:--|:--|
| `size` | 规模超限 | `variables>64` · `rows>4096` · `terms>65536` |
| `shape` | 结构不匹配（该引擎本来就只接这类） | `variables!=2` · `not_binary` · `faces_not_equal` |
| `quota` | 配额不足 | `allowance<4096` |
| `memory` | 内存预留失败 | `tryReserve` 返回 false（既有代码路径，本轮未改） |

**ADMIT_MIXED 拆轴规则**：原 `a || b || c` 短路改为逐轴判定后拼接——语义等价（任一成立即拒），但一次报全所有成立轴。改动前「156 变量 + 962 行」只能报一个轴，改后两条都报。

## 三、改动面（31 处）

| 类 | 轴 | 处数 |
|:--|:--|--:|
| CountAffineLattice | size / shape | 3（构造器 3 轴 · `basis>48` · `exactEquations`） |
| CountBinPackingSearch | size | 2（3 轴合流 · `entries>32768`） |
| CountBoolean | size / shape / quota | 1 |
| CountBranchProbe | size / quota | 1 |
| CountCardinalitySearch | size / quota | 1 |
| CountCdcl | size / quota | 1（4 轴） |
| CountChainSchedule | 轮数 | 1（`rounds==32`） |
| CountCongruence | size / quota | 1 |
| CountCoverCuts | size / quota | 1 |
| CountDecisionDiagram | size / shape / quota | 2（4 轴合流 · `width>8`） |
| CountDiophantine | size / shape / quota | 2（3 轴合流 · `faces_not_equal`） |
| CountDomainSearch | size | 1 |
| CountHall | size / quota | 1 |
| CountHermite | size / quota | 1 |
| CountHittingSet | size / quota | 1 |
| CountObbt | size / quota | 1 |
| CountPacking | size / quota | 1 |
| CountPackingLpSearch | 迭代 | 1（`iterations>8192`） |
| CountPartition | shape / quota | 1 |
| CountProbing | size / quota | 1 |
| CountRangedKernel | shape / size | 3（`!=2` · 域宽 · 行越界） |
| CountSoftSearch | size / quota | 1 |
| CountSymmetry | size / shape / quota | 1 |
| GraphOutputView | size / quota | 1 |

**剔除 1 处**：`PlanSourceCycles.java:244`——那里的 `complete` 是 `for (int pass...)` 内的局部变量，不是引擎拒绝点。扫描脚本按名匹配误判。

## 四、观测实证（改前 / 改后）

探针：`.local/audit/probe/.../AuditGate.java`（构造级，不跑搜索）新增 `refusals()` 列——从 `budget.diagnostics()` 里抽 `admission_refused@` 条目。

```text
# 改前（同一案例）
1d98538b025a  UNKNOWN  1690  4883  REFUSED  REFUSED  REFUSED  ADMITTED

# 改后
1d98538b025a  UNKNOWN  1690  4883  REFUSED  REFUSED  REFUSED  ADMITTED
  owner=CountDomainSearch size:variables>1024 size:rows>4096 ;
  owner=CountObbt size:variables>64 size:rows>256 ;
  owner=CountSoftSearch size:variables>512
```

本批 96 例（`count-model-0/32/64.bin`）：`refusedObbt=16`、`refusedDomain=1`、`refusedSoft=1`——**每一例 REFUSED 都带具名条目，无一条只剩 REFUSED**。

## 五、回归

| 项 | 命令 | 结果 |
|:--|:--|:--|
| 编译 | `gradle jar`（`runtime/` 内 JVM） | **BUILD SUCCESSFUL in 4s** |
| ABI | `python tools\release\check_abi.py --jar build\libs\cgse-core-0.3.2.jar` | **PASS binary API 0.3: 1502 baseline entries preserved** |
| core 套件 | `python tests\run.py --suite core --no-compile --shard 0/4…3/4` | **139 项全 PASS · 0 FAIL**（每片在 120 s 工具窗口边界截断，未跑完全部项） |
| 构造级覆盖 | `AuditGate <bin> 20000000` | 拒绝归因 100%（本批 96 例） |

**为什么 ABI 不破**：`CountContinuation` 与各引擎类都是**包私有**（`interface CountContinuation` 无 `public`），不在 `api/0.3.txt` 的公共签名面内（grep 该文件 0 命中）。

## 六、诚实边界

- **只加观测，未改判定**：30 处改动全部是「在既有 `complete = true` 之前写一条 note」，阈值、短路顺序、拒绝条件一字未动。改后 `refusedObbt` 计数与改前完全一致（16/16、52/274 口径不变）。
- **未跑完 core**：139 项 PASS 不等于全部项 PASS。每片被工具窗口截断，剩余项需更长窗口或 `--shard n/8`。
- **`--model-trace` 未实跑验证**：本轮以 `AuditGate`（构造级）取证，最终用户侧的 trace 通道未独立跑一遍。
- **环形缓冲上限 32 条**：长时间搜索会把早期准入拒绝挤掉（`PlanningBudget.java:394-399`）。补 note 只是让拒绝**可达**，不保证**长存**。要长存需扩缓冲或改落盘，属另一项改动。
- **`count-chain-schedule` 的 `rounds` 与 `iterations>8192` 不是规模阈值**，是收敛/迭代上限——语义上更接近「算法到顶」，本报表按字面归入 ADMIT_STATIC 的同一处理，上游若认定应另立类别，轴名可改。

## 七、复跑

```text
runtime\jdk-17.0.2\bin\java.exe -classpath "runtime\gradle-8.8\lib\*" org.gradle.launcher.GradleMain -p . -g runtime\gradle-home jar
python tests\run.py --java-home <repo>\runtime\jdk-17.0.2 --suite core --no-compile --shard 0/4
python .local\audit\build_model_bins.py
runtime\jdk-17.0.2\bin\javac.exe -encoding UTF-8 --release 17 -cp build\tests\classes -d .local\audit\out\probe-classes .local\audit\probe\org\cgse\core\AuditGate.java
runtime\jdk-17.0.2\bin\java.exe -cp "build\tests\classes;.local\audit\out\probe-classes" org.cgse.core.AuditGate .local\audit\out\bins\count-model-0.bin 20000000
python tools\release\check_abi.py --jar build\libs\cgse-core-0.3.2.jar
```

> 顺序依赖：`AuditGate` 读 `build/tests/classes`（`run.py` 编出的字节）。改源码后必须先跑一次 `run.py`（去掉 `--no-compile`），否则探针读旧字节。
