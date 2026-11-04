# CGSE_CORE — 上游 docs 关键词索引

> 定位：`workspace:tools/CGSE_CORE/docs/` **73 篇 `.txt` 的检索面**——回答「哪个概念 / 类名 / 配额式 / 测试落在哪篇」。
> 用法：先按 **§二 反向关键词表** 定位篇目，再按 **§一 逐篇卡** 取行号锚点，最后一跳到 `text-read_lines`。
> 覆盖：73/73 **逐字读过**（`missing-material-expansion` 1,090 行 / `stateful-inputs` 651 行 / `review-0.1.5-followup` 354 行三篇超 `text-read` 20000 字符上限，按 `text-read_lines` 分段补读，主体全覆盖）。
> 锚点校验：抽查 12 处全部回原文对齐，修正 3 处（`captured-candidate-domains` 的两处 `/4` 与 `/8` 分离 · `stateful-inputs` 主配额式收窄至 `:164-166` · 数值串改回无千分位原文形态）。
> 主题族导航（不含量化锚点）→ `analysis-docs.md` §一；本文件只做检索。
> 基线：`main` @ `b7c4a81`（v0.3.2）

---

## 〇、三条检索约定

| 约定 | 说明 |
|:--|:--|
| **配额式是关键词** | 上游 247 处配额派生几乎都是 `min(cap, remaining/N)` 形态。要查「某层拿到多少额度」，搜 `remaining`（164 处）+ 篇目，不搜类名 |
| **测试类名即篇目标题** | 243 处 `Test` 命中里，`<篇名去连字符>Test` 是最强映射（`equality-sparsification` → `CountRowSparsifierTest` 属例外，需按内容查） |
| **`provenance` 是数据面** | 30 处 `tests/provenance/*.json` 指向该篇的原始测量数据——查「这个数字怎么来的」去那里，不在文档里 |
| **锚点行号会漂** | 索引里所有 `:NNN` 都按 `b7c4a81` 实测过一遍（抽查 12 处全部对齐，修正 3 处）；上游提交后行号会移，**重定位用 `text-grep` 篇名 + 关键词**，不靠死记行号 |

---

## 一、逐篇卡（73）

> 格式：`篇名（行数）— 主题 ｜ 关键词 ｜ 锚点`
> 锚点优先给**行号**或**可 grep 的数值/标识符**。

### A. 契约与架构

**`architecture.txt`（61）— 维护边界**
关键词：Maintenance boundaries · 可移植核心 / view / 适配层 · GTLCore · sync_gtl · GraphJobCodec · 渲染 / 包 / 菜单 / 本地化
锚点：`:52` `sync_gtl.py`；「Maintenance boundaries」段

**`public-input-contracts.txt` — 输入与结果契约**
关键词：Recipe ID 唯一 · 快照只读 · `push()` 非 null → IN_DOUBT · **预算 ≠ 不可行** · 零计数归一 · `equals`/`hashCode` 稳定 · 验证入口 `gradlew check`
锚点：`:36-39` 预算纪律；`:1-51` 全篇

**`artifact-identity.txt`（22）— 构件身份**
关键词：Implementation-Revision · Source-Dirty · Source-SHA256 · CRLF→LF 归一 · `cgseIdentityPaths` · 非仓库根记 unknown
锚点：覆盖面 = `src/main` + `tools/view/src/main` + `build.gradle` + `settings.gradle` + `LICENSE` + `NOTICE`

**`migration-validation.txt`（85）— 可移植边界**
关键词：`java.base` · `java.management` · `java.desktop`（view） · 20,927 例 · 673 可移植探针 · 97 adapter 探针 · 离线装配 · 源码 JAR 逐字节
锚点：`:15-16` 依赖白名单；`:38-40` 语料计数

### B. 求解器总览与调度

**`continuations.txt`（113）— 续跑审计**
关键词：opt-in continuation · 三类阶段（productive / first-visit / completion） · 故意保持终端的限额 · DD 节点载荷 1/8 字节 · 暂停只在 coherent step 边界 · 原子 checkpoint 可超本地切片但不可超请求上限
锚点：`:87` 「Precision refusal, failed admission and natural face limits remain terminal」；`:71-92` 终端清单；`:63-65` DD 配额

**`retained-progress-scheduling.txt`（149）— 保留进度调度**
关键词：LP/LCG 优先配额（**半额上限**：entrance allowance 的一半） · 视图侧 **7/8** 上限 · 262,144 work 停滞窗 · 四次无新 cut 即失宠 · 强制首次访问 · 拒绝策略对照（半额 vs 3/4 vs 全延后）
锚点：`:78-95` 三级判据原文；`:100-109` 两条 `--model-trace` 复跑命令；`:111` 「Local admission refusals remain UNKNOWN」；`:119-127` 124cc943 对照表（760→787 PASS · 44→17 UNKNOWN · 18 unsupported）

