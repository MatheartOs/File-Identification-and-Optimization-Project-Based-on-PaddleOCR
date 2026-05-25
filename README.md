# Mathematical Contest In Modeling

本仓库是 2026 年第十一届数维杯大学生数学建模挑战赛（春季赛）B 题项目代码与结果说明入口。当前题目为“智能办公场景下多源异构文件识别与治理优化”，最新链路已经从早期环境测试脚本升级为：

```text
原始 B 题数据集
  -> 全量多源文件清洗与 PaddleOCR
  -> 问题一：历史文件主题体系归纳
  -> 问题二：新流入文件迁移归属与评价
  -> 问题三：人工复核优先级与资源约束优化
  -> final_results 五类论文交付表
```

当前项目事实源为 `docs/PROJECT_MAIN.md`；论文撰写材料见 `docs/problem1论文描述.md`、`docs/problem2论文描述.md`、`docs/problem3论文描述.md`、`docs/数学符号定义与说明.md` 和 `docs/附录与支撑材料.md`。

## 环境准备

推荐环境：

- Python 3.10.x
- Windows PowerShell
- Python 虚拟环境 `.venv`
- 全量图片 OCR 推荐使用 PaddleOCR / PaddlePaddle GPU
- 本地 OCR 模型目录：`models/paddleocr/{det,rec}`，或通过 `PADDLEOCR_DET_DIR`、`PADDLEOCR_REC_DIR` 指定

基础依赖安装：

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

如果 PowerShell 禁止激活脚本，可以在当前会话临时放开：

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

全量 OCR 前建议显式指定设备。GPU 环境使用：

```powershell
$env:PADDLEOCR_DEVICE='gpu:0'
```

CPU 环境可改为：

```powershell
$env:PADDLEOCR_DEVICE='cpu'
```

## 数据与目录

原始赛题数据放在仓库根目录：

```text
B题数据集/
```

该目录、`outputs/`、`models/`、`PP-OCRv5/` 均已在 `.gitignore` 中忽略，不提交到 git。

核心代码目录：

```text
src/mcm_b/
├── cleaning.py
├── problem1_innovative.py
├── problem2_transfer.py
├── problem3_optimization.py
└── paths.py

scripts/
├── run_b_cleaning.py
├── run_problem1_innovative.py
├── run_problem2_transfer.py
├── run_problem3_optimization.py
├── build_final_deliverables.py
└── recalibrate_cleaning_review.py
```

## 最新完整运行链路

在仓库根目录按顺序运行：

```powershell
$env:PADDLEOCR_DEVICE='gpu:0'

.\.venv\Scripts\python.exe scripts\run_b_cleaning.py `
  --output-dir outputs\b_problem\cleaning_ocr_full `
  --max-chars 30000 `
  --max-file-mb 25

.\.venv\Scripts\python.exe scripts\run_problem1_innovative.py `
  --cleaning-dir outputs\b_problem\cleaning_ocr_full `
  --output-dir outputs\b_problem\problem1_innovative `
  --clusters 10 `
  --max-terms 2500

.\.venv\Scripts\python.exe scripts\run_problem2_transfer.py `
  --cleaning-dir outputs\b_problem\cleaning_ocr_full `
  --output-dir outputs\b_problem\problem2_transfer `
  --clusters 10 `
  --max-terms 2500

.\.venv\Scripts\python.exe scripts\run_problem3_optimization.py `
  --cleaning-dir outputs\b_problem\cleaning_ocr_full `
  --problem2-dir outputs\b_problem\problem2_transfer `
  --output-dir outputs\b_problem\problem3_optimization

.\.venv\Scripts\python.exe scripts\build_final_deliverables.py `
  --cleaning-dir outputs\b_problem\cleaning_ocr_full `
  --problem1-dir outputs\b_problem\problem1_innovative `
  --problem2-dir outputs\b_problem\problem2_transfer `
  --problem3-dir outputs\b_problem\problem3_optimization `
  --output-dir outputs\b_problem\final_results
