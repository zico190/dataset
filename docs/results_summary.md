# SmartFlow: YOLOv8n Detection Results

**Model.** YOLOv8n, the vehicle-detection component of Raza et al., *IEEE Access* 13 (2025), doi:10.1109/ACCESS.2025.3602844.
- Re-implemented on public data, because the paper's Karachi dataset is not publicly available.
- Training settings from the paper's Table 7: AdamW, lr0 0.001, batch 16.
- Image size 640, COCO-pretrained weights, seed 42.
- Checkpoint selected on **validation** mAP@50–95; the **test** split was evaluated once.

**Source of every number below:**
- `metrics/summary_all_experiments.csv`
- `metrics/per_class_all_experiments.csv`
- each experiment's `report/metrics.json`

Produced by notebook 02 (ultralytics 8.4.160) and copied without rounding beyond 3 decimals.

## 1. Overall results (held-out test split)

| Exp. | Dataset | Classes | Train / Val / Test images | Precision | Recall | F1 | mAP@50 | mAP@50–95 |
|---|---|---|---|---|---|---|---|---|
| E1 | UA-DETRAC | 1 (vehicle) | 6,949 / 1,443 / 5,640 | 0.957 | 0.856 | 0.904 | **0.918** | **0.670** |
| E2 | BDD100K | 4 (car, bus, truck, motorcycle) | 20,000* / 7,000 / 10,000 | 0.616 | 0.447 | 0.518 | **0.491** | **0.318** |
| E3 | QMUL Junction | — | — | — | — | — | — | — |
| E4 | AI City Challenge 2021 | — | — | — | — | — | — | — |

\* A fixed random subset (seed 42) of the 63,000 BDD100K training images. Validation and test are complete.

**E3 and E4 were not evaluated.** Neither dataset provides object-detection bounding-box annotations:
- **QMUL Junction** is a single video with clip-level event labels.
- **AI City 2021** offers movement counts (Track 1), ReID crops (Track 2) and anomaly timestamps (Track 4). Its Track 3 boxes are class-agnostic and cover only vehicles seen by two or more cameras, and access requires an approved request.

Reporting accuracy for them would require manual annotation, so no values are given (see `docs/dataset_audit.md`).

## 2. Validation vs. test

| Exp. | Val mAP@50 | Test mAP@50 | Val mAP@50–95 | Test mAP@50–95 |
|---|---|---|---|---|
| E1 | 0.993 | 0.918 | 0.803 | 0.670 |
| E2 | 0.475 | 0.491 | 0.311 | 0.318 |

- **E1:** validation is noticeably more optimistic than test.
  - The validation sequences come from UA-DETRAC's official training sequences, which are likely recorded at the same sites.
  - The official test sequences are harder: about 12 vehicles per image versus about 7 in training.
- **E2:** validation and test agree closely.

## 3. Per-class results, E2 BDD100K (test)

| Class | Train instances | Test instances | Precision | Recall | F1 | mAP@50 | mAP@50–95 |
|---|---|---|---|---|---|---|---|
| car | 204,295 | 102,504 | 0.682 | 0.694 | 0.688 | 0.721 | 0.434 |
| bus | 3,355 | 1,597 | 0.568 | 0.449 | 0.501 | 0.498 | 0.378 |
| truck | 8,537 | 4,244 | 0.599 | 0.482 | 0.534 | 0.520 | 0.362 |
| motorcycle | 866 | 452 | 0.613 | 0.164 | 0.258 | 0.224 | 0.100 |

Accuracy tracks the amount of training data per class:
- **car** makes up 94% of training instances and scores highest.
- **motorcycle** has only 866 training boxes and is mostly missed (recall 0.16).

## 4. Training and efficiency

| Exp. | Epochs (best) | Stop reason | Training time | Weights | Params | GFLOPs @640 | Speed (batch 1, FP32) |
|---|---|---|---|---|---|---|---|
| E1 | 50 / 50 (50) | completed full budget | 1.39 h, Tesla T4 | 5.9 MB | 3.01 M | 8.2 | 4.9 ms/img = 204 FPS end-to-end, Tesla T4 |
| E2 | 27 / 50 (27) | stopped: free Colab GPU quota exhausted | 2.34 h, Tesla T4 | 5.9 MB | 3.01 M | 8.2 | 179.7 ms/img = 5.6 FPS, **CPU** (not comparable) |

## 5. Deviations and limitations (state these in the thesis)

1. **E2 trained 27 of 50 epochs.** Its loss curves were still falling and its validation mAP was still rising, so E2 is **under-trained**. Its numbers are a lower bound, not a converged result.
2. **E2 used a 20,000-image subset** of the training data, because of the Colab compute budget.
3. **Epoch budget.** The paper trained for 100 epochs; here the budget was 50. Only YOLOv8n was evaluated, not YOLOv7-tiny, YOLOv8s or YOLO-NAS-s. The paper's context-based augmentation, PCE density estimation, signal-control algorithm and Jetson/TensorRT deployment were not reproduced.
4. **Not directly comparable across rows.** E1 and E2 differ in number of classes (1 vs 4), viewpoint (fixed elevated camera vs dash-cam), training size and training length. E1 scoring higher reflects all of these factors, not only the dataset.
5. **Not comparable with the paper's 90% mAP / 74 FPS.** The datasets and classes differ, and the paper does not state its mAP IoU threshold or FPS protocol. Its 74.67 FPS was measured on a Tesla T4 (Table 12); on a Jetson Xavier NX it reports 35 FPS (Table 15).
6. **E2 speed was measured on CPU** and must not be compared with E1's GPU speed.
7. **BDD100K labels.** BDD100K uses the official **2018** detection labels. The 2020 labels' download server was unreachable.
8. **Confusion matrices.** They are produced by the mAP evaluation at confidence threshold 0.001. At that threshold the background column counts very low-confidence detections, so it is not a false-positive rate. Use the per-class P/R/F1 table above for interpretation.

## 6. Suggested wording

> We re-implemented the YOLOv8-nano vehicle detector of Raza et al. (2025) with the paper's reported training hyperparameters and evaluated it on two public traffic datasets. On UA-DETRAC (one vehicle class, fixed roadside cameras), the model reached a test precision of 0.957, recall of 0.856, F1 of 0.904, mAP@50 of 0.918 and mAP@50–95 of 0.670. On BDD100K (four vehicle classes, dash-cam imagery; trained on a 20,000-image subset for 27 of 50 planned epochs because of compute limits), it reached a test mAP@50 of 0.491 and mAP@50–95 of 0.318. Per-class results show strong performance on cars (mAP@50 0.721) and weak performance on the rare motorcycle class (0.224). QMUL Junction and AI City Challenge 2021 were excluded from the quantitative comparison because they provide no object-detection bounding-box annotations.