**`effort-and-rational-lp.txt`（136）— 续跑调度 + 精确 LP 提议**
关键词：延后访问留在 continuation 队列 · quarter-share 用**完成工作量**计 · CountLpLearning 连分数提议 · mirror_crystal 配方
锚点：5,903 + 62,284 配方；extra-first / random 达 20M 上限

**`numeric-progress-and-grants.txt`（143）— 数值进度与 grant**
关键词：计数 Gram 稀疏索引 · LLL 进度检查点 · 嵌套 grant 续期只用**未用部分** · grant 不跨请求
锚点：`:95` 822 次运行总 work `1,470,984,329` → `1,340,219,000`；同一基线亦见 D: affine-gram-admission（`:83`）

**`decision-queue-and-publication.txt`（98）— 决策队列与发布**
关键词：CountCdcl 索引化最大队列（≥64 坐标） · `learnedChoices` 改发布 outbox · 广播风暴
锚点：`:30` 广播 61,864 / 69,016 → 48/例

**`explicit-plan-reuse.txt`（65）— 完整计划复用**
关键词：`RequestPlanningWork.ReuseCandidate` 不透明句柄 · 计划复用不重建
锚点：`:53` Radox 全塔 2,571,921 → 27,992 work；峰值 8,960 vs 14,000,176 字节

**`pipeline-cost-attribution.txt`（41）— 成本归因矩阵**
关键词：`enableMetrics()` · `pipelineMetrics()` · **11 origin × 9 stage** · UNATTRIBUTED/OTHER · 子作用域替换父归因 · 每 work 单位 16 tick · 非墙钟延迟
锚点：`:12-13` 启用方式；`:35-38` 「no retained plan, inventory key, per-candidate history」

**`request-work-clocks.txt`（45）— 请求局部 worker 时钟**
关键词：进程级 worker 时钟缺陷 · 分数 work tick 跨请求污染 · `PlanningWorkClockTest` · 修复后历史无关
锚点：`:1-45` 全篇；四前缀 work 3,700,185 / 371,693 / 371,693 / 3,700,181 → 统一 3,700,185

**`solver-throughput.txt`（197）— 固定 work 吞吐**
关键词：LCG 传播残差预计算 · ExactRational 快路径 · ProductQuotient 768 字节复用 · 两组 Time/Allocation 比 · **「No fluid/count quantity is converted through floating point」**
锚点：`:26-27` 浮点纪律原文；`:42` CountLcgWatchTest

### C. 预算、准入与限额

**`audit-followup.txt`（187）— 五项自认缺口**
关键词：8192 资源/配方准入 · 数值可达 · **J 端到端归因** · **H 跨快照驻留** · 集成与旧限 · `known-limits.json` 是 e788d3d3 历史快照
锚点：`:139-187` 五项全文；`:172-174` 历史快照警告；`:185-186` 「The next algorithm change should target a retained, reproducible failure in items 1 or 2」；`:175-181` 逐增量 UNKNOWN 下降链（32 → 25 → 22）

**`hall-support-admission.txt`（71）— Hall 支撑准入**
关键词：ambient **8,192 坐标 / 32,768 行** · matching relaxation **512 坐标 / 2,048 行** · discovery ≤ 8,192 work + Hall allowance 1/4 · 整行准入 · 跨行跳过不置零
锚点：`:19-22` 两组上限 + 额度；`:33-37` 「rejected admission … do not establish feasibility or infeasibility」；`:41-47` HallSupportTest（32/600/4,096 坐标 × 6 序）

**`stateful-inputs.txt`（651）— 有状态输入与物理返还**
关键词：`min(4096, remaining/32)` · 8 轮上限 · 32,768 / 十六分之一生命期 · PRESERVE 语义 · 状态轨迹 · 约 50 个 Test 类
锚点：`:164-166` 主配额式（「At most eight cycles and `min(4096, remaining request work / 32)` work per matching」）；`:169` 32,768 / one-sixteenth lifetime；`:520-651` 回归清单；`:649-650` 「A host must additionally test actual provider capture, dispatch, inventory routing and persistence」

**`root-execution-reachability.txt`（152）— 根执行可达性**
关键词：`min(32768, remaining/32, 当前 count turn 剩余 allowance)` · `min(131072, remaining/32)` · 根执行可达性判定
锚点：`:14-54` 两条配额式

**`structural-completion-service.txt`（95）— 结构补完服务**
关键词：初始编译样本后 **262,144 批次**续同一 portfolio · 已选结构机会仍在 → 续 · 空闲反馈区间不等于组合停摆 · packing/LP 保持交错服务（不抢早期）
锚点：`:20-35` 调度变更；`:36-46` 拒绝面（更宽原型丢历史 witness）

