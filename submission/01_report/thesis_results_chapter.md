# Results and Discussion (draft thesis chapter)

> **Source of every number below:** `reports/final_report/final_report.md`, generated on 2026-10-23 by notebook 03 from the saved run artefacts. No value was entered by hand. Values are rounded to three decimals.
>
> **Terminology:** mAP is the mean average precision of an object detector, a localisation-and-ranking metric. It is **not** classification accuracy.

## 1. Experimental setup

**Detector.** We re-implemented the vehicle-detection component of Raza et al. [1], a YOLOv8-nano detector. It was fine-tuned from COCO-pretrained weights (`yolov8n.pt`; 3.01 M parameters; 8.2 GFLOPs at 640 px; 5.9 MB checkpoint) using the training settings the paper reports for YOLOv8-nano: AdamW, initial learning rate 0.001, batch size 16. Image size was 640 px. The epoch budget was 50, with early-stopping patience 20 on the validation metric, seed 42 and Ultralytics' default augmentation. All runs used Ultralytics 8.4.160 on an NVIDIA Tesla T4 in Google Colab.

**Datasets.** Four public traffic datasets were audited. Two provide object-detection bounding boxes and were used:
- **UA-DETRAC.** Fixed, elevated surveillance cameras; one `vehicle` class. Split by video sequence: 6,949 / 1,443 / 5,640 train / val / test frames.
- **BDD100K.** Dash-cam images with the official 2018 detection labels, mapped to four classes: car, bus, truck, motorcycle. The test set is the official validation split (10,000 images); our validation set is a seeded 10% of the official training split (7,000 images).

QMUL Junction and AI City Challenge 2021 were **excluded**. Neither offers detection bounding boxes: QMUL provides clip-level event labels, and AI City provides counts, re-identification crops, anomaly timestamps and partial multi-camera tracks. Evaluating a detector on them would require new manual annotation.

**Experiments.**

| ID | Dataset | Training data | Epochs |
|---|---|---|---|
| E1 | UA-DETRAC | full training split | 50 / 50 |
| E2 | BDD100K | fixed random subset of 20,000 of 63,000 training images (compute budget) | 27 / 50 (GPU quota exhausted) |
| E2c | BDD100K | same subset as E2 | E2 continued from epoch 27 to 50, identical data and settings |

**Protocol.** The checkpoint with the best validation mAP@50–95 was selected. Each selected model was then evaluated once on the held-out test split using Ultralytics `val()` (conf 0.001, NMS IoU 0.7). Leakage checks passed for all experiments: no shared or byte-identical images across splits, file lists unchanged since validation, training configuration without access to the test split, and selection on validation only.

**Pretrained baseline.** As a reference point, the same COCO-pretrained model was evaluated **zero-shot** on each test split, with its classification head restricted to the corresponding COCO classes:
- UA-DETRAC: car + bus + truck → vehicle;
- BDD100K: car, bus, truck, motorcycle mapped one-to-one.

## 2. Overall detection results

**Table 1. Test-split results (held-out).**

| Exp. | Dataset | Classes | Precision | Recall | F1 | mAP@50 | mAP@50–95 |
|---|---|---|---|---|---|---|---|
| E1 | UA-DETRAC | 1 | 0.957 | 0.856 | 0.904 | 0.918 | 0.670 |
| E2 | BDD100K | 4 | 0.616 | 0.447 | 0.518 | 0.491 | 0.318 |
| E2c | BDD100K | 4 | 0.598 | 0.456 | 0.517 | 0.498 | 0.325 |

**UA-DETRAC.** The detector reaches mAP@50 0.918 and mAP@50–95 0.670. Precision (0.957) is higher than recall (0.856), so the model rarely produces false detections but misses roughly one in seven labelled vehicles at its F1-optimal operating point.

**BDD100K.** Results are substantially lower: mAP@50 0.498 and mAP@50–95 0.325 for E2c.

**These two results should not be read as a ranking of the datasets.** The tasks differ in:
- number of classes (1 vs 4);
- viewpoint (fixed elevated camera vs moving dash-cam);
- object size (Section 6);
- annotation conventions;
- training-set size.

**Validation vs. test.** On UA-DETRAC the validation mAP@50–95 (0.803) is clearly higher than test (0.670). The validation sequences come from UA-DETRAC's official training sequences, while the official test sequences are denser: 12.1 vs 7.3 labelled vehicles per image. On BDD100K, validation and test agree closely.

