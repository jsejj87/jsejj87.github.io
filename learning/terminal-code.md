# 터미널 기본 명령어 모음 (Windows PowerShell 기준)

## 1. 터미널 기본 조작
| 키 / 명령 | 역할 |
|---|---|
| `↑` `↓` | 예전에 친 명령 다시 불러오기 |
| `Tab` | 파일·폴더 이름 자동완성 |
| `Ctrl + C` | 실행 중인 것 멈추기 (서버 끄기) |
| `cls` | 화면 지우기 |
| `` Ctrl + ` `` | VS Code 터미널 열기/닫기 |

---

## 2. 이동·조회
```powershell
pwd                  # 지금 내 위치(경로) 보기
ls                   # 지금 폴더의 파일·폴더 목록
cd 폴더이름            # 그 폴더로 들어가기
cd ..                # 한 칸 위 폴더로
cd ~                 # 내 사용자 폴더로
cd C:\Users\user\Desktop\폴더   # 경로로 바로 이동
code .               # 지금 폴더를 VS Code로 열기
```

## 3. 만들기·삭제·복사·이동
```powershell
mkdir 폴더이름                     # 폴더 만들기
ni 파일이름.py                     # 빈 파일 만들기 (New-Item)
rm 파일이름                        # 파일 삭제
rm -r 폴더이름                     # 폴더 통째로 삭제 (안에 든 것까지)
cp 원본 복사본                      # 복사
mv 원래이름 새이름                   # 이동 또는 이름 바꾸기
cat 파일이름                        # 파일 내용 터미널에 보기
```
- `rm`은 휴지통 안 거치고 바로 지워짐. 조심

---

## 4. 파이썬 실행
```powershell
python --version            # 파이썬 버전 확인
python 파일이름.py            # 파이썬 파일 실행
python -m 이름 ...           # 설치한 도구를 파이썬을 통해 실행 (PATH 안 잡혀도 됨)
```

## 5. 가상환경 (프로젝트 전용 공구함)
```powershell
python -m venv .venv          # .venv 가상환경 만들기 (프로젝트마다 처음 1번)
.venv\Scripts\activate        # 가상환경 켜기 → 앞에 (.venv) 붙으면 성공
deactivate                    # 가상환경 끄기
```
- 켜기에서 "스크립트 실행 안 됨" 에러가 나면 (처음 1번만)
```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```
- `.venv` 폴더는 GitHub에 안 올림 (`.gitignore`에 들어 있음)
- 작업 순서: 폴더 열기 → 가상환경 켜기 → 설치 → 실행

## 6. 패키지 설치
```powershell
pip install 이름                      # 설치
pip install "fastapi[standard]"      # FastAPI + uvicorn 등 한 번에
pip install requests                 # 외부 API 데이터 가져올 때
pip list                             # 설치된 목록 보기
pip uninstall 이름                    # 삭제
pip freeze > requirements.txt        # 지금 설치된 목록을 파일로 저장
pip install -r requirements.txt      # 그 파일 목록대로 한꺼번에 설치
```

## 7. uv (팀 프로젝트가 uv를 쓸 때)
```powershell
pip install uv                          # uv 설치
python -m uv sync                       # pyproject.toml / uv.lock 목록대로 .venv 만들고 설치
python -m uv sync --python 3.12         # 파이썬 버전 지정해서
python -m uv add 이름                    # 패키지 추가 (팀 목록에도 기록됨)
python -m uv run 명령                    # .venv 환경으로 명령 실행
```

---

## 8. FastAPI 서버 + Swagger
```powershell
python -m uvicorn fastapi.main:app --reload     # 서버 켜기
   -> python -m uvicorn  --reload 는 기본값. 
      그사이 fastapi는 폴더명/main은 파일명에서 .py제외 /app은 FastAPI()를 담은변수명

python -m fastapi dev                           # 짧은 버전 (main.py 자동으로 찾음)
```
- `app.main:app` = app 폴더 → main.py → 안의 app
- `--reload` = 코드 저장하면 자동 재시작
- 브라우저: `http://127.0.0.1:8000/docs` → Swagger
- 끄기: `Ctrl + C`

## 9. 터미널에서 API 데이터 바로 확인
```powershell
curl.exe http://127.0.0.1:8000/practice_api/users          # 내 서버 GET
curl.exe https://jsonplaceholder.typicode.com/posts/1       # 외부 API GET
```
- PowerShell에선 `curl` 말고 **`curl.exe`**로 써야 제대로 됨

---

## 10. Git 처음 설정 (컴퓨터당 1번)
```powershell
git config --global user.name "이름"
git config --global user.email "이메일"
git config --global --list          # 설정 확인
```

## 11. Git 저장소 시작
```powershell
git init                            # 지금 폴더를 git 저장소로 만들기
git clone 주소                       # GitHub 저장소를 내 컴퓨터로 복사
git remote -v                       # 연결된 GitHub 주소 보기
git remote add origin 주소           # GitHub 주소 연결
git remote set-url origin 새주소      # 연결 주소 바꾸기
```

## 12. Git 상태 조회
```powershell
git status                # 지금 브랜치, 바뀐 파일, add 됐는지
git log --oneline         # 커밋 기록 한 줄씩
git diff                  # 아직 add 안 한 수정 내용 보기
git branch                # 내 컴퓨터 브랜치 목록 (* = 지금 위치)
git branch -a             # GitHub 브랜치까지 전부
```

## 13. Git 저장·보내기
```powershell
git add 파일경로                        # 그 파일만 준비
git add .                              # 바뀐 파일 전부 준비 (띄어쓰기 필수)
git commit -m "feat: 내용"              # 기록 남기기
git push -u origin 브랜치타입/브랜치명    # 처음 올릴 때 (연결 기억)  
                  #  브랜치타입 : feature, fix, docs, hotfix, refactor, release ... 
                  #  브랜치명 : 작업내용으로 알아보기쉽게 정함
git push                               # 두 번째부터
```

## 14. Git 가져오기
```powershell
git fetch            # GitHub 변경사항 확인만 (내 파일은 그대로)
git pull             # GitHub 최신 내용 받아서 내 파일에 반영
```

## 15. Git 브랜치
```powershell
git switch 브랜치이름              # 브랜치 이동
git switch -c 새브랜치이름          # 새로 만들고 이동
git branch -m 새이름               # 지금 브랜치 이름 바꾸기
git branch -d 브랜치이름            # 브랜치 삭제 (다른 브랜치에 있는 상태에서)
git merge 브랜치이름                # 그 브랜치를 지금 브랜치에 합치기
```

## 16. Git 되돌리기
```powershell
git restore 파일                  # add 전 수정 내용 버리기 (작업 날아감, 조심)
git restore --staged 파일          # add 취소 (수정 내용은 남음)
git revert 커밋번호                # 그 커밋을 되돌리는 새 커밋 만들기 (push한 뒤에도 안전)
git stash                         # 작업 중인 걸 잠깐 치워두기 (브랜치 옮길 때)
git stash pop                     # 치워둔 거 다시 꺼내기
```

---

## 17. 팀 작업 한 바퀴 (외워둘 순서)
```powershell
git switch develop
git pull
git switch -c feature/작업이름
# 코드 작성 → 저장 → 서버 켜서 Swagger 테스트
git status
git add 파일경로
git commit -m "feat: 작업 내용"
git push -u origin feature/작업이름
# GitHub: PR (base: develop) → 리뷰 → merge → Delete branch
git switch develop
git pull
git branch -d feature/작업이름
```
