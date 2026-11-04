# CGSE_CORE — 项目档案

> 类型：**外部开源项目协作开发**（Java 17 + Gradle 8.8 · MPL-2.0）· 版本：v0.26
> 主人格：Coder · 加载 Job：**无**——本支无 Java 技术栈 Job（cs-* 是 C# 专用，不适用）；工具面 = `text-*` + `powershell`（gradle / python 外部通道）
> 路径：`Project/CGSE_CORE/` · 代码：`WorkSpace:tools/CGSE_CORE/`
> 落点说明：按 `Discipline` §三-16 已建档项目默认落 `WorkSpace:\<名>\`；本项目经 Rushiera 指令「建档进 tools」定点落 `tools/CGSE_CORE/`（上游协作仓，视作工具工作区）——档案在 CCBP，工程在 tools。
> 仓库入口：`workspace:tools/CGSE_CORE/build.gradle`（单模块根项目 `cgse-core`，`settings.gradle` 无子项目）· 构建入口 `gradlew.bat`（wrapper 8.8）· 跑测入口 `workspace:tools/CGSE_CORE/tests/run.py`
> 上游：`https://github.com/autumn-2-net/CGSE_CORE.git`（`origin` 直连，公开仓 Public · 275 commits）· **协作形态：本地改动推 fork**（`fork` remote = `https://github.com/Rushiera/CGSE_CORE.git`，Rushiera 2026-11-05 裁决）——`origin` 保持只读、不提 PR；源码可改，代价后置上游评审（见 §范围与边界「裁决记录」）
> 运行环境：**项目内 `runtime/`**（`runtime/jdk-17.0.2` + `runtime/gradle-8.8` + `runtime/gradle-home`）——已入 `.gitignore`，不进仓库、不动系统环境变量
> 创建：2026-11-01
> 缘起：Rushiera 点名——朋友邀请协助开发这个仓库，克隆进 `tools/` 并建档

---

## 项目目的

把上游 CGSE（Compiled Graph Solving Engine，编译图求解引擎）作为**可参与协作的外部工程**纳入本支工作区：能构建、能跑测、能读设计文档、能在上游变更后同步。

| 消费面 | 具体用途 |
|:--|:--|
| 协作开发 | 按朋友邀请参与上游：定位问题 → **改结构解决** → 本地跑测 → 成果交上游评审（协作形态已定，见「未决项」1） |
| 求解引擎知识面 | 精确算术 / 整数与可达性求解 / 证明生成与检查 / 调度与执行序——一套可读的工业级实现 |
| 工程纪律参照 | ABI 基线门禁、可移植边界校验、20,927 例回归语料、发布三闸——可迁移到本支其他项目 |
| 宿主适配面 | Forge mod 打包 + GTLCore / AE2 适配（适配层代码不在此仓，在 GTLCore） |

**一句话定位：** 这是一个**别人的开源工程，我们参与开发**——不是二开改造，改动方向由上游维护者（autumn）与朋友的协作约定决定，不默认「照我们的意志重构」。

## 关键事实（2026-11-01 克隆后实测）