**`count-corpus-followup.txt`（116）— 计数语料跟进**
关键词：**87 条 unresolved 只过了一个 CountLcg 本地额度** · e788d3d3 快照 · `--model-engine lcg-first|lp-retained|affine-retained|mitm-retained|views|main-counts` · 842b3eea 容量边界
锚点：`:9-11` 本地额度原文；`:8-12` 同批重放 194 结论 / 80 unresolved；`:89-93` 全量命令仍 exit 2 的分工；`:106-110` 两条 residual work 对照

**`source-refinement-and-dispatch.txt`（139）— 源修复与协作分派**
关键词：源修复 · 本地计数准入 · 协作式分派 · 8,192 资源准入声明
锚点：`:129` 8192 resources

### D. 数值与格

**`affine-gram-admission.txt`（143）— 仿射准入与整数 Gram 因子**
关键词：精确整数 Gram 因子 · 48 坐标上限 · 精度守卫 · `CountGramSchmidtTest`
锚点：`:90` 测试名；`:11` CountAffineLattice

**`integral-lattice-factors.txt`（138）— 保留整数 Gram 因子**
关键词：Gram 因子保留 · 40 位精度守卫 · 主变量界
锚点：CountAffineLattice

**`sparse-affine-basis.txt`（84）— 稀疏仿射更新**
关键词：稀疏仿射更新 · 保留支撑索引 · 稠密 fallback
锚点：`:6` CountAffineLattice

**`centered-subset-lattice.txt`（176）— 中心化子集格**
关键词：中心化子集 witness · 保留块更新 · 子集和
锚点：—

**`lattice-box-enumeration.txt`（173）— 保留箱枚举**
关键词：向量分配的保留箱枚举 · `CountVectorBox` / `CountAffineLattice`
锚点：`:12` CountVector*；`:16` CountAffineLattice

**`lattice-potential-refinement.txt`（133）— 势引导格细化**
关键词：势引导细化 · 保留细化进度
锚点：`:77-91` UNKNOWN 记账

**`quarter-sum-reuse.txt`（129）— 精确重复四分之一和**
关键词：quarter sums · 精确重复和复用
锚点：—

**`parity-and-root-continuation.txt`（127）— 奇偶闭包与根续跑**
关键词：certified parity closure · 保留根搜索 · CountAffineLattice
锚点：`:63` CountAffineLattice

**`projected-startup-proofs.txt`（60）— 投影启动闭包**
关键词：projected startup closure · 投影行 · 启动期证明
锚点：—

**`numeric-continuation-and-root-proofs.txt`（116）— 根证明聚合与数值续跑**
关键词：根证明 handoff（原模型 / DEAD 状态 / 空分支假设三查） · 算术所有权（单模列变换 / Gram-Schmidt / LLL） · 48 坐标准入不变
锚点：`:37-45` 「Precision refusal, admission failure and natural trial face limits remain terminal candidate failures. **None proves infeasibility.**」；`:49-58` 243 → 246 结论对照；`:103-116` 22 条剩余（15 SAT / 7 UNSAT）

**`cold-integer-lp-continuation.txt`（105）— 冷整数 LP 续跑**
关键词：cold start 整数 LP · 保留基 · 续跑不重建
锚点：—

### E. 整数 LP / 剪枝 / 学习

**`integer-lp-and-row-templates.txt`（129）— 有界整数 LP 学习 + 行模板**
关键词：`CountLcg.integerRelaxation()` vs `learnedRelaxation()` · x = lower + z · 精确割活动 · `CountIntegerLpLearning`（**128 列 / 512 行 / 4,096 项**） · `INTEGER_LP_HINT` 一次性 · 行模板（32 模板 / 2 MiB / 64 KiB 签名缓存）
锚点：`:27-30` portfolio 准入（需先产新精确后果）；`:26-27` kernel 上限；`:42-46` wide-68 对照（direct 41,934 work vs ordinary 500k 耗尽）；`:31-34` broad early admission 挤掉 wide-68 三序

**`retained-integer-lp-portfolio.txt`（82）— 根组合中的保留整数 LP**
关键词：root portfolio 准入 · 保留有界整数 LP
锚点：—

**`retained-finite-hints.txt`（220）— 保留有限 LP 方向与硬案诊断**
关键词：有限方向保留 · 硬案诊断 · n144-s1 优先 · SCIP 独立 witness
锚点：`:199` CountAffineLattice

**`exact-row-activities.txt`（121）— 精确保留行活动**
关键词：LCG 行最小值 long 化（溢出升级 BigInteger） · PB 行要求全权重和入 long
锚点：`:39` CountLcgActivityTest；`:43` CountLcgFiniteTest；批时 −10.22%、分配 −33.05%

**`integer-conflict-minimization.txt`（141）— 辅助整数冲突最小化**
关键词：首次 UIP 后移除非断言界 · **1/32 LCG work 累计份额** · 每冲突 2,048 · 每候选 256
锚点：`:101` wide-79/order-0 UNKNOWN → `5826601` work

