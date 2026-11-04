# Codyssey Mission 1 — 개발자용 작업실 꾸미기

미션1. 개인용작업환경만들기

## 1. 미션 개요

Windows 환경에서 Git, VS Code, Docker를 설치하고 Linux 컨테이너, Dockerfile, 포트 매핑, 바인드 마운트, Docker 볼륨, Git 저장소를 실습했다.

## 2. 실행 환경

- OS: Windows
- Shell: PowerShell
- Docker: 29.9.2
- Docker Desktop: 4.98.0
- WSL: WSL 2
- Git 사용자: Codyssey Student

## 3. 수행 체크리스트

- [x] Windows 환경 확인
- [x] Git 설치 확인
- [x] VS Code 설치 확인
- [x] Docker 및 WSL 설치
- [x] Docker 엔진 점검
- [x] hello-world 실행
- [x] Ubuntu 컨테이너 실행
- [x] Linux 파일 및 디렉터리 권한 실습
- [x] Dockerfile 작성
- [x] 커스텀 NGINX 이미지 빌드
- [x] 포트 매핑 및 웹 접속
- [x] 바인드 마운트 변경 반영
- [x] Docker 볼륨 영속성 검증
- [x] Git 저장소 초기화 및 설정

## 4. Docker 기본 점검

```powershell
docker --version
docker info
docker run hello-world
docker images
docker ps -a
```

점검 결과:

![Docker 버전 확인](screenshots/docker-version.png)

![Docker 엔진 요약](screenshots/docker-info-summary.png)

![hello-world 실행 결과](screenshots/hello-world.png)

![Docker 이미지 목록](screenshots/docker-images.png)

![Docker 컨테이너 목록](screenshots/docker-containers.png)

## 5. 커스텀 웹 서버

베이스 이미지는 `nginx:alpine`을 사용했다. 직접 작성한 `index.html`을 NGINX 웹 루트에 복사했다.

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

### 빌드 및 실행

```powershell
docker build -t mission1-web:1.0 .
docker run -d --name mission1-web-8080 -p 8080:80 mission1-web:1.0
docker run -d --name mission1-web-8081 -p 8081:80 mission1-web:1.0
```

### 접속 검증

```powershell
curl.exe http://localhost:8080
curl.exe http://localhost:8081
```

브라우저 접속 주소:

- http://localhost:8080
- http://localhost:8081

접속 결과에는 다음 문구가 표시된다.

```text
Docker Web Server Test Success
```

접속 증거:

![8080 포트 접속](screenshots/web-port-8080.png)

![8081 포트 접속](screenshots/web-port-8081.png)

## 6. 바인드 마운트

호스트의 `bind-data` 폴더를 컨테이너의 `/data`에 연결했다.

```powershell
docker run --rm -v "C:\Users\82109\Documents\ChatGPT\코디세이\mission1\bind-data:/data" ubuntu cat /data/message.txt
```

호스트 파일을 수정한 뒤 컨테이너에서 변경된 내용을 확인하여 바인드 마운트 반영을 검증했다.

![바인드 마운트 변경 반영](screenshots/bind-mount.png)

## 7. Docker 볼륨 영속성

`mission1-data` 볼륨에 데이터를 저장한 뒤 컨테이너를 삭제하고 새 컨테이너에 같은 볼륨을 연결했다.

```powershell
docker volume create mission1-data
docker run -d --name mission1-volume-1 -v mission1-data:/data ubuntu sleep infinity
docker exec mission1-volume-1 sh -c "echo '볼륨 영속성 테스트 데이터' > /data/keep.txt"
docker rm -f mission1-volume-1
docker run --rm --name mission1-volume-2 -v mission1-data:/data ubuntu cat /data/keep.txt
```

컨테이너 삭제 후에도 데이터가 유지되는 것을 확인했다.

![Docker 볼륨 영속성 확인](screenshots/volume-persistence.png)

## 8. Git 설정

```powershell
git config --local --list
git branch -M main
git status
```

## 9. 증거 자료

모든 캡처는 [`screenshots`](screenshots/) 폴더에 저장했다.

### Linux 권한 실습

![Ubuntu 권한 변경 전후](screenshots/ubuntu-permissions.png)

### GitHub 저장소 반영 결과