```

如果已经存在最新 `outputs/b_problem/cleaning_ocr_full/`，可以跳过第一步，直接重跑问题一、问题二、问题三和最终交付汇总。

## 各阶段输出

数据清洗输出：

```text
outputs/b_problem/cleaning_ocr_full/
├── cleaning_summary.json
├── processed/document_index.csv
├── processed/document_blocks.jsonl
├── processed/file_manifest.csv
├── processed/manual_check_list.csv
├── processed/manual_check_list_S1.csv
├── processed/manual_check_list_S2.csv
├── processed/manual_check_list_S3.csv
├── logs/parse_log.csv
└── logs/ocr_log.csv
```

问题一输出：

```text
outputs/b_problem/problem1_innovative/
├── problem1_graph_topic_assignments.csv
├── problem1_graph_topic_summary.csv
├── problem1_graph_topic_summary.md
└── problem1_graph_metrics.json
```

问题二输出：

```text
outputs/b_problem/problem2_transfer/
├── problem2_transfer_classification.csv
├── problem2_dataset_evaluation.csv
├── problem2_topic_distribution.csv
├── problem2_boundary_samples.csv
├── problem2_transfer_report.md
└── problem2_*.png / problem2_*_plot_data.csv
```

问题三输出：

```text
outputs/b_problem/problem3_optimization/
├── problem3_risk_priority.csv
├── problem3_level_summary.csv
├── problem3_special_focus.csv
├── problem3_scenario_comparison.csv
├── problem3_review_queue_S1.csv
├── problem3_review_queue_S2.csv
├── problem3_review_queue_S3.csv
├── problem3_optimization_report.md
└── problem3_*.png / problem3_*_plot_data.csv
```

最终论文交付表：

```text
outputs/b_problem/final_results/
├── data_preprocessing_statistics.csv
├── problem1_classification_result_table.csv
├── problem2_assignment_evaluation_table.csv
├── problem3_review_priority_table.csv
├── resource_scenario_comparison_table.csv
├── final_deliverables_report.md
└── data_preprocessing_file_type_distribution.png
```

## 当前模型摘要

问题一采用“统一文件画像 + 文本-单词异构图 + PMI 词图传播 + 结构业务特征融合 + 业务锚点增强 + c-TF-IDF 主题归纳”。最新结果从数据集 1 中筛选 2225 份历史文件，归纳出 10 类历史主题，包括资金财政统计类、医药项目审批类、地区统计指标类、制造业产业统计类、教育教学管理类、生态环境治理类等。

问题二复用问题一主题空间，对数据集 2 和数据集 3 的 4519 条新流入记录进行迁移归属，并输出主类概率、ARS 归属合理性、MII 模型解释指数、TAI 迁移适用性和治理状态。最新结果中，数据集 2 的 TAI 为 0.588393，数据集 3 的 TAI 为 0.699263。

问题三在问题二结果上构建紧急程度、错分风险、复核必要性和综合优先级评分，并结合数据集 4 的 S1/S2/S3 资源场景生成复核队列。最新场景复核数分别为 S1: 137、S2: 173、S3: 210，均优先覆盖未知专家研判样本。

## 验证与调试

轻量检查代码可导入和编译：

```powershell
.\.venv\Scripts\python.exe -m compileall src\mcm_b scripts
```

如只想检查数据目录结构，可运行：

```powershell
.\.venv\Scripts\python.exe scripts\inspect_b_data.py
```

如需在不重跑 OCR 的情况下重新校准清洗阶段复核清单，可运行：

```powershell
.\.venv\Scripts\python.exe scripts\recalibrate_cleaning_review.py `
  --cleaning-dir outputs\b_problem\cleaning_ocr_full
```

## 文档索引

| 文档 | 用途 |
| --- | --- |
| `docs/PROJECT_MAIN.md` | 项目主记录、最新命令、阶段结果和 TODO |
| `docs/题目.md` | 题面整理 |
| `docs/解析.md` | 建模建议与结果表要求 |
| `docs/多源异构文件数据清洗流程说明文档.md` | 数据清洗流程说明 |
| `docs/数据清洗环节论文参考说明.md` | 清洗章节论文素材 |
| `docs/problem1论文描述.md` | 问题一论文素材 |
| `docs/problem2论文描述.md` | 问题二论文素材 |
| `docs/problem3论文描述.md` | 问题三论文素材 |
| `docs/问题二求解.md` | 问题二正文式求解说明 |
| `docs/问题三求解.md` | 问题三正文式求解说明 |
| `docs/数学符号定义与说明.md` | 全文数学符号统一说明 |
| `docs/附录与支撑材料.md` | 附录、源程序和结果文件清单 |

## 协作约定

- 重要算法、参数、结果或命令变更后，优先同步 `docs/PROJECT_MAIN.md`。
- 原始数据、OCR 模型和输出结果不提交到 git。
- 论文手优先使用 `outputs/b_problem/final_results/` 下的五类核心表；需要细粒度分析时再查看问题一、二、三各自目录。
- 每张 PNG 图都配套同名 `.png.csv` 和 `_plot_data.csv`，便于论文中重新作图。
