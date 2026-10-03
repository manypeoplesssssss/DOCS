# Custom YOLO26 best.pt CVAT 배포

## 1. 목적

직접 학습한:

```text
white_paint
scratch
```

Detection 모델을 CVAT Automatic Annotation에서 사용한다.

---

# 2. 학습 모델

```text
/home/gi/yolo26-defect/runs/defect_v2/weights/best.pt
```

---

# 3. CVAT 이동

가상환경 종료:

```bash
deactivate
```

CVAT:

```bash
cd /mnt/c/Users/82107/cvat
```

---

# 4. Function Directory

```text
serverless/ultralytics/yolo26/nuclio/
```

최종 구조:

```text
nuclio/
├── best.pt
├── function-gpu.yaml
└── main.py
```

---

# 5. best.pt 복사

```bash
cp /home/gi/yolo26-defect/runs/defect_v2/weights/best.pt \
serverless/ultralytics/yolo26/nuclio/best.pt
```

확인:

```bash
ls -lh serverless/ultralytics/yolo26/nuclio/
```

실제:

```text
best.pt             5.2M
function-gpu.yaml
main.py
```

---

# 6. main.py

핵심:

```python
from ultralytics import YOLO

model = YOLO("/opt/nuclio/best.pt")
```

`/opt/nuclio` 경로를 사용하는 것이 중요하다.

---

# 7. function-gpu.yaml Labels

```yaml
metadata:
  name: ultralytics-yolo26
  namespace: cvat

  annotations:
    name: YOLO26
    type: detector
    framework: pytorch
    spec: |
      [
        { "id": 0, "name": "white_paint", "type": "rectangle" },
        { "id": 1, "name": "scratch", "type": "rectangle" }
      ]
```

---

# 8. Runtime

```yaml
spec:
  description: YOLO26 detector for CVAT
  runtime: python:3.12
  handler: main:handler
```

---

# 9. Base Image

```yaml
build:
  baseImage: ultralytics/ultralytics@sha256:3e1b06201f10c91078b7c185035984bd4d6b48a79c5863c79a56a12b0eebb498
```

---

# 10. GPU

```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```

---

# 11. Trigger

```yaml
triggers:
  myHttpTrigger:
    maxWorkers: 1
    kind: http
    workerAvailabilityTimeoutMilliseconds: 10000
    attributes:
      maxRequestBodySize: 33554432
```

---

# 12. 배포

```bash
nuctl deploy \
  --project-name cvat \
  --path serverless/ultralytics/yolo26/nuclio \
  --file serverless/ultralytics/yolo26/nuclio/function-gpu.yaml \
  --platform local
```

성공:

```text
Function deploy complete
```

Port:

```text
8201
```

---

# 13. 확인

```bash
nuctl get functions
```

최종:

```text
onnx-wongkinyiu-yolov7   cvat   ready   1/1   4842
ultralytics-yolo26       cvat   ready   1/1   8201
```

---

# 14. CVAT Models

CVAT:

```text
Models
```

YOLO26:

```text
Labels

white_paint
scratch
```

Provider:

```text
cvat
```

Type:

```text
detector
```

여기까지 나오면 Custom Model 등록 성공이다.