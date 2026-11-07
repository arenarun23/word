# 끝말잇기 놀이터 (word)

캐릭터 '말랑이'와 하는 끝말잇기. 말랑이 먼저/내가 먼저, 쉬워요/도전! 모드.

## 구조
- `index.html`(약 61KB), 보조 사전 `extra.txt`(약 5.2만 낱말, 없어도 동작), 사전 생성 `build-extra.mjs`+`source/`(hunspell-ko 라이선스 `source/LICENSE-hunspell-ko.txt`), `vercel.json`(허브에서는 미사용). 허브 `exclude`: `source`·`build-extra.mjs`·`vercel.json` — **`extra.txt` 는 배포에 포함**돼야 한다.
- 입력: 키보드 / 🎙️ 말하기(Web Speech, 자모 거리로 비슷한 낱말 보정·"혹시 이 말?" 후보) / ✍️ 쓰기(구글 입력도구 비공식 필기 인식) / 💡 힌트. "모르겠어·몰라·도와줘" 라고 하면 이어갈 낱말 3개를 추천하고 말로도 읽어 준다.
- 규칙: 두음법칙(`startsFor`), 사전에 없는 말도 첫 글자만 맞으면 격려하며 받음, 말랑이가 이을 말이 없으면 아이 승리. 쉬운 낱말 약 1,100개는 `<script id="kidWords">`(낱말 뒤 이모지).
- 구조: `.app > header.top / #home / #play(.colA 문제·대화, .colB 입력·도구·필기판) / #result`. 화면 맞춤: `html{font-size:clamp(13px,min(2.95vh,4.9vw),22px)}`, 가로 화면은 2단, ≤420px 높이는 글자 최소 10.5px. 세로 폰에서 필기판을 연 상태만 스크롤이 생기는 것이 허용 범위.
- 테스트: 로컬 서버로 열고 `#btnGo` → `#inp` 입력 → `.ok-btn`, "모르겠어" 입력 시 `#hints button` 3개, `#bPen` 필기판. 이 앱은 `word` 전용 세션에서도 기능이 추가되므로 push 전 fetch·rebase.

## 공통 규칙 (AIEDU 허브)
- 이 앱은 **AIEDU 허브**(https://aiedu-pearl.vercel.app)의 서비스 중 하나다. 허브는 `arenarun23/aiedu` 저장소이고, 빌드 때 이 저장소를 가져와 `/word/` 하위 경로로 서비스한다. 허브 전체 구조·점검법은 그 저장소의 `.claude/skills/aiedu-hub`, 서비스별 설명은 `.claude/skills/aiedu-services` 에 있다(필요하면 `arenarun23/aiedu` 를 add_repo 로 붙여 읽는다).
- **정적 파일 + 상대경로만** 사용한다(`/foo.js` 같은 절대경로 금지). 서버·API 키 없음. 빌드가 필요한 구조로 바꾸지 않는다.
- **화면이 깨지지 않고 한 화면에 맞는 것**이 중요하다(주 사용 기기: 삼성 태블릿 가로·폰). 수정 후 1280×560, 1366×650, 800×1280, 390×780, 360×640, 800×360 크기에서 확인한다(허브 저장소 `scripts/fit-check.mjs` 참고). 아이용 큰 터치 버튼은 낮은 화면 미디어쿼리 안에서만 줄인다.
- `main` 에 push 하면 `.github/workflows/notify-hub.yml` 이 허브를 즉시 다시 배포한다(저장소 Secret `HUB_DEPLOY_HOOK` 사용, 20~60초). **Secret 값·훅 주소는 파일/커밋/로그에 쓰지 않는다.**
- push 전 `git fetch origin main` 으로 확인하고, 같은 앱을 다른 세션이 고치고 있었다면 rebase 해서 양쪽 기능을 모두 살린다.
- 반영이 안 보이면 코드보다 먼저 배포본을 비교한다: `curl -s https://aiedu-pearl.vercel.app/apps.json` 의 `word` 항목 `sha` 가 이 저장소 최신 커밋과 같은지.
- **AIEDU 복귀 아이콘**(`a.aiedu-home`, 웃는 별 44px): 모든 서비스가 같은 모양·같은 자리(페이지 왼쪽 위 8px, `<body>` 첫 요소, 링크 `../`)에 둔다. 지우거나 옮기지 않는다. 레이아웃을 바꿀 때 왼쪽 위 52×52px 를 비워 둔다 — 이 앱의 자리 확보: `.top{padding-left:52px}`(제목 줄). 원본 코드: 허브 저장소 `templates/aiedu-home.html`.

- **외부 호스트 호출 금지(보안, CSP 가 브라우저에서 차단)**: 허용되는 외부 주소는 필기 인식 `inputtools.google.com` 하나뿐이다. 글꼴은 허브의 자체 호스팅 CSS 를 쓴다 — `<link rel="stylesheet" href="../fonts/jua.css">`(`jua`·`gowun-dodum`·`noto-sans-kr`·`noto-serif-kr`·`noto-serif-tc`). **Google Fonts 링크, CDN 이미지·스크립트를 넣지 않는다.** 이미지·데이터는 이 저장소 안에 둔다. 사용자·인식 결과를 HTML 에 넣을 땐 `esc()`/`textContent`, 속성은 큰따옴표. 자세한 규칙: 허브 저장소 `.claude/skills/aiedu-hub/references/security.md`.

- **이용 기록(익명)**: `<head>` 에 `<script src="../aiedu-log.js"></script>` 를 유지한다. 시작·완료·사용 지점에서 `window.aieduLog && aieduLog("start"|"finish"|"use", "영문소문자태그")` 만 호출하고, **아이가 입력한 낱말·필기·음성·정답·나라 이름 등 내용은 절대 넘기지 않는다**(IP·내용 미기록 원칙). 상세: 허브 저장소 `.claude/skills/aiedu-hub/references/usage-log.md`.
