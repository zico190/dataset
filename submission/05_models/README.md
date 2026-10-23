# Trained models

Copy the trained checkpoints here from Google Drive. The exact Drive paths are listed in `../README.md` under "Before submitting".

| File | Experiment | Test mAP@50 / mAP@50–95 |
|---|---|---|
| `E1_UA-DETRAC_best.pt` | E1, UA-DETRAC, 1 class | 0.918 / 0.670 |
| `E2c_BDD100K_best.pt` | E2c, BDD100K, 4 classes, 50 epochs | 0.498 / 0.325 |
| `E2_BDD100K_27ep_best.pt` (optional) | E2, BDD100K, 27 epochs | 0.491 / 0.318 |

**Using a checkpoint** (Ultralytics 8.4.160):

```python
from ultralytics import YOLO
model = YOLO("E1_UA-DETRAC_best.pt")
results = model.predict("image.jpg", imgsz=640, conf=0.25)
```
