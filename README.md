# git_practice

git/GitHub 업로드 연습용 저장소입니다.

## 실행

```bash
python hello.py
```

## 연습한 과정
1. `git init` — 로컬 git 저장소 초기화
2. `git add`, `git commit` — 변경사항 기록
3. GitHub에 빈 저장소 생성
4. `git remote add origin <저장소 주소>` — 로컬과 원격 연결
5. `git push -u origin main` — 원격 저장소에 업로드

## GitHub 업로드/수정 방법 정리

### 1. 토큰(PAT)으로 올리기 (HTTPS 방식)

**언제 쓰나**: 본인 계정 저장소에 push할 때 가장 간단한 방법

1. GitHub → 우측 상단 프로필 → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token
2. Resource owner: 본인 계정
3. Repository access: "Only select repositories" → 올릴 저장소 선택
4. Permissions → Repository permissions → **Contents: Read and write**
5. Generate → 토큰(`github_pat_...`) 복사 (한 번만 표시됨)

```bash
git remote add origin https://github.com/계정명/저장소명.git
git push -u origin main
# Username: 계정명
# Password: 토큰 붙여넣기
```

**매번 토큰 입력 안 하려면** (1회만 설정):
```bash
git config --global credential.helper store
```
이후 한 번만 더 입력하면 `~/.git-credentials`에 저장되어 자동 인증됨.

**주의**: 저장소를 새로 추가/생성하면 토큰의 Repository access에도 그 저장소를 추가해줘야 함. 안 하면 403 에러.

### 2. SSH로 올리기

**언제 쓰나**: 토큰 재발급/만료 신경 안 쓰고 싶을 때, 여러 저장소에 반복적으로 push할 때 편함

**SSH 키 확인/생성**:
```bash
cat ~/.ssh/id_ed25519.pub
# 없으면 새로 생성
ssh-keygen -t ed25519 -C "이메일주소"
```

**GitHub에 공개키 등록**:
1. GitHub → Settings → SSH and GPG keys → New SSH key
2. 공개키(`.pub` 파일) 내용 전체를 붙여넣기
3. Add SSH key

**등록 확인**:
```bash
ssh -T git@github.com
# "Hi 계정명! You've successfully authenticated..." 뜨면 성공
```

**remote를 SSH 주소로 설정**:
```bash
git remote add origin git@github.com:계정명/저장소명.git
# 이미 https로 등록했다면:
git remote set-url origin git@github.com:계정명/저장소명.git
git push -u origin main
```

⚠️ **주의**: SSH 키는 계정 단위로 등록되는 것이라, 한 컴퓨터에 여러 GitHub 계정을 쓴다면 지금 등록된 키가 어느 계정 것인지 `ssh -T git@github.com` 결과로 꼭 확인할 것.

### 3. 다른 사람(팀원)의 GitHub 저장소 수정하는 법

**방법 A: Collaborator로 초대받기** (같은 저장소에 직접 push 권한을 받는 경우)
1. 저장소 소유자가: 저장소 → Settings → Collaborators and teams → Add people → 내 GitHub 아이디 검색 → 초대
2. 초대 이메일/알림 수락
3. 이후는 위 1번(토큰) 또는 2번(SSH) 방법 그대로 사용 — 내 계정 인증 정보로 push 가능

**방법 B: Fork + Pull Request** (권한 없이 오픈소스/외부 프로젝트에 기여할 때)
1. 원본 저장소 페이지에서 우측 상단 **Fork** 클릭 → 내 계정으로 복사본 생성
2. 내 계정의 fork를 clone
   ```bash
   git clone https://github.com/내계정/저장소명.git
   ```
3. 수정 후 커밋 → 내 fork에 push (1, 2번 방법과 동일)
4. GitHub에서 **Compare & pull request** 버튼으로 원본 저장소에 PR 생성
5. 원본 저장소 관리자가 리뷰 후 Merge하면 반영됨

| 방법 | 필요 조건 | push 대상 |
|---|---|---|
| Collaborator | 소유자가 초대해줌 | 원본 저장소에 직접 |
| Fork + PR | 조건 없음 (누구나) | 내 fork에 push, 원본엔 PR로 제안만 |

### 4. 알아두면 좋은 것들

- **`.gitignore`**: 커밋하면 안 되는 파일(토큰, `__pycache__`, 대용량 데이터 등)을 미리 걸러내는 설정 파일.
- **커밋 전 `git status` / `git diff`**: 뭐가 올라가는지 항상 확인하는 습관. 토큰 같은 민감 정보를 실수로 커밋하는 걸 방지하는 데 중요.
- **`git pull`**: 다른 곳(다른 컴퓨터, 협업자)에서 변경한 내용을 내 로컬로 받아오는 명령. push 전에 습관적으로 해두면 충돌을 줄일 수 있음.
- **브랜치(`git branch`, `git checkout -b <이름>`)**: 기능 개발/실험할 때 main을 건드리지 않고 따로 작업하는 방법.
- **실수 되돌리기**:
  - `git revert <커밋>` — 안전. 되돌리는 내용을 새 커밋으로 남김. 이미 push한 커밋도 안전하게 되돌릴 때 사용.
  - `git reset` — 과거 시점으로 되감음. 이미 push한 커밋에 쓰면 원격과 꼬일 수 있어 위험.
- **토큰/비밀번호는 절대 커밋 금지**: 실수로 올렸다면 파일만 삭제해도 git 히스토리엔 남아있으므로, 히스토리에서 완전히 제거하고 해당 토큰은 즉시 재발급(폐기)해야 함.
- **Private vs Public**: 민감한 코드/데이터가 있으면 Private으로 유지. 나중에 언제든 Public으로 전환 가능 (반대도 가능).
