# SmartFlow — Paper Methodology Extraction & Dataset Audit

Audit date: 2026-10-21. No training code, and no experimental results, are included here.

Labels used throughout:

- **[PAPER]** — stated explicitly in the paper (section/table cited).
- **[VERIFIED]** — checked directly against the official page, docs, or the files themselves during this audit.
- **[UNVERIFIED]** — could not be confirmed from an official source here; confirm before relying on it.
- **[REC]** — our recommendation, not from the paper.

---

## 1. Source paper

M. Raza, M. Kazmi, H. M. Kidwai, H. R. Khan, S. A. Qazi, K. Arshad, K. Assaleh,
"An Edge-Deployed Real-Time Adaptive Traffic Light Control System Using YOLO-Based Vehicle Detection and PCE-Aware Density Estimation,"
*IEEE Access*, vol. 13, pp. 153586–153613, 2025. DOI: [10.1109/ACCESS.2025.3602844](https://doi.org/10.1109/ACCESS.2025.3602844) (gold open access, CC BY).

The paper was not in this repository, so the full text was read from the open-access IEEE PDF.

### 1.1 Verified methodology [PAPER]

| Item | What the paper states | Where |
|---|---|---|
| Training data | **Private custom dataset**, Allama Shabbir Usmani Road, Karachi. Videos at 25 FPS, frames 1280×738. **None of the four selected datasets was used for training.** | §III-A.1–2 |
| Frame sampling | "Adaptive frame extraction … at a 25% recording ratio, capturing one image every four seconds" | §III-A.2 |
| Size | 1,373 source images, 8 classes (Table 4). After augmentation: 2,726 images, split 2,182 train / 272 val / 272 test | Table 4, §III-A.5 |
| Split | "Strict 80:10:10 non-overlapping split". Augmentation was applied **only to the training set**; val and test were kept clean | §III-A.2 |
| Classes (8) | Auto Rickshaw; Car-Jeeps-Vans-Taxi; Cart-Chingchi; Large Buses; Mini Buses; Motorcycles-Scooters; Pickups; Trucks and Trailers | Table 4 |
| Annotation | Written class guidelines; occluded vehicles labelled on visible parts only; **polygon** annotation for heavy overlap; dual annotation with senior arbitration; inter-annotator agreement checks; QC team. The conversion from polygons to boxes is **not described** | §III-A.3 |
| Basic augmentation | Random crop/resize, 0–20% zoom; rotation up to ±15°; sepia and bluish tints; brightness/exposure ±25%; cut-out of three 33% boxes; blur with 1.5 px radius; noise up to 5% of pixels; mosaic. Augmentations were applied **in combinations** | §III-A.5 |
| Context augmentation | Weather (rain streaks, snow, sand/dust sepia, fog haze), time of day (brightness/contrast, histogram equalisation, desaturation/blue shift, synthetic headlight glare), traffic behaviour (cut-out, shear, motion blur), camera perspective (affine, tilt, in-plane rotation) | Table 6, Fig. 3 |
| Models | YOLOv7-tiny, YOLOv8-nano, YOLOv8-small, YOLO-NAS-small | §III-B.1 |
| Architecture detail | Generic backbone/neck/head only (Fig. 5). Says v7-tiny uses Leaky-ReLU, and v8 is anchor-free. **No architectural modifications** were made to the stock models | §III-B.1 |
| Hyperparameter search | Grid search: LR {0.001, 0.01, 0.1}; batch {16, 32, 64}; epochs {50, 100, 150}; optimizer {SGD, Adam, AdamW}. Selection by mAP | §III-B.1 |
| Selected values (Table 7) | YOLOv7-tiny: lr 0.01, bs 32, 100 ep, SGD · YOLOv8n: lr 0.001, bs 16, 100 ep, AdamW · YOLOv8s: lr 0.01, bs 32, 100 ep, AdamW · YOLO-NAS-s: lr 0.005, bs 32, 50 ep, AdamW (note: 0.005 is outside the stated grid) | Table 7 |
| Training hardware | NVIDIA Tesla T4, 16 GB | §IV |
| Metrics | mAP = mean of per-class AP (Eq. 4); FPS = frames / processing time (Eq. 3); model size in MB; Wilcoxon signed-rank test between v8n and v7-tiny (p = 0.002) | §IV-A, Eq. 3–5 |
| Colour-scale study | RGB, grayscale, and "green" scale (night vision), evaluated on Allama Shabbir Usmani video | §IV-A.1, Table 12 |
| Generalisation | KITTI and PASCAL VOC; an unseen second intersection (Teen Talwar); DAWN for weather (qualitative, Fig. 14); time of day (qualitative, Fig. 15) | §IV-A.2, Tables 13–14 |
| Edge deployment | TensorRT on Jetson Nano and Jetson Xavier NX; control on Arduino Portenta H7 over Wi-Fi | §IV-B, Tables 10–11, 15 |
| Density | Traffic Density = Σ count_i × PCE_i. PCE values from ref. [13] (Karachi): Car/Jeep/Van/Taxi 1.00; Motorcycle/Scooter 0.25; Autorickshaw 0.50; Pickup 1.50; Mini bus 2.00; Large bus 2.50; Truck & trailer 3.00 | Eq. 1, Table 8 |

### 1.2 Details the paper does NOT report

These need to be chosen and documented as **[REC]** in your thesis:

- Input image size (e.g. 640), pretrained initialisation (COCO or scratch), LR schedule, warm-up, weight decay, momentum, and random seeds.
- **IoU threshold for mAP** (0.5 or 0.5:0.95). Also the confidence and NMS thresholds.
- The FPS protocol: whether pre/post-processing is included, the batch size, and the TensorRT precision (FP16 or INT8).
- How KITTI and VOC were evaluated with an 8-class South-Asian label set, which those datasets do not share.
- The number of runs behind the Wilcoxon test.
- The annotation tool. Roboflow appears in the reference list [52], but the text does not say it was used for annotation.

### 1.3 Internal inconsistencies to be aware of [PAPER]

1. **The headline "90% mAP at 74 FPS" does not match the edge table.** The 74.67 FPS figure comes from Table 12 (YOLOv8n, RGB), which was run on the Tesla T4 workstation. On Jetson Xavier, Table 15 reports YOLOv8n at **35 FPS / 83.7% mAP** (Allama Shabbir Usmani) and **33.5 FPS / 81.45%** (Teen Talwar).
2. **The dataset counts do not reconcile.** Table 4 gives 1,373 source images and Table 5 gives 2,400. A 10% clean val/test split of 1,373 is about 137 images, yet the paper reports 272 + 272 after augmentation.
3. **Table 4 lists 8 classes but Table 8 gives PCE values for only 7.** Cart-Chingchi has no PCE value.
4. Tables 13 and 14 are duplicates. They also cite PASCAL VOC differently ([89] and [84]).
5. Table 5 lists BDD100K as "120,000,000 images, 70/30 split". The official figures are 100K images with a 70K/10K/20K split.

**Implication:** we cannot replicate the paper's numbers, because its dataset is private. Our experiments are a **re-implementation of its detection protocol on public data**. They are not a reproduction, and they must not be compared to 90% / 74 FPS as like-for-like.

---

## 2. Dataset audit

### 2.1 Summary table

| | **BDD100K** | **UA-DETRAC** | **QMUL Junction** | **AI City Challenge 2021** |
|---|---|---|---|---|
| Media | Images (det set: frame at the 10th second of each video) + videos [VERIFIED] | Frame sequences from 100 videos (≈140K frames), 960×540 [VERIFIED for the mirror] | **One video** `qmul_junction.avi`, 90,000 frames, 360×288, 25 Hz [VERIFIED] | Videos (T1, T3, T4); images (T2 ReID crops) [VERIFIED page] |
| Viewpoint | **Dash-cam (ego vehicle)** | Fixed roadside/overpass cameras (Beijing, Tianjin) | Fixed overhead, one junction | Fixed traffic cameras, US intersections and highways |
| Vehicle bboxes | **Yes** | **Yes** | **No** | T3 (CityFlowV2): **yes, but only vehicles seen by ≥2 cameras** [UNVERIFIED for 2021; stated for CityFlow in the AIC 2020 summary]. T1: **no** (counts). T4: **no** (timestamps). T2: crops only |
| Class labels | 10 evaluated: pedestrian, rider, car, truck, bus, train, motorcycle, bicycle, traffic light, traffic sign [VERIFIED docs] | Official: car, bus, van, others [per paper/literature; UNVERIFIED in files]. **HF mirror: only `vehicle` + `ignored`** [VERIFIED] | Clip-level event classes only | T3: vehicle identity, no type class. T1: car vs. truck *counts* |
| Splits | 70K train / 10K val / 20K test. **Only train and val labels in `det_20`**; test labels withheld [VERIFIED] | 60 train / 40 test sequences. Mirror: 83,791 train frames, 56,340 test frames, **test boxes present** [VERIFIED] | 30 random partitions over 1,800 clip feature vectors [VERIFIED ReadMe] | T3: 3 train / 2 val / 1 test scenario. T1: Set A (public, GT only for a small demo subset) / Set B (held out) |
| Annotation format | Scalabel JSON (`det_train.json`, `det_val.json`), box2d x1,y1,x2,y2 [VERIFIED] | Official: one XML per sequence + ignored regions. Mirror: per-sequence `annotations.csv` (filename, w, h, class, xmin, ymin, xmax, ymax, confidence, target_id) + per-frame Pascal-VOC XML [VERIFIED] | `.mat` files (`mileEnd6class.mat`, `mit8class.mat`): feature matrix x, label vector y | T3: MOT-style `gt.txt` [UNVERIFIED]. T1: count files. T4: `train-anomaly-results.txt` (video id, start, end) |
| Access | Direct download from `dl.cv.ethz.ch/bdd100k/data/` per official docs. No login stated; you agree to the licence by downloading [VERIFIED docs] | Official: Google Drive links on the author page ([sites.google.com/view/daweidu/projects/ua-detrac](https://sites.google.com/view/daweidu/projects/ua-detrac)). The old `detrac-db.rit.albany.edu` now redirects to an Albany lab page. HF mirror is ungated [VERIFIED] | Direct links on the CUHK page [VERIFIED] | **Request form required** (AI City Challenge Datasets Request Form on the "Data Access Instructions" page). Past-year data is requested through the same form [VERIFIED page] |
| License | UC Regents / BAIR licence: free for **educational, research, not-for-profit** use. Commercial use needs an OTL licence [VERIFIED] | Author page: **no licence text**, only a citation request. The mirror self-declares CC-BY-4.0, but it is a **third-party repackaging** and cannot relicense. Treat as research-only until confirmed [REC] | "Research use only, no commercial use" [VERIFIED page] | Not stated on the public page. Terms come with the request form, so read them before use. The page says faces and plates are redacted |
| **YOLO detection training without new annotation?** | **YES** | **YES** (mask ignored regions) | **NO** | **Partial / NO** (see §2.3) |

### 2.2 Per-dataset notes

**BDD100K: suitable as the multi-class detection source.**

- Conversion needed: Scalabel JSON → YOLO txt (normalised cx, cy, w, h). `category_id` starts at 1.
- Downloads needed: "100K Images" (5.3 GB, md5 `5a0359c86a0b8713adab1eee9a3041cb`) and "Detection 2020 Labels" (53 MB, md5 `b86a3e1b7edbcad421b7dad2b3987c94`). Verify both md5 sums.
- Because test labels are not public, use **official val as our test set** and carve our own val out of train.
- Domain gap: dash-cam, ego-motion, and a mostly frontal/rear view. SmartFlow uses an elevated roadside view, so the gap is large.
- The `rider` class is separate from `motorcycle`/`bicycle`. Decide whether to merge rider boxes into the two-wheeler class or ignore them, and document the choice.
- `det_20` may contain extra, non-evaluated categories (e.g. "other vehicle", "trailer"). Count them after download; do not assume.
- This environment's network proxy blocked `dl.cv.ethz.ch`, so the archive could not be inspected here. Do it locally.

**UA-DETRAC: the closest match to the SmartFlow viewpoint (fixed elevated camera).**

- The HF mirror (`blues111/ua_detrac`, duplicated from `abhineet123/ua_detrac`, built for the P2S-Video project) was inspected via remote ZIP listing:
  - `ua_detrac_training_set.zip`: 60 sequence folders, 83,791 JPGs, `annotations.csv` + `annotations.zip` per sequence.
  - `ua_detrac_test_set.zip`: 40 folders, 56,340 JPGs, with ground-truth boxes.
  - Every box has `class` ∈ {`vehicle`, `ignored`}. **The car/bus/van/others type is lost in the mirror.**
- Use the mirror for 1-class experiments. For multi-class experiments, use the **official** DETRAC-Annos XML from the author's Google Drive. Confirm `vehicle_type` values and the licence there before use.
- `ignored` entries are *ignore regions*, not vehicles. YOLO has no ignore label, so **black out (fill) those regions** in the image, or drop them. Never train them as a class.
- At 25 FPS, consecutive frames are near-duplicates. **Split by sequence, never by frame**, and subsample (e.g. every 5th–10th frame) to avoid leakage and inflated mAP.
- Coverage gap: there are no motorcycles, rickshaws or two-wheelers. A missing class here means **unlabelled**, not "absent".

**QMUL Junction: unsuitable for detection training.**

- It is a single low-resolution (360×288) video with **no bounding boxes**.
- The downloadable "ground truth" (`accv_2010_dataset.zip`) contains clip-level event labels with pre-computed features (`mileEnd6class.mat`, `mit8class.mat`) for unusual-event detection (Loy et al., ACCV 2010). It also does not map cleanly to `qmul_junction.avi`. **These are not detection labels.**
- Alternative uses:
  - (a) Qualitative demo or **inference-only** stress test of the trained detector on a low-resolution junction view. Report no mAP.
  - (b) Downstream **density/phase logic** simulation input.
  - (c) If detection metrics are wanted on it, a manually annotated subset would be required. That is new annotation work and must be stated as such.

**AI City Challenge 2021: not a drop-in detection dataset.**

- **Track 1 (counting)**: closest to SmartFlow's purpose (intersection movement counts, car/truck) and evaluated on Jetson NX. But it provides **counts per movement, not boxes**. Use it for **end-to-end count/density validation** (detector + tracker + ROI/movement logic), not for detector training.
- **Track 3 (CityFlowV2 MTMC)**: has boxes (313,931) at 16 intersections, but they are **class-agnostic**. Per the CityFlow benchmark description, they also cover only **vehicles that appear in ≥2 cameras**, so single-camera vehicles are unlabelled. Training on it would teach the detector to suppress real vehicles, and evaluating on it would count correct detections as false positives. It can only be used as **detection-recall-only / qualitative** evaluation, or for tracking. Confirm the exact annotation policy in the 2021 data package.
- **Track 4 (anomaly)**: anomaly start/end timestamps only, **no boxes**. Unsuitable.
- **Track 2 (ReID)**: vehicle crops with identity labels. Unsuitable for detection.
- Access goes through the official request form, and use is subject to the terms supplied with it. External-data rules apply only to leaderboard entries.

### 2.3 Which datasets genuinely support YOLO detection training without new annotation

| Dataset | Verdict |
|---|---|
| BDD100K (det_20) | ✅ Multi-class |
| UA-DETRAC (official XML) | ✅ Multi-class (car/bus/van/others) |
| UA-DETRAC (HF mirror) | ✅ **1-class only** |
| AI City 2021 T3 | ⚠️ Boxes exist but are incomplete and class-agnostic. Not for training; recall-only evaluation |
| AI City 2021 T1, T2, T4 | ❌ Counts, ReID crops, and anomaly timestamps are **not** detection labels |
| QMUL Junction | ❌ No boxes |

---

## 3. Class mapping [REC]

Only vehicle types that actually exist in the trainable datasets are included. The paper's South-Asian classes (Auto Rickshaw, Cart-Chingchi, Mini/Large bus split, Pickup) **exist in none of the four datasets**.

**Primary scheme, 4 classes:**

| SmartFlow class | BDD100K | UA-DETRAC (official) | Paper class(es) | PCE (Table 8) |
|---|---|---|---|---|
| `car` | car | car, van | Car-Jeeps-Vans-Taxi | 1.00 |
| `bus` | bus | bus | Mini Buses (2.00) / Large Buses (2.50) | choose 2.00–2.50 and document it, e.g. 2.25 or 2.50 |
| `truck` | truck | others (*see note*) | Trucks & Trailers (3.00); Pickups (1.50) | 3.00 |
| `motorcycle` | motorcycle | — (not annotated) | Motorcycles-Scooters | 0.25 |

**Excluded / handled separately:**

- BDD `pedestrian`, `traffic light`, `traffic sign`, `train`, `bicycle`: dropped.
- BDD `rider`: drop, or merge into `motorcycle`. Pick one and keep it fixed.
- DETRAC `ignored`: masked out.

**Notes:**

- DETRAC `others` is a mixed bag (trucks, tankers, etc.), and BDD `truck` may include pickups. Check samples of both before finalising. If either is too heterogeneous, drop it to `ignore` instead of polluting `truck`.
- `motorcycle` is unlabelled in DETRAC. Train it from BDD only, and evaluate it only on BDD. Never compute motorcycle AP on DETRAC.

**Secondary scheme, 1 class `vehicle`** (all of the above merged). Use it for the HF mirror and for cross-dataset and CityFlow comparisons.

**For the South-Asian classes**, a different source is required:

- (a) Annotate your own local footage, as the paper did, or
- (b) Use the **India Driving Dataset (IDD)**, which the paper itself cites [57]. It includes autorickshaw. It requires registration on its official site, and its licence must be checked before use. Its availability and current terms were not verified in this audit.

---

## 4. Recommended experimental plan [REC]

**E0 — Data preparation (no training)**

- Download only through the official channels in §2.1, and record the licence for each dataset.
- Convert BDD100K and DETRAC to YOLO txt using the §3 mapping. Mask DETRAC ignore regions.
- Splits:
  - DETRAC: by sequence, with a stratified val chosen by scene and weather.
  - BDD100K: our val is a subset of official train; our test is official val.
- Fix the seeds, and log class histograms per split.

**E1 — Model comparison (mirrors the paper's Table 12 protocol)**

- Models: YOLOv8n, YOLOv8s, YOLOv7-tiny, optionally YOLO-NAS-s. Optionally include a current Ultralytics model as an extra baseline.
- Use the paper's Table 7 hyperparameters as the starting point, e.g. YOLOv8n: AdamW, lr 0.001, bs 16, 100 epochs.
- State the unreported settings explicitly: imgsz 640, COCO-pretrained weights, and the default schedule.
- Train on DETRAC (4-class) and on BDD100K (4-class), separately and combined.

**E2 — Augmentation ablation (mirrors Table 13)**

- (i) Default YOLO augmentation, vs (ii) default plus the paper's context augmentations: rain, fog, dust, night/glare, cut-out, blur, ±15° rotation.
- Apply them to the **train split only**.

**E3 — Colour-scale study (mirrors Table 12)**

- Evaluate on RGB, grayscale, and green-channel versions of the test set.

**E4 — Cross-domain generalisation**

- Train on DETRAC and test on BDD, and the reverse, using the shared classes.
- Run 1-class recall-only checks on CityFlowV2 (AIC T3) if access is granted.

**E5 — Edge benchmark**

- Export with TensorRT FP16 (and INT8 if you have a calibration set) on the target Jetson.
- Report FPS with batch 1, both including and excluding pre/post-processing, alongside mAP measured on the device.
- Do this to avoid the T4-vs-Jetson ambiguity in §1.3.

**E6 — Downstream density and counting**

- Apply Eq. 1 with Table 8 PCEs using the §3 mapping.
- Validate movement counts against AIC 2021 Track 1 demo GT, where access permits.
- Use QMUL Junction **only** for qualitative or inference-only runs.

**Metrics to report for every run**

- P, R, mAP@0.5, mAP@0.5:0.95, per-class AP, and a confusion matrix.
- Params, model size (MB), and FPS (with the protocol stated).
- Mean ± std over ≥3 seeds. Use a Wilcoxon or paired test only if you have enough runs.

**Do not**

- Claim comparability with the paper's 90% mAP / 74 FPS.
- Evaluate on any frames from training sequences.
- Report AP for classes that are unlabelled in a given test set.
