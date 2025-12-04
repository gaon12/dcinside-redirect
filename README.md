# DCInside m → PC Redirector

일부 학교·기업 등의 네트워크 환경에서 `m.dcinside.com` 도메인은 차단되어 있지만  
`gall.dcinside.com`(PC 버전 갤러리 도메인)은 허용되는 경우가 있습니다.

이 크롬 확장 프로그램은 **브라우저가 `m.dcinside.com`으로 접속하기 직전**에  
URL을 자동으로 PC용 도메인으로 변환하여, 모바일 링크도 문제 없이 열 수 있게 만들어 줍니다.

예를 들어,

- 검색 결과나 다른 사이트에서 `https://m.dcinside.com/board/...` 링크를 클릭해도
- 실제로는 `https://gall.dcinside.com/...` 또는 `https://dcinside.com/search...` 등으로 이동하도록 해 줍니다.

Manifest V3 기반이며, 크롬 최신 버전에서 동작합니다.

---

## 지원하는 URL 패턴

### 1. 게시글 보기 (모든 갤러리)

- 원래 모바일 URL

  - `https://m.dcinside.com/board/<gallery_id>/<post_no>`
  - `https://m.dcinside.com/board/<gallery_id>/<post_no>?page=...&exception_mode=...&...`

- 변환되는 PC URL

  - `https://gall.dcinside.com/board/view/?id=<gallery_id>&no=<post_no>`
  - 쿼리 스트링이 있을 경우 → 뒤에 그대로 `&page=...&exception_mode=...` 등의 형식으로 이어 붙입니다.

### 2. 갤러리 리스트(게시글 목록)

- 원래 모바일 URL

  - `https://m.dcinside.com/board/<gallery_id>`
  - `https://m.dcinside.com/board/<gallery_id>?page=...&listnum=...`

- 변환되는 PC URL

  - `https://gall.dcinside.com/board/lists/?id=<gallery_id>`
  - 쿼리 스트링이 있을 경우 → `https://gall.dcinside.com/board/lists/?id=<gallery_id>&page=...&listnum=...`

### 3. 갤로그(gallog)

- 원래 모바일 URL

  - `https://m.dcinside.com/gallog/<nickname>`
  - `https://m.dcinside.com/gallog/<nickname>/YYYYMMDD`
  - `https://m.dcinside.com/index.php/gallog/<nickname>/...`

- 변환되는 PC URL

  - `https://gallog.dcinside.com/<nickname>`
  - `https://gallog.dcinside.com/<nickname>/YYYYMMDD`
  - `index.php` 부분은 제거하고, 그 뒤의 경로를 그대로 유지합니다.

### 4. 검색(통합, 갤러리, 게시물 등)

- 원래 모바일 URL 예시

  - `https://m.dcinside.com/search`
  - `https://m.dcinside.com/search?keyword=...&search_type=all`
  - `https://m.dcinside.com/search/gall_content?keyword=...`
  - `https://m.dcinside.com/search/gall_name?keyword=...`

- 변환되는 PC URL 예시

  - `https://dcinside.com/search`
  - `https://dcinside.com/search?keyword=...&search_type=all`
  - `https://dcinside.com/search/gall_content?keyword=...`
  - `https://dcinside.com/search/gall_name?keyword=...`

즉, `/search` 뒤의 경로와 쿼리 전체를 그대로 유지한 채,  
**도메인만 `m.dcinside.com` → `dcinside.com` 으로 치환**합니다.

> DCInside 측 검색 서비스 내부 구조가 변경되면 일부 검색 URL이 정확히 같은 화면으로 이어지지 않을 수 있습니다.  
> 이 경우, rules_1.json의 검색 관련 규칙(마지막 규칙)을 수정하여 원하는 PC 검색 URL 형태에 맞추어 사용할 수 있습니다.

---

## 설치 방법 (로컬 개발용 / 크롬 웹스토어 미등록 버전)

