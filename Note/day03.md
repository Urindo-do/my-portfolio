# Day 03 — 네트워크 이해 & 웹프로그래밍 & 외부 API 연동 (2026.08.05)

> 피지컬 AI 입문 과정 3일차.
> 로봇을 조종할 **웹 컨트롤 화면**을 만들기 위한 밑작업 —
> "웹은 어떻게 동작하는가"부터 시작해서 HTML/CSS/JS로 UI를 만들고, 마지막엔 외부 API로 실시간 데이터까지 붙였다.

---

## 1교시 — 네트워크 이해와 개발 환경 구축

### 왜 로봇 제어 화면을 웹으로 만들까?

| 장점 | 설명 |
|---|---|
| 설치 불필요 | 브라우저만 있으면 됨 |
| 기기를 가리지 않음 | PC, 스마트폰, 태블릿 어디서든 |
| 즉시 반영 | 코드를 고치면 새로고침만으로 반영 |

### 네트워크: PC 2대 연결부터 시작

- **네트워크** = 컴퓨터와 컴퓨터 사이에 길을 내는 일
- 컴퓨터끼리 직접 케이블로 연결하면 대수가 늘수록 케이블 수가 폭발함
- 해결책: 가운데에 **라우터(Router)** 를 둔다 → 가닥 수가 확 줄어듦
  - 라우터: IP 주소를 연결해주고, 필요한 곳에 패킷을 전달
- 라우터끼리의 연결이 반복되면 → 거대한 네트워크
- 전 세계를 연결한 네트워크의 주체: **ISP (Internet Service Provider)** — 대형 통신사들
- 우리 집 라우터(공유기)도 이 구조의 말단
- 모든 통신은 **TCP/IP** 라는 공통 규약에 따라 **패킷 단위**로 오간다

### 전 세계 인터넷의 물리적 실체