**`incremental-rounding-locks.txt`（34）— 增量 rounding 锁**
关键词：rounding 锁增量复用 · 浮点分贡献按**原行序**重求和防漂移
锚点：`:24` 185,712 → 64,066 work（原文无千分位：`185712` / `64066`）

**`count-row-overlap.txt`（96）— 跨目标行复用**
关键词：`Constraint` 私有不可变有序系数图（免 defensive copy） · 跨目标借未受影响行（四分之三列同位）
锚点：`:5` CountCatalog；跨目标 work −89.3%、分配 −95.4%

**`count-group-transfers.txt`（80）— 成对转移 scout**
关键词：`CountJumpTransfers` 成对转移 · scout **16,384 work** · 须普通走查先花 **262,144**
锚点：`:12` 类名；96 planted 保持 34 witness

**`row-template-concurrency.txt`（45）— 行模板缓存并发**
关键词：模板缓存并发 · 32 并发请求
锚点：—

### F. packing（打包）

**`packing-phase-reuse.txt`（57）— 相位保留与返还**
关键词：枚举 / PB 相位保留 · 切换到 PB 时丢弃已付枚举栈 · 从实际进入点计 PB grant · 不确定后回到同一枚举
锚点：`:3/22/24/25` CountPacking*；147 例丢栈 / 58 例早切

**`packing-conflict-reuse.txt`（87）— 枚举恢复时的限定冲突复用**
关键词：私有 PB 子句复用 · 296 例 146 positives 不变
锚点：`:41` CountPacking*；旧 positive 145 保同 work / 1 加 3,833 work

**`packing-lp-reuse.txt`（120）— 请求内 LP 复用**
关键词：请求内 LP 基复用 · 深度缓冲复用 · 整数价格界证书
锚点：`:68/75` CountPacking*

**`packing-lp-continuation.txt`（95）— 修复后的保留 LP 候选搜索**
关键词：宽交换 + PB 样本后再用 LP · 原模型 frontier 保留 · mknapcb3_01_bks
锚点：`:9/27/57` CountPackingLpSearch；146 → 147 positives / 150 → 149 unresolved；旧 positive 145 同 work、1 加 3.7%

**`packing-priced-propagation.txt`（114）— 精确价格传播**
关键词：保留 LP 盒内必要值定界 · 长路径检查 · BigInteger 绝对量级预检 · `mknapcb3_01_bks`
锚点：`:66-71` 该案 38,862,924 → 38,412,277（−1.160%=450,647）· 保留 LP 1,707 nodes / 4,835 pivots（原 1,863 / 6,612）· 2,317 精确 implication；`mknapcb6_09_bks` 省 87；其余 145 同 work；`:73-75` 48 个新序保持 46 positives，order97 仍 unresolved

**`fractional-packing-neighborhood.txt`（106）— 分数打包邻域**
关键词：分数选择邻域 · 局部四舍五入域
锚点：`:4/32/94/100/103` CountPacking*

**`fractional-packing-rounding.txt`（122）— 分数打包完成**
关键词：分数完成 · 精确非对称互补域（全部分数选择 + 一/四普通变更，选较便宜朝向）
锚点：`:13/106/118` CountPacking*；40M/296 例 141 → 146 positives / 155 → 150；恢复 mknapcb3_07 / 4_25 / 5_24 / 6_11 / 6_23

**`asymmetric-packing-exchange.txt`（129）— 互补非对称交换**
关键词：非对称交换 · 互补域 · CP-SAT / SCIP 原始模型 witness
锚点：`:40/67/70/85` CountPacking*；旧 positive 总 work +2.9%

### G. 等式与匹配

**`equality-sparsification.txt`（91）— 等价等式稀疏化**
关键词：`CountRowSparsifier` · 至少消去 **1/8** 非零项才提供额外表示 · 精确相消（无容差） · 不等式整除向**下**舍全右侧 · 证明 DAG（`CountProof` 校验） · 额度 `min(65536, remaining/64)` + 预留 scratch
锚点：`:23-32` 额度与拒绝面；`:25` 额度式原文；`:33-40` 证明 DAG；32 变量等式：直 LCG `262356`-work 后 UNKNOWN vs 稀疏 LCG `367` work；编译 + `14439` work；64 变量 `2172014..2340944` → `254321..255113`；40M 冻结语料 `792` PASS / `12` UNRESOLVED / `18` unsupported；822 运行编译总 `357664` work，最大相对增幅 ≈4.2%

**`equality-message-search.txt`（45）— 稀疏整数等式的有界消息**
关键词：`CountMessageSearch` · 浮点消息只定分支序 · 前缀窗 `max(0, T-(R-P[k])) <= s <= min(P[k], T)` · 主组合准入 **≥16,777,216 work** 且 >128 变量
锚点：`:1-45` 全篇

