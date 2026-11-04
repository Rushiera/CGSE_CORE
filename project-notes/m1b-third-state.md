# M1b — 归因第三态（`CountContinuation.refusal()`）

> 范围：`src/main/java/org/cgse/core/` 五处接口与实现 + `tests/corpus/java/` 两个回放件；不改任何判定条件、阈值、配额除数
> 基线：`main@b7c4a81`（v0.3.2）+ v0.4 M6 + v0.5 M1 的未提交工作树
> 授权：`project.md` §裁决记录（上游源码可改，代价后置上游评审）· 约束全守：未 push · 未碰 GTLCore · 未动 `LICENSE` / `NOTICE`
> 形态：**A 案**（接口加 `default String refusal()`，实现类按需覆写）——13 个实现类无需改动即可编译；B 案（方法 + 各实现返回枚举）需 13 个类全改，父层强类型分支收益不抵改动面

---

## 一、要解决的问题

M1 把准入拒绝写进 `PlanningBudget.note("admission_refused", ...)`，但 note 是**预算的环形缓冲（32 条）**，父层在代码里读不到，只能事后翻日志。于是父层看到的三种终止形态完全同形：

| 实际形态 | 父层可见信号 |
|:--|:--|
| 准入拒绝（一条 work 都没跑） | `step()==true` · `paused()==false` · `counts()==null` · `infeasible()==false` |
| 配额耗尽（跑到上限） | 同上（`paused()` 可能为真，但循环退出条件把两者合并） |
| 搜索收敛（无解/已完成） | 同上 |

`CountModelReplay` 原文注释即为此：「A local admission refusal stays UNKNOWN.」`CountModelRegression` 打印 `UNRESOLVED`，退出码 2——**2 不可当 PASS、不可当 UNSAT**，但「为什么没答案」在这一层丢失。

## 二、改动面

| 文件 | 改动 | 行数 |
|:--|:--|:--|
| `CountContinuation.java` | 加 `default String refusal() { return ""; }`（空串 = 已准入） | +7 |
| `CountBoolean.java` | 字段 `refusal` · 准入门置值 · `@Override public String refusal()` | +3 |
| `CountDecisionDiagram.java` | 同上 ×2 个准入门（4 轴合流 + `domain_width>8_with_variables>12`） | +4 |
| `CountDiophantine.java` | 同上（3 轴合流门） | +3 |
| `CountDomainSearch.java` | 同上（`variables>1024` / `rows>4096`） | +3 |
| `CountViewSearch.java` | `Search.refusal` 捕获（在 `step()` 里读 specialist 的第三态）+ 类级 `refusal()` 汇总（按引擎名去重拼接） | +16 |
| `CountModelReplay.java` | `Result` 加 `refusal` 分量（保留二参构造，旧调用零改）· 反射读第三态（对 pre-channel 基线仍可编译）· `mitm-retained` / `views` 两条回传 | +15 |
| `CountModelRegression.java` | `REFUSED <原因>` 独立于 `UNRESOLVED` · 计数器 · SUMMARY 加 `refused=` · **退出码 2 语义不变**（refused 同样走 2） | +4 |

**两通道口径一致**：`refusal()` 的赋值点与 M1 的 `admission_refused` note 点**逐点对应**（CountBoolean 1、CountDecisionDiagram 2、CountDiophantine 1、CountDomainSearch 1）——同一处拒绝在接口与 note 里给出同一串。

## 三、观测实证（改前 / 改后，同一探针）

探针 `.local/audit/probe/org/cgse/core/AuditGate.java` 扩了第三态读取：`complete` 为真时按反射取 `refusal()`，非空则打印 `REFUSED(<原因>)`。

**构造级 · 全语料 274 例**（`count-model-0…256.bin`，work=20000000）

| 口径 | 改前（pre-channel 类） | 改后（本次） |
|:--|:--|:--|
| refusedDomain | 5 | **5** |
| refusedObbt | 52 | **52** |
| refusedSoft | 8 | **8** |
| refusedLcg | 0 | **0** |
| refusedAny | 81 | **81** |
| domain 列内容 | 一律 `REFUSED` | **5/5 带具名轴**：`size:variables>1024` · `size:variables>1024 size:rows>4096` |

**同行对照（例）**

```text
1d98538b025a  UNKNOWN  1690  4883  REFUSED(size:variables>1024 size:rows>4096)  REFUSED  REFUSED  ADMITTED
634987e49633  UNSAT    1699  1670  REFUSED(size:variables>1024)                 REFUSED  REFUSED  ADMITTED
```

**回放级 · 96 例**（`views` 引擎，work=20000000，两侧同一回放件）

