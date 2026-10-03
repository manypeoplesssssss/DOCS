# Tailscale 설치 및 CVAT 외부 접속 설정

## 1. 목적

CVAT 서버와 팀원이 서로 다른 장소와 네트워크에 있어도 CVAT에 접속할 수 있도록 Tailscale을 사용한다.

예를 들어:

```text
[CVAT 서버 PC]
Windows + Docker Desktop
CVAT :8080
Tailscale
      │
      │ 인터넷
      │
      ▼
[Tailscale Network]
      │
      ├── 팀원 노트북
      ├── 팀원 PC
      └── 스마트폰
```

같은 Wi-Fi에 연결되어 있을 필요가 없다.

본 프로젝트에서는 CVAT 서버 PC의 Tailscale IP를 이용하여:

```text
http://100.71.200.28:8080
```

으로 접속하도록 구성하였다.

---

# 2. 설치 대상

Tailscale은 CVAT 서버 PC뿐만 아니라 CVAT에 접속할 팀원의 장치에도 설치한다.

```text
CVAT 서버 PC
    └── Tailscale 설치

팀원 PC
    └── Tailscale 설치

팀원 노트북
    └── Tailscale 설치

스마트폰
    └── 필요하면 Tailscale 설치
```

---

# 3. Windows 서버 PC에 Tailscale 설치

Windows에서 Tailscale 설치 프로그램을 설치한다.

설치가 완료되면 Tailscale을 실행한다.

Windows 작업표시줄의 시스템 트레이에서 Tailscale 아이콘을 확인할 수 있다.

---

# 4. Tailscale 로그인

Tailscale을 실행하면 로그인이 필요하다.

지원되는 계정 방식 중 사용할 계정으로 로그인한다.

로그인이 완료되면 해당 PC가 자신의 Tailnet에 등록된다.

---

# 5. 서버 PC 연결 확인

PowerShell을 실행한다.

```powershell
tailscale status
```

현재 연결된 장치들이 표시된다.

본 프로젝트의 서버 PC:

```text
desktop-grc9n5v
```

---

# 6. 서버 PC Tailscale IP 확인

PowerShell:

```powershell
tailscale ip -4
```

본 프로젝트에서는:

```text
100.71.200.28
```

이 할당되었다.

따라서 서버의 Tailscale 주소는:

```text
100.71.200.28
```

이다.

---

# 7. 현재 상태 확인

```powershell
tailscale status
```

여기서 서버 PC와 연결된 장치를 확인한다.

예:

```text
100.71.200.28    desktop-grc9n5v
```

---

# 8. 팀원 추가

팀원도 서버와 같은 Tailnet에 접근할 수 있어야 한다.

관리자는 Tailscale 관리 화면에서 팀원을 초대하거나 접근 권한을 부여한다.

팀원은 초대받은 계정으로 Tailscale에 로그인한다.

---

# 9. 팀원 PC 설정

팀원 PC에서도 Tailscale을 설치한다.

설치 후:

```text
Tailscale 실행
        ↓
초대받은 계정 로그인
        ↓
Tailnet 참여
        ↓
연결 승인
```

순서로 진행한다.

---

# 10. 팀원 연결 확인

서버 PC PowerShell:

```powershell
tailscale status
```

팀원 장치가 표시되는지 확인한다.

구조:

```text
CVAT Server
100.71.200.28

       │
       │ Tailscale
       │
       ├──── Team PC 1
       │
       ├──── Team PC 2
       │
       └──── Smartphone
```

---

# 11. CVAT_HOST 설정

Tailscale만 연결했다고 바로 CVAT가 Tailscale IP로 정상 동작하는 것은 아니다.

CVAT의 Traefik Routing에서도 서버 주소를 알아야 한다.

Windows PowerShell에서 CVAT Repository로 이동한다.

```powershell
cd C:\Users\82107\cvat
```

Tailscale IP를 CVAT_HOST로 설정한다.

```powershell
$env:CVAT_HOST="100.71.200.28"
```

확인:

```powershell
echo $env:CVAT_HOST
```