**`equality-word-support.txt`（123）— 可复用词工作区**
关键词：无符号 long 词位集替代 BigInteger bitset · n144-s1 72 维有理核
锚点：分配 −82.43% / −82.51%

**`sparse-equality-support.txt`（308）— 短等式支撑**
关键词：短行和支撑 + 整数冲突学习 · 从编译视图认证根界起步 · 不导出私有负结论 · 四族仍 UNKNOWN
锚点：`:32` 保留 CountLcg；`:58-158` 五道 Test；40M/720a8141 四族不收敛

**`scalar-count-matching.txt`（120）— 可负担标量计数匹配**
关键词：标量计数匹配 · 四列表 join
锚点：—

**`residue-count-matching.txt`（116）— 保留四列表计数候选**
关键词：四列表 residue 表 · 计数候选
锚点：—

**`complement-count-matching.txt`（138）— 互补 residue 表**
关键词：互补 residue 表 · 残余工具实验
锚点：`:46` / `:91-95` UNKNOWN 记账

**`finite-integer-propagation.txt`（93）— 精确有限域传播**
关键词：`q = floor(r/a)` 避免 `a*width` 溢出 · reason 松弛 `a-1-(r mod a)` · `CountLcg` 特化
锚点：`:6` 类名；`:43` CountLcgFiniteTest；`:48` CountLcgWatchTest；时间 −12.2%、分配 −37.1%

### H. 向量分配 / 源-燃料

**`deferred-vector-allocations.txt`（96）— 延后多维分配修复**
关键词：延后分配修复（0.1.4） · 24 组源/燃料 · 三序同形
锚点：`:19` CountVector*

**`vector-allocation-admission.txt`（80）— 编译期向量分配准入**
关键词：编译期准入 · 容量结构认证
锚点：`:22/59` CountVector*

**`vector-candidate-reuse.txt`（72）— 整数分配候选复用**
关键词：候选复用 · 保留分配 frontier
锚点：`:11` CountAffineLattice；`:19` CountVector*

**`vector-neighborhood-repair.txt`（231）— 宽向量分配受限修复**
关键词：受限箱搜索 · 恢复源/燃料十个边界中的六个 · 额度 1/8
锚点：`:22/25/70/115/151/204` CountVector*；`:24/33` CountAffineLattice；`:56` / `:139` 1/8 配额

**`refill-joint-sources.txt`（102）— 联合源计数与长路径**
关键词：联合源计数 · 完整长路径分析 · refill 质量
锚点：—

### I. 宿主捕获与状态

**`captured-candidate-domains.txt`（398）— 捕获候选域**
关键词：`CapturedCatalog` / `CapturedRecipeDomain` 精确基数与 mixed-radix ordinal · INDIVISIBLE vs MIXED 槽位 · `min(32768, remaining/4)` 受限首轮 · auxiliary 模型 **256 slots / 1,024 变量 / `min(32768, remaining/8)`** · CGD2 编解码与独立 checker
锚点：`:66` `min(32768, remaining search work / 4)` + 256-work 下限；`:187` `min(32768, remaining search work / 8)`；`:182-190` 「Admission refusal or a local cutoff never means infeasible」；`:352-397` cross-pattern 边界与 dispatch-group 未覆盖

**`captured-host-integration.txt`（28）— 捕获域与宿主集成**
关键词：不可变 `CapturedRecipe` · 稳定 provider 绑定 · 目录/覆盖版本化 · 不完整捕获须显式声明 · GTL adapter @a1d5a5ca 仍用旧 expand
锚点：七项必做宿主回归清单

**`captured-result-handoff.txt`（23）— 捕获结果交接**
关键词：INFEASIBLE 仅当覆盖完整 + canonical view · 正 witness 在受限视图仍有效（不主张最优） · 取消在共享预算超时后仍是取消
锚点：`CapturedHandoffTest`

**`missing-material-expansion.txt`（1090）— 缺料展开**
关键词：prefix 需求公式 `max(input + max(0,input-output)*(n-1), old_need-net_output*n)` · `RefillProducerOrder` SCC 排序 · `RefillWordPruning` · `RefillSupplySearch`（12 配方 / 16 键 / 128 端口） · `RefillRepeatSizing` `max(r+max(0,-d)*(n-1), R-d*n)` · `GraphPlan.missingBoundary(key)` **五值枚举** · `refillDiagnosticWork()` 2,048
锚点：`:286` `min(2048, remaining/2)`（扫描访问）；`:331` `min(2048, remaining/2)`（锚点扫描）；`:596` `min(8192, remaining/4)`（两趟可选 pass）；`:631` `min(4096, remaining/4)`（sizing）；`:777` `min(8192, remaining/8)`；`:823` missingBoundary；`:820-832` 五值枚举；`:229-932` 约 30 道 Test

