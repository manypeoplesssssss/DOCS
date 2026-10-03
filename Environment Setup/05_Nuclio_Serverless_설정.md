# CVAT Nuclio Serverless 설정

## 1. Nuclio란?

CVAT의 Automatic Annotation 모델을 Serverless Function 형태로 실행하기 위해 Nuclio를 사용할 수 있다.

구조:

```text
CVAT
 ↓
Automatic Annotation
 ↓
Nuclio
 ↓
YOLO Function
 ↓
GPU
 ↓
Detection Result
 ↓
CVAT Bounding Box
```

---

# 2. CVAT Serverless 경로

CVAT Repository:

```bash
cd /mnt/c/Users/82107/cvat
```

Serverless 관련 경로:

```text
components/serverless/
serverless/
```

---

# 3. Serverless Compose

CVAT Serverless 환경:

```text
components/serverless/docker-compose.serverless.yml
```

CVAT와 함께 실행:

```bash
docker compose \
  -f docker-compose.yml \
  -f components/serverless/docker-compose.serverless.yml \
  up -d
```

---

# 4. Nuclio CLI

사용 버전:

```text
nuctl 1.16.3
```

확인:

```bash
nuctl version
```

---

# 5. Nuclio Project 확인

```bash
nuctl get projects
```

정상 예:

```text
NAMESPACE | NAME
nuclio    | cvat
```

CVAT용 Project:

```text
cvat
```

---

# 6. Function 확인

```bash
nuctl get functions
```

최종 환경:

```text
NAMESPACE | NAME                    | PROJECT | STATE | REPLICAS | NODE PORT
nuclio    | onnx-wongkinyiu-yolov7 | cvat    | ready | 1/1      | 4842
nuclio    | ultralytics-yolo26      | cvat    | ready | 1/1      | 8201
```

---

# 7. 상태 의미

정상:

```text
ready
1/1
```

문제:

```text
unhealthy
```

`unhealthy`이면 CVAT에서 Automatic Annotation을 실행하기 전에 원인을 확인한다.

---

# 8. Nuclio Container 확인

YOLO26:

```bash
docker ps -a --filter "name=ultralytics-yolo26"
```

정상 실행 중이면 Status가 `Up` 상태여야 한다.

종료된 경우 예:

```text
Exited (2)
```

---

# 9. 로그 확인

```bash
docker logs nuclio-nuclio-ultralytics-yolo26
```

본 프로젝트에서 실제 발생한 오류:

```text
Caught unhandled exception while initializing
```

그리고:

```text
FileNotFoundError:
[Errno 2] No such file or directory: 'best.pt'
```

---

# 10. YOLO26 Function 구조

```text
serverless/
└── ultralytics/
    └── yolo26/
        └── nuclio/
            ├── best.pt
            ├── function-gpu.yaml
            └── main.py
```

---

# 11. Custom Model

학습된 모델:

```text
/home/gi/yolo26-defect/runs/defect_v2/weights/best.pt
```

Nuclio Function으로 복사:

```bash
cp /home/gi/yolo26-defect/runs/defect_v2/weights/best.pt \
serverless/ultralytics/yolo26/nuclio/best.pt
```

확인:

```bash
ls -lh serverless/ultralytics/yolo26/nuclio/
```

예:

```text
best.pt             5.2M
function-gpu.yaml
main.py
```

---

# 12. main.py Model Path

잘못된 예:

```python
model = YOLO("best.pt")
```

실행 시:

```text
FileNotFoundError
```

가 발생하였다.

최종:

```python
model = YOLO("/opt/nuclio/best.pt")
```

Nuclio는 Handler 내용을 Container의 `/opt/nuclio`에 배치하기 때문에 최종적으로 이 경로를 사용하였다.

---

# 13. Function Labels

`function-gpu.yaml`:

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

# 14. Runtime

```yaml
runtime: python:3.12
handler: main:handler
```

초기 구성 과정에서 Python Runtime과 Base Image의 Python 버전이 맞지 않는 문제가 있었기 때문에 최종적으로:

```text
python:3.12
```

를 사용하였다.

---

# 15. GPU

```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```

이를 통해 YOLO26 Function이 NVIDIA GPU를 사용할 수 있도록 설정한다.

---

# 16. HTTP Trigger

```yaml
triggers:
  myHttpTrigger:
    maxWorkers: 1
    kind: http
    workerAvailabilityTimeoutMilliseconds: 10000
    attributes:
      maxRequestBodySize: 33554432
```

현재 Nuclio에서는 `maxWorkers`에 대한 Deprecated Warning이 발생할 수 있다.

예:

```text
MaxWorkers is deprecated
```

현재 구축에서는 Warning이 발생했지만 Function 배포 및 실행은 정상적으로 완료되었다.

향후 Nuclio 버전 변경 시 `numWorkers` 설정 방식 확인이 필요하다.

---

# 17. YOLO26 배포

```bash
nuctl deploy \
  --project-name cvat \
  --path serverless/ultralytics/yolo26/nuclio \
  --file serverless/ultralytics/yolo26/nuclio/function-gpu.yaml \
  --platform local
```

정상 완료:

```text
Function deploy complete
```

예:

```text
functionName:
ultralytics-yolo26

httpPort:
8201
```

---

# 18. 배포 후 확인

```bash
nuctl get functions
```

정상:

```text
ultralytics-yolo26
cvat
ready
1/1
8201
```

---

# 19. best.pt COPY 오류

구축 과정에서 다음 설정도 시도하였다.

```yaml
directives:
  postCopy:
    - kind: COPY
      value: best.pt /opt/nuclio/best.pt
```

하지만 Nuclio의 Docker Build Context에서 `best.pt`가 해당 root 경로에 존재하지 않아:

```text
COPY best.pt /opt/nuclio/best.pt
```

에서:

```text
"/best.pt": not found
```

오류가 발생하였다.

따라서 이 `postCopy COPY` 방식은 제거하였다.

Nuclio가 Function Handler Directory를:

```dockerfile
COPY handler /opt/nuclio
```

형태로 복사하도록 두고 `best.pt` 자체를 Function Directory 안에 위치시켰다.

그리고 Python에서는:

```python
YOLO("/opt/nuclio/best.pt")
```

로 로딩하였다.

---

# 20. 최종 정상 상태

```bash
nuctl get functions
```

결과:

```text
onnx-wongkinyiu-yolov7
ready
1/1

ultralytics-yolo26
ready
1/1
```

이 상태가 되면 CVAT:

```text
Models
```

에서 YOLO26이 표시된다.

Labels:

```text
white_paint
scratch
```

Type:

```text
detector
```

이면 Serverless 설정이 완료된 것이다.