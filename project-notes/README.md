# project-notes — 本协作方工作档案

> 性质：**发布快照**。这些文档在协作方本地是活档案，按轮次更新；本目录随 fork 一并发布，供上游评审与读者追溯「每一处改动是怎么被发现、怎么被验证的」。
> 快照时刻：2026-11-05（`README.md` / `CHANGELOG.md` v0.26 同批）。
> 路径记号：文中 `workspace:` / `ccbp:` 是协作方本地工作区的逻辑根前缀，读时按「仓库根」与「本目录」理解。

---

## 一、建议阅读顺序

| 序 | 文档 | 为什么按这个顺序 |
|:--:|:--|:--|
| 1 | `project.md` | 定位 / 边界 / 判据 / 构建入口 / 文档路由——先看这个 |
| 2 | `CHANGELOG.md` | 26 轮完成的全部工作，倒序；**先看顶部 3 条**即可掌握现状 |
| 3 | `analysis-solver.md` | 求解器第一轮静态分析：13 处证书门、三个结构可疑点——后续所有改造的问题源头 |
| 4 | `analysis-scaffold.md` | 结构审计脚手架：342 处 `complete=true` 的分类、配额链路量化 |
| 5 | `m1-admission.md` → `m1b-third-state.md` → `m1c-silence.md` | 三件套：拒绝可归因的完整实施链（31 处 → 第三态 → 静默点齐平） |
| 6 | `m6-baseline.md` | 数值精度改动（`repair` 平局）与基线对照 |
| 7 | `m4-g2-g3.md` → `m4-g3-verdict.md` → `m4-g3-decision.md` | M4 三闸：量测 → 呈报 → 拍板，含被判死的方案 |
| 8 | `analysis-strengthen.md` → `analysis-attribution.md` → `analysis-w3.md` → `analysis-w4.md` → `analysis-w5.md` | 性能归因五轮：work 构成 → 推进步数税 → 停滞口径 → 段级复用 → δ 判定 |
| 9 | `analysis-model-shape.md` | 模型形态侦察：为什么外部建议的「布尔装箱」方向不成立 |
| 10 | `p1-obbt-ab.md` · `m2-quota-ab.md` · `m2-unresolved-attribution.md` | 已判死方向的证据（避免后来人重复投入） |
| 11 | `docs-index.md` · `analysis-docs.md` | 上游 73 篇文档的逐篇索引与主题族导航 |
| 12 | `m7-quota-census.md` · `m7-allowance-census.md` | 312 处配额除法点的逐处台账（查具体调用点时用） |
| 13 | `todo.md` | 未完成项 |

---

## 二、逐篇职责

| 文档 | 职责 |
|:--|:--|
| `project.md` | 项目档案：定位 / 范围边界 / 裁决记录 / 构建与跑测入口 / 8 条关键判据 / 文档索引 |
| `CHANGELOG.md` | 已完成项唯一真相，v0.1–v0.26 倒序 |
| `todo.md` | 未完成项（按 环境 / 分析 / 审计 / 改造 / 文档 前缀登记） |
| `analysis-solver.md` | 求解器静态分析第一轮：健全性主链核验（13 处 infeasible 证书门）+ 三个结构性可疑点 + 本地基线 + 下轮建议 |
| `analysis-scaffold.md` | 结构审计脚手架 + 第二轮：342 处 `complete=true` 全库分类（13 处沉默硬闸）· allowance 配额链路 · 构造级探针 `AuditGate` |
| `analysis-docs.md` | 上游 docs 精读第三轮：73 篇主题族导航 + 九条结论 + 前两轮报告的三处定性修订 |
| `docs-index.md` | 上游 73 篇逐篇关键词索引：逐篇卡（主题 / 关键词 / 行号锚点）+ 反向关键词表 + 缺口清单 |
| `analysis-strengthen.md` | 强化方向取证：真实语料 19/19 PASS · 整数 LP 死锁链 · 七次零收益累积判读 · work 构成图 |
| `analysis-attribution.md` | work 归因脚手架（w2）：tick 加权栈采样 + 行级归因 · 84.9% 的 work 是推进步数税 |
| `analysis-w3.md` | 有效推进归因（w3）：轮次进度序列 · 三档停滞口径 · 停滞占比 41.7%–60.4% |
| `analysis-w4.md` | B' 段级脚手架（w4）：region 级复用探针 · 段命中率 72.9% / 76.8% |
| `analysis-w5.md` | δ 可行性判定：段内 summary 去重 · 真实负载 20,230 次 0 值失配 · 省 2.66% 总 work |
| `analysis-model-shape.md` | 模型形态侦察（v0.26）：packing 族零入场 · 四视图形态画像 · 统一根因链（无界变量） |
| `m1-admission.md` | M1 拒绝语义可归因：31 处准入闸补 `admission_refused` · 四轴枚举 · 前后观测 |
| `m1b-third-state.md` | M1b 归因第三态：`CountContinuation.refusal()` · 实现类置值 · 父层消费 · 274 例构造级 + 96 例回放对照 |
| `m1c-silence.md` | M1c 静默拒绝点齐平（24 处出口）+ `trial` 契约接线 |
| `m6-baseline.md` | M0 基线 + M6 改动前后对照（双 Infinity 用例 / 标准套件 / models / ABI / jar） |
| `m4-g2-g3.md` | M4 三闸口径落地：G2 宽度闸 8→16（收益 0）+ G3 三案取证 |
| `m4-g3-verdict.md` | G3 岔口呈报：c1 组合设计与健全性论证（拍板前口径） |
| `m4-g3-decision.md` | G3 岔口拍板与落地：取 c3 显式 note 收口，c1 判死 |
| `m7-quota-census.md` | M7 配额分母分类：312 处除法点按四形态分类 · 族小计 · 焦点 12 处 |
| `m7-allowance-census.md` | M7 全量逐处表（机读附表，312 行） |
| `m2-quota-ab.md` | M2 配额复核 A/B：配额上限 ×4 零状态变化 · 87 条旧 known-limits 对账 |
| `m2-unresolved-attribution.md` | M2 未决归因：4 例 UNRESOLVED 的插桩分解 · 分支顺序否证 · 100M 预算放大探针 |
| `p1-obbt-ab.md` | P1 Obbt 弃权 A/B：274 例强制上场 vs 原路径，结论零收益 |

---

## 三、同步说明

- 本目录是**快照**，不承担实时维护；协作方本地档案更新后，随下一次提交重新同步。
- 上游文档（`docs/` 73 篇）不在此目录，也不被本协作方修改；检索请用 `docs-index.md`。