결과:

```text
100.71.200.28
```

---

# 12. CVAT Serverless 포함 실행

같은 PowerShell에서:

```powershell
docker compose `
  -f docker-compose.yml `
  -f components/serverless/docker-compose.serverless.yml `
  up -d
```

중요:

```text
$env:CVAT_HOST
```

를 설정한 PowerShell에서 Docker Compose를 실행한다.

---

# 13. Container 확인

```powershell
docker ps
```

CVAT 관련 Container들이 실행되어 있어야 한다.

---

# 14. CVAT API 테스트

서버 PC에서:

```powershell
curl.exe -I http://100.71.200.28:8080/api/server/about
```

정상이라면:

```text
HTTP/1.1 200 OK
```

와 같은 응답이 반환된다.

이 단계가 성공하면:

```text
Tailscale IP
        ↓
Traefik
        ↓
CVAT Server
```

Routing이 정상이라는 의미이다.

---

# 15. 서버 PC 브라우저 테스트

브라우저:

```text
http://100.71.200.28:8080
```

CVAT 로그인 화면이 나타나는지 확인한다.

---

# 16. 팀원 PC에서 접속

팀원 PC에서 Tailscale이 연결되어 있는지 먼저 확인한다.

PowerShell:

```powershell
tailscale status
```

그 다음 브라우저:

```text
http://100.71.200.28:8080
```

으로 접속한다.

CVAT 로그인 화면이 나타나면 성공이다.

---

# 17. 스마트폰에서 접속

스마트폰에도 Tailscale을 설치할 수 있다.

Tailscale 앱:

```text
설치
 ↓
로그인
 ↓
Tailnet 연결
```

그 다음 모바일 브라우저:

```text
http://100.71.200.28:8080
```

으로 접속한다.

---

# 18. 서버 PC를 계속 켜놔야 하는가?

그렇다.

CVAT 서버는 현재 개인 PC에서 Docker로 실행되고 있기 때문에 팀원이 접속하려면 서버 PC가 실행 중이어야 한다.

필요한 상태:

```text
Windows PC ON

        ↓

Internet 연결

        ↓

Tailscale ON

        ↓

Docker Desktop ON

        ↓

CVAT Containers ON
```

서버 PC가 꺼지면 팀원도 CVAT에 접속할 수 없다.

---

# 19. Docker Desktop 확인

팀원이 접속하지 못한다면 서버에서:

```powershell
docker ps
```

를 실행한다.

CVAT Container가 정상 실행 중인지 확인한다.

---

# 20. Tailscale 확인

```powershell
tailscale status
```

서버가 Tailscale Network에 연결되어 있는지 확인한다.

IP:

```powershell
tailscale ip -4
```

현재:

```text
100.71.200.28
```

---

# 21. CVAT_HOST 확인

```powershell
echo $env:CVAT_HOST
```

정상:

```text
100.71.200.28
```

아무것도 나오지 않는다면:

```powershell
$env:CVAT_HOST="100.71.200.28"
```

다시 설정한다.

---

# 22. CVAT 재실행

```powershell
docker compose `
  -f docker-compose.yml `
  -f components/serverless/docker-compose.serverless.yml `
  up -d
```

그 다음:

```powershell
curl.exe -I http://100.71.200.28:8080/api/server/about
```

으로 확인한다.

---

# 23. 중요한 CVAT_HOST 주의사항

다음 명령:

```powershell
$env:CVAT_HOST="100.71.200.28"
```

은 현재 PowerShell 프로세스에서 사용하는 환경변수이다.

따라서 PowerShell을 종료하고 새로운 PowerShell에서 Docker Compose를 실행하면 값이 존재하지 않을 수 있다.

항상 먼저:

```powershell
echo $env:CVAT_HOST
```

를 확인한다.

필요하면:

```powershell
$env:CVAT_HOST="100.71.200.28"
```

를 다시 실행한다.

---

# 24. localhost는 되는데 Tailscale IP는 안 되는 경우

예:

```text
http://localhost:8080
```

은 되는데:

