# CVAT + YOLO26 자동 어노테이션 전체 구축 가이드

## 1. 프로젝트 개요

이 문서는 Windows 환경에서 다음 시스템을 구축하는 전체 과정을 정리한 문서이다.

- Windows 11
- WSL2
- Ubuntu
- Docker Desktop
- NVIDIA RTX 5060 Ti
- CVAT
- Tailscale
- Nuclio Serverless
- YOLOv7
- Ultralytics YOLO26
- Custom YOLO26 모델 학습
- CVAT Automatic Annotation 연동

최종 목표는 다음과 같다.

```text
이미지 데이터
    ↓
CVAT 수동 라벨링
    ↓
YOLO 데이터셋 Export
    ↓
WSL2에서 YOLO26 학습
    ↓
best.pt 생성
    ↓
Nuclio Serverless 배포
    ↓
CVAT Models 등록
    ↓
Automatic Annotation
    ↓
자동 Bounding Box 생성
```

---

# 2. 최종 시스템 구성

전체 구조는 다음과 같다.

```text
Windows 11
│
├── Docker Desktop
│   │
│   ├── CVAT
│   ├── PostgreSQL
│   ├── Redis
│   ├── Traefik
│   ├── Nuclio
│   │
│   ├── YOLOv7 Function
│   │
│   └── YOLO26 Function
│   │
│   └── 기타 CVAT Container
│
├── WSL2
│   └── Ubuntu
│       │
│       ├── nuctl
│       ├── Python
│       ├── PyTorch
│       ├── Ultralytics
│       └── YOLO26 학습 환경
│
├── NVIDIA RTX 5060 Ti
│
└── Tailscale
    │
    └── 외부 팀원 CVAT 접속
```

---

# 3. 구축 환경

실제 구축에 사용한 환경은 다음과 같다.

| 항목 | 환경 |
|---|---|
| Host OS | Windows |
| Linux | WSL2 Ubuntu |
| Container | Docker Desktop |
| GPU | NVIDIA GeForce RTX 5060 Ti 16GB |
| CVAT | Docker 기반 |
| Serverless | Nuclio |
| Nuclio CLI | nuctl 1.16.3 |
| Detection Model | YOLO26 |
| 기존 테스트 모델 | YOLOv7 |
| 원격 접속 | Tailscale |
| Annotation Type | Bounding Box |
| Custom Labels | white_paint, scratch |

---

# 4. 주요 경로

## Windows CVAT Repository

Windows:

```text
C:\Users\82107\cvat
```

WSL에서 접근:

```bash
/mnt/c/Users/82107/cvat
```

CVAT 작업 시:

```bash
cd /mnt/c/Users/82107/cvat
```

---

## YOLO26 학습 Workspace

```bash
/home/gi/yolo26-defect
```

주요 구조:

```text
/home/gi/yolo26-defect/
│
├── .venv/
│
├── dataset/
│
├── dataset_v2/
│   └── dataset/
│       ├── data.yaml
│       ├── images/
│       │   ├── train/
│       │   └── val/
│       └── labels/
│           ├── train/
│           └── val/
│
└── runs/
    ├── defect_v1-2/
    └── defect_v2/
```

---

# 5. Custom YOLO26 모델

최종 학습 모델:

```text
/home/gi/yolo26-defect/runs/defect_v2/weights/best.pt
```

사용 클래스:

```text
0: white_paint
1: scratch
```

학습 데이터:

```text
라벨링 이미지: 107장
Bounding Box: 266개

white_paint: 124개
scratch: 142개
```

Train:

```text
86 images

white_paint: 100
scratch: 116
```

Validation:

```text
21 images

white_paint: 24
scratch: 26
```

---

# 6. YOLO26 최종 Validation 결과

전체 결과:

| Class | Precision | Recall | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|
| all | 0.854 | 0.839 | 0.865 | 0.504 |
| white_paint | 1.000 | 0.947 | 0.992 | 0.684 |
| scratch | 0.709 | 0.731 | 0.737 | 0.323 |

Validation 이미지:

