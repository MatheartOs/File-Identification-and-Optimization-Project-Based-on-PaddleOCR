# File-Identification-and-Optimization-Project-Based-on-PaddleOCR

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

## 完整运行链路

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

如果已经存在最新 `outputs/b_problem/cleaning_ocr_full/`，可以跳过第一步。

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