| 事实 | 证据 |
|:--|:--|
| 版本 **0.3.2** · 兼容线 **0.3**（补丁版保持 Java API 与持久化快照兼容） | `gradle/cgse.properties:1-3` |
| 末次提交 `b7c4a81` · 2026-10-27 20:02:37 +0800 · «chore: advance CGSE to 0.3.2 after host synchronization» | `git log -1` |
| 分支只有 `main`（`origin/main` 单线，无 develop / 无 release 分支） | `git branch -a` |
| 许可 **MPL-2.0** · Copyright (c) 2026 autumn | `LICENSE` / `NOTICE` / jar manifest `Bundle-License` |
| 技术栈 Java 17（`toolchain 17` + `options.release = 17` + UTF-8）· Gradle 8.8（wrapper，`services.gradle.org/distributions/gradle-8.8-bin.zip`） | `gradle/engine.gradle:19-27` · `gradle/wrapper/gradle-wrapper.properties` |
| 规模：25,792 文件（克隆计数）· `src/main/java/org/cgse/core/` **326 个 .java**（`Captured*` / `Count*` / `Graph*` / `Plan*` 等前缀族） | 克隆计数 + `file-find` |
| 四个 sourceSet：`main` · `test`（`tests/core|shared|regression|corpus/java`）· `view`（`tools/view/src/main/java`）· `viewTest`（`tests/view/java`） | `build.gradle:14-28` |
| 领域 = Minecraft **AE2**（Applied Energistics 2）合成图求解；Forge **1.20.1** mod，modId `cgse`，Forge loader `[47,)`、MC `[1.20.1,1.20.2)`，可选依赖 gtlcore `[1.2.3.2-fix6,)` / ae2 `[15,)` | `packaging/forge/META-INF/mods.toml` |
| **适配层不在此仓**——Minecraft 渲染 / 包 / 菜单 / 本地化 / `GraphJobCodec` 全在 **GTLCore** 项目维护 | `docs/architecture.txt`「Maintenance boundaries」 |
| 设计文档 **73 篇 `.txt`**（`docs/`，按主题散列，无 index） | `file-find` |
| ABI 基线三份：`api/0.1.txt` / `0.2.txt` / `0.3.txt`（`check_abi.py` 比对） | `api/` |
| 回归语料 20,927 例 + 673 可移植历史探针 + 97 adapter 探针 | `docs/migration-validation.txt:38-40` |

## 工程结构

| 目录 | 内容 |
|:--|:--|
| `src/main/java/org/cgse/core/` | **可移植核心**（真源）——精确算术 / 图编译 / 整数与可达性求解 / 证明 / 记账 / 调度 / 执行序 / 检查点 |
| `tools/view/src/main/java/org/cgse/view/` | 可选桌面导出（AWT 栅格化 + SVG/PNG），`java.desktop` 依赖 |
| `tests/` | Python 跑测驱动（`run.py`）+ `core|view|shared|regression|corpus|oracles|probes|fixtures|benchmarks|adapter|datasets|ae-dump` 诸面 |
| `api/` | 公共二进制签名基线（ABI 门禁输入） |
| `docs/` | 73 篇专题设计文档 |
| `tools/` | `sync_gtl.py`（向 GTLCore 同步可移植源，**默认只读比对，`--apply` 才写**）· `verify_boundaries.py` · `release/{ApiSurface.java,check_abi.py,verify_artifacts.py}` · `ae-dump/`（Forge 诊断子工程，build 时由 `forge.gradle` 以 remapJar 内嵌进 mod——运行时 `/ae dump` 导 ZIP 到 `logs/ae-dump/`） |
| `packaging/forge/` | Forge mod 打包模板（`mods.toml`） |
| `gradle/` | `engine.gradle`（编译 / 版本强校验 / 构建身份摘要）· `forge.gradle`（Forge 包 + ae-dump 子构建 remapJar）· `cgse.properties`（版本 + 兼容线） |
| `.github/workflows/release.yml` | 发布**仅** `workflow_dispatch` 手动触发、只允许 `main`、tag 已存在即拒 |

## 构建与跑测入口（实测口径）