```text
21 images
50 instances
```

---

# 7. CVAT Serverless 모델

최종적으로 CVAT에 다음 두 모델을 사용할 수 있도록 구성하였다.

```text
YOLOv7
YOLO26
```

Nuclio 상태 확인:

```bash
nuctl get functions
```

정상 예시:

```text
NAMESPACE | NAME                    | PROJECT | STATE | REPLICAS | NODE PORT
nuclio    | onnx-wongkinyiu-yolov7 | cvat    | ready | 1/1      | 4842
nuclio    | ultralytics-yolo26      | cvat    | ready | 1/1      | 8201
```

핵심 확인 항목:

```text
STATE = ready
REPLICAS = 1/1
```

---

# 8. Custom YOLO26 Nuclio 위치

Nuclio Function:

```text
serverless/ultralytics/yolo26/nuclio/
```

구조:

```text
nuclio/
├── best.pt
├── function-gpu.yaml
└── main.py
```

Custom 모델 복사:

```bash
cp /home/gi/yolo26-defect/runs/defect_v2/weights/best.pt \
serverless/ultralytics/yolo26/nuclio/best.pt
```

---

# 9. YOLO26 모델 로딩

`main.py`에서는 Nuclio Container 내부 경로를 사용한다.

```python
model = YOLO("/opt/nuclio/best.pt")
```

중요:

```python
model = YOLO("best.pt")
```

처럼 상대 경로를 사용할 경우 실행 위치에 따라 다음 오류가 발생할 수 있다.

```text
FileNotFoundError:
[Errno 2] No such file or directory: 'best.pt'
```

따라서 최종 구성에서는:

```text
/opt/nuclio/best.pt
```

를 사용한다.

---

# 10. Nuclio YOLO26 Labels

`function-gpu.yaml`

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

CVAT Models 화면에서 다음처럼 표시되어야 한다.

```text
YOLO26

Labels:
white_paint
scratch

Provider:
cvat

Type:
detector
```

---

# 11. YOLO26 Nuclio 배포

CVAT Repository 이동:

```bash
cd /mnt/c/Users/82107/cvat
```

배포:

```bash
nuctl deploy \
  --project-name cvat \
  --path serverless/ultralytics/yolo26/nuclio \
  --file serverless/ultralytics/yolo26/nuclio/function-gpu.yaml \
  --platform local
```

성공 예시:

```text
Function deploy complete

functionName:
ultralytics-yolo26

httpPort:
8201
```

상태 확인:

```bash
nuctl get functions
```

정상:

```text
ultralytics-yolo26
ready
1/1
8201
```

---

# 12. Nuclio 오류 확인

Function이 다음처럼 표시된다면:

```text
unhealthy
```

Docker Container 확인:

```bash
docker ps -a --filter "name=ultralytics-yolo26"
```

로그 확인:

```bash
docker logs nuclio-nuclio-ultralytics-yolo26
```

실제 발생했던 오류:

```text
FileNotFoundError:
[Errno 2] No such file or directory: 'best.pt'
```

해결:

```python
model = YOLO("/opt/nuclio/best.pt")
```

---

# 13. CVAT Automatic Annotation

CVAT에서:

```text
Tasks
 ↓
Task 선택
 ↓
Actions
 ↓
Automatic annotation
```

Model:

```text
YOLO26
```

Label Mapping:

```text
YOLO26              CVAT Task

white_paint   →     white_paint
scratch       →     scratch
```

Threshold:

```text
0.25
```

Region of interest:

```text
비워둠
```

전체 이미지를 대상으로 추론한다.

---

# 14. 기존 Annotation 보호

기존 수동 라벨이 존재하는 경우:

```text
Clean previous annotations
```

옵션을:

```text
OFF
```

상태로 유지한다.

이 옵션을 켜면 기존 Annotation이 제거될 수 있으므로 주의한다.

---

# 15. Automatic Annotation 실행

설정:

```text
Model:
YOLO26

white_paint → white_paint
scratch → scratch

Threshold:
0.25

ROI:
전체 이미지

Clean previous annotations:
OFF
```