## 3. Effect of fine-tuning

**Table 2. Zero-shot COCO baseline vs. fine-tuned (test split).**

| Exp. | Baseline mAP@50 | Baseline mAP@50–95 | Fine-tuned mAP@50 | Fine-tuned mAP@50–95 | Δ mAP@50 | Δ mAP@50–95 |
|---|---|---|---|---|---|---|
| E1 | 0.879 | 0.606 | 0.918 | 0.670 | +0.039 | +0.064 |
| E2 | 0.289 | 0.185 | 0.491 | 0.318 | +0.202 | +0.133 |
| E2c | 0.289 | 0.185 | 0.498 | 0.325 | +0.209 | +0.140 |

**UA-DETRAC.** The off-the-shelf COCO detector is already strong (mAP@50 0.879). Fine-tuning adds a moderate gain, larger in the stricter mAP@50–95 (+0.064). This indicates that fine-tuning mainly improves box localisation and adaptation to the dataset's labelling convention.

**BDD100K.** Fine-tuning is essential: mAP@50 rises from 0.289 to 0.498. Part of this gap reflects class-definition differences between COCO and BDD100K, as well as the dash-cam domain, not only visual ability.

## 4. Effect of training longer (E2 → E2c)

With identical data and settings, extending training from 27 to 50 epochs gave:

| | mAP@50 | mAP@50–95 | Recall | Precision |
|---|---|---|---|---|
| E2 (27 epochs) | 0.491 | 0.318 | 0.447 | 0.616 |
| E2c (50 epochs) | 0.498 | 0.325 | 0.456 | 0.598 |
| Change | +0.007 | +0.006 | +0.009 | −0.018 |

The validation-selected best epoch of E2c was **38**, and the validation curves are flat after roughly epoch 35. The extra epochs therefore produced only a small improvement. With a single seed, a difference of this size cannot be shown to exceed run-to-run variation.

The result suggests that, for this setting, the limiting factors are not training length but:
- the amount of training data per class;
- the proportion of small objects (Section 6).

## 5. Per-class results (BDD100K, E2c, test)

**Table 3. Per-class test results.**

| Class | Training instances | Test instances | Precision | Recall | F1 | mAP@50 | mAP@50–95 |
|---|---|---|---|---|---|---|---|
| car | 204,295 | 102,504 | 0.705 | 0.694 | 0.699 | 0.728 | 0.439 |
| bus | 3,355 | 1,597 | 0.582 | 0.459 | 0.513 | 0.508 | 0.386 |
| truck | 8,537 | 4,244 | 0.604 | 0.488 | 0.540 | 0.528 | 0.370 |
| motorcycle | 866 | 452 | 0.500 | 0.181 | 0.266 | 0.227 | 0.103 |

With the rule mAP@50 < 0.5 or recall < 0.5:
- **Bus, truck and motorcycle are weak.** Bus and truck are flagged only by their recall; their mAP@50 is just above 0.5.
- **Car is the only strong class.** It accounts for about 94% of training instances.
- **Motorcycle is clearly the weakest class** (mAP@50 0.227, recall 0.181). It is also the rarest, with 866 training instances.

For all three weak classes, recall is below precision. Many of their instances are not detected **as that class**: they are either missed or assigned another vehicle class. The confusion matrices at confidence 0.25 (report, Section 5) distinguish the two cases.

## 6. Dataset factors

**Table 4. Measured test-set statistics.**

| Dataset | Resolution | Objects / image | Median box area | Small (< 32² px) | Medium | Large |
|---|---|---|---|---|---|---|
| UA-DETRAC | 960×540 | 12.1 | 4,012 px² | 7.0% | 67.3% | 25.7% |
| BDD100K | 1280×720 | 10.9 | 1,473 px² | 42.3% | 37.9% | 19.8% |

Besides the class count, the clearest measured difference is **object size**:
- In BDD100K, **42%** of labelled vehicles are smaller than 32×32 px, against **7%** in UA-DETRAC.
- BDD100K's median box is less than half as large.

Small objects are the hardest case for a nano-sized detector at 640 px input. This is consistent with the lower BDD100K scores, but it is not isolated as the cause: class count, viewpoint, annotation and training budget also differ.

