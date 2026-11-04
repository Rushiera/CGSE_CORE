# CGSE_CORE — 待办

> 行动进度唯一真相——只列未完成项，完成即核销出表（历史归 `CHANGELOG.md`）。
> 单条 ≤50 字符；复杂项写「详见 xxx.md §N」路由。
> 前缀按面登记：`环境` / `协作` / `分析` / `审计` / `改造` / `文档`。

## 环境（✅ 已通）

> JDK 17.0.2 与 Gradle 8.8 已落 `runtime/`，`.gitignore` 已排除 `/runtime/`——`jar` 37s 通过、标准套件 26/26。详见 `CHANGELOG.md` v0.2。

## 分析（求解器）

- `分析` packing 族结案——域非 0/1，11 请求零入场（详见 `analysis-model-shape.md`）
- `分析` 15~17 无界变量污染全引擎准入，LP 仅覆盖 29%（同上）
- `分析` **δ 判定完成，待裁（ε1 / δ2 / δ3）**：真实负载直测 **省 2.66% 总 work**（= p7 的 28.5%），20,230 次计算 0 值失配；原「上界 7.6%」不成立，详见 `analysis-w5.md` §五
- `分析` **采样口径低估短相位 1.23×**（p7 直测 9.33% ↔ 采样 7.56%，均值 175 work ≪ 1,024 阈值）→ β 12.58% / ε 14.6% 两个对照数需按直测口径复核，详见 `analysis-w5.md` §3.3

- `分析` **6 例 UNRESOLVED 单独收口**（Rushiera 2026-11-05 点名）：count-model 4 例 `generated/sparse-equality-n{144,257}-s{1,2}`（3 例 100M 无 witness · 1 例 33.5M 可解）+ graph 2 例 `pool_mix7__perm0/perm1` 排列超时——清单与复跑命令见 `CHANGELOG.md` v0.24，归因见 `m2-unresolved-attribution.md`

## 审计（结构，见 `analysis-scaffold.md`）


## 改造（改码，2026-11-01 授权放行）

> 授权：`project.md` §范围与边界「裁决记录」——结构不匹配就改结构，代价后置上游评审。
> 约束不变：不 push、不改适配面（GTLCore 在别仓）、不改 LICENSE / NOTICE。
> 每项交付 = 改动 + 可复跑跑测 + 前后对照数；回归不绿即回滚。

- `改造` M3 `CountObbt` 结构放宽：**P1/P1b 实测收益 0/负 → 待裁（不建议直接开工）**，详见 `p1-obbt-ab.md`
- `改造` M5 全量回归 + ABI 门禁：`tests/run.py --suite all` + `check_abi.py` + `verify_boundaries.py`（该脚本基线假阳性见 `project.md` §构建与跑测入口）——已跑 core 全量 385（唯一 385）+ models 274 + ABI 1502，`--suite all` 未跑
- `改造` 分支层 22–28% 细分 / 结案：待裁（窗口探针 D3 已判死，见 `analysis-strengthen.md` §三）
- `改造` `CountNeighborhood:346` 试算面 `.trial(true)` 接线待裁（`infeasible()` 消费面审计先行）——详见 `m1c-silence.md` §六
- `改造` 非 `complete=true` 族归因待裁：`CKS:228` 预算/精度 catch · `CPLS:217` state<0 · 工厂 `return null` 6 处 · `throw Declined` 2 处——详见 `m1c-silence.md` §六
- `改造` `GraphCompiler` maxRecipes 8192→8193 试跑（1 单位卡整模型，见 `analysis-model-shape.md` §四）
- `改造` 查 `count_lp_structure: prepared=0` 调用链为何不触发（同上 §七）

## 文档（索引见 `docs-index.md`）

- `文档` 前两轮报告的三处定性修订（`analysis-docs.md` §三）待回写进原文正文