```text
http://100.71.200.28:8080
```

은 안 되는 경우가 있다.

먼저:

```powershell
echo $env:CVAT_HOST
```

확인.

다시 설정:

```powershell
$env:CVAT_HOST="100.71.200.28"
```

그리고:

```powershell
docker compose `
  -f docker-compose.yml `
  -f components/serverless/docker-compose.serverless.yml `
  up -d
```

실행한다.

본 프로젝트에서도 Traefik Host Routing을 Tailscale IP에 맞추는 과정이 필요했다.

---

# 25. Traefik

CVAT는 Traefik을 Reverse Proxy로 사용한다.

구조:

```text
Team PC

http://100.71.200.28:8080
          │
          ▼
      Tailscale
          │
          ▼
       Windows
          │
          ▼
       Traefik
          │
          ▼
      CVAT Server
```

따라서 Tailscale 자체가 정상이어도 Traefik의 Host Routing이 맞지 않으면 CVAT에 접속하지 못할 수 있다.

---

# 26. Tailscale Serve

Tailscale Serve를 이용해 CVAT를 Forward하는 방법도 테스트하였다.

하지만:

```text
Tailscale hostname
        ↓
Traefik Host Rule
```

이 서로 맞지 않으면 CVAT Routing 문제가 발생할 수 있다.

본 프로젝트에서는 최종적으로 Serve를 통한 우회보다:

```text
Tailscale IP 직접 접속
```

방식을 사용하였다.

최종:

```text
http://100.71.200.28:8080
```

---

# 27. 다른 Wi-Fi에서도 가능한가?

가능하다.

예:

```text
CVAT 서버 PC
광주 집 Wi-Fi

        ↓
     Internet
        ↓
    Tailscale
        ↓

팀원 PC
학교 Wi-Fi
```

처럼 서로 다른 네트워크에서도 Tailnet에 연결되어 있다면 통신할 수 있다.

---

# 28. 보안상 주의

CVAT의 `8080` 포트를 인터넷 전체에 직접 공개하는 방식과 Tailscale 방식은 다르다.

현재 구성은:

```text
Internet
   │
   X
직접 8080 공개
```

가 아니라:

```text
허용된 Tailscale 장치
        ↓
     Tailnet
        ↓
   CVAT Server
```

형태로 운영하는 것이 목적이다.

---

# 29. 팀원 접속 체크리스트

팀원이 접속되지 않으면 다음 순서로 확인한다.

```text
1. CVAT 서버 PC가 켜져 있는가?

2. 서버 PC 인터넷이 연결되어 있는가?

3. Docker Desktop이 실행 중인가?

4. CVAT Container가 실행 중인가?

5. 서버 Tailscale이 연결되어 있는가?

6. 팀원 Tailscale이 연결되어 있는가?

7. 같은 Tailnet에 접근 권한이 있는가?

8. CVAT_HOST가 100.71.200.28인가?

9. API가 200을 반환하는가?

10. http://100.71.200.28:8080 접속이 되는가?
```

---

# 30. 서버 점검 명령어

Tailscale:

```powershell
tailscale status
```

IP:

```powershell
tailscale ip -4
```

CVAT Host:

```powershell
echo $env:CVAT_HOST
```

Docker:

```powershell
docker ps
```

API:

```powershell
curl.exe -I http://100.71.200.28:8080/api/server/about
```

---

# 31. 최종 완료 상태

다음 상태가 되면 Tailscale 설정 완료이다.

```text
CVAT 서버 PC
    │
    ├── Tailscale 연결
    │
    ├── IP: 100.71.200.28
    │
    ├── Docker Desktop 실행
    │
    └── CVAT :8080 실행
             │
             │
          Tailnet
             │
        ┌────┴────┐
        │         │
     팀원 PC    스마트폰
        │         │
        └────┬────┘
             │
             ▼
http://100.71.200.28:8080
```

최종 접속 주소:

```text
http://100.71.200.28:8080
```

이 주소를 이용하여 서로 다른 네트워크에서도 팀원들이 CVAT에 접속할 수 있다.