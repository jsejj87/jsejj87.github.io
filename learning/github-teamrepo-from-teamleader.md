# GitHub 협업 초기 세팅 - 팀장 저장소로 바로 시작하기

- **[웹]** : GitHub 사이트에서 하는 작업
- **[터미널]** : VS Code 터미널에서 하는 작업
- 주소 부분은 실제 주소로 바꿔서 사용
- 남의 템플릿을 가져오는 게 아니라, 팀장 저장소에서 바로 시작하는 경우

---

## 팀장

팀장이 시작하는 상황에 따라 A 또는 B 중 하나만 하면 됨

### 방법 A. 빈 상태에서 새로 시작

#### 1. [웹] 저장소 만들기
- 프로필 사진 → Your repositories → 초록색 **New**
- 이름 입력, **Public** 선택
- **Add a README file** 체크 (이 경우엔 넣어도 됨) → **Create repository**

#### 2. [터미널] 내 저장소 가져오기
해당 위치에 저장소 폴더가 생김. 연결 주소(origin)도 자동으로 내 저장소로 잡힘

```bash
cd ~/Desktop
git clone 내저장소주소
cd 저장소폴더이름
code .
```

#### 3. [VS Code] 기본 파일 만들기
- 폴더 구조, 기본 코드 등 팀원과 같이 쓸 시작 파일 작성

#### 4. [터미널] 저장소에 올리기

```bash
git add .
git commit -m "chore: 초기 프로젝트 세팅"
git push
```

### 방법 B. 내 컴퓨터에 이미 작업 중인 폴더가 있음

#### 1. [웹] 빈 저장소 만들기
- 방법 A와 같지만 **README, .gitignore 추가 옵션은 체크하지 않음**
- 내 컴퓨터 폴더 내용을 올릴 거라 저장소가 비어 있어야 충돌이 안 남

#### 2. [터미널] 작업 폴더를 Git 저장소로 만들기
이미 Git 저장소인 폴더면 `git init`은 건너뜀

```bash
cd 작업폴더경로
git init
git add .
git commit -m "chore: 초기 프로젝트 세팅"
git branch -M main
```

- `git branch -M main` : 기본 브랜치 이름을 main으로 맞춤

#### 3. [터미널] 연결 주소 등록 후 올리기

```bash
git remote add origin 내저장소주소
git remote -v
git push -u origin main
```

- `-v` 결과에 내 저장소 주소가 나오면 성공
- `-u`는 처음 한 번만. 이후엔 `git push`만 쳐도 됨

### 공통. [웹] 팀원 초대
- 저장소 → **Settings** → **Collaborators** → **Add people** → 팀원 GitHub 아이디 입력
- 이메일 인증(Verify via email)을 요구하면 메일로 온 코드 입력

### 공통. 팀원에게 공유
- 저장소 주소 + "초대 수락하고 clone해 가세요" 알리기

---

## 팀원

### 1. [웹] 초대 수락
- 팀장이 보낸 초대 링크 또는 GitHub 알림 → **Accept invitation**
- 수락해야 나중에 push 가능

### 2. [터미널] 팀 레포 가져오기
해당 위치에 팀 레포 폴더가 생김. 연결 주소도 자동으로 팀 레포로 잡혀서 remote 작업 필요 없음

```bash
cd ~/Desktop
git clone 팀레포주소
cd 팀레포폴더이름
code .
```

### 3. [터미널] 연결 확인 (선택)

```bash
git remote -v
```

- 팀 레포 주소가 나오면 정상

### 4. 작업 시작 전 최신 받기 (매번)

```bash
git pull
```

---

## 놓치기 쉬운 것

**팀장**
- 내 컴퓨터 폴더를 올리는 경우(방법 B)는 저장소를 README 없이 만들기
- 팀원 초대 빼먹지 않기
- 팀원이 clone하기 전에 시작 파일을 먼저 push해두기 (빈 저장소를 clone하면 빈 폴더만 생김)

**팀원**
- 초대 수락하기
- 작업 시작 전에는 항상 `git pull`로 최신 받기

**공통**
- 이미 Git 저장소인 폴더 안에서 clone하지 않기 (저장소 안에 저장소가 생겨서 꼬임)