![GitHub 저장소 파일 목록](screenshots/github-repository.png)

## 10. 트러블슈팅

### 문제 1: WSL 버전이 너무 오래됨

- 문제: Docker Desktop에서 `WSL needs updating` 메시지가 표시됨
- 원인 가설: 설치된 WSL 버전이 Docker Desktop 요구사항보다 낮음
- 확인: Docker 설치 화면의 오류 메시지 확인
- 해결:

```powershell
wsl --update
```

- 결과: WSL 업데이트 후 컴퓨터를 재시작하고 Docker Desktop이 정상 실행됨

### 문제 2: 바인드 마운트 폴더를 찾을 수 없음

- 문제: `bind-data\message.txt` 생성 시 경로를 찾을 수 없다는 오류 발생
- 원인 가설: `bind-data` 폴더가 실제로 생성되지 않음
- 해결:

```powershell
New-Item -ItemType Directory -Path .\bind-data -Force
```

- 결과: 폴더를 다시 생성한 뒤 파일 생성과 바인드 마운트 검증에 성공함

## 11. 주요 콘솔 출력 기록

### Docker 버전 및 엔진

```text
Docker version 29.9.2, build ...
Server Version: 29.9.2
Containers: 0
Images: 0
Operating System: Docker Desktop
Kernel Version: 6.18.40.1-microsoft-standard-WSL2
```

### hello-world

```powershell
docker run hello-world
```

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### 이미지·컨테이너 확인

```powershell
docker images
docker ps -a
```

실행 결과에는 `hello-world`, `ubuntu`, `mission1-web` 이미지와
`mission1-web-8080`, `mission1-web-8081`, `mission1-ubuntu` 등의 컨테이너가 표시되었다.

### Docker 이미지 빌드

```text
[+] Building ... FINISHED
=> naming to docker.io/library/mission1-web:1.0
```

### 웹 접속

```text
Docker Web Server Test Success
Codyssey Mission 1
```

### 바인드 마운트

```text
Host file updated successfully
```

### 볼륨 영속성

```text
Volume persistence verified
```

## 12. GitHub 연동 로그

```powershell
git remote -v
```

```text
origin  https://github.com/kilju0809-art/codyssey-mission1.git (fetch)
origin  https://github.com/kilju0809-art/codyssey-mission1.git (push)
```

```powershell
git push -u origin main
```

```text
main -> main
branch 'main' set up to track 'origin/main'
```

최근 커밋:

```powershell
git log --oneline -3
```

```text
실습 증거 스크린샷 추가
미션 1 개발 환경 구축
```

## 13. 프로젝트 구조

```text
mission1/
├── README.md                 # 미션 설명, 명령어, 검증 결과, 개념 정리
├── Dockerfile                # nginx 기반 커스텀 이미지 정의
├── index.html                # 웹 서버에서 제공하는 정적 페이지
├── practice.txt              # PowerShell 파일 관리 실습 파일
├── bind-data/
│   └── message.txt           # 바인드 마운트 검증용 호스트 파일
└── screenshots/              # 실행 결과 및 제출 증거 이미지
```

## 14. 재현 절차

### Windows PowerShell

1. Docker Desktop을 실행하고 Docker 엔진이 준비될 때까지 기다린다.
2. 프로젝트 폴더로 이동한다.
3. `docker --version`, `docker info`로 설치와 엔진을 확인한다.
4. `docker build -t mission1-web:1.0 .`으로 이미지를 빌드한다.
5. `docker run -d --name mission1-web-8080 -p 8080:80 mission1-web:1.0`으로 실행한다.
6. 브라우저에서 `http://localhost:8080`을 연다.
7. 볼륨과 바인드 마운트 명령을 실행해 데이터 연결을 검증한다.

### WSL/Linux 환경

Docker Desktop의 Linux 컨테이너 엔진을 사용한다. 호스트 경로는 환경마다 다르므로
Windows PowerShell에서는 절대 경로를 사용하고, WSL에서는 `/mnt/c/...` 형식으로 바꾼다.

## 15. 이미지와 컨테이너의 차이

이미지는 애플리케이션 실행에 필요한 파일과 설정을 묶은 읽기 전용 템플릿이다.
컨테이너는 이미지를 실제 프로세스로 실행한 인스턴스이며 실행·중지·삭제할 수 있다.
소스나 Dockerfile을 변경해도 기존 이미지가 자동으로 바뀌지 않으므로 이미지를 다시 빌드해야 한다.

