# 개인 연습 계획 — Git/GitHub 팀 협업 미션 리허설

> 목표: 실제 팀 미션 전에 **혼자서 전체 흐름을 한 번 끝까지 돌려보기**.
> 팀원 역할은 "계정 2개 + 로컬 클론 2개"로 흉내 낸다.

---

## 0. 혼자 할 때 막히는 지점과 해결책

| 막히는 지점 | 원인 | 해결책 |
|---|---|---|
| PR 승인(approve) 불가 | 내 PR은 내가 승인 못 함 | **GitHub 부계정 1개 생성** → Collaborator로 초대 (권장) |
| Branch Protection 설정 불가 | 무료 플랜은 private 저장소에 보호 규칙 X | 저장소를 **public**으로 생성 |
| 충돌이 "자연스럽게" 안 생김 | 혼자 순서대로 작업하면 충돌 없음 | 브랜치 2개를 같은 base에서 만든 뒤 **같은 줄을 다르게 수정** |
| 팀원별 기여 구분 | 커밋 작성자가 전부 나 | 클론 2개에 각각 다른 `user.name`/`user.email` 설정 |

### 역할 설정
- **멤버 A** = 본계정 → 폴더 `~/Desktop/git-collab-practice`
- **멤버 B** = 부계정 → 폴더 `~/Desktop/git-collab-practice-B` (나중에 clone)

```bash
# 각 클론 폴더 안에서 (--global 말고 로컬 설정!)
git config user.name "member-A"
git config user.email "본계정 이메일"
```

> 부계정 push 인증: HTTPS + 부계정 Personal Access Token 사용이 가장 간단.
> (macOS 키체인에 본계정 토큰이 저장돼 있으면 B 클론의 remote URL을
> `https://<부계정아이디>@github.com/...` 형태로 지정하면 계정별로 분리됨)

---

## 1단계 — 저장소 준비 (30분)

- [ ] (선택) `brew install gh` → `gh auth login` (Issue/PR을 터미널에서 만들 수 있어 편함. 웹으로 해도 됨)
- [ ] GitHub에서 **public** 저장소 생성 (Organization 연습하고 싶으면 무료 Org 만들어서 그 안에 생성)
- [ ] 부계정을 Collaborator(또는 Org 멤버)로 초대 → 부계정에서 수락
- [ ] 로컬에서 기본 구조 커밋 후 첫 push (**보호 규칙 걸기 전에** 해야 main에 직접 push 가능)
  ```bash
  cd ~/Desktop/git-collab-practice
  git init -b main
  # README.md, docs/, src/, team/ 에 최소 파일 생성 (빈 폴더는 .gitkeep)
  git add . && git commit -m "chore: 프로젝트 기본 폴더 구조 생성"
  git remote add origin https://github.com/<본계정>/<저장소>.git
  git push -u origin main
  ```
- [ ] Settings → Branches(또는 Rules → Rulesets) → `main` 보호 규칙
  - Require a pull request before merging ✅
  - Require approvals: **1** ✅
  - (권장) Do not allow bypassing ✅ ← 관리자인 본인도 직접 push 못 하게
- [ ] **검증**: `main`에서 직접 push 시도 → 거부되는지 확인 (이 에러 메시지도 troubleshooting 소재!)
- [ ] 부계정으로 B 클론 생성: `git clone ... ~/Desktop/git-collab-practice-B`

## 2단계 — 규칙 정하기 (CONTRIBUTING.md 초안, 30분)

이것부터 **첫 번째 Issue → PR**로 처리하면 일석이조.

- 브랜치: `feature/<이슈번호>-<짧은-설명>` / `fix/...` / `docs/...`
  예) `feature/3-add-slugify`
- 커밋: `type: 무엇을 왜` (feat, fix, docs, refactor, chore, test)
  - ❌ `update`, `fix`, `wip`, `bug fix`
  - ✅ `fix: slugify가 연속 공백을 하이픈 여러 개로 바꾸던 문제 수정`
- PR 본문 템플릿: What / Why / How(검증 방법) / `Closes #N`
  → `.github/pull_request_template.md`로 만들어 두면 자동으로 채워짐
- 리뷰 규칙: LGTM 단독 금지, 라인 코멘트 1개 이상(질문/대안/리스크)
- GitHub Flow 선택 이유 3줄

## 3단계 — GitHub Flow 사이클 반복 (핵심, 1~2시간)

결과물은 **(A) 유틸 함수 모음(Python)** 추천 — 리뷰 코멘트 거리가 많이 나와서 연습에 좋음.

### 한 사이클 = 아래 8단계
1. Issue 생성 (예: "#3 문자열 slugify 함수 추가")
2. `git switch main && git pull`
3. `git switch -c feature/3-add-slugify`
4. 코드 작성 → 의미 있는 커밋 1~3개
5. `git push -u origin feature/3-add-slugify`
6. PR 생성 (What/Why/How + `Closes #3`)
7. **다른 계정으로** 리뷰: 특정 라인에 코멘트 (예: "빈 문자열 입력 시 동작은?") → Request changes
8. 작성자가 수정 커밋 push + 코멘트에 답글 → 리뷰어 Approve → Merge → 브랜치 삭제

### 반복 목표 (실제 팀 기준 1인분을 A/B 각각 채우기)
| | 멤버 A | 멤버 B |
|---|---|---|
| PR 생성·병합 | 2개+ | 2개+ |
| 남이 쓴 PR 리뷰 | 2개+ | 2개+ |
| 리뷰 반영 | 1회+ | 1회+ |