| 动作 | 命令 |
|:--|:--|
| 纯库构件（**已验证**） | `java -classpath "runtime\gradle-8.8\lib\*" org.gradle.launcher.GradleMain -p <repo> -g runtime\gradle-home jar`（用 `runtime\jdk-17.0.2\bin\java.exe`；绕开 wrapper 联网下载） |
| 全量打包 | `... assemble`（产物 `build/release/*`） |
| 覆盖版本 | `... -PcgseVersion=0.3.3 assemble`（版本须为 stable `X.Y.Z` 且以兼容线为前缀，否则 gradle 直接抛） |
| 标准回归（**实测 411/412 PASS**；第 412 项 `PortfolioOracle` 默认 2000 例约 125 s，需单跑或减例数） | `python tests\run.py --java-home <repo>\runtime\jdk-17.0.2 --suite standard --timeout 600` |
| 标准回归 · 分批（单次调用 120 s 窗口内收口） | 加 `--shard i/N` 分批，或 `--case "<正则>"` 按名挑；`--no-compile` 复用上次 `classes`（⚠️ 该目录无 `classes/org` 时全部 `ClassNotFoundException`，不是真失败） |
| 全量回归 | `python tests\run.py --java-home <repo>\runtime\jdk-17.0.2 --suite all --milliseconds 0 --model-engine main-counts --output build/all-tests` |
| 数据集回放 | `python tests\datasets\run.py --java-home <repo>\runtime\jdk-17.0.2` |
| 用例清单 | `python tests\run.py --suite all --list-tests --output <dir>`（428 项） |
| 边界校验 | `python tools\verify_boundaries.py` |
| ABI 门禁 | `python tools\release\check_abi.py`（比对 `api/<line>.txt`） |

> 🔴 **toolchain 17 的满足方式**：`gradle/engine.gradle` 硬写 `toolchain.languageVersion = 17`。用 `runtime/jdk-17.0.2` 作为 Gradle 自身 JVM 后，Gradle 会按同一 JVM 满足 toolchain（实测 `jar` 直接成功，未触发下载）。**不改上游 gradle 文件**。
> 🔴 **grep 走 `cmd /c`**：受控根寻址对 `tests\run.py` 的 `--java-home` 参数不生效（外部程序参数位写真实路径）。
> ⚠️ **`verify_boundaries.py` 在基线上就是红的**：它用 `text.startswith('package org.cgse.core;')` 判定包声明，而核心源文件首行是 `// Copyright (c) 2026 autumn` → 几乎每个文件都误报 `unexpected package`（2026-11-02 实测，改前改后一致）。这是脚本与源码头的既有不一致，读它的输出时先减掉这一类假阳性。

**退出码语义（必须遵守）：** `0` = 所选检查全过 · `1` = 断言失败 / 编译或装配失败 / 夹具畸形 · **`2` = 预算未决（UNRESOLVED）——绝不可当 PASS，也绝不可当 UNSAT**。默认请求预算 60M work（= 宿主设置），历史对照用 `--work 40000000` / `20000000`。

## 关键判据（上游契约，改码前必读）

| # | 判据 | 出处 |
|:--:|:--|:--|
| 1 | Recipe ID 在一个 catalog 内唯一（含内容相同的重复对象）；binding 可重复。`GraphCompiler` 立即拒重复 ID | `docs/public-input-contracts.txt` |
| 2 | 构造时快照 inventory——之后调用方改 map 不影响运行中的 continuation；零计数归一掉；key 必须 non-null 且全程稳定 `equals`/`hashCode` | 同上 |
| 3 | adapter 的 `push()` 必须返回非 null `Outcome`——违约则 runtime 判 `IN_DOUBT`，保留不确定所有权与产出义务，**跨持久化与取消**，既不能退输入也不能重发同一 dispatch（provider 可能已接受） | 同上 |
| 4 | **预算耗尽（work / time / memory）不是不可行证明**——只有完整候选覆盖的 canonical proof 才能留存 | 同上 |
| 5 | 实际资源计数**不得转浮点**；`UNKNOWN` / 搜索上限不得当不可行；缺料与缺启动料分开报 | `docs/architecture.txt` |
| 6 | 版本号强校验：stable `X.Y.Z` 且前缀 = 兼容线，否则构建期抛 `GradleException` | `gradle/engine.gradle:9-14` |
| 7 | 可移植边界：核心只依赖 `java.base` + `java.management`；view 另需 `java.desktop` | `docs/migration-validation.txt:15-16` |
| 8 | 发布门禁三闸：`check_abi`（含抽象方法义务）→ `verify_artifacts` → 产物上传；**workflow 不跑回归**，本地需 `gradlew.bat build` | `docs/releases.txt` · `release.yml` |

## 范围与边界

