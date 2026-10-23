# Comparison table: the paper's algorithm on the other papers' datasets

**Algorithm.** The vehicle detector of Raza et al. (IEEE Access, 2025): YOLOv8-nano, fine-tuned with the paper's settings (AdamW, lr 0.001, batch 16, image size 640).

**Evaluation.** One held-out test split per dataset, evaluated once after training.

| # | Dataset (paper) | Classes | Test images | Precision | Recall | F1 | mAP@50 | mAP@50–95 | Note |
|---|---|---|---|---|---|---|---|---|---|
| 1 | UA-DETRAC | 1 | 5,640 | 0.957 | 0.856 | 0.904 | **0.918** | **0.670** | full training data, 50 epochs |
| 2 | BDD100K | 4 | 10,000 | 0.598 | 0.456 | 0.517 | **0.498** | **0.325** | 20,000-image training subset, 50 epochs |
| 3 | QMUL Junction | — | — | — | — | — | — | — | not evaluable: video with no bounding-box labels |
| 4 | AI City Challenge 2021 | — | — | — | — | — | — | — | not evaluable: no complete bounding-box labels; access by request |
| 5 | Paper's own dataset (Karachi) | 8 | 272 | not reported | not reported | not reported | 89.30%* | not reported | *quoted from the paper (Table 12); dataset is private and was not re-evaluated here |

**Highest result: UA-DETRAC.** It is a single class from fixed traffic cameras, with few small objects (7% of boxes are smaller than 32×32 px), against 42% small objects in BDD100K.

**The datasets differ in classes, viewpoint, object size and training size,** so the table describes performance on each dataset; it is not a fair ranking of the datasets.

The paper does not state which IoU threshold its "mAP" uses, so its 89.30% cannot be placed in our mAP@50 or mAP@50–95 columns.

**Effect of fine-tuning.** Zero-shot COCO baseline vs fine-tuned mAP@50:
- UA-DETRAC: 0.879 → 0.918
- BDD100K: 0.289 → 0.498

Source: `final_report.md`, generated from the saved run artefacts.
