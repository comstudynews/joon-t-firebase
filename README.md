# GitHub Codespaces와 Firebase 개발·배포 과정 (10단계 작업 기록)

[Notion 원본 작업 기록](https://app.notion.com/p/40891bd5a9ac8126a2a3c7417b9d5a94)

> 프로젝트: 수비학 기반의 오늘의 운세 웹 애플리케이션 개발  
작업 기록: 2026-10-28 ~ 2026-10-29  
실제 수행한 개발·배포 절차를 10단계로 정리한 기록임. 구성 완료, 배포 성공, 향후 구현을 구분함.


**관련 주소**

- [GitHub 저장소](https://github.com/comstudynews/joon-t-firebase)
- [GitHub Actions](https://github.com/comstudynews/joon-t-firebase/actions)
- [Firebase Console: joon-t](https://console.firebase.google.com/project/joon-t/overview)
- [Firebase Hosting 사이트](https://joon-t.web.app)
- 계획서: [Notion 원문](https://app.notion.com/p/3dc91bd5a9ac8167bfbcdbc97d5fb3b2)
- 실습 가이드: [Notion 원문](https://app.notion.com/p/40791bd5a9ac81f1a59bd881f1c57371)

## 1. GitHub에 새 Repository 생성

**작업:** GitHub에서 새 저장소 생성, `README.md`와 `main` 기본 브랜치 준비.

- 계정: `comstudynews`
- 저장소: `joon-t-firebase`
- 저장소 공개 설정 확인
- [저장소 바로가기](https://github.com/comstudynews/joon-t-firebase)

**확인:** 저장소의 Code 탭에서 `main`, README 및 첫 커밋 확인. **상태: 완료.**

**캡처 위치:** GitHub Repository 목록 및 첫 화면.

## 2. GitHub Codespaces 생성

**작업:** 저장소의 `Code → Codespaces → Create codespace on main` 선택. 브라우저 VS Code 편집기에서 개발환경 실행.

```bash
pwd
git remote -v
git branch --show-current
node -v
npm -v
```

- 작업 경로: `/workspaces/joon-t-firebase`
- [Codespaces 편집기 주소](https://nightmarish-spooky-broomstick-9g7q6xjx647cx7g7.github.dev/) (세션별 주소이므로 접근 상태가 달라질 수 있음)
- 터미널과 Git 명령 실행 확인

**상태: 완료.**  
**캡처 위치:** Codespaces 탐색기와 터미널 화면. 관련 캡처: 02_codespaces_firebase_initialization.png (이미지 첨부 대기).

## 3. Firebase 새 프로젝트 준비

**작업:** Firebase Console에서 이번 프로젝트의 Firebase 리소스 확인. 실제 개발 시 새 프로젝트 생성 대신 **기존 프로젝트** `joon-t`를 선택해 연결함.

| 항목 | 확인된 값 |
| --- | --- |
| Firebase 프로젝트 ID | `joon-t` |
| Hosting 사이트 ID | `joon-t` |
| 기본 Hosting 주소 | [https://joon-t.web.app](https://joon-t.web.app) |
| Firestore 데이터베이스 | `(default)` |
| Firestore 위치(설정 파일 기준) | `asia-northeast3` |

**주의:** `joon-t`가 정확히 언제 신규 생성되었는지는 이번 작업 기록에서 확인하지 못함.  
**캡처 위치:** Firebase Console 프로젝트/Hosting 설정 화면.

## 4. Codespaces에 원격 Firebase Hosting 프로젝트 준비

**Firebase CLI 설치와 로그인**

```bash
npm install -g firebase-tools
firebase --version
firebase login --no-localhost
firebase projects:list
firebase init
```

**실제 구성**

- `Use an existing project` → `joon-t` 선택
- Firestore Rules와 Indexes 설정
- Hosting 공개 폴더 `public` 지정
- SPA 설정 `Yes` 선택: 현재 `firebase.json`에는 모든 경로를 `/index.html`로 보내는 rewrites 존재
- 초기화 중 Storage 설정 파일도 생성됨. 이번 운세 앱의 핵심 기능은 아님
- `firebase.json`, `.firebaserc`, `firestore.rules`, `firestore.indexes.json`, `storage.rules`, `public/index.html` 생성
- 터미널 `Firebase initialization complete!` 확인

**발생한 인증 문제와 해결**

- GitHub 인증 뒤 로컬 브라우저가 `localhost:9005`로 이동해 `ERR_CONNECTION_REFUSED` 발생
- Codespaces는 원격 개발환경이어서 강의장 PC의 [localhost](http://localhost)와 분리됨
- 원래 인증 대기 터미널을 유지하고 Codespaces 새 터미널에서 OAuth Callback 요청을 전달
- HTTP 200 응답 후 인증 및 초기화 진행 완료
- 일회용 OAuth Callback 코드·토큰은 문서나 GitHub에 보관하지 않음

**확인**

```bash
cat .firebaserc
cat firebase.json
ls -la .github/workflows/
```

**상태: 완료.**  
**캡처 위치:** Firebase 초기화 성공 터미널. 캡처 파일: 02_codespaces_firebase_initialization.png (이미지 첨부 대기).

## 5. git push

**작업:** Firebase 초기화로 생긴 설정 파일을 GitHub 저장소에 반영.

```bash
git status
git add .firebaserc .gitignore firebase.json
git add firestore.rules firestore.indexes.json
git add storage.rules public/ .github/workflows/
git diff --cached --name-only
git commit -m "Configure Firebase Hosting and GitHub Actions"
git push origin main
```

- `.github/workflows/`, `firebase.json`, `public/` 등을 Git에 등록
- `.gitignore`로 디버그 로그, 임시 파일, 인증키 제외
- GitHub 저장소에서 파일 및 커밋 반영 확인

**상태: 완료.**  
**캡처 위치:** Codespaces Push 결과 또는 GitHub Commits 목록. 관련 캡처: 05_codespaces_git_push.png (이미지 첨부 대기).

## 6. GitHub Actions 연동

**작업:** `firebase init`에서 GitHub 자동 배포 설정 선택. 두 워크플로가 생성됨.

```txt
.github/workflows/
├── firebase-hosting-merge.yml
└── firebase-hosting-pull-request.yml
```

- `merge.yml`: `main` Push 시 Firebase Hosting 정식 배포
- `pull-request.yml`: PR에서 Firebase Hosting 미리보기 배포
- 배포 인증 설정은 GitHub Secret `FIREBASE_SERVICE_ACCOUNT_JOON_T`를 참조

**첫 번째 실패**

- GitHub Actions의 `npm ci && npm run build` 단계 오류
- 원인: 정적 HTML/CSS/JavaScript 프로젝트에 npm 빌드 설정 및 `package-lock.json`이 없음
- 해결: 두 워크플로에서 불필요한 `- run: npm ci && npm run build` 줄 제거

**수정 이력**

```bash
git switch -c fix/firebase-hosting-workflow
git add .github/workflows/
git commit -m "Fix Firebase Hosting workflow for static site"
git push -u origin fix/firebase-hosting-workflow
```

- [PR #1: 수정 내역](https://github.com/comstudynews/joon-t-firebase/pull/1)
- [PR 자동 배포 성공 기록](https://github.com/comstudynews/joon-t-firebase/actions/runs/40655100922)
- 해당 PR은 마지막 직접 조회 당시 `Open`이었음. PR이 병합되었다고 단정하지 않음

**상태: 워크플로 수정 완료, 자동 배포 검증 성공.**  
**캡처 위치:** 첫 실행 실패와 수정 후 성공 상태. 관련 캡처: 06_github_actions_first_failure.png, 06_pull_request_creation.png, 06_pull_request_preview_success.png (이미지 첨부 대기).

## 7. Firebase 프로젝트 배포 결과

**검증 흐름**

1. GitHub에 변경사항 Push
2. PR에서 `Deploy to Firebase Hosting on PR` 실행
3. Firebase Hosting 미리보기 URL 생성
4. 미리보기 URL에서 `Firebase Hosting Setup Complete` 기본 화면 확인
5. `main` Push에서 `Deploy to Firebase Hosting on merge` 성공 확인

**검증 자료**

- [PR 미리보기 배포 성공](https://github.com/comstudynews/joon-t-firebase/actions/runs/40655100922)
- [2026-10-29 main 배포 성공](https://github.com/comstudynews/joon-t-firebase/actions/runs/40807158219)
- 정식 대상 주소: [https://joon-t.web.app](https://joon-t.web.app)

**주의:** 배포 파이프라인 성공과 운세 웹앱 전체 기능 완성은 별개임. 처음 검증한 미리보기는 Firebase 기본 환영 페이지였음.

**캡처 위치:** Actions 초록색 성공 표시 및 Firebase Hosting 기본 페이지. 관련 캡처: 07_actions_preview_success.png, 07_firebase_hosting_preview_page.png (이미지 첨부 대기).

## 8. index.html 구현

**작업 파일:** `public/index.html`

Figma 와이어프레임을 기준으로 6개 화면을 구성하고 한 HTML 문서 안에서 JavaScript로 화면을 전환하는 방식 적용.

| 순서 | 화면 | 작성 내용 |
| --- | --- | --- |
| 1 | 시작 | 서비스 소개, 시작 버튼 |
| 2 | 정보 입력 | 이름·생년월일 입력, 입력 검증 |
| 3 | 계산 진행 | 솔넘버 계산 결과 안내 |
| 4 | 오늘의 운세 | 오늘 숫자·종합운 표시 영역 |
| 5 | 분야별 운세 | 재물운·건강운·대인관계운 |
| 6 | 이전 기록 | localStorage 조회 기록 목록 |

**작성한 코드**

- HTML5, CSS3, Bootstrap 4 기반 반응형 화면
- 버튼 클릭에 따른 화면 전환
- 이름 및 생년월일 유효성 검사
- 생년월일 숫자 합산으로 개인 솔넘버 계산
- 오늘 날짜 숫자 합산으로 오늘의 솔넘버 계산
- `localStorage`의 `fortuneHistory`에 날짜, 이름, 개인 및 오늘 솔넘버 저장
- 기록 목록 표시 및 빈 기록 안내

**계산 예시**

```txt
1990-01-25
1+9+9+0+1+2+5 = 27
2+7 = 9
개인 솔넘버 = 9
```

**미완료**

- 운세 점수 산출 공식
- Firestore `fortuneContents` 조회
- 실제 분야별 운세 문구와 점수의 기록 저장
- 기능별 통합 실행 테스트

현재 운세 문구는 화면 배치용 예시이며 Firestore 조회가 완료된 상태는 아님.

**캡처 위치:** 시작, 입력, 계산 결과, 기록 목록 등 실제 실행한 화면.

## 9. git commit 및 git push

**작업:** `public/index.html` 작성 결과를 GitHub에 반영.

```bash
git branch --show-current
git status
git add public/index.html
git commit -m "Add fortune app wireframe UI"
git push origin main
```

- 실제 `main`에서 해당 커밋 및 `public/index.html` 파일 확인
- [GitHub 최신 HTML 파일](https://github.com/comstudynews/joon-t-firebase/blob/main/public/index.html)
- [이 커밋의 Hosting 자동 배포 성공 기록](https://github.com/comstudynews/joon-t-firebase/actions/runs/40807158219)

**주의:** 이후 기능 브랜치에서 작업한다면 현재 브랜치로 Push하고 PR을 거쳐 `main`에 반영할 것. `git switch main`은 브랜치 전환 명령이며, 웹에서 PR을 병합한 경우 `git pull origin main`으로 동기화.

**상태: HTML 코드 Push 및 GitHub Actions Hosting 배포 성공.**  
**캡처 위치:** 최신 커밋 기록 및 성공한 Actions 실행. 관련 캡처: 09_branch_workflow_terminal.png (이미지 첨부 대기).

## 10. 결과 확인

**GitHub에서 실제 확인한 사항**

- [x] GitHub 저장소 생성
- [x] Codespaces 개발환경 실행
- [x] Firebase 프로젝트 `joon-t` 연결
- [x] Firebase Hosting 및 Firestore 초기 설정
- [x] GitHub Actions 워크플로 생성
- [x] npm 빌드 오류 수정
- [x] PR 미리보기 Hosting 자동 배포 성공
- [x] `main` 브랜치 Hosting 자동 배포 성공
- [x] `public/index.html` GitHub 반영

**직접 실행하여 추가 검증할 사항**

- [ ] [https://joon-t.web.app](https://joon-t.web.app) 에 최신 화면이 표시되는지 확인
- [ ] 이름·생년월일 입력 정상 동작
- [ ] `1990-01-25` 입력 시 개인 솔넘버 `9` 표시
- [ ] 날짜 기반 오늘 솔넘버 계산
- [ ] 6개 화면 전환
- [ ] 이전 기록 저장 및 목록 표시
- [ ] 새로고침 후 localStorage 기록 유지
- [ ] 모바일 브라우저 표시 확인
- [ ] 점수 계산 및 Firestore 연동 후 전체 기능 재시험

**후속 작업 순서**

1. 운세 점수 계산 기준 확정
2. Firestore `fortuneContents`에 솔넘버 1\~9의 운세 문구 입력
3. 사용자 읽기 및 쓰기 차단을 고려한 Firestore 보안 규칙 설정
4. 운세 콘텐츠 조회 코드 연결
5. 실제 운세 결과와 점수의 localStorage 저장
6. 기능 테스트, 반응형 보완, 시연 영상 및 최종보고서 정리

---

**중간보고 참고 요약**

Codespaces 개발환경 구성, Firebase 초기화, GitHub Actions 오류 수정, PR 미리보기와 main 자동 배포 성공, 와이어프레임 기반 HTML 화면 초안 작성까지 진행함. Firestore 콘텐츠 조회와 운세 점수 계산은 미완료로 구분함.

**과목 규정:** 컴퓨터프로젝트의 생성형 AI 활용 제한을 확인하고, 평가용 소스코드와 제출 문서는 과목 기준에 맞게 직접 작성·검증할 것.
