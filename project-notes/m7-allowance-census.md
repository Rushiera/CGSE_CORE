# M7 · allowance 分母形态全量表（机读）

> 生成：`scan_allowance_v2.py`（判定器，LHS 无关）· 源码根 `src/main/java/org/cgse/core` · 共 **312** 处 · 源码代际 `b7c4a81` + 32 文件未提交改动

> 判据：① 不在 Math.min 内 = 裸固定分母 · ② 在 Math.min 内 = 双限式（按另一侧实参细分）· ③ 除数非字面量 · ④ 分母不是请求剩余


## 一、形态小计

| 形态 | 处数 |
|:--|--:|
| ① 裸固定分母 | 12 |
| ②a 双限（另一侧=绝对常量） | 144 |
| ②b 双限（另一侧=父层额度） | 29 |
| ②c 双限（另一侧=本地剩余） | 12 |
| ③ 除数非字面量 | 4 |
| ④ 本地额度二次切 | 49 |
| ④ 字节面（非工作量） | 53 |
| ④ 总量切分 | 9 |
| **合计** | **312** |

## 二、引擎族分组

| 族 | 处数 | ①裸固定分母 | ②a双限（另一侧=绝对常量） | ②b双限（另一侧=父层额度） | ②c双限（另一侧=本地剩余） | ③除数非字面量 | ④本地额度二次切 | ④字节面（非工作量） | ④总量切分 | 除数集合 |
|:--|--:|--:|--:|--:|--:|--:|--:|--:|--:|--|
| Count* 其余 | 181 | 6 | 73 | 19 | 5 | 1 | 36 | 34 | 7 | 非字面量, 2, 3, 4, 6, 8, 16, 32, 64 |
| Graph* | 25 | 0 | 16 | 1 | 0 | 1 | 2 | 5 | 0 | 非字面量, 2, 3, 4, 8, 16 |
| Captured* | 22 | 4 | 10 | 4 | 0 | 0 | 2 | 0 | 2 | 2, 4, 8, 16 |
| CountLcg | 18 | 1 | 4 | 1 | 3 | 1 | 0 | 8 | 0 | 非字面量, 4, 8, 16, 32, 128 |
| Refill* | 17 | 0 | 8 | 0 | 0 | 1 | 3 | 5 | 0 | 非字面量, 2, 4, 8, 16 |
| IntegerCount* | 14 | 0 | 9 | 1 | 3 | 0 | 1 | 0 | 0 | 2, 4, 5, 8, 16, 32 |
| 单点杂项 | 10 | 1 | 4 | 2 | 0 | 0 | 3 | 0 | 0 | 2, 4, 8, 16, 32 |
| Exact* | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0 | 0 | 4, 16, 64 |
| MissingMaterialExpansion | 3 | 0 | 2 | 0 | 0 | 0 | 0 | 1 | 0 | 4, 16 |
| Plan* | 3 | 0 | 3 | 0 | 0 | 0 | 0 | 0 | 0 | 16, 32, 64 |
| Seed* | 3 | 0 | 2 | 0 | 0 | 0 | 1 | 0 | 0 | 8, 16 |
| State* | 3 | 0 | 3 | 0 | 0 | 0 | 0 | 0 | 0 | 16, 32 |
| CountSoftSearch | 2 | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 3, 16 |
| Quantity* | 2 | 0 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 8 |
| Region* | 2 | 0 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 4, 8 |
| CountJump | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 8 |
| CountObbt | 1 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 32 |

## 三、逐处明细

