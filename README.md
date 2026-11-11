# Development Workstation Setup

## 1. 프로젝트 개요

Windows 환경에서 WSL2 Ubuntu, Docker Desktop, Git, GitHub, VS Code를 이용해 개발 워크스테이션을 구성하고 검증한 기록이다.

이번 수행에서는 다음 항목을 확인했다.

- Linux CLI 기본 조작
- 파일/디렉터리 권한 변경
- Docker 설치 및 기본 동작 확인
- Docker 이미지/컨테이너 실행 및 로그 확인
- Dockerfile 기반 커스텀 웹 서버 이미지 빌드
- 포트 매핑
- Bind Mount
- Docker Volume 영속성
- Git 로컬 버전 관리
- GitHub 공개 저장소 연결 및 Push
- VS Code의 WSL Ubuntu 연결
- 실제 오류 3건에 대한 문제 해결


---

## 2. 실행 환경

| 항목 | 값 |
|---|---|
| Host OS | Windows |
| Linux 환경 | WSL2 |
| Ubuntu | 26.04.1 LTS |
| Shell | Bash (`/bin/bash`) |
| Git | 2.53.0 |
| Docker CLI | 29.9.2 |
| Docker Engine | 29.9.2 |
| Docker Desktop | 4.98.0 |
| Architecture | x86_64 |
| GitHub 사용자 | `shrudals2019` |
| GitHub 저장소 | https://github.com/shrudals2019/dev-workstation |

---

## 3. 수행 체크리스트

- ✅ WSL2 Ubuntu 설치 및 실행
- ✅ Linux 사용자 및 Bash 환경 확인
- ✅ Git 설치 상태 확인
- ✅ Docker Desktop 설치
- ✅ Docker Desktop과 WSL Ubuntu 연동
- ✅ `docker --version`, `docker info` 검증
- ✅ `hello-world` 컨테이너 실행
- ✅ `docker images`, `docker ps`, `docker ps -a` 확인
- ✅ Ubuntu 컨테이너 실행
- ✅ 실행 중 컨테이너에 `docker exec`로 진입
- ✅ 메인 프로세스 종료와 컨테이너 종료 관계 확인
- ✅ `docker logs` 확인
- ✅ Dockerfile 작성
- ✅ 커스텀 이미지 `dev-web:v1` 빌드
- ✅ 포트 매핑 `8080:80`
- ✅ Bind Mount 변경 전/후 확인
- ✅ Docker Volume 생성
- ✅ 컨테이너 삭제 후 Volume 데이터 유지 확인
- ✅ `docker stats` 확인
- ✅ Linux 터미널 기본 조작
- ✅ 파일/디렉터리 권한 변경
- ✅ Git 전역 사용자 설정
- ✅ Git 저장소 초기화 및 첫 커밋
- ✅ GitHub 공개 저장소 연결 및 Push
- ✅ VS Code WSL Ubuntu 연결
- ✅ Source Control에서 저장소 인식

---


> 주요 명령어와 검증 결과는 README 본문에 요약해 두었고, 화면 증거는 `evidence/` 폴더에 별도 이미지 파일로 분리했다.


## 4. 프로젝트 구조

```text
dev-workstation/
├── README.md
├── Dockerfile
├── app/
│   └── index.html
├── terminal-practice/
│   ├── .hidden.txt
│   ├── empty.txt
│   ├── file1.txt
│   └── permission.txt
└── evidence/
    ├── github-public-repository.png
    ├── github-public-settings.png
    └── vscode-wsl-connected.png
```

> 포트 매핑 브라우저 검증 이미지는 실제 `http://localhost:8080` 주소창과 웹페이지가 함께 보이는 캡처를 별도로 추가하는 것이 가장 안전하다.

---

## 5. 터미널 및 권한 실습

터미널 실습에서는 다음을 수행했다.

- `pwd`로 현재 경로 확인
- `ls`, `ls -la`로 일반/숨김 파일 확인
- `mkdir`, `touch`로 폴더/파일 생성
- `echo`, `cat`으로 파일 내용 작성/확인
- `cp`로 복사
- `mv`로 이동 및 이름 변경
- `rm`, `rmdir`로 삭제
- `cd`로 경로 이동

권한 실습:

- 파일: `600` → `644`
- 디렉터리: `700` → `755`

결과:

```text
-rw-------  ->  -rw-r--r--
drwx------  ->  drwxr-xr-x
```