1. 이 저장소를 클론하거나 ZIP으로 내려받습니다.

   ```bash
   git clone https://github.com/<your-account>/dcinside-m2pc-redirect.git
````

2. 크롬 주소창에 아래를 입력하여 확장 프로그램 관리 페이지를 엽니다.

   ```text
   chrome://extensions/
   ```

3. 우측 상단의 **개발자 모드**를 활성화합니다.

4. **압축해제된 확장 프로그램을 로드** 버튼을 클릭하고,
   클론/압축해제한 폴더(예: `dcinside-m2pc-redirect/`)를 선택합니다.

5. 확장 프로그램이 목록에 나타나면 활성화 토글이 켜져 있는지 확인합니다.

6. 이후 브라우저 어디에서든 `https://m.dcinside.com/...` 링크를 클릭하면
   자동으로 PC 도메인(`gall.dcinside.com`, `dcinside.com`, `gallog.dcinside.com`)으로 변환되어 열립니다.

---

## 동작 방식

이 확장 프로그램은 **Manifest V3의 `declarativeNetRequest` API**를 사용합니다.

* 브라우저가 네트워크 요청을 보내기 전에,
* 요청 URL이 `https://m.dcinside.com/...` 패턴과 일치하는지를 정규식으로 검사합니다.
* 일치할 경우, 미리 정의한 규칙에 따라 새로운 URL로 **리디렉트**합니다.
* 리디렉트는 **브라우저 엔진 레벨에서 선언적으로 처리**되므로,

  * 별도의 background 스크립트나 content script가 필요 없습니다.
  * 성능과 안정성 면에서 유리합니다.

---

## 커스터마이징 방법

### 1. 갤러리 ID 패턴 확장

현재 규칙에서는 갤러리 ID를

```regex
[A-Za-z0-9_-]+
```

로 가정하고 있습니다.
특수문자가 더 들어가는 갤러리가 있을 경우, `rules_1.json` 에서
해당 부분을 적절한 정규식으로 수정하여 사용할 수 있습니다.

예: 점(`.`) 문자를 포함하는 갤 ID까지 허용하고 싶다면

```regex
[A-Za-z0-9_.-]+
```

처럼 변경 가능합니다.

### 2. mini 갤러리 등의 추가 패턴

`m.dcinside.com/mini/...` 등 다른 모바일 전용 경로를 PC 도메인으로 옮기고 싶다면,

1. `rules_1.json` 배열의 마지막에 새 객체를 추가하고,
2. `regexFilter` 에 `^https://m\\.dcinside\\.com/mini/...` 형식의 패턴을,
3. `regexSubstitution` 에 원하는 PC 쪽 URL 형식을 지정하면 됩니다.

추가 규칙을 만들 때는 `id`를 기존 값과 겹치지 않게(7, 8, …)만 설정하면 됩니다.

---

## 제한 사항 및 주의점

* 이 확장 프로그램은 **`m.dcinside.com` → PC 도메인으로의 리디렉트만** 처리합니다.

  * 이미 `gall.dcinside.com`, `dcinside.com`, `gallog.dcinside.com` 으로 접속된 URL은 건드리지 않습니다.
* DCInside 웹사이트의 내부 URL 구조가 변경될 경우

  * 일부 규칙(특히 검색, 갤로그 관련)이 제대로 동작하지 않을 수 있습니다.
  * 이 경우 `rules_1.json`을 수정하여 새 구조에 맞게 다시 매핑해야 합니다.
* Manifest V3, `declarativeNetRequest`를 사용하는 관계로

  * 크롬 계열 브라우저(Chrome, Edge, Brave 등)에서의 사용을 전제로 합니다.
  * 다른 브라우저(예: Firefox)에서는 별도의 포팅 작업이 필요합니다.

---

## 라이선스

본 저장소는 MIT 라이선스로 배포됩니다.