Other documented differences:
- **Annotation.** UA-DETRAC has ignore regions, which we masked. BDD100K has crowd-sourced labels; we used the 2018 version, because the 2020 revision with additional quality control was unavailable.
- **Class definitions.** UA-DETRAC merges cars, buses, vans and other vehicles into one class and does not annotate motorcycles. BDD100K separates the four classes, and we mapped trailers to truck.

## 7. Efficiency

All checkpoints have the same architecture and size: 3.01 M parameters, 8.2 GFLOPs and 5.9 MB.

**Measurement.** Latency was re-measured for all models on the same Tesla T4. The protocol was PyTorch, FP32, batch 1, 640 px, 10 warm-up images and 200 timed test images; latency includes pre-processing, inference and post-processing.

| Model | Mean latency | Throughput |
|---|---|---|
| E1 | 5.27 ms / image | 189.8 FPS |
| E2 | 4.15 ms / image | 241.1 FPS |
| E2c | 4.18 ms / image | 239.1 FPS |

Because the networks are identical, these differences come from the pipeline (image decoding and the number of detections per image) and from timing variation, not from the model.

**These figures are not edge-device measurements.** They used neither TensorRT nor a Jetson device.

## 8. Relation to the original paper

Raza et al. [1] report the following for YOLOv8-nano on their private Karachi dataset (8 classes, 2,182 augmented training images, 100 epochs):
- mAP 89.30% and 74.67 FPS on a Tesla T4 (their Table 12);
- mAP 83.7% / 81.45% and 35 / 33.5 FPS on a Jetson Xavier NX with TensorRT (their Table 15).

The paper does not state the IoU threshold of its mAP or its FPS measurement protocol, and reports no precision, recall or F1.

**Our results therefore do not reproduce, and cannot be directly compared with, the paper's numbers.** The dataset, class set, training length and evaluation protocol all differ. The only directly comparable quantity is model size: our 5.9 MB matches the paper's 6.0 MB for the same architecture.

## 9. Threats to validity

1. **Single model, single seed.** Run-to-run variance is unknown, so small differences (such as E2 vs E2c) are not statistically established.
2. **BDD100K training budget.** BDD100K was trained on 20,000 of 63,000 training images.
3. **Label versions and conventions.** We used BDD100K's 2018 labels. Class mappings were our choice, and class definitions differ between datasets and COCO.
4. **Optimistic UA-DETRAC validation split.** The UA-DETRAC validation split is easier than its test split. Selection was unaffected, but validation scores overstate test performance.
5. **Two datasets excluded.** Two of the four planned datasets could not be evaluated, because they provide no detection annotations.
6. **Speed measured off-device.** Speed was measured on a cloud GPU, not on the target edge hardware.

## 10. Conclusion and future work

- **Strong on UA-DETRAC.** Fine-tuned YOLOv8-nano detects vehicles in fixed-camera surveillance imagery well: mAP@50 0.918 and mAP@50–95 0.670 on the held-out UA-DETRAC test split.
- **Weaker on BDD100K.** It is markedly weaker on dash-cam imagery with many small objects and rare classes: mAP@50 0.498 and mAP@50–95 0.325 on BDD100K.
- **Fine-tuning helps most where the domain is unlike COCO.** It added +0.209 mAP@50 on BDD100K but only +0.039 on UA-DETRAC.
- **Training longer gave little.** Going from 27 to 50 epochs added under 0.01 mAP.

Recommended next steps:
1. Train on the full BDD100K training split.
2. Address class imbalance, for example with class-balanced sampling or more motorcycle and bus data.
3. Evaluate the paper's other detectors under the identical protocol: YOLOv8-small, YOLOv7-tiny and YOLO-NAS-small.
4. Measure speed with TensorRT on the target edge device before any comparison with the paper's real-time claims.

---

[1] M. Raza, M. Kazmi, H. M. Kidwai, H. R. Khan, S. A. Qazi, K. Arshad, and K. Assaleh, "An Edge-Deployed Real-Time Adaptive Traffic Light Control System Using YOLO-Based Vehicle Detection and PCE-Aware Density Estimation," *IEEE Access*, vol. 13, pp. 153586–153613, 2025, doi:10.1109/ACCESS.2025.3602844.