```text
Dockerfile + index.html --build--> 이미지(mission1-web:1.0)
이미지 --run--> 컨테이너(mission1-web-8080)
```

## 16. 포트 포워딩과 네트워크 격리

컨테이너는 별도의 네트워크 네임스페이스를 사용하므로 컨테이너의 80번 포트가
호스트에 자동으로 노출되지 않는다. `-p 8080:80`은 호스트 8080번 포트의 요청을
컨테이너 80번 포트로 전달하는 포트 포워딩 설정이다.

포트 충돌이 발생하면 다음 순서로 확인한다.

```powershell
docker ps
docker ps -a
netstat -ano | findstr :8080
docker rm -f mission1-web-8080
```

필요하면 사용하지 않는 컨테이너를 종료하거나 호스트 포트를 `8082`처럼 바꿔 실행한다.

## 17. 경로와 권한 정리

절대 경로는 드라이브부터 시작하는 전체 경로이며, 상대 경로는 현재 폴더를 기준으로 한다.
호스트와 컨테이너를 연결하는 실습에서는 Windows 환경 차이를 줄이기 위해 절대 경로를 사용했다.
팀 공유 문서에서는 프로젝트 내부의 상대 경로와 환경별 대체 경로를 함께 기록하는 것이 재현에 유리하다.

Linux 권한은 소유자·그룹·기타 사용자 순서로 `rwx`를 표시한다.

```text
r = 읽기(4), w = 쓰기(2), x = 실행 또는 디렉터리 접근(1)
644 = 소유자 rw-, 그룹 r--, 기타 사용자 r--
755 = 소유자 rwx, 그룹 r-x, 기타 사용자 r-x
```

권한 확인과 변경:

```bash
ls -ld permission-test.txt permission-test-dir
chmod 644 permission-test.txt
chmod 755 permission-test-dir
ls -ld permission-test.txt permission-test-dir
```

결과 예시:

```text
-rw-r--r-- ... permission-test.txt
drwxr-xr-x ... permission-test-dir
```

## 18. 볼륨 백업 및 데이터 보호

볼륨은 컨테이너와 생명주기가 다르지만, 볼륨 자체가 자동 백업되는 것은 아니다.
중요한 데이터는 정기적으로 호스트 백업 또는 별도 저장소에 복사한다.
운영 환경에서는 스냅샷 정책, 백업 주기, 복구 테스트를 함께 정하고,
실습 환경에서는 데이터를 별도 백업 파일로 내보낸 뒤 복원 여부를 확인하는 방식을 사용한다.

```powershell
docker volume ls
docker volume inspect mission1-data
```

## 19. 최종 검증

- [x] README에 주요 명령과 핵심 출력 기록
- [x] Docker 버전 및 엔진 확인
- [x] hello-world 실행
- [x] Ubuntu 권한 변경 전후 확인
- [x] 이미지·컨테이너 목록 확인
- [x] Dockerfile 빌드 성공
- [x] 8080·8081 포트 접속
- [x] 바인드 마운트 변경 반영
- [x] 볼륨 삭제 후 데이터 유지
- [x] Git remote 및 push 기록
- [x] 프로젝트 구조와 파일 역할 설명
- [x] 재현 절차와 환경별 경로 주의사항
- [x] 이미지·컨테이너, 네트워크, 권한 개념 설명

## 20. Windows 파일 이동 실습

파일 이동은 PowerShell의 `Move-Item` 명령으로 수행했다. 아래 실습에서는
`practice.txt`를 `practice-moved.txt`로 이동한 뒤 원래 이름으로 되돌렸다.

```powershell
Move-Item -LiteralPath .\practice.txt -Destination .\practice-moved.txt
Move-Item -LiteralPath .\practice-moved.txt -Destination .\practice.txt
Get-ChildItem -Name practice*
```

`Move-Item`은 성공하면 별도 메시지를 출력하지 않으며, 마지막 목록 확인 결과는 다음과 같다.

```text
practice.txt
```

이를 통해 파일 이동 후 원래 위치와 파일명이 최종적으로 정상 복구되었음을 확인했다.