| 批次 | 改前 PASS / UNRESOLVED | 改后 PASS / UNRESOLVED / REFUSED |
|:--|:--|:--|
| `count-model-0` | 23 / 9 | **23 / 9 / 0** |
| `count-model-32` | 28 / 4 | **28 / 4 / 0** |
| `count-model-64` | 25 / 7 | **25 / 7 / 0** |

逐例 `work=` 数值亦逐行一致——判定与搜索路径零变化。

## 四、回归

| 项 | 命令 | 结果 |
|:--|:--|:--|
| 编译（主源码） | `GradleMain -p . jar --offline` | **exit 0** |
| 编译（含测试/回放件） | `python tests\run.py --suite core --shard 0/4`（不带 `--no-compile`） | 编译通过（`build/tests/classes` 已刷新为新字节） |
| ABI | `python tools\release\check_abi.py --jar build\libs\cgse-core-0.3.2.jar --java …\java.exe` | **PASS binary API 0.3: 1502 baseline entries preserved** |
| core 套件 | `--no-compile --shard 0/4…3/4` | **184 项 PASS · 0 FAIL**（分片：50 / 45 / 44 / 45；每片在工具 120 s 窗口边界截断，未跑完全部项——覆盖数，不是「全绿」） |

## 五、诚实边界

- **接口只覆盖 13 个实现类，其中仅 4 类有准入门**。`CountObbt`（52 例拒绝）、`CountSoftSearch`（8 例）不实现该接口——它们的理由仍只在 note 通道里，父层代码读不到。M1b 不替代 note 通道，这一点与裁决时的边界一致。
- **静默拒绝点未纳入**：`CountDecisionDiagram` 的 `upper==null` / `width>64`、`CountDiophantine` 的 `upper==null` / `free.size()<2`、各实现类的 `tryReserve` 失败——这些 `complete = true` 既无 note 也无 `refusal`，本批**有意不动**（保持两通道逐点一致；补它们等于再改一轮 M1 的 30 处口径）。
- **回放级 `REFUSED` 分支在 274 例上未触发**（refused=0）：`views` 路径不构造这四个准入门引擎，只经 `retain()` 接管外来 frontier。分支存在且被编译，但缺真实触发样本——构造级由 AuditGate 验证 5/5，回放级尚无实锤。
- **A/B 的「改前」类目录是 pre-M1 编译物**（`runtime/out-core/classes`，2026-11-01）：因此上表 note 列（最后一列）不作为对照——它差的是 M1 的改动，不是 M1b 的。refusal 计数与回放状态列可作对照（M1 与 M1b 均不改判定）。
- **未跑全量**：core 四片各自被窗口截断；`--suite all` 未跑。

## 六、复跑

```text
runtime\jdk-17.0.2\bin\java.exe -classpath "runtime\gradle-8.8\lib\*" org.gradle.launcher.GradleMain -p . -g runtime\gradle-home jar --offline
python tests\run.py --java-home runtime\jdk-17.0.2 --suite core --shard 0/4
python tests\run.py --java-home runtime\jdk-17.0.2 --suite core --no-compile --shard 0/4
python tools\release\check_abi.py --jar build\libs\cgse-core-0.3.2.jar --java runtime\jdk-17.0.2\bin\java.exe
python .local\audit\build_model_bins.py
runtime\jdk-17.0.2\bin\javac.exe -encoding UTF-8 --release 17 -cp build\tests\classes -d .local\audit\out\probe-classes .local\audit\probe\org\cgse\core\AuditGate.java
runtime\jdk-17.0.2\bin\java.exe -cp "build\tests\classes;.local\audit\out\probe-classes" org.cgse.core.AuditGate .local\audit\out\bins\count-model-0.bin 20000000
runtime\jdk-17.0.2\bin\java.exe -ea -cp "build\tests\classes" org.cgse.core.CountModelRegression .local\audit\out\bins\count-model-0.bin 0 20000000 views
```

> 顺序依赖同 M1：探针读 `build/tests/classes`，改源码后先跑一次不带 `--no-compile` 的 `run.py`。


## 七、拍板记录（2026-11-03 · Rushiera）

| # | 决策 | 结论 | 落地 |
|:--:|:--|:--|:--|
| 1 | 下一步优先级 | **P1 Obbt 弃权 A/B**（M3 前置）——52/274 弃权面是当前最大可量化收益，纯跑测不改码 | 已列 `todo.md` §审计，下一轮首选 |
| 2 | 静默拒绝点（`CountDecisionDiagram` `upper==null`/`width>64` · `CountDiophantine` `upper==null`/`free<2` · 各实现类 `tryReserve` 失败） | **B 案·齐平**：补 `refusal()` 与 `admission_refused` note，让第三态完整 | 登记 `todo.md` §改造 `M1c 静默拒绝点补 note+refusal`；**代价已知**：改 M1 的 30 处口径 → 约 35 处，须重跑 M1 对照表与 AuditGate 274 例 |
