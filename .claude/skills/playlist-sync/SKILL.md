---
name: playlist-sync
description: 기사 제목과 동일한 이름의 스포티파이 플레이리스트를 생성/갱신하고, 기사 마지막 '들으러 가기' 슬라이드의 재생 버튼에 연동한다. 기사 제목이 바뀌거나 앨범 목록이 바뀌었을 때, 또는 플리를 새로 만들 때 사용.
---

# playlist-sync — 기사 ↔ 스포티파이 플레이리스트 연동

기사 페이지의 제목과 수록 앨범 목록을 읽어, 같은 제목의 스포티파이 플레이리스트를 만들고
기사 마지막 슬라이드의 재생 버튼(`.pl-btn`)에 그 링크를 거는 작업 절차.

## 입력
- 대상 기사 HTML (기본: `articles/korean-indie-new-wave.html`)

## 절차

1. **기사 제목 추출** — 대상 HTML의 `<h1 class="cover-min">` 텍스트를 읽는다.
   `<title>`이나 README가 아니라 커버 슬라이드의 h1이 기준이다.

2. **앨범 목록 추출** — 마지막 슬라이드의 `.listen` 블록에서 각 `.ls` 항목의
   `.lt` 텍스트(`아티스트 — 앨범명`)를 순서대로 수집한다.

3. **플레이리스트 생성** — Spotify 커넥터(`mcp__Spotify__create_playlist`)를 호출한다.
   - 커넥터가 세션에 없으면 ToolSearch로 로드하고, 그래도 없으면 사용자에게
     Spotify 커넥터 재연결을 요청하고 중단한다.
   - prompt는 영어로 쓰되(ko 미지원) 다음을 반드시 포함한다:
     - `Create a playlist whose name must be exactly this string, character for character: "<1의 기사 제목 그대로>"`
       — "named"만 쓰면 제목이 임의로 줄여지는 사례가 있음. 응답의 `title`이 기사 제목과
       다르면 위 문구로 1회 재시도한다.
     - 곡 구성: **앨범당 타이틀곡 1곡**. 각 항목을 `"곡명" by Artist` 형태로 기사 순서대로
       번호를 붙여 나열하고, `12 tracks total, one per album, in this exact order.`로 끝맺는다.
     - 타이틀곡 목록은 기사 리서치 기준: 씬이 버린 아이들(버둥) · 뱅버스(250) ·
       무지개꽃 피어있네(콩코드) · Yeontral Park(곽태풍) · Put on a Tie(집사) ·
       저회(khc/moribet) · 12가지 말들(봉제인간) · 케이크가불쌍해(김승주) ·
       청호춘가(블루터틀랜드) · Youth Heritage(팔칠댄스) · 좋은 친구들(천진우) · 비가(유령서점)

4. **버튼 연동** — 응답의 playlist URL에서 쿼리스트링을 제거한
   `https://open.spotify.com/playlist/<id>` 형태로 기사 내 `.pl-btn`의 `href`를 교체한다.
   `.pl-btn`이 없으면 마지막 슬라이드 `h2` 아래에 다음 마크업으로 추가한다:
   `<a class="pl-btn r" style="--i:2" href="<URL>" target="_blank" rel="noopener">&#9654; 플레이리스트 재생 — Spotify</a>`
   (뒤따르는 형제들의 `--i` 인덱스를 하나씩 밀어준다.)

5. **검증** — 로컬 서버(`python3 -m http.server`) + Playwright 스크린샷으로
   마지막 슬라이드에서 버튼과 12개 링크가 렌더링되는지 확인한다.

6. **커밋** — 이 레포는 단일 커밋 히스토리를 유지한다:
   `git add -A && git commit --amend -m "<기사 제목>"` 후
   `claude/…` 작업 브랜치와 `main` 둘 다 `--force` 푸시. 커밋 명의는 로컬
   git config(sckahn) 그대로 두고, Claude 관련 트레일러를 추가하지 않는다.

## 주의
- 플리는 사용자 계정에 비공개로 생성된다. 방문자 공개가 필요하면 사용자가
  스포티파이 앱에서 공개로 전환해야 한다는 사실을 결과 보고에 포함할 것.
- 생성 결과의 트랙 구성은 스포티파이가 자동 선곡하므로, 빠진 앨범이 있는지
  사용자에게 확인을 요청할 것.
