# Docker 설명서

> 실습 프로젝트 `oz_class/docker` (FastAPI + MySQL)를 기준으로,
> **"왜 이걸 하는지 → 파일에 뭘 쓰는지 → 터미널에 뭘 치는지 → 그때 안에서 무슨 일이 일어나는지"** 순서로 정리

## 목차
1. [Docker가 뭐고 왜 쓰나](#1-docker가-뭐고-왜-쓰나)
2. [꼭 알아야 할 핵심 개념](#2-꼭-알아야-할-핵심-개념)
3. [오늘 실습의 전체 그림](#3-오늘-실습의-전체-그림)
4. [준비 : Docker Desktop 실행](#4-준비--docker-desktop-실행)
5. [폴더와 파일 만들기 (한 줄씩 해설)](#5-폴더와-파일-만들기-한-줄씩-해설)
6. [실행 순서 따라가기 (Step by Step)](#6-실행-순서-따라가기-step-by-step)
7. [헷갈리는 것 비교 정리](#7-헷갈리는-것-비교-정리)
8. [명령어 사전](#8-명령어-사전)
9. [자주 만나는 에러와 해결](#9-자주-만나는-에러와-해결)
10. [다음 단계 : 개선된 docker-compose.yml](#10-다음-단계--개선된-docker-composeyml)
11. [한 장 요약](#11-한-장-요약)

---

# 1. Docker가 뭐고 왜 쓰나

## Docker 없이 개발하면 생기는 문제
- FastAPI 서버 + MySQL DB로 프로젝트를 하려면 내 컴퓨터에
  - Python 3.13 설치, 패키지(fastapi, sqlalchemy ...) 설치
  - MySQL 설치, 비밀번호 설정, DB 생성 ...
- 팀원 컴퓨터는 Python 3.11, Mac, MySQL 5.7 ... → **"내 컴퓨터에선 되는데 왜 너는 안 돼?"**
- 실제 서버(리눅스)에 올릴 때 또 처음부터 설치

## Docker가 해결하는 방법
- 프로그램 + 실행에 필요한 모든 것(OS, Python, 패키지, 설정)을 **하나의 상자로 포장**
- 이 상자를 **컨테이너(Container)** 라고 함
- 상자만 있으면 **어느 컴퓨터에서든 똑같이 실행**됨 (Windows, Mac, 리눅스 서버 모두)

> 비유 : 화물 **컨테이너**
> 안에 뭐가 들었든(옷, 전자제품, 과일) 규격이 같으니까 배·트럭·기차 어디에든 똑같이 실어 나를 수 있음.
> Docker 컨테이너도 안에 뭐가 들었든(FastAPI, MySQL, Redis) 똑같은 방식(`docker run`)으로 실행 가능.

## 가상머신(VM)과의 차이
- 둘 다 "내 컴퓨터 안에 격리된 또 다른 환경"을 만드는 기술
- VM은 **OS 전체**를 통째로 하나 더 띄움 → 무겁고 느림 (GB 단위, 부팅 수십 초)
- 컨테이너는 **OS의 핵심(커널)은 공유**하고 필요한 것만 담음 → 가볍고 빠름 (MB 단위, 시작 1초)

```
가상머신(VM) : 앱마다 OS 통째로        컨테이너(Docker) : 커널은 공유
┌────────────┐ ┌────────────┐          ┌────────────┐ ┌────────────┐
│   App A    │ │   App B    │          │   App A    │ │   App B    │
├────────────┤ ├────────────┤          ├────────────┤ ├────────────┤
│ 라이브러리 │ │ 라이브러리 │          │ 라이브러리 │ │ 라이브러리 │
├────────────┤ ├────────────┤          └────────────┘ └────────────┘
│ 게스트 OS  │ │ 게스트 OS  │          ┌─────────────────────────┐
└────────────┘ └────────────┘          │      Docker Engine      │
┌─────────────────────────┐            └─────────────────────────┘
│      하이퍼바이저       │            ┌─────────────────────────┐
└─────────────────────────┘            │  호스트 OS (커널 공유)  │
┌─────────────────────────┐            └─────────────────────────┘
│        호스트 OS        │
└─────────────────────────┘
```

> Windows에서는 Docker Desktop이 내부적으로 **WSL2(작은 리눅스)** 를 띄우고 그 위에서 컨테이너를 실행함.
> 그래서 컨테이너 안은 항상 **리눅스** 환경 (`bash`, `/app` 같은 경로를 쓰는 이유)

---

# 2. 꼭 알아야 할 핵심 개념

## 개념 4총사의 관계

```
 ┌────────────────┐                  ┌────────────────┐                  ┌────────────────┐
 │   Dockerfile   │  docker build    │     Image      │   docker run     │   Container    │
 │    (레시피)    │ ───────────────▶ │  (붕어빵 틀)   │ ───────────────▶ │    (붕어빵)    │
 │  텍스트 파일   │                  │   읽기 전용    │                  │  실행 중인 것  │
 └────────────────┘                  └────────────────┘                  └────────────────┘
                                             ▲                         하나의 이미지로
                                             │ docker pull             컨테이너 여러 개 생성 가능
                                    ┌────────────────┐
                                    │   Docker Hub   │
                                    │(이미지 저장소) │
                                    └────────────────┘

 ┌──────────────────────────────────────────────────────────────────────────────────────┐
 │ docker-compose.yml : 위 과정을 여러 컨테이너(api, db)에 대해 한 번에 해주는 설정 파일│
 └──────────────────────────────────────────────────────────────────────────────────────┘
```

### Image (이미지)
- 컨테이너를 만들기 위한 **틀 / 설계도의 완성본**
- OS + 프로그램 + 패키지 + 코드가 **얼려진(스냅샷) 상태**로 들어 있음 → **읽기 전용, 수정 불가**
- 예) `python:3.13-slim`, `mysql:8.0`, 우리가 만든 `docker-api`
- 이름 형식 : `이름:태그` (태그 = 버전) → `mysql:8.0` = MySQL 8.0 버전 이미지

### Container (컨테이너)
- 이미지를 **실행한 것** (이미지 = 프로그램 설치 파일, 컨테이너 = 실행 중인 프로그램)
- 하나의 이미지로 컨테이너를 여러 개 만들 수 있음 (붕어빵 틀 하나 → 붕어빵 여러 개)
- 컨테이너 안에서 바꾼 것(파일, DB 데이터)은 **그 컨테이너에만** 존재 → 컨테이너를 삭제하면 같이 사라짐 ⚠️

### Dockerfile
- **"내 이미지를 이렇게 만들어줘"** 라는 레시피 텍스트 파일
- "파이썬 깔린 리눅스에서 시작해서 → 내 코드 복사하고 → 패키지 설치하고 → 이 명령어로 실행해"
- 이미 남이 만든 이미지(`mysql:8.0`)를 그대로 쓸 땐 필요 없음. **내 코드가 들어간 이미지**가 필요할 때만 작성

### Docker Compose (`docker-compose.yml`)
- 컨테이너 **여러 개를 한 번에** 정의하고 실행하는 도구
- 우리 프로젝트는 컨테이너가 2개 (api 서버 + db) → 각각 `docker run ...`을 길게 치는 대신 yml 파일에 적어두고 `docker compose up` 한 번으로 실행

### Docker Hub
- 이미지를 올리고 내려받는 **온라인 저장소** (GitHub의 이미지 버전)
- `mysql:8.0`, `python:3.13-slim` 같은 공식 이미지가 여기 있음
- 내 컴퓨터에 없는 이미지를 쓰면 자동으로 여기서 다운로드(pull)

> Git에 비유하면
> | Git | Docker |
> |---|---|
> | GitHub (원격 저장소) | Docker Hub (이미지 저장소) |
> | `git clone` | `docker pull` |
> | 커밋(스냅샷) | 이미지(스냅샷) |

## 알아두면 좋은 보조 개념

### 포트 매핑 (Port Mapping)
- 컨테이너는 격리되어 있어서 **기본적으로 밖에서 접근 불가**
- `9000:8000` = "내 컴퓨터(호스트)의 **9000번** 문으로 들어온 요청을 컨테이너의 **8000번** 문으로 연결해줘"
- 순서 주의 : **`호스트포트:컨테이너포트`** (왼쪽이 내 컴퓨터, 오른쪽이 컨테이너)

```
 내 컴퓨터 (Windows)                         컨테이너
 브라우저 → localhost:9000  ═══════════▶  :8000 (uvicorn 서버)
                          포트 매핑 9000:8000
```

### 네트워크 (Network)
- Compose는 실행할 때 컨테이너들끼리 통신할 수 있는 **전용 네트워크**를 자동으로 만듦 (`docker_default`)
- 같은 네트워크 안에서는 **서비스 이름(`api`, `db`)이 곧 주소** → api 컨테이너에서 `db:3306`으로 MySQL에 접속 가능

### 볼륨 (Volume)
- 컨테이너가 삭제돼도 **데이터를 보존**하기 위해 컨테이너 밖에 저장하는 공간
- 지금 실습 파일에는 볼륨이 없음 → 그래서 `docker compose down` 하면 DB 데이터가 날아감 (→ [Step 8](#step-8-docker-compose-down--up--데이터가-사라진다) 참고)

---

# 3. 오늘 실습의 전체 그림

> 이 그림이 머릿속에 있으면 나머지가 전부 이해됨

```
┌─ 내 컴퓨터 (Windows) ──────────────────────────────────────────────────────────────────┐
│ VS Code                    PowerShell (터미널)              브라우저                   │
│ oz_class/docker/           docker compose up -d             localhost:9000/users       │
│  ├ Dockerfile              docker ps                              │                    │
│  ├ docker-compose.yml      docker exec -it docker-db-1 bash       │                    │
│  ├ main.py                     │ 명령 전달                        │ 9000번 포트        │
│  └ connection.py               ▼                                  ▼                    │
│ ┌─ Docker Desktop (Docker Engine) ───────────────────────────────────────────────────┐ │
│ │ 포트 매핑 9000:8000 으로 브라우저 요청을 api 컨테이너에 전달                       │ │
│ │                                                                                    │ │
│ │ ┌─ 네트워크 : docker_default ────────────────────────────────────────────────────┐ │ │
│ │ │ ┌─ 컨테이너 docker-api-1 ──────────┐      ┌─ 컨테이너 docker-db-1 ───────────┐ │ │ │
│ │ │ │ 이미지 : docker-api (직접 빌드)  │      │ 이미지 : mysql:8.0 (Docker Hub)  │ │ │ │
│ │ │ │                                  │      │                                  │ │ │ │
│ │ │ │ /app/main.py, connection.py      │      │ MySQL 서버 :3306                 │ │ │ │
│ │ │ │ uvicorn 서버 :8000               │      │  └ DB : fastapi                  │ │ │ │
│ │ │ │   ▲ 브라우저 요청 (9000→8000)    │      │     └ 테이블 : user              │ │ │ │
│ │ │ │                                  │      │                                  │ │ │ │
│ │ │ │ "db:3306 으로 SQL 보내줘"       │─────▶│ SQL 받아서 결과 돌려줌           │ │ │ │
│ │ │ │                                  │      │                                  │ │ │ │
│ │ │ └──────────────────────────────────┘      └──────────────────────────────────┘ │ │ │
│ │ └────────────────────────────────────────────────────────────────────────────────┘ │ │
│ │                                                                                    │ │
│ └────────────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                        │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

### 요청 하나가 처리되는 흐름
1. 브라우저에서 `localhost:9000/users` 접속
2. 포트 매핑(9000 → 8000)으로 **api 컨테이너**의 uvicorn 서버에 도착
3. `main.py`의 `/users` 함수 실행 → `SELECT * FROM user;` 를 `db:3306`으로 보냄
4. 같은 네트워크의 **db 컨테이너**(MySQL)가 결과 반환
5. FastAPI가 JSON으로 바꿔서 브라우저에 응답

```
브라우저 ──9000──▶ [api 컨테이너 :8000] ──db:3306──▶ [db 컨테이너 MySQL]
    ▲                        │                                │
    └──────────── JSON 응답 ◀┴──────────────────── 조회 결과 ◀┘
```

---

# 4. 준비 : Docker Desktop 실행

## 왜 Docker Desktop을 먼저 켜야 하나?
- 터미널에서 치는 `docker` 명령어는 **리모컨**일 뿐
- 실제로 이미지를 만들고 컨테이너를 돌리는 건 **Docker Engine(엔진)** 이고, Windows에서는 Docker Desktop이 이 엔진을 켜줌

```
 터미널 (docker 명령어) ──"컨테이너 실행해줘"──▶ Docker Engine ──▶ 컨테이너 생성/실행
     = 리모컨 (클라이언트)                          = TV 본체 (Docker Desktop이 켜줌)
```

- Docker Desktop을 안 켜고 명령어를 치면 아래 같은 에러 → **"TV가 꺼져 있다"는 뜻**

```
error during connect: ... open //./pipe/dockerDesktopLinuxEngine: The system cannot find the file specified.
```

## 실행 확인
1. Docker Desktop 실행 → 왼쪽 아래 **Engine running** (초록색) 확인
2. 터미널에서 확인

```bash
docker --version   # Docker가 설치되어 있는지 (버전 출력)
docker ps          # 엔진이 켜져 있는지 (에러 없이 표가 나오면 OK)
```

> Docker Desktop 화면의 **Containers / Images / Volumes** 탭에서 터미널로 한 작업 결과를 눈으로 확인할 수 있음.
> 처음엔 명령어 치고 → Docker Desktop에서 뭐가 생겼는지 같이 보면 이해가 빠름

---

# 5. 폴더와 파일 만들기 (한 줄씩 해설)

## 폴더 구조

```
oz_class/docker/              ← 이 폴더 이름(docker)이 Compose "프로젝트 이름"이 됨
├── Dockerfile                ← api 이미지를 만드는 레시피 (확장자 없음, 대문자 D)
├── docker-compose.yml        ← api + db 컨테이너 2개를 한 번에 정의
├── main.py                   ← FastAPI 서버 코드 (API 엔드포인트)
└── connection.py             ← DB 접속 설정 (SQLAlchemy)
```

### 각 파일이 누구에게 쓰이나

```
 Dockerfile ──────────────▶ docker가 읽음 (api 이미지 만들 때)
 docker-compose.yml ──────▶ docker compose가 읽음 (컨테이너 실행할 때)
 main.py, connection.py ──▶ 이미지 안으로 복사됨 → 컨테이너 안의 Python이 실행
```

> **이름 규칙이 중요한 이유** : 프로젝트 폴더 이름이 `docker`라서
> - 이미지 이름 → `docker-api`
> - 컨테이너 이름 → `docker-api-1`, `docker-db-1`
> - 네트워크 이름 → `docker_default`
>
> 즉 `{폴더이름}-{서비스이름}-{번호}` 규칙. 폴더 이름이 `myproject`였다면 `myproject-db-1`이 됨

---

## ① Dockerfile

```dockerfile
# Linux + Python3.13
FROM python:3.13-slim
WORKDIR /app
COPY . .
RUN pip install "fastapi[standard]" sqlalchemy pymysql cryptography

# 컨테이너 실행(docker run)에 사용할 명령어
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
```

| 줄 | 의미 | 비유 (새 컴퓨터 세팅) |
|---|---|---|
| `FROM python:3.13-slim` | **시작점 이미지**. 리눅스(Debian) + Python 3.13이 깔린 이미지에서 출발. `slim` = 불필요한 것 뺀 가벼운 버전 | 파이썬이 깔린 새 컴퓨터를 받음 |
| `WORKDIR /app` | 컨테이너 안에서 작업할 폴더를 `/app`으로 지정 (없으면 만들고 `cd /app`) | 바탕화면에 `app` 폴더 만들고 들어감 |
| `COPY . .` | **내 컴퓨터의 현재 폴더(`.`)** 전체를 → **컨테이너의 현재 폴더(`.` = `/app`)** 로 복사 | USB로 내 코드 복사 |
| `RUN pip install ...` | 이미지를 **만드는 도중에** 명령 실행. 패키지 설치 결과가 이미지에 저장됨 | 필요한 프로그램 설치 |
| `CMD [...]` | 컨테이너가 **시작될 때** 실행할 기본 명령어 | 전원 켜면 자동 실행되는 프로그램 |

### 패키지 설명
- `fastapi[standard]` : FastAPI + uvicorn(서버 실행기) 등 기본 세트
- `sqlalchemy` : Python에서 DB를 다루는 라이브러리
- `pymysql` : Python ↔ MySQL을 실제로 연결해주는 드라이버
- `cryptography` : MySQL 8.0의 기본 로그인 암호화 방식에 필요

### CMD 해석
```
uvicorn  main:app  --host 0.0.0.0  --port 8000  --reload
   │        │            │              │           └ 코드가 바뀌면 서버 자동 재시작
   │        │            │              └ 8000번 포트로 서버 열기 (compose의 9000:8000 오른쪽과 일치해야 함)
   │        │            └ 모든 곳에서 오는 요청 받기 ⭐
   │        └ main.py 파일 안의 app 변수(FastAPI 객체)를 실행
   └ 서버 실행 프로그램
```

> ⭐ **왜 `--host 0.0.0.0` 인가?**
> 컨테이너 안에서 `127.0.0.1`(localhost)은 **컨테이너 자기 자신**만 뜻함.
> 포트 매핑으로 들어오는 요청은 "밖"에서 오는 것이라 `127.0.0.1`로 열면 못 받음 → `0.0.0.0`(모든 주소)으로 열어야 브라우저 요청이 들어옴

### ⭐ RUN vs CMD (가장 많이 헷갈림)

```
            docker build (이미지 만들 때)            docker run / compose up (컨테이너 시작할 때)
 ─────────────────────────────────────────────┬──────────────────────────────────────────
  FROM → WORKDIR → COPY → RUN (여기서 실행)       │   CMD (여기서 실행)
  결과가 이미지에 "저장"됨                         │   서버처럼 계속 돌아가는 프로그램
  예) pip install                              │   예) uvicorn 서버 실행
```

- `RUN` : 이미지 만들 때 **한 번** 실행하고 끝 (설치 작업)
- `CMD` : 컨테이너 켤 때마다 실행 (서버 실행). 이 프로그램이 끝나면 컨테이너도 꺼짐

---

## ② docker-compose.yml

```yaml
# Docker Compose: 여러개의 컨테이너를 관리하는 기술
services:
  api:
    build: .            # 같은 폴더(.)의 Dockerfile로 이미지를 직접 만들어라
    ports:
      - 9000:8000       # 내 컴퓨터 9000 → 컨테이너 8000
  db:
    image: mysql:8.0    # Docker Hub의 mysql:8.0 이미지를 그대로 써라
    environment:
      MYSQL_ROOT_PASSWORD: password   # root 계정 비밀번호
      MYSQL_DATABASE: fastapi         # 처음 시작할 때 fastapi 라는 DB를 만들어라
```

### 구조 해석

```
services:                 ← "컨테이너 목록"
├── api                   ← 서비스 이름 (= 네트워크에서 주소로 쓰임)
│   ├── build: .          ← 이미지를 Dockerfile로 "직접 만든다"
│   └── ports: 9000:8000  ← 밖에서 접근 가능하게 문 열기
└── db                    ← 서비스 이름 (connection.py의 @db 가 바로 이것!)
    ├── image: mysql:8.0  ← 이미지를 "남이 만든 것 그대로 쓴다"
    └── environment       ← 컨테이너에 넘겨줄 환경변수 (설정값)
```

| 항목 | 의미 |
|---|---|
| `services` | 실행할 컨테이너들의 목록 |
| `build: .` | `.`(현재 폴더)에 있는 **Dockerfile로 이미지를 빌드**해서 사용 |
| `image: mysql:8.0` | 이미지를 빌드하지 않고 **Docker Hub에서 받아서** 사용 |
| `ports` | `호스트:컨테이너` 포트 연결 |
| `environment` | 컨테이너 안에 전달할 **환경변수**. mysql 이미지는 이 값을 읽어서 초기 설정을 함 |

### `build` vs `image`
- `api`는 **내 코드**가 들어가야 하니까 → Dockerfile로 직접 만듦 (`build`)
- `db`는 MySQL 그 자체면 충분하니까 → 공식 이미지 그대로 (`image`)

### environment가 하는 일
- `MYSQL_ROOT_PASSWORD: password` → 나중에 `mysql -u root -p` 할 때 입력하는 비밀번호가 바로 이것 (`password`)
- `MYSQL_DATABASE: fastapi` → `show databases;` 했을 때 **`fastapi`가 이미 있었던 이유**가 이것

### 왜 db에는 ports가 없나?
- api 컨테이너는 **같은 네트워크 안**에서 `db:3306`으로 접근하니까 포트를 열 필요가 없음
- 대신 내 컴퓨터(Windows)에서 DBeaver, MySQL Workbench 같은 툴로 직접 접속은 불가
- 필요하면 `ports: - 3307:3306` 추가 (내 컴퓨터에 MySQL이 이미 깔려 3306을 쓰고 있을 수 있어서 보통 3307 등 다른 번호 사용)

---

## ③ connection.py

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "mysql+pymysql://root:password@db:3306/fastapi"
│                                             ▲ 서비스 이름이 곧 주소│
engine = create_engine(DATABASE_URL, echo=True)   # echo=True : 실행되는 SQL을 로그로 출력

SessionFactory = sessionmaker(
    bind=engine,
    autocommit=False,
    autoflush=False,
    expire_on_commit=False,
)
```

### ⭐ DATABASE_URL 해부 — docker-compose.yml과 연결됨

```
mysql+pymysql :// root : password @ db : 3306 / fastapi
  │      │        │       │        │     │       │
  │      │        │       │        │     │       └ DB 이름   ← compose의 MYSQL_DATABASE
  │      │        │       │        │     └ 포트      ← MySQL 기본 포트
  │      │        │       │        └ 호스트(주소)    ← compose의 서비스 이름 "db" ⭐
  │      │        │       └ 비밀번호                ← compose의 MYSQL_ROOT_PASSWORD
  │      │        └ 사용자                          ← root
  │      └ 드라이버 (pymysql)
  └ DB 종류 (mysql)
```

> ⭐ **왜 `localhost`가 아니라 `db`인가?**
> 이 코드는 **api 컨테이너 안**에서 실행됨. 거기서 `localhost`는 api 컨테이너 자기 자신 → MySQL이 없음.
> MySQL은 **옆 컨테이너(db)** 에 있으므로, Compose 네트워크가 제공하는 이름 `db`로 찾아가야 함.
>
> ```
> api 컨테이너에서 "localhost:3306" → api 컨테이너 자신 (MySQL 없음) ❌
> api 컨테이너에서 "db:3306"        → db 컨테이너의 MySQL ✅
> ```

- `SessionFactory` : DB와 대화하는 세션(연결 통로)을 찍어내는 공장. `main.py`에서 `with SessionFactory() as session:`으로 사용

---

## ④ main.py

```python
from fastapi import FastAPI
from sqlalchemy import text

from connection import SessionFactory

app = FastAPI(title="Docker 서버")       # CMD의 main:app 에서 app이 이것

@app.get("/users", summary="전체 사용자 조회 API")
def get_users_handler():
    with SessionFactory() as session:              # DB 연결 열기 (끝나면 자동으로 닫힘)
        stmt = text("SELECT * FROM user;")          # 실행할 SQL
        rows = session.execute(stmt).fetchall()     # db 컨테이너에 SQL 보내고 결과 받기

        users = [row._asdict() for row in rows]     # 각 행을 dict로 변환 → list[dict]
    return users                                    # FastAPI가 JSON으로 변환해서 응답
```

- `GET /users` 요청이 오면 → `user` 테이블 전체를 조회해서 JSON으로 돌려줌
- **`user` 테이블은 이 코드가 만들지 않음** → 그래서 Step 5에서 MySQL에 직접 들어가 테이블을 만들어야 했던 것

---

# 6. 실행 순서 따라가기 (Step by Step)

## 전체 흐름 한눈에

```
 Step 0  Docker Desktop 실행                    (엔진 켜기)
    │
 Step 1  파일 4개 작성                           (레시피와 설정 준비)
    │
 Step 2  docker compose up                     (이미지 빌드 + 컨테이너 2개 실행)
    │      → 터미널이 로그로 묶임 → Ctrl+C로 종료
 Step 3  docker compose up -d                  (백그라운드로 다시 실행)
    │
 Step 4  docker ps                             (잘 돌고 있는지 확인)
    │
 Step 5  docker exec -it docker-db-1 bash      (db 컨테이너 안으로 들어가서)
    │      mysql -u root -p                    (MySQL 접속 →)
    │      CREATE TABLE / INSERT               (테이블과 데이터 만들기)
    │
 Step 6  브라우저 localhost:9000/users           (API로 데이터 조회 확인)
    │
 Step 7  코드 수정 → docker compose up -d --build (이미지 다시 만들어서 반영)
    │
 Step 8  docker compose down → up -d            (전부 삭제 후 재생성 → 데이터 사라짐 확인)
```

---

## Step 2. `docker compose up`

```bash
cd C:\Users\user\Desktop\WEB\oz_class\docker   # docker-compose.yml이 있는 폴더에서 실행해야 함
docker compose up
```

### 이 한 줄이 내부에서 하는 일

```
docker compose up
  │
  ├─ 1. docker-compose.yml 읽기
  │
  ├─ 2. 이미지 준비
  │     ├─ db  : mysql:8.0 이 내 컴퓨터에 있나? → 없으면 Docker Hub에서 다운로드(pull)
  │     └─ api : docker-api 이미지가 있나? → 없으면 Dockerfile로 빌드(build)
  │
  ├─ 3. 네트워크 생성 : docker_default
  │
  ├─ 4. 컨테이너 생성 : docker-db-1, docker-api-1
  │
  └─ 5. 컨테이너 시작 + 두 컨테이너의 로그를 터미널에 계속 출력
```

- 실행하면 터미널에 `db-1 | ...`, `api-1 | ...` 로그가 계속 흐르고 **터미널이 묶여서** 다른 명령을 못 침
- `Ctrl + C` 누르면 컨테이너가 **멈춤(stop)** (삭제는 아님)

> **첫 실행은 오래 걸림** : 이미지 다운로드 + 빌드 때문. 두 번째부터는 이미 있는 이미지를 재사용해서 빠름

---

## Step 3. `docker compose up -d`

```bash
docker compose up -d
```

- `-d` = **detached** (분리) → 컨테이너를 **백그라운드**에서 실행하고 터미널은 바로 돌려줌
- 서버는 계속 돌고 있고, 나는 같은 터미널에서 다른 명령을 칠 수 있음

```
 docker compose up      → 터미널 ═══ 로그 계속 출력 ═══ (Ctrl+C 전까지 묶임)
 docker compose up -d   → 터미널 ─ 실행 완료 메시지 ─ 바로 다음 명령 입력 가능
```

### 출력 읽는 법

```
[+] up 2/2
 ✔ Container docker-api-1 Running     ← 이미 실행 중이라 아무것도 안 함
 ✔ Container docker-db-1  Running
```

| 상태 | 의미 |
|---|---|
| `Created` | 새로 만들어짐 |
| `Started` | (멈춰 있던 or 새로 만든 걸) 시작함 |
| `Running` | 이미 실행 중이라 그대로 둠 |
| `Recreated` | 설정/이미지가 바뀌어서 지우고 새로 만듦 |
| `Removed` | 삭제됨 (down 할 때) |

> 로그를 보고 싶으면 : `docker compose logs -f` (`-f` = follow, 계속 따라가기 / `Ctrl+C`로 보기만 종료, 컨테이너는 계속 실행)

---

## Step 4. `docker ps`

```bash
docker ps        # 실행 중인 컨테이너 목록
docker ps -a     # 멈춘 것까지 전부 (-a = all)
```

### 출력 예시와 읽는 법

```
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                    NAMES
a1b2c3d4e5f6   docker-api   "uvicorn main:app --…"   2 minutes ago   Up 2 minutes   0.0.0.0:9000->8000/tcp   docker-api-1
f6e5d4c3b2a1   mysql:8.0    "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   3306/tcp, 33060/tcp      docker-db-1
```

| 열 | 의미 |
|---|---|
| CONTAINER ID | 컨테이너 고유 번호 (앞 몇 글자만 써도 명령어에 사용 가능) |
| IMAGE | 어떤 이미지로 만들었는지 |
| COMMAND | 시작할 때 실행된 명령 (Dockerfile의 CMD) |
| STATUS | `Up` = 실행 중 / `Exited` = 멈춤 |
| PORTS | `0.0.0.0:9000->8000` = 포트 매핑됨 / `3306/tcp` = 컨테이너 안에서만 열림(밖에서 접근 불가) |
| NAMES | 컨테이너 이름 → **`docker exec` 등에서 이 이름을 사용** |

---

## Step 5. db 컨테이너 안으로 들어가서 테이블 만들기

### 왜 들어가야 하나?
- `main.py`는 `user` 테이블을 **조회만** 함. 테이블이 없으면 에러
- MySQL은 db 컨테이너 **안**에서 돌고 있고, 포트도 밖으로 안 열려 있음 → 컨테이너 안으로 들어가서 직접 SQL을 쳐야 함

### ⭐ 지금 내가 "어디"에 있는지 — 3단계 층

```
┌─ 1층 : 내 컴퓨터 PowerShell ───────────────────────────── 프롬프트  C:\...\docker>
│    docker exec -it docker-db-1 bash     ↓ 들어가기
│  ┌─ 2층 : db 컨테이너 안의 리눅스 bash ────────────────── 프롬프트  bash-5.1#
│  │    mysql -u root -p                  ↓ 들어가기
│  │  ┌─ 3층 : MySQL 프로그램 ───────────────────────────── 프롬프트  mysql>
│  │  │    SHOW DATABASES; CREATE TABLE ...
│  │  └─ exit 또는 Ctrl+D                  ↑ 나오기 (화면에 "Bye")
│  └─ exit 또는 Ctrl+D                     ↑ 나오기
└─ 다시 PowerShell
```

> **프롬프트(입력줄 앞 글자)를 보면 지금 몇 층인지 알 수 있음**
> `C:\...>` 내 컴퓨터 / `bash-5.1#` 컨테이너 안 / `mysql>` MySQL 안
> 층마다 쓸 수 있는 명령어가 다름 (`mysql>`에서 `docker ps` 치면 안 됨!)

### 5-1. 컨테이너 안으로 들어가기

```bash
docker exec -it docker-db-1 bash
```

```
docker exec   -it   docker-db-1   bash
    │          │         │          └ 실행할 명령어 : bash(리눅스 터미널)
    │          │         └ 어느 컨테이너에서? (docker ps의 NAMES)
    │          └ -i(입력 가능) + -t(터미널 화면) = 대화형으로
    └ "실행 중인" 컨테이너 안에서 명령 하나 실행해줘
```

- 결과 : 프롬프트가 `bash-5.1#`로 바뀜 → db 컨테이너 안의 리눅스에 들어온 것
- 같은 동작 (compose 버전, 서비스 이름 사용) : `docker compose exec db bash`

### 5-2. MySQL 접속

```bash
mysql -u root -p
```

- `-u root` : root 사용자로
- `-p` : 비밀번호를 입력하겠다 → `Enter password:` 에 `password` 입력 (compose의 `MYSQL_ROOT_PASSWORD`)
- ⚠️ 비밀번호는 **입력해도 화면에 아무것도 안 보임** (정상). 그냥 치고 Enter

### 5-3. SQL 실행

```sql
SHOW DATABASES;          -- DB 목록 보기 → fastapi 가 보임 (compose가 만들어준 것)
USE fastapi;             -- fastapi DB를 사용하겠다 (이후 명령은 이 DB 안에서)

CREATE TABLE user (                       -- user 테이블 만들기
  id INT PRIMARY KEY AUTO_INCREMENT,      -- id : 정수, 기본키, 자동으로 1,2,3... 증가
  name VARCHAR(50),                       -- name : 최대 50글자 문자열
  age INT                                 -- age : 정수
);

INSERT INTO user (name, age) VALUES ('alex', 20), ('bob', 30), ('chris', 40);   -- 데이터 3개 넣기

SELECT * FROM user;      -- 전체 조회
```

| SQL | 의미 |
|---|---|
| `SHOW DATABASES;` | DB 목록 |
| `USE 이름;` | 사용할 DB 선택 |
| `SHOW TABLES;` | 현재 DB의 테이블 목록 |
| `DESC 테이블;` | 테이블 구조(컬럼) 보기 |
| `CREATE TABLE` | 테이블 생성 |
| `INSERT INTO` | 데이터 추가 |
| `SELECT * FROM` | 데이터 조회 |
| `DROP TABLE 테이블;` | 테이블 삭제 |

> **MySQL 입력 팁**
> - SQL은 **`;`로 끝나야 실행**됨. `;` 없이 Enter 치면 `->` 가 나오며 다음 줄을 기다림 (여러 줄 입력 가능한 이유)
> - `clear`(= `\c`)는 **화면 지우기가 아니라** 지금 입력 중이던 SQL을 취소하는 것
> - 나가기 : `exit` 또는 `Ctrl + D` (`^DBye` = Ctrl+D 누르고 Bye 출력된 것)

### 5-4. 빠져나오기

```
mysql> exit          → bash-5.1# 로 돌아옴 (MySQL 종료)
bash-5.1# exit       → PowerShell 로 돌아옴 (컨테이너에서 나옴)
```

- `exit`로 나와도 **컨테이너는 계속 실행 중** (exec로 띄운 bash만 종료된 것, MySQL 서버는 그대로)

---

## Step 6. 브라우저에서 확인

| 주소 | 내용 |
|---|---|
| `http://localhost:9000/users` | 방금 넣은 데이터가 JSON으로 나옴 |
| `http://localhost:9000/docs` | FastAPI 자동 문서 (Swagger UI) — 여기서 API 직접 실행해볼 수 있음 |

```json
[
  {"id": 1, "name": "alex", "age": 20},
  {"id": 2, "name": "bob", "age": 30},
  {"id": 3, "name": "chris", "age": 40}
]
```

> 왜 8000이 아니라 **9000**? → compose의 `9000:8000` 때문. 내 컴퓨터 입장에선 9000번 문으로 들어가야 함

---

## Step 7. 코드 수정 → `docker compose up -d --build`

### 왜 `--build`가 필요한가?
- Dockerfile의 `COPY . .`는 **빌드하는 순간의 코드를 이미지 안에 복사(스냅샷)** 함
- 그 후 VS Code에서 `main.py`를 고쳐도 **이미지 안의 코드는 옛날 그대로**
- `docker compose up -d`만 하면 이미 있는 이미지를 재사용 → 변경 반영 안 됨
- `--build`를 붙이면 Dockerfile로 **이미지를 다시 만들고** 컨테이너를 새로 띄움

```
 VS Code의 main.py (수정함 ✏️)
        │
        │  docker compose up -d          → 기존 이미지 재사용 → 옛날 코드로 실행 ❌
        │  docker compose up -d --build  → 다시 COPY → 새 이미지 → 새 코드로 실행 ✅
        ▼
 이미지 안의 /app/main.py
```

### 빌드 로그 읽는 법 (실제 출력)

```
 => [1/4] FROM docker.io/library/python:3.13-slim        ← Dockerfile 1번째 단계
 => CACHED [2/4] WORKDIR /app                             ← CACHED : 바뀐 게 없어서 이전 결과 재사용
 => [3/4] COPY . .                               0.1s    ← 코드가 바뀌어서 다시 복사
 => [4/4] RUN pip install "fastapi[standard]" ...  28.0s  ← 그 다음 단계라 패키지도 다시 설치 (오래 걸림)
 => exporting to image                                    ← 이미지로 저장
 => => naming to docker.io/library/docker-api:latest      ← 이미지 이름 : docker-api
...
 ✔ Image docker-api       Built                           ← 이미지 새로 만듦
 ✔ Container docker-db-1  Running                         ← db는 바뀐 게 없으니 그대로
 ✔ Container docker-api-1 Started                         ← api는 새 이미지로 다시 시작
```

- `[1/4]` ~ `[4/4]` = Dockerfile의 명령어 4단계(FROM, WORKDIR, COPY, RUN)
- **레이어 캐시** : Docker는 각 단계 결과를 저장해두고, 바뀐 게 없으면 재사용(`CACHED`)
- 단, **한 단계가 바뀌면 그 아래 단계는 전부 다시 실행** → 코드 한 줄 고쳤는데 `pip install`을 28초 동안 다시 함

> 💡 **개선 팁** : 자주 바뀌는 것(코드)을 아래로, 잘 안 바뀌는 것(패키지)을 위로
> ```dockerfile
> FROM python:3.13-slim
> WORKDIR /app
> COPY requirements.txt .                 # 패키지 목록만 먼저 복사
> RUN pip install -r requirements.txt     # requirements.txt가 안 바뀌면 CACHED
> COPY . .                                # 코드는 마지막에 복사
> CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
> ```
> → 코드만 고치면 `pip install`은 CACHED, 빌드가 몇 초로 줄어듦

> 참고 : 지금 CMD에 `--reload`가 있지만 코드가 이미지에 "복사"된 상태라 VS Code에서 고쳐도 컨테이너 안 파일은 안 바뀜 → 사실상 동작 안 함. 고치면 바로 반영되게 하려면 [10장](#10-다음-단계--개선된-docker-composeyml)의 볼륨(bind mount) 사용

---

## Step 8. `docker compose down` → `up` : 데이터가 사라진다?!

```bash
docker compose down
```
```
 ✔ Container docker-api-1 Removed     ← 컨테이너 삭제
 ✔ Container docker-db-1  Removed     ← 컨테이너 삭제 (안의 MySQL 데이터도 같이!)
 ✔ Network docker_default Removed     ← 네트워크 삭제
```

```bash
docker compose up -d
```
```
 ✔ Network docker_default Created     ← 네트워크 새로 생성
 ✔ Container docker-db-1  Started     ← 컨테이너 "새로" 생성 후 시작
 ✔ Container docker-api-1 Started
```

### ⚠️ 이 상태에서 `localhost:9000/users`에 접속하면?
- **에러** 발생 (`Table 'fastapi.user' doesn't exist`)
- 이유 : 볼륨이 없어서 MySQL 데이터가 **db 컨테이너 안에만** 있었음 → `down`으로 컨테이너가 삭제되면서 user 테이블도 같이 삭제
- `fastapi` DB는 다시 있음 → 새 컨테이너가 시작될 때 `MYSQL_DATABASE` 설정으로 **빈 DB를 새로 만들었기 때문**

```
down 전                        down 후 up                  볼륨을 썼다면
┌─ docker-db-1 ───┐            ┌─ docker-db-1 (새것) ─┐    ┌─ docker-db-1 (새것) ─┐
│ fastapi         │            │ fastapi (빈 DB)      │    │                      │
│  └ user (3행)   │  down/up   │  (user 없음) ❌      │    └───────────┬──────────┘
└─────────────────┘ ─────────▶ └──────────────────────┘                │ 연결
                                                           ┌─ 볼륨 db_data ───────┐
                                                           │ fastapi              │
                                                           │  └ user (3행) ✅     │
                                                           └──────────────────────┘
                                                           컨테이너 밖에 저장 → 안 사라짐
```

### `stop` vs `down`
- `docker compose stop` : 컨테이너를 **멈추기만** 함 → `start`하면 데이터 그대로
- `docker compose down` : 컨테이너를 **삭제** → 볼륨 없으면 데이터 사라짐
- 해결책 : 볼륨 사용 ([10장](#10-다음-단계--개선된-docker-composeyml))

---

# 7. 헷갈리는 것 비교 정리

## 컨테이너의 일생 (상태 변화)

```
  docker run / compose up
  [이미지] ───────────────────────────▶ [실행 중 Up] ◀────┐
                                              │           │ docker start / compose start
                                   docker stop│           │ (데이터 유지)
                         compose stop / Ctrl+C│           │
                                              ▼           │
                                        [멈춤 Exited] ────┘
                                              │
                               docker rm      │
                               compose down   │ (컨테이너 + 안의 데이터 삭제)
                                              ▼
                                          [삭제됨]
  ※ compose down은 실행 중이어도 멈추고 바로 삭제
```

## 비교표

| 헷갈리는 쌍 | 차이 |
|---|---|
| **이미지 vs 컨테이너** | 이미지 = 틀(읽기 전용) / 컨테이너 = 틀로 찍어낸 실행 중인 것 |
| **`build` vs `image`** (compose) | build = Dockerfile로 직접 만듦 / image = Docker Hub 것 그대로 사용 |
| **`RUN` vs `CMD`** (Dockerfile) | RUN = 이미지 만들 때 실행(설치) / CMD = 컨테이너 시작할 때 실행(서버) |
| **`docker run` vs `docker exec`** | run = 이미지로 **새 컨테이너**를 만들어 실행 / exec = **이미 실행 중인** 컨테이너 안에서 명령 실행 |
| **`up` vs `up -d`** | up = 로그 보면서 터미널 점유 / -d = 백그라운드 실행 |
| **`up` vs `up --build`** | up = 이미지 있으면 재사용 / --build = 이미지 다시 빌드 (코드 변경 반영) |
| **`stop` vs `down`** | stop = 멈춤(데이터 유지) / down = 삭제(볼륨 없으면 데이터 사라짐) |
| **`docker` vs `docker compose`** | docker = 컨테이너 하나씩 / docker compose = yml에 적힌 여러 개를 한 번에 |
| **컨테이너 이름 vs 서비스 이름** | `docker exec docker-db-1` (컨테이너 이름) / `docker compose exec db` (서비스 이름) |
| **`localhost` (어디서 쓰냐에 따라)** | 내 컴퓨터에서 = Windows / 컨테이너 안에서 = 그 컨테이너 자신 |

## `docker run -it python:3.13` 은 뭐였나?

```bash
docker run -it python:3.13
```

- Compose 없이 docker만으로 컨테이너 하나를 실행해본 것
- `python:3.13` 이미지로 **새 컨테이너를 만들고** 그 안의 Python 대화형 모드(`>>>`)로 바로 들어감
- 내 컴퓨터에 Python이 없어도 Python을 쓸 수 있다는 걸 보여주는 예시
- `exit()` 하면 Python이 끝나고 → **컨테이너도 꺼짐** (컨테이너의 주 프로그램이 Python이었기 때문)
  - 반면 `docker exec`로 들어간 bash는 `exit`해도 컨테이너가 안 꺼짐 (주 프로그램은 MySQL이고 bash는 손님일 뿐)

```bash
docker run -it --rm python:3.13        # --rm : 끝나면 컨테이너 자동 삭제 (테스트용으로 좋음)
docker run -it python:3.13 bash        # Python 대신 bash로 들어가기 (CMD 덮어쓰기)
```

> `docker run`을 칠 때마다 **새 컨테이너가 계속 생김** → `docker ps -a`로 보면 멈춘 컨테이너가 쌓여 있음 → `--rm` 쓰거나 `docker rm`으로 정리

---

# 8. 명령어 사전

> 📍 **어디서 치는지** 꼭 확인 : 아래 docker 명령어는 전부 **내 컴퓨터 터미널(PowerShell)** 에서.
> `docker compose ...` 는 **`docker-compose.yml`이 있는 폴더**에서 쳐야 함

## docker compose (여러 컨테이너, 실습에서 주로 사용)

| 명령어 | 설명 |
|---|---|
| `docker compose up` | 이미지 준비 + 컨테이너 생성/시작 (로그 출력, 터미널 점유) |
| `docker compose up -d` | 위와 같지만 백그라운드 실행 |
| `docker compose up -d --build` | 이미지를 다시 빌드하고 실행 (코드/Dockerfile 변경 후) |
| `docker compose down` | 컨테이너 + 네트워크 **삭제** |
| `docker compose down -v` | 볼륨까지 삭제 (DB 데이터 완전 초기화) ⚠️ |
| `docker compose ps` | 이 프로젝트의 컨테이너 상태 |
| `docker compose logs` | 전체 로그 보기 |
| `docker compose logs -f api` | api 서비스 로그를 실시간으로 계속 보기 |
| `docker compose stop` | 멈추기 (삭제 X) |
| `docker compose start` | 멈춘 것 다시 시작 |
| `docker compose restart api` | api 서비스 재시작 |
| `docker compose exec db bash` | db 서비스 컨테이너 안에 bash로 들어가기 |
| `docker compose build` | 이미지만 빌드 (실행 X) |

## docker (컨테이너/이미지 개별 관리)

### 컨테이너
| 명령어 | 설명 |
|---|---|
| `docker ps` | 실행 중인 컨테이너 목록 |
| `docker ps -a` | 멈춘 것 포함 전체 목록 |
| `docker run 이미지` | 이미지로 새 컨테이너 생성 + 실행 |
| `docker exec -it 컨테이너 bash` | 실행 중인 컨테이너 안으로 들어가기 |
| `docker logs 컨테이너` | 컨테이너 로그 (`-f` 붙이면 실시간) |
| `docker stop 컨테이너` | 멈추기 |
| `docker start 컨테이너` | 다시 시작 |
| `docker rm 컨테이너` | 삭제 (멈춘 상태여야 함, `-f`면 강제) |
| `docker container prune` | 멈춘 컨테이너 전부 삭제 |

### 이미지
| 명령어 | 설명 |
|---|---|
| `docker images` | 내 컴퓨터에 있는 이미지 목록 |
| `docker pull 이미지:태그` | Docker Hub에서 이미지 다운로드 |
| `docker build -t 이름 .` | 현재 폴더 Dockerfile로 이미지 빌드 (`-t` = 이름 붙이기) |
| `docker rmi 이미지` | 이미지 삭제 |
| `docker image prune` | 안 쓰는 이미지 정리 |

### 정리 / 기타
| 명령어 | 설명 |
|---|---|
| `docker --version` | 설치 확인 |
| `docker volume ls` | 볼륨 목록 |
| `docker network ls` | 네트워크 목록 (`docker_default` 확인 가능) |
| `docker system df` | Docker가 디스크를 얼마나 쓰는지 |
| `docker system prune` | 안 쓰는 컨테이너/네트워크/이미지 한 번에 정리 ⚠️ |

## `docker run` 자주 쓰는 옵션

| 옵션 | 의미 | 예시 |
|---|---|---|
| `-it` | 대화형 터미널 (입력 + 화면) | `docker run -it python:3.13` |
| `-d` | 백그라운드 실행 | `docker run -d mysql:8.0` |
| `-p 호스트:컨테이너` | 포트 매핑 | `-p 9000:8000` |
| `-e 키=값` | 환경변수 | `-e MYSQL_ROOT_PASSWORD=password` |
| `--name 이름` | 컨테이너 이름 지정 | `--name my-db` |
| `--rm` | 종료 시 자동 삭제 | `docker run -it --rm python:3.13` |
| `-v 호스트경로:컨테이너경로` | 볼륨/폴더 연결 | `-v ./data:/var/lib/mysql` |

> 💡 compose 파일은 결국 이 옵션들을 yml로 적어둔 것
> ```bash
> # db 서비스를 docker run 으로 직접 치면 이런 느낌
> docker run -d --name docker-db-1 -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=fastapi mysql:8.0
> ```
> → 이걸 매번 치기 귀찮고, 컨테이너가 여러 개면 더 복잡하니까 Compose를 쓰는 것

---

# 9. 자주 만나는 에러와 해결

| 증상 / 에러 메시지 | 원인 | 해결 |
|---|---|---|
| `error during connect ... dockerDesktopLinuxEngine` | Docker Desktop이 안 켜짐 | Docker Desktop 실행 후 Engine running 확인 |
| `no configuration file provided: not found` | yml이 없는 폴더에서 `docker compose` 실행 | `cd`로 `docker-compose.yml` 있는 폴더로 이동 |
| `port is already allocated` / `Bind for 0.0.0.0:9000 failed` | 내 컴퓨터에서 9000번을 이미 누가 사용 중 | 그 프로그램 종료 or `ports`를 `9001:8000`처럼 변경 |
| `No such container: docker-db-1` | 컨테이너 이름 틀림 or 실행 안 됨 | `docker ps`로 NAMES 확인 |
| `Can't connect to MySQL server on 'db'` | MySQL이 아직 켜지는 중 (첫 실행 시 초기화에 10~30초) | 잠시 후 다시 시도 / `depends_on` + `healthcheck` 사용 (10장) |
| `Table 'fastapi.user' doesn't exist` | `down` 후 데이터 사라짐 or 테이블 안 만듦 | Step 5 다시 / 볼륨 사용 |
| 코드를 고쳤는데 반영이 안 됨 | 이미지 안의 코드는 빌드 시점 그대로 | `docker compose up -d --build` |
| `Access denied for user 'root'` | 비밀번호 틀림 | compose의 `MYSQL_ROOT_PASSWORD` 값 확인 |
| 비밀번호 칠 때 아무것도 안 보임 | 정상 (보안상 숨김) | 그냥 치고 Enter |
| `mysql>`에서 명령이 실행 안 되고 `->` 만 나옴 | `;`를 안 붙임 | `;` 입력 후 Enter |
| api 컨테이너가 바로 `Exited` | 코드 에러로 uvicorn이 죽음 | `docker compose logs api`로 에러 확인 |

> 🔍 **문제가 생기면 일단 이 3개**
> ```bash
> docker ps -a                  # 컨테이너가 살아 있나? (Up / Exited)
> docker compose logs api       # api에 무슨 에러가 났나?
> docker compose logs db        # db에 무슨 에러가 났나?
> ```

---

# 10. 다음 단계 : 개선된 docker-compose.yml

> 실습 버전에서 불편했던 3가지를 해결한 버전
> 1. `down`하면 DB 데이터가 사라짐 → **named volume**
> 2. 코드 고칠 때마다 `--build` 해야 함 → **bind mount**
> 3. db가 준비되기 전에 api가 먼저 뜰 수 있음 → **depends_on + healthcheck**

```yaml
services:
  api:
    build: .
    ports:
      - 9000:8000
    volumes:
      - .:/app                      # ② 내 폴더(.)를 컨테이너 /app에 "연결" → 고치면 바로 반영(--reload 동작)
    depends_on:
      db:
        condition: service_healthy  # ③ db가 healthy 상태가 된 후에 api 시작

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: fastapi
    volumes:
      - db_data:/var/lib/mysql      # ① MySQL 데이터 폴더를 db_data 볼륨에 저장
    healthcheck:                    # ③ MySQL이 응답하는지 주기적으로 검사
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-ppassword"]
      interval: 5s
      timeout: 3s
      retries: 10

volumes:
  db_data:                          # ① 사용할 볼륨 선언
```

## COPY vs bind mount

```
COPY (Dockerfile)                            bind mount (volumes: - .:/app)
┌─ 내 폴더 ┐               ┌─ 이미지 ─┐      ┌─ 내 폴더 ┐             ┌─ 컨테이너 ┐
│ main.py  │ 빌드할 때 1번 │ main.py  │      │ main.py  │ 실시간 연결 │ /app     │
└──────────┘ ── 복사 ──▶   └──────────┘      └──────────┘ ◀════════▶  └──────────┘
수정해도 이미지는 그대로                     VS Code에서 저장하면
→ --build 필요                               → 컨테이너에도 즉시 반영
```

- 볼륨 종류
  - **named volume** (`db_data:/var/lib/mysql`) : Docker가 관리하는 저장공간. DB 데이터 보존용
  - **bind mount** (`.:/app`) : 내 컴퓨터의 실제 폴더를 그대로 연결. 개발 중 코드 실시간 반영용
- `docker compose down` 해도 볼륨은 남음 → 다시 `up` 하면 user 테이블 그대로 ✅
- 완전히 초기화하고 싶으면 `docker compose down -v`

> 💡 Windows에서 bind mount 사용 시 파일 변경 감지가 안 될 때가 있음 → api에 환경변수 추가
> ```yaml
>     environment:
>       WATCHFILES_FORCE_POLLING: "true"
> ```

## 테이블 자동 생성 (매번 exec로 들어가기 귀찮을 때)
- mysql 이미지는 `/docker-entrypoint-initdb.d/` 폴더의 `.sql` 파일을 **DB가 처음 만들어질 때** 자동 실행함

```sql
-- init.sql
CREATE TABLE user (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50),
  age INT
);
INSERT INTO user (name, age) VALUES ('alex', 20), ('bob', 30), ('chris', 40);
```

```yaml
  db:
    volumes:
      - db_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
```

- ⚠️ "처음"에만 실행 → 이미 `db_data` 볼륨에 데이터가 있으면 실행 안 됨 (다시 실행하려면 `down -v` 후 `up`)

## .dockerignore
- 빌드 로그에 `load .dockerignore`가 있었던 것 → `COPY . .` 할 때 **빼고 싶은 파일 목록**
- `.gitignore`와 같은 개념

```
.venv
__pycache__
.git
.vscode
```

---

# 11. 한 장 요약

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│  [개념]  Dockerfile ─build─▶ Image ─run─▶ Container      (레시피 → 틀 → 붕어빵)      │
│          docker-compose.yml = 컨테이너 여러 개를 한 번에 정의                        │
├──────────────────────────────────────────────────────────────────────────────────────┤
│  [파일]  Dockerfile         : FROM → WORKDIR → COPY → RUN(설치) → CMD(실행)          │
│          docker-compose.yml : services: api(build) + db(image, environment)          │
│          connection.py      : mysql+pymysql://root:password@db:3306/fastapi          │
│                                                             ▲ 서비스 이름이 곧 주소  │
├──────────────────────────────────────────────────────────────────────────────────────┤
│  [순서]  0. Docker Desktop 켜기                                                      │
│          1. docker compose up -d           실행 (백그라운드)                         │
│          2. docker ps                      확인                                      │
│          3. docker exec -it docker-db-1 bash → mysql -u root -p → SQL                │
│          4. localhost:9000/users           API 확인                                  │
│          5. 코드 수정 → docker compose up -d --build                                 │
│          6. docker compose down            정리 (볼륨 없으면 데이터 삭제!)           │
├──────────────────────────────────────────────────────────────────────────────────────┤
│  [기억]  포트 9000:8000 = 내컴퓨터:컨테이너                                          │
│          컨테이너 안의 localhost = 그 컨테이너 자신 → 옆 컨테이너는 서비스 이름으로  │
│          프롬프트로 위치 확인 : C:\> (PC) / bash-5.1# (컨테이너) / mysql> (MySQL)    │
│          막히면 : docker ps -a → docker compose logs                                 │
└──────────────────────────────────────────────────────────────────────────────────────┘
```