**`funded-cycle-entries.txt`（121）— 有资金循环入口**
关键词：`GraphSeedEntries` 扫 cyclic region 替代 producer · 本地额度 **max 16,384-work** · UNKNOWN 计为「未定论一轮」 · 旧 deficit 不参与当前评分
锚点：`:1-121` 全篇；四组排序 40M 等预算对照表；`SeedEntryRepairTest` 72 witness

**`seed-preservation-and-loop-growth.txt`（149）— 种子保留与循环增长**
关键词：PRESERVE 对净消耗循环输入同样生效 · `SeedStartupGuard` 乐观闭包 · 有理证书豁免耗散转换 · `SequenceLoopIndex` 准入 **65,536-work** + 每 tick ≤ **1,024** · `PlanSourceCycles`
锚点：`:1-149` 全篇；22 道回归程序名清单

**`state-review-followup.txt`（753）— 状态模型复核**
关键词：interned 转换 · 类型化本地结局与整键分配 · 跨规则匹配与 Hall 解释 · 同类数量序修复 · 替代实体选择 · 五处补齐各带构造族 + UNKNOWN 前后数
锚点：`:1-200` F1/F2 补齐；`:428-434` 配额式 `min(cap, remaining/N)` + scratch

### J. 回归与复核记录

**`full-regression-20261011.txt`（60）— 全量回归基线**
关键词：205 源文件指纹 · 三处 solver 修复（CountJump 双 double 平局 / 稀疏 cut 维度 / 强制 target 净增益） · 两处测试缺陷
锚点：`:4-6` 基线 66f22b5b + 指纹；`:10-14` **CountJump 平局用精确残差先减**（浮点修法先例）；`:16-22` 稀疏 cut 与 1024 变量稠密界分离；`:24-32` 强制 target 净增益 ≥1；`:50-57` 数字（standard 83 / fixtures 244×3 / 20653 manifest 19642 PASS / count 822 行 790 PASS）

**`regression-0.1.5.txt`（77）— 0.1.5 回归记录**
关键词：20,927 记录 · 20765 PASS / 156 unresolved / 6 unsupported · 明确未执行面
锚点：`:70` CountPacking*

**`review-0.1.5-followup.txt`（354）— 0.1.5 复核跟进**
关键词：复核整改项 · 配额式分布
锚点：`:68/145` `remaining`

**`historical-hard-families.txt`（175）— 按机制的硬案族**
关键词：**canonical backlog 结构**：156 = 150 布尔打包 + 4 稀疏等式（144/257 变量）+ 2 源/燃料（96/128 组 · 9 维）+ seed31 · 四条机制族 · 打包终止相位归因（57/56/34/9） · 维度组合清单（`(100,10)9` … `(500,30)28`）
锚点：`:1-30` backlog 分解；`:31-46` 相位统计；`:47-60` 维度表；`:86-110` 各族增量台账；`:163-175` prioritization rule

**`releases.txt`（63）— 发布链**
关键词：发布门禁三闸（`check_abi` → `verify_artifacts` → 上传） · workflow 不跑回归 · `mods.toml` 落点
锚点：—

---

## 二、反向关键词表

> 查「某概念在哪篇」用这里。数字为篇目所属族编号（A–J）。

### 类名 → 篇目

| 类名 | 篇目 |
|:--|:--|
| `CountAffineLattice` | D: affine-gram-admission · integral-lattice-factors · sparse-affine-basis · lattice-box-enumeration · lattice-potential-refinement · parity-and-root-continuation · numeric-continuation-and-root-proofs · cold-integer-lp-continuation · vector-candidate-reuse · retained-finite-hints · audit-followup · continuations · numeric-progress-and-grants |
| `CountVector*`（Box / Search / Neighborhood） | H: deferred-vector-allocations · vector-allocation-admission · vector-candidate-reuse · vector-neighborhood-repair · lattice-box-enumeration |
| `CountPacking*` | F: 全 8 篇 + C: structural-completion-service · J: regression-0.1.5 |
| `CountLcg` | B: solver-throughput · numeric-progress-and-grants / E: exact-row-activities · finite-integer-propagation · integer-conflict-minimization · incremental-rounding-locks · integer-lp-and-row-templates · count-group-transfers / C: count-corpus-followup |
| `CountCdcl` | B: decision-queue-and-publication / E: integer-conflict-minimization |
| `CountMeetInMiddle` | D: quarter-sum-reuse · centered-subset-lattice · numeric-continuation-and-root-proofs |
| `CountKernelSearch` | D: integral-lattice-factors · cold-integer-lp-continuation · numeric-continuation-and-root-proofs |
| `CountDiophantine` | D: numeric-continuation-and-root-proofs · retained-progress-scheduling · cold-integer-lp-continuation |
| `CountViewSearch` | B: retained-progress-scheduling · structural-completion-service / D: numeric-continuation-and-root-proofs |
| `CountMessageSearch` | G: equality-message-search |
| `CountRowSparsifier` | G: equality-sparsification |
| `CountLpSearch` / `CountLpLearning` | B: retained-progress-scheduling · effort-and-rational-lp / E: integer-lp-and-row-templates |
| `CountJump` / `CountJumpTransfers` | E: count-group-transfers · integer-conflict-minimization / J: full-regression-20261011（浮点平局先例） |
| `CountProof` | G: equality-sparsification / E: integer-lp-and-row-templates / D: numeric-continuation-and-root-proofs |
| `CountHall` | C: hall-support-admission / I: state-review-followup |
| `CountRowTemplates` | E: integer-lp-and-row-templates · row-template-concurrency |
| `GraphPlanner` | A: architecture / B: pipeline-cost-attribution / J: historical-hard-families / I: captured-host-integration |
| `PlanningBudget` | C: audit-followup · hall-support-admission / B: pipeline-cost-attribution · request-work-clocks / I: funded-cycle-entries |
| `IntegerCountSearch` | B: structural-completion-service · decision-queue-and-publication / C: source-refinement-and-dispatch |
| `GraphPlan.missingBoundary` | I: missing-material-expansion（`:823`，五值枚举） |

