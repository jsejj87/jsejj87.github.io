# 팀 프로젝트 Git 협업 흐름 정리 (Git Flow 기반)

## 0. 큰 그림
- 코드는 **세 군데**에 있다
  - 내 컴퓨터 브랜치 (VS Code에서 작업하는 곳)
  - GitHub 브랜치 (push하면 올라가는 곳)
  - develop (팀 전체 작업이 모이는 곳, PR로만 들어감)
- 흐름: **develop에서 내 브랜치 만들기 → 작업 → push → PR → 승인 → merge → 정리**
- main: 최종본 / develop: 개발 중인 합본 / feature/이름: 내 작업용

---

## 1. 작업 시작 (터미널)
```bash
git switch develop                 # develop으로 이동
git pull                           # GitHub develop 최신 내용 받아오기
git switch -c feature/브랜치이름     # develop을 바탕으로 내 브랜치 새로 만들고 이동
```
- `-c` = create (새로 만들기)
- 브랜치 이름: 영어 소문자, 띄어쓰기 대신 `-`, 무슨 작업인지 알 수 있게
- 새 브랜치는 **항상 최신 develop에서** 만든다 (그래야 팀원 작업이 포함된 상태로 시작)

## 2. 작업하고 확인하기
- VS Code에서 코드 수정 → `Ctrl + S` 저장
- 서버 켜기
```bash
python -m uvicorn app.main:app --reload     # 서버 켜기
python -m fastapi dev                       # 짧은 버전 (main.py 자동으로 찾음)
```
  - `app.main:app` = app 폴더의 main.py 안의 app을 실행
  - `--reload` = 저장하면 서버가 알아서 다시 켜짐
  - 끄기: `Ctrl + C`
- 브라우저 `http://127.0.0.1:8000/docs` → Swagger에서 `Try it out` → `Execute`로 테스트

## 3. 올리기 (터미널)
```bash
git status                                  # 바뀐 파일 확인 (내 파일만 있는지)
git add 파일경로                              # 올릴 파일 준비
git commit -m "feat: 작업 내용"               # 기록 남기기
git push -u origin feature/브랜치이름          # GitHub에 내 브랜치 올리기
```
- `add`는 파일 이름을 콕 집어서 (실수로 다른 파일 섞이지 않게)
- 커밋 메시지: `feat:`(기능 추가), `fix:`(수정), `docs:`(문서) + 한글 설명
- `-u` = 이 브랜치와 GitHub 연결을 기억. **처음 한 번만** 필요, 다음부터는 `git push`만
- PR을 만든 뒤 같은 브랜치에 또 push하면 **그 PR에 자동으로 추가**됨

## 4. PR 만들기 (GitHub)
1. 팀 레포 **Pull requests** 탭 → **New pull request**
2. **base: develop** / **compare: 내 브랜치** ← base가 main으로 돼 있으면 꼭 바꾸기
3. **Create pull request** → 제목, 설명 작성
4. 오른쪽 **Reviewers** 톱니바퀴 → 팀원 선택
5. **Create pull request**
- PR = "내 브랜치를 develop으로 가져가 주세요"라는 **요청서** (아직 합쳐진 게 아님)
- `Review required`, `Merging is blocked` = 에러 아님. 승인 1명 이상 필요하다는 뜻 (보호 규칙)

## 5. 팀원 PR 리뷰하기 (GitHub)
1. **Pull requests** 탭 → 팀원 PR 클릭
2. **Files changed** 탭에서 코드 확인
3. **Review changes** → **Approve** → **Submit review**
- 분홍(-) 지운 줄 / 연두(+) 추가한 줄 / 파란 줄은 안 바뀐 부분 숨김

## 6. merge 후 정리
- GitHub: PR 화면 아래 **Delete branch** 클릭
- 내 컴퓨터:
```bash
git switch develop                     # 지금 서 있는 브랜치는 못 지우니 먼저 이동
git pull                               # merge된 최신 develop 받기
git branch -d feature/브랜치이름          # 내 컴퓨터 브랜치 지우기
```
- GitHub와 내 컴퓨터 브랜치는 **따로**라서 양쪽 다 지워야 함

---

## 7. 상태 확인용 명령어
```bash
git status            # 지금 브랜치, 바뀐 파일, add 됐는지
git branch            # 내 컴퓨터 브랜치 목록 (* = 지금 위치)
git log --oneline     # 커밋 기록
git fetch             # GitHub 변경사항 확인만 (내 파일은 안 바뀜)
git remote -v         # 이 폴더가 연결된 GitHub 주소
```

## 8. 터미널 경로 옆 표시
- `[develop ≡]` : develop에 있고 GitHub와 똑같음
- `[main ↓1]` : GitHub보다 1개 뒤처짐 → `git pull` 필요
- `[feature/... +0 ~1 -0 !]` : 수정한 파일 1개, 아직 add 안 함

## 9. 헷갈렸던 것
- PR은 **브랜치 하나당 하나**
- merge 전이면 같은 브랜치에서 고치고 push → 기존 PR에 반영 (승인 다시 받아야 할 수 있음)
- 응답 코드: 삭제 성공을 204로 하면 **메시지 없이** `return`만