1. **上游优先**——这是别人的仓库（`autumn` 维护）。改动方向以上游约定为准，不自作主张重构；本支既有纪律（C# 工具链 / CCBP 结构）不往这里搬。
2. **不碰宿主适配面**——Minecraft 渲染、包、菜单、本地化、`GraphJobCodec` 都在 **GTLCore**；本仓只做可移植核心 + 可选 view + 打包模板。
3. **`sync_gtl.py` 默认只读**——不带 `--apply` 只做比对；写 GTLCore 检出的动作属于跨仓写面，需显式授权。
4. **不伪造回归结论**——退出码 `2` 永远不写成 PASS；不跑过的测试写「未跑」，不写「应该没问题」。
5. **许可合规**——MPL-2.0：修改本仓文件后的分发需保留同许可；`LICENSE` / `NOTICE` 在 `cgseIdentityPaths` 内，构建身份摘要覆盖它们，改动会被记为 dirty。
6. **不向 `origin` push**——上游单分支，直接 push 会撞协作约定；本地 `main` 推 `fork`（`Rushiera/CGSE_CORE`），不提 PR。

### 裁决记录（2026-11-01 · Rushiera）

> **「结构不匹配就改结构，我们的目的是找出求解器问题并且解决掉，代价是那是后续上游看了成果再决定的事。」**

| 项 | 裁决 | 生效范围 |
|:--:|:--|:--|
| **上游代码可改** | ✅ 放行——本轮起允许改 `src/main/java/org/cgse/core/` 的结构（拒绝语义、契约面、配额链路），目标是**把问题解决掉**而非只报告 | 取消原「只做静态分析、不改上游一行」的限制 |
| **代价归属** | 改动代价（回归面 / 性能面 / ABI 兼容）**后置**至上游评审阶段——本支只负责把问题定位准、把解法做出来 | 不因"可能被上游拒"而降低改造深度 |
| **仍然不做** | 不向 `origin` push（上游只读）；不改适配面（GTLCore 在别仓）；不改 `LICENSE` / `NOTICE` | 2026-11-05 增补：本地改动推 `fork`（`Rushiera/CGSE_CORE`）· 不提 PR |

**执行含义：** 审计发现的 13 处 ADMIT_STATIC 结构不匹配（只接二元 / 要求宽度 ≤8 / 要求形状相等）——**可以改结构让它匹配**，不必停在"报告缺口"。补 `budget.note`、扩 `CountContinuation` 契约、调整 allowance 除数，均在此授权内。

## 未决项（诚实边界）

| # | 项 | 状态 |
|:--:|:--|:--|
| 1 | **协作形态**——2026-11-01 定为「只做静态分析，不推 git」；2026-11-05 改判：**推 fork `Rushiera/CGSE_CORE`** | ✅ 本地 `main` 推 fork，`origin` 仍只读、不提 PR；上游源码可改（代价后置上游评审） |
| 2 | 朋友的期望改动面尚未明示（性能？求解器覆盖面？适配层？） | 🟡 M1 / M1b / P1 / P1b 已交付——拒绝可归因 + 父层第三态 + **Obbt 两条改造路径实测收益 0/负**（size 闸实链 0 触发；入场闸唯一多给的机会 = 0 状态变化 + 66,326 步成本）· M7 配额分类清单已出（312 处）· **M2 A/B 已出**（`m2-quota-ab.md`：配额上限 ×4 零状态变化；87 条 2.5M 旧记录 0 复现）· **M2 归因已出**（`m2-unresolved-attribution.md`：分支顺序否证 · 3 例启发式到顶 + 1 例深度不足）· 下一轮待裁：窗口 4096 探针 / 分支层细分 / 结案 · **v0.17 已收口**：窗口探针（D3 判死）· 真实语料基线（19/19 PASS，funded x1000 用 95% 预算）· work 构成图（STOCK_VIEW 48–60% / COORDINATOR.OTHER 16–18% / 分支层 6.4%）· **下一步待裁：`COORDINATOR.OTHER` 细分 vs STOCK_VIEW 轮次 vs 结案**（详见 `analysis-strengthen.md` §六） |
| 3 | `docs/` 73 篇已建**逐篇关键词索引**（`docs-index.md`：逐篇卡 + 反向表 + 缺口清单）· 主题族导航（`analysis-docs.md` §一）· **73/73 逐字读过**（三篇超 `text-read` 20000 字符上限的按 `text-read_lines` 分段补读） | ✅ 已完成 |
| 4 | 主人格无 Java Job 可挂——Java 面方法论本支空白 | 🟡 若长期参与，考虑建 `L2/Job/Java`（判据：消费频率足够） |
| 5 | `runtime/` 在 `tools` 工作区内占约 500 MB（JDK + Gradle + gradle-home + build） | 🟢 已入 `.gitignore`；`runtime/` 可随时删、重建成本 = 两次解压 |