> 🔴 **`CountObbt` / `CountSoftSearch` / `CountBranchProbe` / `CountDomainSearch` / `CountContinuation` 在 docs 全文 0 命中**——这五个类只有源码，无上游文档。查它们只能读码（各自的「无声准入」问题即 `analysis-solver.md` §二 / `analysis-scaffold.md` §二 的主题）。

### 概念 → 篇目

| 概念 / 关键词 | 篇目 |
|:--|:--|
| **预算 ≠ 不可行** | A: public-input-contracts / C: audit-followup · hall-support-admission · count-corpus-followup / B: continuations · retained-progress-scheduling / D: numeric-continuation-and-root-proofs / I: captured-candidate-domains · captured-result-handoff |
| **准入拒绝保持 UNKNOWN** | B: retained-progress-scheduling（`:111`） / D: numeric-continuation-and-root-proofs（`:39-40`） / B: continuations（`:87`） / C: hall-support-admission（`:33-37`） / I: captured-candidate-domains（`:187-189`） |
| **配额式 `min(cap, remaining/N)`** | C: hall-support-admission · stateful-inputs · root-execution-reachability / G: equality-sparsification / I: captured-candidate-domains · missing-material-expansion / H: vector-neighborhood-repair / B: retained-progress-scheduling |
| **1/8 额度** | C: count-corpus-followup（`:9-11` 单引擎本地额度）· captured-candidate-domains（`:187` `min(32768, remaining/8)`）· continuations（`:63-65` DD 节点载荷） / G: equality-sparsification（`1/8` 非零项阈值） / H: vector-neighborhood-repair（`:56` 合并上限 = 剩余 1/8；`:139` 每深度 `2,097,152`，同受 1/8 约束） / I: missing-material-expansion（`:777` `min(8192, remaining/8)`） |
| **262,144 批次** | B: retained-progress-scheduling（样本上限） / C: structural-completion-service（续跑批次） / E: count-group-transfers（走查前置） |
| **保留进度三阶段**（productive / first-visit / completion） | B: retained-progress-scheduling（`:78-95`） / C: structural-completion-service / E: integer-lp-and-row-templates（`:27-30` portfolio 准入） |
| **8192 资源/配方准入** | C: audit-followup（`:141-146`）· hall-support-admission（`:19-20`）· source-refinement-and-dispatch（`:129`） |
| **48 坐标格准入** | D: 全族（`numeric-continuation…:45` 明写「unchanged」） / C: audit-followup（`:148-153`） |
| **浮点只用于候选排序** | B: solver-throughput（`:26-27` 纪律） / J: full-regression-20261011（`:10-14` CountJump 修法） / G: equality-message-search · integer-conflict-minimization |
| **精确整数算术 / BigInteger** | B: solver-throughput / E: exact-row-activities · finite-integer-propagation / G: equality-sparsification · equality-word-support / F: packing-priced-propagation |
| **IN_DOUBT / push() 非 null** | A: public-input-contracts / I: captured-result-handoff |
| **已知硬案族群** | J: historical-hard-families（156 分解 + 相位归因） / B: retained-progress-scheduling（14 UNKNOWN 归并） / D: numeric-continuation-and-root-proofs（22 剩余） |
| **已知限额表 `known-limits.json`** | C: audit-followup（`:172-174` e788d3d3 快照）· count-corpus-followup（`:8-12` 单引擎额度） |
| **`--model-trace` 复跑** | B: retained-progress-scheduling（`:108-109` 两条命令） / C: count-corpus-followup（`:87`） |
| **`tests/regression/count-boundaries.json` 钉界** | B: retained-progress-scheduling（27 条新结论 @20M） / D: numeric-continuation-and-root-proofs（3 + 7 边界） / E: integer-lp-and-row-templates（wide-68） |
| **`tests/provenance/*.json` 数据面** | 30 处：B: numeric-progress-and-grants / D: numeric-continuation-and-root-proofs / E: integer-lp-and-row-templates / F: packing-lp-continuation / H: vector-neighborhood-repair / I: funded-cycle-entries · state-review-followup |
| **ABI 基线 1,478 条目** | A: migration-validation · artifact-identity / J: releases · full-regression-20261011 / C: structural-completion-service |
| **CP-SAT / SCIP / Z3 独立核** | D: numeric-continuation-and-root-proofs（`:99-116`） / J: historical-hard-families / F: asymmetric-packing-exchange / E: retained-finite-hints |
| **AE dump / Forge 打包** | A: architecture / J: releases · full-regression-20261011 / C: structural-completion-service |
| **`sync_gtl.py` 跨仓同步** | A: architecture（`:52`） |
| **WCOJ / 四列表 join** | G: residue-count-matching · complement-count-matching · scalar-count-matching / D: quarter-sum-reuse |
| **SCC / 循环 / 拓扑** | I: missing-material-expansion（`RefillProducerOrder`）· seed-preservation-and-loop-growth（`PlanSourceCycles`）· funded-cycle-entries（`:244` 16 轮上限） |