`r = 4`, `w = 2`, `x = 1`이며 `644`, `755`는 이 값을 조합한 숫자 권한이다.


---

## 6. Docker 기본 검증

Docker CLI와 Engine이 정상 동작함을 확인했다.

주요 검증:

```text
Docker version 29.9.2
Server Version: 29.9.2
Operating System: Docker Desktop
Kernel: WSL2
```

`hello-world` 실행 결과:

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

또한 다음 명령으로 이미지와 컨테이너 상태를 확인했다.

```bash
docker images
docker ps
docker ps -a
docker logs attach-test
docker stats --no-stream
```


---

## 7. Ubuntu 컨테이너와 exec

실행 중인 Ubuntu 컨테이너:

```bash
docker run -dit --name ubuntu-run ubuntu sleep infinity
```

실행 중 컨테이너에 진입:

```bash
docker exec -it ubuntu-run bash
```

컨테이너 내부에서:

```bash
ls
echo "Hello Docker Container"
```

을 실행했다.

`docker exec`로 실행한 bash를 종료해도 메인 프로세스 `sleep infinity`가 계속 실행 중이므로 컨테이너는 유지되었다.

반면 `bash` 자체를 메인 프로세스로 실행한 `attach-test` 컨테이너는 `exit` 시 메인 프로세스가 끝나면서 컨테이너도 `Exited (0)` 상태가 되었다.

---

## 8. Dockerfile과 커스텀 웹 서버

`Dockerfile`:

```dockerfile
FROM nginx:alpine
COPY app/index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

빌드:

```bash
docker build -t dev-web:v1 .
```

결과:

```text
[+] Building 6.6s (8/8) FINISHED
naming to docker.io/library/dev-web:v1
```

이미지 목록에서 `dev-web:v1` 생성도 확인했다.

---

## 9. 포트 매핑

웹 서버 컨테이너 실행:

```bash
docker run -d --name dev-web -p 8080:80 dev-web:v1
```

`docker ps` 결과:

```text
0.0.0.0:8080->80/tcp
```

의미:

```text
Host 8080 → Container 80 → Nginx
```

검증 주소:

```text
http://localhost:8080
```

> 평가 증거로는 주소창의 `http://localhost:8080`과 페이지 내용이 함께 보이는 브라우저 캡처를 `evidence/port-mapping-browser.png`로 추가하는 것이 가장 확실하다.

---

## 10. Bind Mount

실행:

```bash
docker run -d --name bind-web -p 8081:80 -v "$PWD/app:/usr/share/nginx/html:ro" nginx:alpine
```

변경 전:

```text
Hello Docker Web!
Dockerfile web server test
```

호스트의 `app/index.html`을 수정한 뒤 다시 요청했을 때:

```text
Bind Mount Changed!
```

로 즉시 변경되었다.

이미지를 다시 빌드하거나 컨테이너를 재생성하지 않고도 호스트 파일 변경이 컨테이너에 반영됨을 확인했다.

---

## 11. Docker Volume 영속성

Volume 생성:

```bash
docker volume create mission-data
```

데이터 저장:

```text
persistent data
```

기존 컨테이너 삭제:

```bash
docker rm volume-write
```

새 컨테이너에서 동일 Volume을 연결한 뒤:

```text
persistent data
```

가 다시 확인되었다.

즉, 컨테이너를 삭제해도 Docker Volume의 데이터는 유지되는 것을 검증했다.

---

## 12. Git / GitHub

Git 설정:

```text
user.name  = shrudals2019
user.email = shrudals2019@users.noreply.github.com
branch     = main
```

저장소 초기화와 첫 커밋:

```text
f3fd21b Initial development workstation setup
```

원격 저장소:

```text
https://github.com/shrudals2019/dev-workstation.git
```

GitHub 공개 저장소에 Push한 뒤 웹 페이지에서 `Dockerfile`, `app/`, `terminal-practice/`가 업로드된 것을 확인했다.

관련 증거:

- `evidence/github-public-settings.png`
- `evidence/github-public-repository.png`

---

## 13. VS Code와 WSL Ubuntu

처음에는 VS Code가 Windows Local 모드로 열려 `Download Git for Windows`가 표시되었다.

Git은 WSL Ubuntu에 설치되어 있었으므로 Microsoft WSL 확장을 설치한 뒤 Ubuntu에 연결했다.

