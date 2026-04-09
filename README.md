# 인증 시스템 설치 가이드

멤버들이 매일 글을 작성하고 인증하는 시스템입니다.  
**GitHub Pages** (무료 웹호스팅) + **Google Sheets** (데이터 저장) 조합으로 구성됩니다.

---

## 전체 구조

```
멤버 브라우저
    ↓ 글 등록
GitHub Pages (index.html)
    ↓ API 호출
Google Apps Script (Code.gs)
    ↓ 데이터 저장
Google Sheets (스프레드시트)
```

---

## STEP 1 — Google 스프레드시트 만들기

1. [Google Sheets](https://sheets.google.com) 접속
2. 새 스프레드시트 생성 → 이름: `인증 기록`
3. 스프레드시트 URL에서 ID 복사 (나중에 필요)
   ```
   https://docs.google.com/spreadsheets/d/【여기가 ID】/edit
   ```

### 스프레드시트 공유 설정 (뷰어 전용)
1. 우측 상단 **공유** 버튼 클릭
2. **링크가 있는 모든 사용자** 선택
3. 권한을 **뷰어** 로 설정
4. 완료 → 이제 누구든 볼 수 있지만 **직접 편집은 불가**

---

## STEP 2 — Google Apps Script 설정

1. 스프레드시트 메뉴에서 **확장 프로그램 → Apps Script** 클릭
2. 기존 코드 전체 삭제 후 `Code.gs` 파일 내용 전체 붙여넣기
3. 상단 함수 선택 드롭다운에서 `initSheet` 선택
4. **실행(▶)** 클릭 → 권한 허용 (Google 계정 로그인 팝업)
   - "확인되지 않은 앱" 경고 → **고급** → **계속** 클릭
5. 실행 완료 후 스프레드시트에 시트 2개가 자동 생성됨 확인

### 웹앱으로 배포
1. Apps Script 우측 상단 **배포 → 새 배포** 클릭
2. 설정:
   - 유형: **웹 앱**
   - 설명: `인증 시스템 API`
   - 다음 사용자로 실행: **나 (내 계정)**
   - 액세스 권한: **모든 사용자**
3. **배포** 클릭
4. 생성된 **웹앱 URL** 복사 (나중에 사용)
   ```
   https://script.google.com/macros/s/AKfycb.../exec
   ```

> ⚠️ 코드를 수정하면 **배포 → 배포 관리 → 편집 → 새 버전**으로 재배포해야 반영됩니다.

---

## STEP 3 — GitHub Pages 배포

### 방법 A: GitHub 웹에서 직접 (가장 쉬움)

1. [GitHub](https://github.com) 로그인 (없으면 회원가입)
2. 우측 상단 **+** → **New repository**
3. 설정:
   - Repository name: `certification` (원하는 이름)
   - **Public** 선택
   - **Add a README file** 체크
4. **Create repository** 클릭
5. **Add file → Upload files** 클릭
6. `index.html` 파일 업로드
7. **Commit changes** 클릭

### GitHub Pages 활성화
1. 레포지토리 **Settings** 탭
2. 좌측 메뉴 **Pages** 클릭
3. Source: **Deploy from a branch**
4. Branch: **main** / **/(root)** 선택
5. **Save** 클릭
6. 약 1-2분 후 URL 생성됨:
   ```
   https://【내아이디】.github.io/certification/
   ```

---

## STEP 4 — Apps Script URL 연결

1. 위에서 만든 GitHub Pages URL 접속
2. **설정** 탭 클릭
3. Apps Script URL 입력란에 STEP 2에서 복사한 URL 붙여넣기
4. **저장** 클릭
5. **순위표** 탭에서 멤버 목록이 뜨면 완료!

> 💡 이 URL은 각 멤버의 브라우저에 로컬 저장됩니다.  
> 멤버들이 처음 접속 시 설정 탭에서 URL을 입력해야 합니다.  
> 또는 URL을 `index.html` 코드에 하드코딩하면 설정 불필요.

---

## URL 하드코딩 (선택사항 — 권장)

`index.html` 파일에서 아래 줄을 찾아 수정하면 멤버들이 URL 설정을 안 해도 됩니다:

```js
// 변경 전:
let GAS_URL = localStorage.getItem('gasUrl') || '';

// 변경 후 (실제 URL로 교체):
let GAS_URL = 'https://script.google.com/macros/s/여기에URL입력/exec';
```

---

## 사용 방법

| 탭 | 기능 |
|---|---|
| **순위표** | 오늘 인증한 멤버 / 미인증 멤버 현황 |
| **글 등록** | 멤버 선택 → 1,000자 이상 글 작성 → 제출 |
| **전체 내역** | 날짜별 제출된 글 목록 조회 |
| **설정** | Apps Script URL 설정, 멤버 추가/삭제 |

---

## 자주 묻는 문제

**Q: 글 등록 시 CORS 오류가 나요**  
A: Apps Script 재배포 필요. 배포 → 배포 관리 → 편집 → 버전: 새 버전 → 배포

**Q: 멤버가 오늘 이미 등록했는데 또 등록되려고 해요**  
A: 시스템이 날짜 기준으로 중복 방지 처리를 합니다. 오류 메시지가 표시됩니다.

**Q: 스프레드시트가 뷰어 전용인데 어떻게 글이 저장되나요?**  
A: 저장은 Apps Script가 대신 합니다. Script는 스프레드시트 소유자 권한으로 실행되므로 멤버들은 직접 편집 불가지만 시스템을 통해서만 데이터가 들어갑니다.

---

## 파일 구성

```
certification/
├── index.html    ← 메인 앱 (GitHub Pages에 올리는 파일)
├── Code.gs       ← Google Apps Script에 붙여넣는 코드
└── README.md     ← 이 가이드
```