이후:

```text
Annotate
```

클릭.

정상 완료 시:

```text
Automatic annotation accomplished
```

메시지가 표시된다.

실제 구축 환경에서는 Task #3의 전체 532 Frame에 대해 Automatic Annotation 작업이 정상 완료되었다.

---

# 16. Tailscale CVAT 접속

Windows Tailscale IP:

```text
100.71.200.28
```

CVAT:

```text
http://100.71.200.28:8080
```

CVAT Host 설정:

```powershell
$env:CVAT_HOST="100.71.200.28"
```

CVAT 실행:

```powershell
docker compose `
  -f docker-compose.yml `
  -f components/serverless/docker-compose.serverless.yml `
  up -d
```

주의:

PowerShell의 `$env:CVAT_HOST`는 해당 PowerShell 세션에만 적용될 수 있다.

새 PowerShell에서 CVAT를 다시 실행할 경우 `CVAT_HOST` 값을 다시 확인한다.

---

# 17. 전체 작업 흐름

```text
[1]
Windows / WSL2 설치

        ↓

[2]
Docker Desktop 설치

        ↓

[3]
NVIDIA GPU 확인

        ↓

[4]
CVAT 설치

        ↓

[5]
Tailscale 설정

        ↓

[6]
Nuclio Serverless 설치

        ↓

[7]
YOLOv7 Serverless 테스트

        ↓

[8]
CVAT에서 데이터 라벨링

        ↓

[9]
YOLO Dataset Export

        ↓

[10]
YOLO26 Dataset 생성

        ↓

[11]
YOLO26 Training

        ↓

[12]
best.pt 생성

        ↓

[13]
best.pt → Nuclio Function

        ↓

[14]
YOLO26 Function 배포

        ↓

[15]
nuctl get functions

ready 1/1 확인

        ↓

[16]
CVAT Models에서 YOLO26 확인

        ↓

[17]
Label Mapping

        ↓

[18]
Automatic Annotation

        ↓

[19]
Bounding Box 결과 검수

        ↓

[20]
잘못된 박스 수정 + 추가 학습
```

---

# 18. 문서 구성

상세 내용은 다음 문서로 분리한다.

```text
00_README_전체가이드.md
01_Docker_Desktop_설치.md
02_WSL2_Ubuntu_설정.md
03_CVAT_설치_실행.md
04_Tailscale_외부접속.md
05_Nuclio_Serverless_설정.md
06_NVIDIA_GPU_CUDA_확인.md
07_YOLOv7_CVAT_배포.md
08_YOLO26_학습환경_구축.md
09_YOLO26_데이터셋_구성.md
10_YOLO26_학습_검증.md
11_YOLO26_best_pt_CVAT_배포.md
12_CVAT_자동어노테이션.md
13_트러블슈팅.md
14_명령어_치트시트.md
```

---

# 19. 가장 중요한 확인 명령어

CVAT Container:

```bash
docker ps
```

Nuclio Function:

```bash
nuctl get functions
```

GPU:

```bash
nvidia-smi
```

YOLO26 모델:

```bash
ls -lh /home/gi/yolo26-defect/runs/defect_v2/weights/best.pt
```

Nuclio 모델:

```bash
ls -lh serverless/ultralytics/yolo26/nuclio/best.pt
```

Nuclio 오류:

```bash
docker logs nuclio-nuclio-ultralytics-yolo26
```

---

# 20. 최종 구축 완료 기준

다음 조건을 모두 만족하면 구축 완료이다.

- CVAT 접속 가능
- Tailscale을 통한 팀원 접속 가능
- NVIDIA RTX 5060 Ti 사용 가능
- Nuclio 실행 가능
- YOLOv7 `ready 1/1`
- Custom YOLO26 `ready 1/1`
- CVAT Models에 YOLO26 표시
- `white_paint` Label 표시
- `scratch` Label 표시
- YOLO26 Automatic Annotation 실행 가능
- Bounding Box 자동 생성 가능

현재 구축 환경은 위 단계까지 완료된 상태이다.
