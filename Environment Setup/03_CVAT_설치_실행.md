# CVAT 설치 및 실행

## 1. CVAT 역할

CVAT는 본 프로젝트의 이미지 Annotation 플랫폼이다.

사용 목적:

```text
이미지 업로드
Bounding Box 라벨링
Dataset Export
YOLO 학습 데이터 생성
YOLO26 Automatic Annotation
팀원 협업
```

---

# 2. 프로젝트 경로

Windows:

```text
C:\Users\82107\cvat
```

WSL:

```bash
/mnt/c/Users/82107/cvat
```

이동:

```bash
cd /mnt/c/Users/82107/cvat
```

---

# 3. CVAT 실행

기본적으로 Docker Compose를 사용한다.

```bash
docker compose up -d
```

Serverless 환경까지 사용하는 경우:

```bash
docker compose \
  -f docker-compose.yml \
  -f components/serverless/docker-compose.serverless.yml \
  up -d
```

---

# 4. Windows PowerShell 실행

본 프로젝트에서는 Tailscale IP를 CVAT Host로 사용하였다.

```powershell
$env:CVAT_HOST="100.71.200.28"
```

실행:

```powershell
docker compose `
  -f docker-compose.yml `
  -f components/serverless/docker-compose.serverless.yml `
  up -d
```

---

# 5. CVAT 접속

현재 Tailscale 기반 주소:

```text
http://100.71.200.28:8080
```

Local 환경에서는 구성에 따라:

```text
http://localhost:8080
```

또는:

```text
http://127.0.0.1:8080
```

을 사용할 수 있다.

---

# 6. CVAT 상태 확인

```bash
docker ps
```

CVAT 관련 Container 검색:

```bash
docker ps --format "table {{.Names}}\t{{.Status}}" | grep cvat
```

---

# 7. API 확인

Windows PowerShell:

```powershell
curl.exe -I http://100.71.200.28:8080/api/server/about
```

정상이라면 HTTP 응답이 반환된다.

예:

```text
HTTP/1.1 200 OK
```

---

# 8. CVAT 재시작

```bash
docker compose \
  -f docker-compose.yml \
  -f components/serverless/docker-compose.serverless.yml \
  restart
```

필요한 경우 다시 생성:

```bash
docker compose \
  -f docker-compose.yml \
  -f components/serverless/docker-compose.serverless.yml \
  up -d
```

---

# 9. CVAT 종료

```bash
docker compose \
  -f docker-compose.yml \
  -f components/serverless/docker-compose.serverless.yml \
  down
```

주의:

```bash
docker compose down -v
```

는 Volume까지 제거할 수 있으므로 데이터 보존이 필요한 환경에서는 함부로 실행하지 않는다.

---

# 10. Task 생성

CVAT:

```text
Tasks
→ Create new task
```

Task 이름을 지정하고 이미지를 업로드한다.

본 테스트 Task:

```text
car
```

Frame 수:

```text
532
```

---

# 11. Label 생성

사용 Label:

```text
white_paint
scratch
```

두 Label 모두 Bounding Box Detection 용도로 사용한다.

---

# 12. Annotation

Task:

```text
Task
→ Job
→ Annotation
```

이미지에서 결함 영역을 Bounding Box로 지정한다.

중요:

이미지에 보이는 결함은 가능한 한 모두 라벨링한다.

예를 들어 `scratch`만 라벨링하고 실제로 존재하는 `white_paint`를 누락하면 모델은 해당 `white_paint`를 정상 배경으로 학습할 가능성이 있다.

---

# 13. Dataset Export

Annotation이 완료되면 Dataset을 Export한다.

YOLO 학습용 형식을 사용한다.

예:

```text
Ultralytics YOLO Detection
```

Export 파일을 WSL 학습 환경으로 옮긴다.

---

# 14. Models

Serverless 모델이 정상 배포되면 상단:

```text
Models
```

에서 확인할 수 있다.

최종 환경:

```text
YOLOv7
YOLO26
```

YOLO26:

```text
Provider: cvat
Type: detector

Labels:
white_paint
scratch
```

---

# 15. Automatic Annotation

Task:

```text
Actions
→ Automatic annotation
```

Model:

```text
YOLO26
```

Mapping:

```text
white_paint → white_paint
scratch → scratch
```

Threshold:

```text
0.25
```

기존 Annotation을 유지해야 한다면:

```text
Clean previous annotations = OFF
```

실행:

```text
Annotate
```

성공:

```text
Automatic annotation accomplished
```

---

# 16. 정상 구축 기준

다음 기능이 모두 작동하면 된다.

```text
CVAT 로그인
Task 생성
Image Upload
Bounding Box Annotation
Dataset Export
Models 화면
YOLO26 표시
Automatic Annotation 실행
```