### 测试类 → 篇目（243 处的常用子集）

| Test | 篇目 |
|:--|:--|
| `PlanningWorkClockTest` | B: request-work-clocks |
| `CountProgressSchedulingTest` / `RootPortfolioTest` | B: retained-progress-scheduling |
| `CountCompletionServiceTest` / `CountPackingServiceTest` | C: structural-completion-service |
| `HallSupportTest` | C: hall-support-admission |
| `CountRowSparsifierTest` / `SparseEqualityReview` | G: equality-sparsification |
| `NumericContinuationTest` / `CountKernelTest` | D: numeric-continuation-and-root-proofs |
| `IntegerLpLearningTest` / `CountRowTemplatesTest` / `CountNormalizedTemplatesTest` / `IntegerLpProposalTest` | E: integer-lp-and-row-templates |
| `CountLcgActivityTest` / `CountLcgFiniteTest` / `CountLcgWatchTest` / `CountLcgMinimizeTest` | E: exact-row-activities · finite-integer-propagation · integer-conflict-minimization |
| `CapturedStateForcedWearTest` / `CapturedStateCapacityChoiceTest` 等约 50 道 | I: stateful-inputs（`:492-646`）+ state-review-followup |
| `SeedEntryRepairTest` | I: funded-cycle-entries |
| `RefillNormalizedWordTest` / `FixtureProductionTest` | I: missing-material-expansion / J: full-regression-20261011 |
| `CapturedHandoffTest` | I: captured-result-handoff |
| `GraphFixtureRegression` / 741 fixture checks | J: full-regression-20261011 · C: structural-completion-service |

---

## 三、没写进文档的东西（要读码的缺口）

> 判断依据：`text-grep` 全 docs 目录 0 命中。

| 缺口 | 说明 | 去哪 |
|:--|:--|:--|
| `CountObbt` 三轴准入（`lower>64` / `rows>256` / `allowance<4096`） | docs 零提及——64 / 256 两个常数从哪来、为何与被收紧对象无关，上游没写 | 源码 `CountObbt.java:23-45` |
| `CountDomainSearch.trial` 字段 | docs 零提及（`trial` 只在 `equality-message-search` / `continuations` 作通用词出现） | 源码 `CountDomainSearch.java:30,151,170` |
| `CountContinuation` 接口无归因口 | docs 未讨论接口面；只在各篇强调「拒绝不得当不可行」的原则 | 源码 `CountContinuation.java:10-28` |
| `CountLcg` 的 `/8` 具体绑定点（`CountLcg.java:405`） | docs 只说「一个本地额度」，不点位置 | 源码 + `analysis-scaffold.md` §三 |
| `CountDemandProjection.repair` 浮点打分 | docs 零提及该类 | 源码 `CountDemandProjection.java:457-482` |

---

## 四、维护

- **新增上游文档时**：在 §一 对应族插卡（主题 / 关键词 / 锚点）+ §二 反向表补类名与概念 + §三 若填了旧缺口则核销
- **上游提交后**：锚点行号会漂——`text-grep` 篇名 + 关键词重定位比死记行号可靠
- **不写「已读/未读」状态**：本文件是检索面，不是进度表；进度归 `todo.md` / `CHANGELOG.md`
- **数值锚点照抄原文**：上游文档多数不带千分位（`262356` 而非 `262,356`），索引里数值串保持原样，便于直接 grep
