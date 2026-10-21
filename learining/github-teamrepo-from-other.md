# GitHub 협업 초기 세팅 (팀장편 · 팀원편) - 팀장이 다른 사람 템플릿 가져와서 협업

- **[웹]** : GitHub 사이트에서 하는 작업
- **[터미널]** : VS Code 터미널에서 하는 작업
- 주소 부분은 실제 주소로 바꿔서 사용

---

## 팀장

### 1. [웹] 빈 저장소 만들기
- 프로필 사진 → Your repositories → 초록색 **New**
- 이름 입력, **Public** 선택 → **Create repository**
- README, .gitignore 추가 옵션은 체크하지 않음 (빈 저장소여야 push가 깔끔하게 됨)
- 만들고 나면 나오는 저장소 주소 복사해두기

### 2. [터미널] 템플릿 가져오기
가져올 위치로 이동 후 clone → 해당 위치에 템플릿 폴더가 생김

```bash
cd ~/Desktop
git clone 템플릿주소
```

### 3. [터미널] 가져온 폴더로 이동

```bash
cd 템플릿폴더이름
```

### 4. [터미널] 연결 주소를 내 저장소로 교체
clone하면 연결 주소(origin)가 템플릿 저장소로 잡혀 있음 → 지우고 내 저장소로 다시 등록

```bash
git remote remove origin
git remote add origin 내저장소주소
git remote -v
```

- `-v` 결과에 내 저장소 주소가 나오면 성공

### 5. [터미널] 내 저장소로 올리기

```bash
git push -u origin main
```

- `-u`는 처음 한 번만. 이후엔 `git push`만 쳐도 됨
- [웹]에서 저장소 새로고침해서 파일이 올라왔는지 확인

### 6. [웹] 팀원 초대
- 저장소 → **Settings** → **Collaborators** → **Add people** → 팀원 GitHub 아이디 입력
- 이메일 인증(Verify via email)을 요구하면 메일로 온 코드 입력

### 7. 팀원에게 공유
- 저장소 주소 + "초대 수락하고, push 완료됐으니 받아가세요" 알리기

---

## 팀원

### 1. [웹] 초대 수락
- 팀장이 보낸 초대 링크 또는 GitHub 알림 → **Accept invitation**
- 수락해야 나중에 push 가능 (clone, pull은 수락 안 해도 됨)

### 2. [터미널] 저장소 가져오기
방법은 두 가지, 결과는 같음

**방법 A. 팀 레포를 바로 clone (간단)**
연결 주소가 처음부터 팀 레포로 잡혀서 3번 필요 없음

```bash
cd ~/Desktop
git clone 팀레포주소
cd 팀레포폴더이름
```

**방법 B. 템플릿을 clone**
연결 주소가 템플릿으로 잡혀 있어서 3번 필요

```bash
cd ~/Desktop
git clone 템플릿주소
cd 템플릿폴더이름
```

### 3. [터미널] 연결 주소를 팀 레포로 교체 (방법 B만)

```bash
git remote remove origin
git remote add origin 팀레포주소
git remote -v
```

- `-v` 결과에 팀 레포 주소가 나오면 성공

### 4. [터미널] 팀 레포 최신 내용 받기
팀장이 push 완료라고 한 뒤에 실행

```bash
git pull origin main
git branch -u origin/main
```

- `Already up to date.`가 나와도 정상
- 두 번째 줄은 앞으로 `git pull`, `git push`만 쳐도 팀 레포 main과 주고받게 하는 설정 (한 번만)

### 5. [터미널] VS Code로 열기

```bash
code .
```

---

## 놓치기 쉬운 것

**팀장**
- 저장소는 README 없이 만들기
- push 전에 `git remote -v`로 주소 확인하기
- 팀원 초대 빼먹지 않기

**팀원**
- 초대 수락하기
- 템플릿을 clone했다면 remote 꼭 바꾸기
- 작업 시작 전에는 항상 `git pull`로 최신 받기

**공통**
- 이미 Git 저장소인 폴더 안에서 clone하지 않기 (저장소 안에 저장소가 생겨서 꼬임)