- 대륙 간 통신은 인공위성이 아니라 **해저 광케이블**이 담당
  - 🔗 [Submarine Cable Map](https://www.submarinecablemap.com/) — 전 세계 해저 케이블 지도
- 케이블을 깔기 어려운 오지·사막·해상은 **저궤도 위성 인터넷(스타링크)** 이 커버
- → 지구 어디든 사실상 인터넷이 닿는 시대

### 인터넷과 웹의 관계

- **인터넷(inter-net)** = 선로 (물리적 연결망)
- **웹(WWW)** = 그 선로 위를 달리는 열차 중 하나 (정보를 주고받는 서비스)

### 웹의 일 = 요청과 응답

```
클라이언트(손님, 브라우저)  ──요청──▶  서버(주방)
클라이언트                 ◀──응답──  서버
```

### URL 구조

```
https://  계정명.github.io  /my-portfolio/
─┬─────   ─┬─────────────   ─┬──────────
프로토콜    도메인             경로
(대화 규칙,  (사람이 기억하는    (서버 안에서
 s=보안)    문자 주소)         파일을 찾아가는 길)
```

### 이름을 번호로 — DNS

- **IP 주소**: 네트워크상에서 컴퓨터를 찾아가는 고유한 숫자 주소
- **DNS (Domain Name System)**: 문자 주소(도메인) → IP 주소로 변환해주는 시스템

### 포트(Port)

- 주소(IP)로 서버를 찾은 뒤, 그 서버 안의 **여러 프로그램 중 누구와 대화할지** 구분하는 번호
- 범위: 0 ~ 65535
- 미리 약속된 번호: **80 = HTTP**, **443 = HTTPS**

### 브라우저가 하는 일 — 렌더링

- 브라우저는 단순한 창이 아니라 **코드를 화면으로 옮겨주는 번역기**
- HTML/CSS/JS 코드를 계산해서 화면으로 그리는 것 = **렌더링(Rendering)**
- 예: 크롬, 엣지, 사파리, 파이어폭스
- 과거 인터넷 익스플로러 시절엔 브라우저·버전마다 해석이 달라 개발이 매우 힘들었음 (표준화의 중요성)

### FE / BE

- **프론트엔드(FE)**: 사용자가 보는 화면 쪽 — 오늘의 주제
- **백엔드(BE)**: 서버 쪽 로직/데이터
- 웹을 만드는 세 가지 언어:

| 언어 | 역할 |
|---|---|
| HTML | 뼈대 (구조) |
| CSS | 디자인 (색상·크기·정렬) |
| JS | 두뇌 (동적 동작) |

### 개발 환경 — VS Code + Live Server

- **Live Server(Five Server)**: 내 컴퓨터를 서버로 사용 가능하게 해주는 확장
- `Go Live` 버튼으로 바로 브라우저에서 결과 확인 가능
- **localhost** = 어디로도 나가지 않는 주소 (루프백 IP, `127.0.0.1`)
- 포트 번호도 직접 설정해서 바꿀 수 있음 → 나중에 포트 충돌 나면 여기서 조정 가능

---

## 2교시 — 웹프로그래밍 (HTML / CSS / JS)

**실습 목표: 로봇 컨트롤 웹페이지 제작**

### HTML — 웹의 뼈대

#### 태그(Tag)

- 웹 브라우저에게 웹 요소(제목, 본문, 설정 등)의 역할을 알려주는 명령어(꼬리표)
- `<head>`: 웹 페이지 설정 정보(제목, 스타일 등)를 담는 **비노출 백그라운드 영역**
- `<body>`: 브라우저 화면에 실제로 렌더링되는 **본문 영역**
- `<h1>`: 대제목 / `<p>`: 본문 문단

#### 엘리먼트(Element) — 태그와 내용이 쌍을 이룬 완성 단위

```html
<p> 안녕하세요 </p>
─┬─ ─┬──────  ─┬──
여는  내용      닫는
태그  (렌더링됨) 태그
```

→ 이 전체 하나가 **엘리먼트**

#### 관련 있는 엘리먼트 묶기 — 컨테이너 태그 & 클래스

- `<div>`: 화면상 변화는 없지만, 흩어진 엘리먼트들을 **하나의 논리적 상자**로 그룹화
- `class="control-card"`: 묶은 구역에 이름표를 붙여, 나중에 카드 전용 스타일을 지정할 수 있게 준비

```html
<div class="control-card">
  <!-- 로봇 제어 엘리먼트들 -->
</div>
```

#### 닫는 태그가 없는 빈 엘리먼트(Empty Element)

- 콘텐츠를 감싸지 않고, 지정된 위치에 단독으로 역할을 수행하는 태그
- 속성(Attribute)에만 의존해 작동 — 예: `<input>`, `<img>`

### CSS — 시각적 디자인

- **Cascading Style Sheets**: 뼈대에 색상·크기·정렬 등 시각 서식을 입히는 도구
- **선택자(Selector)** 로 대상을 지정해 스타일 적용

```css
img {
  width: 60px;   /* img(선택자)의 너비를 60px로 */
}
.control-card {  /* class로 묶은 덩어리에 한 번에 적용 */
  ...
}
```

- HTML 안에 `<style>` 태그로 내장 가능 (오늘 실습 방식)
- 이미지가 너무 클 때 액자 사이즈 조절도 CSS로 해결

### JavaScript — 실시간으로 돌아가는 웹의 두뇌

- HTML이 뼈대, CSS가 디자인이라면 **JS는 두뇌**
- 프로그래밍 언어이며, **브라우저 자체에 내장**되어 있음
- 엘리먼트들을 찾아가서 동작을 수행하는 역할
- JS는 FE를 넘어 **BE까지 확장**됨 (Node.js)
- 기본 사용: `<script> ... </script>` 안에 작성

#### DOM (Document Object Model)

- 브라우저가 HTML을 분석해 **메모리에 트리 구조로 올린 '웹 문서 지도'**
- JS는 이 지도를 통해 특정 태그를 과녁처럼 지목해서 제어함
- → JS가 엘리먼트/이벤트를 컨트롤하려면 이 구조를 알아야 함

#### 이벤트(Event) 연동 흐름

1. 제어할 HTML 엘리먼트를 선택한다 → `document.querySelector`
2. 감시할 이벤트와 실행할 행동을 지정한다 → `addEventListener`

```js
const btn = document.querySelector('button');  // DOM에서 엘리먼트 찾기

btn.addEventListener('click', () => {
  alert('로봇 제어를 시작했습니다');
});
```

#### 클릭할 때마다 상태 토글하기

```js
btn.addEventListener('click', () => {
  if (btn.textContent === '동작 시작') {
    btn.textContent = '동작 중지';
  } else {
    btn.textContent = '동작 시작';
  }
});
```

→ **화면의 텍스트를 읽어와(감지) → 조건에 따라 바꾼다(변경)** 가 핵심 포인트

#### 슬라이더 제어

- `disabled` 속성이 걸려 있으면 움직이지 않음 → JS로 활성화 제어 가능
- `const slider = document.querySelector(...)` 로 가져와서 **style도 JS에서 직접 변경 가능**
- 각도 값도 실시간으로 화면에 표시 가능

#### id vs class

| 구분 | 용도 |
|---|---|
| `id` | 페이지 내 **유일한** 값 (예: `#temp`) |
| `class` | **여러 엘리먼트**에 공통으로 부여 가능 |

> 💡 이후 서보모터 제어에서도 이 UI를 활용할 예정.
> **FE 페이지의 조작이 실제 서보모터 동작으로 이어지게 하는 것**이 다음 핵심 과제.

### 실습 코드 — 로봇 컨트롤 페이지

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>첫 페이지</title>
  <style>
    /* 전체 배경 및 폰트 설정 */
    body {
      background-color: #1a1a2e; /* 어두운 배경색 */
      color: #e0e0e0;            /* 밝은 글자색 */
      text-align: center;        /* 가운데 정렬 */
    }
    h1 {
      color: #00d4ff;
      margin-bottom: 30px;
    }
    /* 제어 패널 (카드) 스타일 */
    .control-card {
      background-color: #162447; /* 카드 배경색 */
      padding: 20px;             /* 안쪽 여백 */
      border-radius: 12px;       /* 모서리 둥글게 */
      max-width: 300px;          /* 가로 크기 제한 */
      margin: 0 auto;            /* 화면 가운데 정렬 */
    }
    .control-card img {
      width: 60px;
      margin-bottom: 16px; /* 아래 요소와의 간격 */
    }
    .control-card h3 {
      color: #ffffff;      /* 흰색 글자 */
      margin: 10px 0;      /* 위아래 여백 */
    }
    .control-card p {
      color: #a0a0b0;      /* 연한 회청색 */
      margin-bottom: 20px; /* 아래 여백 */
    }
    /* 버튼 스타일 */
    button {
      background-color: #00d4ff; /* 버튼 배경색 */
      border: none;              /* 테두리 제거 */
      padding: 10px 20px;        /* 버튼 여백 */
      border-radius: 6px;        /* 버튼 모서리 */
      display: block;            /* 세로 배치를 위해 블록으로 변경 */
      width: 100%;               /* 가로폭 가득 채우기 */
      margin-bottom: 15px;       /* 아래 슬라이더와의 간격 */
    }
    /* 슬라이더(Range) 스타일 */
    input[type="range"] {
      display: block;            /* 세로 배치를 위해 블록으로 변경 */
      width: 100%;               /* 가로폭 가득 채우기 */
    }
  </style>
</head>
<body>
  <h1>나의 첫 웹 페이지!</h1>

  <div class="control-card">
    <img src="img/robot_icon.png" alt="로봇 아이콘">
    <h3>로봇 팔 제어하기</h3>
    <p>아래 버튼과 슬라이더를 이용해 각도를 조절하세요.</p>

    <button>동작 시작</button>
    <input type="range" min="0" max="180" value="90">
  </div>
</body>
</html>
```

---

## 3교시 — 외부 API 연동 (날씨 대시보드)

### 실습 단계

1. **API 개념 이해**: 클라이언트-서버-API의 기본 개념
2. **fetch API 원리**: HTTP 통신 흐름과 비동기 요청 / JSON 데이터
3. **HTML 뼈대 작성**: 날씨 대시보드 UI 마크업
4. **다크 모드 카드 스타일링**: CSS로 완성도 높이기
5. **fetch API 연동 (콘솔)**: 데이터 요청 후 콘솔 출력으로 확인
6. **DOM 데이터 바인딩 (최종)**: 수신된 온도 값을 화면에 실시간 노출

### API란?

- **Application Programming Interface** — 데이터를 주고받을 때의 **약속과 통로**
- 데이터 연동의 필요성: 내가 갖고 있지 않은 데이터를 외부에서 가져와 쓸 수 있음

```
클라이언트  ──"날씨 데이터 주세요"──▶  API  ──▶  서버(기상 관측 DB 보유)
클라이언트  ◀────── JSON 응답 ──────  API  ◀──  서버
```

- API는 서버의 DB를 직접 열어주는 대신, **외부 개발자가 정해진 방식으로 가져갈 수 있게** 만든 창구
- API를 쓰면 **연산은 남의 서버가 하고, 결과만 내가 가져올 수 있음**
  - 예: OpenAI / Claude / Google Gemini API
  - 활용 분야: 결제·커머스, 지도·위치 정보, 인증·소셜 로그인 등
- 흐름: **요청 → 처리(서버) → 응답**

### JSON — 데이터를 담는 포맷

- **JavaScript Object Notation**: 이름(key)과 값(value)의 쌍을 텍스트로 묶어 표현
- 사람이 읽기 쉽고, 컴퓨터가 분석하기도 좋음

```json
{
  "latitude": 35.2,
  "current_units": { "temperature_2m": "°C" },
  "current": { "time": "2026-08-05T06:45", "temperature_2m": 30.9 }
}
```

→ 이 중에서 **쓸 만한 값(`current.temperature_2m`)만 골라서** 화면에 바인딩

### fetch — 브라우저 내장 통신 함수

```js
fetch('서버가 제공하는 API 주소')
  .then(response => response.json())  // 응답을 JSON으로 파싱
  .then(data => console.log(data));   // 파싱된 데이터 사용
```

- 주소만 넣으면 응답을 받아 파싱까지 이어짐
- **새로고침 없이 실시간으로 데이터를 갱신**할 수 있는 게 가장 큰 장점
- 오늘은 무료 공개 API인 **Open-Meteo** 사용 (정제·가공된 데이터를 주는 유료 API도 존재)
- 사용한 주소:
  `https://api.open-meteo.com/v1/forecast?latitude=35.1768&longitude=126.9061&current=temperature_2m`
  (전남대 좌표, 현재 기온 요청)

### 실습 코드 — 실시간 날씨 대시보드

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>실시간 날씨 대시보드</title>
  <style>
    body {
      background-color: #121214;
      color: #e1e1e6;
      font-family: sans-serif;
      text-align: center;
      padding-top: 50px;
    }
    .weather-card {
      background-color: #202024;
      border-radius: 12px;
      padding: 20px;
      max-width: 300px;
      margin: 0 auto;
      border-left: 6px solid #00d4ff;
    }
  </style>
</head>
<body>
  <h1>📍 실시간 날씨 대시보드</h1>

  <div class="weather-card">
    <h3>광주(전남대) 날씨</h3>
    <p>현재 기온: <span id="temp" style="font-weight: bold; color: #00d4ff;">조회 중...</span></p>
  </div>
  <script>
    const apiUrl = 'https://api.open-meteo.com/v1/forecast?latitude=35.1768&longitude=126.9061&current=temperature_2m';
    fetch(apiUrl)
      .then(response => response.json())
      .then(data => {
        // 1. JSON 데이터의 'current.temperature_2m' 경로에서 기온 추출
        const temp = data.current.temperature_2m;

        // 2. 화면 내 기온 출력 대상 태그(#temp) 선택 후 기온 값 바인딩
        document.getElementById('temp').textContent = `${temp}°C`;
      });
  </script>
</body>
</html>
```

---

## 오늘의 발견 & 감상

- **Live Server**: "굳이?"가 아니라 진짜 편하게 즉시 테스트 가능. 포트도 직접 설정해서 바꿀 수 있어서, 나중에 포트 충돌이 나면 조정하면 되겠다는 대비책도 생김
- **Partial Diff 확장** (수업 외 자율 학습): 예제 코드와 내 코드를 나란히 비교하며 학습 — 전에는 이런 비교를 LLM에게 물어봤는데, 이제 스스로 확인할 수 있는 도구가 생김
- **짧은 시간에 FE/BE 전반을 조망**: HTML 구조 → CSS → JS `<script>` 동적 동작까지 한 번에. 이후 로봇 제어에 바로 추가 활용할 수 있겠다는 감이 옴
- **React 경험과의 연결**: 전에 React 하면서 봤던 JSX가 사실 이 구조 위에 있었구나 — 한 단위가 element이고, `div`로 묶어 class처럼 관리하고, `.클래스명` 선택자로 스타일을 채우는 원리를 제대로 처음 봄
- **이벤트 핸들러 문법**: `click => ...` 같은 문법이 실제로 어떤 엘리먼트를 건드리는지까지 눈으로 확인 가능
- **JS의 정체**: 언어로서 HTML 안에서도 동적 스크립트 역할을 한다는 걸 처음 앎. 렌더링·DOM 이야기도 복습이 됨
- **API 관점의 전환**: 통신 → 서버 접근 → JSON 패킹 → fetch → CRUD 흐름이 복습되니, 코드가 "데이터를 가져오는 관점"으로 그냥 읽히기 시작함. BE도 JS(Node.js)로 된다는 것도 신기한 포인트

## 다음으로 이어지는 것

- FE 페이지의 버튼/슬라이더 조작이 **실제 서보모터 동작으로 이어지게** 만드는 것 (4일차: 스마트폰으로 PC 자원 제어)
