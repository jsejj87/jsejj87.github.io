# GitHub 팀 협업 ③ 브랜치 전략으로 작업하고 합치기 (팀장편 · 팀원편)

- **[웹]** : GitHub 사이트에서 하는 작업
- **[터미널]** : VS Code 터미널에서 하는 작업
- 팀 레포 세팅(clone, remote)이 끝난 상태에서 시작

---

## 1. 브랜치 3종 정리

main, develop, feature 모두 **branch**이고, 역할만 다르다.

| 브랜치 | 누가 쓰나 | 얼마나 있나 | 역할 |
|---|---|---|---|
| main | 팀 전체 | 끝까지 | 완성본. 직접 작업·push 금지 |
| develop | 팀 전체 | 끝까지 | 각자 만든 기능을 모으는 곳. 팀의 현재 진행 상태 |
| feature/기능명 | 나 혼자 | 기능 하나 만들 동안 | 개인 작업 공간. develop에 합쳐지면 삭제 |

```
main        ●────────────────────────────────────●   (최종본)
             \                                  /
develop       ●──────────●───────────●─────────●
               \        /  \        /
feature/A       ●──●──●     \      /
                             \    /
feature/B                     ●──●
```

- 점(●) = 커밋
- 아래로 갈라지는 선 = 브랜치 만들기
- 위로 합쳐지는 선 = PR 리뷰 후 merge
- feature/B는 feature/A가 합쳐진 **뒤의 develop**에서 시작 (앞 작업이 필요한 경우)

---

## 2. 팀장 - develop 만들기 (처음 한 번)

main을 기준으로 develop을 만들고 팀 레포에 올린다.

```bash
git switch main
git pull
git switch -c develop
git push -u origin develop
```

- [웹] 팀 레포 왼쪽 위 브랜치 목록에 develop이 보이면 성공
- 팀원들에게 "develop 만들었어요" 알리기

---

## 3. 팀원 - develop 처음 받아오기 (처음 한 번)

팀장이 만든 develop을 내 컴퓨터로 가져온다.

```bash
git fetch
git switch develop
```

- `git fetch` : 팀 레포에 새로 생긴 브랜치 정보를 받아옴
- 경로 옆에 `[develop]`이 보이면 성공

---

## 4. 전원 - 기능 하나 작업하는 사이클 (매번 반복)

### 4-1. [터미널] develop 최신 받기
남들이 합쳐둔 것까지 포함해서 시작해야 충돌이 적다.

```bash
git switch develop
git pull origin develop
```

### 4-2. [터미널] 내 브랜치 만들기

```bash
git switch -c feature/기능명
```

- 예) feature/db-models, feature/patient-api, feature/predict-api

### 4-3. [VS Code] 작업 → [터미널] 커밋
여러 번 커밋해도 된다.

```bash
git add .
git commit -m "feat: 기능 설명"
```

- 커밋 메시지 머리말 : feat(기능 추가) / fix(버그 수정) / docs(문서) / refactor(코드 정리) / chore(설정·기타)

### 4-4. [터미널] 내 브랜치를 팀 레포에 올리기
이 단계까지는 develop에 아무 변화 없음.

```bash
git push -u origin feature/기능명
```

### 4-5. [웹] PR 만들기
- 팀 레포에 들어가면 뜨는 **Compare & pull request** 클릭
- **base: develop / compare: feature/기능명** 확인 (base가 main으로 되어 있으면 develop으로 변경)
- 제목, 작업 내용 작성 → 오른쪽 **Reviewers**에서 리뷰할 팀원 지정
- **Create pull request**

### 4-6. [웹] 리뷰 (리뷰어 역할)
- PR 들어가서 **Files changed** 탭에서 바뀐 코드 확인
- 의견은 코멘트로 (존댓말·쿠션어)
- 괜찮으면 **Review changes → Approve**
- 고칠 게 있으면 작성자가 같은 브랜치에서 수정 후 다시 push → 같은 PR에 자동 반영

### 4-7. [웹] merge
- 승인 후 **Merge pull request → Confirm merge**
- 이때 처음으로 develop에 합쳐짐
- merge 후 뜨는 **Delete branch** 클릭

### 4-8. [터미널] 정리
내 컴퓨터의 develop도 최신으로 맞추고, 다 쓴 브랜치 삭제.

```bash
git switch develop
git pull origin develop
git branch -d feature/기능명
```

다음 기능은 다시 4-1부터.

---

## 5. 작업 순서가 있는 경우

앞 작업이 develop에 merge된 뒤에 다음 사람이 시작한다.

```
DB 모델 → 환자 API → AI 판독 API → 통계 → Docker(전원)
```

- 앞 담당 : merge 끝나면 팀에 "develop에 합쳤어요" 알리기
- 다음 담당 : 4-1(develop 최신 받기)부터 시작해서 앞 작업을 포함한 상태로 브랜치 만들기

---

## 6. 팀장 - 최종 병합 (마지막에)

develop에 모든 기능이 모이고 확인이 끝나면 main에 합친다.

- [웹] New pull request → **base: main / compare: develop**
- 팀원 확인 후 **Merge pull request**
- 팀원 전원 [터미널]에서 main 최신 받기

```bash
git switch main
git pull origin main
```

---

## 7. 예외 - md 문서만 올릴 때
코드가 아닌 문서는 main에 바로 올려도 된다 (팀 규칙에 정한 경우).

```bash
git switch main
git pull
git add .
git commit -m "docs: 문서 설명"
git push
```

---

## 놓치기 쉬운 것

**전원**
- 작업 시작 전 항상 develop 최신 받기 (4-1)
- main에 직접 push하지 않기
- PR 만들 때 **base가 develop인지** 꼭 확인
- 경로 옆 `[브랜치이름]`으로 지금 어느 브랜치에 있는지 확인하고 작업

**팀장**
- develop 만들고 팀에 알리기
- 마지막 develop → main 병합

**리뷰어**
- 리뷰 요청 받으면 당일 안에 확인