최종적으로 VS Code 왼쪽 아래에:

```text
WSL: Ubuntu
```

가 표시되는 상태에서 프로젝트를 열었다.

관련 증거:

- `evidence/vscode-wsl-connected.png`

---

# 14. Troubleshooting

## 1) WSL Ubuntu 설치 실패

### 문제

```powershell
wsl --install -d Ubuntu
```

실행 시:

```text
Wsl/InstallDistro/WININET_E_NAME_NOT_RESOLVED
서버 이름이나 주소를 확인할 수 없습니다.
```

오류가 발생했다.

### 원인 가설

Ubuntu 배포판 다운로드 과정에서 서버 주소를 정상적으로 해석하지 못한 네트워크/DNS 계열 문제로 판단했다.

### 확인

명령줄 설치가 진행되지 않았고 Ubuntu 배포판이 정상 설치되지 않은 상태였다.

### 해결

Microsoft Store에서 Ubuntu를 직접 설치했다.

설치 후:

```bash
uname -s
cat /etc/os-release
whoami
echo "$SHELL"
```

을 실행해 Ubuntu 26.04.1 LTS, Linux, Bash 환경이 정상임을 확인했다.

---

## 2) Ubuntu에서 Docker 명령 미인식

### 문제

```bash
docker --version
```

결과:

```text
docker: command not found
```

### 원인 가설

Docker Desktop은 Windows에 설치되어 있었지만 WSL Ubuntu와 Docker CLI가 연결되지 않은 상태라고 판단했다.

### 확인

Ubuntu 터미널에서 Docker 명령 자체를 찾지 못했다.

### 해결

Docker Desktop의 WSL Integration을 활성화하고 Ubuntu와 연결했다.

이후:

```text
Docker version 29.9.2
```

가 정상 출력되었다.

---

## 3) Docker API 소켓 권한 오류

### 문제

WSL Integration 이후:

```bash
docker info
```

실행 시:

```text
permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
```

오류가 발생했다.

### 원인 가설

Docker Engine 자체 문제보다 일반 사용자 `nkm`의 Docker 소켓 접근 권한 문제로 판단했다.

### 확인

```bash
sudo docker info
```

를 실행하자 Server 정보가 정상적으로 출력되었다.

따라서 Docker Engine은 정상이고 사용자 권한 문제임을 확인했다.

### 해결

```bash
sudo usermod -aG docker $USER
```

로 사용자를 Docker 그룹에 추가한 뒤 PowerShell에서:

```powershell
wsl --shutdown
```

을 실행했다.

Ubuntu를 다시 열고 `sudo` 없이:

```bash
docker info
```

를 실행했을 때 Client와 Server 정보가 모두 정상 출력되어 해결을 확인했다.

---

## 15. 검증 파일 위치

### 상세 터미널/도커/Git 로그

```text
```

### GitHub 공개 저장소 증거

```text
evidence/github-public-settings.png
evidence/github-public-repository.png
```

### VS Code WSL 연결 증거

```text
evidence/vscode-wsl-connected.png
```

### 추가로 반드시 확보 권장

```text
evidence/port-mapping-browser.png
```

이 파일은 실제 브라우저에서 `http://localhost:8080` 주소창과 웹페이지가 함께 보이도록 캡처한 이미지여야 한다.

---

## 16. Git과 GitHub의 차이

- **Git**: 로컬에서 파일 변경 이력을 관리하는 버전 관리 도구
- **GitHub**: Git 저장소를 온라인에 저장하고 공유하는 서비스

정리:

```text
Git    = 버전 관리
GitHub = 원격 저장소 및 협업 서비스
```

---

## 17. 최종 결과

이번 수행을 통해 다음 개발 환경을 구성했다.

```text
Windows
└─ WSL2 Ubuntu 26.04.1 LTS
   ├─ Bash
   ├─ Git 2.53.0
   ├─ Docker CLI 29.9.2
   │  └─ Docker Desktop 4.98.0 / Engine 29.9.2
   └─ VS Code WSL
```

수행 흐름:

```text
Linux CLI
→ 권한
→ Docker 설치/검증
→ 이미지와 컨테이너
→ Dockerfile
→ 포트 매핑
→ Bind Mount
→ Volume
→ Git
→ GitHub
→ VS Code WSL
```

최종 공개 저장소:

**https://github.com/shrudals2019/dev-workstation**