# R01-01 工作区长期记忆

## 项目性质
ThreeC Historical Universe Expansion 的 **R01-01（高端装备 / 机器人）** 独立研究工作区。
`standalone = true`。本工作区只负责 **Historical Research → Research Intake Package**，
不负责 Canonicalization → DB Import → Product。交付即停止，**不得进入 ThreeC 主仓库**。

## 环境约束（每次会话必读）
- **bash 工具不可用**（shim 损坏：`dirname: command not found`）。→ 一律用 **PowerShell 工具**。
- **PowerShell stdout 不内联回显**（只返回 "Command completed with exit code 0"）。
  → 重定向到 `$env:TEMP\xxx.txt`，再用 Read 工具读。
- Python：`C:\Users\86115\.workbuddy\binaries\python\versions\3.13.12\python.exe`
- Read / Grep / Glob / Edit / Write 工具正常。

## 三条验收命令（交付前必须全绿）
```
"C:\Users\86115\.workbuddy\binaries\python\versions\3.13.12\python.exe" tools\validate_historical_research_intake.py 05_OUTPUT   → RESULT: PASS / 0 FAIL
... validate_historical_research_intake.py --check                                                                              → RESULT: PASS
... test_validate_historical_research_intake.py                                                                                 → tests: 37 / failed: 0
```

## 不要动的东西
- `00_TASK/` `01_PROTOCOL/` `02_BASELINE/` `03_REFERENCE/` `04_RESEARCH/` `tools/` — 全部只读。
- `workspace_manifest.json` 的 `task_status` 必须保持 `PREPARED_NOT_STARTED`（`--check` 依赖它）。
- `05_OUTPUT/README.md` 是既有契约说明，不属于交付文件，不要删改。
- `workspace_checksums.sha256` 是工作区规则版本锁定，改动规则文件会导致 C20 FAIL。

## 交付契约（11 文件，一个不能少）
`manifest.json` `coverage.md` `candidates.json` `evidence.json` `sources.json` `securities.json`
`exclusions.json` `conflicts.json` `research_questions.json` `quality_summary.json` `checksums.sha256`
（`DIR_FILES` 定义见 validator；`coverage.md` 与 `checksums.sha256` 非 JSON。）

## 校验器易踩的坑（已实测）
1. **ID 正则**：`^R\d{2}-[A-Z0-9]+-[A-Z]*\d{3,}$` → `R01-HIEQ-001` 合法。canonical ID 前缀（`C-2024-*` / `RC-*` / `CC-*` / `TH-*` / `rule_*`）会 C13 FAIL。
2. **禁用字段名（C14）**：`campaign_id` `start_date` `end_date` `peak_date` `strength` `result` `classification` `macro_theme_id` `macro_theme_ids`。
   - ✅ `classification_proposal`  ✅ `research_status`  ❌ `classification`  ❌ `strength`
3. **禁用评分键（C15）**：后缀 `_score` `_rank` `_ranking` `_probability` `_prediction` `_forecast` `_win_rate`。
4. `campaign_candidate.macro_theme_proposal` 是 **字符串**（不是对象引用）。
5. `**confidence_distribution` 只统计 candidate 级 confidence**（不含 evidence）。
6. `counts` 各项必须 **精确等于** 对应数组长度（C21）。
7. **全文件 LF-only**，任何 CRLF → C20 FAIL。
8. `checksums.sha256` 行格式：`<64位hex><两空格><文件名>`，且覆盖除自己以外的 10 个文件。
9. `quality_summary.no_quantity_kpi_acknowledged` 必须 `true`（C22 const）。
10. `exclusions` minItems = 1。
11. `date_precision` 条件必填（C12）：`EXACT_DATE`→必须 `date`；`DATE_WINDOW`→必须 `window_start`+`window_end`；`PHASE_WINDOW`→必须 `phase_note`。
12. `temporal_relation` 为 `subsequent`/`retrospective` 时 `point_in_time_note` 必须非 null（C19）。

## 角色边界（不得越界）
- **不得自裁 Macro Theme**：只提交 `macro_theme_proposals` + `theme_name_candidates`，最终由 ThreeC Agent 走 CMTR v1（`canonical-macro-theme-resolution-1`）解析。
- **不得自裁 Campaign 独立性**：由 ThreeC Agent 走 Research Model v1.0 §5 + Campaign Independence Gate Q1–Q5。
- **不得因可能重复而删除候选**：只标记 `possible_duplicate_of` / `cross_task_notes`。
- **不得声明 VERIFIED**：`research_status` 只能是 `PROVISIONAL` / `CONFLICT` / `INSUFFICIENT`。
- **不做数量 KPI**：候选/证据/来源数量是研究结果，不是目标。
- **不运行** Time Observation、Structural Analogy。

## 任务边界（R01-01 对外）
| 归属 | 内容 |
|---|---|
| R01-02 | 半导体设备与材料 |
| R01-06 | 纯军工订单驱动的装备 |
| R01-03 | 资源/化工（原材料成本、大宗商品周期） |
| R01-04 | 3C 消费电子与家电制造设备 |
| 已有 family「汽车」 | 整车/零部件、智驾、Robotaxi |
| 已有 family「电力设备」 | 工控与电力自动化企业重叠（如汇川技术） |

## 本任务已交付状态
`05_OUTPUT/` 完整，三条验收命令全绿（24 checks / 0 FAIL；37 tests / 0 failed）。
7 候选 / 41 证据 / 35 来源 / 17 标的 / 10 排除 / 8 冲突 / 12 研究问题。
状态：**研究完成，已停止，等待 ThreeC Agent 评审**。
