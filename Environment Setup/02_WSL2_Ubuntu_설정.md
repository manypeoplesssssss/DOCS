# WSL2 Ubuntu 설치 및 설정

## 1. 역할

본 프로젝트에서 WSL2 Ubuntu는 다음 용도로 사용한다.

```text
CVAT Repository 접근
Nuclio nuctl 실행
Docker 명령 실행
YOLO26 학습
Python 가상환경
PyTorch CUDA 사용
Ultralytics 실행
```

Windows에서 CVAT는 Docker Desktop으로 실행하지만 개발 및 모델 학습 명령은 WSL Ubuntu에서 주로 실행한다.

---

# 2. 설치된 WSL 확인

PowerShell:

```powershell
wsl --list --verbose
```

또는:

```powershell
wsl -l -v
```

정상 예시:

```text
NAME              STATE       VERSION
Ubuntu            Running     2
```

중요:

```text
VERSION = 2
```

이어야 한다.

---

# 3. Ubuntu 실행

PowerShell:

```powershell
wsl
```

특정 배포판:

```powershell
wsl -d Ubuntu
```

---

# 4. WSL 종료

전체 종료:

```powershell
wsl --shutdown
```

문제가 발생했을 때:

```powershell
wsl --shutdown
```

후 Docker Desktop을 다시 실행하는 방법도 사용할 수 있다.

---

# 5. Windows 파일 접근

Windows C 드라이브:

```bash
/mnt/c
```

본 프로젝트 CVAT 경로:

```bash
cd /mnt/c/Users/82107/cvat
```

Windows에서는 같은 위치가:

```text
C:\Users\82107\cvat
```

이다.

---

# 6. Linux Home

현재 사용자:

```text
gi
```

Home:

```bash
/home/gi
```

이동:

```bash
cd ~
```

확인:

```bash
pwd
```

---

# 7. YOLO26 Workspace

```bash
cd /home/gi/yolo26-defect
```

이 경로에서는:

```text
Python venv
Dataset
Training
Validation
Inference
best.pt
```

등을 관리한다.

---

# 8. WSL IP 확인

```bash
hostname -I
```

또는:

```bash
ip addr
```

주의:

WSL 내부 IP는 재시작 시 변경될 수 있다.

외부 팀원의 CVAT 접속에는 WSL IP 대신 Windows에서 실행되는 Tailscale IP를 사용하였다.

---

# 9. 사용자 확인

```bash
whoami
```

결과:

```text
gi
```

---

# 10. Ubuntu 버전 확인

```bash
cat /etc/os-release
```

또는:

```bash
lsb_release -a
```

---

# 11. GPU 확인

WSL Ubuntu:

```bash
nvidia-smi
```

본 프로젝트에서는 다음 GPU가 정상 인식되었다.

```text
NVIDIA GeForce RTX 5060 Ti
```

---

# 12. Docker 연결 확인

```bash
docker ps
```

정상 작동해야 한다.

오류가 발생한다면 Docker Desktop:

```text
Settings
→ Resources
→ WSL Integration
```

에서 Ubuntu가 활성화되어 있는지 확인한다.

---

# 13. WSL 비밀번호를 잊었을 경우

Windows PowerShell에서 root로 Ubuntu를 실행한다.

배포판 이름 확인:

```powershell
wsl -l -v
```

root 실행 예:

```powershell
wsl -d Ubuntu -u root
```

Linux에서:

```bash
passwd gi
```

새 비밀번호를 설정한다.

끝:

```bash
exit
```

---

# 14. WSL 배포판 이름 주의

다음처럼 오타가 발생하면:

```text
Ubunutu-22.04
```

다음 오류가 발생할 수 있다.

```text
WSL_E_DISTRO_NOT_FOUND
```

반드시:

```powershell
wsl -l -v
```

로 실제 배포판 이름을 먼저 확인한다.

---

# 15. 정상 확인

```bash
whoami
pwd
docker ps
nvidia-smi
python3 --version
```

위 명령이 정상적으로 실행되면 기본 WSL 환경이 준비된 것이다.