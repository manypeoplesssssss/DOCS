# Docker Desktop 설치 및 설정

## 1. 목적

CVAT는 여러 서비스를 Docker Container로 실행한다.

주요 Container:

```text
CVAT Server
CVAT UI
PostgreSQL
Redis
Traefik
Nuclio
YOLO Serverless Functions
```

Windows에서는 Docker Desktop + WSL2 조합을 사용한다.

---

# 2. Docker Desktop 설치

Docker Desktop을 Windows에 설치한다.

설치 과정에서 WSL2 기반 Engine을 사용한다.

설치 후 Docker Desktop을 실행한다.

---

# 3. Docker 정상 작동 확인

PowerShell:

```powershell
docker --version
```

예시:

```text
Docker version ...
```

Compose 확인:

```powershell
docker compose version
```

---

# 4. Docker Desktop WSL2 설정

Docker Desktop에서:

```text
Settings
→ General
→ Use the WSL 2 based engine
```

활성화한다.

그리고:

```text
Settings
→ Resources
→ WSL Integration
```

Ubuntu를 활성화한다.

예:

```text
Ubuntu       ON
Ubuntu-22.04 ON
```

실제 설치된 Ubuntu 배포판만 활성화하면 된다.

---

# 5. WSL에서 Docker 확인

Ubuntu 실행:

```bash
docker --version
```

그리고:

```bash
docker ps
```

정상이라면 Docker daemon 연결 오류가 발생하지 않는다.

---

# 6. GPU Container 확인

NVIDIA GPU가 Docker에서 사용 가능한지 확인한다.

먼저:

```bash
nvidia-smi
```

GPU가 표시되어야 한다.

본 프로젝트 GPU:

```text
NVIDIA GeForce RTX 5060 Ti
VRAM 16GB
```

---

# 7. Docker Container 확인

실행 중인 Container:

```bash
docker ps
```

종료된 Container까지:

```bash
docker ps -a
```

특정 Container 검색:

```bash
docker ps -a --filter "name=cvat"
```

Nuclio YOLO26:

```bash
docker ps -a --filter "name=ultralytics-yolo26"
```

---

# 8. Docker 로그 확인

기본 형식:

```bash
docker logs <container-name>
```

예:

```bash
docker logs nuclio-nuclio-ultralytics-yolo26
```

실시간 로그:

```bash
docker logs -f <container-name>
```

---

# 9. Docker Desktop 재시작

CVAT 또는 Traefik Routing이 이상할 경우 Docker Desktop 자체를 재시작하면 해결되는 경우가 있다.

순서:

```text
Docker Desktop 종료
→ 다시 실행
→ Containers 준비될 때까지 대기
→ CVAT 확인
```

---

# 10. 주의사항

다음 명령은 함부로 실행하지 않는다.

```bash
docker compose down -v
```

`-v` 옵션은 Docker Volume을 삭제할 수 있다.

CVAT 데이터베이스나 관련 데이터가 Volume에 저장되어 있다면 데이터 손실 가능성이 있다.

단순 종료가 목적이라면:

```bash
docker compose down
```

을 사용한다.

---

# 11. 자주 사용하는 명령어

```bash
docker ps
```

```bash
docker ps -a
```

```bash
docker images
```

```bash
docker logs <container>
```

```bash
docker stats
```

```bash
docker system df
```

---

# 12. 정상 구축 기준

다음이 모두 정상이어야 한다.

```text
Docker Desktop 실행
WSL2 Engine 활성화
Ubuntu WSL Integration 활성화
docker ps 정상 실행
CVAT Container 실행 가능
Nuclio Container 실행 가능
GPU Container 사용 가능
```