> 💡 Merge 방식은 **"Create a merge commit"** 으로 하면 나중에 `git log --graph`가 갈라지고 합쳐지는 모양으로 예쁘게 나옴.

## 4단계 — 충돌 2회 만들기 (1시간)

### 충돌 ① 같은 hunk (비자명 충돌 조건 충족)
1. 최신 main에서 A가 `feature/10-readme-intro`, B가 `feature/11-readme-intro` 생성
2. 둘 다 `README.md`의 **같은 줄(예: 프로젝트 소개 문단)** 을 다르게 수정 → 각자 PR
3. A의 PR 먼저 병합 → B의 PR에 "This branch has conflicts" 표시
4. B 로컬에서 해결:
   ```bash
   git switch feature/11-readme-intro
   git fetch origin && git merge origin/main   # 충돌 발생
   # <<<<<<< HEAD (내 변경) ======= (main 변경) >>>>>>> origin/main  → 직접 편집
   git add README.md && git commit   # 기본 메시지 or "merge: main 반영 및 README 소개 충돌 해결"
   git push
   ```
5. 충돌 마커 원본을 **스크린샷/복사**해서 문서에 남기기 (해결 전 상태가 중요!)

### 충돌 ② 파일 이동/삭제 vs 내용 수정
- A: `git mv src/string_utils.py src/utils/string_utils.py` (또는 `git rm`)
- B: 같은 파일 내용 수정
- ⚠️ **주의**: 단순 이름변경 + 내용수정은 Git이 rename을 감지해서 **자동 병합되는 경우가 많음**.
  확실하게 충돌을 보려면 → A는 **삭제(git rm)**, B는 **수정** → `modify/delete` 충돌 발생
  또는 A/B 둘 다 **서로 다른 이름으로 rename** → `rename/rename` 충돌
- 자동 병합됐다면 그것도 "왜 충돌이 안 났는지(rename detection)" 기록하면 좋은 학습 소재

### 기록 템플릿 (docs/conflict-resolution.md)
```
## 충돌 #1
- 관련 PR/이슈: #
- 충돌 유형: 같은 hunk / modify-delete / rename
- 발생 원인:
- 충돌 당시 마커(원문):
- 해결 결정과 이유:
- 사용한 명령어:
- 해결 커밋: <sha>
- 담당: 이름(역할)
```

## 5단계 — 트러블슈팅 4종 (1시간)

전부 **feature 브랜치에서** 수행 (main은 보호 중이라 직접 못 건드림).

| 시나리오 | 재현 방법 | 해결 명령 | 핵심 포인트 |
|---|---|---|---|
| amend | 오타 있는 메시지로 커밋 (push 전) | `git commit --amend -m "올바른 메시지"` | 커밋 SHA가 바뀜 → **push 전에만** 안전 |
| reset --soft | 아직 덜 된 걸 커밋해버림 | `git reset --soft HEAD~1` | 커밋만 취소, 변경은 staged로 남음. `--mixed`/`--hard` 차이도 같이 적기 |
| revert | 잘못된 커밋이 **이미 main에 병합됨** | 새 브랜치에서 `git revert <sha>` → PR | 히스토리를 지우지 않고 "취소 커밋"을 추가. 공유된 커밋엔 reset 대신 revert |
| stash | 작업 중 급한 다른 이슈 처리 요청 | `git stash` → 다른 브랜치 작업 → 복귀 후 `git stash pop` | `git stash list`, untracked 포함은 `-u` |

각 항목마다 **명령 실행 전/후 `git log --oneline -3`, `git status` 출력**을 붙여두면 "재현 가능한 기록" 요건 충족.

## 6단계 — 마무리 문서 (30분)

- [ ] `git log --oneline --graph --all > docs/git-log.txt` (PR로 올리기)
- [ ] `SUBMISSION.md`: 저장소 URL, 멤버별 Issue/PR 링크 표, 문서 3종 링크, 증빙 링크
  - 팁: Issue/PR 목록 필터 URL 활용 → `.../pulls?q=is:pr+author:<아이디>`, `.../issues?q=author:<아이디>`
- [ ] README에 결과물 목록/사용 예시 링크

---

## 최종 체크리스트 (요구사항 매핑)

- [ ] main 보호 규칙 (직접 push 금지 / PR만 / 승인 1)
- [ ] CONTRIBUTING.md: 브랜치 규칙, 커밋 컨벤션, PR 규칙, 리뷰 규칙, 충돌 대응 흐름, GitHub Flow 이유 3줄
- [ ] 모든 PR에 `Closes #N` + What/Why/How
- [ ] 멤버별 PR 2+, 리뷰 2+, 리뷰 반영 1+
- [ ] 각 PR에 실질 리뷰 코멘트 1+ 및 상호작용 기록
- [ ] 충돌 2회 이상 (1회는 비자명) → conflict-resolution.md
- [ ] 트러블슈팅 4종 → troubleshooting-log.md (담당자 이름 표시)
- [ ] 결과물: 멤버별 함수 1+ & 사용 예시, 멤버별 기여 커밋 1+
- [ ] git log graph 증빙
- [ ] SUBMISSION.md

## 연습 후 스스로 설명해볼 질문 (과제 목표)
1. 브랜치는 왜 "커밋을 가리키는 포인터"인가? (`cat .git/refs/heads/main` 해보기)
2. GitHub Flow가 Git Flow보다 우리 팀에 맞는 이유는?
3. PR/코드 리뷰가 없으면 어떤 문제가 생기나?
4. 충돌은 왜 생기고 마커 세 줄은 각각 무엇인가?
5. reset / revert / stash는 각각 언제 쓰나? (특히 push 이후엔 왜 reset이 위험한가)
