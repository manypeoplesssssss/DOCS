# NVIDIA GPU / CUDA 확인

## 1. 목적

YOLO26 학습과 CVAT Nuclio 추론에서 NVIDIA GPU를 사용한다.

현재 환경:

```text
GPU: NVIDIA GeForce RTX 5060 Ti
VRAM: 16GB
Windows + WSL2
```

---

# 2. Windows GPU 확인

PowerShell:

```powershell
nvidia-smi
```

RTX 5060 Ti가 표시되는지 확인한다.

---

# 3. WSL2 GPU 확인

Ubuntu:

```bash
nvidia-smi
```

실제 환경에서 확인된 정보:

```text
NVIDIA GeForce RTX 5060 Ti
VRAM: 16311 MiB
```

WSL에서도 GPU가 나타나야 한다.

---

# 4. PyTorch CUDA 확인

YOLO26 Workspace:

```bash
cd /home/gi/yolo26-defect
```

가상환경 활성화:

```bash
source .venv/bin/activate
```

확인:

```bash
python -c "import torch; print(torch.cuda.is_available())"
```

정상:

```text
True
```

GPU 이름:

```bash
python -c "import torch; print(torch.cuda.get_device_name(0))"
```

정상 예:

```text
NVIDIA GeForce RTX 5060 Ti
```

---

# 5. PyTorch 정보

확인:

```bash
python -c "import torch; print(torch.__version__); print(torch.version.cuda)"
```

현재 구축 과정에서는 CUDA 지원 PyTorch가 정상적으로 GPU를 인식하였다.

---

# 6. Ultralytics GPU 테스트

```bash
yolo checks
```

또는 실제 YOLO 명령에서:

```text
device=0
```

을 사용한다.

예:

```bash
yolo detect predict \
  model=yolo26n.pt \
  source=test.jpg \
  device=0
```

---

# 7. Nuclio GPU 설정

`function-gpu.yaml`:

```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```

YOLO26 Function이 NVIDIA GPU를 사용하도록 설정한다.

---

# 8. GPU 사용량 확인

다른 터미널:

```bash
watch -n 1 nvidia-smi
```

종료:

```text
Ctrl + C
```

---

# 9. 문제 발생 시 확인

순서:

```text
Windows nvidia-smi
        ↓
WSL nvidia-smi
        ↓
PyTorch CUDA True
        ↓
torch GPU 이름 확인
        ↓
Docker Desktop WSL Integration
        ↓
Nuclio GPU 설정
```

---

# 10. 완료 기준

다음이 모두 정상이어야 한다.

```text
Windows에서 RTX 5060 Ti 인식
WSL에서 RTX 5060 Ti 인식
torch.cuda.is_available() = True
YOLO device=0 실행 가능
Nuclio GPU Function 실행 가능
```