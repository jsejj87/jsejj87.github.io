# 프로젝트 진행 매뉴얼 (VS Code + 가상환경 + GitHub)

## 0. 기본 규칙

- **VS Code는 프로젝트 폴더 자체로 연다** (상위 폴더로 열지 않기)
- **프로젝트 1개 = 폴더 1개 = 가상환경(.venv) 1개**
- 터미널에서 뭘 치기 전에 항상 `(.venv)` 붙어 있는지 확인
- 터미널 열기: `Ctrl + ~` (숫자 1 왼쪽 키) / 기본 터미널은 PowerShell

---

## 1. 새 프로젝트 초기 세팅

### 1-1. 폴더 만들고 열기
- 폴더 만들기 → VS Code `파일 → 폴더 열기` → 그 폴더 선택

### 1-2. 가상환경 만들기 + 적용 (터미널)
```bash
python -m venv .venv          # 가상환경 만들기 (안 되면 py -m venv .venv)
.venv\Scripts\activate        # 적용 (Mac: source .venv/bin/activate)
```
- 터미널 앞에 `(.venv)` 붙으면 성공
- "스크립트를 실행할 수 없으므로" 에러 뜨면 아래 한 번 실행 후 다시 activate
```bash
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

### 1-3. VS Code에 가상환경 연결
- `Ctrl + Shift + P` → `Python: Select Interpreter` → `.venv` 들어간 항목 선택
- 목록에 없으면 `Enter interpreter path` → `.venv\Scripts\python.exe`
- 이걸 해야 새 터미널 열 때 자동으로 `(.venv)` 적용됨

### 1-4. .gitignore 만들기 (git 시작 전에!)
- 탐색기에서 새 파일 → `.gitignore`
```
# 가상환경
.venv/

# 비밀 정보
.env

# 파이썬 캐시
__pycache__/
*.pyc

# DB 파일, AI 모델 파일
*.db
models/

# 개인 설정
.vscode/
.DS_Store
```

### 1-5. 패키지 설치 (터미널, (.venv) 확인 후)
```bash
pip install "fastapi[standard]"       # FastAPI + 서버 실행 도구
```
- 필요한 것만 그때그때 추가 설치
```bash
pip install SQLAlchemy                # DB (ORM)
pip install aiosqlite greenlet        # 비동기 DB
pip install pydantic-settings         # .env에서 키 불러오기
pip install openai                    # OpenAI API
pip install llama-cpp-python          # Llama 모델 실행
```
- 이미 설치된 걸 또 치면 `Requirement already satisfied` 뜨고 넘어감 (문제없음)

### 1-6. API 키 쓸 때만: .env / .env.example
- `.env` (진짜 키, GitHub에 안 올라감)
```
OPENAI_API_KEY=sk-진짜키
```
- `.env.example` (키 이름만, GitHub에 올림)
```
OPENAI_API_KEY=
```
- `=` 앞뒤 띄어쓰기 X, 따옴표 X
- `.env`는 **서버를 실행하는 위치(프로젝트 폴더)** 에 둬야 읽힘

### 1-7. GitHub 연결 + 첫 업로드
1. GitHub에서 New repository → **README, .gitignore 체크 안 하고** 빈 레포로 만들기
2. 터미널
```bash
git status        # "not a git repository" 나와야 정상 (이미 저장소 안이면 init 하지 않기)
git init          # 이 폴더를 git 저장소로
git status        # .env, .venv 목록에 없는지 확인!
git add .
git commit -m "프로젝트 초기 세팅"
git branch -M main
git remote add origin https://github.com/아이디/레포이름.git
git push -u origin main     # 처음 한 번만 -u, 다음부턴 git push
```

### 1-8. 팀 레포 받아서 시작할 때 (1-1 ~ 1-7 대신)
```bash
git clone https://github.com/팀/레포이름.git    # 받기
cd 레포이름                                      # 받은 폴더로 이동
```
- VS Code로 그 폴더 다시 열기 → 1-2, 1-3 (가상환경) → 1-5 (설치)
- 레포에 `requirements.txt`가 있으면 `pip install -r requirements.txt`
- `.env.example` 복사해서 `.env` 만들고 내 키 넣기
- `.gitignore`는 이미 레포에 있음

---

## 2. 매일 작업 루틴

```bash
# 시작
(.venv) 확인
git pull                      # GitHub 최신 내용 받기