| # | 位置 | 族 | 分母对象 | 除数 | 形态 | 上限来源 |
|--:|:--|:--|:--|--:|:--|:--|
| 1 | `BackwardCoverability.java:313` | 单点杂项 | `allowance` | 4 | ④ 本地额度二次切 | `L1=8192` |
| 2 | `BackwardCoverability.java:320` | 单点杂项 | `allowance` | 2 | ④ 本地额度二次切 | `L1=16384` |
| 3 | `CapturedCatalog.java:421` | Captured* | `budget.remainingWork()` | 2 | ① 裸固定分母 | `—` |
| 4 | `CapturedCatalog.java:628` | Captured* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 5 | `CapturedFlowAllocation.java:59` | Captured* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 6 | `CapturedFlowAllocation.java:93` | Captured* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 7 | `CapturedFlowWitness.java:38` | Captured* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 8 | `CapturedInputAllocation.java:96` | Captured* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 9 | `CapturedProposalRepair.java:70` | Captured* | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=65_536 ； L2=maximumWork` |
| 10 | `CapturedProposalRepair.java:124` | Captured* | `allowance` | 2 | ④ 本地额度二次切 | `L1=32_768` |
| 11 | `CapturedReachableWitness.java:63` | Captured* | `budget.remainingWork()` | 8 | ① 裸固定分母 | `—` |
| 12 | `CapturedRefillRecovery.java:44` | Captured* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=1_048_576` |
| 13 | `CapturedStartupVerification.java:72` | Captured* | `budget.remainingWork()` | 2 | ②a 双限（另一侧=绝对常量） | `L1=131072` |
| 14 | `CapturedStateExpansion.java:64` | Captured* | `budget.remainingWork()` | 8 | ① 裸固定分母 | `—` |
| 15 | `CapturedStateExpansion.java:279` | Captured* | `allowance` | 16 | ④ 本地额度二次切 | `L1=8192` |
| 16 | `CapturedStatePlanning.java:132` | Captured* | `budget.remainingWork()` | 2 | ① 裸固定分母 | `—` |
| 17 | `CapturedStatePlanning.java:139` | Captured* | `budget.remainingWork()` | 2 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 18 | `CapturedStatePlanning.java:912` | Captured* | `choiceRepair.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=2048` |
| 19 | `CapturedStatePlanning.java:1183` | Captured* | `choiceRepair.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=1024` |
| 20 | `CapturedStateSupport.java:82` | Captured* | `maximumWork` | 8 | ④ 总量切分 | `L1=budget.remainingWork() / 16 ； L2=8192` |
| 21 | `CapturedStateSupport.java:82` | Captured* | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=maximumWork / 8 ； L2=8192` |
| 22 | `CapturedStateSupport.java:83` | Captured* | `maximumWork` | 4 | ④ 总量切分 | `L1=budget.remainingWork() / 8 ； L2=262_144` |
| 23 | `CapturedStateSupport.java:83` | Captured* | `budget.remainingWork()` | 8 | ②b 双限（另一侧=父层额度） | `L1=maximumWork / 4 ； L2=262_144` |
| 24 | `CapturedStateSupport.java:84` | Captured* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=65_536` |
| 25 | `ConservationAnalysis.java:40` | 单点杂项 | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=1_000_000` |
| 26 | `ConservationKernel.java:62` | 单点杂项 | `allowance` | 16 | ④ 本地额度二次切 | `—` |
| 27 | `ConservationKernel.java:62` | 单点杂项 | `budget.remainingWork()` | 32 | ① 裸固定分母 | `—` |
| 28 | `ContinuousCoverability.java:37` | 单点杂项 | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=750_000` |
| 29 | `CountAffineLattice.java:70` | Count* 其余 | `budget.remainingWork()` | 32 | ②b 双限（另一侧=父层额度） | `L1=131_072 ； L2=maximumWork` |
| 30 | `CountBoolean.java:91` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=262_144` |
| 31 | `CountBounds.java:199` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 32 | `CountBounds.java:240` | Count* 其余 | `allowance` | 4 | ④ 本地额度二次切 | `—` |
| 33 | `CountBounds.java:243` | Count* 其余 | `allowance` | 8 | ④ 本地额度二次切 | `L1=16384` |
| 34 | `CountBounds.java:265` | Count* 其余 | `allowance` | 4 | ④ 本地额度二次切 | `—` |
| 35 | `CountBounds.java:331` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 36 | `CountBounds.java:341` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 37 | `CountBounds.java:395` | Count* 其余 | `allowance` | 4 | ④ 本地额度二次切 | `—` |
| 38 | `CountBounds.java:404` | Count* 其余 | `allowance` | 4 | ④ 本地额度二次切 | `—` |
| 39 | `CountBounds.java:544` | Count* 其余 | `budget.remainingWork()` | 8 | ②c 双限（另一侧=本地剩余） | `L1=continuationLimit - allowance` |
| 40 | `CountBranchProbe.java:65` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 41 | `CountCanonicalModel.java:46` | Count* 其余 | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=MAX_WORK` |
| 42 | `CountCatalog.java:68` | Count* 其余 | `budget.remainingWork()` | 32 | ① 裸固定分母 | `—` |
| 43 | `CountCdcl.java:142` | Count* 其余 | `budget.remainingWork()` | 8 | ②b 双限（另一侧=父层额度） | `L1=maxWork` |
| 44 | `CountCdcl.java:575` | Count* 其余 | `allowance` | 32 | ④ 本地额度二次切 | `L1=16384` |
| 45 | `CountCdcl.java:671` | Count* 其余 | `allowance` | 32 | ④ 本地额度二次切 | `L1=32768` |
| 46 | `CountCdcl.java:1047` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=16384` |
| 47 | `CountCdcl.java:1052` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 48 | `CountChainSchedule.java:56` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=131072` |
| 49 | `CountCliques.java:41` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 50 | `CountComponents.java:49` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=524_288` |
| 51 | `CountCongruence.java:28` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=262_144` |
| 52 | `CountConservationPools.java:20` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 53 | `CountCoreMinimize.java:42` | Count* 其余 | `budget.remainingWork()` | 32 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 54 | `CountCoreSoft.java:34` | Count* 其余 | `budget.remainingWork()` | 32 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 55 | `CountCoverCuts.java:51` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 56 | `CountCoverCuts.java:83` | Count* 其余 | `allowance` | 4 | ④ 本地额度二次切 | `—` |
| 57 | `CountCyclicStockSearch.java:48` | Count* 其余 | `budget.remainingWork()` | 2 | ②a 双限（另一侧=绝对常量） | `L1=8_388_608` |
| 58 | `CountDecisionDiagram.java:42` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 59 | `CountDecisionDiagram.java:43` | Count* 其余 | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 60 | `CountDemandSearch.java:55` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=2_000_000` |
| 61 | `CountDemandSearch.java:132` | Count* 其余 | `allowance` | 4 | ④ 本地额度二次切 | `—` |
| 62 | `CountDifference.java:18` | Count* 其余 | `budget.remainingWork()` | 32 | ②b 双限（另一侧=父层额度） | `L1=Math.min(8192, maximumWork)` |
| 63 | `CountDiophantine.java:41` | Count* 其余 | `budget.remainingWork()` | 32 | ②b 双限（另一侧=父层额度） | `L1=131_072 ； L2=maximumWork` |
| 64 | `CountDiving.java:42` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 65 | `CountDiving.java:60` | Count* 其余 | `allowance` | 4 | ④ 本地额度二次切 | `—` |
| 66 | `CountDiving.java:130` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `L1=4L << 20` |
| 67 | `CountExecution.java:62` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 68 | `CountFeasibilityPump.java:31` | Count* 其余 | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 69 | `CountGomory.java:27` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=basis.revised == null ? 32768 : 131072` |
| 70 | `CountGroupedPartition.java:91` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 71 | `CountGroupedPartition.java:117` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=4_194_304` |
| 72 | `CountGroupedPartition.java:118` | Count* 其余 | `maximumWork` | 8 | ④ 总量切分 | `L1=32768` |
| 73 | `CountGroupedPartition.java:350` | Count* 其余 | `budget.availableBytes()` | 4 | ④ 字节面（非工作量） | `—` |
| 74 | `CountGroupedPartition.java:352` | Count* 其余 | `budget.availableBytes()` | 4 | ④ 字节面（非工作量） | `—` |
| 75 | `CountHall.java:38` | Count* 其余 | `budget.remainingWork()` | 64 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 76 | `CountHall.java:58` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 77 | `CountHall.java:61` | Count* 其余 | `allowance` | 4 | ④ 本地额度二次切 | `L1=8192` |
| 78 | `CountHermite.java:26` | Count* 其余 | `budget.remainingWork()` | 64 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 79 | `CountHittingSet.java:48` | Count* 其余 | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 80 | `CountImplications.java:38` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=65_536` |
| 81 | `CountImplications.java:65` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 82 | `CountImplications.java:193` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 83 | `CountInduction.java:56` | Count* 其余 | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 84 | `CountIntegerLpLearning.java:27` | Count* 其余 | `budget.remainingWork()` | 64 | ②a 双限（另一侧=绝对常量） | `L1=16384` |
| 85 | `CountIntegerLpLearning.java:65` | Count* 其余 | `allowance` | 3 | ④ 本地额度二次切 | `—` |
| 86 | `CountIntegerLpLearning.java:150` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 87 | `CountIntegerLpLearning.java:195` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 88 | `CountInterfaceSearch.java:62` | Count* 其余 | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 89 | `CountInterfaceSearch.java:272` | Count* 其余 | `budget.availableBytes()` | 2 | ④ 字节面（非工作量） | `—` |
| 90 | `CountJump.java:97` | CountJump | `budget.remainingWork()` | 8 | ②c 双限（另一侧=本地剩余） | `L1=allowance` |
| 91 | `CountKernelSearch.java:44` | Count* 其余 | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 92 | `CountLatticeRepair.java:39` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=1_000_000` |
| 93 | `CountLcg.java:331` | CountLcg | `budget.availableBytes()` | 16 | ④ 字节面（非工作量） | `L1=budget.remainingWork() / Math.max(1, 8L * low.length) ； L2=2048` |
| 94 | `CountLcg.java:332` | CountLcg | `budget.remainingWork()` | 非字面量 | ③ 除数非字面量 | `L1=budget.availableBytes() / 16 / perCut ； L2=2048` |
| 95 | `CountLcg.java:405` | CountLcg | `budget.remainingWork()` | 8 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 96 | `CountLcg.java:527` | CountLcg | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 97 | `CountLcg.java:656` | CountLcg | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 98 | `CountLcg.java:727` | CountLcg | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 99 | `CountLcg.java:751` | CountLcg | `budget.availableBytes()` | 4 | ④ 字节面（非工作量） | `—` |
| 100 | `CountLcg.java:836` | CountLcg | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 101 | `CountLcg.java:859` | CountLcg | `budget.remainingWork()` | 16 | ②c 双限（另一侧=本地剩余） | `L1=remaining ； L2=Math.max(0, allowance - work)` |
| 102 | `CountLcg.java:864` | CountLcg | `budget.remainingWork()` | 16 | ① 裸固定分母 | `—` |
| 103 | `CountLcg.java:886` | CountLcg | `budget.remainingWork()` | 16 | ②c 双限（另一侧=本地剩余） | `L1=remaining - (before - preparationStarted) ； L2=Math.max(0, allowance - work)` |
| 104 | `CountLcg.java:921` | CountLcg | `budget.remainingWork()` | 8 | ②c 双限（另一侧=本地剩余） | `L1=Math.max(0, allowance - work) ； L2=65536` |
| 105 | `CountLcg.java:929` | CountLcg | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=200000` |
| 106 | `CountLcg.java:963` | CountLcg | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 107 | `CountLcg.java:1980` | CountLcg | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 108 | `CountLcg.java:1998` | CountLcg | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=16384` |
| 109 | `CountLcg.java:2026` | CountLcg | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 110 | `CountLcgReliability.java:92` | CountLcg | `budget.remainingWork()` | 128 | ②a 双限（另一侧=绝对常量） | `L1=ordinaryWork / 64 - probeWork ； L2=4096` |
| 111 | `CountLockBranch.java:40` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 112 | `CountLockBranch.java:141` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 113 | `CountLpCoreSearch.java:42` | Count* 其余 | `budget.remainingWork()` | 8 | ②b 双限（另一侧=父层额度） | `L1=MAX_WORK` |
| 114 | `CountLpCoreSearch.java:141` | Count* 其余 | `allowance` | 4 | ④ 本地额度二次切 | `—` |
| 115 | `CountLpCoreSearch.java:159` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 116 | `CountLpProposal.java:29` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=262144` |
| 117 | `CountLpProposal.java:59` | Count* 其余 | `budget.remainingWork()` | 4 | ②c 双限（另一侧=本地剩余） | `L1=allowance` |
| 118 | `CountLpProximity.java:82` | Count* 其余 | `budget.remainingWork()` | 非字面量 | ③ 除数非字面量 | `L1=priceOrdered ? 524288 : 131072` |
| 119 | `CountLpProximity.java:171` | Count* 其余 | `budget.availableBytes()` | 4 | ④ 字节面（非工作量） | `—` |
| 120 | `CountLpSearch.java:58` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 121 | `CountLpSearch.java:109` | Count* 其余 | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=262144` |
| 122 | `CountLpStructure.java:45` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 123 | `CountMeetInMiddle.java:190` | Count* 其余 | `budget.availableBytes()` | 4 | ④ 字节面（非工作量） | `—` |
| 124 | `CountMeetInMiddle.java:190` | Count* 其余 | `budget.remainingWork()` | 2 | ① 裸固定分母 | `—` |
| 125 | `CountMeetInMiddle.java:196` | Count* 其余 | `budget.remainingWork()` | 3 | ②a 双限（另一侧=绝对常量） | `L1=shape.work() + lattice + Math.max(262144, lattice / 4)` |
| 126 | `CountMeetInMiddle.java:206` | Count* 其余 | `budget.availableBytes()` | 4 | ④ 字节面（非工作量） | `—` |
| 127 | `CountMeetInMiddle.java:206` | Count* 其余 | `budget.remainingWork()` | 2 | ① 裸固定分母 | `—` |
| 128 | `CountMeetInMiddle.java:215` | Count* 其余 | `budget.availableBytes()` | 4 | ④ 字节面（非工作量） | `—` |
| 129 | `CountMeetInMiddle.java:215` | Count* 其余 | `budget.remainingWork()` | 16 | ① 裸固定分母 | `—` |
| 130 | `CountMeetInMiddle.java:219` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=2 * estimatedWork` |
| 131 | `CountMeetInMiddle.java:278` | Count* 其余 | `budget.remainingWork()` | 3 | ②b 双限（另一侧=父层额度） | `L1=maxWork` |
| 132 | `CountModelViews.java:248` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 133 | `CountModelViews.java:380` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 134 | `CountModelViews.java:391` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 135 | `CountNeighborhood.java:107` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=131_072` |
| 136 | `CountNetwork.java:70` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=131072` |
| 137 | `CountNetwork.java:89` | Count* 其余 | `allowance` | 4 | ④ 本地额度二次切 | `L1=16384` |
| 138 | `CountNumericRelaxation.java:59` | Count* 其余 | `maximumWork` | 8 | ④ 总量切分 | `—` |
| 139 | `CountNumericRelaxation.java:94` | Count* 其余 | `maximumWork` | 8 | ④ 总量切分 | `—` |
| 140 | `CountNumericRelaxation.java:246` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 141 | `CountNumericRelaxation.java:341` | Count* 其余 | `budget.remainingWork()` | 4 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 142 | `CountObbt.java:27` | CountObbt | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 143 | `CountPacking.java:34` | Count* 其余 | `budget.remainingWork()` | 6 | ②a 双限（另一侧=绝对常量） | `L1=2_000_000` |
| 144 | `CountPackingCore.java:93` | Count* 其余 | `budget.remainingWork()` | 3 | ②a 双限（另一侧=绝对常量） | `L1=12_582_912` |
| 145 | `CountPackingCore.java:94` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=2_097_152` |
| 146 | `CountPackingCore.java:95` | Count* 其余 | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=8_388_608` |
| 147 | `CountPackingCore.java:276` | Count* 其余 | `budget.remainingWork()` | 3 | ②a 双限（另一侧=绝对常量） | `L1=12_582_912` |
| 148 | `CountPackingCore.java:290` | Count* 其余 | `maximumWork` | 16 | ④ 总量切分 | `L1=1_048_576 ； L2=normalWork` |
| 149 | `CountPackingCore.java:422` | Count* 其余 | `budget.remainingWork()` | 2 | ②a 双限（另一侧=绝对常量） | `L1=8_388_608 ； L2=normalWork` |
| 150 | `CountPartition.java:48` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=524_288L` |
| 151 | `CountPlaceSets.java:24` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=16384` |
| 152 | `CountProbing.java:37` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=131_072` |
| 153 | `CountProbing.java:137` | Count* 其余 | `budget.remainingWork()` | 8 | ②c 双限（另一侧=本地剩余） | `L1=allowance - work ； L2=16_384` |
| 154 | `CountProgramSummaries.java:55` | Count* 其余 | `budget.availableBytes()` | 16 | ④ 字节面（非工作量） | `—` |
| 155 | `CountRangeIndex.java:59` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 156 | `CountRecovery.java:59` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=262144` |
| 157 | `CountRecovery.java:71` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=262144` |
| 158 | `CountRecovery.java:189` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 159 | `CountRecovery.java:255` | Count* 其余 | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=4_000_000` |
| 160 | `CountRecovery.java:256` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=1_000_000` |
| 161 | `CountRecovery.java:257` | Count* 其余 | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=4_000_000` |
| 162 | `CountRecoveryFuel.java:34` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 163 | `CountRecoveryFuel.java:191` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=524288` |
| 164 | `CountRecoveryFuel.java:325` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=131072` |
| 165 | `CountRecoveryFuel.java:326` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 166 | `CountRecoveryShape.java:35` | Count* 其余 | `budget.remainingWork()` | 16 | ① 裸固定分母 | `—` |
| 167 | `CountReducedCost.java:29` | Count* 其余 | `budget.remainingWork()` | 32 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 168 | `CountReduction.java:73` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=131_072` |
| 169 | `CountReduction.java:200` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 170 | `CountReduction.java:344` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 171 | `CountReduction.java:360` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 172 | `CountReduction.java:441` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 173 | `CountReduction.java:461` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 174 | `CountReduction.java:462` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 175 | `CountReduction.java:502` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 176 | `CountReduction.java:517` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 177 | `CountReduction.java:519` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 178 | `CountReduction.java:567` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 179 | `CountReduction.java:575` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 180 | `CountRelaxationNeighborhood.java:48` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=4_194_304` |
| 181 | `CountRelaxationNeighborhood.java:60` | Count* 其余 | `maximumWork` | 2 | ④ 总量切分 | `—` |
| 182 | `CountRelaxationNeighborhood.java:62` | Count* 其余 | `maximumWork` | 2 | ④ 总量切分 | `L1=262144 ； L2=budget.remainingWork()` |
| 183 | `CountRelaxationNeighborhood.java:98` | Count* 其余 | `maximumWork` | 4 | ④ 总量切分 | `L1=524288` |
| 184 | `CountResiduePresolve.java:33` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=16_384` |
| 185 | `CountReverseSchedule.java:38` | Count* 其余 | `budget.remainingWork()` | 2 | ②a 双限（另一侧=绝对常量） | `L1=1_048_576` |
| 186 | `CountRoundingCuts.java:35` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 187 | `CountRowSparsifier.java:34` | Count* 其余 | `budget.remainingWork()` | 64 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 188 | `CountRowSparsifier.java:45` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 189 | `CountRowTemplates.java:37` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 190 | `CountRowTemplates.java:192` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 191 | `CountScale.java:76` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=1_048_576` |
| 192 | `CountScale.java:80` | Count* 其余 | `budget.remainingWork()` | 3 | ②a 双限（另一侧=绝对常量） | `L1=8_388_608` |
| 193 | `CountScale.java:152` | Count* 其余 | `budget.remainingWork()` | 2 | ②a 双限（另一侧=绝对常量） | `L1=12_000_000` |
| 194 | `CountScheduleContinuations.java:124` | Count* 其余 | `budget.availableBytes()` | 16 | ④ 字节面（非工作量） | `—` |
| 195 | `CountScheduleOrder.java:23` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=16384` |
| 196 | `CountScheduleOrder.java:40` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 197 | `CountScheduleOrder.java:48` | Count* 其余 | `allowance` | 2 | ④ 本地额度二次切 | `—` |
| 198 | `CountScheduleWord.java:23` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=16_384` |
| 199 | `CountSeparator.java:62` | Count* 其余 | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=maxWork` |
| 200 | `CountSessions.java:162` | Count* 其余 | `budget.availableBytes()` | 16 | ④ 字节面（非工作量） | `—` |
| 201 | `CountShellCompilation.java:35` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=131072` |
| 202 | `CountSoftSearch.java:49` | CountSoftSearch | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 203 | `CountSoftSearch.java:107` | CountSoftSearch | `allowance` | 3 | ④ 本地额度二次切 | `L1=65536` |
| 204 | `CountStartup.java:45` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=65_536` |
| 205 | `CountStride.java:37` | Count* 其余 | `budget.remainingWork()` | 64 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 206 | `CountStructureSearch.java:30` | Count* 其余 | `budget.remainingWork()` | 3 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 207 | `CountSupportSearch.java:62` | Count* 其余 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=full ? 65_536 : 16_384` |
| 208 | `CountSupportSearch.java:278` | Count* 其余 | `allowance` | 8 | ④ 本地额度二次切 | `—` |
| 209 | `CountSupportSearch.java:308` | Count* 其余 | `allowance` | 8 | ④ 本地额度二次切 | `—` |
| 210 | `CountSymmetry.java:42` | Count* 其余 | `budget.remainingWork()` | 8 | ②c 双限（另一侧=本地剩余） | `L1=allowance` |
| 211 | `CountSymmetry.java:288` | Count* 其余 | `budget.remainingWork()` | 16 | ① 裸固定分母 | `—` |
| 212 | `CountSymmetry.java:307` | Count* 其余 | `allowance` | 4 | ④ 本地额度二次切 | `L1=65536` |
| 213 | `CountVectorNeighborhood.java:61` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=3 * REPAIR_WORK` |
| 214 | `CountVectorPartition.java:73` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 215 | `CountVectorPartition.java:97` | Count* 其余 | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=8_388_608` |
| 216 | `CountVectorPartition.java:443` | Count* 其余 | `budget.availableBytes()` | 4 | ④ 字节面（非工作量） | `—` |
| 217 | `CountViewConflicts.java:40` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 218 | `CountViewConflicts.java:103` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 219 | `CountViewConflicts.java:117` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 220 | `CountViewCuts.java:38` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 221 | `CountViewCuts.java:58` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 222 | `CountViewCuts.java:91` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 223 | `CountViewSearch.java:248` | Count* 其余 | `budget.remainingWork()` | 4 | ②c 双限（另一侧=本地剩余） | `L1=allowance` |
| 224 | `CountViewSearch.java:395` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=16384` |
| 225 | `CountViewSearch.java:431` | Count* 其余 | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 226 | `CountViewSearch.java:468` | Count* 其余 | `budget.remainingWork()` | 64 | ②a 双限（另一侧=绝对常量） | `L1=16384` |
| 227 | `CountViewSearch.java:858` | Count* 其余 | `budget.remainingWork()` | 3 | ②a 双限（另一侧=绝对常量） | `L1=packing.remainingAllowance()` |
| 228 | `CountViewSearch.java:863` | Count* 其余 | `budget.remainingWork()` | 3 | ②a 双限（另一侧=绝对常量） | `L1=packing.remainingAllowance()` |
| 229 | `CountViewSearch.java:864` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=4_194_304` |
| 230 | `CountViewSearch.java:914` | Count* 其余 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=2_097_152 ； L2=vector.remainingAllowance()` |
| 231 | `CountZeroHalfCuts.java:93` | Count* 其余 | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 232 | `ExactColumnProgram.java:28` | Exact* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=262144` |
| 233 | `ExactLinearProgram.java:242` | Exact* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=500_000` |
| 234 | `ExactLinearProgram.java:253` | Exact* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=2_000_000` |
| 235 | `ExactLinearProgram.java:320` | Exact* | `budget.remainingWork()` | 64 | ②a 双限（另一侧=绝对常量） | `L1=16384` |
| 236 | `ExactRevisedProgram.java:65` | Exact* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=750000` |
| 237 | `ForceCraftProof.java:33` | 单点杂项 | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=65536` |
| 238 | `ForwardCoverability.java:82` | 单点杂项 | `budget.remainingWork()` | 32 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 239 | `GraphCatalogIndex.java:93` | Graph* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=262144` |
| 240 | `GraphCatalogIndex.java:94` | Graph* | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `L1=8L << 20` |
| 241 | `GraphCompilation.java:245` | Graph* | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 242 | `GraphCompilation.java:313` | Graph* | `budget.availableBytes()` | 8 | ④ 字节面（非工作量） | `—` |
| 243 | `GraphFallback.java:135` | Graph* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=4096` |
| 244 | `GraphFallbackSearch.java:54` | Graph* | `budget.remainingWork()` | 3 | ②a 双限（另一侧=绝对常量） | `L1=43_690` |
| 245 | `GraphOutputView.java:68` | Graph* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=262144` |
| 246 | `GraphOutputView.java:129` | Graph* | `allowance` | 4 | ④ 本地额度二次切 | `L1=32768` |
| 247 | `GraphPlanningWork.java:178` | Graph* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=16384` |
| 248 | `GraphPlanningWork.java:279` | Graph* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=2_097_152` |
| 249 | `GraphPlanningWork.java:298` | Graph* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=phase == 12 ? 131_072 : phase == 6 || phase == 19 || !pending.isEmpty() ? 1_048_576 : 32_768` |
| 250 | `GraphPlanningWork.java:896` | Graph* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=1_048_576` |
| 251 | `GraphPlanningWork.java:952` | Graph* | `budget.remainingWork()` | 非字面量 | ③ 除数非字面量 | `L1=1_048_576` |
| 252 | `GraphPlanningWork.java:993` | Graph* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=1_048_576` |
| 253 | `GraphPlanningWork.java:1096` | Graph* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=262_144` |
| 254 | `GraphPlanningWork.java:1201` | Graph* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 255 | `GraphPlanningWork.java:1379` | Graph* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=Math.max(32_768, Math.min(1_048_576, firstGraphWork) * 2) ； L2=2_097_152` |
| 256 | `GraphPlanningWork.java:1388` | Graph* | `budget.availableBytes()` | 4 | ④ 字节面（非工作量） | `L1=33_554_432` |
| 257 | `GraphSourceIndex.java:49` | Graph* | `budget.availableBytes()` | 4 | ④ 字节面（非工作量） | `L1=16L << 20` |
| 258 | `GraphSourceRanking.java:34` | Graph* | `budget.remainingWork()` | 8 | ②b 双限（另一侧=父层额度） | `L1=393_216 ； L2=maximumWork` |
| 259 | `GraphSourceScout.java:22` | Graph* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=131_072` |
| 260 | `GraphStockViewPortfolio.java:38` | Graph* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=4_194_304` |
| 261 | `GraphStockViewPortfolio.java:39` | Graph* | `budget.remainingWork()` | 2 | ②a 双限（另一侧=绝对常量） | `L1=8_388_608` |
| 262 | `GraphStockViewPortfolio.java:81` | Graph* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=262144` |
| 263 | `GraphSupportNeighborhood.java:92` | Graph* | `allowance` | 2 | ④ 本地额度二次切 | `L1=65536` |
| 264 | `IntegerCountSearch.java:103` | IntegerCount* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=2_000_000` |
| 265 | `IntegerCountSearch.java:104` | IntegerCount* | `budget.remainingWork()` | 5 | ②a 双限（另一侧=绝对常量） | `L1=16_000_000` |
| 266 | `IntegerCountSearch.java:245` | IntegerCount* | `budget.remainingWork()` | 16 | ②c 双限（另一侧=本地剩余） | `L1=scoutMaximum - allowance ； L2=262144` |
| 267 | `IntegerCountSearch.java:256` | IntegerCount* | `allowance` | 16 | ④ 本地额度二次切 | `L1=budget.remainingWork() / 16 ； L2=262144` |
| 268 | `IntegerCountSearch.java:256` | IntegerCount* | `budget.remainingWork()` | 16 | ②c 双限（另一侧=本地剩余） | `L1=allowance / 16 ； L2=262144` |
| 269 | `IntegerCountSearch.java:479` | IntegerCount* | `budget.remainingWork()` | 2 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 270 | `IntegerCountSearch.java:743` | IntegerCount* | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=131072` |
| 271 | `IntegerCountSearch.java:770` | IntegerCount* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=1_000_000` |
| 272 | `IntegerCountSearch.java:805` | IntegerCount* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=524288` |
| 273 | `IntegerCountSearch.java:806` | IntegerCount* | `budget.remainingWork()` | 32 | ②c 双限（另一侧=本地剩余） | `L1=allowance - work ； L2=32768` |
| 274 | `IntegerCountSearch.java:841` | IntegerCount* | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=131072` |
| 275 | `IntegerCountSearch.java:858` | IntegerCount* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=2_097_152 ； L2=reachability.proofCompletionWork()` |
| 276 | `IntegerCountSearch.java:867` | IntegerCount* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=reachabilityMaximum - reachabilityWork ； L2=Math.max(32768, reachabilityWork)` |
| 277 | `IntegerCountSearch.java:931` | IntegerCount* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=262144` |
| 278 | `MissingMaterialExpansion.java:143` | MissingMaterialExpansion | `budget.availableBytes()` | 16 | ④ 字节面（非工作量） | `—` |
| 279 | `MissingMaterialExpansion.java:293` | MissingMaterialExpansion | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=Math.min(MIX_WORK_LIMIT, Math.max(512, Math.min(MIX_WORK_LIMIT, improvementStart - startedWork) * 4))` |
| 280 | `MissingMaterialExpansion.java:752` | MissingMaterialExpansion | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=Math.min(MIX_WORK_LIMIT, Math.max(512, Math.min(MIX_WORK_LIMIT, budget.nodes() - startedWork) * 4))` |
| 281 | `PdrCoverability.java:61` | 单点杂项 | `budget.remainingWork()` | 16 | ②b 双限（另一侧=父层额度） | `L1=maximumWork` |
| 282 | `PlanEGraph.java:42` | Plan* | `budget.remainingWork()` | 64 | ②a 双限（另一侧=绝对常量） | `L1=8192` |
| 283 | `PlanFlowPruning.java:46` | Plan* | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=32_768` |
| 284 | `PlanSourceCycles.java:30` | Plan* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=262144` |
| 285 | `QuantityAnalysis.java:320` | Quantity* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=16_384L` |
| 286 | `QuantityPotential.java:39` | Quantity* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=1_500_000` |
| 287 | `RefillPlanningSearch.java:83` | Refill* | `budget.remainingWork()` | 2 | ②a 双限（另一侧=绝对常量） | `L1=Math.min(8_000_000, Math.max(modelAllowance, started / 4))` |
| 288 | `RefillPlanningSearch.java:300` | Refill* | `allowance` | 非字面量 | ③ 除数非字面量 | `L1=524288 ； L2=allowance - (trialStarted - started)` |
| 289 | `RefillPlanningSearch.java:301` | Refill* | `allowance` | 4 | ④ 本地额度二次切 | `L1=allowance - (trialStarted - started)` |
| 290 | `RefillRescheduling.java:46` | Refill* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=16_384` |
| 291 | `RefillShortBlockHint.java:23` | Refill* | `budget.availableBytes()` | 16 | ④ 字节面（非工作量） | `—` |
| 292 | `RefillSourceSearch.java:95` | Refill* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=cap` |
| 293 | `RefillSourceSearch.java:96` | Refill* | `allowance` | 4 | ④ 本地额度二次切 | `L1=reallocate || cap > 8192 ? 8192 : 1024` |
| 294 | `RefillSupplySearch.java:76` | Refill* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=8192` |
| 295 | `RefillSupplySearch.java:436` | Refill* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=8192` |
| 296 | `RefillSupplySearch.java:444` | Refill* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=4096` |
| 297 | `RefillSupplySearch.java:458` | Refill* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=8192` |
| 298 | `RefillTerminalSearch.java:39` | Refill* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=4_194_304` |
| 299 | `RefillUnitPeriods.java:30` | Refill* | `budget.availableBytes()` | 16 | ④ 字节面（非工作量） | `L1=65536` |
| 300 | `RefillWordBlocks.java:32` | Refill* | `budget.availableBytes()` | 16 | ④ 字节面（非工作量） | `—` |
| 301 | `RefillWordNormalForms.java:50` | Refill* | `budget.availableBytes()` | 16 | ④ 字节面（非工作量） | `L1=262144` |
| 302 | `RefillWordPruning.java:261` | Refill* | `allowance` | 4 | ④ 本地额度二次切 | `L1=1024` |
| 303 | `RefillWordShapes.java:39` | Refill* | `budget.availableBytes()` | 16 | ④ 字节面（非工作量） | `L1=262144` |
| 304 | `RegionBootstrap.java:58` | Region* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=131_072` |
| 305 | `RegionCounts.java:63` | Region* | `budget.remainingWork()` | 4 | ②a 双限（另一侧=绝对常量） | `L1=262_144` |
| 306 | `SeedOptimization.java:76` | Seed* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=262_144` |
| 307 | `SeedOptimization.java:403` | Seed* | `this.allowance` | 16 | ④ 本地额度二次切 | `L1=16384` |
| 308 | `SeedPortfolio.java:122` | Seed* | `budget.remainingWork()` | 8 | ②a 双限（另一侧=绝对常量） | `L1=262_144` |
| 309 | `SourceExplanation.java:39` | 单点杂项 | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=32768` |
| 310 | `StateChoiceRepair.java:31` | State* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=32_768` |
| 311 | `StateEntityMinCost.java:54` | State* | `budget.remainingWork()` | 32 | ②a 双限（另一侧=绝对常量） | `L1=4096` |
| 312 | `StateScheduleRepair.java:30` | State* | `budget.remainingWork()` | 16 | ②a 双限（另一侧=绝对常量） | `L1=32_768` |