## 文档索引

| 文档 | 职责 | 加载 |
|:--|:--|:--|
| `project.md` | 定位 / 边界 / 判据 / 路由（本文件） | 🔴 上工必读 |
| `analysis-solver.md` | **求解器静态分析第一轮**——健全性主链核验（13 处 infeasible 证书门）+ 三个结构性可疑点 + 本地基线跑测 + 下轮建议 | 🔴 分析轮必读 |
| `docs-index.md` | **上游 73 篇关键词索引**——逐篇卡（主题 / 关键词 / 行号锚点）+ 反向关键词表（类名 / 概念 / 测试类 → 篇目）+ 未入文档的缺口清单 | 🔴 查上游文档先查这里 |
| `analysis-docs.md` | **上游 docs 精读第三轮**——73 篇主题族导航 + 九条精读结论 + 对前两轮报告的修订清单 + 重排靶点 | 🔴 读上游文档前必读 |
| `analysis-scaffold.md` | **结构审计脚手架 + 第二轮**——342 处 `complete=true` 全库分类（13 处不可绕过的沉默硬闸）· allowance 配额链路量化（1/8 切片定位）· 构造级探针 `AuditGate`（274 例准入判定）· 脚手架清单与复跑命令 | 🔴 审计轮必读 |
| `analysis-strengthen.md` | **求解器强化方向取证**——真实语料 19/19 PASS（funded x1000 = 95% 预算）· 整数 LP 死锁链与接通 A/B（零收益）· 窗口 4096→32768（状态 0 变化 / work +2.44%）· **work 构成图**（STOCK_VIEW 48–60% / COORDINATOR.OTHER 16–18% / 分支层 6.4%）· 七次零收益累积判读 · 复跑命令 | 🔴 强化轮必读 |
| `analysis-attribution.md` | **work 归因脚手架（w2）**——按 tick 加权的栈采样器 + 单点插桩 + 行级归因 · 自证与构成图逐行吻合 · **84.9% 的 work 是推进步数税**（32.3M 次 `check()`）· 失败模式分层（`MISSING_SEED` 吃 86% 轮次成本）· 时间片 32,768 轮转机制 · 循环内 check 上限仅 13.1% · 复跑命令 | 🔴 强化/归因轮必读 |
| `analysis-w3.md` | **有效推进归因（w3）**——轮次进度序列（单请求 65 轮 / 11 线；扩样 15 请求 / 161 线）· 三档停滞口径（P/Q/**R 缺口密度**）· 停滞占比 41.7%–60.4%（单请求）/ 40.9%（15 请求）· `stalledRounds` 非放弃判据的源码修正 · **C 组合闸（R k=3 M=2 零误杀 · 19.5%）· A 实测判死（+7.7% / +42% 含 1 例 ERROR）已回滚 · D 首轮特征零分离度 · B 成本与产出无关（0.87）→ B' 跨轮增量复用** · 复跑命令 | 🔴 强化/归因轮必读 |
| `analysis-w4.md` | **B' 段级脚手架（w4）**——region 级复用探针（region 身份 + 局部输入摘要 + 命中率）· `region_counts` 自身仅 **1.0%**（§10.2 的 46.2% 是 note 区间口径）· 段级成本 **29.67%**（↔ `RegionSelection.step` 30.8%）· 段命中率 **72.9% / 76.8%（成本加权）→ β 可省上界 12.58%** · 三条口径修正（累计量差值失效 / 单槽状态 / 未收口段）· 段内 `SummaryComputation` 占 65% · 复跑命令与验收判据 | 🔴 强化/归因轮必读 |
| `analysis-model-shape.md` | **模型形态侦察（v0.26）**——外部建议核查：packing 族零入场实测 + 四视图形态画像（无界 6% / 多值 46% / 混合符号行占多数）· 统一根因链（无界变量 → LP 29% + 全引擎域拒）· 8192/8193 边界 · 冲突面为零 · 探针脚手架 `.local/scaffold/probe.py` | 🔴 强化轮必读 |
| `analysis-w5.md` | **δ 可行性判定 + 真实负载直测（w5）**——段内 p3/p7 两次 `SummaryComputation` 的去重：机制合法（`SequenceSummary.repeat` 闭式 + `known` 复用有上游先例）· 真实负载 20,230 次计算 **0 值失配**（known 与纯代数双路）· **省 2.66% 总 work = p7 的 28.5%**（原「上界 7.6%」不成立）· **采样口径低估短相位 1.23×**（p7 直测 9.33% ↔ 采样 7.56%，根因 = 每次平均 175 work 远低于 1,024 work 采样阈值）· 尾波闭式已否证 · 复跑命令与分档 | 🔴 强化/归因轮必读 |
| `m6-baseline.md` | **M0 基线 + M6 改动前后对照**——双 Infinity 用例改前改后实跑输出 · 标准套件 / models 274 例 / ABI / jar 前后数 · 改动面与诚实边界 | 🟡 改码轮按需 |
| `m1-admission.md` | **M1 拒绝语义可归因**——31 处准入闸补 `admission_refused` · 四轴枚举（size / shape / quota / memory）· 改前改后观测 · 回归与复跑命令 | 🟡 改码轮按需 |
| `m1b-third-state.md` | **M1b 归因第三态**——`CountContinuation.refusal()`（A 案 default 方法）· 5 个实现类置值 · 父层消费（ViewSearch / Replay / Regression）· 274 例构造级 + 96 例回放前后对照 · 拍板记录 | 🟡 改码轮按需 |
| `p1-obbt-ab.md` | **P1 Obbt 弃权 A/B**——274 例 `main-counts` 强制上场 vs 原路径（代际核对 / 单变量构建 / 逐例对账 / 插桩归因 / 复跑命令）；结论：零收益，闸门在入场条件 | 🟡 裁决轮必读 |
| `m7-quota-census.md` | **M7 配额分母分类**——312 处除法点按「裸固定分母 / 双限式（②a 常量 · ②b 父层额度 · ②c 本地剩余）/ 非字面量除数 / 分母非请求剩余」分类 · 三口径对齐（312 / 247 / 246）· 族小计 · 焦点 12 处 · 上一轮定性的实测修正 | 🟡 改码轮按需 |
| `m7-allowance-census.md` | **M7 全量逐处表（机读附表）**——312 行：位置 / 族 / 分母对象 / 除数 / 形态 / 上限来源；判定器 `scan_allowance_v2.py` 产物 | 🟡 查具体调用点时读 |
| `m2-quota-ab.md` | **M2 配额复核 A/B**——D 靶面证伪（补丁未生效 · work 274/274 全同）· 配额上限 ×4 零状态变化 · 87 条旧 known-limits 对账（0 条 2.5M 复现）· 代际核对与复跑命令 | 🟡 改码轮必读 |
| `m2-unresolved-attribution.md` | **M2 未决归因**——4 例 UNRESOLVED 的插桩分解（分支层 / 视图层 / 引擎层三段切法）· 分支顺序否证（`branches=1`）· 100M 预算放大探针（1 例 1.68× 解出 / 3 例 5× 仍无 witness）· 4096 窗口硬帽与 MAX_QUANTUM 路径解释 · 待裁三案 | 🟡 裁决轮必读 |
| `m1c-silence.md` | **M1c 静默拒绝点 note+refusal 齐平 + trial 契约接线**——24 处静默出口补具名 note（实现类 15 处同补 `refusal()`）· P4 轴对齐 · `CountDomainSearch.trial` 契约接线（`faceInfeasible()` 分离面判定与导出）· AuditGate 274 例零变化对照 + M1cProbe 6/6 · 拍板记录 | 🟡 改码轮按需 |
| `workspace:tools/CGSE_CORE/.local/m4/out/m4-gate-census.md` | **M4-0 三闸触发面 census**——G1（RangedKernel）/G2（DD 宽度）/G3（Diophantine 形状）触发×终态直方图 + UNRESOLVED∩清单 + CRK 链 d 分布 + 复跑命令；探针 `M4GateProbe.java` + 驱动 `m4_gate_census.py` 同目录 | 🔴 M4 拍板轮必读 |
| `todo.md` | 未完成项 | 🔴 必要 |
| `m4-g2-g3.md` | **M4 三闸口径落地**——G1 留档 / G2 b1 改码（宽度闸 8→16）+ census 前后对照（274 例 status 零变化·收益 0）+ models pre8/post16 逐例对照 + core 全量 385 PASS + ABI 1502 + G3 三案实现层取证（c2 实测判否 0/27） | 🟡 改码轮按需 |
| `m4-g3-verdict.md` | **G3 岔口呈报**——c1 分组后跨组耦合可行组合设计 + 健全性论证（净收益面 2 例·分解形态实测否掉·变量分量 27/27=1）· G3/G3b 交集 0 勘误 · c3 已接线 · 当前字节 census + models 复跑 + core 385 PASS + ABI 1502 | 🟡 决策轨迹（拍板前口径） |
| `m4-g3-decision.md` | **G3 岔口拍板与落地**——取 c3（显式 note 收口）· c1 判死（放宽闸后 274 例终态 0 变化 · 两例 200M 预算仍 UNRESOLVED · 等式系统 RREF 结构）· 诊断行 `multi_shape_conflict` · core 385 / models 274 / ABI 1502 前后同数 | 🔴 G3 相关轮必读 |
| `CHANGELOG.md` | 已完成 | 🔴 必要 |
| `workspace:tools/CGSE_CORE/docs/architecture.txt` | 上游维护边界与入口点契约（**改码前首读**） | 🟡 按需 |
| `workspace:tools/CGSE_CORE/docs/public-input-contracts.txt` | 输入与结果契约（ID 唯一 / 快照只读 / IN_DOUBT / 预算≠不可行） | 🟡 按需 |
| `workspace:tools/CGSE_CORE/docs/releases.txt` | 发布链与 mods 落点 | 🟡 按需 |
| `workspace:tools/CGSE_CORE/tests/README.md` | 跑测面全集（suite 表 / 退出码 / 预算语义） | 🟡 按需 |
| `workspace:tools/CGSE_CORE/docs/migration-validation.txt` | 可移植边界与迁移校验基线 | 🟡 按需 |
| 其余 68 篇专题 | **检索走 `docs-index.md`**（关键词 / 类名 / 测试类反向表）；主题族导航 → `analysis-docs.md` §一 | 🟡 先查索引再读原文 |

## 素材清单

| 素材 | 位置 |
|:--|:--|
| 上游仓库 | `workspace:tools/CGSE_CORE/`（`origin` = GitHub 上游，`main` @ `b7c4a81`） |
| 上游文档 | `workspace:tools/CGSE_CORE/docs/`（73 篇） |
| 回归语料与夹具 | `workspace:tools/CGSE_CORE/tests/`（`cases/manifest.json` / `known-limits.json` / 各 suite） |
| 本机 JDK | `workspace:tools/REA/tools/jdk21`（21，可用于运行 Gradle；**不满足 toolchain 17**） |
| Python | 系统 `python` 3.12.10 |