# 작업 → 확인 (3번)

# 끝
git status                    # 바뀐 파일 확인
git add .
git commit -m "작업 내용"
git push                      # GitHub에 올리기
```
- 팀에서 브랜치 쓰면: `git switch -c 브랜치명` → 작업 → `git push -u origin 브랜치명` → GitHub에서 Pull Request

---

## 3. 작업 중 확인 방법

### 서버 실행
```bash
fastapi dev main.py           # 서버 켜기 → http://127.0.0.1:8000
```
- 끄기: 터미널에서 `Ctrl + C`
- 서버 켠 터미널은 계속 사용 중 → 다른 명령은 터미널 `+`로 새 터미널

### Swagger로 API 테스트
- 브라우저 `http://127.0.0.1:8000/docs` → 엔드포인트 열기 → `Try it out` → `Execute`

### 스트리밍 확인 (Swagger에선 한 번에 보여서 curl로)
```bash
curl -N -X POST http://127.0.0.1:8000/주소 -H "Content-Type: application/json" -d '{"user_input": "질문"}'
```

### 파이썬 대화형 모드 (코드 조각 바로 실행)
```bash
python -i model.py            # 파일 실행 후 >>> 모드로 남음
```
```python
>>> from connection import engine
>>> Base.metadata.create_all(bind=engine)   # 예: 테이블 만들기
>>> exit()                                  # 나가기
```

### DB 확인
- DB Browser for SQLite 실행 → `데이터베이스 열기` → `.db` 파일 선택 → `데이터 보기`
- DB Browser에서 직접 수정했으면 `변경사항 쓰기` 눌러야 함 (안 누르면 서버에서 database is locked 에러)

---

## 4. 헷갈릴 때 확인

```bash
pwd                                             # 지금 터미널 위치
python -c "import sys; print(sys.executable)"   # 지금 쓰는 파이썬 (경로에 .venv 있어야 정상)
pip list                                        # 이 환경에 설치된 패키지
git status                                      # 바뀐 파일, 현재 브랜치
git remote -v                                   # 연결된 GitHub 주소
git log --oneline                               # 커밋 기록 (q 누르면 나가기)
cd 폴더이름 / cd ..                             # 폴더 들어가기 / 위로 나가기
```

| 증상 | 확인할 것 |
|---|---|
| 설치했는데 `No module named ...` | `(.venv)` 적용됐는지, 인터프리터 선택했는지 |
| `openai_api_key Field required` | `.env`가 실행 위치에 있는지 (`pwd`) |
| push 안 됨 | `git pull` 먼저 |
| `.env`가 git status에 보임 | `.gitignore`에 `.env` 있는지, 커밋 전에 멈추기 |

---

## 5. 프로젝트 전체 흐름 (요약)

### 큰 그림
데이터 불러오기 → EDA / 전처리 → 모델 학습 (코랩) → 모델 파일 저장 → FastAPI로 API 만들기 → 화면(HTML/CSS/JS)에서 API 호출 → GitHub에 정리

### 파일 나누기 예시
```
project/
├── .venv/  .env  .env.example  .gitignore  README.md
├── main.py          # API 엔드포인트 (서버 실행 파일)
├── config.py        # .env 불러오기
├── connection.py    # DB 연결
├── model.py         # DB 테이블 정의
├── models/          # AI 모델 파일 (.gguf, .pt 등)
└── frontend/        # index.html, style.css, script.js (화면)
```

### 진행 절차
1. 초기 세팅 (1번)
2. 학습한 모델 파일을 `models/`에 넣기
3. `model.py`, `connection.py` 작성 → `python -i model.py`로 테이블 만들기 → DB Browser로 확인
4. `main.py`에 API 작성 → `fastapi dev main.py` → `/docs`에서 테스트
5. 모델 연결 (lifespan) → `/docs`, curl로 테스트
6. 화면 만들기 (html/css/js) → `fetch`로 API 호출
   - 브라우저 콘솔에 CORS 에러 뜨면 FastAPI에 CORS 설정 필요
7. 중간중간 commit / push (2번 루틴)
8. README에 프로젝트 설명, 실행 방법 정리 → 마지막 push
