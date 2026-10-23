# SmartFlow — YOLO-based vehicle detection: submission package

**Graduation project:** SmartFlow, an AI-based traffic management system.

**Task:** re-implement the vehicle-detection component of the reference paper, and evaluate it on the datasets of the related papers.

**Reference paper:** M. Raza et al., "An Edge-Deployed Real-Time Adaptive Traffic Light Control System Using YOLO-Based Vehicle Detection and PCE-Aware Density Estimation," *IEEE Access*, vol. 13, pp. 153586–153613, 2025, doi:10.1109/ACCESS.2025.3602844.

## Main result

YOLOv8-nano, fine-tuned with the paper's training settings and evaluated on held-out test splits:

| Dataset | Classes | mAP@50 | mAP@50–95 | Precision | Recall | F1 | Status |
|---|---|---|---|---|---|---|---|
| UA-DETRAC | 1 (vehicle) | **0.918** | **0.670** | 0.957 | 0.856 | 0.904 | evaluated (E1) |
| BDD100K | 4 (car, bus, truck, motorcycle) | **0.498** | **0.325** | 0.598 | 0.456 | 0.517 | evaluated (E2c, 50 epochs) |
| QMUL Junction | — | — | — | — | — | — | excluded: no bounding-box annotations |
| AI City Challenge 2021 | — | — | — | — | — | — | excluded: no complete bounding-box annotations |

**What the numbers mean:**
- mAP is a **detection** metric (localisation plus ranking), not classification accuracy.
- The rows are **not directly comparable**: they differ in classes, viewpoint, object size and training data.
- The results are **not a reproduction of the paper**: the paper's Karachi dataset is private, and its evaluation protocol is not fully reported. The paper's own numbers appear separately, quoted, in the report.

## Folder contents

| Folder | Contents |
|---|---|
| `01_report/` | `final_report.md`: full comparison report (generated from the run artefacts) · `thesis_results_chapter.md`: draft Results & Discussion chapter · `dataset_audit.md`: paper methodology extraction and dataset audit · `comparison_table.md`: summary table for the supervisor |
| `02_figures/` | Baseline vs fine-tuned, training curves, per-class results, example predictions, PR curve |
| `03_metrics/` | `final_comparison.xlsx` (6 sheets) plus one CSV per sheet: comparison, per-class, dataset statistics, held-out checks, paper-reported values (quoted), YOLOv8n vs paper |
| `04_code/` | The three Google Colab notebooks: 01 data preparation → 02 training & evaluation → 03 comparison & report |
| `05_models/` | Trained checkpoints (`best.pt`). **These must be copied from Google Drive; see the checklist below.** |

## Experiments

| ID | Dataset | Training data | Epochs | Notes |
|---|---|---|---|---|
| E1 | UA-DETRAC (HF mirror `blues111/ua_detrac`, rev `206bb8fa86cd`) | 6,949 frames (sequence-level split) | 50 / 50 | ignored regions masked |
| E2 | BDD100K (100K images, official 2018 labels) | fixed 20,000 of 63,000 images, seed 42 | 27 / 50 | stopped when the free Colab GPU quota ran out |
| E2c | same as E2 | same subset | 50 / 50 | E2 continued from epoch 27; E2's results kept unchanged |
| E3 | QMUL Junction | — | — | not evaluated: no detection boxes |
| E4 | AI City Challenge 2021 | — | — | not evaluated: access by request; counts, ReID crops, anomaly times and partial multi-camera tracks only |

**Shared settings.**
- **Model and training:** YOLOv8n, initialised from COCO-pretrained weights; image size 640; AdamW; lr0 0.001; batch 16; seed 42.
- **Checkpoint selection:** on validation mAP@50–95; each selected checkpoint was tested once.
- **Software and hardware:** Ultralytics 8.4.160 on an NVIDIA Tesla T4 (Google Colab).

## How to reproduce

Open the notebooks in Google Colab, in order, with a GPU runtime. Each one is safe to re-run: it resumes or reuses finished work.

1. **`SmartFlow_01_data_preparation.ipynb`.** Downloads UA-DETRAC (Hugging Face mirror) and BDD100K (UC Berkeley server), verifies them, converts them to YOLO format, builds leakage-free splits and writes the data gate.
2. **`SmartFlow_02_training_evaluation.ipynb`.** Runs the preflight checks, a smoke test, the experiments, and validation/test evaluation.
3. **`SmartFlow_03_comparison_report.ipynb`.** Evaluates the zero-shot COCO baseline, re-measures speed on one device, and builds the comparison tables, figures and report.

## Before submitting: files to add from Google Drive

These files live in your Drive (`MyDrive/SmartFlow/`) and could not be copied here. Download them and put them in the folders shown.

| Put in | File on Drive |
|---|---|
| `05_models/E1_UA-DETRAC_best.pt` | `checkpoints/E1_ua_detrac_mirror_1cls__yolov8n_img640_e50_s42_e92689f0d0/weights/best.pt` |
| `05_models/E2c_BDD100K_best.pt` | `checkpoints/E2c_bdd100k_4cls_train20000__yolov8n_img640_e50_s42_e92689f0d0/weights/best.pt` |
| `05_models/E2_BDD100K_27ep_best.pt` *(optional)* | `checkpoints/E2_bdd100k_4cls_train20000__yolov8n_img640_e50_s42_e92689f0d0/weights/best.pt` |
| `02_figures/final_1_test_metrics.png` | `reports/final_report/final_1_test_metrics.png` |
| `02_figures/E1_test_conf0.25_confusion_matrix_normalized.png` | `reports/final_report/` (same name) |
| `02_figures/E2_test_conf0.25_confusion_matrix_normalized.png` | `reports/final_report/` (same name) |
| `02_figures/E2c_test_conf0.25_confusion_matrix_normalized.png` | `reports/final_report/` (same name) |
| `02_figures/E2c_test_predictions.png` *(optional)* | `plots/E2c_bdd100k_4cls__report_test_predictions.png` |

Each checkpoint is about 6 MB.

**Do not use the confusion matrices saved by notebook 02.** They were produced at confidence 0.001 and their background column is misleading. Use the conf-0.25 matrices listed above.

## Data and licences

- **UA-DETRAC:** used via a third-party Hugging Face mirror; the authors' page states no licence, so it is treated as research-only.
- **BDD100K:** UC Berkeley licence (educational, research and not-for-profit use).

Neither dataset is redistributed in this package. Use the notebooks to download them from their sources.
